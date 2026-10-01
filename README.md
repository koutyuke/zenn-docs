# zenn-doc

Zennの記事・本を管理するリポジトリです。記事は`articles/`、本は`books/`に置きます。

## 開発環境

Nix、direnv、nix-direnvを利用します。シェルにdirenvのフックを設定してから、リポジトリのルートで実行してください。

```sh
direnv allow
pnpm install --frozen-lockfile
```

以後はディレクトリに入ると、Node.js、pnpm、nixfmtとローカルのCLIが自動で利用できます。
ランタイムのバージョンは`flake.lock`で固定します。
direnvを使わない場合は`nix develop`で同じ環境を起動してください。

## 執筆

```sh
pnpm new:art
pnpm new:book
pnpm preview
```

CLIの詳しい使い方は[Zenn公式ガイド](https://zenn.dev/zenn/articles/zenn-cli-guide)を参照してください。
