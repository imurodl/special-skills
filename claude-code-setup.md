# Claude Code Power Setup — A Shareable Playbook

Hand this whole file to your Claude Code agent and say: *"Help me set this up, one section at a time. Skip anything I don't need."*

It's a battle-tested config distilled from a working setup. Nothing here contains private data — every server IP, key, or account ID has been replaced with a `<PLACEHOLDER>`. Fill in your own or delete what doesn't apply.

---

## 0. Mental model — how the pieces fit

Claude Code reads config from a few places, in order of scope:

| File | Scope | What goes in it |
|---|---|---|
| `~/CLAUDE.md` | Your machine | Hardware, OS, installed tools, shell/editor setup |
| `~/projects/CLAUDE.md` (or any project root) | A directory tree | Who you are, infra/access, **project rules** |
| `~/.claude/settings.json` | Global, shareable | Model, effort, hooks, statusline, plugins |
| `~/.claude/settings.local.json` | Global, **private** | Secrets, env vars, allowlists — never commit this |

The single highest-leverage thing here is the **two-tier CLAUDE.md pattern** plus a tight **Rules** block. Do that first; everything else is polish.

---

## 1. The two-tier CLAUDE.md pattern ⭐ (do this first)

Claude reads `CLAUDE.md` automatically on every session and treats it as standing instructions. Split it in two.

### 1a. `~/CLAUDE.md` — machine context

A plain-language description of your machine so Claude stops guessing. Template:

```markdown
# System Context for Claude

## Hardware
- **Laptop**: <e.g. MacBook Air M1, 8GB RAM>

## OS & Tools
- **OS**: <macOS / Linux / WSL>
- **Shell**: <zsh + Oh My Zsh, etc.>
- **Terminal**: <Ghostty / iTerm / Windows Terminal>
- **Editor**: <Neovim / VS Code>
- **Package Manager**: <Homebrew / apt>

## Languages & Runtimes
- **Node.js**: <via nvm / fnm>
- **Python 3**: <via Homebrew / pyenv> + **uv** (fast venv/pkg manager)
- **bun**: <if used>

## Dev Tools
- ripgrep, fd, jq, tree, gh (GitHub CLI), lazygit, prettier, ruff, ...
  (list what you actually have — Claude will prefer tools it knows exist)

## Important Paths
- Projects: `~/projects/`
- Configs: `~/.config/`
```

> Tip: the point isn't documentation for humans — it's telling Claude which tools it can reach for so it doesn't propose `grep` when you have `rg`, or `pip` when you have `uv`.

### 1b. `~/projects/CLAUDE.md` — who you are + rules

```markdown
# Projects Context for Claude

## Who
<Your name, role, location. GitHub handle.>

## Access & Infrastructure
<ONLY if useful and you're comfortable. Servers, cloud accounts, CLIs you
have authed (gh, aws, vercel, wrangler...). Keep real secrets OUT — they
belong in a private file, not here.>

## Agent Model Routing
When spawning subagents via the Agent tool:
- Use a small/fast model (e.g. Haiku) for: web searches, file lookups,
  codebase exploration, grep/glob, reading docs, quick fact checks.
- Use the top model (Opus/Sonnet) for: implementation, debugging,
  architecture, code review.

## Rules
- Never add `Co-Authored-By` to git commits
- Never use emojis in code, commits, or docs unless asked
- Never create README or documentation files unless asked
- Don't add comments, docstrings, or type annotations to code you didn't change
- Don't add features or refactoring beyond what's asked
- Be direct and concise — no filler, no end-of-response summaries
- <add your own: preferred package manager, commit style, test framework...>
```

**The `## Rules` block is the single most valuable thing to steal.** It stops the most common annoyances (emoji spam, unrequested READMEs, scope creep, drive-by refactors). Edit it to taste.

---

## 2. `settings.json` — good global defaults

Merge these into `~/.claude/settings.json`. All safe to share; contains no secrets.

```json
{
  "model": "opus[1m]",
  "effortLevel": "high",
  "permissions": { "defaultMode": "auto" },
  "statusLine": {
    "type": "command",
    "command": "node /Users/YOU/.claude/statusline.js"
  },
  "hooks": {
    "Stop": [
      { "matcher": "", "hooks": [
        { "type": "command", "command": "osascript -e 'display notification \"Done\" with title \"Claude Code\" sound name \"Glass\"'" },
        { "type": "command", "command": "printf '\\a'" }
      ]}
    ],
    "Notification": [
      { "matcher": "", "hooks": [
        { "type": "command", "command": "osascript -e 'display notification \"Needs attention\" with title \"Claude Code\" sound name \"Ping\"'" },
        { "type": "command", "command": "printf '\\a'" }
      ]}
    ]
  }
}
```

Notes:
- **`model` / `effortLevel`** — `opus[1m]` is the 1M-context Opus; use whatever your plan allows. `effortLevel: high` = more thorough reasoning.
- **`permissions.defaultMode: "auto"`** — auto-approves tool calls. Convenient but only enable if you trust the workflow; otherwise leave it off and approve manually.
- **Notification hooks** — fire a macOS notification + terminal bell when Claude finishes (`Stop`) or needs you (`Notification`). On **Linux** swap `osascript ...` for `notify-send "Claude Code" "Done"`. On **Windows/WSL** use `powershell.exe -c "..."` or just keep the `printf '\a'` bell.
- Also handy in `settings.local.json`: `"env": { "ENABLE_TOOL_SEARCH": "true" }` to lazy-load tool schemas (saves context).

---

## 3. Custom statusline ⭐ — context-usage meter

Shows `model | directory | ▐████░░░░ 42%` with a color-coded bar (green → yellow → orange → red) as context fills. It accounts for Claude Code's ~16.5% autocompact reserve so the number reflects *usable* context.

Save as `~/.claude/statusline.js`:

```javascript
#!/usr/bin/env node
// Claude Code Statusline — context usage meter + model + directory

const path = require('path');

let input = '';
const timeout = setTimeout(() => process.exit(0), 3000);
process.stdin.setEncoding('utf8');
process.stdin.on('data', chunk => input += chunk);
process.stdin.on('end', () => {
  clearTimeout(timeout);
  try {
    const data = JSON.parse(input);
    const model = data.model?.display_name || 'Claude';
    const dir = path.basename(data.workspace?.current_dir || process.cwd());
    const remaining = data.context_window?.remaining_percentage;

    let ctx = '';
    if (remaining != null) {
      // Claude Code reserves ~16.5% for autocompact buffer
      const BUFFER = 16.5;
      const usable = Math.max(0, ((remaining - BUFFER) / (100 - BUFFER)) * 100);
      const used = Math.max(0, Math.min(100, Math.round(100 - usable)));

      const filled = Math.floor(used / 10);
      const bar = '█'.repeat(filled) + '░'.repeat(10 - filled);

      if (used < 50) {
        ctx = ` \x1b[32m${bar} ${used}%\x1b[0m`;
      } else if (used < 65) {
        ctx = ` \x1b[33m${bar} ${used}%\x1b[0m`;
      } else if (used < 80) {
        ctx = ` \x1b[38;5;208m${bar} ${used}%\x1b[0m`;
      } else {
        ctx = ` \x1b[31m${bar} ${used}%\x1b[0m`;
      }
    }

    process.stdout.write(`\x1b[2m${model}\x1b[0m | \x1b[2m${dir}\x1b[0m${ctx}`);
  } catch (e) {
    // silent fail
  }
});
```

Then point `settings.json` → `statusLine.command` at it (see §2). Replace `/Users/YOU/` with your real home path.

---

## 4. Official plugins (one install each)

Run `/plugin` inside Claude Code, add the `claude-plugins-official` marketplace, and install:

- **`code-review`** — structured diff review for correctness + cleanup
- **`security-guidance`** — security review of pending changes
- **`frontend-design`** — better UI/frontend output
- **`vercel`** — Vercel/Next.js deploy + best-practice tooling (skip if not using Vercel)

---

## 5. Skills worth having

Skills are reusable capability modules Claude auto-invokes. The stock ones (`docx`, `pdf`, `pptx`, `xlsx`, `dataviz`, `artifact-design`) already ship with Claude Code — nothing to install.

The high-value **add-on set** is Matt Pocock's engineering skill suite. Ask your agent to find and install them (or use `/find-skills`):

- **`diagnose`** — disciplined bug/perf debugging loop (reproduce → minimize → fix → regression-test)
- **`tdd`** — red-green-refactor test-first workflow
- **`grill-me` / `grill-with-docs`** — makes Claude interrogate *your* plan until it's airtight
- **`handoff`** — compact the session into a handoff doc for a fresh agent
- **`zoom-out`** — "map this area of code + its callers" when you're lost
- **`to-issues` / `to-prd` / `triage`** — turn plans into tracked issues
- **`improve-codebase-architecture`** — find refactoring/deepening opportunities
- **`setup-matt-pocock-skills`** — run once per repo to wire the above into that project's issue tracker + docs

Also nice: **`caveman`** (ultra-compressed replies to save tokens), **`write-a-skill` / `skill-creator`** (build your own).

Install tool-specific skills only if you use the tool: `notebooklm`, `obsidian-vault`, `slides-grab*`.

> **Slash-invoke anything.** Any skill can be triggered explicitly with `/<skill-name>` (e.g. `/diagnose`, `/caveman`), not just by auto-detection. Handy when you want to force a specific workflow.

---

## 6. Advanced / optional — Headroom context compression

For heavy users burning context fast: **Headroom** runs a local proxy that compresses conversation context on the fly to stretch how far a session goes.

Rough setup (do this last, only if you feel context pressure):
1. `uv tool install headroom-ai` (gives you a `headroom` CLI)
2. Add its marketplace + enable the `headroom` plugin via `/plugin`
3. Run the proxy: `headroom proxy --host 127.0.0.1 --port 8787 --mode token --backend anthropic`
4. In `settings.local.json`: `"env": { "ANTHROPIC_BASE_URL": "http://127.0.0.1:8787" }`
5. Add its `SessionStart` + `PreToolUse` hooks (the plugin documents these)

This is the most involved piece and easy to get wrong — treat it as a "graduate to it later" item, not a day-one setup.

---

## 7. Persistent memory ⭐ — teach Claude once, remember forever

Claude Code keeps durable facts across sessions in a `memory/` folder indexed by a `MEMORY.md` file that loads every session. You don't install anything — but knowing the **format** turns this from a toy into a real second brain.

**One fact per file.** Each memory file uses frontmatter + a short body:

```markdown
---
name: <short-kebab-case-slug>
description: <one-line summary — used to decide relevance during recall>
metadata:
  type: user | feedback | project | reference
---

<the fact. For feedback/project, add **Why:** and **How to apply:** lines.
Link related memories with [[their-slug]].>
```

The four `type`s, and when to use each:

| type | Stores | Example |
|---|---|---|
| `user` | Who you are — role, stack, preferences | "Prefers `bun` over `npm`; ships Next.js + NestJS." |
| `feedback` | Corrections / confirmed ways of working (include the *why*) | "Don't summarize at the end of responses. **Why:** noise. **How to apply:** stop after the answer." |
| `project` | Ongoing work, goals, constraints not in the code | "Migrating prod to a new host by <date>; old one decommissioned." |
| `reference` | Pointers to external stuff (URLs, dashboards, tickets) | "Deploy dashboard: <url>" |

**The index (`MEMORY.md`)** holds only one line per memory — a title, a link, and a hook — never the content itself:

```markdown
# Memory index
- [Deploy workflow](deploy-workflow.md) — pm2 + nginx + certbot on the prod box
```

Rules that make it work well:
- **Convert relative dates to absolute** before saving ("next Friday" → the actual date).
- **Don't save what the repo already records** (code structure, past fixes, git history, existing CLAUDE.md).
- **Link liberally** with `[[slug]]` — even to memories you haven't written yet; it marks them as worth writing.
- **Update, don't duplicate** — if a fact changes, edit the existing file; delete memories that turn out wrong.

To use it, just say *"remember that ..."* and let Claude write the file — or, better, tell it the workflow you keep repeating and let it capture that as a `feedback` memory.

---

## 8. Hooks beyond notifications

Hooks run shell commands on lifecycle events. Notifications (§2) are the friendly intro; the powerful uses are **guardrails** and **session bootstrap**.

- **`PreToolUse`** — inspect/gate a tool call before it runs. Pair with the **`git-guardrails`** skill to block destructive git (`push --force`, `reset --hard`, `clean -fd`, `branch -D`) before it executes.
- **`SessionStart`** (matcher `startup|resume`) — run setup when a session opens: warm a cache, print a repo cheat-sheet, start a proxy, `git fetch`.
- **`Stop` / `Notification`** — desktop alerts so you can walk away from long runs (already covered).

Keep hook commands fast and `timeout`-bounded — they run inline and a slow hook stalls the turn.

---

## 9. Fewer permission prompts (`settings.local.json`)

Auto mode aside, you can pre-approve safe, read-only commands so Claude stops asking. In your **private** `settings.local.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(gh auth:*)",
      "Bash(gh api:*)",
      "Bash(gh repo:*)",
      "Bash(brew install:*)",
      "Bash(npm install:*)",
      "WebSearch",
      "WebFetch(domain:github.com)"
    ]
  }
}
```

Add only commands you're comfortable running unattended. There's also a **`/fewer-permission-prompts`** helper that scans your history and proposes a tailored allowlist.

---

## 10. Daily workflow habits (the part config can't do)

The setup is only half of it. Habits that pay off:

- **Plan before big changes.** Use plan mode (or the `Plan` agent) for anything multi-file; approve the plan, *then* let it build.
- **Delegate lookups to subagents.** "Find where X is handled" → spawn an `Explore`/Haiku agent so the file dumps stay out of your main context. Cheaper and keeps your window clean.
- **`/code-review` before you commit.** Catches correctness + cleanup issues on your working diff.
- **`/security-review`** on anything touching auth, uploads, or external input.
- **`/handoff` when context gets full** (watch the statusline bar) — compacts the session so a fresh window picks up cleanly.
- **`/caveman` when you're burning tokens** on a long session and don't need prose.
- **Let it remember.** When you correct Claude the same way twice, tell it to save that as a `feedback` memory (§7) so you never repeat it.

---

## Suggested order to actually do this

1. Write your two `CLAUDE.md` files — copy the **Rules** block, fill in your own machine/infra. *(biggest win)*
2. Drop in `statusline.js` + the `settings.json` block (model, effort, statusline, notification hooks).
3. `/plugin` install the official plugins you'll use.
4. Install the Matt Pocock skill suite; run `setup-matt-pocock-skills` in a repo.
5. Learn the memory format (§7) — it compounds over weeks.
6. Add guardrail hooks + a permission allowlist once you know your common commands.
7. Later, if needed: Headroom.

Start at step 1 and stop whenever you've got enough. Everything after the CLAUDE.md files is optional polish.
