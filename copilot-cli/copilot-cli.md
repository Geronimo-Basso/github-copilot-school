# Lab 03 — GitHub Copilot CLI

Terminal-native AI assistant that runs in bash/zsh/PowerShell. Use it interactively for server-side code reviews, programmatically for CI/CD automation, and customize it with agents, plugins, skills, and hooks that extend VS Code's Copilot surface right into your shell.

## Table of Contents

- [What You'll Learn](#what-youll-learn)
- [Pre-Lab Setup](#pre-lab-setup)
- [Part 1 — Theory](#part-1--theory)
  - [1.1 What Copilot CLI is](#11-what-copilot-cli-is)
  - [1.2 When to use CLI vs VS Code](#12-when-to-use-cli-vs-vs-code)
  - [1.3 Interactive vs programmatic](#13-interactive-vs-programmatic)
  - [1.4 Slash commands overview](#14-slash-commands-overview)
- [Part 2 — Core CLI Usage](#part-2--core-cli-usage)
  - [Exercise A — Install + auth + hello world](#exercise-a--install--auth--hello-world)
  - [Exercise B — Context management](#exercise-b--context-management)
  - [Exercise C — Slash commands deep dive](#exercise-c--slash-commands-deep-dive)
  - [Exercise D — Permission flags & patterns](#exercise-d--permission-flags--patterns)
  - [Exercise E — Multi-turn autonomous task](#exercise-e--multi-turn-autonomous-task)
- [Part 3 — Customization](#part-3--customization)
  - [Custom instructions — CLI paths](#custom-instructions--cli-paths)
  - [Custom agents](#custom-agents)
  - [Hooks (read-through)](#hooks-read-through)
  - [Skills — CLI invocation](#skills--cli-invocation)
  - [Copilot memory via /chronicle](#copilot-memory-via-chronicle)
  - [CLI plugins](#cli-plugins)
  - [MCP — CLI config paths (read-through)](#mcp--cli-config-paths-read-through)
- [Part 4 — Programmatic & Automation](#part-4--programmatic--automation)
  - [Exercise F — Headless invocation](#exercise-f--headless-invocation)
- [Q&A](#qa)
- [What's Next?](#whats-next)

## What You'll Learn

By the end of this lab you will be able to:

- **Use Copilot CLI interactively and programmatically** — from interactive sessions with slash commands to headless one-shot invocations with JSON output.
- **Master context management** — use `@` mentions, `/context`, `/compact`, and `/clear` to control what the model sees and how much of your token budget is left.
- **Apply permission patterns** — allowlist/denylist tools and shell commands with patterns like `shell(git push)`, `shell(git:*)`, `MyMCP(tool_name)`, and `url(domain)`.
- **Customize the CLI** — add custom agents (`.agent.md`), invoke skills via `@skill:name`, explore experimental memory with `/chronicle`, and install plugins from marketplaces.
- **Script headless workflows** — use `copilot -p` for CI/CD pipelines, automated PR reviews, and scheduled codebase audits.

**Core differentiation:** Terminal-native AI assistant with scripting + plugin capabilities VS Code cannot provide.

---

## Pre-Lab Setup

Get the boring stuff out of the way before the clock starts.

### Required ✅

1. **Open this repository in your terminal.** All commands assume you're starting from the workspace root (`github-copilot-101/`).

1. **Check that you have one of these package managers installed:**

   | Platform | Command to verify |
   |----------|-------------------|
   | macOS (Homebrew) | `brew --version` |
   | npm (any platform) | `npm --version` |
   | Windows (winget) | `winget --version` |

   If none of these are available, install Node.js (which includes npm) from [nodejs.org](https://nodejs.org/).

---

## Part 1 — Theory

Read straight through Part 1 before touching the keyboard. Every exercise in Part 2 assumes the vocabulary below.

### 1.1 What Copilot CLI is

Copilot CLI is a **terminal-native AI assistant** that runs in bash, zsh, or PowerShell. It's the same Copilot model you use in VS Code, but packaged for command-line workflows.

**Key properties:**
- Runs in your terminal (no IDE required)
- Device-flow authentication on first run (opens browser to authenticate)
- Three modes:
  - **`interactive`** — interactive session (default when you run `copilot`)
  - **`plan`** — generates a plan without executing (for review before action)
  - **`autopilot`** — autonomous multi-turn execution (keeps looping until done)

### 1.2 When to use CLI vs VS Code

**Reach for CLI when:**
- SSH'd into a server (no IDE available)
- CI/CD pipelines
- Repo-wide refactors driven from shell
- Quick one-off tasks already in terminal

**Reach for VS Code when:**
- Multi-file inline editing with visual feedback
- Deep debugging with integrated tools
- Workspace agents tied to `.github/agents/`
- Real-time inline completions

> 📌 **Key distinction:** The CLI is for **automation and headless workflows**. VS Code is for **interactive editing with visual feedback**.

### 1.3 Interactive vs programmatic

```bash
# Interactive (default mode)
copilot

# Programmatic (one-shot, exits when done)
copilot -p "Generate tests for @backend/app.py"
```

**Interactive mode:**
- Opens an interactive session that stays open
- Accepts multiple prompts in a conversation
- Slash commands available (`/plan`, `/clear`, `/context`)
- Type `exit` or `Ctrl+C` to quit

**Programmatic mode:**
- One prompt, exits when done
- Output goes to stdout (no interactive session)
- Perfect for CI/CD automation

### 1.4 Slash commands overview

These are the CLI-specific commands you can use in interactive mode. We'll explore them in depth in Part 2 Exercise C.

**Core commands:**
- `/plan` — create an implementation plan (doesn't execute)
  > 📌 **Note:** `/plan` is a slash command for one-off planning. Distinct from **plan mode** (a persistent session state toggled with `--plan` flag or Shift+Tab). See the [Q&A section](#qa) for the full distinction.
- `/yolo` — allow all tools without approval prompts
- `/fleet` — parallel subagent execution
- `/research` — deep investigation (GitHub + web search)
- `/delegate` — send session to GitHub cloud agent → creates a PR
- `/chronicle` — session insights (**EXPERIMENTAL** — requires `/experimental on`)
- `/skills` — list/info/add/remove/reload skills

**Context management:**
- `/context` — show token usage and visualization
- `/compact` — summarize history to free context
- `/clear` — start new conversation (alias: `/new`)
- `/undo` — rewind last turn (revert file changes)

> 📌 **Note:** There is **no** `/reset` or `/stop` command. Use `/clear` to start fresh, `/undo` to revert the last turn, or `/compact` to free up context space.

---

## Part 2 — Core CLI Usage

Now that the vocabulary is in place, run Exercises A–E in order. They build on each other (A installs the CLI, B teaches context, C explores slash commands, D covers permissions, E ties it all together with a multi-turn task).

### Exercise A — Install + auth + hello world

🛠️ **Hands-on**

**Step 1: Install the CLI**

Pick the command for your platform:

| Platform | Command |
|----------|---------|
| macOS (Homebrew) | `brew install copilot-cli` |
| macOS / Linux (npm) | `npm install -g @github/copilot` |
| Windows (winget) | `winget install GitHub.Copilot` |
| Windows (npm) | `npm install -g @github/copilot` |

Run the command, then verify:

```bash
copilot --version
```

> ✅ **You should now see:** A version string (e.g., `1.0.32` or later).

**Step 2: Navigate to the `app/` folder**

```bash
cd github-copilot-101/copilot-cli/app/
```

This ensures the CLI has the FastAPI app from this lab's workspace as context for the next step. The full path works whether or not you're already in the repo.

**Step 3: Start interactive mode and authenticate**

```bash
copilot
```

On first run, you'll see:
- A **trust folder prompt** — type `yes` to trust the `app/` directory.
- An **authentication prompt** — the CLI will either show a `/login` command suggestion or device-flow instructions directly (a code + URL). Either way, open the URL in your browser, enter the code, and authenticate with GitHub.

Once authenticated, you'll see a chat prompt (`>`).

**Step 4: Send your first query**

At the `>` prompt, type:

```prompt
What does @backend/app.py do?
```

> ✅ **You should now see:** A summary of the FastAPI app — it manages activity signups for Mergington High School, with endpoints like `GET /activities` and `POST /activities/{activity_name}/signup` for viewing and signing up for extracurricular activities.

Type `exit` or press `Ctrl+C` to quit.

---

### Exercise B — Context management

🛠️ **Hands-on**

Use `@` mentions and slash commands to control context.

**Step 1: Start interactive mode**

```bash
cd copilot-cli/app/
copilot
```

**Step 2: Add context with `@` mentions**

> 📌 **Note:** `@.` means "current directory" — it adds all files in the folder to context (use carefully in large directories).

Try these prompts one by one:

```prompt
> Explain @backend/app.py
```

```prompt
> What endpoints are in @backend/?
```

```prompt
> Review @. for security issues
```

> ✅ **Expected output:** You should see the CLI summarize files in the backend folder, including `app.py` and its endpoints.

**Step 3: Inspect context usage**

```prompt
> /context
```

**Step 4: Manage the context window**

```prompt
> /compact
```

```prompt
> /clear
```

Alias: `/new`.

> 📌 **Important:** There is **no** `/reset`, `/stop`, or natural-language "remove context" command. Manage scope via `@` mentions per message and use `/compact` or `/clear` to reduce window pressure.

**Step 5: Undo the last turn**

If the CLI makes a change you want to revert, use:

```prompt
> /undo
```

> ✅ **You should now see:** Context usage displayed via `/context`, a summarized history after `/compact`, a fresh session after `/clear`, and confidence that you can revert mistakes with `/undo`.

Type `exit` to quit.

---

### Exercise C — Slash commands deep dive

🛠️ **Hands-on**

The CLI ships with several slash commands for specialized workflows. This exercise explores the most useful ones.

| Command | Purpose | Example |
|---------|---------|---------|
| `/plan` | Create implementation plan (doesn't execute) | `/plan Build waitlist feature` |
| `/yolo` | Allow all tools, paths, URLs (no approval prompts) | `/yolo Refactor auth and run tests` |
| `/fleet` | Parallel subagent execution | `/fleet Review backend, frontend, tests` |
| `/research` | Deep investigation (GitHub + web) | `/research Best rate limiting approaches` |
| `/delegate` | Send session to GitHub cloud agent → creates a PR | `/delegate Refactor auth and open PR` |
| `/chronicle` | Session insights (**EXPERIMENTAL**) | `/chronicle standup` |
| `/skills` | List/info/add/remove/reload skills | `/skills list` |

**Important notes:**

- **`/delegate` does NOT invoke local sub-agents.** It hands the session to GitHub's cloud agent, which opens a PR on your behalf. For local parallel work, use `/fleet`.
- **Skills are invoked via `@skill:skill-name` mentions** in prompts, not via a `/skills invoke` subcommand. The `/skills` command is for *managing* skills (list/info/add/remove), not invoking them.
- **`/chronicle` requires enabling experimental features first:** `/experimental on` (covered in Part 3).

**Step 1: Use `/plan` to preview work**

```bash
cd copilot-cli/app/
copilot
```

At the prompt:

```prompt
> /plan Add a GET /activities/{name}/description endpoint that returns the activity description
```

The CLI generates a numbered plan (files to touch, steps to take) but **doesn't execute** anything. Review the plan, then type `go` (no slash — it's a literal response at the plan prompt) to execute, or `exit` to cancel.

**Step 2: Research a design decision**

```prompt
> /research Best practices for rate limiting in FastAPI
```

The CLI searches GitHub repositories and the web, then summarizes findings with citations.

**Step 3: Chain commands in a workflow**

Try this multi-turn workflow:

1. `/plan Build a waitlist feature for full activities`
2. Review the plan, type `go`
3. After execution, use `/research Best UX patterns for waitlist notifications`
4. Apply findings to improve the feature

> ✅ **You should now see:** A plan generated by `/plan`, research findings from `/research`, and a mental model of how to chain slash commands for real-world tasks.

Type `exit` to quit.

> 📌 **Practice task:** Use `/plan` → review → execute → `/research` to investigate a tricky design choice in your own codebase.

---

### Exercise D — Permission flags & patterns

🛠️ **Hands-on**

The CLI uses **permission patterns** to control what tools the model can use. Patterns take the form `tool` or `tool(arg)`.

**Pattern syntax:**

| Pattern | What it matches | Example |
|---------|-----------------|---------|
| `bash`, `write`, `view`, `edit`, `web_fetch` | Bare tool names | Allow all file reads: `--allow-tool=view` |
| `shell(git push)` | Specific shell command | Deny pushes: `--deny-tool='shell(git push)'` |
| `shell(git:*)` | All subcommands of a tool | Allow all git: `--allow-tool='shell(git:*)'` |
| `MyMCP(tool_name)` | Specific MCP server tool | Allow one MCP tool: `--allow-tool='github-mcp-server(create_issue)'` |
| `url(github.com)` | URL access by domain | Allow GitHub URLs: `--allow-tool='url(github.com)'` |

**Step 1: Allow all tools (no prompts)**

```bash
cd copilot-cli/app/
copilot --allow-all-tools
```

At the prompt:

```prompt
> Review @backend/app.py and run pytest
```

The CLI runs tools without asking for approval. This is equivalent to `/yolo` but set at session start.

Type `exit`, then try the next step.

**Step 2: Deny destructive git operations**

```bash
copilot --deny-tool='shell(git push)' --deny-tool='shell(git reset --hard)'
```

At the prompt:

```prompt
> Refactor the registration endpoint and push to a new branch
```

The CLI will refuse to execute `git push` and prompt you to approve or deny manually.

> ✅ **You should now see:** The CLI attempts the denied operation (git push), and you see a deny prompt confirming that the permission constraint worked as intended.

Type `exit`, then try the next step.

**Step 3: Allowlist read-only operations**

```bash
copilot --allow-tool=view --allow-tool='shell(git:*)' --deny-tool=write
```

At the prompt:

```prompt
> Review @backend/app.py for security issues
```

The CLI can read files and run git commands, but cannot write files. It will prompt if it tries to edit.

> ✅ **You should now see:** A session where the CLI respects your permission constraints, prompting you when it tries to use a denied tool.

Type `exit` to quit.

> 📌 **Key takeaway:** Permission patterns are your safety net for headless automation. In CI/CD, use `--deny-tool='shell(git push)'` to prevent accidental commits.

---

### Exercise E — Multi-turn autonomous task

🛠️ **Hands-on**

Now tie everything together: use the CLI in **autopilot mode** to build a feature end-to-end, leveraging context management, slash commands, and tool permissions.

**The task:** Add a waitlist feature to the `app/` FastAPI app. When an activity is full (31st student tries to sign up), show a "Join Waitlist" button in the frontend instead of "Sign Up".

**Step 1: Run the task in autopilot mode**

```bash
cd copilot-cli/app/
copilot --mode autopilot -i "Build a waitlist feature in @backend and @frontend. Add a backend endpoint for waitlist signup and update the frontend to show 'Join Waitlist' when an activity is full."
```

> 📌 **`--mode autopilot` vs `-i` flag:** `--mode autopilot` enables autonomous multi-turn execution. `-i` passes the initial prompt inline and exits when done.

**Step 2: Observe the loop**

The CLI will:
1. Read the existing code
2. Plan the changes (backend endpoint + frontend logic)
3. Edit `app/backend/app.py` (add `/activities/{name}/waitlist` endpoint)
4. Edit `app/frontend/index.html` or `app/frontend/script.js` (add waitlist button logic)
5. Run tests (if they exist) or start the server to verify
6. Iterate if anything fails

**Step 3: Verify the result**

Start the server:

```bash
uvicorn backend.app:app --reload
```

Open `app/frontend/index.html` in your browser (or use Live Server). Try to sign up for an activity with 30 participants already signed up.

> ✅ **You should now see:** The 31st signup attempt shows a "Join Waitlist" button instead of "Sign Up". The backend has a new `/activities/{name}/waitlist` endpoint (verify with `curl -X POST http://127.0.0.1:8000/activities/Chess%20Club/waitlist?email=test@example.com`).

**Step 4: Use `/undo` if the agent drifts**

If the CLI makes changes you don't like, restart and use `/undo` to revert the last turn:

```bash
copilot
> /undo
```

> 📌 **Alias:** `/rewind` is an alias for `/undo`.

---

## Part 3 — Customization

The CLI inherits many of the customization features from VS Code — custom agents, skills, MCP servers, and plugins — but with different file paths and invocation patterns. This section maps the CLI equivalents.

### Custom instructions — CLI paths

🛠️ **Hands-on**

Custom instructions work the same way in the CLI as in VS Code (covered in Lab 2), but the file paths differ.

| Scope | Path |
|-------|------|
| User | `~/.copilot/copilot-instructions.md` |
| Repo | `.github/copilot-instructions.md` |

> 📌 **Note:** Both paths use `.github/` at the repo level — **not** `.copilot/`. This is intentional to match GitHub's convention for repo-wide config files.

**Step 1: Create a user-level instruction**

```bash
mkdir -p ~/.copilot
echo "Always prefer list comprehensions over map/filter in Python." > ~/.copilot/copilot-instructions.md
```

**Step 2: Test the instruction**

```bash
cd copilot-cli/app/
copilot
```

At the prompt:

```prompt
> Refactor @backend/app.py to use functional programming patterns
```

> ✅ **You should now see:** The CLI suggests list comprehensions instead of `map()`/`filter()` calls, per your custom instruction.

Type `exit` to quit.

> 📌 **Reminder:** See Lab 2 for authoring patterns (tone, formatting rules, edge-case handling). Lab 4 focuses on CLI-specific paths.

---

### Custom agents

🛠️ **Hands-on**

Custom agents in the CLI work similarly to VS Code (covered in Lab 3), but with one key difference: they're **user-level only** (no `.github/agents/` in the CLI).

**File location:** `~/.copilot/agents/`  
**File extension:** `*.agent.md` (the `.agent` infix is **required** — files without it are ignored)

**Frontmatter format:**

```yaml
---
name: fastapi-code-reviewer
description: "Specialized code review agent for FastAPI codebases. Use when reviewing backend Python code for security, validation, and FastAPI best practices."
---

[Your system prompt goes here]
```

> 📌 **Key difference from VS Code:** The `tools` field is **not required** by the CLI — agents inherit available tools from the session. Only `name` and `description` are confirmed required.

**Step 1: Create a custom agent**

You have two ways to create one:

**Option A — Guided (recommended for first-timers)**

Inside an interactive `copilot` session, run:

```prompt
> /agents
```

Copilot walks you through naming the agent, writing its description, and dropping the file in `~/.copilot/agents/` for you. Pick this when you want the CLI to scaffold the front-matter and persona prompt step by step.

**Option B — Manual**

```bash
mkdir -p ~/.copilot/agents
```

Create `~/.copilot/agents/fastapi-code-reviewer.agent.md` with your editor and paste:

```markdown
---
name: fastapi-code-reviewer
description: "Specialized code review agent for FastAPI codebases. Use when reviewing backend Python code for security, validation, and FastAPI best practices."
---

You are a code review specialist focusing on:
- FastAPI best practices (dependency injection, async patterns, lifecycle hooks)
- Input validation and Pydantic models
- Security vulnerabilities (auth, injection, secrets in code)
- Test coverage gaps

Always provide specific file paths and line numbers, plus an actionable fix per finding.
```

**Step 2: Invoke the custom agent**

```bash
cd copilot-cli/app/
copilot
```

At the prompt:

```prompt
> @fastapi-code-reviewer Review @backend/app.py
```

The CLI uses your custom agent's persona and system prompt.

> ✅ **You should now see:** A review focused on FastAPI-specific issues (Pydantic validation, async patterns, dependency injection) with file/line references and fixes.

Type `exit` to quit.

**Step 3: Browse available agents**

You can list all custom agents with:

```prompt
> /agents
```

> 📌 **Note:** Custom agents in the CLI are user-level only (`~/.copilot/agents/`). There is no `.github/agents/` equivalent in the CLI at time of writing.

---

### Hooks (read-through)

📖 **Read-through**

Hooks are a real CLI feature. They let you run scripts in response to CLI events (e.g., before a session starts, after a file is edited).

**Configuration locations:**

| Scope | Location |
|-------|----------|
| User | `hooks` key in `~/.copilot/config.json` |
| Repo | `.github/hooks/*.json` |

**Skeleton (event types are under-documented at time of writing — refer to current docs):**

```json
{
  "hooks": {
    "event-name": {
      "command": "/path/to/script.sh",
      "description": "What this hook does"
    }
  }
}
```

**Disable globally:** Add `"disableAllHooks": true` to `~/.copilot/config.json`.

> 📌 **Why read-through?** Hook event types are evolving and under-documented. Once you've identified an event type in your version of the CLI, the syntax above plugs in directly. Lab 4 keeps this as read-through to avoid shipping stale event names.

---

### Skills — CLI invocation

🛠️ **Hands-on**

Skills (covered in Lab 2 for authoring) are invoked differently in the CLI.

**Invocation syntax:**

```prompt
> @skill:fastapi-best-practices Review this endpoint
```

> 📌 **Key difference from VS Code:** Skills are **invoked** via `@skill:` mentions in prompts, not via a slash command. The `/skills` command is for *managing* skills (list/info/add/remove), not invoking them.

**Step 1: List available skills**

```bash
copilot
```

At the prompt:

```prompt
> /skills list
```

**Step 2: Get details about a skill**

```prompt
> /skills info fastapi-best-practices
```

**Step 3: Invoke a skill**

```prompt
> @skill:fastapi-best-practices Review @backend/app.py for validation issues
```

> ✅ **You should now see:** The skill's specialized knowledge applied to your code review.

Type `exit` to quit.

> 📌 **Reminder:** See Lab 2 for skill authoring (SKILL.md format, domain/description/examples). Lab 4 focuses on CLI-specific invocation.

---

### Copilot memory via /chronicle

🛠️ **Hands-on** — **EXPERIMENTAL**

`/chronicle` is an **experimental** feature that analyzes your session history to generate insights, standup reports, and custom-instructions improvements.

**Enable experimental features:**

```bash
copilot
```

At the prompt:

```prompt
> /experimental on
```

**Subcommands:**

| Command | Purpose |
|---------|---------|
| `/chronicle standup` | Generate a standup report from recent sessions |
| `/chronicle tips` | Personalized CLI usage tips based on your history |
| `/chronicle improve` | Analyze session history → propose custom-instructions improvements |
| `/chronicle reindex` | Rebuild session store from history |

**Storage:** All session data is stored locally:
- `~/.copilot/session-state/` (JSONL format)
- `~/.copilot/session-store.db` (SQLite)

**Step 1: Enable experimental features and generate a standup**

```prompt
> /experimental on
> /chronicle standup
```

> ✅ **You should now see:** A standup-style summary of your recent CLI sessions (tasks completed, files touched, commands run).

**Step 2: Get personalized tips**

```prompt
> /chronicle tips
```

> ✅ **You should now see:** Tips based on your usage patterns (e.g., "You use `/plan` frequently — consider adding a custom agent for planning workflows").

**Step 3: Improve your custom instructions**

```prompt
> /chronicle improve
```

> ✅ **You should now see:** Proposed edits to your `~/.copilot/copilot-instructions.md` based on recurring patterns in your sessions.

Type `exit` to quit.

> 📌 **Privacy note:** All `/chronicle` data is stored locally on your machine. No session data is sent to GitHub or third parties.

---

### CLI plugins

🛠️ **Hands-on**

The CLI has its **own** plugin system (separate from VS Code's). Plugins can bundle custom agents (`*.agent.md`), skills (`SKILL.md`), hooks, MCP configs, and LSP configs.

**Default marketplaces:**
- `github/copilot-plugins`
- `github/awesome-copilot`

**Management commands:**

```bash
copilot
```

At the prompt:

```prompt
> /plugin list
```

Shows installed plugins.

```prompt
> /plugin marketplace
```

Browses available plugins in the default marketplaces.

```prompt
> /plugin install <name>
```

Installs a plugin by name.

```prompt
> /plugin uninstall <name>
```

Removes a plugin.

```prompt
> /plugin update
```

Updates all installed plugins.

**Outside a session:**

```bash
copilot plugin list
copilot plugin marketplace
copilot plugin install <name>
```

> ✅ **You should now see:** A list of installed plugins (possibly empty) and at least one marketplace entry when you run `/plugin marketplace`.

Type `exit` to quit.

> 📌 **Key difference from VS Code:** The CLI's plugin system is separate from VS Code's. A plugin installed in the CLI is not automatically available in VS Code, and vice versa.

---

### MCP — CLI config paths (read-through)

📖 **Read-through**

Model Context Protocol (MCP) servers (covered in Lab 3) work in the CLI, but with different config paths.

| Scope | Path |
|-------|------|
| User | `~/.copilot/mcp-config.json` |
| Repo | `.github/mcp.json` |

> 📌 **Note:** Different from VS Code (`.vscode/mcp.json`) and different from what you might expect. Note `mcp-config.json` for user, plain `mcp.json` for repo.

**Example `~/.copilot/mcp-config.json`:**

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"]
    }
  }
}
```

> ✅ **You should now see** (mentally): MCP servers work the same way in the CLI as in VS Code, but with different config-file paths. Locate `~/.copilot/mcp-config.json` and identify one configured server (if any exist).

> 📌 **Reminder:** See Lab 3 for MCP server authoring (FastMCP in Python, server registration, tool exposure). Lab 4 focuses on CLI-specific config paths.

---

## Part 4 — Programmatic & Automation

The CLI's killer feature: headless invocation for CI/CD, PR reviews, and scheduled audits.

### Exercise F — Headless invocation

🛠️ **Hands-on**

**One-shot invocation:**

```bash
cd copilot-cli/app/
copilot -p "Generate tests for @backend/app.py"
```

The CLI executes the prompt and exits when done.

**Combine with permission patterns for safety:**

```bash
copilot -p "Review @backend/ for security issues. Write to review.md" \
  --allow-tool=write --deny-tool='shell(git push)'
```

**Step 1: Write a script**

Create `review-script.sh`:

```bash
#!/bin/bash
copilot -p "Review @backend/ for security. Write report to review-report.md" \
  --allow-all-tools
cat review-report.md
```

Make it executable:

```bash
chmod +x review-script.sh
```

**Step 2: Run the script**

```bash
cd copilot-cli/app/
./review-script.sh
```

> ✅ **You should now see:** `review-report.md` created autonomously with a security review of the backend code.

> 📌 **Key pattern:** Headless invocation + `--allow-all-tools` (or specific permission patterns) = fully automated code reviews, test generation, or refactoring in CI/CD.

---

## Q&A

### **Q: What is the difference between using "Plan mode" and the `/plan` slash command in another mode (like interactive or autopilot)?**

**Short answer:** Plan mode is a persistent interactive state where every prompt generates a structured plan before execution. The `/plan` slash command is a one-off request within any mode that creates a plan without executing it.

**Plan mode (persistent state):**
- Entered with `copilot --plan` flag or by pressing **Shift+Tab** in an interactive session
- A mode of the CLI where Copilot builds structured implementation plans for every request
- Asks clarifying questions to understand scope before planning
- Plans are generated before any code is written
- Stays in this mode until you toggle back with Shift+Tab or restart

**`/plan` slash command (one-off request):**
- Typed as `/plan <task description>` within any interactive or autopilot session
- Generates a single plan for the specified task
- Does not change the session mode — you remain in interactive/autopilot mode
- Useful when you want a plan once without switching the entire session's behavior

**📌 When to use which:**
- Use **plan mode** (Shift+Tab) when you want to iterate on planning across multiple tasks and stay in a "plan-first" mindset for the entire session
- Use **`/plan` command** when you're in a regular interactive session and want a one-off plan before deciding whether to proceed

**Sources:**
- [About GitHub Copilot CLI - Modes of use](https://docs.github.com/en/copilot/concepts/agents/about-copilot-cli#modes-of-use)
- [Use GitHub Copilot CLI - Use plan mode](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/use-copilot-cli#use-plan-mode)

---

## What's Next?

Congratulations! You've mastered the GitHub Copilot CLI — from interactive sessions to headless automation.

**You now know how to:**

1. ✅ Use the CLI interactively with slash commands, context management, and permission patterns
2. ✅ Customize the CLI with custom agents, skills, plugins, and hooks
3. ✅ Script headless workflows with `copilot -p` for CI/CD
4. ✅ Apply permission patterns to control what tools the model can use
5. ✅ Explore experimental features like `/chronicle` for session insights

**What we skipped (and where it's covered):**

| Topic | Where it was taught |
|-------|---------------------|
| Custom instructions (theory + authoring) | Lab 2 |
| Prompt files (`.github/prompts/`) | Lab 2 |
| Skills (authoring) | Lab 2 |
| `AGENTS.md` | Lab 2 |
| MCP server creation | Lab 3 |
| VS Code custom agents (`.github/agents/`) | Lab 3 |
| VS Code agent plugins | Lab 3 |

**Extensions to try:**

- **Multi-agent workflows:** Chain custom agents in scripts (e.g., `fastapi-code-reviewer` → `test-author` → `endpoint-scaffolder`)
- **Chronicle-driven improvements:** Run `/chronicle improve` weekly to refine your custom instructions based on real usage patterns
- **Custom hooks:** Once event types are documented, write hooks to auto-format code before sessions or auto-commit after successful test runs

**Further reading:**

- [Copilot CLI documentation](https://code.visualstudio.com/docs/copilot/agents/copilot-cli)
- [Permission levels reference](https://code.visualstudio.com/docs/copilot/agents/agent-tools#_permission-levels)
- [MCP specification](https://modelcontextprotocol.io/)
- [GitHub Copilot plugins marketplace](https://github.com/github/copilot-plugins)

---

**Lab complete.** 🎉

The CLI is your terminal-native AI assistant — use it for automation, headless workflows, and CI/CD integration that VS Code can't provide.
