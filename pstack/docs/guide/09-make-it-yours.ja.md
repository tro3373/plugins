# 自分のものにする

poteto-mode は一人のスタイルだ。その下にある機械 (プレイブック、ルーティング、モデルの役割) は、あなたのスタイルを着せても同じように動く。このページでは、個人用モードの生成、セッションからの学びの取り込み、焦点を絞った skill の作成、そして信頼する前に skill の変更をテストすることを扱う。

## `/automate-me` で自分のモードを生成する

```text
/automate-me
```

自分のスタイルを説明する必要はない。[`/automate-me`](../../skills/automate-me/SKILL.md) が履歴からそれを読み取るからだ。アクティブなワークスペースの最近のトランスクリプトを掘り、返答の好み、委譲、検証、コード、散文、プロセスにおける繰り返しの選好を見つけ、そのうちどれが本当にあなたらしいかを尋ねる。Cursor 組み込みの `create-skill` フローを通して `.cursor/skills/<your-name>-mode/SKILL.md` を起草し、そのドラフトを [`/unslop`](../../skills/unslop/SKILL.md) に通し、worktree から PR を開くので、他の変更と同じようにレビューできる。

習慣がずれてきたら、いつでも再実行する:

```text
/automate-me update my mode skill with everything since its last edit
```

update モードは、skill が最後に変更されて以降の履歴だけを掘る。あなたが否定していないルールは維持し、新しい証拠があるものは改訂し、本当に新しいパターンについてだけセクションを追加する。

## `/reflect` でセッションの学びを取り込む

何かを学んだタスクの直後に実行する:

```text
/reflect that took way too long. capture what we learned so the next run doesn't repeat it.
```

[`/reflect`](../../skills/reflect/SKILL.md) はトランスクリプトを 3 体の並列レビュワーに送り、続いて synthesizer が提案を `Accepted`、`Rejected`、`Backlog` に仕分け、skill を変更する前にあなたの承認を待つ。提案を承認するのは、それが将来の判断を変える場合だけにする。奇妙なセッション 1 回は逸話であってルールではない。

## 焦点を絞った skill を書く

取り込みたいワークフローが既に分かっているとき:

```text
/poteto-mode write a skill for verifying database migrations in this repo
```

skill を書くことは [skill の作成・変更のプレイブック](../../skills/poteto-mode/playbooks/authoring-a-skill.md) に対応し、Cursor 組み込みの `create-skill` を経由し、frontmatter とリンクを検証し、PR を開くプレイブックで結果を出す。エージェント向けの散文は人間向けの散文より基準が高い。役に立たない 1 文が、将来のエージェントが従う指示になるからだ。`SKILL.md` を手ずから書くのではなく、その基準はプレイブックに保たせる。

特別なケースが 1 つあり、専用のジェネレータを持つ。アプリを動かして挙動を証明しなければならない skill は検証 skill なので、代わりに [`/create-verification-skill`](../../skills/create-verification-skill/SKILL.md) と [`/maintain-verification-skill`](../../skills/maintain-verification-skill/SKILL.md) を使う。[検証して出す](./06-verify-and-ship.ja.md#プロジェクトの検証-skill-を作る) が両方を扱う。

## `/technical-writing` で標準に沿ってドキュメントを書く

あなたが出す散文は skill だけではない。docs、RFC、readme、PR の説明文、コミットメッセージには:

```text
/technical-writing review the readme changes
```

[`/technical-writing`](../../skills/technical-writing/SKILL.md) は、疲れたエンジニアが一読で理解できる散文という 1 つのゴールを持つ階層化された標準を適用する。まずドキュメントのモード (tutorial、how-to、reference、explanation) を選び、次に文単位で作業する: 誰が何をするか、1 文 1 思考、二通りに読めるものが無いこと。あなたやエージェントが今書いたものをレビューするのに使うか、ドキュメントを依頼するときに先に名指しして使う。

## skill の変更をブラインドでテストする

skill の編集は将来のすべてのセッションに影響するので、実験としてテストする:

```text
/poteto-mode run the eval playbook on this skill change. same task for both variants, candidates stay blind.
```

[Eval プレイブック](../../skills/poteto-mode/playbooks/eval.md) は 1 つの失敗モード、観察者効果を中心に組み立てられている。評価されていると知っているエージェントは違う振る舞いをする。だから候補エージェントには、サニタイズされたディレクトリで自然に見えるタスクが与えられ、"eval" や "candidate" という語は決して使われず、互いの存在も決して知らされない。1 体のジャッジが中立なラベルの下ですべての出力を採点し、連鎖に従ったかは各候補が実際にどのファイルを読んだかから採点され、候補の主張からは採点されない。

評決を受け入れる前に、すべての出力を自分の目で読む。ジャッジに同意できないなら、自分の判断を疑う前にルーブリックを疑う。

**落とし穴:** 挙動がおかしいからといってタスクの途中で skill を編集しないこと。それ専用の PR で直し、タスクは進め続ける。機能の作業に絡まって出た skill の編集は、レビューから見えず、評価も不可能になる。

次: [レシピと落とし穴](./10-recipes-and-pitfalls.ja.md)。
