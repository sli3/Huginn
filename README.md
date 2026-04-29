# Huginn — OpenCode Local LLM Workflow Template

A reusable agentic coding workflow template for [OpenCode](https://opencode.ai) backed by a local [llama.cpp](https://github.com/ggerganov/llama.cpp) inference server. Drop this into any project to get a structured, safe, session-aware AI coding workflow — without sending your code to a cloud API.

> Named after Huginn, one of Odin's ravens — representing *thought*.

<img width="1584" height="672" alt="Huginn" src="https://github.com/user-attachments/assets/8aa634b8-5671-4c30-86e4-a78f8e278cca" />

---

## Table of Contents

1. [What This Is](#1-what-this-is)
2. [Prerequisites](#2-prerequisites)
3. [Quick Setup](#3-quick-setup)
4. [The 4-Session Workflow](#4-the-4-session-workflow)
5. [Skills Reference](#5-skills-reference)
6. [Session Memo Guide](#6-session-memo-guide)
7. [OpenCode Keyboard Shortcuts](#7-opencode-keyboard-shortcuts)
8. [Token Management Tips](#8-token-management-tips)
9. [Customising for Your Project](#9-customising-for-your-project)
10. [Common Problems & Fixes](#10-common-problems--fixes)

---

## 1. What This Is

This template gives you a structured, repeatable workflow for using an AI coding agent locally — with no cloud API keys, no data leaving your machine, and no runaway edits.

It is built around four principles:

- **Safety gates** — the agent must show its plan and wait for your approval before editing anything
- **Scope discipline** — one change per session, no unrelated fixes
- **Session memory** — timestamped memos keep context across sessions even when the context window resets
- **Local inference** — all LLM calls go to your own llama.cpp server

The template is model-agnostic. Any model compatible with the OpenAI-compatible API (e.g. Qwen, Mistral, Llama, DeepSeek Coder) will work.

---

## 2. Prerequisites

| Tool | Purpose | Install |
|------|---------|---------|
| [OpenCode](https://opencode.ai) | AI coding agent TUI | `npm install -g opencode-ai` |
| [llama.cpp](https://github.com/ggerganov/llama.cpp) | Local LLM inference server | See llama.cpp docs |
| Node.js ≥ 18 | Required by OpenCode | [nodejs.org](https://nodejs.org) |
| npx | MCP server runner (bundled with npm) | Comes with Node.js |
| shellcheck *(optional)* | Used by `code-sanity-check` | `apt install shellcheck` |

### Verify your setup

```bash
opencode --version
node --version      # should be ≥ 18
npx --version
```

### Start your llama.cpp server

```bash
./llama-server \
  --model /path/to/your/model.gguf \
  --host 0.0.0.0 \
  --port 8080 \
  --ctx-size 8192
```

Note your server IP and port — you'll need them in `opencode.json`.

---

## 3. Quick Setup

### Step 1 — Copy the template into your project

```bash
# From your project root
cp -r /path/to/Huginn/.opencode ./
cp /path/to/Huginn/opencode.json ./
cp /path/to/Huginn/.gitignore ./   # or append if you have one
mkdir -p .session-memos
```

Or clone this repo and copy the files manually.

### Step 2 — Configure `opencode.json`

Open `opencode.json` and replace every `[PLACEHOLDER]`:

| Placeholder | Replace with |
|-------------|-------------|
| `[SERVER-IP]` | Your llama.cpp server IP (e.g. `192.168.1.100`) |
| `[PORT]` | Your server port (e.g. `8080`) |
| `[API-KEY]` | Any string — llama.cpp does not validate it (e.g. `local`) |
| `[DEFAULT-MODEL]` | The model ID you want as default |
| `[MODEL-ID-1]` | Your model's identifier (check llama.cpp `/v1/models`) |
| `[Model Display Name]` | Friendly name shown in OpenCode |
| `[ABSOLUTE-PATH-TO-PROJECT]` | Full path to your project root |

### Step 3 — Configure `agent.md`

Open `.opencode/agent.md` and fill in:

| Placeholder | Replace with |
|-------------|-------------|
| `[PROJECT NAME]` | Your project name |
| `[ROLE]` | The agent's role (e.g. `Bash developer`, `Python engineer`) |
| `[ONE LINE PROJECT DESCRIPTION]` | What the project does |
| `[FILENAME]` | Your main script or entry point |
| `[LIST MAIN TASKS]` | What the project does, comma-separated |
| `[TARGET ENVIRONMENT]` | e.g. `Ubuntu 24.04 server`, `Raspberry Pi 5` |
| `[REPO URL]` | Your Git remote URL |

### Step 4 — Verify MCP is working

```bash
opencode
```

In the OpenCode TUI, check that the `local-files` MCP server connects. You should see it listed in the session header or in the tools panel.

### Step 5 — Run your first Explore session

Use the prompt template from [Section 4](#4-the-4-session-workflow).

---

## 4. The 4-Session Workflow

Each project phase uses a different session type. Keep sessions short and focused — one goal per session.

---

### Session 1: Explore

**Goal:** Understand the codebase before touching anything.

**Opening prompt:**
```
This is an Explore session. Read [FILENAME] top to bottom.
List every function, what it does, and flag any issues you notice.
Do not suggest changes yet — just map the territory.
```

**What to expect:**
- The agent reads the file and lists every function with a one-line description
- Issues are flagged with a priority level (High / Medium / Low)
- No edits are made

**Close the session:**
```
memo, this was an Explore session
```

---

### Session 2: Plan

**Goal:** Decide what to change and why, before writing any code.

**Opening prompt:**
```
This is a Plan session. Read the most recent session memo.
Based on the Explore findings, I want to tackle [ISSUE/FEATURE].
Propose an approach — pros, cons, and what files will be affected.
Do not write any code yet.
```

**What to expect:**
- The agent proposes an approach with trade-offs
- You discuss and decide on the implementation strategy
- A clear "Agreed Next Step" is established

**Close the session:**
```
memo, this was a Plan session
```

---

### Session 3: Code

**Goal:** Make exactly one agreed change.

**Opening prompt:**
```
This is a Code session. Run the code-preflight skill now.
```

**What to expect:**
1. Agent reads the latest memo
2. Agent confirms scope and shows the plan
3. You say "OK"
4. Agent makes the edit
5. Agent reminds you to run sanity check and memo

**After the edit:**
```
Run the code-sanity-check skill on [FILENAME]
```

Then commit:
```
Run the git-workflow skill
```

**Close the session:**
```
memo, this was a Code session
```

---

### Session 4: Review

**Goal:** Verify the change works as intended and look for regressions.

**Opening prompt:**
```
This is a Review session. Read the most recent session memo.
Check [FILENAME] and confirm the change from the last Code session
is correct. Look for any regressions or unintended side effects.
```

**What to expect:**
- Agent reads the file and the memo
- Confirms the change is in place and correct
- Flags any regressions or issues
- No edits unless you explicitly ask

**Close the session:**
```
memo, this was a Review session
```

---

## 5. Skills Reference

Skills live in `.opencode/skills/`. OpenCode loads them automatically at startup.

| Skill | Trigger | What it does |
|-------|---------|--------------|
| `session-memo` | `"memo"`, `"save session"`, `"end session"` | Summarises the session and saves a timestamped `.md` file to `.session-memos/` |
| `code-preflight` | `"run preflight"`, or automatic at Code session start | Reads the last memo, reviews previous mistakes, confirms scope, shows plan — waits for OK |
| `code-sanity-check` | `"sanity check"`, `"check this script"` | Runs shellcheck, checks logic, idempotency, and flags suspicious flags |
| `git-workflow` | `"commit"`, `"push"`, `"save to git"` | Staged file review → diff gate → UK English commit → push gate |
| `bash-style` | Applied automatically during script writes/edits | Enforces shebang, `set -euo pipefail`, quoting, snake_case functions, stderr for errors |

### Triggering skills manually

Type the trigger phrase in the OpenCode chat:

```
Run the code-preflight skill
```
```
Run the code-sanity-check skill on setup.sh
```
```
Run the git-workflow skill
```
```
memo, this was a Code session
```

---

## 6. Session Memo Guide

### Where memos are saved

All memos save to `.session-memos/` in your project root.  
Filename format: `YYYY-MM-DD_HH-MM.md`

Example: `.session-memos/2025-07-14_09-32.md`

### What goes in a memo

| Section | Purpose |
|---------|---------|
| **Type** | Explore / Plan / Code / Review / Mixed |
| **What We Did** | 2–3 bullet points summary |
| **Functions Found** | Explore sessions only — every function listed |
| **Issues Found** | Explore sessions only — every issue with priority |
| **Agreed Next Step** | Explore sessions only — what was decided |
| **Files Touched** | Which files changed and how |
| **Decisions Made** | What was chosen and what was rejected |
| **Mistakes Made** | What went wrong, with category |
| **Not Finished** | Up to 3 next steps |
| **Next Session Starter** | Ready-to-paste opening for the next session |

### Reading the latest memo

The agent will do this automatically during preflight, but you can also ask:

```
Read the most recent session memo
```

The agent runs `ls -t .session-memos/*.md | head -1` to find the right file.

### Why `.session-memos/` is in `.gitignore`

Memos are working notes — they contain draft thinking, mistakes, and rejected approaches. They are not part of your codebase. Keep them local.

If you want to archive them, copy them elsewhere or remove `.session-memos/` from `.gitignore`.

### Adding to the Mistakes section manually

After saving a memo, open it in your editor and add any mistakes you noticed that the agent didn't catch. This is valuable — the preflight skill reads these before every Code session.

---

## 7. OpenCode Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+C` | Cancel current generation |
| `Ctrl+L` | Clear the chat view |
| `Ctrl+N` | New session |
| `Ctrl+K` | Open model/config picker |
| `Esc` | Focus input box |
| `↑` / `↓` | Scroll through chat history |
| `Tab` | Autocomplete file paths in input |

> **Tip:** When context is getting full (OpenCode shows a warning), start a new session and paste the **Next Session Starter** from your latest memo.

---

## 8. Token Management Tips

Local models have smaller context windows than cloud APIs. Keep usage low with these habits:

### Keep sessions short
One goal per session. When the goal is done, save the memo and start fresh.

### Use the preflight skill instead of re-explaining
The preflight skill reads the memo so you don't have to re-paste context at the start of every session.

### Read functions, not whole files
When the agent needs context, ask it to read only the relevant function:
```
Read only the [function_name] function in [file.sh]
```

### Use compaction
`opencode.json` has `"compaction": { "enabled": true, "threshold": 0.85 }`. This auto-compacts the context when it reaches 85% full. Keep this enabled.

### Watch the context indicator
OpenCode shows a token usage bar. When it hits ~70%, finish the current task, save the memo, and start a new session.

### Avoid pasting large files into chat
Use the MCP `local-files` server instead — the agent can read files directly without them consuming your context window.

---

## 9. Customising for Your Project

### `agent.md` — what to change

| Section | What to customise |
|---------|------------------|
| **Role** | Change `[ROLE]` to match your stack (e.g. `Python developer`, `C systems programmer`) |
| **Project Context** | Fill in your real filenames, tasks, target environment, and repo URL |
| **Safety Gates** | Add project-specific rules (e.g. "never restart the production service without asking") |
| **Thinking Mode** | If your model does not support `/no_think`, remove that line |

### `opencode.json` — what to change

| Key | What to set |
|-----|------------|
| `model` | Change to your preferred default model ID |
| `baseURL` | Your llama.cpp server address |
| `models` | Add/remove model entries as needed |
| `max_tokens` | Adjust per model — check your model's context size |
| `mcp.local-files.command` | Set the absolute path to your project root |
| `compaction.threshold` | Lower to `0.75` if your model has a small context window |

### Adding a new skill

1. Create a new folder in `.opencode/skills/[skill-name]/`
2. Add a `SKILL.md` with:
   - `name:` and `description:` at the top
   - A clear **Trigger** section
   - Numbered **Steps**
   - A **Rules** section
3. OpenCode will auto-load it at next startup.

### Removing skills you don't need

Simply delete the skill folder. The remaining skills are unaffected.

### Using a cloud model instead of local

Replace the `provider` block in `opencode.json`:

```json
"provider": {
  "anthropic": {
    "apiKey": "your-api-key-here"
  }
},
"model": "anthropic/claude-sonnet-4-20250514"
```

The skills and `agent.md` work identically with any model.

---

## 10. Common Problems & Fixes

### The agent edited something it wasn't supposed to

**Why it happens:** Scope creep — the agent noticed something "easy to fix" and changed it.

**Fix:**
1. Run `git diff` to see exactly what changed
2. Revert the unintended change: `git checkout [file]`
3. Add a Mistakes entry to the session memo: `Scope Creep — changed [X] when only [Y] was authorised`
4. The preflight skill will read this mistake before the next Code session

**Prevention:** The preflight skill and Edit Rules in `agent.md` both reinforce single-change discipline. If it happens repeatedly, add a project-specific rule to `agent.md`.

---

### The agent said "no changes to apply" but nothing was fixed

**Why it happens:** The agent tried to edit a line that had already changed, or it targeted the wrong line.

**Fix:**
1. Ask the agent to read the current file state: `Read [function] in [file] now`
2. Ask it to show a fresh plan based on what it actually sees
3. Never ask it to retry the same edit — this triggers a loop

---

### Context runs out mid-session

**Why it happens:** Long files, repeated reads, or large diffs consumed the window.

**Fix:**
1. Say: `Stop. Save the session memo now.`
2. Start a new session
3. Paste the **Next Session Starter** from the memo
4. Run preflight again

**Prevention:** Save a memo every 20–30 minutes. Watch the context indicator.

---

### The MCP `local-files` server won't connect

**Why it happens:** The absolute path in `opencode.json` is wrong, or `npx` can't reach the registry.

**Fix:**
1. Confirm the path: `echo $PWD` — use this exact output in `opencode.json`
2. Pre-download the MCP package: `npx @modelcontextprotocol/server-filesystem --help`
3. Check Node.js version: `node --version` — must be ≥ 18

---

### The model ignores the skills

**Why it happens:** The model does not follow instructions reliably, or the skill trigger phrase was not recognised.

**Fix:**
1. Be explicit: `Run the code-preflight skill now — follow every step in order`
2. Check that the skill folder name and `SKILL.md` filename are correct
3. Restart OpenCode — skills are loaded at startup

---

### Commits are going in with the wrong format

**Why it happens:** The `git-workflow` skill was not triggered, or the agent committed directly.

**Fix:**
1. `git log --oneline` to review recent commits
2. `git commit --amend -m "fix: corrected log rotation path"` to fix the last commit message
3. Always trigger `git-workflow` explicitly — never let the agent commit silently

---

### The agent is adding `//` comments to Bash scripts

**Why it happens:** The model confused Bash with another language.

**Fix:** Remind it: `Use # for Bash comments — not //`  
Add this to `agent.md` under Edit Rules if it happens repeatedly.

---

## Appendix: Recommended Model Settings

These settings work well for coding tasks on llama.cpp. Adjust to taste.

| Setting | Recommended value |
|---------|-----------------|
| Context size (`--ctx-size`) | 8192 minimum; 16384+ preferred |
| Temperature | 0.2–0.4 for code; 0.7 for exploration |
| Repeat penalty | 1.1 |
| GPU layers (`--n-gpu-layers`) | As many as your VRAM allows |
| Threads (`--threads`) | Half your physical core count |

---

## Appendix: Session Type Quick Reference

| Type | You want to... | Agent does... | Edits files? |
|------|---------------|---------------|-------------|
| Explore | Understand the code | Maps functions, flags issues | Never |
| Plan | Decide on an approach | Proposes strategy, pros/cons | Never |
| Code | Make a change | Pre-flight → edit → sanity check | Yes, one change |
| Review | Verify a change | Reads file, checks for regressions | Only if asked |

---

*Template version 1.0 — built for OpenCode with llama.cpp local inference.*
