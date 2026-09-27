---
name: typescript-best-practices
description: TypeScript のベストプラクティス。.ts または .tsx ファイルを読む・編集するときに使う。
paths: ["**/*.ts", "**/*.tsx"]
disable-model-invocation: true
---

# TypeScript best practices

先に **type-system-discipline** の principle skill を適用する。

| ルール | 要約 |
|------|---------|
| 判別可能な union | `kind` のリテラル判別子でバリアントをモデル化し、ありえない状態を表現できないようにする。オプショナルフィールドの寄せ集めにしない。 |
| ブランド型 | プリミティブを `& { readonly __brand: "X" }` でブランド化し、取り違えられないようにする。検証は境界で一度だけ。 |
| 構成的モデリング | 不正な値を構築できない形にする。非空なら `[T, ...T[]]`、偶数長なら `[T, T][]`、範囲なら `start` と `duration`。ランタイムのガードでもなければ、refinement type への願望でもない。 |
| 最も単純で全域な型 | その型に対する全操作が全域であるうちは `T[]` のままにする。緩い型のせいで `!`、キャスト、「起きないはず」の throw が必要になる箇所でだけ `NonEmpty<T>` に強める。 |
| `any` より `unknown` | 外部データは `unknown` にする。 |
| ガードよりスキーマ | プロパティを 1 つずつ検査する type guard を手書きする前に、リポジトリのランタイムスキーマライブラリを使い、`z.infer` などでスキーマから型を導出する。 |
| `as` キャスト禁止 | `as` はすべて、いつか起きるランタイムクラッシュだ。キャストは検証の後にだけ行う。 |
| 絞り込みの優先順位 | 判別子の switch > `in` 演算子 > `typeof`/`instanceof` > ユーザ定義の type guard > `as`。 |
| Type guard | 主張の内容を必ず検証すること。嘘をつくガードは `as` より悪い。安全だと名乗る名前の裏にバグが隠れるからだ。名前は `isX` か `hasX` にする。 |
| 網羅性 | default 節に `const _exhaustive: never = x;` をインラインで置き、新しいバリアントが追加されたらコンパイラがエラーを出すようにする。 |
| `as` より `satisfies` | リテラル型を広げずに値を検証できる。 |
| 境界での validation | データが入ってくる場所でパースし、名前付きのドメイン型にする。`Record<string, unknown>` (どんな綴りであれ) はそのパースで止める。内側では型を信頼する。**boundary-discipline** の principle skill を参照。 |
| スキーマ由来の型 | 新しい interface を宣言する前に、`Pick`/`Omit`/`Parameters`/`ReturnType`/`Awaited`/`typeof` に手を伸ばす。 |
| オブジェクト引数 | 位置引数ではなくオブジェクトを渡し、引数の順序が自明になるようにする。ホットパス (毎フレームの描画、トークナイザ、パーサ) では見送る。 |
| 本物のテスト | 実行できるものをモックしない。フレームワークが用意する本物のテストプリミティブを leak/disposable チェック付きで使い、UI は動作中のビルドで検証する。モックはローカルで実行できないものにだけ使う。 |
| 構造化テレメトリ | id から追跡してデバッグできるだけの文脈を持つ、構造化ロガーによる診断を優先する。出荷するコードに `console.log` を残さない。 |

例: `references/patterns.md`。
