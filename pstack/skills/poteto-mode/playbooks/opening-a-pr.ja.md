### PR を開く

他のすべてのプレイブックの最後に呼び出される。

**Worktree。** main から切った git worktree で作業する。サブエージェントはそれを継承する。同じブランチに対する複数の `Task` 呼び出しは、それぞれ自分の worktree を持つか、その間に `git fetch && git reset --hard origin/<branch>` を挟む。無関係な作業でブランチが汚れている場合: パッチを取り出し、新しい worktree を作り、適用する。worktree がもつれた場合: main からリセットし、最小限にやり直す。

**コミット。** 惜しみなくコミットする。PR を開く前に、小さく順序立ったコミットへ rebase する。各コミットは将来の PR である。単体で着地でき、物語を語る順序になっていること。修正が直前のコミットに属するなら amend し、分離できるなら新しいコミットにする。

**PR。** コミット前に、diff に対して `cursor-team-kit` の `/deslop` を走らせる。レビュー前に `/no-comments` を走らせる。PR のタイトル、PR の説明、コミット本文はすべて `/technical-writing` で書き、その後 `/unslop` を適用する。Diátaxis を除く technical-writing のすべてのレイヤーを適用する。各アクションに 1 語を使い、冠詞は残し、素の動詞で足りるなら `-ing` を避ける。

**タイトル。** `type(scope): subject` の形で Conventional Commits を使う。type には `feat`、`fix`、`docs`、`refactor`、`test`、`chore`、`perf` を使う。scope には `pstack` や `poteto-mode` のような変更した領域を使う。subject は短く命令形に保つ。変更を担う実在のシンボルがあるなら、それを名指しする。例: `fix(pstack): retarget opening-a-pr babysit trigger`。末尾にピリオドを付けてはならない。

**説明。** PR 本文は簡潔なブリーフィングであって、実験ノートではない。diff を持っているレビュワーは、なぜその変更が存在するのか、何がスコープ外か、変更が動くことをどう証明したかを読み取れればよい。squash コミット本文は PR 本文である。本文が squash コミットをおよそ 40 行より長くしてしまうなら、本文を削る。

次のセクションをこの順で使う。書くことが無いセクションは落とす。

- `## Why`。意図とアプローチを 1、2 段落の短い文で述べる。SHA の列挙や rebase の系譜を書かない。「based on main」のような前置きも書かない。
- `## Scope`。実在のシンボルとパスを箇条書きで挙げる。リネームやターゲット変更は両側を名指しする。境界が重要なときだけ、何が含まれ何が含まれないかを述べる。ファイル単位の長文にしない。
- `## Tradeoffs`。レビュワーがそうでなければ尋ねてくる、却下した代替案だけを名指しする。本物の選択が無かったならこのセクションを省く。
- `## Blast Radius`。1 から 3 文で、変更が誰に何に触れるか、なぜその変更が安全か危険かを名指しする。修正が無いまま main が赤いままなら、その継続コストを述べる。
- `## Verification`。各実在の実行経路とその結果を名指しする。パフォーマンス変更なら、主要な数値 1 つを単位付きで `before → after` の形で報告する。残りの証拠は arena または swarm のディレクトリへリンクする。サンプルサイズの方法論、swarm の詠唱、メトリクステーブルは含めない。

これらのセクションの後に、主張を裏付ける動画やスクリーンショットがあれば添付する。フル SHA、swarm や arena のレーンの詠唱、レバー補正のエッセイ、ファイル単位のチェックリスト、"CLEAN" という判定を貼り付けない。これらの詳細はリンクした成果物に置く。`## Summary` や `## Test plan` のような定型は使わない。コミット本文は subject を繰り返さない。

**Forge。** 最初の PR 操作の前に forge を決め、create、edit、view、watch、merge を通してその選択を保つ。既定は GitHub CLI (`gh`)。`command -v origin` が成功し、Origin がリポジトリを解決できるなら `origin pr ...` を優先する。Origin が無いかリポジトリを解決できないなら `gh` のままにし、フォールバックしたことを記録する。Graphite (`gt`) を必須としてはならない。

**サイズとスタック。** 大きな PR 1 本より、狭い PR 5 本を選ぶ。スタックはベースブランチの連鎖である。ルートの PR は trunk をターゲットにし、各子ブランチは親のちょうど先端に rebase され、その PR は親ブランチをターゲットにする。子は解決した forge に応じて `origin pr create --status open --base <parent-branch>` または `gh pr create --base <parent-branch>` で作る。既存の子のターゲット変更は `origin pr edit <pr> --base <parent-branch>` または `gh pr edit <pr> --base <parent-branch>` で行う。trunk から切るのは独立した作業のときだけである。実質的なスタック作業の前に trunk に rebase する。

**Readiness。** すべての PR は ready で開く。draft では決して開かない。Origin なら `--status open` を渡し、`gh` なら `--draft` を省く。クラウドエージェントの PR ツールは既定が draft なので、すべての PR 作成呼び出しで `draft: false` を設定する。それでも PR が draft で開いたら、解決した forge に応じて `origin pr ready <number>` または `gh pr ready <number>` を走らせる。PR のステータスに言及する前に `origin pr view <number>` または `gh pr view <number>` を走らせる。

**Babysit。** PR を開いても babysit は始まらない。URL を投稿し、作り続ける。まずフェーズまたはスタックを終わらせる。スタック全体が揃った後にユーザが求めたときだけ、別途 babysit のパスを走らせる。新しい PR ごとの babysit はビルドを止め、後の波が再実行するコミットにチェックを浪費する。フィードバックが意図から逸れたら押し返す。

PR を開くサブエージェントは `interrogate`、`/deslop`、`/no-comments` を走らせ、URL を投稿する。それが Autopilot-full または Autopilot-stack のオーナーでない限り、babysit せずに親へ戻る。そのオーナーのブリーフは babysit ループを割り当てるものであり、`playbooks/babysit.md` が待っている ask がそれである。オーナーはコード準備完了の報告の後にループを開始し、そのプレイブックの指示どおりに merge-ready または STACK-READY を報告する。スタック全体が組み上がるまで babysit を保留するこのファイルと `playbooks/babysit.md` の規則は、そのオーナーには適用されない。
