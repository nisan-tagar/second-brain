# Second Brain Setup & Workflow

**Created:** 2026-04-07  
**Author:** Nisan Tagar

---

## Overview

This document captures the full setup and workflow for the second-brain system — a GitHub-backed Obsidian vault accessible from desktop, WSL2, Claude Code, and Android mobile.

---

## Architecture

```
Obsidian (Windows)
  ↕ Obsidian Git plugin (auto-sync every 10 min)
    GitHub → nisan-tagar/second-brain (private repo)
      ↕ Claude Code + GitHub MCP (WSL2)
      ↕ MGit (Android)
      ↕ Claude Web/Mobile (read via raw URL, write via paste → MGit push)
```

---

## Vault Location

|Environment|Path|
|---|---|
|Windows (Obsidian)|`C:\Users\tnisan\projects\second-brain`|
|WSL2 (Claude Code / git ops)|`~/projects/second-brain`|
|GitHub (source of truth)|`https://github.com/nisan-tagar/second-brain`|
|Android (MGit)|`/storage/emulated/0/MGit/second-brain`|

---

## Vault Structure

```
second-brain/
  Business/
    PROJECT_NAME/
      PRD.md
      tech-design.md
      decisions/
  Career/
  Goals/
  Health/
  Templates/
  .gitignore
```

---

## One-Time Setup

### 1. GitHub Repo

- Private repo: `nisan-tagar/second-brain`
- Fine-grained PAT with `Contents: Read and Write` + `Metadata: Read` permissions
- Store PAT securely in password manager

### 2. WSL2 — Git Setup

```bash
# Clone the vault
mkdir -p ~/projects
cd ~/projects
git clone https://github.com/nisan-tagar/second-brain.git

# Cache credentials
git config --global credential.helper store
```

### 3. Windows — Obsidian Setup

```powershell
# Clone on Windows side for Obsidian
cd C:\Users\tnisan\projects
git clone https://github.com/nisan-tagar/second-brain.git
```

Then in Obsidian:

- Open vault as folder → `C:\Users\tnisan\projects\second-brain`
- Settings → Community Plugins → install **Git by Vinzent**
- Configure plugin:

|Setting|Value|
|---|---|
|Pull on startup|✅|
|Auto commit interval|10 min|
|Auto push after commit|✅|
|Commit message|`vault: auto-sync {{date}}`|

### 4. Claude Code + GitHub MCP (WSL2)

**Install Claude Code:**

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
npm install -g @anthropic-ai/claude-code
claude  # login via browser OAuth
```

**Add GitHub MCP:**

```bash
claude mcp add github \
  -s user \
  -e GITHUB_PERSONAL_ACCESS_TOKEN=YOUR_PAT \
  -- npx -y @modelcontextprotocol/server-github
```

**Verify:**

```bash
cd ~/projects/second-brain
claude
# Then type: list files in nisan-tagar/second-brain
```

### 5. Android — MGit Setup

- Install MGit from **F-Droid** (not Play Store — Play version is broken)
    - `https://f-droid.org/packages/com.manichord.mgit`
- MGit → Settings → Root Directory → set to `/storage/emulated/0/Documents/`
- Clone: `https://github.com/nisan-tagar/second-brain.git`
- Open vault in Obsidian mobile → Open folder → navigate to MGit clone

**Install Obsidian Git on mobile separately** (plugins don't sync between devices):

- Settings → Community Plugins → Browse → Git by Vinzent → Install

---

## Daily Workflows

### Desktop — Claude Code Editing Docs

```
1. Open WSL2 terminal
2. cd ~/projects/second-brain
3. claude
4. "Read Business/JOURN/PRD.md and update section X"
5. Claude reads, edits, commits directly via GitHub MCP
6. Obsidian auto-pulls on next sync cycle
```

### Desktop — Manual Obsidian Edit

```
1. Edit note in Obsidian
2. Obsidian Git auto-commits and pushes every 10 min
   OR: Ctrl+P → "Obsidian Git: Commit-and-sync"
```

### Mobile — Claude Chat → Vault

```
1. Chat with Claude on mobile app or claude.ai
2. Copy Claude's output
3. Paste into Obsidian mobile note
4. Switch to MGit → Push
5. Changes are now live in GitHub → available to all devices
```

### Mobile — MGit Sync

```
Pull (get latest): MGit → second-brain → ↓ pull
Push (send changes): MGit → second-brain → ↑ push
```

### Reading Vault Files from Claude Web/Mobile

Paste a raw GitHub URL into Claude chat:

```
https://raw.githubusercontent.com/nisan-tagar/second-brain/main/Business/JOURN/PRD.md
```

Claude fetches and reads the full file. Edit output → copy → paste to Obsidian → MGit push.

---

## Project Integration

Each project repo should have a `CLAUDE.md` pointing to the second-brain docs:

```markdown
# Project Intelligence

## Documentation
All project docs live in the second-brain repo:
- Repo: nisan-tagar/second-brain
- Path: Business/PROJECT_NAME/

Always read docs from there before starting any task.
Always write doc updates back there after completing tasks.
```

---

## Capability Matrix

|Surface|Read Vault|Write Vault|
|---|---|---|
|Claude Code (WSL2 + MCP)|✅ Direct|✅ Direct commit|
|Obsidian Windows|✅ Auto-sync|✅ Auto-sync|
|Claude Web / Mobile|✅ Via raw URL|⚠️ Paste → MGit push|
|Obsidian Android (MGit)|✅ MGit pull|✅ MGit push|

---

## Key Files & Credentials

|Item|Location|
|---|---|
|GitHub PAT|Password manager|
|MCP config|`/home/tnisan/.claude.json`|
|Obsidian Git config|Obsidian Settings → Git plugin|
|WSL2 git credentials|`~/.git-credentials`|

---

## Troubleshooting

**Obsidian Git not pushing:**

```bash
# On Windows side
cd C:\Users\tnisan\projects\second-brain
git pull
git push
# Enter PAT as password to re-cache credentials
```

**Claude Code MCP failed:**

```bash
claude mcp remove github
claude mcp add github \
  -s user \
  -e GITHUB_PERSONAL_ACCESS_TOKEN=YOUR_PAT \
  -- npx -y @modelcontextprotocol/server-github
```

**MGit sync conflict:**

- Always pull before editing on mobile
- If conflict: MGit → repo → reset to remote (loses local changes)

**WSL2 path issues:**

- Never store vault in WSL2 filesystem for Obsidian — file watcher breaks
- Keep Windows clone at `C:\Users\tnisan\projects\second-brain` for Obsidian
- Keep WSL2 clone at `~/projects/second-brain` for Claude Code / git ops