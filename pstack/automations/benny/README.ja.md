# benny

benny は slack の issue レポート向けに 2 つの cursor automation を提供する。ひとつは各レポートを triage する。もうひとつは確認済みのバグを再現し、小さな draft の修正を用意することがある。

このディレクトリのファイルは休眠状態のセットアップ用・automation 用ソースである。slash skill としては現れない。

## セットアップ

1. cursor に [`FOR_AGENTS.md`](./FOR_AGENTS.ja.md) を指し示し、対象リポジトリを伝える。
2. セットアップにこのディレクトリ全体を対象リポジトリの `.cursor/automations/benny/` へマージさせる。マージは destination 側だけに存在するファイルを保全し、ローカルの編集を上書きせず衝突をレビューしなければならない。
3. セットアップに、共有依存のため対象リポジトリの `.cursor/settings.json` で pstack を有効化させる:

```json
{
	"plugins": {
		"pstack": { "enabled": true }
	}
}
```

4. ユーザ所有の設定はコピーされたパックの外、例えば `.cursor/benny/` に置く。[`configuration.example.yaml`](./templates/configuration.example.yaml) と [`feature-map.example.md`](./skills/reproduce-and-fix-issues/references/feature-map.example.md) を元に書き換える。
5. どちらの automation を有効化するより前に、`.cursor/settings.json`、`.cursor/automations/benny/`、および秘密情報を含まない設定をコミットする。
6. 新しい automation の draft をレビューするか、既存の automation をそのエディタで更新する。その後、無害なテストレポートを送り、source channel への投稿がすべて元のスレッド内に留まることを検証する。
