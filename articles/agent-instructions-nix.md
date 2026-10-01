---
title: "CLAUDE.md や AGENTS.md を Nix で管理する agent-instructions-nix を作った話"
emoji: "❄️"
type: "tech"
topics: ["nix", "homemanager", "claudecode", "codex"]
published: true
---

Claude CodeやCodexなどでは、全プロジェクトに共通するエージェントへの指示を、所定のファイル（`~/codex/AGENTS.md`や`~/claude/CLAUDE.md`など）に書く必要があります。

https://learn.chatgpt.com/docs/agent-configuration/agents-md

https://code.claude.com/docs/ja/memory#claude-md-files

複数のエージェントを使っていると、共通する指示をそれぞれのファイルで管理するのが面倒です。

¯\\_(ツ)\_/¯

また、自分の中でNixがかなり熱く、MacBookの環境はNixで管理しています。

知人の[@キュレェ](https://x.com/kyremoe)がAgent SkillsをNixで管理するための[agent-skills-nix](https://github.com/Kyure-A/agent-skills-nix)を作っており、実際に自分もこれを使ってスキルをNixで管理しています。

https://github.com/Kyure-A/agent-skills-nix

このモジュールは、ざっくり言うとGitHubなどで公開されているスキルを取得し、Nixのderivationとしてビルドして、各エージェントのスキル用ディレクトリにシンボリックリンクを張ります。

この仕組みの指示ファイル版があったら楽だろうなぁと思ったため、この挙動を参考に、複数のMarkdownファイルをNixで組み立て・配置する[agent-instructions-nix](https://github.com/koutyuke/agent-instructions-nix)を作りました。

https://github.com/koutyuke/agent-instructions-nix

## エージェントに渡す指示を一か所で管理・編集したい

前述のとおり、エージェント別の指示ファイルを用意すると、共通の方針を各ファイルへ繰り返し書くことになります。
「検証していない結果を成功として報告しない」といった方針を変えた場合、各ファイルを更新しないといけず、とても面倒です。

解決方法の一つは、共通の指示ファイルを`~/.agents/AGENTS.md`に置き、各エージェント用のファイルから、このファイルへのシンボリックリンクを張ることです。

```text
~/.agents/AGENTS.md
  ↑
  ├─ ~/.claude/CLAUDE.md
  └─ ~/.codex/AGENTS.md
```

この方法の難点は、エージェント固有の指示を加えにくいことです。

共通する指示は一元管理したいのに、特定の指示は各エージェントごとに管理したい、という無理難題の壁に当たります。

そこで、シンボリックリンクのアイディアはそのままに、複数のファイルを組み合わせて、リンク先のファイルを生成すればよいと考えました。

このような3つのファイルがあるとします。

| ファイル      | 内容                           | 対象               |
| ------------- | ------------------------------ | ------------------ |
| `COMMON.md`   | 返答、作業、検証などの共通方針 | Claude CodeとCodex |
| `CONTEXT7.md` | Context7を利用する手順         | Claude CodeとCodex |
| `CODEX.md`    | サンドボックスや結果報告の指示 | Codexのみ          |

:::details ファイルの中身

```md:COMMON.md
## Communication

- Lead with the conclusion, then give the reasons needed to act on it.
- Keep explanations short and skip filler praise.
- Separate verified facts from assumptions and open questions.

## Workflow

- Read the relevant code, tests, and docs before editing.
- Check `git status` and protect changes you did not make.
- Choose the smallest change that meets the requirements; do not add unrequested abstractions.
- Ask only when a decision belongs to the user and cannot be resolved by investigation.

## Verification

- Run the tests, linters, and type checks that cover the change.
- For bug fixes, reproduce the failure first and confirm the fix afterwards.
- Report failed or skipped checks as they are; never claim unverified results.

## Safety

- Never hard-code secrets or print them in logs or replies.
- Confirm before pushing, deploying, or running destructive commands.
```

````md:CONTEXT7.md
# Context7

Use the Context7 MCP server to fetch current documentation whenever a question involves a library, framework, SDK, API, or CLI tool. Prefer it over web search, and use it even for well-known libraries, since training data may be outdated.

## When not to use it

- Refactoring or debugging business logic
- Writing scripts from scratch
- General programming concepts

## Steps

1. Call `resolve-library-id` with the library name, unless the user gives an ID in `/org/project` form.
2. Pick the best match by name, description, and source reputation.
3. Call `query-docs` with that ID, one concept per query.
4. Answer based on the fetched documentation.

```text
# Headings inside code fences are left as they are.
resolve-library-id "next.js" -> /vercel/next.js
query-docs /vercel/next.js "middleware configuration"
```
````

```md:CODEX.md
---
description: Instructions only for Codex
---

# Codex Instructions

## Sandbox

- Work inside the sandbox and request escalation only when a command needs network or files outside the workspace.
- Prefer `rg` over `grep` for searching.

## Final Report

- Summarize what changed and which files were touched.
- List the checks you ran and their results.
- Mention anything left unverified and what is needed to finish it.

```

:::
これらのファイルを組み合わせて`codex.md`と`claude.md`を生成し、それぞれへのシンボリックリンクを張ります。

```text
codex.md  = COMMON.md + CONTEXT7.md + CODEX.md
  ↑
  └─ ~/.codex/AGENTS.md

claude.md = COMMON.md + CONTEXT7.md
  ↑
  └─ ~/.claude/CLAUDE.md
```

こうすれば、共通する指示は`COMMON.md`で一元管理でき、各エージェント固有の指示はそれぞれのファイルに分けて管理できます。

## Nixを使って管理したい

自分はPCとしてMacBookを使っており、ここに入れるCLIツールやMac自体の設定もNixを使って宣言的に管理しています。

https://github.com/koutyuke/dotfiles

（Nixはいいぞ）

エージェント関連も一部はここで管理しています。
エージェントのインストールには[llm-agents.nix](https://github.com/numtide/llm-agents.nix)を使い、Agent Skillsは[agent-skills-nix](https://github.com/Kyure-A/agent-skills-nix)を使って管理しています。

そのため、先ほどの指示ファイルも、当然Nixを使って管理したくなります。

このような背景があって、[agent-instructions-nix](https://github.com/koutyuke/agent-instructions-nix)を作成しました。

## 使い方

```nix:flake.nix
inputs.agent-instructions = {
  url = "github:koutyuke/agent-instructions-nix";
  inputs.nixpkgs.follows = "nixpkgs";
};
```

```nix:default.nix
{ inputs, ... }:
{
  imports = [
    inputs.agent-instructions.homeManagerModules.default
  ];

  programs.agent-instructions = {
    enable = true;
    sources = [
      ./COMMON.md
      {
        path = ./CONTEXT7.md;
        headingStrategy = "demote";
      }
    ];
    targets = {
      claude.enable = true;
      codex = {
        enable = true;
        dest = ".codex/AGENTS.md";
        sources = [
          {
            path = ./CODEX.md;
            headingStrategy = "drop";
          }
        ];
        inheritSources = true;
        insertH1 = {
          enable = true;
          title = "Codex Instructions";
        }
      };
    };
    insertH1.enable = true;
  };
}
```

`sources`には指示ファイルの場所を指定します。後述する見出しの扱いを変更したい場合は、`headingStrategy`で指定できます。

`targets`で配置先を決めます。
各ターゲットには、配置先を変える`dest`、エージェント固有の指示ファイルの場所を指定する`sources`、共通の指示ファイルを使用するかを切り替える`inheritSources`を指定できます。

`insertH1`は生成する各ターゲットファイルの先頭に挿入するH1見出しの設定です。
`insertH1.title`で挿入する見出しのタイトルを指定できます。

設定を反映すると、生成した指示ファイルはNix storeに保存され、Home Managerによって`~/.claude/CLAUDE.md`などへシンボリックリンクが張られます。

```text
COMMON.md ───────┐
                 │
CONTEXT7.md ───→ +
                 │
CODEX.md ──────→ +
                 │
              (build)
                 ↓
    /nix/store/<hash>/codex.md ←─ ~/.codex/AGENTS.md
```

:::details 生成したファイル

````md:/nix/store/<hash>/claude.md
# CLAUDE.md

## Communication

- Lead with the conclusion, then give the reasons needed to act on it.
- Keep explanations short and skip filler praise.
- Separate verified facts from assumptions and open questions.

## Workflow

- Read the relevant code, tests, and docs before editing.
- Check `git status` and protect changes you did not make.
- Choose the smallest change that meets the requirements; do not add unrequested abstractions.
- Ask only when a decision belongs to the user and cannot be resolved by investigation.

## Verification

- Run the tests, linters, and type checks that cover the change.
- For bug fixes, reproduce the failure first and confirm the fix afterwards.
- Report failed or skipped checks as they are; never claim unverified results.

## Safety

- Never hard-code secrets or print them in logs or replies.
- Confirm before pushing, deploying, or running destructive commands.

## Context7

Use the Context7 MCP server to fetch current documentation whenever a question involves a library, framework, SDK, API, or CLI tool. Prefer it over web search, and use it even for well-known libraries, since training data may be outdated.

### When not to use it

- Refactoring or debugging business logic
- Writing scripts from scratch
- General programming concepts

### Steps

1. Call `resolve-library-id` with the library name, unless the user gives an ID in `/org/project` form.
2. Pick the best match by name, description, and source reputation.
3. Call `query-docs` with that ID, one concept per query.
4. Answer based on the fetched documentation.

```text
# Headings inside code fences are left as they are.
resolve-library-id "next.js" -> /vercel/next.js
query-docs /vercel/next.js "middleware configuration"
```

````

````md:/nix/store/<hash>/codex.md
# AGENTS.md

## Communication

- Lead with the conclusion, then give the reasons needed to act on it.
- Keep explanations short and skip filler praise.
- Separate verified facts from assumptions and open questions.

## Workflow

- Read the relevant code, tests, and docs before editing.
- Check `git status` and protect changes you did not make.
- Choose the smallest change that meets the requirements; do not add unrequested abstractions.
- Ask only when a decision belongs to the user and cannot be resolved by investigation.

## Verification

- Run the tests, linters, and type checks that cover the change.
- For bug fixes, reproduce the failure first and confirm the fix afterwards.
- Report failed or skipped checks as they are; never claim unverified results.

## Safety

- Never hard-code secrets or print them in logs or replies.
- Confirm before pushing, deploying, or running destructive commands.

## Context7

Use the Context7 MCP server to fetch current documentation whenever a question involves a library, framework, SDK, API, or CLI tool. Prefer it over web search, and use it even for well-known libraries, since training data may be outdated.

### When not to use it

- Refactoring or debugging business logic
- Writing scripts from scratch
- General programming concepts

### Steps

1. Call `resolve-library-id` with the library name, unless the user gives an ID in `/org/project` form.
2. Pick the best match by name, description, and source reputation.
3. Call `query-docs` with that ID, one concept per query.
4. Answer based on the fetched documentation.

```text
# Headings inside code fences are left as they are.
resolve-library-id "next.js" -> /vercel/next.js
query-docs /vercel/next.js "middleware configuration"
```

## Sandbox

- Work inside the sandbox and request escalation only when a command needs network or files outside the workspace.
- Prefer `rg` over `grep` for searching.

## Final Report

- Summarize what changed and which files were touched.
- List the checks you ran and their results.
- Mention anything left unverified and what is needed to finish it.

````

:::

## 見出しやfront matterなどを調整する

Markdownをそのまま連結すると、ファイルごとのタイトルがH1として残ります。
そこで、ファイルごとに`headingStrategy`を指定できるようにしました。

| 設定       | 動作                           |
| ---------- | ------------------------------ |
| `"none"`   | 見出しをそのまま使う。既定値   |
| `"demote"` | すべての見出しを1段下げる      |
| `"drop"`   | ファイル先頭のH1だけを削除する |

たとえば`CONTEXT7.md`の`# Context7`は、`"demote"`によって`## Context7`になります。
その下の`## Steps`も`### Steps`に変わります。
コードブロック内の`#`は無視するようにしています。

`CODEX.md`は`# Codex Instructions`から始まるため、`"drop"`でタイトルを外します。
本文や、その下の見出しは残ります。
先頭がH1でないファイルに`"drop"`を指定しても、見出しは削除されません。

front matterは、結合前に取り除きます。
これを残しておくと、区切り線として解釈されるためです。

## おわりに

`agent-skills-nix`を使ったことがある人ならわかると思いますが、設定項目はほとんど同じです ^^;
オプションが似通ってしまったのはご愛嬌ということで...

ぜひ、使っていただけると嬉しいです！

https://github.com/koutyuke/agent-instructions-nix
