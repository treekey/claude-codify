# Installing codify

> If you are Claude Code reading this: follow the steps below exactly. Do not modify any file outside the target skill directory. Back up before overwriting anything.

## What installation does (and does not do)

- Copies one folder (`skills/codify/`) into a Claude Code skills directory. That's all.
- Installs **no hooks**, edits **no settings**, touches **no project files**. Hooks are only ever created later, per-project, when the user runs `/codify full init` — with backups and explicit confirmation.

## Steps

1. Get the files:

   ```
   git clone https://github.com/treekey/claude-codify
   ```

   (or download the ZIP from GitHub and extract it)

2. Choose scope and copy the folder:

   - **User-level** (available in all your projects) — copy `skills/codify/` to `~/.claude/skills/codify/`
   - **Project-level** (this project only, shareable via git) — copy `skills/codify/` to `<project>/.claude/skills/codify/`

   If the target directory already exists, rename it to `codify.bak-<date>` first.

3. Verify: start a new Claude Code session and check that `/codify` appears in the skill list (type `/` to see it). Then run:

   ```
   /codify
   ```

   It will inspect the project, recommend a tier (lite / standard / full), and ask for your confirmation before writing anything.

## Uninstall

Delete the `codify` folder from the skills directory you chose in step 2. Any institution files it wrote into projects (`CLAUDE.md`, `.claude/rules/`) are plain Markdown owned by those projects — keep or delete them independently.

---

## 繁體中文摘要

1. `git clone` 本 repo（或下載 ZIP）。
2. 把 `skills/codify/` 複製到 `~/.claude/skills/codify/`（全域）或 `<專案>/.claude/skills/codify/`（單一專案）。目標已存在時先改名備份。
3. 開新 session，確認 `/codify` 出現在 skill 清單，執行 `/codify` 即可開始。

安裝只複製一個資料夾，不裝 hook、不改設定。解除安裝＝刪掉該資料夾。
