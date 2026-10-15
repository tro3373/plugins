---
name: reflect
description: アクティブなトランスクリプトに対して 3 つのレビューサブエージェントを並列に起動し、学びを洗い出して、それぞれを既存 skill への具体的な編集へ振り分ける。ユーザが reflect と言ったときに使う。
disable-model-invocation: true
---

# Reflect

現在の会話から持続的な学びを掘り出し、それを skill の編集へ振り分ける。

ユーザが "reflect" または "/reflect" と言ったときに起動する。会話が些末なとき、話題から外れているとき、あるいは既存 skill が既にカバーしていて親が正しく従っていたときは飛ばす。一度きりの事象は学びではない。

## 1. アクティブなトランスクリプトを特定する

ファンアウトの前に、システムプロンプトが示す `agent-transcripts/` ディレクトリからこの会話のトランスクリプトを見つける。`~/.cursor/projects/*/` を横断して glob してはならない。ワークスペースの境界を越え、無関係なプロジェクトの私的チャットを読むことになる。

```bash
ls -t <agent-transcripts>/*.jsonl <agent-transcripts>/*/*.jsonl <agent-transcripts>/*/subagents/*.jsonl 2>/dev/null | head -10
```

これでレガシーのフラット (`<id>.jsonl`)、現行のネスト (`<id>/<id>.jsonl`)、サブエージェント (`<parent>/subagents/<child>.jsonl`) の各レイアウトを拾える。各候補について、JSONL の最初の行を読む。`message.content[0].text` に会話の冒頭のユーザプロンプトが含まれるファイルを採る。どれも一致しないなら、セッションの引き締まった要約を代わりに渡す。

## 2. レビュワー 3 体を並列に起動する

1 メッセージ、3 つの `Task` 呼び出し、`subagent_type: generalPurpose`、エージェントモード (`readonly: false`)。レビュワーはトランスクリプトが参照するチケット、チャットスレッド、可観測性のトレースを引くために MCP アクセスを必要とし、readonly は MCP を剥がす。

各レビュワーと synthesizer は、`pstack-models.mdc` ルール中のロール行と既定値を指す。`model` にはその行の値を、ルールまたはその行が無ければ既定値を設定する。値が `auto` または `inherit-parent` のときは `model` を未設定のままにする。Task ツールがスラッグを拒否したら既定値を使い、その旨を述べる。既定値も拒否されたら、そのエラーメッセージにある同じファミリーの中で最も近い有効なスラッグを使う。各サブエージェントの reasoning level と fast / standard の速度は、**poteto-mode** skill の Subagents セクションに従って仕事ごとに選ぶ。

| レンズ | ロール行 | 既定の `model` | プロンプトテンプレート |
|---|---|---|---|
| Judgment | `reflect judgment, divergent, synthesizer` | `claude-opus-5-5-xhigh` | `references/judgment-reviewer.md` |
| Tooling | `reflect tooling` | `grok-4.7-xhigh-fast` | `references/tooling-reviewer.md` |
| Divergent | `reflect judgment, divergent, synthesizer` | `claude-opus-5-5-xhigh` | `references/divergent-reviewer.md` |

各テンプレートは、印の付いた箇所にトランスクリプトのパスか要約を差し込むだけで、そのまま渡す。レビュワーは `Task` のレスポンス本文で発見を返す。

## 3. 統合する

`Task` 呼び出し 1 つ、`subagent_type: generalPurpose`、`model` は `reflect judgment, divergent, synthesizer` 行から (既定 `claude-opus-5-5-xhigh`)、エージェントモード (`readonly: false`)。synthesizer は MCP 経由で引用を抜き取り検証するので、readonly では MCP が剥がれて困る。`references/synthesizer.md` をそのまま渡し、印の付いた箇所に各レビュワーの全出力をインライン展開する。synthesizer は Accepted / Rejected / Backlog のリストを返す。

## 4. 構造による強制のチェック

lint ルール、スクリプト、メタデータのフラグ、ランタイムのチェックのほうが確実に強制できる Accepted の項目は、Backlog へ移す。**encode-lessons-in-structure** の原則 skill を参照。

## 5. 適用する

Accepted の編集を適用する前に、synthesizer の Accepted / Rejected / Backlog の全出力をユーザに提示し、明示的な承認を待つ。ユーザが部分集合を選び、振り分け先を変更してもよい。skill の変更は組織の将来のすべてのエージェントに影響する。自動適用してはならない。

Backlog の各項目は、承認を待たずにチームの devex / backlog トラッカーへ登録する。承認を待つのは Accepted リストだけだ。

承認された各行の Routing に厳密に従う:

- 既存 skill への些細な編集 (1 行の箇条書き、文の引き締め、古い事実の訂正): 親が直接行う。
- 既存 skill への実質的な編集 (新しいセクション、新しいパターン表、およそ 10 行超): Cursor 組み込みの `create-skill` skill に渡し、その draft / test / iterate のループを回す。
- `tune description: <skill path>` (skill は存在するが、発火すべきときに発火しなかった): `create-skill` に渡し、その description 最適化のループを回す。
- `new skill via create-skill: <kebab-name>`: 作成を `create-skill` に渡す。その場しのぎで形を作らない。

環境に SKILL.md のバリデータがあるなら、完了を宣言する前に触れたすべての skill に対して実行する。

## 6. ユーザ向けにまとめる

短いリスト、前置きなし:

- 適用した編集: `<skill path>`。何が変わったかを 1 行ずつ。
- 新規作成した skill: `<skill path>`。1 行ずつ (稀)。
- devex トラッカーへ起票した Backlog: `<issue title>` (`<tags>`)。1 行ずつ。
- 落としたもの: 却下された発見ごとに 1 行、synthesizer による理由付き。
