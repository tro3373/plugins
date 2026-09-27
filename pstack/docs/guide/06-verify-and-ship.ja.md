# 結果を検証して PR を出す

「コンパイルが通った」は証拠ではない。[Prove It Works の原則](../../skills/principle-prove-it-works/SKILL.md) は、成功を報告する前に本物の成果物をエージェントに確認させる。そして「本物の成果物」を確認可能にするのはあなたの仕事である。このページでは、完了条件を述べること、自分のアプリ用の検証 skill を生成すること、PR を出すこと、そしてそれをマージ可能な状態まで持っていくことを扱う。

![試作機が実際のテストコースを飛び、彼女がストップウォッチで計測し、ロボットが飛行を撮影しチェックリストを付ける。ターミナルには verify: pass, evidence: captured と表示されている。](./images/verification.jpg)

## 完了条件を最初に述べる

done が何を意味するかを、どんな言葉でもいいので最初のプロンプトに入れる:

```text
/poteto-mode add json output to this command. text output stays byte-identical, the json parses, both run against the sample project. show me the evidence.
```

これでエージェントは、満たすべき気分ではなく、実行できるチェックを 3 つ手にする。返答が返ってきたら、そこには正確なコマンドと出力が載っているはずである。チェックが実行できなかったなら、良い返答は「inconclusive」と言う。証拠のない自信満々の返答は危険信号として扱うべきである。

チェックは変更に合わせる:

- CLI の変更なら、実際のコマンドを実行する。
- UI の変更なら、動いているアプリで変更したフローを歩く。
- パーサやマイグレーションなら、保存した入力を再生する。
- パフォーマンスの変更なら、前後のプロファイルを比較する。
- ストレージの変更なら、書き込んだ値を読み戻す。

完全には信用しきれない小さな diff には、[`/blast-radius`](../../skills/blast-radius/SKILL.md) が、それが他のどこを壊しうるかを見つける。この skill は、変更が安全である根拠となる 1 つの事実を選び、それについて小論文を書く代わりにコードを実行して証明する。

## プロジェクトの検証 skill を作る

上の UI の項目は、本当の要件を隠している。エージェントには、アプリを操作するスクリプト化された手段が必要である。プロジェクトに既にあるなら結構。ないなら、これを実行する:

```text
/create-verification-skill
```

[`/create-verification-skill`](../../skills/create-verification-skill/SKILL.md) は、あなたではなくリポジトリにインタビューする。ユーザが何に触れるか、アプリはローカルでどう起動するか、何がそれを操作できるか (既存のハーネスが最優先、なければブラウザと CDP、PTY、あるいは素の HTTP)、何が挙動を証明する証拠になるか、2 インスタンスを並べて実行できるかを割り出す。あなたに聞くのは、コードが答えられないことだけである。

これは `.cursor/skills/verify-<app>/` を書き出す - Launch、Doctor、Drive、Evidence、Cleanup の各セクションを正確に持つエージェント向けの手順書に加えて、`features/` の下に、アプリが何をするか、そして各機能が動いていることを何の結果が証明するかを索引した feature map を置く。この skill には [feature map の実例](../../skills/create-verification-skill/references/feature-map-example/) が同梱されており、README の索引と、必須の 4 つの H2 を使った機能ごとに 1 ファイルの構成になっている。引き渡す前に、ジェネレータはその skill をエンドツーエンドで 1 回証明する - 起動し、doctor でチェックし、1 機能を操作し、証拠を取得し、後始末する。この証明が失敗したなら、その出力を使ってはならない。

以降、「アプリで検証する」は、このリポジトリで、セットアップの会話なしに、どのエージェントでも実行できる 1 ステップになる。

verify skill が動くようになれば、[`/swarm`](../../skills/swarm/SKILL.md) が feature map のエントリ単位でフルパスを分割し、結果を集約できる。

## 検証 skill を正直に保つ

アプリは変わり、feature map は腐る。ドリフトしたら、これを実行する:

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

次: [眠っている間に作業を回す](./07-overnight.ja.md)。
