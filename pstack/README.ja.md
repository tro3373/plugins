# pstack

私は [poteto](https://x.com/poteto) だ。社長でも CEO でもないが、Meta、Netflix、Cursor で数百万行のコードを扱ってきた。react コアチームの一員でもあり、そこで react compiler の開発と保守に携わっている。

ai が slop なコードを書きすぎている、という感覚が広がっている。私も同意する。20 人の slop アーティストのチームのようにコードを出したくはない。品質を伴わないスループットは、私が目指すゴールではない。速く進みたいなら、まず深く潜れ。

**pstack が私の答えだ。** これは、私が Cursor で高品質なコードを出すために毎日使っているのと同じ skill 群だ。これは cursor を本物のエンジニアリングチームに変える。ゴールは loc を最大化することではない。実際、むしろ逆だ。pstack は、より少なく、しかしより高品質なコードを書く助けになる。

**pstack は恐れのない並列化をもたらす。** 1 つのエージェントに深く潜らせ、良質で検証可能なコードを書くと信頼できるなら、本当の意味で自信を持って並列化できる。`poteto-mode` で複数のエージェントを立ち上げ、それぞれが厳密なエンジニアリング原則を仕事に適用すると信頼せよ。

**cursor はすべての世界の best をもたらす。** どのフロンティアモデルにも強みと弱みがある。pstack はどのモデルとでも使える。実際、私の skill の多くは、各モデル固有の強みを活かすためにマルチモデルのワークフローを使っている。

fork せよ。改善せよ。自分のものにせよ。PR は歓迎だ!

## インストール

```bash
/add-plugin pstack
```

## はじめかた

2 ステップだ:

1. [`/setup-pstack`](./skills/setup-pstack/SKILL.md) を実行し、reasoning budget を選び、使いたいモデルを選ぶ。
2. 厳密さを要する作業をするときは常に [`/poteto-mode`](./skills/poteto-mode/SKILL.md) を使う。

はじめてか? [pstack ガイド](./docs/guide/README.md) が、セットアップとプロンプトから検証・夜間実行まで、最初の実タスクを通しで案内する。

以上だ。他の skill は状況次第のもので、mode skill が必要に応じて使ってくれる。標準では、mode はモデルの強みで仕事を分ける: コードの委任先 (feature、refactoring、bug fix、perf、hillclimb) は grok へ、最も難しい変更・散文・判断は opus 5.5 へ回る。既定のパネルは opus 5.5 / sol / grok だ。[`/setup-pstack`](./skills/setup-pstack/SKILL.md) でそのどれでも変えられる。

## 使い方

タスクの開始時に [`/poteto-mode`](./skills/poteto-mode/SKILL.md) を使う。リクエストを読み、playbook の集合から 1 つを選び、手順が必要とするタイミングで他の skill を実行する。

### とにかく [`/poteto-mode`](./skills/poteto-mode/SKILL.md) を使え

この skill が主要なショートカットだ。エージェントに厳密なエンジニアリング作業をさせたいときは常にこれを使う。23 本の playbook が付属する:

```
/poteto-mode this pr has a subtle bug where the scroll drifts every 750ms even when idle. repro
first, then fix and verify.
```

```
/poteto-mode i'm going to bed. land the stack even if ci flakes. i want everything merged by
morning.
```

<details>
<summary>23 本の playbook</summary>

| playbook | 用途 |
|---|---|
| [investigation](./skills/poteto-mode/playbooks/investigation.md) | 読み取り専用の問い。x はどう動くのか、なぜ y はこう作られたのか、本当にそうだと言い切れるのか。 |
| [bug fix](./skills/poteto-mode/playbooks/bug-fix.md) | 欠陥を再現し、根本原因を突き止め、ランタイムの証拠を伴って直す。 |
| [perf](./skills/poteto-mode/playbooks/perf-issue.md) | 計測された遅さをトレースし、ベースラインに対して改善する。 |
| [hillclimb](./skills/poteto-mode/playbooks/hillclimb.md) | 1 つのメトリクスを目標に対して継続的・科学的に改善する。仮説をループで回し、前後を計測し、採用された勝ち 1 件につき 1 コミットとする。 |
| [runtime forensics](./skills/poteto-mode/playbooks/runtime-forensics.md) | 稼働中の症状 (リーク、アイドル時の cpu スピン、グリッチ) を計装から診断する。 |
| [trace forensics](./skills/poteto-mode/playbooks/trace-forensics.md) | 取得済みのプロファイリング成果物 (cpuprofile、trace、spindump、ヒープスナップショット) を診断する。 |
| [feature](./skills/poteto-mode/playbooks/feature.md) | 新規または変更された挙動を、名前の付いたデータ形状から組み立てる。 |
| [refactoring](./skills/poteto-mode/playbooks/refactoring.md) | 構造や形状に対する、挙動を保つ変更。 |
| [prototype](./skills/poteto-mode/playbooks/prototype.md) | 設計や挙動の決定を安く下すため、あるいは経験的な分岐を観察によって決着させるための使い捨てスケッチ。 |
| [visual parity](./skills/poteto-mode/playbooks/visual-parity.md) | 2 つの実装間での、ピクセル単位に正確な ui の等価性。 |
| [authoring a skill](./skills/poteto-mode/playbooks/authoring-a-skill.md) | SKILL.md を書く、または編集する。 |
| [eval](./skills/poteto-mode/playbooks/eval.md) | skill やプロンプトの変更がエージェントの挙動にどう影響するかを、ブラインドでテストする。 |
| [babysit](./skills/poteto-mode/playbooks/babysit.md) | pr またはスタックを merge-ready まで進める: 競合、レビュースレッド、ci。 |
| [shipping](./skills/poteto-mode/playbooks/shipping.md) | green なスタックを独立に検証し、検証済みで連続した並びを、既定では github 経由、使えるなら origin 経由で、下から順に land する。 |
| [autonomous run](./skills/poteto-mode/playbooks/autonomous-run.md) | 長いタスクを止まらずに完了まで進める。 |
| [orchestrate](./skills/poteto-mode/playbooks/orchestrate.md) | 1 つのコーディネータチャットに委ねられる常設プロジェクト: 複数日、多数の積み重なった pr、サブエージェントの艦隊。 |
| [autopilot-full](./skills/poteto-mode/playbooks/autopilot-full.md) | 独立した pr を、pr ごとに 1 人のオーナーを付けて merged まで走らせ、code-ready な head 以降の各ラウンドをルートの swarm 判定にかける。 |
| [autopilot-stack](./skills/poteto-mode/playbooks/autopilot-stack.md) | オペレータがレビューして land できるよう、1 本の線形な base-branch スタックを組み立てて検証する。 |
| [session pickup](./skills/poteto-mode/playbooks/session-pickup.md) | 先行エージェントの進行中の作業を再開する、または引き継ぐ。 |
| [pause safely](./skills/poteto-mode/playbooks/pause-safely.md) | 後で再開できるよう、進行中の作業をきれいに中断する。 |
| [multi-phase plan](./skills/poteto-mode/playbooks/multi-phase-plan.md) | フェーズや積み重なった PR にまたがる作業。 |
| [worktree cleanup](./skills/poteto-mode/playbooks/worktree-cleanup.md) | マージ済みまたは放棄された worktree と、古い ios シミュレータを刈り取ってディスクを回収する。安全ゲート付き。 |
| [opening a pr](./skills/poteto-mode/playbooks/opening-a-pr.md) | 小さく順序立てられたコミットから、conventional commits のタイトルと briefing 形式の本文を持つ、ready な pr を開く。他のすべての playbook の末尾で呼ばれる。 |

</details>



起動されると、次を行う:

1. タスクを [playbook](./skills/poteto-mode/playbooks/) に対応付け、todo リストを開く。最初の項目群はその手順を一字一句そのままコピーしたものだ。
2. 手順が発火するのに合わせて他の skill へルーティングする。
3. 利用者と保守者に向けて枠付けされた、unslop 済みの返答を書く。

完全なルールと playbook は [`skills/poteto-mode/SKILL.md`](./skills/poteto-mode/SKILL.md) にある。

[`/poteto-mode`](./skills/poteto-mode/SKILL.md) は sticky なモードでもある: 一度入るとターンをまたいで有効なままになり、playbook が一致するか、タスクに厳密さが要るときに自らを適用し、それ以外では邪魔をしない。抜けたいときは、そう言えばいつでも抜けられる。

[`/poteto-mode`](./skills/poteto-mode/SKILL.md) は cursor の `/loop` コマンドと極めて相性が良い。厳密さを犠牲にせず、cursor を何時間も働かせられる。

## skill 群

手順がそれらを必要とするとき、[`/poteto-mode`](./skills/poteto-mode/SKILL.md) がこれらの大半を代わりに実行する (`how`, `why`, `architect`, `arena`, `swarm`, `interrogate`, `unslop`, `no-comments`, `technical-writing`, `tdd`、および原則群)。下の表は、どれか 1 つを直接使いたいときのためのものだ:

```
/how do we cancel runs? do we have an n+1 when we look up every run to cancel?
```

```
/interrogate review this pr.
```

<details>
<summary>すべての skill</summary>

| skill | 使いどころ |
|---|---|
| [`/poteto-mode`](./skills/poteto-mode/SKILL.md) | 自明でないあらゆるタスクの既定の入口。 |
| [`/how`](./skills/how/SKILL.md) | サブシステムがどう動くのかのウォークスルーが欲しいとき。 |
| [`/why`](./skills/why/SKILL.md) | 何かがなぜこう作られたのかを知りたいとき。実行時に利用可能な MCP を発見し、証拠のカテゴリごとに並列で問い合わせる (ソース管理、課題トラッカー、長文ドキュメント、リアルタイムチャット、インフラの可観測性、エラートラッキング、分析ウェアハウス)。 |
| [`/recall`](./skills/recall/SKILL.md) | 作業を始める、または再開するにあたって、あるトピックについての最近のコンテキストを、自分のチャット履歴と共有された記録から再構築し、簡潔な現状ブリーフとして受け取りたいとき。 |
| [`/blast-radius`](./skills/blast-radius/SKILL.md) | 小さく見える変更があり、他に何を壊しうるかを知りたいとき。安全だと言える根拠となるその 1 つの事実を、主張ではなく実際に動くコードで証明する。 |
| [`/architect`](./skills/architect/SKILL.md) | 関数境界をまたぐコードをこれから書くので、呼び出し側の使い方、型、モジュールの形を先に固めたいとき。 |
| [`/arena`](./skills/arena/SKILL.md) | 同じ対象への並列な試行を N 個走らせ、それぞれの最良の部分を取り込みたいとき。 |
| [`/swarm`](./skills/swarm/SKILL.md) | 異なるスライスやレースにまたがる並列ワーカーを N 個走らせ、集約された 1 本のレポートが欲しいとき。 |
| [`/interrogate`](./skills/interrogate/SKILL.md) | diff があり、複数の異なるモデルにそれを壊させたいとき。厳格なコード品質の観点も含む。 |
| [`/automate-me`](./skills/automate-me/SKILL.md) | 実際の働き方から起こした、自分自身の `-mode` skill が欲しいとき。 |
| [`/make-bot-ui`](./skills/make-bot-ui/SKILL.md) | ボタンが webhook 経由で Grok Bot を起こすページやダッシュボードが欲しいとき。sender-key の受け渡しと Tailscale も含む。 |
| [`/setup-pstack`](./skills/setup-pstack/SKILL.md) | pstack がロールごとにどのモデルを使うかを選びたいとき。利用できるモデルを検出し、設定ルールを書き出す。 |
| [`/reflect`](./skills/reflect/SKILL.md) | 長いタスクが着地し、その手順を skill の編集として残したいとき。 |
| [`/teach`](./skills/teach/SKILL.md) | 変更やサブシステムを、要約されるだけでなく実際に理解したいとき。how + why を実行し、図を 1 枚ずつ積み上げながら 1 本の平易な説明に織り上げる。 |
| [`/tdd`](./skills/tdd/SKILL.md) | バグを直していて、安価なローカルテスト経路があるとき。失敗するテストを先に書き、それから修正する。 |
| [`/no-comments`](./skills/no-comments/SKILL.md) | レビュー前にコメントを剥がす。Comment Sicko を起動し、受け入れた指摘を修正し、主張された制約については構造への符号化を提案する。 |
| [`/typescript-best-practices`](./skills/typescript-best-practices/SKILL.md) | typescript を読む、または編集するとき。type-system-discipline の原則を構文に接地させる。 |
| [`/figure-it-out`](./skills/figure-it-out/SKILL.md) | 同梱の playbook がどれも合わないとき。そのタスクのために厳密で監査可能な playbook を設計する。 |
| [`/show-me-your-work`](./skills/show-me-your-work/SKILL.md) | レビュー可能な意思決定の軌跡が欲しいとき。決定を、コミットできる tsv に記録する。 |
| [`/create-verification-skill`](./skills/create-verification-skill/SKILL.md) | プロジェクトに、アプリの挙動を証明するスクリプト化された手段が無いとき。どの言語・プラットフォームでも、feature map 付きのプロジェクトローカルな verify skill を生成する。 |
| [`/maintain-verification-skill`](./skills/maintain-verification-skill/SKILL.md) | verify skill の feature map がアプリから乖離したとき。ソースの波 + 1 回のライブパスで、証明済みの修正を最大 1 本の PR にまとめる。 |
| [`/unslop`](./skills/unslop/SKILL.md) | 文章を整えているとき。AI っぽさの痕跡を取り除く。 |
| [`/bro`](./skills/bro/SKILL.md) | 直前のメッセージを、専門用語なしの平易な人間の言葉で言い直してほしいとき。 |
| [`/technical-writing`](./skills/technical-writing/SKILL.md) | ドキュメント、RFC、readme、PR の説明、コミットメッセージのための、階層化されたドキュメント標準 (Diátaxis + Google developer style + STE + Global English)。 |

</details>



### 例

ほとんどの場合、私はタスクの開始時に [`/poteto-mode`](./skills/poteto-mode/SKILL.md) と打ち、playbook へのルーティングを任せる。他の skill は手順が必要とするタイミングで発火する。いくつかは直接使う。


<details>
<summary>すべての例</summary>

```
bug fix:           /poteto-mode this pr has a subtle bug where the scroll drifts every 750ms even
                   when idle. repro first, then fix and verify.
perf:              /poteto-mode a big list takes a second or two to load even though we virtualize.
                   run a cpu trace and tell me why.
feature:           /poteto-mode build a small feature behind a feature flag. verify it really works.
prototype:         /poteto-mode build two prototypes of the markdown renderer so we can compare.
                   spawn an agent for each.
multi-phase:       /poteto-mode open source these skills as a plugin. nothing internal leaks, work
                   in a temp dir, show me the dependency graph first.
overnight run:     /poteto-mode i'm going to bed. land the stack even if ci flakes. i want
                   everything merged by morning.
babysit:           /poteto-mode check on pr 123. anything outstanding?
visual parity:     /poteto-mode the row spacing is too tall when this flag is on. the second image
                   is correct. repro and fix until it matches.
figure it out:     /poteto-mode i'm stepping away. migrate every caller from the synchronous store
                   to the new async one, keeping behavior identical. i want to trust it was done
                   right when i'm back.
how:               /how do we cancel runs? do we have an n+1 when we look up every run to cancel?
why:               /why is this feature flag not on yet?
architect:         design this instrumentation to be high signal with no false positives. /architect
                   this first.
arena:             /arena take my prompt to the arena verbatim. i want to compare their proposals
                   with yours.
swarm:             /swarm check every package under packages/ against its check.sh. one worker per
                   package. one report.
interrogate:       /interrogate review this pr.
tdd:               /tdd implement
unslop:            can we unslop and tighten the new changes?
reflect:           /reflect that took too long. capture what we learned so the next run doesn't
                   repeat it.
show-me-your-work: /show-me-your-work keep a decision trail i can review when i'm back.
automate-me:       /automate-me
```

</details>

## `poteto-agent` と Comment Sicko サブエージェント

pstack は、私のスタイルを端から端まで実行するサブエージェントも同梱している。親エージェントから [`subagent_type: "poteto-agent"`](./agents/poteto-agent.md) で起動する。作業に取りかかる前に、インラインの原則インデックスを含めて `poteto-mode` を全文読む。`generalPurpose` で代用するとその読み込みが飛ばされ、逸脱する。

[`/poteto-mode`](./skills/poteto-mode/SKILL.md) と [`subagent_type: "poteto-agent"`](./agents/poteto-agent.md) は同じラッパーを経由する。

pstack は [Comment Sicko](./agents/comment-sicko.md) も同梱している。`subagent_type: "Comment Sicko"` として使える、読み取り専用のコメントレビュワーだ。通常は直接ではなく [`/no-comments`](./skills/no-comments/SKILL.md) 経由で起動する。

## 原則

短い skill が 23 本あり、それぞれが 1 つの原則を担う。`poteto-mode` はそれらをインラインでインデックス化し、タスク開始時にそのインデックスを読む。独立したファイルが存在するのは、他の skill が原則を名前で参照できるようにするため、そしてインデックスが各原則の完全なルールを指せるようにするためだ。

<details>
<summary>23 個の原則すべて</summary>

| 原則 | グループ | ルール |
|---|---|---|
| [laziness-protocol](./skills/principle-laziness-protocol/SKILL.md) | コア | 削除と、問題を解決する最小の変更に倒す。 |
| [foundational-thinking](./skills/principle-foundational-thinking/SKILL.md) | コア | ロジックを書く前に適用する: 中核の型とデータ構造を選ぶ、足場作りと機能実装の順序を決める、並行アクタが何を共有するのかを問う。データ構造を正しくすれば、下流のコードは自明になる。 |
| [redesign-from-first-principles](./skills/principle-redesign-from-first-principles/SKILL.md) | コア | 後付けで継ぎ足すのではなく、その要件が初日から基礎的な前提だったかのように再設計する。 |
| [attack-the-premise](./skills/principle-attack-the-premise/SKILL.md) | コア | 1 つの前提を共有する 2 件以上の修正が同じゲートに失敗したときに適用する。次の修正を書く前に、どのアクタが不均衡を抱えているかを一通り調べ、その前提を仮定した修正をもう 1 本書くのではなく前提そのものを疑う。 |
| [subtract-before-you-add](./skills/principle-subtract-before-you-add/SKILL.md) | コア | まず死荷重、冗長なバリデータ、スタブへの参照を取り除き、それから単純になった土台の上に作る。 |
| [minimize-reader-load](./skills/principle-minimize-reader-load/SKILL.md) | コア | 問いと答えの間にある層と、読み手の頭の中の隠れた状態を数える。呼び出し元が 1 つだけのラッパーを畳み、ミュータブルなスコープを縮める。 |
| [outcome-oriented-execution](./skills/principle-outcome-oriented-execution/SKILL.md) | コア | 明示的なフェーズ境界を持つ、計画された書き換えや移行の最中に適用する。目標アーキテクチャへ収束させる。使い捨ての互換コードで滑らかな中間状態を保とうとしない。 |
| [experience-first](./skills/principle-experience-first/SKILL.md) | コア | 実装の都合よりユーザの喜びを選ぶ。粗い機能を多く出すより、磨かれた機能を少なく出す。 |
| [exhaust-the-design-space](./skills/principle-exhaust-the-design-space/SKILL.md) | コア | コミットする前に、競合するプロトタイプを 2-3 個作り、並べて比較する。 |
| [build-the-lever](./skills/principle-build-the-lever/SKILL.md) | コア | 一括作業に限らず、自明でないあらゆる作業に適用する: 編集、移行、分析、チェック。手作業でやる代わりに、それを行うか、それを証明する道具 (codemod、スクリプト、ジェネレータ、あるいはサブエージェントが従う skill) を作る。その道具こそ、レビュワーが再実行できる成果物だ。 |
| [model-the-domain](./skills/principle-model-the-domain/SKILL.md) | アーキテクチャ | 散らばった条件分岐ではなく、構造にドメインを符号化する。 |
| [boundary-discipline](./skills/principle-boundary-discipline/SKILL.md) | アーキテクチャ | ガードをシステム境界 (CLI、設定、ネットワーク、外部 API) に集中させる。内部の型は信頼し、ビジネスロジックは純粋関数に保つ。 |
| [type-system-discipline](./skills/principle-type-system-discipline/SKILL.md) | アーキテクチャ | 不正な状態を表現不可能にする、意味を持つプリミティブに brand を付ける、外部データは境界でパースする、コンパイラに嘘をつくことを拒む、バリアントを網羅する、正となるスキーマから導出する。 |
| [make-operations-idempotent](./skills/principle-make-operations-idempotent/SKILL.md) | アーキテクチャ | 途中まで実行された過去の試行があっても、同じ最終状態へ収束する。 |
| [migrate-callers-then-delete-legacy-apis](./skills/principle-migrate-callers-then-delete-legacy-apis/SKILL.md) | アーキテクチャ | 互換レイヤを残すのではなく、呼び出し側の移行と古い API の削除を同じ波で行う。 |
| [separate-before-serializing-shared-state](./skills/principle-separate-before-serializing-shared-state/SKILL.md) | アーキテクチャ | まず共有そのものを無くす。共有された書き手が 1 つであることが本物の不変条件であるときにのみ、構造的に直列化する。 |
| [prove-it-works](./skills/principle-prove-it-works/SKILL.md) | 検証 | タスクを完了した後、done を宣言する前に適用する。代理物や自己申告や「コンパイルは通る」ではなく、実物 (機能を動かす、実際の値を読む、diff を検分する) に対して検証する。 |
| [fix-root-causes](./skills/principle-fix-root-causes/SKILL.md) | 検証 | 各症状を根本原因まで辿り、そこで直す。まず再現し、根本原因に届くまで why を問い、クラッシュを黙らせる nil チェックのガードに抗う。 |
| [sequence-verifiable-units](./skills/principle-sequence-verifiable-units/SKILL.md) | 検証 | 多段の作業 (一括処理、移行、似た編集の連続) と、コミットや PR の積み方に適用する。作業を、それぞれが検証可能な状態で終わる小さな単位に分割し、次へ進む前に各単位を確認し、その並び自体がレビュワーに対して自らを証明するように届ける順序を決める。 |
| [test-behavior-not-implementation](./skills/principle-test-behavior-not-implementation/SKILL.md) | 検証 | テストを書く、変更する、あるいは残すときに適用する。コードはそのユーザと同じ呼び方で呼び、観測される結果を具体的な期待値と照らして assert する。import した関数がすべて undefined を返しても通ってしまうテストなら、assert を書き直すかテストを消す。 |
| [guard-the-context-window](./skills/principle-guard-the-context-window/SKILL.md) | 委譲 | かさばるものはサブエージェントへ回す。メインスレッドには生のペイロードではなく要約を置く。 |
| [never-block-on-the-human](./skills/principle-never-block-on-the-human/SKILL.md) | 委譲 | 進め、結果を提示し、人間には事後に軌道修正させる。確認は不可逆な操作のために取っておく。 |
| [encode-lessons-in-structure](./skills/principle-encode-lessons-in-structure/SKILL.md) | メタ | ルールを、さらなる文章ではなく lint、メタデータフラグ、ランタイムチェック、スクリプトとして符号化する。 |

</details>

## ここに同梱していないもの

`poteto-mode` が参照するが同梱していないものがいくつかある:

- `/deslop` と `deslop` skill は `cursor-team-kit` プラグインに含まれる。
- `control-cli` (CLI と TUI 向け) と `control-ui` (ブラウザ、Electron、web 向け) も `cursor-team-kit` に含まれる。
- `/create-skill` は cursor の組み込みだ。cursor は組み込みの `/babysit` も同梱しているが、`poteto-mode` の中では、pr のステータスを問うリクエストについて [babysit playbook](./skills/poteto-mode/playbooks/babysit.md) がそれに優先する。

フルセットが欲しいなら、pstack と併せて `cursor-team-kit` をインストールする。

## なぜプランニングの skill が無いのか?

cursor には既に優れた plan モードがあり、pstack とよく噛み合う。だが個人的には、私はプランニングを信じていない。最良の仕様はコードだ。それでもプランを作りたいなら [`/poteto-mode`](./skills/poteto-mode/SKILL.md) がそれをカバーするが、既定ではない。

## 自分のものにする

`poteto-mode` は私のスタイルだ。まったく同じものが欲しいとは限らない。

[`/automate-me`](./skills/automate-me/SKILL.md) と打つ。最近のトランスクリプトを掘り、実際の働き方から `<your-name>-mode` skill を起こし、その下では pstack を経由してルーティングする。pstack を土台に保ったまま、`poteto-mode` と並ぶ自分専用のルーティング skill が手に入る。

モデルも設定できる。[`/setup-pstack`](./skills/setup-pstack/SKILL.md) と打つ。アクセスできるモデルを検出し、各ロール (コード、判断、レビューパネル) をモデルへ対応付ける、常に適用される小さなルールを書き出す。どの skill もそれを読み、ルールが無ければ妥当な既定値にフォールバックする。だから上書きしたいものだけを上書きすればよい。

0.15.3 より前に書かれたルールは、古い既定モデルを固定したままになる。該当するロールの行を削除するか、ファイルごと削除してから `/setup-pstack` を再実行する。再実行では、既定値と異なるモデルが設定されているロールはそのまま保たれる。

## 自動化

pstack は、休眠状態の [benny automation パック](./automations/benny/) も同梱している。benny は slack の課題報告をトリアージし、確認されたバグを実際の ui の証拠付きで再現・修正する。そのファイル群はスラッシュ skill としては登録されていない。

セットアップするには、cursor に [`FOR_AGENTS.md`](./automations/benny/FOR_AGENTS.md) を指し示す。セットアップはパックを対象リポジトリの `.cursor/automations/benny/` へコピーし、共有 skill のためにそこで pstack を有効にし、ユーザ設定はコピーしたパックの外に保つ。

## ライセンス

MIT
