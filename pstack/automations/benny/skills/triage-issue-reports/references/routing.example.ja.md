# ルーティングマップの例

このファイルを `.cursor/automations/benny/` の外へコピーし (例えば `.cursor/benny/routing.md`)、すべてのプレースホルダを置き換える。`routing.map_path` をそのコピーへ向ける。パックの更新がそれを上書きしてはならない。

triage skill はこれをデータとして扱う。ルートには、レポートまたは原因のトレースからの証拠が必要である。キーワードの一致だけでは足りない。

```yaml
routes:
  - name: "billing-example"
    match:
      product_areas:
        - "billing-area-placeholder"
      code_paths:
        - "billing-code-path-placeholder"
      error_signatures:
        - "billing-error-placeholder"
    destination:
      slack_channel: "billing-channel-placeholder"
      tracker_team: "billing-team-placeholder"
    owners:
      - "billing-owner-placeholder"
    allow_feature_owner_ping: false

  - name: "desktop-example"
    match:
      product_areas:
        - "desktop-area-placeholder"
      code_paths:
        - "desktop-code-path-placeholder"
      error_signatures:
        - "desktop-error-placeholder"
    destination:
      slack_channel: "desktop-channel-placeholder"
      tracker_team: "desktop-team-placeholder"
    owners:
      - "desktop-owner-placeholder"
    allow_feature_owner_ping: false

fallback:
  destination: ""
  owners: []
  allow_feature_owner_ping: false

ping_policy:
  default: "off"
  allow:
    - "configured-feature-owner"
    - "confirmed-regression-author"
  deny:
    - "broad-on-call-group"
    - "unverified-owner"
```

## ルール

- 1 つのチームがマッチしないレポートをすべて引き受けるのでない限り、`fallback.destination` は空のままにする。
- 安定したプロダクト領域、コードパス、エラーシグネチャを使う。
- 公開されるコピーにプライベートなデータを含めてはならない。
- 公開される例に、生のユーザ ID やチャンネル ID を貼ってはならない。
- 対象チームが同意するまで、フィーチャーオーナーへの ping はオフのままにする。
- リルートは、報告者にどこへ行けばよいかを伝えるものである。この自動化はクロスポストを決してしない。
