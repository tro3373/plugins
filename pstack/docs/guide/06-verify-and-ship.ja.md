# 結果を検証して PR を出す

「コンパイルが通った」は証拠ではない。[Prove It Works の原則](../../skills/principle-prove-it-works/SKILL.md) は、成功を報告する前に本物の成果物をエージェントに確認させる。そして「本物の成果物」を確認可能にするのはあなたの仕事である。このページでは、完了条件を述べること、測った数値を吟味すること、自分のアプリ用の検証 skill を生成すること、PR を出すこと、そしてそれをマージ可能な状態まで持っていくことを扱う。

多くのエージェント作業で、検証は最も遅いステップである。たいてい人間を待つステップだからだ。エージェントがそれをこなせるようにすれば、あなたはボトルネックでなくなる。省けば、エージェントを増やしてもレビューすべき未検証の作業が増えるだけである。

![試作機が実際のテストコースを飛び、彼女がストップウォッチで計測し、ロボットが飛行を撮影しチェックリストを付ける。ターミナルには verify: pass, evidence: captured と表示されている。](./images/verification.jpg)

## 完了条件を最初に述べる

done が何を意味するかを、どんな言葉でもいいので最初のプロンプトに入れる:

```text
/poteto-mode add json output to this command. text output stays byte-identical, the json parses, both run against the sample project. show me the evidence.
```

これでエージェントは、満たすべき気分ではなく、実行できるチェックを 3 つ手にする。返答が返ってきたら、そこには正確なコマンドと出力が載っているはずである。チェックが実行できなかったなら、良い返答は「inconclusive」と言う。証拠のない自信満々の返答は危険信号として扱うべきである。

チェックは変更に合わせる:

- CLI の変更なら、実際のコマンドを実行する。
- UI の変更なら、動いているアプリで変更したフローを歩く。参照とピクセル単位で一致させねばならないときは、[Visual parity playbook](../../skills/poteto-mode/playbooks/visual-parity.md) が目視で判断する代わりに、凍結したベースラインとスクリーンショットの差分を取る。
- パーサやマイグレーションなら、保存した入力を再生する。
- パフォーマンスの変更なら、前後のプロファイルを比較する。
- ストレージの変更なら、書き込んだ値を読み戻す。

証明は、自分で確かめられる成果物として求める。失敗するテストとその後に通るテスト、前後の動画、トレース、スクリーンショット。修正が既にマージ済みなら、同じチェックを main でもう一度求める。もっともらしい説明より成果物のほうが強い。実行全体を再生しなくても、それに異議を唱えられるからだ。

完全には信用しきれない小さな diff には、[`/blast-radius`](../../skills/blast-radius/SKILL.md) が、それが他のどこを壊しうるかを見つける。この skill は、変更が安全である根拠となる 1 つの事実を選び、それについて小論文を書く代わりにコードを実行して証明する。

## `/benchmark-checklist` で測った数値を吟味する

前後の数値は、うっかり間違えやすい証拠の筆頭である。温まったキャッシュ、片側だけのデバッグビルド、計測区間の中で実際には走らなかった処理。どれも説得力のある高速化を生み出しうる。数値を報告したり、それに基づいて動いたりする前に、こう打つ:

```text
/benchmark-checklist vet the export speedup before it goes in the pr
```

[`/benchmark-checklist`](../../skills/benchmark-checklist/SKILL.md) は 7 つの質問をし、それぞれに実行からの証拠を求める:

1. 何がその数値を制限しているのか。なぜ倍になっていないのか。
2. どの側も本番と同じようにチューニングされた状態で走ったか。
3. 結果はディスク帯域やコア数のような物理的な限界を破っていないか。
4. 何かがエラーになったり、間違った出力を返したりしていないか。
5. 交互に走らせた実行で、中央値と範囲付きで再現するか。
6. ユーザが待つ経路で、エンドツーエンドで意味があるか。
7. 処理は実際に計測区間の中で行われたか。

判定は faster、slower、no measurable difference、inconclusive のいずれかで返り、実行回数、範囲、制限要因が付く。制限要因を名指しできないとき、あるいはチューニングされていない側があったときは inconclusive と言う。`/poteto-mode` は Perf issue と Hillclimb の playbook の中で既にこのチェックリストを走らせるので、自分で打つのは、それらの外で何かを測ったときか、他人の数値が出来すぎに見えるときである。これは [Explain the Number の原則](../../skills/principle-explain-the-number/SKILL.md) を実行可能な形にしたものだ。

## プロジェクトの検証 skill を作る

上の UI の項目は、本当の要件を隠している。エージェントには、アプリを操作するスクリプト化された手段が必要である。プロジェクトに既にあるなら結構。ないなら、これを実行する:

```text
/create-verification-skill
```

[`/create-verification-skill`](../../skills/create-verification-skill/SKILL.md) は、あなたではなくリポジトリにインタビューする。ユーザが何に触れるか、アプリはローカルでどう起動するか、何がそれを操作できるか (既存のハーネスが最優先、なければブラウザと CDP、PTY、あるいは素の HTTP)、何が挙動を証明する証拠になるか、2 インスタンスを並べて実行できるかを割り出す。あなたに聞くのは、コードが答えられないことだけである。

これは `.cursor/skills/verify-<app>/` を書き出す - Launch、Doctor、Drive、Evidence、Cleanup の各セクションを正確に持つエージェント向けの手順書に加えて、`features/` の下に、アプリが何をするか、そして各機能が動いていることを何の結果が証明するかを索引した feature map を置く。この skill には [feature map の実例](../../skills/create-verification-skill/references/feature-map-example/) が同梱されており、README の索引と、必須の 4 つの H2 を使った機能ごとに 1 ファイルの構成になっている。引き渡す前に、ジェネレータはその skill をエンドツーエンドで 1 回証明する - 起動し、doctor でチェックし、1 機能を操作し、証拠を取得し、後始末する。この証明が失敗したなら、その出力を使ってはならない。

以降、「アプリで検証する」は、このリポジトリで、セットアップの会話なしに、どのエージェントでも実行できる 1 ステップになる。特定の形で証明がほしいときは、プロンプトでそれを名指しする:

```text
/poteto-mode build the bulk-archive action. use /verify-<app> to verify your changes and show me a video and screenshots as proof.
```

```text
/poteto-mode repro this with /verify-<app>. if it repros on main, fix it and show me a video as proof.
```

verify skill が動くようになれば、[`/swarm`](../../skills/swarm/SKILL.md) が feature map のエントリ単位でフルパスを分割し、結果を集約できる。verifier の swarm は、十分に大きなサンプルでパフォーマンスの改善を確かめたり、PR を出す前にアプリを fuzz してリグレッションを探したりもする。

検証 skill は使い捨てではなくインフラとして扱う。コミットして、チームの全員と全エージェントが同じ方法でアプリを操作するようにする。そのうえで [レバーを作る](../../skills/principle-build-the-lever/SKILL.md)。エージェントがアプリをクリックして回るための使い捨てスクリプトを書き続けているなら、代わりに skill から呼ぶ小さな操作用 CLI を求める。エージェントが使うトークンは減り、どの実行も再現可能になる。エージェントがうまく使える CLI には次の特徴がある:

- 薄いコマンドを多数並べるのではなく、それぞれが実際の仕事をする、組み合わせ可能な少数のコマンド。
- 破壊的な操作すべてに `--dry-run` オプション。
- 機能を一度に全部ではなく段階的に見せるサブコマンド。
- 代わりに何をすべきかを述べるエラーメッセージ。
- 充実した `--help` テキスト。
- JSON のような機械可読な出力。

ついでに開発環境のセットアップも再現可能にする。シードデータ、テストユーザ、そして毎回同じ方法で環境を立ち上げる 1 つのコマンド。

## 検証 skill を正直に保つ

アプリは変わり、feature map は腐る。少なくとも 1 日 1 回はこれを実行する。誰も覚えておかなくて済むよう、スケジュールされた automation から走らせるのが理想である:

```text
/maintain-verification-skill
```

[`/maintain-verification-skill`](../../skills/maintain-verification-skill/SKILL.md) は生成された skill を監査する - 機能ごとに読み取り専用のソースリーダを 1 つ並列で走らせ、その後、マップされた全機能を操作するライブパスを 1 回走らせる。結末はちょうど 3 つのうち 1 つである。`clean` はカバレッジが完全で出すものがないこと。`changed` は証明済みの修正 1 PR で、検証 skill 自身のディレクトリに限定される。`blocked` はブロッカーを名指しする。プロダクトコードは決して編集しない。ライブパスがプロダクトのリグレッションを捕まえたら、ドキュメントで取り繕う代わりにリグレッションとして報告する。

## PR を出す

```text
/poteto-mode open the pr. small ordered commits, evidence in the description.
```

[PR を出す playbook](../../skills/poteto-mode/playbooks/opening-a-pr.md) は worktree から作業し、成果を小さく順序立てたコミットに rebase し、diff を整え、文章を unslop し、PR のリンクを返す。狭い PR 5 本は太い PR 1 本に勝り、スタックされたフォローアップは膨らんでいくブランチに勝る。

## Babysit で PR をマージ可能な状態まで持っていく

開いた PR は、その瞬間からブロッカーを集め始める。チェックが落ち、レビュワーがコメントし、trunk が動く。その混乱は [Babysit playbook](../../skills/poteto-mode/playbooks/babysit.md) に渡す:

```text
/poteto-mode babysit this pr. get it green.
```

Babysit は同梱のウォッチャで PR を監視し、ブロッカーを順に処理する - まずコンフリクト、次にレビュースレッド、そして CI。既知の修正はすべて 1 回の push にまとめられるので、チェックは修正のたびではなく 1 回だけ再実行される。コメントのトリアージは懐疑的である。人間もボットも、本物の指摘とノイズを同じリストに投げ込むからである。本物の指摘には修正が入り、ノイズは反証をスレッドに投稿した上で退ける。状況だけ知りたいときは小さく尋ねれば、Babysit はループを開始せずに答える:

```text
/poteto-mode check on pr 123. anything outstanding?
```

Babysit はマージ可能な状態で止まる。全て緑でも決してマージしない。マージは別の決定だからである。

## Shipping でスタックを着地させる

緑であることは安全であることと同じではない。着地の準備ができたら、そう言う:

```text
/poteto-mode land the stack.
```

[Shipping playbook](../../skills/poteto-mode/playbooks/shipping.md) は、何かを準備する前に各 PR を独立に検証する。PR ごとに新しいエージェント 1 体が挙動をライブで証明し、変更を判定するエージェントは決してそれを書いたエージェントではない。その上で Shipping は、下から連続して検証済みの区間だけを、既定では GitHub 経由で、Origin の CLI が使えるならそちら経由で、1 PR ずつ着地させ、連鎖を断ち切った最初の PR を報告する。未検証の PR の上にある検証済みの PR は待たされる。マージすればその隙間を下に引き込むことになるからである。

次: [寝ている間に作業を走らせる](./07-overnight.ja.md)。
