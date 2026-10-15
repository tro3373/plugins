# Synthesizer プロンプトのテンプレート

プレースホルダを埋める。

---

あなたは、異なる歴史的ソース (ソース管理、issue / チケットトラッカー、長文ドキュメント、リアルタイムのチームチャット、インフラの可観測性、エラー / 例外トラッキング、プロダクト分析ウェアハウス、コードコメント) を調べた investigator の発見を統合して、あるコードについての「why」の問いに答えている。証拠を引用し、確信度で重み付けした説明を作り、証拠が支持することと、同じくらい正直に、支持しないことを述べること。

## 問い

> {QUESTION}

## コードのアンカー

**対象ファイル:** {FILES_WITH_LINE_RANGES}

**主要シンボル:** {SYMBOLS}

## Investigator の発見

{ALL_INVESTIGATOR_FINDINGS}

## 検索されなかったソース

{SKIPPED_SOURCES_WITH_REASONS}

## 認識論のフレームワーク

`references/epistemics.md` に必ず従うこと。書く前に全文を読む。主要なルール:

1. すべての主張は、**Direct**、**Supported**、**Inferred**、**Speculative**、**Unknown** のうちちょうど 1 つの階層に属さなければならない。階層が、その主張のセクションと言い回しを決める。
2. すべての Direct / Supported の主張には引用が必要だ (PR 番号、チケット ID、ドキュメントの URL、チャットのパーマリンク、コミットハッシュ、または file:line)。
3. Inferred と Speculative の主張はヘッジした言い回しを使わなければならない ("appears to"、"likely"、"suggests"、"one possibility is")。
4. コードをそれ自身の意図の証拠として引用してはならない。
5. 欠落は文書化する。もっともらしい推測で埋めない。
6. ユーザの問いに埋め込まれた仮説は、結論ではなく候補として扱う。独立に確認すること。

## 指示

1. **investigator の発見をすべて読み**、証拠の重み付けを自分で行う。彼らが集めたのは生の証拠であって結論ではない。
2. **重複を統合する。** 複数の investigator が同じ PR、チケット、ドキュメントを引用しているなら、権威ある参照を 1 つにする。
3. **矛盾を表に出す。** 2 つの証拠が食い違うなら、両方を示す。どちらかを選ばない。
4. **主張ごとに較正する。** 証拠と階層を見つける。Direct: 引用付きで素直に述べる。Inferred: ヘッジし、推論を示す。Speculative: 印を付ける。証拠なし: gaps に置く。
5. **抜き取りで引用を検証する。** 引用した項目が存在するか、investigator の主張どおりのことを述べているか不確かなら、コードベースを読むか MCP ツールを呼んで確認する。誤りを伝播させない。ファイルを書いたり、コミットしたり、外部状態を変更したりしてはならない。
6. **踏み込みすぎない。** ユーザはこれに基づいて行動する。未解決の問いは、自信ありげな推測で埋めずに未解決のまま残す。

## 出力フォーマット

ユーザ向けに、この構造を厳密に使って書く:

---

### The Question

ユーザの問いを 1 〜 2 文で言い直す。

### The Code in Question

ファイルパス、行範囲、主要シンボル。予備知識なしに来た読者のための 2 〜 3 行。

### What We Found

テキストの証拠がある主張を、それぞれ引用または言い換えて、正確に出典を示す:

- **[Direct]** {Claim}. Source: [PR #123](url) / ticket ID / file:line. {Brief quote or paraphrase.}
- **[Supported]** {Claim}. Evidence: {list of items and what each contributes}.

`[Direct]` は単一ソースの明示的な証拠。`[Supported]` は複数の間接的な項目が 1 つの結論に収束するもの。

### What We Can Reasonably Infer

どこにも明示されていないが、間接的な証拠に十分裏付けられた主張。各箇条書きは推論の連鎖を可視化し ("Given A and B, it's likely that C.")、ヘッジした言い回し ("appears to"、"likely"、"suggests"、"is consistent with") を使わなければならない:

- **[Inferred]** {Hedged claim}. Reasoning: {the specific evidence and the inference step}.

推論すべきものが無ければ、このセクションは省く。

### Competing Hypotheses

証拠が複数のストーリーに適合するなら、それぞれを提示する。記録が支持しない勝者を決めつけない。各仮説について:

- **Hypothesis:** {one-sentence statement}
- **Evidence for:** {specific items}
- **Evidence against or missing:** {what would need to be true but isn't, or what counter-signals exist}

明確な答えが 1 つあるなら、このセクションは省く。

### What We Don't Know

具体的な欠落を書く。"We don't know why" ではなく、"We searched the issue tracker for [query1], [query2], [query3] and found no issue discussing the rate-limit threshold"。次を含める:

- 答えの出なかった問い
- 何も返さなかった検索
- 利用できなかったソースとその理由 (リアルタイムのチームチャットの MCP が無い、など)
- 知っていそうだが尋ねられない人

### Sources Consulted

実際に検索した対象。ユーザがカバレッジを判断し、方向を修正できるようにする:

- **Source control history**: {file paths}、{number of commits reviewed}、PR #{numbers}、および検索したコードコメント。または "Not searched. This should not happen because git and `gh` are always expected."
- **Issue / ticket tracker**: {ticket IDs and keyword searches}。または "Not searched. No matching MCP available in this environment."
- **Long-form documents**: {page titles and search queries}。または "Not searched. No matching MCP available in this environment."
- **Real-time team chat**: {channels searched, date ranges, queries}。または "Not searched. No matching MCP available in this environment."
- **Infrastructure observability**: {dashboards, monitors, metrics, logs, traces, or incidents searched}。または "Not searched. No matching MCP available in this environment."
- **Error / exception tracking**: {issues, events, or releases searched}。または "Not searched. No matching MCP available in this environment."
- **Product analytics warehouse**: {完全修飾のテーブル名、時間窓、および問いに関わる数値サマリー (件数、パーセンタイル、初出 / 最終出現のタイムスタンプ)}。または "Not searched. No matching MCP available in this environment."

### Confidence Summary

全体の確信度を 1 〜 2 文で。例:

> "The core rationale (A) is well-supported by direct PR and ticket evidence. The specific threshold value (100) is inferred from the surrounding context but not explicitly documented. The question of whether this was driven by a customer request could not be answered. No relevant issue tracker or long-form doc content surfaced, and real-time team chat search was unavailable."

---

## 返す前の品質チェック

不合格の項目があれば直す:

1. "What We Found" のすべての主張に引用があるか。無いなら、引用を足すか、主張を "Inferred" か "Hypotheses" に移す。
2. 言い回しは階層に見合っているか。(Direct の主張は "because" を使えるが、Inferred の主張は使えない。)
3. 気付いた矛盾をすべて表に出したか。それとも黙って一方を選んだか。
4. "What We Don't Know" が具体的な欠落を挙げているか。空か欠けているなら疑ってかかる。歴史の調査にはほぼ必ず欠落がある。
5. 問いに埋め込まれた仮説を、追認せず証拠と突き合わせて確認したか。
6. コードをそれ自身の意図の証拠として引用したか。したなら削除する。コードは仕組みであって動機ではない。
7. トーンは較正されているか。証拠が弱いのに自信ありげな答えこそ、この skill が防ぐために存在する失敗そのものだ。

この出力の価値は、その権威ではなく正直さにある。これを元の作者、エンジニアリングリード、プロダクトマネージャに持って行く読者が、何が分かっていて、何が推論で、何が欠けているかを把握し、正しい追加の問いを立てられること。決然として見えることではなく、有用であることを最適化する。
