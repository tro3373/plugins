# Reproduce automation prompt

> コピーされたセットアップワークフローの元となる素材。オートメーションが動くリポジトリにコピーされたパックがコミットされていることを `automate` が確認した後、この意図を言い換えて組み込みの `automate` ドラフトにする。

この実行では `.cursor/automations/benny/skills/reproduce-and-fix-issues/SKILL.md` を読んで従う。

設定の出所。このリポジトリ相対パスを含めるのは、同じ対象リポジトリにコミットされている場合だけである。そうでなければ設定値を言い換える。プラグインのソースやキャッシュのパスを使ってはならない:

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

作成の意図は、これを設定された Slack のソースチャンネルにおける新しいトップレベルの報告として記述すべきである。設定されたリポジトリ、既定ブランチ、issue トラッカー、control アダプタ、フィーチャーマップ、draft プルリクエストの能力を含めるべきである。

ソースチャンネルとルートスレッドのタイムスタンプは不変として扱う。どちらかが欠けているか設定と一致しないなら、投稿せずに停止する。

まさにこのスレッドで、設定された triage の identity からの、設定された triage のマーカーを待つ。`[benny:bug]` または `[benny:performance]` の場合にのみ進む。

再現を試みる前に、設定された control アダプタの skill を必須とする。決め手となる症状そのものを、本物の UI を通して 2 回再現する。既存のプルリクエストやコミットは、その上に書き加えることなく検証する。確認済みの再現と、運用ファイルの fix ゲートを通過した後に限り、範囲を限った修正を試みる。

Slack に投稿するのはコーディネータだけである。すべての子プロンプトは `SendSlackMessage`、`PostToSlack`、`chat.postMessage`、およびその他すべての Slack への書き込みを禁じなければならない。子は発見のみを返す。

ソースチャンネルにルートメッセージを投稿してはならない。
