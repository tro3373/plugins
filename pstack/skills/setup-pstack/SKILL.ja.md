---
name: setup-pstack
description: pstack がロールごとにどのモデルを、どの reasoning budget で使うかを設定する。利用可能なモデルを検出し、skill のデフォルトを上書きする always-applied ルールを書き出す。/setup-pstack、「configure pstack models」、「pstack budget」、pstack のモデル選択を変更したいときに使う。
---

# Setup pstack

`~/.cursor/rules/pstack-models.mdc` を書く。これは pstack のロールごとのモデルを設定する always-applied ルールである。

## 手順

### 1. 利用可能なモデルを検出する

このセッションで `Task` サブエージェントに渡せるモデル slug を列挙する。これが信頼できる情報源だ。Cursor がユーザの権限付きモデルを列挙するモデル API や CLI も公開しているなら、網羅性のためそちらを優先する。何も検出できないなら、アクセスできる slug を貼り付けるようユーザに頼む。利用可能だと確認していない実在の slug を書いてはならない。エイリアス `inherit-parent` と `auto` は、検出された slug でなくても常に有効である。

### 2. 現在の状態を読み込む

ロールとモデルの既定の対応は、下の手順 5 に示すルールの形そのものである。`~/.cursor/rules/pstack-models.mdc` が既に存在するなら、それを読み、その `# budget` 行とロールの値を現在の選択として扱う。存在しないならそのデフォルトから始める。手順 5 のロールに無いロールの行 (例えば `how critics`) は廃止されたロールのものだ。取り除く。

### 3. budget、マッピング、確認

**(a) budget を尋ねる。** 自由記述より AskQuestion を優先する。次の 4 つの選択肢を、このラベルのまま正確に提示する。ルールに既存の budget が記録されている場合はそれも名前を挙げる。

- `unlimited — keep max`
- `large — xhigh reasoning`
- `medium — high reasoning`
- `small — medium reasoning`

**(b) 適用する。** 作業用テーブルを skill のデフォルトから組み立て、再実行時はファミリ・リスト・エイリアス (`inherit-parent`、`auto`) で変更済みのロールをそのまま保つ。`unlimited` はそのテーブルのすべての effort をそのままにする。`large`、`medium`、`small` は、パネルのエントリも含め、すべての実在 slug の effort トークンを `xhigh`、`high`、`medium` のいずれかに設定する。effort トークンとは、`max` > `xhigh` > `high` > `medium` > `low` の段階のうち最後のトークン、または末尾に `fast` が付く場合はその手前のトークンである。結果が検出済みの slug でなければ、同じファミリの中で目標以下の最も高い effort を持つ検出済み slug を使い、それも無ければそのロールを選び直しが必要な印にする。`inherit-parent` と `auto` は変化しない。したがって `small` は `claude-opus-5-5-max` を `claude-opus-5-5-medium` に、`grok-4.7-xhigh-fast` を `grok-4.7-medium-fast` に変える。

**(c) ロールを示して確認する。** すべてのロールをモデルとともに示し、検出済みの集合に無い実在の slug には選び直しが必要である旨の印を付ける。手順 2 で取り除いた行もそれぞれ列挙する。そのまま受け入れるか、特定のロールを変更するかを尋ね、選択肢として検出されたモデルに加えて `inherit-parent` と `auto` (どちらも「このロールは親のチャットモデルで走る」という意味で、Auto ユーザが Auto のままでいられる方法) を提示する。自由記述より AskQuestion を優先する。パネル系のロール (arena の runners、architect の runners、interrogate の reviewers) では値がリストになり、エントリ 1 つにつきサブエージェントが 1 本走る。エイリアスのエントリも含まれるので、リストの長さがその本数を決める。`arena cross-judge pool` もリストだが、Arena はその中から、可能な限り親とは異なるモデルファミリの値を 1 つ選ぶ。`swarm workers` は、レースや比較でアームごとに別のモデルが割り当てられない限り、すべての worker の既定モデルとなる。

### 4. 検証する

書き込まれる実在の slug はすべて検出済みの集合に含まれていなければならない。`inherit-parent` と `auto` は常に通る。選ばれた実在の slug が利用不可なら、そこで止めてもう一度尋ねる。

### 5. ルールを書く

`~/.cursor/rules/pstack-models.mdc` を `alwaysApply: true` 付きで、選んだ budget のラベルと目標 effort を記した `# budget` 行とともに書き、ロールごとに 1 行ずつ、poteto-mode が使うのと同じラベルを用いる。再実行が冪等になるよう、ファイル全体を上書きする。形は次のとおり。

```
---
description: pstack per-role model choices (overrides skill defaults)
alwaysApply: true
---
# pstack model configuration. One line per role. Delete a line to fall back to the skill default.
# `inherit-parent` or `auto` as a value: the role runs on the parent chat model (omit Task `model`). Alias entries in a panel list still count toward its fan-out.
# budget: unlimited (max)
feature, refactoring: grok-4.7-xhigh-fast
bug-fix: grok-4.7-xhigh-fast
perf-issue: grok-4.7-xhigh-fast
hillclimb: grok-4.7-xhigh-fast
judgment and prose: claude-opus-5-5-max
hardest tasks: claude-opus-5-5-max
how explorer: grok-4.7-xhigh-fast
how explainer: claude-opus-5-5-max
why investigators: grok-4.7-xhigh-fast
why synthesizer: claude-opus-5-5-max
reflect tooling: gpt-5.6-sol-max
reflect judgment, divergent, synthesizer: claude-opus-5-5-max
arena runners: claude-opus-5-5-max, gpt-5.6-sol-max, grok-4.7-xhigh-fast
arena cross-judge pool: claude-opus-5-5-max, gpt-5.6-sol-max, grok-4.7-xhigh-fast
swarm workers: grok-4.7-xhigh-fast
architect runners: claude-opus-5-5-max, gpt-5.6-sol-max, grok-4.7-xhigh-fast
interrogate reviewers: claude-opus-5-5-max, gpt-5.6-sol-max, grok-4.7-xhigh-fast
```

### 6. 報告する

ルールを書いたこと、そしてそれが新しいセッションに適用されることをユーザに伝える。この skill を再実行すれば更新される。

### 7. 検証 skill を提案する (任意)

プロジェクトに、証拠のために実際のアプリを駆動する手段 (`verify-*` skill、または既存のハーネス) があるかを確認する。無ければ一度だけ提案する: 「プロジェクトローカルの検証 skill が欲しいか。エージェントがユーザと同じようにアプリを駆動し、変更が動くことを証明できるようになる。/create-verification-skill で生成できる」。yes なら `/create-verification-skill` を起動する (pstack がインストールされている場所 - workspace、user、plugin のいずれか - で解決される)。no なら押し付けずに先へ進む。
