# 変更を作り、diff を掃除する

build 系の playbook はひとつの規律を共有している。観測したことを述べ、証拠の要求は playbook に任せる。このページでは、よくある build タスクごとにプロンプトへ何を書くかを示し、その後に diff をレビュー可能に保つ掃除の習慣を扱う。

## 知っていることを各 build playbook に渡す

バグのプロンプトは症状を述べ、まず再現を求める:

```text
/poteto-mode this command emits two records after a retry. repro first, then fix and verify.
```

機能のプロンプトは挙動と、変わってはいけないものを述べる:

```text
/poteto-mode add a --json flag. text output stays byte-identical. verify both forms.
```

リファクタリングのプロンプトは、構造が動く前に挙動を固定する:

```text
/poteto-mode move parsing into one module, zero behavior change. record the current output first and prove it's unchanged after.
```

パフォーマンスのプロンプトは、雰囲気ではなく測定値を述べる:

```text
/poteto-mode startup takes 1.8s on this fixture. trace it, fix the measured cause, show me before and after.
```

これらはそれぞれ対応する playbook ([Bug fix](../../skills/poteto-mode/playbooks/bug-fix.md)、[Feature](../../skills/poteto-mode/playbooks/feature.md)、[Refactoring](../../skills/poteto-mode/playbooks/refactoring.md)、[Perf issue](../../skills/poteto-mode/playbooks/perf-issue.md)) に振り分けられ、playbook が自分で書かなかった手順を補う。修正の前に再現する、実装の前にデータの形を決める、構造を変える前に挙動を固定する、最適化の前にプロファイルを取る。

ひとつの数値を継続的に改善するなら [Hillclimb playbook](../../skills/poteto-mode/playbooks/hillclimb.md) がある。メトリクス、目標、試行回数の下限を与えると、測定ハーネスを凍結したまま一度にひとつの仮説をループする。勝ったものを残し、それ以外はすべて revert する。

## `/tdd` で先に落ちるテストを書く

バグに安価なローカルのテスト経路があるとき、プロンプト全体は 2 語で済む:

```text
/tdd implement
```

文脈があれば、それで十分だ。[`/tdd`](../../skills/tdd/SKILL.md) は意図した理由で落ちる最小のテストを書き、次に修正を書き、そしてテストを再実行する。テストに広範なハーネスの用意や壊れやすいモックが要るなら、この skill はそう述べ、代わりに最も近い実行可能なチェックを使う。本物のコマンドのほうが強い証拠になる場面でテストを無理強いしないこと。

## TypeScript のルールは自分で読み込ませる

[`typescript-best-practices`](../../skills/typescript-best-practices/SKILL.md) には、あなたのワークフロー上のスラッシュコマンドが無い。エージェントが `.ts` や `.tsx` ファイルに触れるたびに読み込まれ、型システムの原則を具体的なルールに変える。判別可能な union、境界での `unknown`、網羅的なバリアント、スキーマ由来の型。

## コミット前に掃除する

[Opening a PR playbook](../../skills/poteto-mode/playbooks/opening-a-pr.md) は各コミットの前に diff へ `/deslop` をかけ、PR の説明とコミット本文には [`/unslop`](../../skills/unslop/SKILL.md) を適用する。`/deslop` は pstack ではなく `cursor-team-kit` プラグインに入っている。持っていないなら、同じ結果を平易な言葉で頼めばいい。実況コメント、根拠のないガード、死んだ互換パス、無関係な編集を取り除く、と。

散文には、`/unslop` に対象と手持ちの追加ルールを渡す:

```text
/unslop the readme changes, no emdashes
```

自分なりの略記は自然にできてくる。この skill は `unslop that, tighten it` のような短いプロンプトからでも意図を十分に読む。

## `/no-comments` でコメントを剥がす

コメントには専用のパスが要る。しかもそれを書いたエージェントに任せてはならない。作者は自分のコメントを、あなたが自分のコメントを守るのと同じように守る。だからレビューの前に、新鮮な目に渡す:

```text
/no-comments the diff
```

[`/no-comments`](../../skills/no-comments/SKILL.md) は [Comment Sicko](../../agents/comment-sicko.md) を起動する。これは読み取り専用のレビュワーで、短い keep リストを持つ。ライセンスヘッダ、公開 API の doc コメント、コードでは表せないことを説明するリンク、作り変えられない外部依存によって強制される挙動。それ以外はすべて消える。自分のコードの中の意外な挙動にはその免除は与えられない。そのコメントはリファクタリングのフラグとして返ってきて、`/no-comments` は受理したフラグを根本原因のところで直す。コメントが「消すな」といった制約を主張しているときは、この skill はその主張を型・テスト・lint として符号化することを提案する。いずれにせよ、コメントは外に出る。

分業の整理は覚えておく価値がある。`/deslop` はコードから slop を掃除し、`/unslop` は散文から掃除し、`/no-comments` はコメントを、それを書いていないレビュワーに渡す。

**落とし穴:** 掃除は任意の仕上げではない。実況コメントと防御的な余分を抱えた diff は、レビュワーには未完成に見えるし、その余分なコードこそ次のバグが隠れる場所だ。diff が水増しに感じたら、レビューで指摘される後ではなく、コミットする前に `deslop it` と言うこと。

次: [検証して出す](./06-verify-and-ship.ja.md)。
