# Bugbot triage

Babysit プレイブック (`../playbooks/babysit.md`) が Bugbot やレビュー自動化のコメントを扱うときに、このリファレンスを使う。目的は Bugbot を既定で無視することではない。すべてのコメントを必須のコード変更として扱うのをやめることだ。

## 判断ルーブリック

行動する前に、Bugbot の各スレッドを分類する。

- `fix`: コメントが、もっともらしい正しさ、セキュリティ、プライバシー、データ損失、認証、課金、マイグレーション、冪等性、レース、または出荷済みの挙動に関する問題を指摘している。それを所有する最下位の PR で修正し、コミット SHA を添えて返信し、スレッドを解決する。
- `dismiss`: コメントが、文書化された低リスクのノイズパターンに合致し、かつ現在のコード/文脈がその懸念にコード変更を要さないことを証明している。短い理由を返信し、スレッドを解決する。
- `ask`: コメントが新種、高深刻度、セキュリティ/プライバシー/データ関連、または曖昧である。推測せずユーザに聞く。

迷ったら聞く。ノイズなコード品質コメントを飛ばすのは安いが、本物のデータやセキュリティのバグを飛ばすのは安くない。

## 学習パターンの書式

今後のパターンはこの形で追加する。

```markdown
### <short pattern name>

- Confidence: candidate | recurring | strong
- Skip when: <conditions that must be true>
- Do not skip when: <risk boundaries>
- Example signal: <phrases or code context that identify the pattern>
- Source: <PR/comment URL or short historical note>
```

例が 1 件か 2 件なら `candidate` を使う。実際の dismiss が複数回あった後に `recurring` を使う。パターンが狭く、繰り返し検証され、低リスクなときにのみ `strong` を使う。

## 繰り返し現れる skip 候補

### 意図的な UI またはデザインシステムの見た目の変更

- Confidence: candidate
- Skip when: PR の説明、スクリーンショット、デザインレビュー、または近傍のコードが、その見た目の変更を明示しており、Bugbot のコメントが共有の視覚的デフォルトが変わったことを言い直しているだけのとき。
- Do not skip when: コメントが、アクセシビリティ、フォーカスの視認性、キーボードナビゲーション、色のコントラスト、または PR が意図的に変えていないコンポーネントの API 契約を指しているとき。
- Example signal: フォーカスのアウトライン、ボタンのサイズ、余白、共有コンポーネントの視覚的デフォルトに関するコメントで、オーナーが「intentional」「intended」と返信しているもの。

### Bugbot からは見えない upstack またはスタック内での利用

- Confidence: candidate
- Skip when: Bugbot が export、コンポーネント、ヘルパー、またはファイルを未使用としてフラグしたが、稼働中の forge の PR 一覧と差分、上位スタックの差分、または PR の文脈が、スタック内の後続 PR で使われていることを示しているとき。
- Do not skip when: 現在の PR がスタックの一部でない、シンボルが公開 API である、または upstack での利用とされるものを検証できないとき。
- Example signal: 「Exported component is never used」に対して人間が「used upstack」と返信しているもの。

### 並行実装中の一時的な重複

- Confidence: candidate
- Skip when: 削除中・置き換え中・検証中の旧パスと新パスを並行させておくために、PR が意図的に少量のコードを重複させているとき。
- Do not skip when: 重複したコードがセキュリティ、課金、データアクセス、API の挙動を変える、または長期的に共有される抽象化が明らかにリスクを下げるとき。
- Example signal: 「Significant duplication」や「duplicated validation logic」に対し、オーナーが旧パスは削除される、または重複ロジックは意図的にローカルだと説明しているもの。

### 既存のフレームワークまたはコンポーネントの不変条件が警告をカバーしている

- Confidence: candidate
- Skip when: その懸念が、現在の差分または近傍のコードから見える共有コンポーネント、フレームワークの契約、型の不変条件、または単一の情報源によってすでに保証されているとき。
- Do not skip when: 不変条件が仮定されているだけで強制されていない、タイミングに依存する、または値が乖離しうる async/state の境界をまたぐとき。
- Example signal: 共有 popover がビューポート境界を強制しているのに内側の popover に max-height が無いというコメントや、ローカルでチェックした値と渡す値が同じ情報源を共有しているのに nullable だとするコメント。

### オーナーが宣言したフォローアップまたは先送りのクリーンアップ

- Confidence: candidate
- Skip when: PR のオーナーが既知のフォローアップだと明言しており、現在の PR で挙動が悪化せず、コメントが高リスク領域についてのものでないとき。
- Do not skip when: エージェントがオーナーの入力なしに動いている、問題が中/高深刻度のプロダクト挙動である、または先送りすると新しいリグレッションをマージすることになるとき。
- Example signal: 「I'll worry about that later」や「we'll delete this eventually」。

### 自己撤回、または明示的に false positive とされたルールコメント

- Confidence: recurring
- Skip when: コメント本文または後続の Bugbot の返信が、その指摘は撤回された、準拠している、または false positive だと明言しており、エージェントが該当ルールをローカルで検証できるとき。
- Do not skip when: 唯一の根拠が、高リスクな問題に対して人間が説明抜きで「false positive」と言っているだけのとき。
- Example signal: ファイル命名ルールのコメントで、本文にそのファイルはすでに準拠していると書かれているもの。

## 既定で聞くもの

以前の PR が似たものを dismiss していたとしても、次のカテゴリを自動 skip してはならない。

- セキュリティ、プライバシー、認証、課金、データ保持、学習データ、権限境界に関する指摘。
- 高深刻度の指摘。
- マイグレーション、スキーマ、冪等性、並行性、システム横断の挙動に関する指摘。
- 提案された修正が小さく、プロダクトの意図を変えずに明らかにリスクを下げるコメント。

過去のデータでは、人間がセキュリティ/データフローのコメントを dismiss することがあった。それらはオーナーの判断であって、チーム全体の skip ルールではないと扱う。

## 最近の babysit からの候補学習

チームに有用そうだがまだ成熟していない学習は、babysit 中または後にここへ追記する。複数の PR がパターンを裏付けたら、繰り返し現れる候補を上のセクションへ昇格させるのが望ましい。

### ネイティブなブラウザ挙動を手作業で再実装したもの

- Confidence: candidate
- Skip when: 実質的に決してない。差分がネイティブなブラウザ挙動を手製の等価物へ置き換えているとき (ネイティブ sticky => JS で配置したクローン、ネイティブのスクロール対象指定 => wheel/touch イベントの転送、描画順によるオクルージョン => マスク/clip-path)、そのコードに対する Bugbot のロジックバグ指摘は一貫して正当だった。
- Do not skip when: 指摘が、イベント転送の抜け (wheel の deltaMode、タッチのパン、端でのスクロールチェイン、タップの許容ずれ)、マスク/clip とヒットテストの乖離、またはそうしたコードにおける observer と React state のタイミングレースに関するとき。既定で fix に倒す。
- Example signal: 「masks do not affect hit-testing」「overlay blocks wheel scroll」「ignores deltaMode」「runs in the IntersectionObserver callback before React applies state」。
- Source: sticky オクルージョンの PR 1 本。Bugbot が 6 パス、指摘はおよそ 18 件、そのすべてが dismiss ではなく修正された。

### 契約テストのずれという主張は安く検証できる - まずテストを走らせる

- Confidence: candidate
- Skip when: 検証そのものは決して skip しない。コマンド 1 本で済む。PR が
  プロトコルやドキュメントの文面を固定する契約テスト (SKILL.md に対する正規表現、
  文面のスナップショット) を出しており、Bugbot が「テストがもうドキュメントと
  合っていない」(またはその逆) と主張したら、分類する前に PR の tip でその
  テストを走らせる。赤なら主張が経験的に裏付けられ、緑なら dismiss の返信に
  使える具体的な反証になる。
- Do not skip when: 該当なし - これは dismiss のパターンではなく検証の近道である。
  なお、パスを重ねるほど dismiss に寄せるヒューリスティックはここで誤作動する。
  文面を固定するテストは、まさに以前の修正ラウンドが文面を編集するからこそずれる。
- Example signal: 以前の修正コミットが固定対象の文面を書き換えた PR に対する
  「Contract test omits the pre-fix wait」。tip で走らせたテストは、引用された
  まさにそのアサーションで失敗した。
- Source: 文面固定の PR 1 本、Bugbot は 8 パス。以前のパスがすべて修正・解決済み
  だったにもかかわらず、パス 7 の主張は本物だった。

### 同じ PR の後段ですでに修正済みの、古いセキュリティレビュー指摘

- Confidence: candidate
- Skip when: エージェント型のセキュリティレビュー (または類似) が認可/バリデーションの呼び出しが欠けていると主張したが、現在の PR の tip にそのゲートが明確に含まれている (テスト付き) とき。典型的にはレビュー実行後の hardening コミットで追加されている。
- Do not skip when: 引用されたヘルパーが議論対象の principal に対して no-op である、チェックがそれが守る副作用の後に走る、または主張された principal のカバレッジが欠けているとき。
- Example signal: tip では副作用の前にまさにそのガードが呼ばれているのに、HIGH の「missing authorization check」が指摘されているもの。
- Source: hardening コミットがレビュー実行より後だった webhook エンドポイントの PR 1 本。

### 意図的に狭くしたエラー条件を広げると本当のエラーを隠す

- Confidence: candidate
- Skip when: 指摘が、狭いエラー条件 (特定の `errno`、エラーコード、ステータス
  クラス) を包括的なものへ広げるよう求めており、その狭さが実際の区別を
  符号化しているとき。典型形は `ENOENT` を条件とする依存フォールバックだ。
  「バイナリがインストールされていない」は「コマンドが走って失敗した」とは
  別の状況である。ゼロ以外の終了すべてで再試行すると、正当な失敗 (見つからない、
  認証切れ、ネットワーク) をフォールバックに対して再実行し、フォールバックの
  エラーを報告して本当のエラーを隠すことになる。
- Do not skip when: 狭い条件が同じカテゴリのケースを取りこぼしている
  (`EACCES` のような別の「バイナリが使えない」errno、別のトランスポート層の失敗)、
  未処理のパスがデータを失うか部分的な状態を残す、または再試行が冪等で
  かつ元のエラーが依然として表に出るとき。
- Example signal: 「only retries when X fails with ENOENT … never tries the
  fallback even when a working Y exists」が、失敗した操作ではなく欠けた依存の
  ために存在するフォールバックのコードを指しているもの。
- Source: フォールバックが失敗したコマンドではなく欠けたバイナリのために存在
  していた CLI リネームの PR 1 本。
