# トリアージ automation のプロンプト

> コピーされたセットアップワークフローのための素材。automation が実行されるリポジトリにコピー済みパックがコミットされていることを `automate` が確認した後、この意図を言い換えて組み込みの `automate` ドラフトにする。

この実行では `.cursor/automations/benny/skills/triage-issue-reports/SKILL.md` を読み、それに従うこと。

設定の情報源。このリポジトリ相対のパスは、同じ対象リポジトリにコミットされている場合にのみ含める。そうでなければ、設定された値を言い換える。プラグインのソースパスやキャッシュパスを使ってはならない:

```text
{{BENNY_CONFIG_PATH}}
```

トリガー:

```json
{
	"source_channel_id": "{{SLACK_CHANNEL_ID}}",
	"message_ts": "{{SLACK_MESSAGE_TS}}",
	"thread_ts": "{{SLACK_THREAD_TS_OR_EMPTY}}"
}
```

作成の意図は、これを設定された発生元 Slack チャンネルにおける新しいトップレベルのレポートとして記述すべきである。

発生元チャンネルとルートスレッドのタイムスタンプは不変として扱う。どちらかが欠落している、または設定と一致しない場合は、投稿も issue tracker への書き込みもせずに停止する。

分類、添付のレビュー、原因追跡、ルーティング、重複排除、tracker への書き込み、最終的な判定は、コミットされた運用ファイルが所有する。進捗メッセージを投稿してはならない。発生元チャンネルにルートメッセージを投稿してはならない。

Slack に投稿するのはコーディネータだけである。委譲されたワーカーは read-only でなければならず、findings のみを返し、あらゆる Slack 書き込みアクションの明示的な禁止を受け取らなければならない。

唯一の判定は、設定されたマーカーちょうど 1 つで終えること:

```text
[benny:bug]
[benny:performance]
[benny:other]
```

bug または performance のマーカーには `tracker=<URL>` を付けてよい。
