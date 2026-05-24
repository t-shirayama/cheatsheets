# TypeScript 基本チートシート

確認日: 2026-05-24

JavaScript に静的型を追加する TypeScript の基本構文です。

## このページで扱う範囲

- TypeScript の基本型と型定義
- Union、Generics、Utility Types の最小例
- strict 前提の初歩的な注意点

## 扱わない範囲

- 高度な型レベルプログラミング
- フレームワーク固有の型設計
- バージョン別の細かいコンパイラ挙動

## 基本型

値に型を付けることで、代入ミスや呼び出しミスを早めに検出できます。

```ts
const name: string = "Alice";
const age: number = 30;
const enabled: boolean = true;
const tags: string[] = ["admin", "editor"];
const point: [number, number] = [10, 20];
```

## オブジェクト型

オブジェクトの形は `type` で定義し、任意項目には `?` を付けます。

```ts
type User = {
  id: number;
  email: string;
  // name は省略可能です。
  name?: string;
};

const user: User = {
  id: 1,
  email: "alice@example.com",
};
```

## interface と type

クラスや外部実装との契約には `interface`、合成しやすい型には `type` が便利です。

```ts
interface Repository {
  findById(id: number): Promise<User | null>;
}

// 成功と失敗を Union で表します。
type ApiResult<T> =
  | { ok: true; data: T }
  | { ok: false; error: string };
```

- `interface` はオブジェクトやクラス契約に向いています。
- `type` は Union、Intersection、Utility Types と相性がよいです。

## Union と絞り込み

文字列リテラル Union を使うと、許可する値を限定できます。

```ts
type Status = "draft" | "published" | "archived";

function label(status: Status): string {
  switch (status) {
    case "draft":
      return "下書き";
    case "published":
      return "公開";
    case "archived":
      return "保管";
  }
}
```

`typeof` などで型を絞り込むと、その分岐内で安全にメソッドを呼べます。

```ts
function printId(id: string | number) {
  if (typeof id === "string") {
    console.log(id.toUpperCase());
    return;
  }

  console.log(id.toFixed(0));
}
```

## Generics

Generics は、呼び出し側の型を保ったまま再利用できる関数や型を作る仕組みです。

```ts
function first<T>(items: T[]): T | undefined {
  // 空配列なら undefined を返します。
  return items[0];
}

type Page<T> = {
  items: T[];
  total: number;
};
```

## Utility Types

標準の Utility Types を使うと、既存の型から派生型を作れます。

```ts
type UserDraft = Partial<User>;
type UserRequired = Required<User>;
type UserSummary = Pick<User, "id" | "email">;
type UserWithoutEmail = Omit<User, "email">;
type UserMap = Record<number, User>;
```

## satisfies

`satisfies` は値の具体的な型を保ちながら、指定した型を満たすか確認できます。

```ts
const routes = {
  home: "/",
  users: "/users",
} satisfies Record<string, string>;
```

## 非同期関数

非同期関数の戻り値は `Promise<T>` として表します。外部 API の値は、型アサーションだけに頼らず実行時チェックを挟むと安全です。

```ts
function isUser(value: unknown): value is User {
  if (typeof value !== "object" || value === null) {
    return false;
  }

  const candidate = value as Record<string, unknown>;
  return typeof candidate.id === "number" && typeof candidate.email === "string";
}

async function fetchUser(id: number): Promise<User> {
  const response = await fetch(`/api/users/${id}`);

  if (!response.ok) {
    throw new Error(`Request failed: ${response.status}`);
  }

  const data: unknown = await response.json();

  if (!isUser(data)) {
    throw new Error("Invalid user response");
  }

  return data;
}
```

## tsconfig の最小例

型チェックを厳しめに始めるための最小設定例です。

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "strict": true,
    "moduleResolution": "Bundler",
    "noUncheckedIndexedAccess": true,
    "skipLibCheck": true
  }
}
```

## 注意点

- 新規プロジェクトでは `strict: true` を基本にします。
- `any` は境界部分に閉じ込め、可能なら `unknown` から絞り込みます。
- `as` による型アサーションは実行時チェックではありません。
- API レスポンスは TypeScript 型だけで安全にならないため、必要に応じてバリデーションします。

## 参考

- 公式ドキュメント: [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- 関連: [TSConfig Reference](https://www.typescriptlang.org/tsconfig/)
