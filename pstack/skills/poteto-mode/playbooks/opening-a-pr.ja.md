### PR を開く

他のすべてのプレイブックの最後に呼び出される。

**Worktree。** main から切った git worktree で作業する。サブエージェントはそれを継承する。同じブランチに対する複数の `Task` 呼び出しは、それぞれ自分の worktree を持つか、その間に `git fetch && git reset --hard origin/<branch>` を挟む。無関係な作業でブランチが汚れている場合: パッチを取り出し、新しい worktree を作り、適用する。worktree がもつれた場合: main からリセットし、最小限にやり直す。

**コミット。** 惜しみなくコミットする。PR を開く前に、小さく順序立ったコミットへ rebase する。各コミットは将来の PR である。単体で着地でき、物語を語る順序になっていること。修正が直前のコミットに属するなら amend し、分離できるなら新しいコミットにする。コミット本文は subject を繰り返さない。

**PR。** コミット前に diff に対して `cursor-team-kit` の `/deslop` を、レビュー前に `/no-comments` を走らせる。PR のタイトル、説明、コミット本文はすべて `/technical-writing` で書き (Diátaxis を除くすべてのレイヤーと、STE の語彙規則を適用する: アクションごとに 1 語、冠詞は残す、素の動詞で足りるなら `-ing` より素の動詞)、その後 `/unslop` を適用する。

**タイトル。** Conventional Commits の `type(scope): subject` で、末尾にピリオドを付けない。type は `feat`、`fix`、`docs`、`refactor`、`test`、`chore`、`perf` など。scope は `pstack` や `poteto-mode` のような領域。subject は短く命令形で、変更を担う実在のシンボルがあればそれを名指しする。例: `fix(pstack): retarget opening-a-pr babysit trigger`。

**説明。** PR 本文は、diff を持っているレビュワー向けの 1 分間のブリーフィングであって、実験ノートではない。なぜその変更が存在するのか、何を含めないのか、何を壊しうるのか、動くことをどう証明したかを述べる。短く平易な文で書き、識別子は少なくし、文字の壁を書かない。本文は squash コミット本文になるので、そのコミットをおよそ 40 行未満に収める。太字の書き出しではなく `##` 見出しを、この順で使う。書くことが無いセクションは落とす:

- `## Why`: 問題とアプローチを 1 から 3 の短い文で述べる。SHA、rebase の系譜、「based on main」のような前置きは書かない。
- `## What changed`: 1 から 3 の短い箇条書きにする。実在のシンボルやパスは、それが変更を担うときだけ名指しする。リネームやターゲット変更は両側を名指しする。
- `## Scope`: 常に書く。何をカバーし、何を意図的に含めないか (フォローアップや既知の穴など) を 1 から 3 の短い項目にする。シンボル、パス、ファイル単位の長文は書かない。
- `## Tradeoffs`: レビュワーがそうでなければ尋ねてくる、却下した代替案だけを書く。本物の選択が無かったなら省く。
- `## Blast Radius`: 変更が誰に何に触れるか、なぜそれが安全か危険かを 1、2 文で述べる。main が赤いなら、赤いまま放置するコストを述べる。
- `## Verification`: 1 から 3 の箇条書きにし、各箇条は実在の実行経路とその結果を書く。パフォーマンス変更なら、主要な数値 1 つを単位付きで `before → after` の形で書き、残りは arena または swarm のディレクトリへリンクする。サンプルサイズの方法論、swarm の詠唱、メトリクステーブルは含めない。

セクションの後に、主張を裏付ける動画やスクリーンショットがあれば添付する。フル SHA、swarm や arena のレーンの詠唱、レバー補正のエッセイ、ファイル単位のチェックリスト、"CLEAN" という判定は、本文ではなくリンクした成果物に置く。

**Forge。** 最初の PR 操作の前に forge を決め、create、edit、view、watch、merge を通してその選択を保つ。既定は GitHub CLI (`gh`)。`command -v origin` が成功し、Origin がリポジトリを解決できるなら `origin pr ...` を優先する。Origin が無いかリポジトリを解決できないなら `gh` のままにし、フォールバックしたことを記録する。Graphite (`gt`) を必須としてはならない。

**組み込みの PR ツール。** 実行環境が組み込みの PR ツールを提供しているなら、作成、編集、ターゲット変更、ready 化はそれを通して行い、forge の CLI は決して使わない。CLI で作った PR は、そのツールが追跡するものを取りこぼす。ツールがカバーしないことすべてと、そのようなツールが無い実行でのすべての PR 操作には、解決した forge を使う。

**サイズとスタック。** 大きな PR 1 本より、狭い PR 5 本を選ぶ。スタックはベースブランチの連鎖である。ルートの PR は trunk をターゲットにし、各子ブランチは親のちょうど先端に rebase され、その PR は親ブランチをターゲットにする。親がマージされていくにつれ、その子を `playbooks/shipping.md` の手順 4 に従って trunk へ移す。組み込みの PR ツールが無いなら、解決した forge に応じて、子は `origin pr create --status open --base <parent-branch>` または `gh pr create --base <parent-branch>` で作り、ターゲット変更は `origin pr edit <pr> --base <parent-branch>` または `gh pr edit <pr> --base <parent-branch>` で行う。trunk から切るのは独立した作業のときだけである。実質的なスタック作業の前に trunk に rebase する。

**Readiness。** すべての PR は ready で開く。draft では決して開かない。組み込みの PR ツールは既定が draft のことがあるので、それを通すすべての作成呼び出しで `draft: false` を設定する。Origin なら `--status open` を渡し、`gh` なら `--draft` を省く。それでも PR が draft で開いたら、PR ツールで ready にするか、解決した forge に応じて `origin pr ready <number>` または `gh pr ready <number>` を走らせる。PR のステータスに言及する前に `origin pr view <number>` または `gh pr view <number>` を走らせる。

**Babysit。** PR を開いても babysit は始まらない。URL を投稿し、作り続ける。まずフェーズまたはスタックを終わらせる。スタック全体が揃った後にユーザが求めたときだけ、別途 babysit のパスを走らせる。新しい PR ごとの babysit はビルドを止め、後の波が再実行するコミットにチェックを浪費する。フィードバックが意図から逸れたら押し返す。

PR を開くサブエージェントは `interrogate`、`/deslop`、`/no-comments` を走らせ、URL を投稿する。Autopilot-full または Autopilot-stack のオーナーでない限り、babysit せずに親へ戻る。そのオーナーのブリーフは babysit ループを割り当てるものであり、`playbooks/babysit.md` が待っている ask がそれである。オーナーはコード準備完了の報告の後にループを開始し、そのプレイブックに従って merge-ready または STACK-READY を報告する。スタック全体が組み上がるまで babysit を保留するこのファイルと `playbooks/babysit.md` の規則は、そのオーナーには適用されない。
