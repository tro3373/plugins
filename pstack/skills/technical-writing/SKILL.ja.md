---
name: technical-writing
description: "階層化されたテクニカルライティング標準: Diátaxis による構造、Google developer style の文、STE の指示文ルール、Global English の構文。/technical-writing のとき、あるいは docs、RFC、readme、PR 説明文、コミットメッセージを書くかレビューするときに使う。"
disable-model-invocation: true
---

# テクニカルライティング

ゴールは、疲れたエンジニアが一読で理解できる文章を書くことだ。そこに至るのは 4 つの層であり、それぞれ問いは 1 つ。これはどんな種類のドキュメントか、文は読者にどう語りかけるか、1 文がどれだけの荷物を運ぶか、そして二通りに読める文はないか。4 つすべてを適用する。

層より上位に 3 つのルールがある:

- **仕事をしていない語はすべて削る。** その語が無くても文が成立するなら、その語は消す。"In order to" は "to" だ。"It is important to note that" は無だ。
- **短く日常的な語を使う。** "utilize" ではなく "use"。"facilitate" ではなく "help"。"perform" ではなく "do"。長い語は、その長さを精度で買わねばならない。
- **ルールが文を悪くするなら、別の方法で文を直すか、そのままにする。** ルールは読者に仕えるものだ。すべてのルールに従っていて機械が書いたように読める文は、失敗している。

コードベースが単語リストだ。同義語やその説明ではなく、実在するシンボル名・ファイル名・フラグ名・コマンド名をそのまま書く。

ジャーゴンを作らない。開発者が口に出して言う語を使う: "evacuate"、"ratchet"、"endgame" ではなく、"move"、"delete"、"a budget that only decreases"。名前の付いたパターンは、初出でその意味をドキュメントが述べるなら構わない。新たな違反例とその置き換え語は、diff とともに `unslop` の abstract-metaphor ルールへの追加として自分の返答の中で提案する。その skill 自体は編集しない。

## リズムを変化させる

層はドキュメントが何を言い、1 文がどれだけ運ぶかを決める。ドキュメントはそのすべてに従いながら、なお機械が書いたように読めることがある。どの文も切り詰められ、どこにも見解が無く、具体的なものが何も無い状態だ。

- 文の長さを意図的に混ぜる。短い文は要点を着地させる。時間をかける長い文は、事実をその条件や帰結ごと運ぶ。
- 1 文 1 思考は、1 文 1 長さを意味しない。2 つの思考を運ぶ文は分割する。1 つを運ぶ長い文はそのまま残す。
- モードが許すところでは見解を持つ。Explanation はトレードオフを比較検討するのだから、長所短所を列挙するのではなく、それをどう評価するかを述べる。Reference は乾いたままにする。
- 無味乾燥より具体を。"schema changes can cause issues" ではなく "a column rename fails the build" だ。

## まずモードを選ぶ (Diátaxis)

1 ドキュメント 1 モード。選ぶ問いは 2 つ。内容は行動 (doing) に資するのか理解 (thinking) に資するのか、そして学習に仕えるのか仕事に仕えるのか。

- 行動 + 学習: **tutorial**。
- 行動 + 仕事: **how-to**。
- 理解 + 仕事: **reference**。
- 理解 + 学習: **explanation**。

このコンパスはドキュメント全体にも 1 つの文にも使う。

**Tutorial: やって学ぶ。** あなたは教師だ。学習者の成功はあなたの仕事であって、学習者の仕事ではない。学習者が何を「学ぶ」かではなく、何を作るかを述べて始める。すべてのステップが目に見える結果を、早く、頻繁に生む。何が見えるはずかを伝える: 期待される出力、プロンプトの変化、ログの行。説明は 1 節とリンクに切り詰める。教育的な立ち止まりはレッスンを壊す。具体に留まる。"we" で、命令形で書く: "First, do x. Now, do y."

**How-to: ゴールへの手順。** 機械が実行できる操作ではなく、人が抱えている問題を解く。能力があることを前提にする。教えない。行動のみ: 脱線なし、背景なし、それ自体のための網羅性なし。それらはリンクする。分岐と判断を許す: "If you want x, do y."。ガイドはタスクで名付ける: "Radar array calibration" ではなく "How to calibrate the radar array"。

**Reference: 引くための事実。** 記述する。記述だけをする。指示なし、説得なし、意見なし。乾いていて、網羅的で、確信を持って書く。事実、オプション、限界、エラーをぼかさずに述べる。記述対象の構造を鏡写しにし、コードとドキュメントを一緒に辿れるようにする。読者が期待する場所に材料を置く。可能ならコードから生成し、真実であり続けるようにする。

**Explanation: 理解と why。** 境界の定まった 1 トピックを、製品から離れても読めるように書く。どのタイトルも先頭に暗黙の "About..." を許容できるべきだ。本物の why の問いに錨を下ろす。文脈を与える: 設計上の決定、経緯、制約、代替案。意見が許されるのはここだけであり、他のどこでもない。

モードを混ぜない: tutorial の中に reference の表を置かない、reference の中で tutorial のように手を引かない、how-to の中で議論しない。分割してリンクする。

出典: diataxis.fr、2026-07-18 取得。

## 読者に向けて文を書く (Google developer style)

- 読者に "you" で、現在形で語りかける。"Will" は本当に後で起きることにだけ使う。
- 誰が何をするかを述べる: "is checked" ではなく "the compiler checks"。受動態が許されるのは、行為者が不明か、どうでもいいときだけだ。
- 指示は命令形で書く: "Click Submit."。事実は素直に述べる。"should be done" は使わない。
- 条件を指示の前に置く: "To delete the document, click Delete."。読者は当てはまらないものを読み飛ばせる。
- 一般的なケースを先に置く。例外は後だ。
- 物知りの友人のように聞こえるようにする。バズワードなし、比喩表現なし、指示に "please" なし、手順の中で "simply"、"easy"、"quickly" は絶対に使わない。簡単なら読者はここに来ていない。
- 先触れをしない ("we will soon support...")。連続する文を同じ語句で始めない。
- リンク先がどこかを示す語でリンクする: ページタイトルか短い説明。"click here" は絶対に使わない。ページ外へのリンクより、ページ内の 1 文の文脈を優先する。
- 見出しはトピックではなく要点を運ぶ ("Modes" ではなく "Pick the mode first")。センテンスケース。タスクの見出しは裸の動詞句 ("Create an instance")。概念の見出しは名詞句。1 ページに h1 は 1 つ、レベルを飛ばさない。
- 順序には番号付きリスト、それ以外はすべて箇条書き。リストは完全な文で導入する。項目は並列に保つ。
- コードはコードフォントに。UI 要素は太字に。シリアルコンマを使う。"etc." は捨て、リストが一部であることは先に述べる。

出典: developers.google.com/style、2026-07-18 取得。

## 記述を一度に 1 つずつ載せる (STE ルール)

- 1 文 1 指示。それ以外のどこでも 1 文 1 思考。
- 指示文はおよそ 20 語、その他の文はおよそ 25 語を超えたら分割する。
- 警告や条件は、それが守るステップの前に置く: "If hot oil touches your skin, injuries can occur."
- "the" と "a" を保つ: "Remove backup file" は二通りに読める。"Remove the backup file" は一通りだ。
- 各語に 1 つの意味と 1 つの役割を与え、それを守る。"check" が inspect の意味なら、restrain の意味でも使わない。
- 1 つの動作につき 1 つの語を選び、貫く: ここでは "start"、あそこでは "initiate" とはしない。
- 手順は直接の命令として書き、決して語りとして書かず、決して受動態で書かない: "the component must be installed" ではなく "Install the component"。
- 可能な限り "-ing" 形を避ける。文法上の役割を持ちすぎ、誤読を生む。

出典: asd-ste100.org (Issue 9, 2025)、2026-07-18 取得。番号付きルールと辞書は仕様 PDF にある。上記の原則は移植可能な中核だ。

## どの文も二通りに読めないようにする (Global English)

- "only" や "not" のような語は、それが修飾する語の隣に置く: "only fails on growth" と "fails only on growth" は別のことを言う。
- 長い名詞連結を分解する: "the proto import budget check script" は "the script that checks the proto-import budget" になる。
- すべての "it"、"they"、"this" が 1 つの明白なものを指すようにする。疑わしければ名詞を繰り返す。"this" や "which" で節全体を指してはならない。
- 動詞を落とさない: "Phase 1 moves the converters and Phase 2 the runtime" は Phase 2 に動詞が無い。与える。
- 構造を示す小さな語を保つ。"Ensure that the switch is off" が "that" を保つのは、文の解析を一通りにするからだ。明快さを語数と引き換えにしてはならない。
- 誤読を防ぐなら、並列で冠詞を繰り返す: 2 つのものなら "the client and host" ではなく "the client and the host"。
- 文が二通りにグループ化できるとき、"and" や "or" が何を繋ぐかを述べる。"Both...and"、"either...or"、"if...then" はコストのかからない曖昧性解消装置だ。
- セミコロンではなくピリオドを使う。em dash は新しい文に置き換える。
- 括弧内のテキストは、文法的に完結した単位にするか、独立した文にする。"(s)" で複数形を作ってはならない。
- スラッシュは使わない: "a/b" や "and/or" ではなく "a, b, or both" と書く。
- どこでも 1 つのものは 1 つの名前で呼ぶ。1 つのものを "the gate"、"the ratchet"、"the budget check" と呼ぶドキュメントは、3 つのものを教えている。変わっていない文を編集のたびに言い換えるのも同じコストを払う。変わっていないものを churn しない。
- 慣用句、口語表現、ラテン語の略記、比喩を避ける。非ネイティブの読者も、翻訳者も、エージェントも、平明な構文を最もよく解析する。

出典: Kohl, The Global English Style Guide (SAS Press)。ガイドライン本文は Internet Archive と SAS のサンプル章から 2026-07-18 に取得。

## 声とリポジトリ固有の事項

- この skill が触れるすべてのドキュメントに **unslop** skill を適用する。slop パターンのカタログ (AI 語彙、埋め草、ヘッジ、フォーマット上の兆候) はその skill が所有する。
- PR の説明文とコミットメッセージも文章だ。Diátaxis を除くすべての層が適用される。PR 本文は、レビュワーが 1 分以内に読めるブリーフィングだ。swarm のログ、SHA のリスト、メトリクスの表を貼り付けない。それらはリンクする。
- 製品の UI 文言はドキュメントではない。それには自分の製品のコピーガイドラインを使う。
- コードスニペットはタブでインデントする。実在するパスと実在するシンボルを書く。数やツリーに関する主張は、それを含むコミットの時点で真であるようにし、それを再生成するコマンドを添える。

## 実例

Before:

> Configuration of the proto import ratchet budget script parameters is performed via budget.json. Note that it's important to remember that running with --write, which updates the committed budget to reflect the current count, should only be done when lowering it. If exceeded, CI fails.

After:

> `budget.mjs` reads the committed budget from `budget.json` and counts the files that import protos. If the count exceeds the budget, CI fails. Run `budget.mjs --write` only to lower the budget.
