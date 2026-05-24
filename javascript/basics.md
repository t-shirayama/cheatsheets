# JavaScript 基本チートシート

確認日: 2026-05-24

ブラウザと Node.js の両方でよく使う JavaScript の基本構文です。

## このページで扱う範囲

- JavaScript の基本構文
- ブラウザと Node.js の両方でよく使う最小例
- 初歩的な非同期処理、モジュール、npm コマンド

## 扱わない範囲

- フレームワーク固有の書き方
- ビルドツールやトランスパイラの詳細設定
- パフォーマンスチューニングやブラウザ互換性の細かい差分

## 変数と値

再代入の有無に応じて `const` と `let` を使い分けます。

```js
const name = "Alice";
let count = 0;

// オブジェクトと配列はよく使うデータ構造です。
const user = { id: 1, name: "Alice" };
const items = ["apple", "banana"];
```

- 再代入しない値は `const` を使います。
- `var` は関数スコープのため、新規コードでは原則使いません。

## 関数

関数宣言、アロー関数、デフォルト引数の基本形です。

```js
function add(a, b) {
  return a + b;
}

const multiply = (a, b) => a * b;

const greet = (name = "guest") => `Hello, ${name}`;
```

## 配列

配列メソッドは、変換、絞り込み、集計、検索でよく使います。

```js
const numbers = [1, 2, 3, 4];

numbers.map((n) => n * 2);
numbers.filter((n) => n % 2 === 0);
// reduce は配列を1つの値に畳み込みます。
numbers.reduce((sum, n) => sum + n, 0);
numbers.find((n) => n > 2);
numbers.includes(3);
```

## オブジェクト

分割代入とスプレッド構文を使うと、必要な値の取り出しやコピーが簡潔になります。

```js
const user = { id: 1, name: "Alice", role: "admin" };
const { id, name } = user;

const nextUser = {
  // 既存の値をコピーして、一部だけ上書きします。
  ...user,
  name: "Alice Example",
};

Object.keys(user);
Object.values(user);
Object.entries(user);
```

## 条件分岐

複数行の分岐は `if`、短い値の出し分けは三項演算子が便利です。

```js
if (user.role === "admin") {
  console.log("allowed");
} else {
  console.log("denied");
}

const label = count > 0 ? "active" : "empty";
```

## Optional Chaining / Nullish Coalescing

深いプロパティを安全に読むときは `?.`、`null` または `undefined` のときだけ代替値を使うなら `??` を使います。

```js
const user = { profile: { displayName: "Alice" } };

// profile が存在しない場合も例外にせず undefined を返します。
const displayName = user.profile?.displayName;
const label = displayName ?? "Guest";
```

## クラス

状態と振る舞いをまとめたいときは `class` を使います。

```js
class Counter {
  constructor(initialValue = 0) {
    this.value = initialValue;
  }

  increment() {
    // インスタンスの状態を 1 増やします。
    this.value += 1;
    return this.value;
  }
}
```

## 非同期処理

`async` / `await` は Promise を返す処理を同期的な見た目で書けます。

```js
const fetchUser = async (id) => {
  const response = await fetch(`/api/users/${id}`);

  // HTTP エラーは fetch 自体では例外にならないため確認します。
  if (!response.ok) {
    throw new Error(`Request failed: ${response.status}`);
  }

  return response.json();
};
```

複数の非同期処理を並列実行したい場合は `Promise.all` を使います。

```js
Promise.all([fetchUser(1), fetchUser(2)])
  .then(([first, second]) => {
    console.log(first, second);
  })
  .catch((error) => {
    console.error(error);
  });
```

## モジュール

値や関数を `export` し、別ファイルから `import` して使います。

```js
export const formatDate = (date) => date.toISOString().slice(0, 10);
export default function App() {}
```

```js
import App, { formatDate } from "./app.js";
```

## 例外処理

失敗する可能性のある処理は `try` / `catch` で扱います。

```js
try {
  await fetchUser(1);
} catch (error) {
  console.error(error);
} finally {
  console.log("done");
}
```

## よく使う npm コマンド

プロジェクト作成、依存追加、スクリプト実行でよく使うコマンドです。

```sh
npm init -y
npm install package-name
npm install -D package-name
npm run dev
npm test
```

## 注意点

- 等価比較は `==` ではなく `===` を使います。
- `null` と `undefined` の扱いはプロジェクト内でそろえます。
- 日付やタイムゾーンの処理は標準 `Date` だけで完結しない場合があります。
- 外部入力を HTML に入れる場合は XSS に注意します。

## 参考

- 公式ドキュメント: [ECMAScript Language Specification](https://tc39.es/ecma262/)
- 関連: [MDN JavaScript Guide](https://developer.mozilla.org/docs/Web/JavaScript/Guide)
