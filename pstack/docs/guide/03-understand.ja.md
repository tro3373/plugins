# 変更する前にコードを理解する

理解していないコードを編集することが、微妙なリグレッションが世に出る経路だ。pstack は 4 つの入口を用意している。`/how` はコードが今何をしているかを説明する。`/why` はなぜその形になっているのかの理由を掘り起こす。`/teach` は両者を 1 つの説明に織り込む。`/recall` はあるトピックについての自分自身の直近のコンテキストを再構築する。

![A detective studies a machine blueprint with a magnifying glass while robots fetch case files; the evidence board behind her links clues under /how and /why.](./images/understanding.jpg)

## `/how` で挙動を辿る

```text
/how do we dedupe notifications? is there an n+1 when we look up subscribers?
```

実際に持っている疑問をそのまま聞く。[`/how`](../../skills/how/SKILL.md) はコードを読み、そのサブシステムにあなたをオンボードするシニアエンジニアのレベルで答える。ランタイムのフロー、主要な型、非自明な部分を含む。大きなサブシステムに対しては、まず読み取り専用のエクスプローラを 2 - 4 本ファンアウトさせる。狭い質問なら、ただ読んで説明するだけだ。

## `/why` で歴史を掘り起こす

```text
/why was the retry limit set to five? does the reason still hold?
```

[`/why`](../../skills/why/SKILL.md) は迷宮入り事件を追う探偵のように働く。ソース管理から始め、次にあなたの MCP が公開している証拠カテゴリ (課題トラッカー、長文ドキュメント、チームチャット、オブザーバビリティ、エラートラッキング、アナリティクスなど) をすべて並列に問い合わせる。レポートはすべてを引用し、直接的な証拠と推論を分け、記録が薄いときは「〜のように見える」と書く。ヌルの結果も報告される。「なぜかを誰も書き残さなかった」こと自体が 1 つの答えだからだ。

この 2 つは自然に組み合わさる。混乱の理由が歴史にあると疑うなら、`do why first then how` は十分によいプロンプトだ。

## `/teach` で本当に理解する

```text
/teach me how this PR changes retries. convince me it fixes the cause and not the symptom.
```

[`/teach`](../../skills/teach/SKILL.md) は、要約では足りないときのためのものだ。`/how` と `/why` を走らせ (小さな変更なら片方だけのこともある)、その発見を、図を 1 枚ずつ積み上げていく平易な説明へ織り込む。「convince me」という枠付けは盗む価値がある。説明を、見学ツアーではなく突つける論証に変えてくれる。

## `/recall` で自分のコンテキストを再構築する

```text
/recall catch me up on the export work from last week
```

[`/recall`](../../skills/recall/SKILL.md) は、自分自身の直近のチャットに加えて共有された記録 (課題、過去の修正、まだ発火しているエラー) を掘り、今どこまで進んでいて次に何をするかのブリーフを返す。冷えた状態でトピックに戻るときに使う。特定のチャットを 1 本再開したいなら、それは `/recall` ではなく下記の Session pickup プレイブックだ。

## Session pickup で先行作業を引き継ぐ

別のエージェント (あるいは先週の自分) がブランチを途中で置いていったとき:

```text
/poteto-mode take over this branch. read the decision log, figure out what's done, and continue from there. don't redo finished work.
```

[Session pickup プレイブック](../../skills/poteto-mode/playbooks/session-pickup.md) は、先行する作業の軌跡を正典として扱う。ブランチの状態と決定を再構築し、再開地点を名指しし、引き継いだ主張を元のゴールに照らして検証する。すべてをゼロから導き直すことはしない。

**落とし穴:** 「どうせエージェントはコードを読むのだから」と言ってこのページの skill を飛ばしてはならない。辿ったモデルを持たずに編集を始めるエージェントは、最初にもっともらしく見えた箇所で症状を直しがちだ。先に `/how` を走らせる方が、2 つ目のバグより安い。

次: [変更を設計する](./04-design.ja.md)。
