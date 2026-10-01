<div align="center">
  <h1>zenn-docs</h1>
  <p>Zennで公開する技術記事・本の執筆リポジトリ</p>
  <p>
    <a href="./articles">記事</a>
    &nbsp;&bull;&nbsp;
    <a href="./books">本</a>
    &nbsp;&bull;&nbsp;
    <a href="https://zenn.dev/zenn/articles/zenn-cli-guide">Zenn CLIガイド</a>
  </p>
</div>

## 開発環境

Nixで開発ツールを固定します。direnvがディレクトリへの移動時に環境を自動で読み込みます。
JavaScriptの実行にはNode.js、依存管理にはpnpmを使います。

| 用途                       | ツール                  | 設定・バージョン管理                           |
| -------------------------- | ----------------------- | ---------------------------------------------- |
| 開発環境                   | Nix、direnv、nix-direnv | `flake.nix`、`flake.lock`、`.envrc`            |
| ランタイム・依存管理       | Node.js 24、pnpm 11     | `flake.lock`、`package.json`、`pnpm-lock.yaml` |
| 記事・本の作成とプレビュー | Zenn CLI                | `package.json`、`pnpm-lock.yaml`               |
| Markdown・JSONなどの整形   | oxfmt                   | `.oxfmtrc.jsonc`、`.editorconfig`              |
| Nixの整形                  | nixfmt                  | `flake.nix`                                    |
| 日本語の校正               | textlint                | `.textlintrc.json`                             |

## セットアップ

[Nix](https://nixos.org/download/)、[direnv](https://direnv.net/)、[nix-direnv](https://github.com/nix-community/nix-direnv)を用意します。
シェルにdirenvのフックを設定し、direnvからnix-direnvを読み込める状態にしてください。
リポジトリのルートで次を実行します。

```sh
direnv allow
pnpm install --frozen-lockfile
pnpm preview
```

プレビューは[localhost:8000](http://localhost:8000)で開きます。終了するには`Ctrl+C`を押してください。
以後はディレクトリに入ると開発環境が読み込まれます。
direnvを使わない場合は、先に`nix develop`を実行してください。

## 執筆

```sh
pnpm new:art --slug <slug>
pnpm new:book
pnpm preview
```

記事は`articles/<slug>.md`に作成されます。先頭のFront Matterでタイトル・トピックなどを設定し、本文をMarkdownで書きます。
下書きの間は`published: false`にして、プレビューで表示を確認してください。
本は`books/<slug>/`に設定ファイルと章を置きます。

詳しい使い方は[Zenn CLIガイド](https://zenn.dev/zenn/articles/zenn-cli-guide)と[Markdown記法一覧](https://zenn.dev/zenn/articles/markdown-guide)を参照してください。

## コマンド

| コマンド         | 内容                             |
| ---------------- | -------------------------------- |
| `pnpm preview`   | 記事・本をブラウザでプレビュー   |
| `pnpm new:art`   | 記事のひな形を作成               |
| `pnpm new:book`  | 本のひな形を作成                 |
| `pnpm fmt`       | Markdown・JSONなどをoxfmtで整形  |
| `pnpm fmt:check` | 整形の差分を検査                 |
| `pnpm lint`      | 記事・本・READMEをtextlintで校正 |
| `pnpm lint:fix`  | 校正の指摘を自動修正             |
| `nix fmt`        | Nixファイルをnixfmtで整形        |

oxfmtはMarkdownの文章の改行を維持します。
記事中のコード例は自動整形しません。
textlintは日本語の技術文書向けルールと、スペースに関するルールを使います。
`lint:fix`は対象ファイルを書き換えます。残った指摘は内容を確認し、手動で修正してください。

## リポジトリ構成

```text
.
├── articles/          # 記事のMarkdown
├── books/             # 本の設定・各章のMarkdown
├── flake.nix          # 開発ツールとNixのformatter
├── flake.lock         # Nixの依存固定
├── .envrc             # direnvによる環境の読み込み
├── .editorconfig      # インデント・改行などの共通設定
├── .oxfmtrc.jsonc     # oxfmtの設定
├── .textlintrc.json   # 日本語の校正ルール
├── package.json       # 執筆・検査コマンドと依存
└── pnpm-lock.yaml     # JavaScriptの依存固定
```

## Zed

ローカルの`.zed/settings.json`でdirenvの読み込みと保存時整形を設定します。
Markdown・JSON・YAMLはoxfmt、Nixはnixfmtを使います。
`.zed/tasks.json`からプレビュー、校正、整形チェックを実行できます。

`.zed/`は`.git/info/exclude`で追跡対象から外しています。
エディター設定はローカル専用で、リポジトリには共有しません。

## ライセンス

記事本文・コード・設定を含む、このリポジトリの著作物は[MIT License](./LICENSE)で公開しています。

Copyright (c) 2026 koutyuke
