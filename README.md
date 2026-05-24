# Developer Cheatsheets

開発者向けチートシートをまとめるためのリポジトリです。コマンド、設定例、デバッグ手順、設計メモなど、日々の開発で何度も参照する情報を短く探しやすい形で蓄積します。

## 方針

- 1つのチートシートは、できるだけ短く実用的にまとめる
- コピペして使うコマンドや設定例には、前提条件と注意点を添える
- ソースコード例には、短い説明と必要最小限のコメントを添える
- 古くなりやすい情報には、対象バージョンや確認日を書く
- 関連する公式ドキュメントや一次情報へのリンクを残す

## 推奨構成

```text
.
├── README.md
├── AGENTS.md
├── git/
├── shell/
├── docker/
├── mysql/
├── javascript/
├── typescript/
├── python/
├── php/
├── react/
├── java/
├── security/
└── tools/
```

カテゴリは必要になった時点で追加します。迷った場合は、まず `tools/` や言語別ディレクトリに置き、あとから整理します。

## 収録チートシート

- [MySQL 基本チートシート](mysql/basics.md)
- [JavaScript 基本チートシート](javascript/basics.md)
- [TypeScript 基本チートシート](typescript/basics.md)
- [Python 基本チートシート](python/basics.md)
- [PHP 基本チートシート](php/basics.md)
- [React 基本チートシート](react/basics.md)
- [Java 基本チートシート](java/basics.md)

## 書き方の目安

各チートシートは次のような構成を推奨します。

````markdown
# タイトル

## よく使うコマンド

このコマンドは、サンプル操作を実行する最小例です。

```sh
command --example
```

## コード例

この例では、入力値を受け取り結果を返す基本形を示します。

```js
function example(input) {
  // ここに処理の意図が分かる短いコメントを書く
  return input;
}
```

## 注意点

- 前提条件
- 失敗しやすい点
- 参考リンク
````

## 運用メモ

- ファイル名は小文字の kebab-case を基本にします。例: `git-rebase.md`
- Markdown は読みやすさを優先し、過度に長い説明は避けます
- ソースコード例には、何を示すコードか分かる短い説明を添えます
- コード内コメントは、意図や注意点を補う必要がある箇所だけに入れます
- React のコード例は TypeScript/TSX で書きます
- 破壊的なコマンドには必ず警告を付けます
- 秘密情報、API キー、社内固有の認証情報はコミットしません
