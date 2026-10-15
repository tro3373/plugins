---
name: how
description: "\"X はどう動くのか\"、何かを変更する前のコードウォークスルー、配置 / 所有 / レイヤリングの問い (\"これはどこに置くべきか\"、\"どのパッケージが所有するか\"、\"これは正しいレイヤか\") に使う。サブシステムのアーキテクチャ、ランタイムのフロー、オンボーディング用のメンタルモデルを説明する。動機については why を使う。"
disable-model-invocation: true
---

# How

「X はどう動くのか」という問いに答えるためにコードベースを探索する。サブシステムにオンボーディングするシニアエンジニアのレベルで、アーキテクチャの説明を作る。動くメンタルモデルを組み立てるのに十分なだけであり、注釈付きのソースコードのように読めるほどではない。

すべてのスポーンは `subagent_type: generalPurpose`、`readonly: true` の Task サブエージェントである。explorer は `pstack-models.mdc` ルールの `how explorer` 行 (既定 `grok-4.7-xhigh-fast`) を、explainer はすべて `how explainer` 行 (既定 `claude-opus-5-5-xhigh`) を使う。`model` にはその行の値を、ルールまたはその行が無ければ既定値を設定する。値が `auto` または `inherit-parent` のときは `model` を未設定のままにする。Task ツールがスラッグを拒否したら既定値を使い、その旨を述べる。既定値も拒否されたら、そのエラーメッセージにある同じファミリーの中で最も近い有効なスラッグを使う。各サブエージェントの reasoning level と fast / standard のスピードは、**poteto-mode** skill の Subagents セクションに従って仕事に合わせて選ぶ。

## 1. 複雑さを見積もる

スコープが曖昧なら、自分の解釈を述べたうえで探索する。ユーザが軌道修正すればよい。

- **単純** (単一モジュール、小さなユーティリティ、「how does function X work」のような狭い問い): explorer は使わない。探索と説明を 1 パスで行う explainer を 1 体スポーンする。プロンプトは `references/explainer-prompt.md` から、explorer の findings セクションを除いて組み立てる。手順 4 へ進む。
- **複雑** (複数のファイル / サービスにまたがるサブシステム、横断的な機能、アーキテクチャ全体の概観): 手順 2 へ進む。

迷ったら単純の側を取る。

## 2. 探索する (複雑な問いのみ)

問いを 2-4 本の切り口に分解する。それぞれをサブシステムの別々のスライスにする。すべての explorer を 1 つのメッセージでスポーンし、それぞれに `references/explorer-prompt.md` へ担当の切り口を書き込んだものを渡す。

## 3. 統合する (複雑な問いのみ)

すべての explorer が返ってきたら、`references/explainer-prompt.md` にすべての explorer の findings を書き込んで、explainer を 1 体スポーンする。

## 4. 提示する

explainer の出力を提示する。明快さのために軽く手を入れたり、会話からの文脈を足したりしてよいが、大幅に書き直してはならない。

## 出力形式

説明は `references/explainer-prompt.md` で定義されたセクションを使い、当てはまらないものは落とす: Overview、Key Concepts、How It Works、Where Things Live、Gotchas。
