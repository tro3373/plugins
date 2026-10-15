# Sentry のエラー履歴

## 検索の仕方

Sentry MCP を使う。

1. **方向を定める。** project slug と organization が分からない場合:

   ```
   find_organizations
   find_projects
   ```

2. **対象に関連する issue を検索する。**

   ```
   search_issues (natural language, e.g., "errors in PaymentService timeout", "unhandled exceptions in uploadFile")
   ```

   クエリは、対象が処理している例外クラス名、対象の関数名やクラス名、対象がチェックしているエラーメッセージ文字列、対象のファイルパスから組み立てる。

3. **リリースと時間ウィンドウで絞る。**

   ```
   search_issue_events (filter by release, time, environment, trace ID, tags)
   get_issue_tag_values (for an issue, see distribution across versions, users, environments)
   ```

   疑わしい issue については、**初回検知**と**最終検知** (止まった時期が対象の出荷日と一致するか?)、**影響を受けたリリース** (どのバージョンで発生し、どれが修正か?)、**頻度の推移** (スパイクした後、解消したか?) を確認する。

4. **文脈のためにイベント全体を取得する。**

   ```
   get_sentry_resource (pass a Sentry URL or type+ID)
   ```

   スタックトレースは対象のコードを通っているか? タグと breadcrumb は、対象が防御している条件と一致するか?

5. **対象の近くに着地したリリースを確認する。**

   ```
   find_releases (around the commit date of the target)
   ```

   リリースバージョンと PR のマージ日を突き合わせる。

6. **Seer は控えめに使う。**

   ```
   analyze_issue_with_seer
   ```

   Seer の AI による根本原因分析は仮説生成器だ。推論として扱い、権威あるものとしては扱わない。実際のイベントとスタックトレースが一次証拠である。

## 何を返すか

関連する issue ごとに:
- Issue ID とタイトル
- Project と organization
- 初回検知 / 最終検知のタイムスタンプ
- イベント数 (分かればサンプリング率も)
- 影響を受けたリリース
- 対象との関連を示す代表的なスタックトレースの抜粋 (要約ではなく逐語の抜粋)
- 対象の出荷日との初回/最終検知の相関
- issue へのリンク
- 作者のコメントや解決ノート
