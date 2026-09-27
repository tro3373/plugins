# ノートを作成する

ノート作成機能は、ユーザがブラウザまたは CLI からタイトル付きのノートを保存し、未完成の下書きをキャンセルし、保存済みのノートを別のユーザ向けビューから確認できるようにする。

## サブ機能

- `create-open` は各ブラウザのエントリポイントから空のエディタを開く。
- `create-save` はタイトルと本文を永続化する。
- `create-cancel` は未完成のブラウザ下書きを破棄する。
- `create-cli` はターミナルから同じ形のノートを作成する。

## そこへの到達方法 (ユーザ視点)

- ブラウザのツールバーで `New note` ボタンを選ぶ。
- ブラウザで、フォーカスが編集可能フィールドの外にある状態で `n` を押す。
- ターミナルで `notes create --title <title> --body <body>` を実行する。

## control-notes による駆動

前提条件:

- Notes が `http://127.0.0.1:4173` で正常稼働している。
- `Release checklist` というタイトルのノートが存在しない。
- `control-notes doctor` が期待どおりの URL と使い捨てデータディレクトリを報告する。

- **エディタを開く。** `New note` を選ぶ。`control-notes browser click --role button --name "New note"` を実行する。`Note editor` という名前のフォームが現れ、`Title` テキストボックスにフォーカスが当たる。
- **内容を入力する。** タイトルと本文を打ち込む。`control-notes browser fill --role textbox --name "Title" --value "Release checklist"` と `control-notes browser fill --role textbox --name "Body" --value "Tag and publish"` を実行する。`Save note` ボタンが有効になる。
- **ノートを保存する。** `Save note` を選ぶ。`control-notes browser click --role button --name "Save note"` を実行する。`Note saved` という名前のステータスが現れ、見出しが `Release checklist` になる。
- **永続化を確認する。** ノート一覧に戻り、ノートを開き直す。`control-notes browser click --role link --name "All notes"` と `control-notes browser click --role link --name "Release checklist"` を実行する。エディタに保存した両方の値が表示される。
- **下書きをキャンセルする。** 新しいノートを開き、`Discard me` と入力し、`Cancel` を選ぶ。`control-notes browser click --role button --name "New note"`、`control-notes browser fill --role textbox --name "Title" --value "Discard me"`、`control-notes browser click --role button --name "Cancel"` を実行する。ノート一覧に戻り、`Discard me` のリンクは存在しない。
- **CLI からのエントリ。** 2 つ目のノートを作成する。`control-notes cli -- notes create --title "CLI note" --body "Created from terminal" --format json` を実行する。終了コードは `0` で、stdout に新しいノートの ID とタイトルが含まれる。
- **証拠。** `All notes` から保存済みの両ノートを開き直す。`control-notes browser snapshot --aria --path artifacts/create-note/list.aria.txt` と `control-notes browser screenshot --path artifacts/create-note/list.png` を実行する。アーティファクトに `Release checklist` と `CLI note` が写っている。

## 落とし穴

- テキストボックスにフォーカスがある状態で `n` を押すと、新しいエディタが開くのではなく文字が入力される。
- タイトルは保存時にトリムされる。下書きの入力値ではなく、レンダリングされたタイトルをアサートすること。
- 保存ステータスだけでは証拠として不十分。一覧からノートを開き直すこと。
- フィクスチャのクリーンアップで `Release checklist` と `CLI note` は削除するが、証拠アーティファクトは残すこと。
