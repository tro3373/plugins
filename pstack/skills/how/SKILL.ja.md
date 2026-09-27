---
name: how
description: "\"X はどう動くのか\"、何かを変更する前のコードウォークスルー、配置 / 所有 / レイヤリングの問い (\"これはどこに置くべきか\"、\"どのパッケージが所有するか\"、\"これは正しいレイヤか\") に使う。サブシステムのアーキテクチャ、ランタイムのフロー、オンボーディング用のメンタルモデルを説明する。動機については why を使う。"
disable-model-invocation: true
---

# How

「X はどう動くのか」という問いに答えるためにコードベースを探索する。サブシステムにオンボーディングするシニアエンジニアのレベルで、アーキテクチャの説明を作る。動くメンタルモデルを組み立てるのに十分なだけであり、注釈付きのソースコードのように読めるほどではない。

以下の各スポーンは、`pstack-models.mdc` ルール中のロール行と既定値を指す。`model` にはその行の値を、ルールまたはその行が無ければ既定値を設定する。値が `auto` または `inherit-parent` のときは `model` を未設定のままにする。Task ツールがスラッグを拒否したら既定値を使い、その旨を述べる。既定値も拒否されたら、そのエラーメッセージにある同じファミリーの中で最も近い有効なスラッグを使う。

## Step 1. 複雑さを見積もる

スコープが曖昧なら、自分の解釈を述べたうえで探索する。ユーザが軌道修正すればよい。

- **単純** (単一モジュール、小さなユーティリティ、「how does function X work」のような狭い問い): explorer は使わない。explainer が 1 パスで探索して説明する。Step 2b へ進む。
- **複雑** (複数のファイル / サービスにまたがるサブシステム、横断的な機能、アーキテクチャ全体の概観): まず並列の explorer をスポーンし、それから explainer へ引き渡す。Step 2a へ進む。

迷ったら単純の側を取る。

## Step 2a. 探索する (複雑な問いのみ)

問いを 2-4 本の探索の切り口に分解する。それぞれをサブシステムの別々のスライスにする。すべての explorer を 1 つのメッセージでスポーンする:

- `subagent_type`: `generalPurpose`
- `model`: `how explorer` 行、既定 `grok-4.7-xhigh-fast`
- `readonly`: `true`

各 explorer には `references/explorer-prompt.md` のプロンプトに、担当の切り口を書き込んで渡す。その後 Step 3 へ進む。

## Step 2b. 直接説明する (単純な問い)

探索と説明を 1 パスで行う Task サブエージェントを 1 体スポーンする:

- `subagent_type`: `generalPurpose`
- `model`: `how explainer` 行、既定 `claude-opus-5-5-max`
- `readonly`: `true`

`references/explainer-prompt.md` から、explorer の findings セクションを除いてプロンプトを組み立てる。Step 4 へ進む。

## Step 3. 統合する (複雑な問いのみ)

すべての explorer が返ってきたら、その findings を 1 つの説明へ統合する Task サブエージェントを 1 体スポーンする:

- `subagent_type`: `generalPurpose`
- `model`: `how explainer` 行、既定 `claude-opus-5-5-max`
- `readonly`: `true`

`references/explainer-prompt.md` から、すべての explorer の findings を書き込んでプロンプトを組み立てる。

## Step 4. 提示する

explainer の出力をユーザに提示する。明快さのために軽く手を入れたり、会話からの文脈を足したりしてよいが、大幅に書き直してはならない。

## 出力形式

説明は `references/explainer-prompt.md` で定義されたセクションを使い、当てはまらないものは落とす: Overview、Key Concepts、How It Works、Where Things Live、Gotchas。
