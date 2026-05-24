# MySQL 基本チートシート

確認日: 2026-05-24

MySQL でよく使う DDL、DML、検索条件、JOIN、集計、関数、トランザクション、確認コマンドの早見表です。例では `users`、`orders`、`accounts` などのテーブルを使います。

## このページで扱う範囲

- SQL の基本操作とよく使う構文
- 実務で確認頻度が高いクエリの最小例
- データ削除、ロック、トランザクションの初歩的な注意点

## 扱わない範囲

- 詳細なパフォーマンスチューニング
- レプリケーション、パーティショニング、運用監視の詳細
- MySQL バージョン別の細かい差分

## 接続

MySQL クライアントからローカルまたは指定ホストのデータベースへ接続します。

```sh
mysql -u root -p
mysql -h 127.0.0.1 -P 3306 -u app_user -p app_db
```

## SHOW

データベース、テーブル、カラム、作成 SQL、現在の設定などを確認します。

```sql
-- 利用できるデータベース一覧を表示します。
SHOW DATABASES;
-- 現在選択しているデータベース内のテーブル一覧を表示します。
SHOW TABLES;
-- users テーブルのカラム定義を表示します。
SHOW COLUMNS FROM users;
-- users テーブルを作成する SQL を縦表示で確認します。
SHOW CREATE TABLE users\G
-- time_zone に関係する設定値を表示します。
SHOW VARIABLES LIKE 'time_zone';
-- 現在接続しているスレッド数を表示します。
SHOW STATUS LIKE 'Threads_connected';
```

## SET / 変数

セッション変数やユーザー変数を設定します。`SET SESSION` は現在の接続だけに効きます。

```sql
-- 現在の接続だけタイムゾーンを日本時間に変更します。
SET SESSION time_zone = '+09:00';
-- SQL 内で再利用できるユーザー変数に値を入れます。
SET @target_user_id := 1;

-- ユーザー変数に入っている値を確認します。
SELECT @target_user_id AS target_user_id;
```

## CREATE TABLE

テーブルを作成します。主キー、ユニーク制約、日時カラムを最初から定義しておくと運用しやすくなります。

```sql
-- users テーブルを新規作成し、メールアドレスの重複を禁止します。
CREATE TABLE users (
  id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  email VARCHAR(255) NOT NULL,
  name VARCHAR(100) NOT NULL,
  status VARCHAR(20) NOT NULL DEFAULT 'active',
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (id),
  UNIQUE KEY uq_users_email (email)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

## ALTER TABLE

既存テーブルのカラム、インデックス、制約を変更します。本番の大きなテーブルではロックや実行時間に注意します。

```sql
-- users テーブルに最終ログイン日時カラムを追加します。
ALTER TABLE users ADD COLUMN last_login_at DATETIME NULL;
-- name カラムの最大文字数を 150 文字へ変更します。
ALTER TABLE users MODIFY COLUMN name VARCHAR(150) NOT NULL;
-- 追加した last_login_at カラムを削除します。
ALTER TABLE users DROP COLUMN last_login_at;
```

## DROP TABLE

テーブルを削除します。データも定義も消えるため、実行前にバックアップと対象確認が必要です。

> [!WARNING]
> この操作はテーブル定義とデータを削除します。実行前に対象とバックアップを確認してください。

```sql
-- old_users テーブルが存在する場合だけ削除します。
DROP TABLE IF EXISTS old_users;
```

## CREATE INDEX

検索条件、JOIN 条件、並び替えでよく使うカラムにインデックスを作成します。

```sql
-- status と created_at で絞り込みや並び替えをしやすくします。
CREATE INDEX idx_users_status_created_at ON users (status, created_at);
-- user_id ごとの注文履歴を作成日時順で探しやすくします。
CREATE INDEX idx_orders_user_id_created_at ON orders (user_id, created_at);
```

## CREATE VIEW

よく使う SELECT をビューとして保存します。複雑な JOIN や絞り込みを再利用したいときに便利です。

```sql
-- active のユーザーだけを参照できるビューを作成します。
CREATE VIEW active_users AS
SELECT id, email, name
FROM users
WHERE status = 'active';
```

## TRUNCATE

テーブルの全データを高速に削除します。`DELETE` と違い、全件削除専用で AUTO_INCREMENT もリセットされます。DDL 扱いなので暗黙コミットに注意します。

> [!WARNING]
> この操作はテーブル内の全データを削除します。実行前に対象とバックアップを確認してください。

```sql
-- 一時取り込み用テーブルの全データを削除し、採番もリセットします。
TRUNCATE TABLE temp_import_users;
```

## SELECT

テーブルから必要なカラムを取得します。`*` は調査時に便利ですが、アプリでは必要なカラムを明示します。

```sql
-- users テーブルから主要なユーザー情報を取得します。
SELECT id, email, name
FROM users;
```

## INSERT

新しい行を追加します。複数行をまとめて追加することもできます。

```sql
-- users テーブルに 1 人分のユーザーを追加します。
INSERT INTO users (email, name, status)
VALUES ('alice@example.com', 'Alice', 'active');

-- users テーブルに複数ユーザーをまとめて追加します。
INSERT INTO users (email, name, status)
VALUES
  ('bob@example.com', 'Bob', 'active'),
  ('carol@example.com', 'Carol', 'inactive');
```

## UPDATE

既存行を更新します。`WHERE` を付け忘れると全行更新になるため、事前に同じ条件で `SELECT` します。

```sql
-- id が 1 のユーザー名を更新します。
UPDATE users
SET name = 'Alice Example'
WHERE id = 1;
```

## DELETE

条件に一致する行を削除します。全件削除したい場合でも、意図が分かるように条件確認を行います。

> [!WARNING]
> この操作は条件に一致するデータを削除します。実行前に `WHERE` の対象件数とバックアップを確認してください。

```sql
-- inactive 状態のユーザーを削除します。
DELETE FROM users
WHERE status = 'inactive';
```

## REPLACE

主キーまたはユニークキーが重複した場合、既存行を削除してから挿入します。関連する外部キーやトリガーがある場合は副作用に注意します。

> [!WARNING]
> `REPLACE` は内部的に既存行を削除してから挿入する場合があります。外部キー、トリガー、採番への影響を確認してください。

```sql
-- id または email が重複した場合、既存行を削除してから挿入します。
REPLACE INTO users (id, email, name, status)
VALUES (1, 'alice@example.com', 'Alice Replaced', 'active');
```

## INSERT ... ON DUPLICATE KEY UPDATE

主キーまたはユニークキーが重複したときに、既存行を更新します。UPSERT としてよく使います。

```sql
-- email が重複しなければ追加し、重複したら名前と状態を更新します。
INSERT INTO users (email, name, status)
VALUES ('alice@example.com', 'Alice', 'active')
ON DUPLICATE KEY UPDATE
  name = VALUES(name),
  status = VALUES(status),
  updated_at = CURRENT_TIMESTAMP;
```

## WHERE

取得、更新、削除の対象を条件で絞ります。

```sql
-- active かつ 2026-01-01 以降に作成されたユーザーだけを取得します。
SELECT id, email
FROM users
WHERE status = 'active'
  AND created_at >= '2026-01-01';
```

## ORDER BY

結果の並び順を指定します。`ASC` は昇順、`DESC` は降順です。

```sql
-- 新しく作成されたユーザーから順に取得し、同時刻なら id の大きい順にします。
SELECT id, email, created_at
FROM users
ORDER BY created_at DESC, id DESC;
```

## LIMIT / OFFSET

取得件数と開始位置を指定します。ページングでは並び順を固定して使います。

```sql
-- id 順に並べたユーザーの 41 件目から 20 件を取得します。
SELECT id, email
FROM users
ORDER BY id
LIMIT 20 OFFSET 40;
```

## GROUP BY

行をグループ化して集計します。SELECT する非集計カラムは `GROUP BY` に含めます。

```sql
-- status ごとにユーザー数を集計します。
SELECT status, COUNT(*) AS user_count
FROM users
GROUP BY status;
```

## HAVING

集計後の結果に条件を付けます。集計前の絞り込みは `WHERE`、集計後の絞り込みは `HAVING` です。

```sql
-- 注文数が 3 件以上あるユーザーだけを集計結果から取得します。
SELECT user_id, COUNT(*) AS order_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) >= 3;
```

## DISTINCT

重複行を除いた結果を返します。対象カラムの組み合わせで重複判定されます。

```sql
-- users テーブルに存在する status の種類だけを重複なしで取得します。
SELECT DISTINCT status
FROM users;
```

## UNION / UNION ALL

複数の SELECT 結果を縦に結合します。`UNION` は重複を除き、`UNION ALL` はすべて残します。

```sql
-- users と newsletter_subscribers のメールアドレスを重複なしでまとめます。
SELECT email FROM users
UNION
SELECT email FROM newsletter_subscribers;

-- users と newsletter_subscribers のメールアドレスを重複も含めてまとめます。
SELECT email FROM users
UNION ALL
SELECT email FROM newsletter_subscribers;
```

## SUBQUERY

SELECT の中に別の SELECT を書きます。条件や派生テーブルとして使えます。

```sql
-- 高額注文をしたユーザーだけをサブクエリで絞り込みます。
SELECT id, email
FROM users
WHERE id IN (
  SELECT user_id
  FROM orders
  WHERE total_amount >= 10000
);
```

## WITH (CTE)

共通テーブル式です。複雑なクエリを名前付きの中間結果に分けて読みやすくします。MySQL 8.0 以降で使えます。

```sql
-- ユーザーごとの注文合計を CTE に分け、合計金額が高いユーザーを取得します。
WITH high_value_orders AS (
  SELECT user_id, SUM(total_amount) AS total_amount
  FROM orders
  GROUP BY user_id
  HAVING SUM(total_amount) >= 10000
)
SELECT u.id, u.email, h.total_amount
FROM users AS u
INNER JOIN high_value_orders AS h ON h.user_id = u.id;
```

## LIKE

文字列の部分一致検索に使います。`%` は任意の長さ、`_` は1文字に一致します。

```sql
-- example.com のメールアドレスを持つユーザーを部分一致で探します。
SELECT id, email
FROM users
WHERE email LIKE '%@example.com';
```

## IN / NOT IN

複数候補のいずれかに一致する、または一致しない行を絞ります。`NULL` を含む `NOT IN` は結果が分かりにくくなるため注意します。

```sql
-- active または trial のユーザーを取得します。
SELECT id, email
FROM users
WHERE status IN ('active', 'trial');

-- deleted と banned 以外のユーザーを取得します。
SELECT id, email
FROM users
WHERE status NOT IN ('deleted', 'banned');
```

## BETWEEN

範囲条件です。両端の値を含みます。

```sql
-- 注文金額が 1000 以上 5000 以下の注文を取得します。
SELECT id, total_amount
FROM orders
WHERE total_amount BETWEEN 1000 AND 5000;
```

## IS NULL / IS NOT NULL

`NULL` の判定に使います。`= NULL` では判定できません。

```sql
-- 削除されていないユーザーを取得します。
SELECT id, email
FROM users
WHERE deleted_at IS NULL;

-- 削除済みのユーザーを取得します。
SELECT id, email
FROM users
WHERE deleted_at IS NOT NULL;
```

## EXISTS / NOT EXISTS

関連する行が存在するかどうかで絞ります。存在チェックだけなら `IN` より意図が明確です。

```sql
-- 注文が 1 件以上存在するユーザーだけを取得します。
SELECT u.id, u.email
FROM users AS u
WHERE EXISTS (
  SELECT 1
  FROM orders AS o
  WHERE o.user_id = u.id
);

-- 注文が 1 件も存在しないユーザーだけを取得します。
SELECT u.id, u.email
FROM users AS u
WHERE NOT EXISTS (
  SELECT 1
  FROM orders AS o
  WHERE o.user_id = u.id
);
```

## CASE WHEN

条件に応じて表示値や分類値を切り替えます。

```sql
-- status の値に応じて、日本語の表示ラベルを作ります。
SELECT
  id,
  email,
  CASE
    WHEN status = 'active' THEN '利用中'
    WHEN status = 'inactive' THEN '停止中'
    ELSE '不明'
  END AS status_label
FROM users;
```

## COALESCE / IFNULL

`NULL` の代替値を返します。`COALESCE` は複数候補、`IFNULL` は2つの値を扱います。

```sql
-- 表示名、名前、メールアドレスの順に最初の NULL でない値をラベルにします。
SELECT
  id,
  COALESCE(display_name, name, email) AS label,
  IFNULL(deleted_at, '9999-12-31') AS deleted_at_or_default
FROM users;
```

## INNER JOIN

両方のテーブルに一致する行だけを取得します。

```sql
-- 注文が存在するユーザーと注文情報だけを取得します。
SELECT u.id, u.email, o.id AS order_id
FROM users AS u
INNER JOIN orders AS o ON o.user_id = u.id;
```

## LEFT JOIN

左側のテーブルを全て残し、右側に一致する行があれば結合します。一致しない右側カラムは `NULL` になります。

```sql
-- すべてのユーザーを残し、注文があれば注文情報も取得します。
SELECT u.id, u.email, o.id AS order_id
FROM users AS u
LEFT JOIN orders AS o ON o.user_id = u.id;
```

## RIGHT JOIN

右側のテーブルを全て残し、左側に一致する行があれば結合します。読みやすさのため `LEFT JOIN` に書き換えられることも多いです。

```sql
-- すべての注文を残し、対応するユーザーがあればユーザー情報も取得します。
SELECT u.id, u.email, o.id AS order_id
FROM users AS u
RIGHT JOIN orders AS o ON o.user_id = u.id;
```

## CROSS JOIN

両テーブルの全組み合わせを作ります。行数が掛け算で増えるため注意します。

```sql
-- 色とサイズの全組み合わせを作ります。
SELECT c.color, s.size
FROM colors AS c
CROSS JOIN sizes AS s;
```

## SELF JOIN

同じテーブルを別名で2回参照して、親子関係や比較を扱います。

```sql
-- categories テーブルを親カテゴリと子カテゴリとして自己結合します。
SELECT child.id, child.name, parent.name AS parent_name
FROM categories AS child
LEFT JOIN categories AS parent ON parent.id = child.parent_id;
```

## COUNT

行数を数えます。`COUNT(*)` は行数、`COUNT(column)` は `NULL` 以外の件数です。

```sql
-- users テーブルの総行数を数えます。
SELECT COUNT(*) AS user_count
FROM users;
```

## SUM

数値の合計を求めます。

```sql
-- ユーザーごとの注文金額合計を計算します。
SELECT user_id, SUM(total_amount) AS total_amount
FROM orders
GROUP BY user_id;
```

## AVG

数値の平均を求めます。

```sql
-- 全注文の平均注文金額を計算します。
SELECT AVG(total_amount) AS average_order_amount
FROM orders;
```

## MAX / MIN

最大値と最小値を求めます。日時や数値の範囲確認でよく使います。

```sql
-- 最も新しい注文日時と最も古い注文日時を取得します。
SELECT
  MAX(created_at) AS latest_order_at,
  MIN(created_at) AS first_order_at
FROM orders;
```

## GROUP_CONCAT

グループ内の値を文字列として連結します。表示用の一覧作成に便利です。

```sql
-- ユーザーごとの注文 ID をカンマ区切りでまとめます。
SELECT user_id, GROUP_CONCAT(id ORDER BY id SEPARATOR ',') AS order_ids
FROM orders
GROUP BY user_id;
```

## 文字列関数

`CONCAT` は文字列結合、`SUBSTRING` は一部取り出し、`LOWER` / `UPPER` は大文字小文字変換に使います。

```sql
-- 名前とメールアドレスを結合し、メールの一部や正規化した値も取得します。
SELECT
  CONCAT(name, ' <', email, '>') AS label,
  SUBSTRING(email, 1, 5) AS email_prefix,
  LOWER(email) AS normalized_email
FROM users;
```

## 日付関数

`NOW` は現在日時、`DATE_FORMAT` は日時の表示形式変換、`DATE_ADD` は日時の加算に使います。

```sql
-- 現在日時、作成日の表示用文字列、作成日の 7 日後を取得します。
SELECT
  NOW() AS current_time,
  DATE_FORMAT(created_at, '%Y-%m-%d') AS created_date,
  DATE_ADD(created_at, INTERVAL 7 DAY) AS one_week_later
FROM users;
```

## 数値関数

`ROUND` は丸め、`RAND` は乱数、`FLOOR` / `CEIL` は切り下げと切り上げに使います。

```sql
-- 税込み想定金額の丸め、乱数、切り下げ、切り上げを計算します。
SELECT
  ROUND(total_amount * 1.1, 0) AS tax_included,
  RAND() AS random_value,
  FLOOR(123.9) AS floored,
  CEIL(123.1) AS ceiled
FROM orders;
```

## CAST / CONVERT

値の型を変換します。文字列から数値、日時から文字列などに使います。

```sql
-- 文字列を数値に変換し、日時を日付だけの値に変換します。
SELECT
  CAST('123' AS UNSIGNED) AS number_value,
  CONVERT(created_at, DATE) AS created_date
FROM users;
```

## WINDOW 関数

集計結果を行ごとに保持しながら、順位や前後行との差分を計算します。MySQL 8.0 以降で使えます。

```sql
-- ユーザー内の注文順、全体の金額順位、前回注文金額を計算します。
SELECT
  user_id,
  id AS order_id,
  total_amount,
  ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC) AS row_number,
  RANK() OVER (ORDER BY total_amount DESC) AS amount_rank,
  LAG(total_amount) OVER (PARTITION BY user_id ORDER BY created_at) AS previous_amount
FROM orders;
```

## BEGIN / COMMIT / ROLLBACK

複数の更新を1つの単位として扱います。成功したら `COMMIT`、失敗したら `ROLLBACK` します。

```sql
-- ここからトランザクションを開始します。
BEGIN;

-- 口座 1 から 1000 減らします。
UPDATE accounts SET balance = balance - 1000 WHERE id = 1;
-- 口座 2 に 1000 増やします。
UPDATE accounts SET balance = balance + 1000 WHERE id = 2;

-- 2 つの更新を確定します。
COMMIT;
-- 問題があれば COMMIT せず ROLLBACK します。
-- ROLLBACK;
```

## SAVEPOINT

トランザクションの途中に戻り地点を作ります。一部だけ取り消したい場合に使います。

```sql
-- ここからトランザクションを開始します。
BEGIN;

-- 口座 1 から 1000 減らします。
UPDATE accounts SET balance = balance - 1000 WHERE id = 1;
-- この時点へ戻れるように保存点を作ります。
SAVEPOINT after_withdraw;

-- 口座 2 に 1000 増やします。
UPDATE accounts SET balance = balance + 1000 WHERE id = 2;
-- 保存点まで処理を戻し、口座 2 への更新を取り消します。
ROLLBACK TO SAVEPOINT after_withdraw;

-- 残った変更を確定します。
COMMIT;
```

## LOCK TABLES

明示的にテーブルをロックします。暗黙コミットが発生するため、トランザクション中の利用は注意します。使い終わったら必ず `UNLOCK TABLES` します。

> [!WARNING]
> ロック中は他の処理を待たせる可能性があります。対象テーブルと解除手順を確認してから実行してください。

```sql
-- users テーブルを書き込みロックします。
LOCK TABLES users WRITE;

-- ロック中に対象ユーザーの状態を更新します。
UPDATE users SET status = 'inactive' WHERE id = 1;

-- テーブルロックを解除します。
UNLOCK TABLES;
```

## EXPLAIN

クエリの実行計画を確認します。インデックスが使われているか、読み取り行数が多すぎないかを見ます。

```sql
-- user_id と created_at を使う検索の実行計画を確認します。
EXPLAIN
SELECT *
FROM orders
WHERE user_id = 1
ORDER BY created_at DESC;

-- 実際に実行して、処理時間や行数を含む実行計画を確認します。
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE user_id = 1;
```

## バックアップ

`mysqldump` で論理バックアップを取り、`mysql` コマンドでリストアします。

```sh
mysqldump -u root -p --single-transaction --routines --triggers app_db > app_db.sql
mysql -u root -p app_db < app_db.sql
```

## 注意点

- 本番で `UPDATE` や `DELETE` を実行する前に、同じ `WHERE` の `SELECT` で対象件数を確認します。
- `DROP TABLE`、`TRUNCATE`、`REPLACE`、`LOCK TABLES` は影響が大きいため、実行前に目的と対象を確認します。
- DDL や `LOCK TABLES` は暗黙コミットが発生することがあります。
- 文字コードは原則 `utf8mb4` を使います。
- 金額は `FLOAT` ではなく `DECIMAL` や整数の最小通貨単位で扱います。
- 外部入力は文字列結合せず、プレースホルダー付きのプリペアドステートメントを使います。
- CTE と WINDOW 関数は MySQL 8.0 以降を前提にします。

## 参考

- 公式ドキュメント: [MySQL Reference Manual](https://dev.mysql.com/doc/refman/en/)
- 関連: [SQL Statements](https://dev.mysql.com/doc/refman/en/sql-statements.html)
