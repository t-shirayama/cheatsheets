# Java 基本チートシート

確認日: 2026-05-24

Java のコンパイル、基本構文、コレクション、Stream、例外処理の早見表です。

## コンパイルと実行

`javac` で `.java` ファイルをコンパイルし、生成されたクラスを `java` で実行します。

```sh
javac Main.java
java Main
```

最小の Java プログラムは `main` メソッドから実行されます。

```java
public class Main {
    public static void main(String[] args) {
        // 標準出力に文字列を表示します。
        System.out.println("Hello, Java");
    }
}
```

## 変数と型

Java は静的型付けのため、変数宣言時に型を明示します。

```java
String name = "Alice";
int count = 3;
long total = 100L;
boolean enabled = true;
double rate = 0.1;
```

## 条件分岐とループ

条件に応じた処理と、回数指定やコレクション走査の基本形です。

```java
import java.util.List;

if (count > 0) {
    System.out.println("active");
} else {
    System.out.println("empty");
}

for (int i = 0; i < 3; i++) {
    System.out.println(i);
}

List<String> items = List.of("apple", "banana");

for (String item : items) {
    System.out.println(item);
}
```

## クラス

不変にしたい値は `private final` にし、コンストラクタで初期化します。

```java
public class User {
    private final long id;
    private final String email;

    public User(long id, String email) {
        // 生成時に受け取った値をインスタンスへ保持します。
        this.id = id;
        this.email = email;
    }

    public long getId() {
        return id;
    }

    public String getEmail() {
        return email;
    }
}
```

## record

値を運ぶだけの型は `record` で簡潔に定義できます。

```java
public record UserDto(long id, String email) {
}
```

## コレクション

`List` は順序付きの要素、`Map` はキーと値の対応を扱います。

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

List<String> items = new ArrayList<>();
items.add("apple");

Map<Long, String> namesById = new HashMap<>();
namesById.put(1L, "Alice");
```

## Stream

Stream はコレクションの絞り込みや変換を連続して書けます。

```java
import java.util.List;

List<String> names = List.of("Alice", "Bob", "Carol");

List<String> filtered = names.stream()
    // A で始まる名前だけを残します。
    .filter(name -> name.startsWith("A"))
    .map(String::toUpperCase)
    .toList();
```

## Optional

`Optional` は値が存在しない可能性を戻り値で表すときに使います。

```java
import java.util.Optional;

Optional<String> email = Optional.ofNullable(user.getEmail());
// 値がなければデフォルト値を使います。
String value = email.orElse("unknown@example.com");
```

## 例外処理

失敗する可能性のある処理は、例外を捕まえてログや代替処理につなげます。

```java
try {
    service.run();
} catch (IllegalArgumentException e) {
    System.err.println(e.getMessage());
} finally {
    System.out.println("done");
}
```

## try-with-resources

閉じる必要があるリソースは、try-with-resources で自動的にクローズします。

```java
import java.nio.file.Files;
import java.nio.file.Path;

try (var input = Files.newInputStream(Path.of("data.txt"))) {
    // ブロックを抜けると input は自動的に close されます。
    byte[] bytes = input.readAllBytes();
}
```

## 文字列比較と BigDecimal

文字列比較は `.equals()` を使い、金額のような小数は `BigDecimal` で扱います。

```java
import java.math.BigDecimal;

String status = "active";

if ("active".equals(status)) {
    System.out.println("enabled");
}

// 文字列から作ると、double 由来の丸め誤差を避けやすくなります。
BigDecimal price = new BigDecimal("123.45");
```

## Maven と Gradle

```sh
mvn test
mvn package
```

```sh
./gradlew test
./gradlew build
```

## 注意点

- `String` の比較は `==` ではなく `.equals()` を使います。
- `BigDecimal` は文字列から作ると小数の丸め誤差を避けやすくなります。
- `Optional` をフィールドや引数に多用せず、戻り値での不在表現を中心に使います。
- Java の機能はバージョン差があるため、プロジェクトの LTS バージョンを確認します。

## 参考リンク

- [Java Documentation](https://docs.oracle.com/en/java/)
