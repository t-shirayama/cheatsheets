# Python 基本チートシート

確認日: 2026-05-24

Python の仮想環境、基本構文、ファイル操作、テストの早見表です。

## 仮想環境

プロジェクトごとに仮想環境を作り、依存関係を分離します。

```sh
python -m venv .venv
```

```powershell
.\.venv\Scripts\Activate.ps1
```

```sh
python -m pip install --upgrade pip
python -m pip install requests pytest
python -m pip freeze > requirements.txt
python -m pip install -r requirements.txt
```

## 変数とコレクション

Python は代入時に型を推論し、リストや辞書などを日常的に使います。

```py
name = "Alice"
count = 3
enabled = True

# よく使う組み込みコレクションです。
items = ["apple", "banana"]
user = {"id": 1, "name": "Alice"}
unique_ids = {1, 2, 3}
point = (10, 20)
```

## 条件分岐とループ

インデントでブロックを表し、`for` で要素を順に処理します。

```py
if count > 0:
    print("active")
else:
    print("empty")

for item in items:
    print(item)

for index, item in enumerate(items):
    # enumerate はインデックスと値を同時に取り出します。
    print(index, item)
```

## 関数

型ヒントを付けると、エディタ補完や静的解析が効きやすくなります。

```py
def greet(name: str = "guest") -> str:
    return f"Hello, {name}"


def total(numbers: list[int]) -> int:
    return sum(numbers)
```

## リスト内包表記

リスト内包表記は、変換や絞り込みを短く書くための構文です。

```py
numbers = [1, 2, 3, 4]
squares = [n * n for n in numbers]
even_numbers = [n for n in numbers if n % 2 == 0]
```

## 辞書内包表記

キーと値を組み立てる処理は、辞書内包表記で短く書けます。

```py
users = [{"id": 1, "name": "Alice"}, {"id": 2, "name": "Bob"}]

# id をキーにして、ユーザー名を引ける辞書を作ります。
names_by_id = {user["id"]: user["name"] for user in users}
```

## ファイルと JSON

ファイルパスは `pathlib.Path` で扱うと、OS 差分を吸収しやすくなります。

```py
from pathlib import Path

path = Path("data.txt")
# 文字コードを明示して環境差分を減らします。
path.write_text("hello\n", encoding="utf-8")
text = path.read_text(encoding="utf-8")
```

JSON は外部 API や設定ファイルとのやり取りでよく使います。

```py
import json

data = {"id": 1, "name": "Alice"}
Path("data.json").write_text(json.dumps(data, ensure_ascii=False, indent=2), encoding="utf-8")
loaded = json.loads(Path("data.json").read_text(encoding="utf-8"))
```

## with

ファイルや接続などのリソースは `with` を使うと自動的に閉じられます。

```py
from pathlib import Path

path = Path("data.txt")

with path.open("r", encoding="utf-8") as file:
    # with ブロックを抜けると file は自動で閉じられます。
    text = file.read()
```

## 例外処理

変換や I/O など失敗する処理は、例外を捕まえて扱います。

```py
try:
    value = int("123")
except ValueError as error:
    print(error)
else:
    print(value)
finally:
    print("done")
```

## dataclass

データを運ぶだけのクラスは `dataclass` で簡潔に書けます。

```py
from dataclasses import dataclass


@dataclass(frozen=True)
class User:
    # frozen=True のため、生成後に属性を変更できません。
    id: int
    email: str
```

## pytest

テスト関数は `test_` で始め、期待する結果を `assert` で確認します。

```py
def add(a: int, b: int) -> int:
    return a + b


def test_add() -> None:
    assert add(1, 2) == 3
```

```sh
python -m pytest
```

## 注意点

- 仮想環境を使い、グローバル環境へのインストールを避けます。
- ファイルパスは文字列結合より `pathlib.Path` を優先します。
- 文字コードは明示しておくと環境差分を減らせます。
- 秘密情報はコードに直書きせず、環境変数やシークレット管理を使います。

## 参考リンク

- [Python Documentation](https://docs.python.org/3/)
