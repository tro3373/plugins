---
name: show-me-your-work
description: "長時間動く作業や無人の作業に、レビュー可能な決定の軌跡を残す: 1 決定 1 行の TSV ログ (何を、なぜ、証拠、結果)。既定ではローカルに置き、レビュワーが結果を信頼するのに軌跡が要るときだけコミットする。/show-me-your-work、自律的または複数フェーズの実行、人が席を外した後にレビューする作業で使う。"
disable-model-invocation: true
---

# Show me your work

正典となるログを 1 つだけ保つ。

## フォーマット

TSV ファイル 1 つ、1 決定 1 行。セルは 1 行に収める。証拠はポインタであって散文ではない。

まっさらなログを始めるには `references/decision-log-template.tsv` (ヘッダ行) をコピーする。カラム:

- **ts.** ISO8601 のタイムスタンプ。
- **phase.** フェーズまたはワークストリーム。
- **decision.** 何を選んだか、何をしたか。1 行で。
- **why.** 平易な言葉での理由。原則が動機なら、ジャーゴンのタグではなく素直に述べる。
- **evidence.** それを証明するリンクかパス: コミット SHA、PR 番号、`file:line`、または成果物・トレース・スクリーンショットのパス。段落は絶対に書かない。
- **result.** 結果または述語の状態: `tests green`、`reverted`、`pixel-diff 0`、`INCONCLUSIVE`、`open`。

例を挙げる。レビュワーが一目で読めるよう平明な語り口にしてある。

```
ts	phase	decision	why	evidence	result
2026-05-24T09:02:00Z	frame	counted the work first, about 100 components and roughly 75 hours	wanted to know the size before starting a long run	commit 3a9f1c2	found 5 things to sort out before starting
2026-05-24T09:40:00Z	harness	took screenshots of the old version before changing anything	so we can compare old against new and catch any visual change	scripts/snapshot.sh, baseline/	saved 120 reference screenshots
2026-05-24T11:15:00Z	widget	moved the widget styles over without changing how it looks	keep the change small and the result identical	commit 7c21e0a, pixel-diff 0	looks identical, tests pass
2026-05-24T12:30:00Z	widget	threw out a helper's work because its screenshots were blank	checked the real files instead of trusting its summary	worktree reset	reverted, tightened the instructions for next time
```

## 行を記録する

各エントリは、同僚に自分が何をしたかを話すように書く。平易な言葉、具体的な行動、AI 語や抽象的なジャーゴンなし (**unslop** skill はログのテキストにも適用される)。

ヘルパー `scripts/log.sh <logfile> <phase> <decision> <why> <evidence> <result>` を使う。これは `ts` を打刻し、初回にヘッダを書き、紛れ込んだタブや改行を除去し、`=`、`+`、`-`、`@` で始まるセルにはシングルクォートを前置する。素の `printf` で 1 行を追記してもよいが、セルが生成物やユーザ提供のテキストから来る場合は同じバイト列に注意すること。

すべての行動ではなく、決定点とチェックポイントを記録する: 選んだ分岐、検証結果を伴う 1 単位の完了、トリガーを伴う方針転換や revert、表面化したブロッカー、直したゲート。ループ実行では 1 反復 1 行。些末で自明なものは飛ばす。

run とは 1 つのエージェントの会話を指し、その後続ターンやその要約も含む。引き継ぎ、代わりのエージェント、新しいチャットは新しい run を始める。ある run が既に行のあるログに追記するとき、その run の最初の行の phase は `start` になる。他の run の `start` 行の直後に来る、その run 自身の最初の行も同様だ。したがって、ある run が後のターンでログに戻ってくるときは、まず他の run が書き足していないかログの末尾の行を読んで確認する。`start` 行は、それより前にありこの run が書いていない行の `ts` の範囲を名指しし、evidence にはこの run 自身を示すもの (エージェント id など) を記す。phase `start` は他の用途に使わない。

## どこに置くか

既定ではログは作業用の成果物であり、コミットしない。作業ディレクトリの `decisions.tsv`、あるいは複数の取り組みが同時に走るなら `.audit/<task-slug>.tsv` に置き、git の対象外にする。

コミットするのは、レビュワーが結果を信頼するのに軌跡が必要なほど野心的な作業のときだけだ。

## ルール

- 追記専用。誤った判断は、それを上書きする新しい行で扱う。履歴を編集・削除してはならない。
- 手作りの使い捨てより、コミットされたスクリプトが生む証拠を優先する (**encode-lessons-in-structure** の原則 skill)。

## トランスクリプトと突き合わせてログを監査する

実行の最後、引き渡す前に、ログが真実を語っていたか確認する。アクティブなワークスペースの `agent-transcripts/` ディレクトリ配下にある今回の実行のトランスクリプトを読む (システムプロンプトがそのパスを示している)。`~/.cursor/projects/*/` を横断して glob してはならない。無関係な私的チャットを読むことになる。この run の行を、実際に起きたことに照らして歩く。それぞれの区間は、この run の `start` 行のいずれか (この run がログを作った場合は最初の行) から始まり、他の run の次の `start` 行の手前で終わる。

- すべての行が実在する決定または行動に対応しているかを確認する。
- 各行の証拠が解決でき、その行が主張する内容を示しているかを確認する。
- 作業を形作ったのにログに無い分岐、方針転換、放棄したアプローチは欠落だ。追加する。

ストーリーではなくログを直す。この監査は行を編集も削除もしない。捏造された行であってもだ。ある行が実在する決定にも行動にも対応していない場合、あるいはその主張や証拠が誤っている場合は、実際に起きたことと解決するポインタを添えて、その行を上書きする新しい行を追加する。この監査はこの run の区間の外にある行はチェックしない。この run 自身の作業がそのうちの 1 つが誤っていることを示すなら、他の誤った判断と同様に上書きする。

## 軌跡のクロスモデルレビュー

引き渡す前に、作業を行ったモデルとは別のモデルファミリでサブエージェントを起動する。自己レビューは代替にならない。サブエージェントは監査の軌跡と実行のトランスクリプトを読み、ユーザが注意すべき点を指摘する。作業のやり直しではなく、最適でない点やリスクのある点の走査だ。

- 証拠が弱い、あるいは無いまま記録された決定。
- 飛ばされた検証ステップ、またはトランスクリプトに証明の無いまま主張された検証。
- 後から見てリスクがあるように見える選択 (時期尚早、スコープの膨張、症状の糊塗)。
- ユーザがざっと見るだけでは見落とす欠落。

軌跡を生んだ実行の返答は、すべて "Attention" セクションで終える。まずレビュワーのモデルを単独行で示し (`reviewed by <model>`)、続いて各指摘を具体的な行や場面を指して列挙する。"No flags" は妥当な値だ。モデル名は妥当な値ではない。

## 軌跡をレビューする

上から下へ読み、証拠のポインタを辿り、抜き取りで確認する。GitHub はコミットされた TSV をテーブルとして描画する。`column -s$'\t' -t decisions.tsv` はターミナルで描画する。

## この skill を組み合わせる

他の skill は、監査の軌跡を自前で作らずここへ回す。名前で参照し、フォーマットはこの skill に所有させる。カラムを再掲しない。
