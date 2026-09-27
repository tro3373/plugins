---
name: reflect
description: アクティブなトランスクリプトに対して 3 つのレビューサブエージェントを並列に起動し、学びを洗い出して、それぞれを既存 skill への具体的な編集へ振り分ける。ユーザが reflect と言ったときに使う。
disable-model-invocation: true
---

# Reflect

現在の会話から持続的な学びを掘り出し、それを skill の編集へ振り分ける。

## いつ起動するか

ユーザが "reflect" または "/reflect" と言ったときに起動する。会話が些末なとき、話題から外れているとき、あるいは既存 skill が既にカバーしていて親が正しく従っていたときは飛ばす。一度きりの事象は学びではない。

## プロセス

### 1. アクティブなトランスクリプトを特定する

親はファンアウトの前に自分のトランスクリプトファイルを見つける。システムプロンプトがアクティブなワークスペースの `agent-transcripts/` ディレクトリを示している。そのパスを使う。`~/.cursor/projects/*/` を横断して glob してはならない。ワークスペースの境界を越え、無関係なプロジェクトの私的チャットを読むことになる。

```bash
ls -t <agent-transcripts>/*.jsonl <agent-transcripts>/*/*.jsonl <agent-transcripts>/*/subagents/*.jsonl 2>/dev/null | head -10
```

トランスクリプトのレイアウトは 3 種類: レガシーのフラット (`<id>.jsonl`)、現行のネスト (`<id>/<id>.jsonl`)、サブエージェント (`<parent>/subagents/<child>.jsonl`)。

各候補について、JSONL の最初の行を読み、`message.content[0].text` に会話の冒頭のユーザプロンプトが含まれるか確認する。一致したパスを採る。どのパスも解決しないなら、セッションの引き締まった要約を書き、それを代わりに渡す。

### 2. レビュワー 3 体を並列に起動する

1 メッセージ、3 つの `Task` 呼び出し、`subagent_type: generalPurpose`、`model` は下記のとおり設定し、エージェントモード (`readonly: false`)。レビュワーは文脈の参照 (チケット、チャットスレッド、トランスクリプトで参照される可観測性のトレース) のために MCP アクセスを必要とする。readonly は MCP を剥がす。

各レビュワーと synthesizer は、`pstack-models.mdc` ルール中のロール行と既定値を指す。`model` にはその行の値を、ルールまたはその行が無ければ既定値を設定する。値が `auto` または `inherit-parent` のときは `model` を未設定のままにする。Task ツールがスラッグを拒否したら既定値を使い、その旨を述べる。既定値も拒否されたら、そのエラーメッセージにある同じファミリーの中で最も近い有効なスラッグを使う。

| レンズ | ロール行 | 既定の `model` | プロンプトテンプレート |
|---|---|---|---|
| Judgment | `reflect judgment, divergent, synthesizer` | `claude-opus-5-5-max` | `references/judgment-reviewer.md` |
| Tooling | `reflect tooling` | `gpt-5.6-sol-max` | `references/tooling-reviewer.md` |
| Divergent | `reflect judgment, divergent, synthesizer` | `claude-opus-5-5-max` | `references/divergent-reviewer.md` |

各テンプレートは、印の付いた箇所にトランスクリプトのパスか要約を差し込むだけで、そのまま渡す。レビュワーは `Task` のレスポンス本文で発見を返す。

### 3. 統合する

`Task` 呼び出し 1 つ、`subagent_type: generalPurpose`、`model` は `reflect judgment, divergent, synthesizer` 行から (既定 `claude-opus-5-5-max`)、エージェントモード (`readonly: false`)。synthesizer の品質チェックには引用の抜き取り検証が含まれ、それに MCP アクセスが要ることがある。readonly は MCP を剥がす。`references/synthesizer.md` をそのまま使い、印の付いた箇所に各レビュワーの全出力をインライン展開する。synthesizer は構造化された Accepted / Rejected / Backlog のリストを返す。

### 4. 構造による強制のチェック

synthesizer の Accepted リストを健全性チェックする。lint ルール、スクリプト、メタデータのフラグ、ランタイムのチェックのほうが確実に強制できる項目は、Accepted から Backlog へ移す。**encode-lessons-in-structure** の原則 skill を参照。

### 5. 適用する

Accepted の編集を適用する前に、synthesizer の Accepted / Rejected / Backlog の全出力をユーザに提示し、明示的な承認を待つ。ユーザがどの部分集合を適用するかを選び、振り分け先を変更してもよい。skill の変更は組織の将来のすべてのエージェントに影響する。自動適用してはならない。

Backlog の項目は、チームが使っている devex / backlog トラッカーへ自動で登録する。承認を待つのは Accepted リストだけだ。

承認された Accepted の各項目について、Routing フィールドに厳密に従う:

- 既存 skill への些細な編集 (1 行の箇条書き、文の引き締め、古い事実の訂正): 親が直接行う。
- 既存 skill への実質的な編集 (新しいセクション、新しいパターン表、およそ 10 行超): Cursor 組み込みの `create-skill` skill に渡し、その draft / test / iterate のループを回す。
- `tune description: <skill path>` (skill は存在するが、発火すべきときに発火しなかった): `create-skill` に渡し、その description 最適化のループを回す。
- `new skill via create-skill: <kebab-name>`: 作成を `create-skill` に渡す。その場しのぎで形を作らない。

環境に SKILL.md のバリデータがあるなら、完了を宣言する前に触れたすべての skill に対して実行する。無いならこのステップは飛ばす。

### 6. ユーザ向けにまとめる

短いリスト、前置きなし:

- 適用した編集: `<skill path>`。何が変わったかを 1 行ずつ。
- 新規作成した skill: `<skill path>`。1 行ずつ (稀)。
- devex トラッカーへ起票した Backlog: `<issue title>` (`<tags>`)。1 行ずつ。
- 落としたもの: 却下された発見ごとに 1 行、synthesizer による理由付き。
