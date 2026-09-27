---
name: unslop
description: あらゆる文章から AI 臭 (AI tells) を取り除く。常に適用しなければならない。
disable-model-invocation: true
---

# Unslop

テキストを編集して AI 的なパターンを取り除く。

## 手順

1. 下記のパターンを走査する。
2. 書き直す。意味を保ち、意図されたトーンに合わせる。

## 検出して直すパターン

ルール番号は他の skill が参照する安定した id だ。ルールを削除しても、その番号は空けたままにする。

### 内容

3. **表層的な -ing 句。** 「highlighting...」「ensuring...」「reflecting...」「showcasing...」「fostering...」。削除するか、実際の出典で展開する。
5. **曖昧な帰属。** 「Experts believe」「Industry reports suggest」「Some critics argue」。出典を名指しするか削除する。

### 言葉遣い

7. **AI 語彙。** Additionally, crucial, delve, enduring, enhance, fostering, garner, interplay, intricate, landscape (抽象的な意味), pivotal, showcase, tapestry (抽象的な意味), testament, underscore, vibrant。平易な語に置き換える。
8. **「である」の気取った言い換え。** 「serves as」「stands as」「boasts」「features」。単に「is」「has」と言えばよい。
9. **「Not just X, but Y」。** 代わりに要点を直接述べる。
10. **3 つの法則。** アイデアを無理に 3 つ組にする。自然な個数を使う。
11. **同義語の使い回し。** Protagonist、main character、central figure、hero が 1 段落に全部出てくる。1 つ選び、それを繰り返す。
12. **偽の範囲。** 「from X to Y」で X と Y が意味のあるスケール上に無いもの。トピックを直接列挙する。

### スタイル

13. **em dash の乱用。** em dash は一切使わない。ピリオドかカンマのみを使う (括弧も en dash もハイフンによる代用もしない)。思考を切り離す必要があるなら、文を終えるかカンマを使う。
14. **コロンの乱用。** コロンはリストや例の前なら問題ない。文中の接続詞としては駄目だ。「If you're coming from traditional automation: instead of registering event handlers, you describe conditions」はコロンによって何も足していない。比較の枠組みなしに要点が自立するよう書き直す。「Describing when the scheduler should fire works best as plain English.」意味は同じで、松葉杖の句読点は無い。
15. **太字の乱用。** 固有名詞や略語を片っ端から太字にしない。
16. **インラインヘッダのリスト。** AI 臭なのは、太字のラベルとコロンがその行を言い直しているもの: 「**Performance:** Performance improved...」。それらは散文に変える。太字の導入がピリオドで終わり、対象を名指しし、その後に本当に新しい詳細が続くもの (「**Schema in TypeScript.** Tables live in one file.」) は問題なく、AI 臭ではない。
17. **タイトルケースの見出し。** センテンスケースを使う。
18. **装飾的な絵文字。** 見出しと箇条書きから取り除く。
19. **カーリークォート。** 直線クォートに置き換える。

### コミュニケーションの残骸

20. **チャットボット的な言い回し。** 「I hope this helps!」「Let me know if...」「Of course!」「Certainly!」「Found the smoking gun!」削除する。
22. **迎合的なトーン。** 「Great question! You're absolutely right!」直接答える。

### 埋め草

23. **埋め草フレーズ。** 「In order to」は「To」になる。「Due to the fact that」は「Because」になる。「It is important to note that」は削除される。
24. **過剰なヘッジ。** 「could potentially possibly be argued that it might」は「may」になる。
25. **一般論の結び。** 「The future looks bright.」具体的な計画か事実を述べる。

### ジャーゴン

26. **抽象的なメタファー名詞。** Substrate, wedge, vector, locus, vantage, nexus, primitive (名詞として), harness (メタファーとして), surface (「API surface」のような), bedrock, scaffolding (メタファーとして), modality, paradigm, gold-plating, ratchet (メタファーとして), evacuate (コードの移動を指して), endgame, north star, flywheel。これらは技術的に読めるが、たいていもっと平易で具体的な語がある。「Substrate」は「base」になる。「Wedge in」は「add」になる。「Vector」は「way」か「method」になる。「Gold-plating」は「more than the job needs」になる。「Ratchet」はその仕組みの実際の名前か「a limit that only tightens」になる。「Evacuate」は「move out」になる。「Endgame」は「the last phase」になる。具体的な語を選ぶ。

### 平易な物言い

27. **どう感じるかではなく、何をするかを書け。** 「the database stays close at hand」「SQL you can read」「types that follow your schema」は感覚を名指ししている。修正は仕組みか数字を名指しする: 「`.toSQL()` returns the exact string sent to the database」「a column rename fails the build」。その文が読者に何をさせるか、何を知らせるかを問い、それを書く。具体的な指示・事実・数字に言い換えられないなら、削る。もう 1 つのチェック: その文が別のプロジェクトのドキュメントにそのまま載るなら、このプロジェクトについて何も言っていない。削る。
28. **密な文は短くするか分割しろ。** 読者が文を解釈するのに読み返しを要するなら、2 つに割るか節を落とす。1 文につき 1 アイデア。
29. **能動態。** 能動態を優先する。「is/are/was/were + 過去分詞」を捕まえて、行為者を名指しする: 「queries are validated」は「the compiler validates queries」になり、「the file is parsed by the loader」は「the loader parses the file」になる。受動態が許されるのは、行為者が不明であるか本当にどうでもよい場合だけだ。
30. **副詞を削るか、より強い動詞を使え。** 「runs quickly」は「is fast」または数字になる。「significantly improves」は計測された差分になる。弱い動詞を副詞で支えているなら、その動詞が間違っている。
31. **平易な語を選べ。** 「utilize」は「use」に、「leverage」は「use」に、「facilitate」は「help」に、「numerous」は「many」に、「in the event that」は「if」になる。気取った同義語のほうが明快であることはめったにない。
32. **気取った文飾。** 文字通りの言い方があるのに、比喩や修辞を使うもの: 警句 (「wire it or delete it」)、効果狙いの修辞的断片、擬人化されたコード (「the plan holds it」)、比喩的な動詞 (「rides along」「stands on」)、決まり文句の枠組み。「A dial worth turning」は「a parameter worth varying」になる。言いたいことをそのまま言う。比喩的な名詞についてはルール 26 が扱う。
33. **過剰な圧縮。** 冠詞の省略、動詞の無い断片、記号による省略記法、読者に読ませず解読させる略語。「Parser rejects bad date → exit 2, no write」は「The parser rejects a bad date, exits with code 2, and writes nothing.」になる。冠詞と動詞を備えた完全な文を書き、矢印や略語は書き出す。
