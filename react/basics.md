# React 基本チートシート

確認日: 2026-05-24

React と TypeScript を組み合わせるときの、コンポーネント、Props、State、Hooks の基本構文です。

## コンポーネント

Props の型を先に定義し、コンポーネントの引数に適用します。

```tsx
type GreetingProps = {
  name: string;
};

function Greeting({ name }: GreetingProps) {
  // JSX 内では波括弧で JavaScript の値を埋め込みます。
  return <h1>Hello, {name}</h1>;
}

export default function App() {
  return <Greeting name="Alice" />;
}
```

## Props と分割代入

親から渡されるデータとコールバックを Props として型定義します。

```tsx
type User = {
  id: number;
  name: string;
};

type UserCardProps = {
  user: User;
  onSelect: (userId: number) => void;
};

function UserCard({ user, onSelect }: UserCardProps) {
  return (
    <button type="button" onClick={() => onSelect(user.id)}>
      {user.name}
    </button>
  );
}
```

## State

`useState` は現在値と更新関数を返します。前の値を使う更新は関数形式にします。

```tsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button type="button" onClick={() => setCount((current) => current + 1)}>
      {count}
    </button>
  );
}
```

## Effect

`useEffect` は API 取得など、React の外側にある処理と同期するときに使います。

```tsx
import { useEffect, useState } from "react";

type User = {
  id: number;
  name: string;
};

function UserList() {
  const [users, setUsers] = useState<User[]>([]);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    // アンマウント後に state を更新しないためのフラグです。
    let cancelled = false;

    const fetchUsers = async () => {
      try {
        const response = await fetch("/api/users");

        if (!response.ok) {
          throw new Error(`Request failed: ${response.status}`);
        }

        const data = (await response.json()) as User[];

        if (!cancelled) {
          setUsers(data);
          setError(null);
        }
      } catch (caughtError) {
        if (!cancelled) {
          setError(caughtError instanceof Error ? caughtError : new Error("Unknown error"));
        }
      }
    };

    void fetchUsers();

    return () => {
      cancelled = true;
    };
  }, []);

  if (error) {
    return <p role="alert">{error.message}</p>;
  }

  return users.map((user) => <div key={user.id}>{user.name}</div>);
}
```

## Callback

子コンポーネントへ渡す関数を安定させたいときは `useCallback` を使います。

```tsx
import { useCallback, useState } from "react";

function UserSearch() {
  const [keyword, setKeyword] = useState("");

  const clearKeyword = useCallback(() => {
    // setter に固定値を渡すだけなので依存配列は空にできます。
    setKeyword("");
  }, []);

  return <button type="button" onClick={clearKeyword}>{keyword || "Clear"}</button>;
}
```

## 条件付き表示

早期 return を使うと、状態ごとの表示を読みやすく分けられます。

```tsx
type StatusProps = {
  loading: boolean;
  error: Error | null;
};

function Status({ loading, error }: StatusProps) {
  if (loading) {
    return <p>Loading...</p>;
  }

  if (error) {
    return <p role="alert">{error.message}</p>;
  }

  return <p>Ready</p>;
}
```

## リスト

配列を表示するときは、各要素に安定した `key` を付けます。

```tsx
type Todo = {
  id: number;
  title: string;
};

type TodoListProps = {
  todos: Todo[];
};

function TodoList({ todos }: TodoListProps) {
  return (
    <ul>
      {todos.map((todo) => (
        // key には index ではなく、データの ID を使います。
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  );
}
```

## フォーム

フォーム送信ではブラウザの標準送信を止め、React 側の関数へ値を渡します。

```tsx
import type { FormEvent } from "react";
import { useState } from "react";

type SearchFormProps = {
  onSearch: (keyword: string) => void;
};

function SearchForm({ onSearch }: SearchFormProps) {
  const [keyword, setKeyword] = useState("");

  const handleSubmit = (event: FormEvent<HTMLFormElement>) => {
    // ページ遷移を防ぎ、React の処理として送信します。
    event.preventDefault();
    onSearch(keyword);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={keyword} onChange={(event) => setKeyword(event.target.value)} />
      <button type="submit">Search</button>
    </form>
  );
}
```

## メモ化

`memo` と `useMemo` は、再レンダーや重い計算が問題になったときに検討します。

```tsx
import { memo, useMemo } from "react";

type User = {
  id: number;
  name: string;
  active: boolean;
};

type UserRowProps = {
  user: User;
};

const UserRow = memo(function UserRow({ user }: UserRowProps) {
  return <div>{user.name}</div>;
});

type UserCountProps = {
  users: User[];
};

function UserCount({ users }: UserCountProps) {
  const activeCount = useMemo(
    () => users.filter((user) => user.active).length,
    [users],
  );

  return <span>{activeCount}</span>;
}
```

## 注意点

- React のチートシートでは、コード例は TypeScript/TSX で書きます。
- Props は `type XxxProps` で明示し、イベントハンドラーの引数型も必要に応じて書きます。
- `key` には配列の index より安定した ID を優先します。
- State は直接変更せず、必ず setter で新しい値を渡します。
- `useEffect` は外部システムとの同期に使い、計算だけならレンダー中か `useMemo` を検討します。
- 入力、ボタン、エラー表示にはアクセシビリティ属性を意識します。

## 参考リンク

- [React Documentation](https://react.dev/)
