# Code Archaeology (git + リポジトリ内)

## このソースに含まれるもの

- コミット履歴 (メッセージ、日付、著者、差分)
- PR の説明、レビューコメント、議論スレッド (`gh` 経由)
- インラインのコードコメント、TODO、FIXME、非推奨の注記
- ADR (アーキテクチャ決定記録)。リポジトリが保持していれば
- テスト。名前とアサーションは、変更の動機となったエッジケースを符号化していることが多い
- 同じコミットで変更された関連ファイル (共変更のシグナル)
- リポジトリ内の CHANGELOG エントリ、リリースノート
- コミットメッセージや PR 本文で言及される issue/チケット ID

最も信頼できるソースであり、コードに直接結び付いていて、最も完全である。リポジトリを通ったものはすべてここにあるはずだ。

## 検索の仕方

シードのコミット一覧を広げる。

```bash
# Full history of the file through renames
git log --follow --oneline -- <file>

# Pickaxe: commits that added or removed this exact text
git log -S '<exact_string_from_code>' -- <file>

# Or for patterns:
git log -G '<regex>' -- <file>

# Who wrote each line and when
git blame -L <start>,<end> <file>

# The full diff of a specific commit
git show <hash>

# Commits between two points affecting this file
git log <old>..<new> -p -- <file>
```

実質的なコミットごとに、PR の文脈を引く。

```bash
# Find the PR number from the merge commit or branch
git log -1 --format=%B <hash>

# Full PR context: body, review comments, linked issues
gh pr view <number> --json title,body,author,createdAt,mergedAt,labels,closingIssuesReferences,comments,reviews,files

# The --json reviews and comments fields are where the real signal is
```

リポジトリ外のドキュメントを探す。

```bash
# ADRs often live in docs/adr/ or similar
rg -l -i 'architecture.decision' --glob '*.md'

# TODOs and FIXMEs near the target
rg -n -C2 '(TODO|FIXME|HACK|XXX|NOTE)' <target_file>

# Related tests. Names often encode the "why"
rg -l '<symbol>' --glob '*test*'
```

## ここでの良い証拠とは

- 変更だけでなく、解こうとしていた問題を説明している PR の説明 (「これは X を引き起こしていたページネーションのバグを修正する」)
- 代替案が議論された長いレビュースレッド
- 対象行の近くにある、自明でない制約を説明したインラインコメント
- `test_handles_edge_case_when_X` のような、コードの動機となったエッジケースを明かすテスト名
- チケットやインシデント ID を参照しているコミットメッセージ
- ユーザから見える設計判断の根拠をまとめている CHANGELOG エントリ

## よくある落とし穴

- **Squash マージによる平坦化。** リポジトリが PR を squash していると、ブランチ履歴中の個々のコミットは失われる。PR 本文とコメントにフォールバックする。
- **誤解を招くコミットメッセージ。** 「Small refactor」が意図的な挙動変更を隠していることがある。メッセージではなく差分を見る。
- **カーゴカルト的なパターン。** 著者が理由を理解しないままパターンをコピーした可能性がある。そのパターンがコードベースのより早い時点で生まれていないか確認し、*その*コミットを調べる。
- **ボットのコミットと自動マージ。** Dependabot、Renovate、自動バックポートは通常、動機を伴わない。意図を探すときは飛ばす。
- **コードを意図の証拠として扱うこと。** コード自体は、それがなぜ存在するかの証拠にはならない。証拠はコミットメッセージ、PR、コメント、テスト、ドキュメントから来る。「関数の名前が X である」を意図の証拠として引用してはならない。

## 何を返すか

問いに関わるすべてのコミット/PR/コメントを、次とともに返す。
- 正確なテキスト (引用)
- ハッシュ / PR 番号 / file:line
- 著者と日付
- 直接的か (問いに明示的に答えているか)、状況証拠か
