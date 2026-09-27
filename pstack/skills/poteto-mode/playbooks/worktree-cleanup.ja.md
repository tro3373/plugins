### Worktree と simulator のクリーンアップ

**ディスクと安全ゲートはあなたの担当だ。** merge 済みまたは放棄された git worktree と、古い iOS simulator を刈り取って容量を取り戻す。削除は不可逆なので、使用中のものや未コミットの作業を抱えたものを消さないよう、各ステップでガードする。

1. スナップショットと監査。まず `df -h /` を記録し、次に `scripts/worktree-audit.sh` を実行する (principle-build-the-lever)。このスクリプトはパスを手打ちではなく `git worktree list` から読む。手打ちの `myrepo-worktrees/x` では `.cursor/worktrees/myrepo/x` にあるものを取りこぼすからだ (principle-encode-lessons-in-structure)。各 worktree をサイズ、経過時間、merge 状態、未コミットの作業、PR の状態、それに触れた最新のチャットで分類し、バケットを提案する。transcript のスキャンは遅いので、バックグラウンドで走らせる。
2. バケットは助言であって許可ではない。ピン留めされたチャットとアクティブなチャットこそが本物の材料だ (principle-prove-it-works)。その集合をユーザかサイドバーから取得し、候補すべてと突き合わせる。この lever は、ユーザがピン留めしていた worktree を `safe` と印を付けたことがある。ピン留めされた集合が勝つ。
3. 削除前に使用状況を検証する。`verify-recent-chat` の行すべて、および疑わしいものすべてについて、サブエージェントを展開して transcript を読ませ、そのチャットがピン留めか進行中か、どの worktree に触れているかを報告させる (principle-guard-the-context-window。transcript は嵩張る)。ピン留めされたチャットは、バックグラウンドのサブエージェント経由で兄弟 worktree に arena や repro のツリーを生やす。それらは名前がサイドバーに出てこなくても使用中だ。
4. 不可逆な損失の前では止まる。`wip:N` は追跡対象の未コミット編集が N 件あるということだ。まず diff を見せて判断を仰ぐ。clean な worktree を消してもブランチから復元できるが、未コミットの作業は失われるからだ。`scratch:N` は untracked の使い捨てで、消して問題ないが、ファイル名は挙げる。Autonomy に従い、clean かつ merge 済みかつ未使用なら進める。`wip` と使用中は止まる。
5. 確認済みの集合を刈る。パスごとに `git worktree remove --force <path>` を実行する。無視対象のビルド成果物でディレクトリが残る場合は `rm -rf` してから `git worktree prune` する。ブランチの ref は残るので、コミットは失われない。`df -h /` と再リストで確認する。
6. Simulator とその他の回収先。次に大きい回収先はたいてい simulator だ。`xcrun simctl --set testing delete all` (XCTestDevices のクローン)、`xcrun simctl delete unavailable`、古いランタイムには `xcrun simctl runtime list` の後に `runtime delete <id>`。必要ならさらに: Xcode の `DerivedData` と `iOS DeviceSupport`、`~/Library/Application Support/Cursor` (`state.vscdb.backup`、およびワークスペースとして開いたフォルダ名を持つ `<root>` が肥大する `snapshots/roots/<root>`)、パッケージキャッシュ (pnpm, uv, brew, yarn)。ユーザが残すと言っていないキャッシュだけを消す。

これは、ミスを捕まえるコードレビューの無いまま、ユーザの状態を削除する唯一の playbook だ。上のゲートがそのレビューの代わりである。

**返答:** 前後の `df -h /` と回収した容量、刈った worktree、保留したものそれぞれの 1 行の理由 (どのチャットが使用中か、または未コミットの作業があるか)。
