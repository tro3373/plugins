# ノートを検索する

検索により、ユーザはタイトルや本文のテキストからノートを見つけ、一致したノートを確認し、一致なしと検索が使えないことを区別できる。

## サブ機能

- `search-open` は、サポートされている各ブラウザ入口から検索を開く。
- `search-match` は、ノートのデータを変更せずにタイトルと本文の一致を返す。
- `search-open-result` は、結果をノートエディタで開く。
- `search-empty` は、一致の無いクエリに対して完全な空状態を表示する。
- `search-clear` は、クエリを除去して最近のノートのビューを復元する。
- `search-cli` は、ターミナルから同じ一致ノートを返す。

## そこへの到達方法 (ユーザ視点)

- ブラウザのツールバーで `Search` ボタンを選ぶ。
- 編集可能なフィールドの外にフォーカスがある状態で、ブラウザ内で `/` を押す。
- ターミナルで `notes search <query>` を実行する。

## control-notes での操作

前提条件:

- Notes が `http://127.0.0.1:4173` で健全であること。
- 使い捨てのデータディレクトリに、本文テキスト `Draft budget` を持つ `Quarterly plan` が含まれること。
- `control-notes doctor` が期待どおりの URL とデータディレクトリを報告すること。

- **ツールバーからの入口。** `Search` ボタンを選ぶ。`control-notes browser click --role button --name "Search"` を実行する。`Search notes` という名前のダイアログが現れ、その searchbox にフォーカスがある。
- **キーボードからの入口。** ダイアログを閉じ、ページにフォーカスして `/` を押す。`control-notes browser press --key "/"` を実行する。同じダイアログが現れ、ページはスラッシュを挿入しない。
- **タイトル一致。** `quarterly` と入力する。`control-notes browser fill --role searchbox --name "Search notes" --value "quarterly"` を実行する。`Search results` のリストは `Quarterly plan` を含み、`Grocery list` を含まない。
- **本文一致。** クエリを `budget` に置き換える。`control-notes browser fill --role searchbox --name "Search notes" --value "budget"` を実行する。結果 `Quarterly plan` は本文一致の抜粋とともに表示され続ける。
- **結果を開く。** `Quarterly plan` を選ぶ。`control-notes browser click --role link --name "Quarterly plan"` を実行する。ダイアログが閉じ、エディタの見出しが `Quarterly plan` になる。
- **空状態。** 検索を開き直し、`volcano` を入力する。`control-notes browser fill --role searchbox --name "Search notes" --value "volcano"` を実行する。検索の完了後、`No matching notes` という名前のステータスが現れる。
- **クエリの消去。** `Clear search` を選ぶ。`control-notes browser click --role button --name "Clear search"` を実行する。searchbox が空になり、`Recent notes` のリージョンが結果リストを置き換える。
- **CLI での一致。** ターミナルから検索する。`control-notes cli -- notes search "quarterly" --format json` を実行する。終了コードは `0` で、stdout は title が `Quarterly plan` のオブジェクトを 1 つ含む。
- **CLI での不一致。** 存在しない値を検索する。`control-notes cli -- notes search "volcano" --format json` を実行する。終了コードは `0` で、stdout は `[]` だ。
- **証拠。** 結果が表示された状態を取得する。`control-notes browser snapshot --aria --path artifacts/search/results.aria.txt` と `control-notes browser screenshot --path artifacts/search/results.png` を実行する。どちらの成果物も Notes、クエリ、`Quarterly plan` を識別できる。

## 落とし穴

- エディタや searchbox にフォーカスがある状態で `/` を押すと、検索が開かずテキストが挿入される。
- 結果は短いデバウンスの後に更新される。固定の sleep ではなく、結果リストか空のステータスを待つ。
- アーカイブ済みのノートは、ユーザが `Include archived` を有効にしない限り除外される。
- CLI の既定は人間可読な出力だ。安定したアサーションには `--format json` を使う。
- 結果を開くとブラウザの状態が変わる。別のクエリを証明する前に検索を開き直す。
