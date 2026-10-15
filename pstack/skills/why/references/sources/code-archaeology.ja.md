# Code Archaeology (git + リポジトリ内)

## このソースに含まれるもの

- コミット履歴 (メッセージ、日付、著者、差分)
- PR の説明、レビューコメント、議論スレッド (`gh` 経由)
- インラインのコードコメント、TODO、FIXME、非推奨の注記
- ADR (アーキテクチャ決定記録)。リポジトリが保持していれば
- テスト。名前とアサーションは、変更の動機となったエッジケースを符号化していることが多い
- 同じコミットで変更された関連ファイル (共変更のシグナル)
- リポジトリ内の CHANGELOG エントリとリリースノート
- コミットメッセージや PR 本文の issue/チケット ID

最も信頼できるソースであり、コードに直接結び付いていて、最も完全である。リポジトリを通ったものはすべてここにあるはずだ。

## 検索の仕方

シードのコミット一覧を広げる。

```bash
# History through renames
git log --follow --oneline -- <file>

# Pickaxe: commits that added or removed this exact text
git log -S '<exact_string_from_code>' -- <file>

# Or for patterns:
git log -G '<regex>' -- <file>

# Who wrote each line and when
git blame -L <start>,<end> <file>

# One commit's full diff
git show <hash>

# Commits between two points affecting this file
git log <old>..<new> -p -- <file>
```

リポジトリ外のドキュメントを探す。

```bash
# ADRs often live in docs/adr/ or similar
rg -l -i 'architecture.decision' --glob '*.md'

# TODOs and FIXMEs near the target
rg -n -C2 '(TODO|FIXME|HACK|XXX|NOTE)' <target_file>

# Related tests
rg -l '<symbol>' --glob '*test*'
```
