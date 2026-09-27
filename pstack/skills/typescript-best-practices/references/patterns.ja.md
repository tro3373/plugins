# TypeScript パターン

`SKILL.md` の各ルールに対応するコード例。背後にある原則は言語非依存である。**type-system-discipline** と **boundary-discipline** の principle skill を参照。

## Branded types

プリミティブにブランドを付けて、取り違えられないようにする。境界で一度だけ検証する。下流のコードは型を信頼する。

```ts
type AgentId = string & { readonly __brand: "AgentId" };

function parseAgentId(input: string): AgentId {
  if (!isUUID(input)) throw new Error(`Invalid agent id: ${input}`);
  return input as AgentId;
}

function focusAgent(id: AgentId): void {
  /* input is trusted */
}
```

`readonly __brand: 'X'` の形に合わせること。新しい規約を発明しない。

## Discriminated unions

リテラルの判別子でバリアントをモデル化する。すべてのバリアントが同じフィールド名を共有し、各バリアントの値は一意であるため、ありえない組み合わせは表現できない。

```ts
// Don't. Boolean + optionals lets contradictory states exist.
type DiffState = { loading: boolean; diff?: GitDiff; error?: string };

// Do. Only valid states exist.
type DiffState =
  | { kind: "loading" }
  | { kind: "ready"; diff: GitDiff }
  | { kind: "error"; error: string };
```

判別子の名前 (`kind`、`type`、`tag`) を 1 つ選び、それを貫く。

## 構成的モデリング

緩い型をランタイムチェックで制限するのではなく、すべてが合法な部品から型を組み立てる。

非空を可変長タプルで表す:

```ts
type NonEmpty<T> = [T, ...T[]];

// Don't: T[] plus a length check every caller must repeat
function pickWinner(entries: string[]): string {
  if (entries.length === 0) throw new Error("no entries");
  return entries[Math.floor(Math.random() * entries.length)];
}

// Do: an empty value of the type can't exist
function pickWinner(entries: NonEmpty<string>): string {
  return entries[Math.floor(Math.random() * entries.length)];
}
```

素の `T[]` が届く場所では、ガードで一度だけ絞り込む。その事実は以降、型に乗って運ばれる:

```ts
const isNonEmpty = <T>(arr: T[]): arr is NonEmpty<T> => arr.length > 0;
```

偶数長は、ペアとして表す:

```ts
type Pairs<T> = [T, T][];
```

時間範囲は、開始時刻と継続時間として表す:

```ts
// Don't: a comment holds the invariant
type TimeRange = { start: Date; end: Date }; // start <= end

// Do: a negative range can't be written; derive end when needed
type TimeRange = { start: Date; durationMs: number };
```

`durationMs` は素の number のままにする。生の number が duration を期待する場所に渡されうる場合にのみ (Branded types に従って) ブランドを付ける。反射的に付けない。悪い状態を構成不能にする表現を選び、その上に必要な読み方を載せる (`pairs.flat()`、`rangeEnd()` ヘルパ)。

## 最も単純な全域型

何でも強くすればよいわけではない。その上のすべての操作が全域なら `T[]` のままにする:

```ts
const sum = (xs: number[]) => xs.reduce((a, b) => a + b, 0); // [] is 0, fine
```

緩い型が使用箇所で嘘を強いるときに強くする。その兆候は `!`、`arr[0] as T`、そして "should never happen" の throw である:

```ts
// Don't: partiality smuggled past the compiler
function newestSession(sessions: Session[]): Session {
  return sessions.at(0)!;
}

// Do: strengthen the input; the assertion disappears
function newestSession(sessions: NonEmpty<Session>): Session {
  return sessions[0];
}
```

結果を `Session | undefined` に弱めるのが、もう 1 つの全域なシグネチャである。

## `any` より `unknown`

外部データは常に `unknown` である。使う前に絞り込む。

```ts
// Don't
function handle(input: any) {
  return input.foo.bar;
}

// Do
function handle(input: unknown) {
  if (typeof input === "object" && input !== null && "foo" in input) {
    // narrowed; compiler verifies access
  }
}
```

外部の情報源には、RPC ペイロード、`JSON.parse`、`postMessage`、IPC、ファイルの内容、環境変数、データベースの結果が含まれる。

## 手書きのガードよりスキーマ

外部データにプロパティごとの型ガードを書く前に、リポジトリのランタイムスキーマライブラリと既存のスキーマを探す。検証は 1 つのスキーマに所有させ、TypeScript の型はそこから導出する。スキーマと重複した interface とガードを別々に保守して、ずれさせてはならない。

```ts
import { z } from "zod";

const UserSchema = z.object({
  id: z.string().uuid(),
  role: z.enum(["admin", "member"]),
});

type User = z.infer<typeof UserSchema>;

function parseUser(input: unknown): User {
  return UserSchema.parse(input);
}
```

失敗が想定内の分岐である場合は `safeParse` を使う。リポジトリが別のスキーマライブラリを使っているなら、それに対応する推論ヘルパを使う。ガード 1 つのために新しいスキーマ依存を追加しない。このルールは、コードベースが既に信頼しているスキーマ機構を優先する。

## `as` キャストを使わない

すべての `as` は潜在的なランタイムクラッシュである。型システムがその主張を検証した後にのみキャストする。

```ts
// Don't
const user = data as User;

// Do. Earn the cast at the boundary.
function parseUser(data: unknown): User {
  if (typeof data !== "object" || data === null) {
    throw new Error("expected object");
  }
  if (!("id" in data) || typeof (data as Record<string, unknown>).id !== "string") {
    throw new Error("expected id");
  }
  // ... validate all fields
  return data as User; // OK, earned cast after full validation
}
```

既存コードから `as` をリファクタで取り除くときは、TypeScript が推論できない理由を特定する:

- 判別子が無い: 追加し、discriminated union に切り替える。
- ソースの型が広すぎる (例: `Record<string, unknown>`): 絞り込む。
- 型の無い境界: パース関数かスキーマを追加する。
- 本当に表現不能: branded type か `satisfies` を使う。

## 絞り込みの優先順位

良い順から最後の手段まで:

1. **Discriminated union の switch / if。** コンパイラが自動で絞り込む。
2. **`in` 演算子。** `"key" in obj` は、そのキーを含むバリアントに絞り込む。
3. **`typeof` / `instanceof`。** プリミティブとクラスインスタンス向け。
4. **ユーザ定義の型ガード。** 上記で足りないとき。
5. **`as` キャスト。** 検証の後にのみ。

```ts
function area(s: Shape): number {
  if ("radius" in s) return Math.PI * s.radius ** 2; // narrowed to circle
  return s.width * s.height; // narrowed to rect
}
```

## 型ガード

ガードは、その主張を実際に検証しなければならない。嘘をつくガードは `as` より悪い。

```ts
function isCircle(s: Shape): s is Shape & { kind: "circle" } {
  return s.kind === "circle";
}
```

可能なら判別子による絞り込みを優先する。

## 網羅性

default の分岐では、判別子を `never` 型のローカル変数に代入する。

```ts
// Value-returning switch
function area(s: Shape): number {
  switch (s.kind) {
    case "circle":
      return Math.PI * s.radius ** 2;
    case "rect":
      return s.width * s.height;
    default: {
      const _exhaustive: never = s;
      return _exhaustive;
    }
  }
}

// Void switch
function handle(s: Shape): void {
  switch (s.kind) {
    case "circle":
      drawCircle(s);
      break;
    case "rect":
      drawRect(s);
      break;
    default: {
      const _exhaustive: never = s;
      void _exhaustive;
    }
  }
}
```

値を返す switch では return スタイル、文としての switch では void スタイル。

## `as` より `satisfies`

`satisfies` はリテラル型を広げずに検証する。

```ts
// Don't. Widens, loses literal types.
const config = { theme: "dark", cols: 3 } as Config;

// Do. Validates AND preserves literal types.
const config = { theme: "dark", cols: 3 } satisfies Config;
// config.theme is "dark" (literal), not string
```

## 境界での検証

データが入ってくる場所で一度だけ検証する。内側では型を信頼する。**boundary-discipline** の principle skill を参照。

- **ワイヤフォーマット** (proto、JSON-RPC): `ignoreUnknownFields` でパースし、前方互換な変更が古いクライアントを壊さないようにする。
- **永続化された JSON:** バージョン付きの blob とし、パースを try/catch で囲む。
- **呼び出しチェーンの深いところで再検証しない。**

## スキーマから導出した型

`.proto`、OpenAPI 仕様、GraphQL スキーマ、データベースマイグレーションが既に形を定義しているなら、それを複製せず、生成された型から導出する。

```ts
// Don't. Duplicate shape, drifts when the schema changes.
type CheckSummary = {
  totalCount: number;
  checks: { name: string; status: string }[];
};
function renderChecks(s: CheckSummary) {
  /* ... */
}

// Do. Derive from the generated schema type.
import type { ChecksMessage } from "<generated module>";
function renderChecks(s: Pick<ChecksMessage, "totalCount" | "checks">) {
  /* ... */
}
```

新しい interface を書く前に、`Pick`、`Omit`、`Parameters`、`ReturnType`、`Awaited`、`typeof` に手を伸ばす。

## オブジェクト引数

```ts
// Don't. Swap two args, still compiles.
openFile(uri, {
  startLineNumber: 10,
  startColumn: 1,
  endLineNumber: 10,
  endColumn: 1,
});

// Do. Order-independent, self-documenting.
openFile({
  uri,
  selection: {
    startLineNumber: 10,
    startColumn: 1,
    endLineNumber: 10,
    endColumn: 1,
  },
});
```

ホットパスでは適用しない。フレームごとのレンダリング、トークナイザ、パーサ、アロケーションのコストが効くタイトなループの中。
