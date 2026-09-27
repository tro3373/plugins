# 原則の名前で舵を取る

pstack は 23 の原則を個別の skill として同梱している。`/poteto-mode` は多段タスクの開始時に必ずその索引を読み、タスクが発火させた原則を適用し、適用した各原則の名前と、それが変えた決定を返答で名指しする。

原則は呼び出すものではない。名前を使って舵を取るものである。各名前は、エージェントが既に読んだ完全なルールを指しているので、1 つのフレーズが段落 1 つ分の指示より正確に作業の向きを変える。

## 実際の舵取り

エージェントが既存の 3 つのアダプタに新しいアダプタを継ぎ足そうとしているとする:

```text
use subtract before you add. delete the obsolete adapters first, then design what's left.
```

ビルドが通ったことを理由に成功を主張しているとする:

```text
apply prove it works. run the real import flow and show me the written records.
```

2 つの並行した試行が同じブランチに書き込もうとしているとする:

```text
separate before serializing shared state. give each attempt its own worktree, no locks.
```

どのフレーズも効くのは、その背後にあるルールが具体的だからである。エージェントは依然として、そのルールがどの決定を変えたかを返答で述べねばならない。決定を伴わない原則の引用は、適用ではなく名前を出しただけだという兆候である。

## 23 の原則、手短に

コアの原則は、どこまで作るか、いつ設計を考え直すかを決める:

- [Laziness Protocol](../../skills/principle-laziness-protocol/SKILL.md) は削除と、問題を解く最小の変更を選ぶ。
- [Foundational Thinking](../../skills/principle-foundational-thinking/SKILL.md) はロジックを書く前にコアのデータ構造を選ぶ。
- [Redesign from First Principles](../../skills/principle-redesign-from-first-principles/SKILL.md) は新しい要件を、初日からそこにあったかのように統合する。
- [Attack the Premise](../../skills/principle-attack-the-premise/SKILL.md) は、失敗した修正を 2 つ以上並べたときに共通していた前提を、どのアクターが不均衡を抱えているかの棚卸しの後に問い直す。
- [Subtract Before You Add](../../skills/principle-subtract-before-you-add/SKILL.md) は、その上に作り始める前に不要なものを取り除く。
- [Minimize Reader Load](../../skills/principle-minimize-reader-load/SKILL.md) は、読み手が頭に抱えねばならない層と隠れた状態を畳む。
- [Outcome-Oriented Execution](../../skills/principle-outcome-oriented-execution/SKILL.md) は、使い捨ての互換状態を保つ代わりに、書き直しをターゲットの設計へ収束させる。
- [Experience First](../../skills/principle-experience-first/SKILL.md) は、実装の都合よりユーザにとっての結果を選ぶ。
- [Exhaust the Design Space](../../skills/principle-exhaust-the-design-space/SKILL.md) は、前例がないときに競合する試作を 2, 3 個作る。
- [Build the Lever](../../skills/principle-build-the-lever/SKILL.md) は、その作業をこなすか証明するスクリプトを作り、レビュワーが再実行できるようにする。

アーキテクチャの原則は、状態・バリデーション・互換性をどこに置くかを決める:

- [Model the Domain](../../skills/principle-model-the-domain/SKILL.md) は、繰り返されるルールを散らばった条件分岐ではなく 1 つの構造に表現する。
- [Boundary Discipline](../../skills/principle-boundary-discipline/SKILL.md) は境界で検証し、内部の型は信頼する。
- [Type System Discipline](../../skills/principle-type-system-discipline/SKILL.md) は不正な状態を表現不能にする。
- [Make Operations Idempotent](../../skills/principle-make-operations-idempotent/SKILL.md) はリトライを同じ最終状態に収束させる。
- [Migrate Callers Then Delete Legacy APIs](../../skills/principle-migrate-callers-then-delete-legacy-apis/SKILL.md) は移行と削除を 1 波でやる。
- [Separate Before Serializing Shared State](../../skills/principle-separate-before-serializing-shared-state/SKILL.md) は、調整を加える前に共有そのものを取り除く。

検証の原則は、何を証明とみなすかを定める:

- [Prove It Works](../../skills/principle-prove-it-works/SKILL.md) は代用物ではなく本物の成果物を検証する。
- [Fix Root Causes](../../skills/principle-fix-root-causes/SKILL.md) は、コードを変える前に再現し原因まで辿る。
- [Sequence Work into Verifiable Units](../../skills/principle-sequence-verifiable-units/SKILL.md) は、次を始める前に小さな各単位をチェックで終える。
- [Test Behavior, Not Implementation](../../skills/principle-test-behavior-not-implementation/SKILL.md) は、コードをそのユーザと同じ方法で呼び出し、リテラルな期待値をアサートし、インポートした全ての関数が `undefined` を返しても通ってしまうテストを削除する。

委譲の原則は、並行作業を正気に保つ:

- [Guard the Context Window](../../skills/principle-guard-the-context-window/SKILL.md) は大量の読み取りをサブエージェントへ回し、知見をメインのチャットに残す。
- [Never Block on the Human](../../skills/principle-never-block-on-the-human/SKILL.md) は可逆な作業は進め、結果を提示する。

そしてメタ原則が 1 つ:

- [Encode Lessons in Structure](../../skills/principle-encode-lessons-in-structure/SKILL.md) は、2 度繰り返したアドバイスを lint、チェック、スクリプトに変える。

このリストを暗記しなくてよい。今はざっと眺めておき、ここにある名前なら防げたはずのことをエージェントがやっているのを見つけたときに戻ってくればよい。語彙はそうやって身に付く。

次: [自分のものにする](./09-make-it-yours.ja.md)。
