# PHP 基本チートシート

確認日: 2026-05-24

PHP の基本構文、Composer、PDO、実行方法の早見表です。

## 実行と Composer

PHP の実行、簡易サーバー起動、Composer による依存管理の基本コマンドです。

```sh
php -v
php script.php
php -S localhost:8000 -t public
composer init
composer install
composer require vendor/package
composer dump-autoload
```

## 基本構文

`strict_types` を有効にし、変数と文字列出力の基本形を示します。

```php
<?php

declare(strict_types=1);

$name = 'Alice';
$count = 3;
$enabled = true;

// ダブルクォート文字列では変数を展開できます。
echo "Hello, {$name}\n";
```

## 配列

PHP の配列は、リストとしても連想配列としても使えます。

```php
<?php

$items = ['apple', 'banana'];
$user = [
    'id' => 1,
    'email' => 'alice@example.com',
];

foreach ($items as $index => $item) {
    // foreach ではキーと値を同時に取り出せます。
    echo "{$index}: {$item}\n";
}
```

## 関数

引数と戻り値に型を付けると、意図しない値を検出しやすくなります。

```php
<?php

function greet(string $name = 'guest'): string
{
    return "Hello, {$name}";
}
```

## クラス

コンストラクタプロモーションを使うと、プロパティ定義を短く書けます。

```php
<?php

final class User
{
    public function __construct(
        // readonly は生成後の再代入を防ぎます。
        public readonly int $id,
        public readonly string $email,
    ) {
    }
}
```

## 例外処理

失敗する処理は例外として扱い、ログ出力や後片付けにつなげます。

```php
<?php

try {
    throw new RuntimeException('failed');
} catch (RuntimeException $e) {
    error_log($e->getMessage());
} finally {
    echo "done\n";
}
```

## PDO

PDO ではプリペアドステートメントを使い、SQL インジェクションを防ぎます。

```php
<?php

$pdo = new PDO(
    'mysql:host=127.0.0.1;dbname=app_db;charset=utf8mb4',
    'app_user',
    'change_me',
    [
        // SQL エラーを例外として受け取ります。
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    ],
);

$stmt = $pdo->prepare('SELECT id, email FROM users WHERE id = :id');
// :id に値をバインドしてから実行します。
$stmt->execute(['id' => 1]);
$user = $stmt->fetch();
```

## JSON

JSON のエンコードとデコードでは、失敗時に例外を投げる設定にします。

```php
<?php

$json = json_encode(['id' => 1], JSON_THROW_ON_ERROR);
$data = json_decode($json, true, flags: JSON_THROW_ON_ERROR);
```

## HTML エスケープ

ユーザー入力を HTML に出力するときは、XSS を防ぐためにエスケープします。

```php
<?php

$name = $_GET['name'] ?? 'guest';

// HTML に出す値は必ずエスケープします。
echo htmlspecialchars($name, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
```

## 環境変数

パスワードや API キーなどの秘密情報は、コードに直書きせず環境変数から読みます。

```php
<?php

$databaseUrl = getenv('DATABASE_URL');

if ($databaseUrl === false) {
    throw new RuntimeException('DATABASE_URL is not set');
}
```

## 注意点

- 新規ファイルでは `declare(strict_types=1);` を検討します。
- SQL は文字列結合せず、PDO のプリペアドステートメントを使います。
- HTML 出力では `htmlspecialchars($value, ENT_QUOTES, 'UTF-8')` を使います。
- 依存関係は Composer で管理し、`vendor/` の扱いはプロジェクト方針に合わせます。

## 参考リンク

- [PHP Manual](https://www.php.net/manual/en/)
- [Composer Documentation](https://getcomposer.org/doc/)
