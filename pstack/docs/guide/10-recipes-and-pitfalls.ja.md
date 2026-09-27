# レシピと落とし穴

コピーする価値のあるプロンプトと、そのあとに誰もが一度はやる間違い。パスと完了条件は自分のものに差し替えること。レシピはわざと口語的にしてある。実際に打ち込まれるのはその形だし、skill は意図をきちんと読む。

![She tastes a finished dish while robots cook from a recipe box, with pinned cards reading /how, /tdd, and /loop above the counter.](./images/recipes.jpg)

## 見知らぬサブシステムを理解する

```text
use /how first to understand how this initialization works. then use /why to figure out why it broke recently.
```

まず仕組み、次に経緯。各 skill のレポートはどのソースを検索したかを教えてくれるので、その答えが何に接地しているか分かる。

## 設計についてセカンドオピニオンを得る

```text
ask /arena for a second opinion on this thread and our approach
```

今の設計が複数候補のうちの 1 つになり、統合結果が、パネルがより良いものを見つけたのか、手元のものを追認したのかを教えてくれる。高くつくコミットメントの前の、安い保険である。

## 独立したスライスを並列に確認する

```text
/swarm check every package under packages/ against its check.sh. one worker per package. one report.
```

各ワーカーが 1 パッケージを所有する。親はすべてのスライスを待ち、ワーカーの生ダンプではなく `PASS`、`ISSUES`、`BLOCKED` のレポートを 1 つ返す。

## ブランチを懐疑的にレビューする

```text
/interrogate the whole branch, but skeptically. don't change anything yet. no nitpicks unless it's an actual bug or regression in behavior.
```

この修飾句は実際に効く。「don't change anything yet」が読み取り専用に保ち、nitpick のルールがノイズを事前に濾すので、`Act on` の指摘があなたの時間に見合うものになる。

## 失敗するテストを通してバグを直す

```text
/poteto-mode repro the duplicate write first. if there's a cheap test path, /tdd it. then fix and rerun.
```

「if there's a cheap test path」が重要である。壊れやすいモックを通してテストを無理に通しても、実際のコマンドを走らせるより証明できることは少ない。playbook はそう言ってよいことになっている。

## 離席中も実行を正直に保つ

```text
im going to bed, keep going autonomously until every fixture passes. do not stop. keep a decision log i can audit in the morning.
```

完全な契約は [overnight のページ](./07-overnight.ja.md) にある。タスクと完了条件が既に会話に出ていれば、この短い形で足りる。

## ずれていく実行を引き戻す

舵取りのプロンプトは 1 行である:

```text
i said the goal is to repro. i did not ask for a fix yet.
```

```text
apply prove it works. show me the real output, not the build log.
```

```text
/unslop that, no emdashes
```

たいてい、それ以上の言葉は要らない。要るのは正しい名前であり、[principles のページ](./08-principles.ja.md) がその語彙である。

## 返答を平易な言葉で得る

```text
/bro
```

これでプロンプト全体である。[`/bro`](../../skills/bro/SKILL.md) は直前のメッセージを、人間が人間に話すように、専門用語なしで、より短く言い直す。返答が技術的には網羅的なのに、結局何を言われたのか分からないときに使う。

## 落とし穴

- **プロンプトで skill を列挙すること。** 「use /how then /architect then /arena」は、playbook が既に順序付けているステップを並べ替えてしまう。ゴールと制約を述べること。skill を名指しするのは、既定を上書きするときだけにする。
- **曖昧な完了条件。** 「make it better」では `/loop` に確認するものが何も無い。合格か不合格かを判定できるコマンドか成果物を与えること。
- **1 つの worktree で並列エージェントを走らせること。** 互いに上書きし合い、diff が考古学になる。「own worktree per attempt」と言えば、隔離はタダで手に入る。
- **カバレッジ目的で `/arena` を使うこと。** `/arena` は 1 つの設計またはコードのブリーフを繰り返し、ベースを選んで最良の部分を接ぎ木する。`/swarm` はスライスや宣言されたレースのアームを分割し、1 つのレポートへ集約する。
- **レビューコメントをすべて受け入れること。** bot も人間も、本物の指摘とノイズを同じリストに出してくる。`/interrogate` は指摘を、理由付きで act-on と dismissed のバケツへ仕分ける。どちらの方向にも上書きできる。
- **`auto` をモデルの slug として扱うこと。** `auto` と `inherit-parent` は「モデルのフィールドを省略して、サブエージェントが親チャットのモデルを継承する」という意味である。[Setup](./01-setup.ja.md) がロールを扱っている。
- **ビルドが green だったことで成功を報告すること。** ビルドが証明するのはコンパイルが通ることだけである。実際のコマンド、フロー、保存された値、プロファイルを要求し、その証拠が返答に含まれることを期待すること。
- **`SKILL.md` を手なりで書くこと。** [skill のオーサリング・改変の playbook](../../skills/poteto-mode/playbooks/authoring-a-skill.md) を通し、検証とレビューが行われるようにすること。

以上がこのガイドである。読み飛ばしてきたなら、[setup](./01-setup.ja.md) へ戻って実際のタスクを 1 つ走らせること。習慣は読むことではなく、使うことで身につく。

[ガイドの索引](./README.ja.md) へ戻る。
