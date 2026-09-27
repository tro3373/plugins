# フィーチャーマップの例

Benny が再現しうるユーザ向け機能をすべてマップする。アプリを駆動する前に、該当するセクションを読むこと。このマップはユーザ視点に保つ。内部構造や現在のコードパスは、ここに凍結するのではなく実行時に発見する。

このファイルは `.cursor/automations/benny/` の外へコピーし (例: `.cursor/benny/feature-map.md`)、`control.feature_map_path` をそのコピーに設定する。パックのリフレッシュがそれを上書きしてはならない。

## 機能ごとのテンプレート

### `<feature name>`

`<one-line user-visible purpose>`

#### ユーザがそこへ到達する方法

- クリック経路: `<screen> -> <menu, tab, or panel> -> <control>`
- キーボードショートカット: `<shortcut or none>`

#### control アダプタによる駆動方法

- `<inputs>` を伴う `<adapter action>` は `<visible result>` となるべきである。
- リセット: `<how the adapter returns to a fresh state>`。

#### 安定したセレクタ

- `<role and accessible name>`
- `<ARIA relationship>`
- `<data-component or purpose-named data attribute>`

生成された CSS や StyleX のクラス、動的なハッシュ、子要素のインデックス、壊れやすい DOM 上の位置を使ってはならない。

#### 試すべき状態

- Default、hover、focus-visible、active、disabled
- Loading、empty、error
- Selected、open、expanded
- `<relevant feature-specific variants>`

該当しない状態には印を付ける。

#### 前提条件とセットアップ

- 認証: `<account state>`
- データ: `<fixture>`
- 権限: `<role>`
- フラグ: `<flag or none>`
- サービス: `<required availability>`

#### 証拠とクロスチェック

- スクリーンショット: `<app identity, feature, and discriminating state>`
- 動画: `<entry path, interaction, and final state>`
- クロスチェック: `<read-only state or value that confirms the UI>`

#### 落とし穴

- `<known dead end or wrong surface>`
- `<safe environment translation>`

## 架空の例

以下の機能は架空のタスクアプリのものである。これらは例であって、Benny に必須の機能ではない。

### サインイン

ユーザがタスクアプリに入れるようにする。

#### ユーザがそこへ到達する方法

- アプリを開いて `Sign in` を選ぶ。ショートカットは無い。

#### control アダプタによる駆動方法

- `open_app`、`click Sign in`、`fill credentials`、`click Continue` でアイテム一覧が開くべきである。
- サインアウトして使い捨てセッションをクリアすることでリセットする。

#### 安定したセレクタ

- ボタン `Sign in`、テキストボックス `Email` と `Password`、`data-component="sign-in-form"`

#### 試すべき状態

- Default、focus-visible、submitting、disabled、loading、error

#### 前提条件とセットアップ

- 使い捨てアカウントと、利用可能な認証サービス

#### 証拠とクロスチェック

- ランディングページからアイテム一覧までを録画する。読み取り専用のセッション状態を確認する。

#### 落とし穴

- マーケティングページは誤ったサーフェスである。認証サービスが無い場合はブロッカーである。

### アイテム一覧と詳細

ユーザがアイテムを一覧し、1 つを開けるようにする。

#### ユーザがそこへ到達する方法

- `Items` タブを開き、行を選ぶ。

#### control アダプタによる駆動方法

- `select_tab Items` と `click <fixture item>` でその詳細が開くべきである。
- 詳細を閉じ、選択をクリアすることでリセットする。

#### 安定したセレクタ

- `Items` という名前のタブと一覧、フィクスチャ名の行、`data-component="item-detail"`

#### 試すべき状態

- Loading、empty、error、selected、open、expanded

#### 前提条件とセットアップ

- 名前付きのフィクスチャアイテム、読み取り権限、利用可能なアイテムサービス

#### 証拠とクロスチェック

- 選択と、それに対応する詳細タイトルを示す。選択中アイテムの ID を確認する。

#### 落とし穴

- 検索結果は似て見えるが、別の経路を使う。

### アイテムエディタ

ユーザがアイテムを作成・編集できるようにする。

#### ユーザがそこへ到達する方法

- 詳細から `Edit` を選ぶか、一覧から `New item` を選ぶ。

#### control アダプタによる駆動方法

- `click Edit`、`fill <field>`、`click Save` で詳細が更新されるべきである。
- フィクスチャを復元することでリセットする。

#### 安定したセレクタ

- ボタン `Edit`、`New item`、`Save`、フォーム `Item editor`、ラベルと紐付いたフィールド

#### 試すべき状態

- Default、focus-visible、dirty、validating、disabled、saving、error、success

#### 前提条件とセットアップ

- 編集可能なフィクスチャ、書き込み権限、利用可能な保存サービス

#### 証拠とクロスチェック

- フィールドの変更から更新後の詳細までを示す。保存されたアイテムの値を読み取り専用で確認する。

#### 落とし穴

- フォームの状態を注入してはならない。読み取り専用の詳細フィールドはエディタではない。

### 設定

ユーザが個人設定を変更できるようにする。

#### ユーザがそこへ到達する方法

- プロフィールメニューを開き、`Settings` を選ぶ。

#### control アダプタによる駆動方法

- `open_menu Profile`、`click Settings`、`toggle <preference>` でコントロールが更新されるべきである。
- 元の設定値を復元することでリセットする。

#### 安定したセレクタ

- ボタン `Profile`、メニュー項目 `Settings`、リージョン `Settings`、目的に沿って命名された設定用属性

#### 試すべき状態

- Closed、open、selected、focus-visible、disabled、loading、error

#### 前提条件とセットアップ

- サインイン済みのテストアカウント、既知の設定値、利用可能な設定サービス

#### 証拠とクロスチェック

- メニューの経路と最終的なコントロールの状態を示す。設定値を読み取り専用で確認する。

#### 落とし穴

- OS の設定は別のサーフェスである。

## 網羅性チェックリスト

- 再現可能なユーザ向け機能すべてにセクションがある。
- すべてのセクションがユーザ経路、アダプタの操作、リセット方法を明示している。
- セレクタはロール、名前、ARIA、安定したコンポーネントマーカー、目的に沿って命名された属性を使っている。
- 生成されたクラスや DOM 上の位置を使うセレクタが 1 つも無い。
- 関連するインタラクション、loading、empty、error、selected、expanded の状態がカバーされている。
- 認証、フィクスチャ、権限、フラグ、サービスが明示されている。
- スクリーンショット、動画、裏側のクロスチェックの要件が明示されている。
- 誤ったサーフェス、行き止まり、安全な環境の読み替えが列挙されている。
- 実装の詳細は実行時の発見に委ねられたままである。
