---
title: "bunでパッケージにpatchをあててみた"
emoji: "🔧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["bun", "typescript"]
published: true
---

こんにちは〜 NITICのこうちゅけです。
以前にクラスメイトの[@れおはかせ](https://zenn.dev/reohakase)が[Bunでパッチを当てるためにpatch-packageを動かす方法](https://zenn.dev/reohakase/articles/65d8062bc74721)という記事を執筆していました。
この記事は`bun patch`コマンドが実装される前に執筆されていて、`patch-package`をbunで動かすために`patch-package`で`patch-package`にpatchを当てるということをした、という記事でした。
現在は`bun patch`が実装され、bunだけでpatchが当てられるようになったので実際に使ってみた、という記事です。

**bunの公式ページ**
https://bun.sh/docs/install/patch

# 0. 環境

この記事では以下の環境で開発、コード提案を行なっています。

```sh
❯ bun -v
1.1.26
```

# 1. パッチを当てるパッケージを準備する

今回、patchを当てるパッケージはElysiaJSというWebフレームワークのOpenAPI用のドキュメントを生成してくれる[elysia-swagger](https://github.com/elysiajs/elysia-swagger)というパッケージです。
patchの対象パッケージは基本的にどれでも大丈夫だと思います。

```sh
# パッケージのインストール
bun add @elysiajs/swagger@1.1.1
```

パッケージがインストールできたら、以下のコマンドを実行します。

```sh
bun patch <pkg>
```

今回の場合:(pathが長いので置き換えています。)

```sh
bun patch @elysiajs/swagger@1.1.1
```

出力結果:

```sh
To patch @elysiajs/swagger, edit the following folder:

  /your/dir/node_modules/@elysiajs/swagger

Once you're done with your changes, run:

  bun patch --commit '/your/dir/node_modules/@elysiajs/swagger'
```

patchコマンドの引数でパッケージを指定するのですが、以下のような指定方法があるらしいです。

|     対象パッケージ      |              今回の場合だと              |
| :---------------------: | :--------------------------------------: |
|      パッケージ名       |            @elysiajs/swagger             |
| パッケージ名@バージョン |         @elysiajs/swagger@1.1.1          |
|          パス           | /your/dir/node_modules/@elysiajs/swagger |

今回はバージョン込みの指定方法で行っています。
コマンドの実行後、パッケージがある場所のpathと、patchを反映するためのコマンドが出力されます。

# 2. コードを変更する

さて、下準備が終わったので実際にコードを書き換えていきましょう。
記述方法は何でもいいんですが自分はVSCodeで開いて書き換えました。

```sh
code /your/dir/node_modules/@elysiajs/swagger
```

```diff js:index.mjs
// ...

// src/utils.ts
- import path from "path";
+ import path from "node:path";


// node_modules/@sinclair/typebox/build/esm/type/symbols/symbols.mjs
var TransformKind = Symbol.for("TypeBox.Transform");
var ReadonlyKind = Symbol.for("TypeBox.Readonly");
var OptionalKind = Symbol.for("TypeBox.Optional");
var Hint = Symbol.for("TypeBox.Hint");
var Kind = Symbol.for("TypeBox.Kind");

// ...
```

終わったらきちんと変更を保存しましょう。

# 3. 変更を反映する

コードの変更を反映するために以下のコマンドを実行します。
これは最初に実行したコマンドの出力にあったコマンドです。

```sh
bun patch --commit <path or pkg>
```

今回の場合だと:

```sh
bun patch --commit '/your/dir/node_modules/@elysiajs/swagger'
```

そうすると変更を加えたパッケージに`.bun-tag-<hash値>`というファイルが生成されます。
また、プロジェクトのrootに`patches`というフォルダが生成され、その中に`@elysiajs%2Fswagger@1.1.1.patch`というファイルが作成されます。

<!-- prettier-ignore -->
```diff js:@elysiajs%2Fswagger@1.1.1.patch
diff --git a/dist/index.mjs b/dist/index.mjs
index c421da0e95452e80b8fa74c02af601dc85337bde..37eff454ce1b722acda9b7fdf2f8be8f6d96c099 100644
--- a/dist/index.mjs
+++ b/dist/index.mjs
@@ -274,7 +274,7 @@ var ScalarRender = (version, config, cdn) => `<!doctype html>
 </html>`;
 
 // src/utils.ts
-import path from "path";
+import path from "node:path";
 
 // node_modules/@sinclair/typebox/build/esm/type/symbols/symbols.mjs
 var TransformKind = Symbol.for("TypeBox.Transform");
```

さらに、プロジェクトの`package.json`に`patchedDependencies`というフィールドが作成されます。

```json:package.json
{
  ...
  "patchedDependencies": {
    "@elysiajs/swagger@1.1.1": "patches/@elysiajs%2Fswagger@1.1.1.patch"
  }
}
```

たったこれだけでパッチを当てることができます。

# 4. おわりに

今回、`bun patch`コマンドを使用してpatchをあててみましたが、意外と簡単にpatchをあてれてびっくりしました。
今後どんどん使用していこうかなと思いました。
