# レシピと落とし穴

コピーする価値のあるプロンプトと、そのあとに誰もが一度はやる間違い。パスと完了条件は自分のものに差し替えること。レシピはわざと口語的にしてある。実際に打ち込まれるのはその形だし、skill は意図をきちんと読む。

![She tastes a finished dish while robots cook from a recipe box, with pinned cards reading /how, /tdd, and /loop above the counter.](./images/recipes.jpg)

## 見知らぬサブシステムを理解する

```text
use /how first to understand how this initialization works. then use /why to figure out why it broke recently.
```

まず仕組み、次に経緯。各 skill のレポートはどのソースを検索したかを教えてくれるので、その答えが何に接地しているか分かる。

## コードに触る前にノイズの多いレポートを言い直させる

```text
/poteto-mode read this thread. restate the underlying issue in your own words, in plain english. don't change any code yet.
```

読み違いは言い直しの中に現れ、そこなら直すのにメッセージ 1 通しかかからない。エージェントが自分の見立てを述べるまで、自分の仮説は伏せておく。

## 選ぶ前に試作する

```text
/poteto-mode prototype a few options for the settings layout. put them behind a switcher and send me screenshots of each.
```

説明からではなく、動くものから選べる。エージェントはその途中で、レイアウトやタイミングの問いに自分で答える。

## 固まった設計を計画に変える

```text
/poteto-mode turn this design into a plan. small verifiable PRs, each with its own proof.
```

頼むのは設計が固まってからにする。計画そのものが成果物であり、それを実行する playbook を名指しする。

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

## レポートを証明付きで再現して直す

```text
/poteto-mode repro this with /verify-<app>. if it repros on main, fix it and show me a video as proof.
```

「if it repros on main」があれば、バグが既に消えているときに実行を早めに止められる。動画があれば、diff を読む前に修正を確かめられる。

## 数値を出す前に吟味する

```text
/benchmark-checklist vet this 40% speedup before it goes in the pr description
```

faster、slower、no measurable difference、inconclusive のいずれかが、実行回数、範囲、数値を制限しているものとともに返る。

## 同じ間違いを正すのをやめる

```text
/correct agents keep adding new config flags without registering them in the schema
```

修正はアーキテクチャ、型、lint、テストとしてリポジトリに入るので、次のエージェントはその間違いを犯せない。

## 作業を始めずに方法を尋ねる

```text
/poteto-help how do i get poteto-mode to stay on every turn?
```

答え、送るべきプロンプト、ソースへのリンクが返る。そのプロンプトを送るまで何も走らない。

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
- **原因についての自分の仮説から切り出すこと。** エージェントは指さされた場所を探す。まず問題を言い直させ、それから自分の勘を伝える。
- **最初の設計をそのまま採ること。** 1 回の試行は、モデルが最初に思いついた形に固定してしまう。試作か `/architect` を求め、証拠から選ぶ。
- **抽象的な計画を磨き上げること。** 裏にコードの無い計画を敵対的にレビューすると、決して起きないリスクを捏造する。未決の問いは試作で片付け、それから作られたものをレビューする。
- **1 つの worktree で並列エージェントを走らせること。** 互いに上書きし合い、diff が考古学になる。クラウドエージェントとして走らせるか、「own worktree per attempt」と言う。
- **ループを信頼する前にループさせること。** 自分の作業を検証できないループは、未検証の作業を速くするだけである。まず検証 skill を動くようにする。
- **吟味していない数値を信用すること。** 温まったキャッシュや飛ばされたコードパスは、高速化を偽装しうる。数値をどこかへ出す前に `/benchmark-checklist` を走らせる。
- **同じ間違いを手で正し続けること。** チャットでの訂正は 1 回の実行を助けるだけである。`/correct` はリポジトリを直し、後のどの実行も同じ間違いを繰り返さないようにする。
- **カバレッジ目的で `/arena` を使うこと。** `/arena` は 1 つの設計またはコードのブリーフを繰り返し、ベースを選んで最良の部分を接ぎ木する。`/swarm` はスライスや宣言されたレースのアームを分割し、1 つのレポートへ集約する。
- **レビューコメントをすべて受け入れること。** bot も人間も、本物の指摘とノイズを同じリストに出してくる。`/interrogate` は指摘を、理由付きで act-on と dismissed のバケツへ仕分ける。どちらの方向にも上書きできる。
- **`auto` をモデルの slug として扱うこと。** `auto` と `inherit-parent` は「モデルのフィールドを省略して、サブエージェントが親チャットのモデルを継承する」という意味である。[Setup](./01-setup.ja.md) がロールを扱っている。
- **ビルドが green だったことで成功を報告すること。** ビルドが証明するのはコンパイルが通ることだけである。実際のコマンド、フロー、保存された値、プロファイルを要求し、その証拠が返答に含まれることを期待すること。
- **`SKILL.md` を手なりで書くこと。** [skill のオーサリング・改変の playbook](../../skills/poteto-mode/playbooks/authoring-a-skill.md) を通し、検証とレビューが行われるようにすること。

以上がこのガイドである。読み飛ばしてきたなら、[setup](./01-setup.ja.md) へ戻って実際のタスクを 1 つ走らせること。習慣は読むことではなく、使うことで身につく。

[ガイドの索引](./README.ja.md) へ戻る。
