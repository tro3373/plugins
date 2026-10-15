# インシデントとポストモーテムの文脈

- **Linear**: `incident`、`sev-*`、`postmortem-action-item`、`reliability` のラベルが付いたチケットを探す
- **Slack**: 対象コードが追加された日付の周辺で `#sev-*` と `#incident-*` のチャンネルを検索する
- **Git**: "fix for incident"、"add defensive check"、"revert" の後に "re-apply with..." が続く、といったメッセージのコミットは強い信号だ
