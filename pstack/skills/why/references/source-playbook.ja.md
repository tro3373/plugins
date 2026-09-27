# ソース別 playbook

why skill は、利用できる証拠のカテゴリごとに調査担当を 1 つ起動し、それぞれが下のソース固有の playbook を 1 つ読む。playbook はよくある MCP に対する具体例だ。同じカテゴリの別の MCP には、読み替えて使う。

| カテゴリ | playbook | それが文書化している MCP の例 |
|---|---|---|
| ソース管理の履歴 | [`code-archaeology.md`](./sources/code-archaeology.md) | git, `gh` |
| 課題 / チケットトラッカー | [`linear.md`](./sources/linear.md) | Linear (Jira、GitHub Issues、Plane、Shortcut には読み替える) |
| 長文ドキュメント | [`notion.md`](./sources/notion.md) | Notion (Confluence、Google Docs、Coda には読み替える) |
| リアルタイムのチームチャット | [`slack.md`](./sources/slack.md) | Slack (Discord、Microsoft Teams、Mattermost には読み替える) |
| インフラの可観測性 | [`datadog.md`](./sources/datadog.md) | Datadog (New Relic、Honeycomb、Grafana、Splunk には読み替える) |
| エラー / 例外のトラッキング | [`sentry.md`](./sources/sentry.md) | Sentry (Rollbar、Bugsnag、Airbrake には読み替える) |
| プロダクト分析のウェアハウス | [`databricks.md`](./sources/databricks.md) | Databricks SQL (Snowflake、BigQuery、ClickHouse、dbt には読み替える) |

横断的なもの:

- [`incident-postmortem.md`](./sources/incident-postmortem.md)。対象コードが防御的に見えるなら (null チェック、リトライ、タイムアウト、レート制限、フィーチャーフラグ、egress のガード、OOM ハンドラ)、これを追加する。
