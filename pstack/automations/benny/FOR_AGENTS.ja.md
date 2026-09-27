# benny automation の意図

## 自動化したいこと

ひとつの slack issue channel で連携して動く cursor automation を 2 つ欲しい。

### automation 1: issue レポートの triage

- trigger: 設定した source slack channel に誰かが新しいトップレベルのレポートを投稿したら、この automation がそのレポート上で開始し、元のスレッド座標を保持してほしい。
- behavior: スレッドと添付を読み、レポートをバグまたはパフォーマンス問題、機能要望、質問やフィードバック、あるいは再ルーティングに分類し、ルーティングの前に担当らしきレイヤーを辿ってほしい。
- tracker: 設定した tracker で重複を検索し、確信のある重複は更新し、明確に新規のバグのときだけチケットを作ってほしい。
- tools: slack スレッドの読み取りと返信のアクセス、設定した tracker との連携、任意のルーティングマップが欲しい。
- outcome: source スレッドへの返信をちょうど 1 件、短い verdict と `[benny:bug]`、`[benny:performance]`、`[benny:other]` のいずれかを含む形で欲しい。bug または performance のマーカーには tracker url を含めてよい。
- boundary: この automation が source channel にルートメッセージを投稿することは決して望まない。

### automation 2: 確認済みバグの再現と修正

- trigger: この automation は同じ新しいトップレベルのレポートから、あるいはセットアップ時に選んだ別の対応トリガーから開始し、元のスレッドで信頼できる triage マーカーを待ってほしい。
- gates: 誰かが明確に修正を引き受けているときは止まってほしい。既存の pull request や merge 済みの commit がそのレポートを修正しうるなら、競合する変更ではなく検証をしてほしい。
- behavior: 設定した control adapter と feature map を使い、実際の ui を通して厳密な症状を 2 回再現し、スクリーンショット、動画、読み取り専用の状態のクロスチェックを取ってほしい。
- fix: 既存の pull request の上に書き足すことなく、それを検証してほしい。再現が確認できた後は、根本原因に対する範囲を限った修正を 1 回試みてよく、テストが安価なら tdd を使い、影響範囲をスモークし、before-and-after の証明が通ったときにだけ draft の pull request を開いてよい。
- tools: slack スレッドの読み取りと返信のアクセス、リポジトリと履歴へのアクセス、draft pull request の作成、設定した tracker、そして control adapter が欲しい。
- outcome: source または任意の operations スレッドに証拠と検証済みの結果、加えて任意で draft pull request が欲しい。更新は簡潔にしてほしい。
- boundary: この automation が source channel にルートメッセージを投稿することは決して望まない。

### 共通ルール

- source channel と root スレッドの座標は、実行の全体を通じて不変であってほしい。
- ユーティリティ bot やデバッグ bot は証拠として扱い、委任や修正の所有権とは見なさない。
- サブエージェントの助けは許すが、slack に投稿することも slack の資格情報を受け取ることもできない。
- このパック全体を対象リポジトリの `.cursor/automations/benny/` にコミットしてほしい。その `SKILL.md` は automation への直接の指示であり、登録されたプラグイン skill ではない。
- pstack は、対象リポジトリにコミットされた `.cursor/settings.json` を通じてのみ、`how`、`why`、`tdd`、`unslop` および必要な principle skill といった共有依存のために有効化してほしい。
- 稼働中の各 automation プロンプトは、コミットされた運用ファイルを直接読んでほしい。プラグインのキャッシュパスも、抜粋のコピーも、slash-skill による発見も望まない。
- ユーザ所有の設定、feature map、ルーティングマップ、シークレットは `.cursor/automations/benny/` の外に置き、パックの更新で上書きされないようにする。
- channel の座標、tracker へのアクセス、control adapter、feature map が欠けているか不確かなときは、どちらの automation も fail closed してほしい。
- pull request は draft のみが欲しい。merge も deploy もしないこと。

### 私の設定

- source slack channel: `<channel>`
- optional operations channel: `<channel or none>`
- repository and default branch: `<repo>`, `<branch>`
- tracker: `<type, team, project, labels, intake status>`
- routing map: `<path or none>`
- triage identity: `<slack identity>`
- control skill: `<configured skill or adapter>`
- feature map: `<committed same-repo path outside the copied pack, or behavior to paraphrase>`
- models: `<triage, reproduce, code, media review>`
- status emoji strings: `<seen, reproducing, reproduced, blocked, fixing, failed, pull request opened>`
- budgets: `<polling, verdict wait, follow-up, repro, rejection, fix>`
- optional bot token capability: `<none, file download, or editable operations status>`

[`configuration.example.yaml`](./templates/configuration.example.yaml) と [`feature-map.example.md`](./skills/reproduce-and-fix-issues/references/feature-map.example.md) から始める。このパックの外、例えば `.cursor/benny/` の下にコピーして埋める。シークレットの値はシークレットマネージャか環境変数に置く。

## エージェント向け

人間はこのファイルに cursor を向けることでセットアップに入る。発見された benny の slash skill を探したり呼び出したりしないこと。

1. どのリポジトリで automation を走らせるのかを尋ねる。
2. この `FOR_AGENTS.md` を含むディレクトリをソースパックとして扱う。
3. ソースパック全体を `<target-repository>/.cursor/automations/benny/` にマージする。
4. destination 側だけに存在するファイルをすべて保全する。無関係なファイルを削除したり、ユーザ所有の設定、feature map、ルーティングマップを上書きしたりしない。
5. ソース管理下のパスにある既存の destination ファイルが差分を持つ場合は、その diff をレビューし、ローカルの編集を捨てずにマージする。所有権が曖昧なら、置き換える前に止まって尋ねる。
6. コピーした `FOR_AGENTS.md` と `skills/setup-benny/SKILL.md` が対象リポジトリに存在することを検証する。
7. 対象リポジトリから `.cursor/automations/benny/skills/setup-benny/SKILL.md` を直接読み、それに従う。

対象リポジトリの `.cursor/settings.json` に、次のエントリをマージしてほしい:

```json
{
	"plugins": {
		"pstack": { "enabled": true }
	}
}
```

無関係な設定とプラグインはすべて保全する。既存のファイルが jsonc なら、コメントと妥当な jsonc の構文を保つ。

対象リポジトリを起点とする新しいエージェントによる検証が欲しい。pstack の `how`、`why`、`tdd`、`unslop`、および benny が使う principle skill が project スコープで解決することを確認する。現在のセッションで読み込まれた skill や、user スコープのインストールから来たものを数に入れないこと。

project スコープのプラグインが利用できない場合、または共有依存のいずれかが解決しない場合は、止まって何が失敗したかを説明する。`.cursor/automations/benny/skills/` をプラグインのマニフェストに追加したり、そのファイルが slash-skill のリストに現れることを期待したりしない。

`.cursor/settings.json`、`.cursor/automations/benny/`、そして参照されるシークレットを含まない設定を、どちらの automation を有効化するよりも前にコミットしなければならないことを私に伝える。私が明示的に頼むまで automation を作成も更新もしない。

初回の作成では、組み込みの `/automate` を triage 用に 1 回、repro and fix 用に 1 回使う。2 つ目に取りかかる前に、1 つ目の automation の draft レビュー、承認、readiness チェック、Automations エディタへの引き継ぎを完了させる。

この意図と完成した設定を、各 draft に言い換えて入れる。triage のプロンプトは `.cursor/automations/benny/skills/triage-issue-reports/SKILL.md` を読んで従わなければならない。repro のプロンプトは `.cursor/automations/benny/skills/reproduce-and-fix-issues/SKILL.md` を読んで従わなければならない。これらの repo 相対パスは、automation を走らせるリポジトリにコミット済みだと `/automate` が確認した後にのみ使う。

既存の automation については、`/automate` を使って調べたり更新したりしない。設定を検証したうえで、コピーされたセットアップファイルにある簡潔なフィールドのチェックリストを使い、私が各 automation をそのエディタで直接編集できるようにする。重複を作らないこと。
