---
name: poteto-help
description: pstack のセットアップ、/poteto-mode、そしてタスクに合う skill、プレイブック、principle の選び方をユーザに案内する。/poteto-help に続けて質問を打つ。
disable-model-invocation: true
---

# Poteto help

pstack についてのユーザの質問に答え、ユーザが送れるプロンプトを渡し、答えの出どころのファイルをリンクする。ヘルプの質問に対しては作業を始めない。ユーザが聞いたのはやり方であり、pstack の実行は本物のトークンを消費するので、プロンプトを送るのはユーザに任せる。

"use pstack to fix this bug" のように作業を依頼するメッセージは、ヘルプの質問ではない。[`poteto-mode`](../poteto-mode/SKILL.md) を読み、その下で作業を行い、Custom Mode にすればそれが有効なまま保たれることを一度だけ伝える。

このファイルは、質問を、答えを持つ skill とガイドページに対応付ける。詳細はそれらのファイルが所有する。引用する前に振り分け先のファイルを読み、それがこの対応表と食い違うときはそのファイルを信じる。ここのリンクはインストールされたプラグインの中を指しており、ユーザはそれを開けないかもしれない。だからユーザには公開版のファイルを渡す: `https://github.com/cursor/plugins/blob/main/pstack/` の後にそのパスを続けたもの。

## 何が必要かを見極める

必要なことは、メッセージと会話から推測する。"which skill reviews a PR?" のように状況が名指しされていれば、そのセクションへ直行する。それでも不明なら、次の選択肢で多肢選択の質問を 1 つ聞き、選ばれたセクションだけに答える:

- セットアップする
- `/poteto-mode` でタスクを始める
- 状況に合う skill を選ぶ
- うまくいかなかった実行を直す
- pstack を自分のものにする

答えを変える状態を確認し、答えが変わるときだけそれに触れる:

- `~/.cursor/rules/pstack-models.mdc` が無いなら、このユーザについて `/setup-pstack` はまだ走っておらず、すべてのロールがデフォルトのモデルを使う。
- プロジェクトに `verify-*` skill もその他のアプリ用ハーネスも無いなら、エージェントにはアプリを操作するスクリプト化された手段が無い。変更が動くことを証明する方法についての質問なら `/create-verification-skill` に触れる。

モデルのルールが無く、それが効いてくる場合は、ユーザが今ロールごとのモデルと reasoning budget を選びたいかを聞く。効いてくるのは、ユーザが初めてのとき、質問がセットアップかコストについてのとき、答えがどのモデルが走るかに依存するときだ。聞くのはチャットあたり最大 1 回。必要なことも不明なら、両方の質問をまとめて聞く。選択肢は 2 つ:

- 今: 打つための `/setup-pstack` を渡し、質問にも答える。
- 後で: 質問に答え、`/setup-pstack` を走らせるまで各ロールはデフォルトのモデルのままだと 1 行添える。

## セットアップする

1. チャットで `/add-plugin pstack`、またはサイドバーの Customize からインストールする。
2. [`/setup-pstack`](../setup-pstack/SKILL.md) を走らせる。reasoning budget を尋ね、各ロールにモデルを割り当て、ルールを書く。ルールは新しいチャットに適用される。
3. `/poteto-mode` と、ゴールと、pass か fail かになるチェックで本物のタスクを始める。

インストールしただけでは、ユーザが skill を呼び出すまで何も変わらない。ユーザの言葉から読み込まれるのは `/setup-pstack` だけだ。詳細は [README](../../README.md) と [ガイドページ 1](../../docs/guide/01-setup.md) にある。[`references/prompting.md`](references/prompting.md) に従い、最初のプロンプトを一緒に言葉にすることを申し出る。

コストが心配なら、トークンがどこに使われ、どうすれば減らせるかを伝える。pstack はサブエージェントとレビューパネルに余分なトークンを使う。`/setup-pstack` を再実行し、小さい budget か安いモデルを選ぶ。`auto` か `inherit-parent` に設定したロールはチャットのモデルで走るので、チャットが Auto か安いモデルで走っているならトークンの節約になる。パネルのリストを短くすると、エントリ 1 つにつき 1 つのサブエージェントなので、走るサブエージェントが減る。`/poteto-mode` は厳密さが要る作業のために取っておく。

pstack は Cursor 向けに作られている。skill は Agent Skills 形式を使うので、他のツールでも読める。しかし `/poteto-mode`、`/how`、`/why`、`/teach` を含むほとんどのワークフロー skill は、ロールごとのモデルで Cursor のサブエージェントを起動する。また Custom Modes と `/loop` は Cursor の機能なので、それらの部分は他のツールでは動かないかもしれない。

## `/poteto-mode` でタスクを始める

`/poteto-mode` はタスクをプレイブックに対応付け、プレイブックの手順を todo リストにコピーし、手順が必要とするときに他の skill を走らせる。飛ばした手順は `skip: <reason>` としてリストに残る。良いプロンプトはゴールと完了の見分け方を述べる。skill は列挙しない。手で書いた順序は、プレイブックなら保つ手順を落としたり並べ替えたりしがちだからだ。プロンプトの言い方を手伝う前に [`references/prompting.md`](references/prompting.md) を読む。例は [ガイドページ 2](../../docs/guide/02-poteto-mode.md) にある。

`/poteto-mode` が有効なまま保たれるかは、ユーザがどう始めるかで決まる:

- `/poteto-mode` で Enter を押すと、skill は 1 つのメッセージに添付される。チャットが進むにつれて薄れていく。
- Mac では Option+Enter、Windows では Alt+Enter、または skill のエントリから Use as Mode を選ぶと、Custom Mode になる。ユーザがモードを抜けるまで毎ターン文脈に残り、雑談のターンには入らない。
- Cursor のドキュメントでは、Custom Modes は Agents Window と CLI にあると書かれている。それ以外の場所では、新しいタスクのたびに `/poteto-mode` で始める。

この話題が出たら [Cursor の skills ドキュメント](https://cursor.com/docs/skills) をリンクする。チャットの途中で "new task" と言うと、モードは新しいプレイブックに対応付け直す。`/poteto-mode` は、プレイブックの手順が起動するサブエージェントに既に `poteto-agent` を使っている。自分で起動するサブエージェントで同じスタイルを得るには、`subagent_type: "poteto-agent"` で起動する。

## skill を選ぶ

デフォルトの答えは `/poteto-mode` だ。手順が必要とするときに他のほとんどを走らせる。ユーザがプレイブックの与えるものより多く、または少なく何かを望むときは、skill を直接名指しする。勧める前にその skill を読み、プロンプト例を 1 つ渡す。

| ユーザがしたいこと | Skill |
|---|---|
| 自明でないタスクを厳密にこなす | [`/poteto-mode`](../poteto-mode/SKILL.md) |
| 着手前に、エージェントが自分のゴールと問題を理解したかを確かめる | [`/in-your-own-words`](../in-your-own-words/SKILL.md) |
| 着手前にタスクをより高く狙う: より専門的に、成果志向で、過剰に仕様化しない | [`/sharpen`](../sharpen/SKILL.md) |
| コードが今どう動くか、新しいコードをどこに置くべきかを知る | [`/how`](../how/SKILL.md) |
| コードがなぜこの形なのか、数値がどこから来たのかを知る | [`/why`](../why/SKILL.md) |
| 変更やサブシステムを平易な説明で理解する | [`/teach`](../teach/SKILL.md) |
| あるトピックについての自分の最近の作業に追いつく | [`/recall`](../recall/SKILL.md) |
| このチャットでの前回のメッセージ以降の変化に、短い平易な文と小さな図で追いつく。各アクション項目を決めるのに足る文脈つき | [`/ketchup`](../ketchup/SKILL.md) |
| 小さな diff がその外側で何を壊し得るかを知る | [`/blast-radius`](../blast-radius/SKILL.md) |
| 関数の境界をまたぐコードの前に、型とモジュールの形を決める | [`/architect`](../architect/SKILL.md) |
| 1 つのブリーフに対する複数の試みを得て、最良のものに統合する | [`/arena`](../arena/SKILL.md) |
| スライスに対する並列チェックやワーカーの競争を、クラウドエージェントとして走らせる | [`/swarm`](../swarm/SKILL.md) |
| 異なるモデルに diff をレビューさせ、壊しにかからせる | [`/interrogate`](../interrogate/SKILL.md) |
| 安いローカルテストがあるとき、バグをテストファーストで直す | [`/tdd`](../tdd/SKILL.md) |
| `.ts` や `.tsx` の作業に TypeScript のルールを適用する | [`/typescript-best-practices`](../typescript-best-practices/SKILL.md) |
| レビューの前に、それを書いていないレビュワーを使ってコメントを取り除く | [`/no-comments`](../no-comments/SKILL.md) |
| 文章から AI っぽさを取り除く | [`/unslop`](../unslop/SKILL.md) |
| ドキュメント、RFC、README、PR の説明、コミットメッセージを基準どおりに書く | [`/technical-writing`](../technical-writing/SKILL.md) |
| 直前の返答を平易な言葉でもう一度聞く | [`/bro`](../bro/SKILL.md) |
| エージェントにアプリを操作して振る舞いを証明するスクリプト化された手段を与える | [`/create-verification-skill`](../create-verification-skill/SKILL.md) |
| verification skill とその feature map をアプリに合わせ直す | [`/maintain-verification-skill`](../maintain-verification-skill/SKILL.md) |
| 性能の数値を、報告したりそれに基づいて動いたりする前に検分する | [`/benchmark-checklist`](../benchmark-checklist/SKILL.md) |
| 大きな変更や横断的な変更、または離席後にレビューする変更を走らせる | [`/figure-it-out`](../figure-it-out/SKILL.md) |
| 実行中に決定ログを残し、後でレビューする | [`/show-me-your-work`](../show-me-your-work/SKILL.md) |
| ロールごとのモデルと reasoning budget を選ぶ | [`/setup-pstack`](../setup-pstack/SKILL.md) |
| 自分の作業習慣を個人用のモード skill にする | [`/automate-me`](../automate-me/SKILL.md) |
| 終わったタスクから学んだことを skill の編集にする | [`/reflect`](../reflect/SKILL.md) |
| この repo でエージェントが同じ間違いを繰り返すのを止める | [`/correct`](../correct/SKILL.md) |
| ボタンが webhook 経由で Grok Bot を起こすページを作る | [`/make-bot-ui`](../make-bot-ui/SKILL.md) |
| pstack の中で道を見つける | `/poteto-help` |

このディレクトリの隣にある skill ディレクトリが表に無いなら、その frontmatter を読み、description で振り分ける。`principle-*` ディレクトリは下の principle の項で扱う。

際どい選択:

- `/how` はコードが何をするかを説明する。`/why` は理由を説明する。`/teach` はその片方か両方を走らせ、結果を平易に説明する。
- `/arena` はすべてのワーカーに同じブリーフを与え、最良の部分を統合する。`/swarm` は作業をスライスか競争に分け、レポートを 1 つ返す。
- `/architect` は設計を決めた直後に実装する。コードを書く前に設計をレビューするには "with checkpoint" を添える。
- `/interrogate` は diff をレビューする。`/blast-radius` は diff の外側の破損を探し、変更を安全にする 1 つの事実を証明する。
- `/recall` は最近のチャットをまたいで文脈を組み直す。`/ketchup` はユーザの前回のメッセージ以降のこのチャットだけを扱う。特定の 1 つのチャットやブランチを再開するのは Session pickup プレイブックだ。
- `/figure-it-out` は 1 回の厳密な実行を設計する。Orchestrate プレイブックは何日もかかり多くの PR にまたがるプログラムを走らせる。Autonomous run プレイブックは 1 つのタスクを完了条件まで駆動する。

pstack に無いもの:

- `/deslop`、`control-cli`、`control-ui` は `cursor-team-kit` プラグインに入っている。
- `/loop` と `/create-skill` は Cursor の組み込みだ。
- pstack に `/orchestrate` skill は無い。Orchestrate は `/poteto-mode` のプレイブックである。スラッシュメニューに `/orchestrate` が出るなら、別のプラグインが提供している。

## プレイブックと principle

プレイブックは `/poteto-mode` の中の手順リストで、skill ではないのでスラッシュコマンドは無い。`/poteto-mode` の中では、タスクを説明すればプレイブックが選ばれ、次の言い回しは直接プレイブックを名指しする:

- "babysit this pr" や "check on pr 123" は Babysit を走らせる。PR を merge-ready まで持っていき、そこで止まる。ユーザが merge、land、ship を頼まない限りマージはしない。
- "land the stack" は Shipping を走らせる。
- "take over this branch" は Session pickup を走らせる。
- "pause safely" は Pause safely を走らせる。
- "full autopilot on this queue" は Autopilot-full を走らせる。"stack them, don't ship" は Autopilot-stack を走らせる。
- "run the eval playbook" は Eval を走らせる。

`/poteto-mode` 無しだと、"babysit this pr" のような言い回しは、同じ仕事をする Cursor 自身の skill を代わりに起動し得る。[`poteto-mode`](../poteto-mode/SKILL.md) の Playbooks セクションに、すべてのプレイブックとそれが適用される場面が載っている。[ガイドページ 6](../../docs/guide/06-verify-and-ship.md) は PR を開く、babysit する、着地させることを扱う。

pstack に計画用の skill は無い。Cursor の Plan Mode が並行して使える。フェーズや積み重なった PR にまたがる作業では、`/poteto-mode` に計画を頼むと [Multi-phase plan プレイブック](../poteto-mode/playbooks/multi-phase-plan.md) が走り、計画を書いて実装はしない。設計の問いには、Prototype プレイブックか `/architect` がまずコードで決着をつける。

principle は、`/poteto-mode` が読み、返答の中で引用する 1 ルールの skill だ。ユーザが呼び出すことはまれで、代わりに "apply prove it works. show me the real output." のように名前で舵を取る。`/principle-<name>` と打てば、オンデマンドで読み込める。[ガイドページ 8](../../docs/guide/08-principles.md) に一覧がある。

## うまくいかなかった実行を直す

| 症状 | 直し方 |
|---|---|
| 数ターンでモードが効かなくなった | Enter で始めている。Custom Mode として始めるか、タスクごとに `/poteto-mode` で始める。 |
| 質問が、前のタスクの次の手順として扱われた | "new task" と言うか、そのターンにモードは要らないと言う。 |
| 新しいモデル選択が効かなかった | `/setup-pstack` のルールは新しいチャットに適用される。新しいチャットを始める。 |
| 実行のコストが想定より高い | セットアップするの項のコストの段落を見る。 |
| skill が勝手に読み込まれなかった | ユーザの言葉から読み込まれるのは `/setup-pstack` だけだ。他はユーザが打ったときか `/poteto-mode` が走らせたときに読み込まれ、`/poteto-mode` はすべての skill を走らせるわけではない。 |
| 並列のエージェントが互いを上書きした | エージェントごとに専用の worktree を与えるか、それぞれ専用のマシンを得るクラウドエージェントとして走らせる。 |
| 夜通しの実行が動いたが何も終わらなかった | `/loop` には期間ではなく、pass か fail かになるチェックが要る。[ガイドページ 7](../../docs/guide/07-overnight.md) を見る。 |
| 返答がビルドがグリーンなことを根拠に成功を主張する | 実際のコマンド、フロー、保存された値、プロファイルを求める。それが prove-it-works principle だ。 |

ずれていく実行には、[`references/prompting.md`](references/prompting.md) に 1 行の舵取りがある。[ガイドページ 10](../../docs/guide/10-recipes-and-pitfalls.md) に、さらに多くの落とし穴とコピーする価値のあるレシピがある。

## pstack を自分のものにする

- [`/automate-me`](../automate-me/SKILL.md) は、ユーザ自身の履歴から個人用のモード skill の草稿を作る。`/poteto-mode` と並べて使う。
- セッション後の [`/reflect`](../reflect/SKILL.md) は、その教訓をユーザが承認する skill の編集に変える。
- `/poteto-mode write a skill for <workflow>` は authoring プレイブックを走らせる。eval プレイブックは skill の変更をブラインドでテストする。
- 振る舞いのおかしい skill は、それがうまくいかなかった機能の作業の中ではなく、専用の PR で直す。

[ガイドページ 9](../../docs/guide/09-make-it-yours.md) がそれぞれを扱う。

## 返答

答えを先に述べる。プロンプト例はコードブロックで最大 1 つ。合うものがあれば [`references/recipes.md`](references/recipes.md) から改変し、そのファイルへのリンクを続ける。ユーザが対応表全体を求めていない限り短く保つ。
