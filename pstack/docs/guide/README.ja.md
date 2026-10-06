# pstack ガイド

pstack はエージェントのマイクロマネジメントをやめたときに最もよく働く。何が欲しいか、どうなれば完了とわかるかを述べる。`/poteto-mode` がプレイブックを選び、ステップが要求するままに他の skill を呼び、証拠を見せる。このガイドは現実的なプロンプトを通してその習慣を教える。

学ぶ内容は次のとおり:

1. [pstack をセットアップする](./01-setup.ja.md)。プラグインを入れ、モデルを選ぶ。
2. [`/poteto-mode` に作業を流す](./02-poteto-mode.ja.md)。ゴールを渡し、プレイブックが選ばれるのを見る。
3. [コードを理解する](./03-understand.ja.md)。読み取り専用の調査から入り、編集の前に `/how`、`/why`、`/teach`、`/recall`。
4. [変更を設計する](./04-design.ja.md)。コードが形を固める前に `/architect`、`/arena`、`/swarm`、`/interrogate`、プロトタイプ、プラン。
5. [変更を作り、掃除する](./05-build-and-clean.ja.md)。ビルド系プレイブックと `/tdd`、`/unslop`、`/no-comments`。
6. [検証して出す](./06-verify-and-ship.ja.md)。実アプリで挙動を証明し、`/benchmark-checklist` で数値を吟味し、焦点の絞られた PR を開いて merged まで運ぶ。
7. [寝ている間に作業を走らせる](./07-overnight.ja.md)。ループの前に信頼を築くこと、夜間の契約、監査できる決定ログ、エージェント 1 体を超えてスケールする Projects と automation。
8. [原則名で舵を取る](./08-principles.ja.md)。タスクの途中でエージェントの向きを変える 24 の名前。
9. [自分のものにする](./09-make-it-yours.ja.md)。自前のモード、繰り返す間違いに効く `/correct`、skill 変更のテスト方法。
10. [レシピと落とし穴](./10-recipes-and-pitfalls.ja.md)。コピーして使えるプロンプトと、避けるべき失敗。

最初は順番に読む。その後は各ページが単独で成立する。

行き詰まったとき、またはどの skill が合うか分からないときは、質問を添えて [`/poteto-help`](../../skills/poteto-help/SKILL.md) と打つ:

```text
/poteto-help which skill should i use to review this branch?
```

答えを返し、送るべきプロンプトを渡し、答えの出どころとなった skill やガイドのページへのリンクを示す。作業そのものは始めない。pstack の実行は実際にトークンを使うので、準備ができたらあなたがプロンプトを送る。自分で打ったときだけ動く。

## 1 つだけ覚えるなら

ゴールと、それを確認する方法を、自分の言葉でエージェントに渡す:

```text
/poteto-mode the export writes duplicate rows when a retry lands mid-run. repro first, then fix and verify.
```

プレイブック名を挙げる必要も skill を列挙する必要もない。「repro first」とチェック可能な結果、それだけで `/poteto-mode` に必要なルーティング信号は足りている。Bug fix プレイブックにマッチし、ステップを todo リストへコピーし、各ステップが発火するたびに適切な skill を呼ぶ。

次: [pstack をセットアップする](./01-setup.ja.md)。
