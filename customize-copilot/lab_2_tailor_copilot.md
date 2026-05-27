# Lab 02 — Customizing GitHub Copilot

Teach **GitHub Copilot** to speak your project's language. Custom instructions, path-specific rules, reusable prompt files, custom agents, agent skills, MCP servers, and plugins turn Copilot from a generic assistant into a domain expert that knows your conventions by heart.

## Table of Contents

- [What You'll Learn](#what-youll-learn)
- [Setup](#setup)
- [Phase 1: Custom Instructions](#phase-1-custom-instructions)
  - [Step 1: Setting Up Custom Instructions](#step-1-setting-up-custom-instructions)
  - [Step 2: Path-Specific Custom Instructions](#step-2-path-specific-custom-instructions)
- [Phase 2: Reusable Prompt Files](#phase-2-reusable-prompt-files)
- [Phase 3: Custom Agents](#phase-3-custom-agents)
- [Phase 4: Agent Skills](#phase-4-agent-skills)
- [Phase 5: Model Context Protocol (MCP)](#phase-5-model-context-protocol-mcp)
- [Phase 5b: Agent Hooks (Preview)](#phase-5b-agent-hooks-preview)
- [Phase 6: Plugins](#phase-6-plugins)
- [Congratulations! 🎉](#congratulations-)

## What You'll Learn

Today's goal is to learn how to **customize GitHub Copilot's behavior** so its output consistently matches your project standards — without repeating the same guidance in every prompt.

- **Custom Instructions** — Give Copilot project-wide, personal, and organization-level context
- **Path-Specific Custom Instructions** — Target conventions to specific files or directories
- **Prompt Files** — Build reusable slash commands that automate multi-step workflows
- **Custom Agents** — Define reusable personas with their own instructions, tool restrictions, and handoffs
- **Agent Skills** — Package domain expertise so Copilot writes better code in specialized areas
- **Model Context Protocol (MCP)** — Expose data and capabilities to Copilot via standardized server processes
- **Agent Hooks** — Execute shell commands at key agent lifecycle points to enforce quality checks and guide agent behavior deterministically
- **Plugins** — Bundle agents, MCP servers, and skills into a single shareable, installable package

By the end of this lab you will have set up custom instructions at every scope, fixed non-compliant content in your activities data, built a prompt file to automate adding new activities, defined a pair of custom agents that hand off work to each other, created an agent skill that layers expert defaults on top of that prompt, written your own MCP server that exposes the school activities to Copilot, and packaged everything as a distributable plugin — all on the FastAPI school activities website from Lab 01. ♟️ ⚽️ 🎻

## Setup

We'll keep building on the same project from Lab 01 — the **GitHub Copilot High School** activities website.

1. Open this repository in VS Code.

2. Verify the **GitHub Copilot** and **Python** extensions are installed and enabled.

3. From the **workspace root**, create and activate a virtual environment:

   - **macOS / Linux:**

     ```bash
     python3 -m venv venv
     source venv/bin/activate
     ```

   - **Windows:**

     ```bash
     python -m venv venv
     venv\Scripts\activate
     ```

4. Install the dependencies:

   ```bash
   pip install -r requirements.txt
   ```

5. Start the development server from the workspace root:

   ```bash
   uvicorn app.backend.app:app --reload
   ```

6. Open your browser at **http://127.0.0.1:8000** and confirm the activities page loads.

   > ❕ **Important:** Keep the server running throughout the lab so you can see live changes after each step.

> 💡 **Tip:** This lab assumes you completed **Lab 01**, which migrated the activities data from a hardcoded dictionary into [`app/backend/data/activities.json`](../app/backend/data/activities.json). If your `app/backend/app.py` still uses the hardcoded dictionary, ask Copilot in Agent Mode to *"Load the activities from `app/backend/data/activities.json` at startup instead of using the hardcoded dictionary"* before continuing.

## Phase 1: Custom Instructions

In this phase you will configure Copilot custom instructions across **both repository types** (repository-wide and path-specific) and learn how the three **scope levels** (organization, repository, personal) interact.

### Step 1: Setting Up Custom Instructions

### 📖 Theory: Custom Instructions in a Nutshell

Custom instructions are **natural-language rules** automatically injected into every Copilot request. They eliminate repetitive guidance — set them once and Copilot follows them every time.

Copilot supports three **scope levels** that combine on every request. When they conflict, the higher-priority level wins:

| Priority | Level | Set by · Where it lives | Typical use |
| -------- | ----- | ----------------------- | ----------- |
| 🥇 Highest | **Personal** | You · GitHub.com personal settings | Your role, language, style preferences |
| 🥈 Medium  | **Repository** | Any contributor · `.github/copilot-instructions.md` | Project conventions, tech stack |
| 🥉 Lowest  | **Organization** | Org owners · GitHub.com Org Settings | Company-wide standards |

Inside a repository there are also **two types** of instructions that complement each other:

| Type | File | Scope |
| ---- | ---- | ----- |
| **Repository-wide** | `.github/copilot-instructions.md` | All requests in the repo |
| **Path-specific** | `.github/instructions/NAME.instructions.md` | Files matching a glob pattern (covered in Step 2) |

> 💡 **Tip:** This step focuses on the hands-on case — repository-wide instructions. Organization and personal instructions are configured on GitHub.com; see the docs ([organization](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-organization-instructions), [personal](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-personal-instructions)) to enable them. Org instructions currently apply to Copilot Chat on GitHub.com, code review, and the coding agent.

> 🪧 **Note:** Personal custom instructions are only supported for GitHub Copilot Chat in GitHub.

---

#### Repository-Wide Custom Instructions

Repository-wide instructions live in **`.github/copilot-instructions.md`** and are automatically attached to every Copilot Chat request in the repo. Keep them short and focused on the **"how"** of the project: purpose, structure, coding standards, expected formats.

> 📚 Full documentation: [Adding repository custom instructions for GitHub Copilot](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions?tool=vscode)

### Activity: Create repository instructions with Copilot 🤖

Right now Copilot doesn't **truly** know our project conventions. If we ask it to add an activity, it might use the wrong email domain, a casual schedule format, or pick an arbitrary `max_participants` value. Let's fix that by creating repository-level instructions.

Instead of creating the file manually, let's use **Agent Mode** to do the heavy lifting!

1. Open the **Copilot Chat** panel and switch to **Agent** mode.

2. Ask Copilot to create the instructions file for you. Provide enough context so it understands what the file should contain:

   > ![Static Badge](https://img.shields.io/badge/-Prompt-text?style=social&logo=github%20copilot)
   >
   > ```prompt
   > Create a .github/copilot-instructions.md file for this project.
   > It should describe:
   > - The project (a school extracurricular activities portal built with Python/FastAPI/Uvicorn)
   > - The tech stack (FastAPI, vanilla JS, static HTML, JSON data)
   > - The project structure (app/backend/app.py, app/backend/data/activities.json, app/frontend/)
   > - How to run it: `uvicorn app.backend.app:app --reload` from the workspace root
   > - Conventions:
   >   - Activity names in Title Case (e.g., "Chess Club" not "chess club")
   >   - Student emails must use the @githubcopilot.edu domain
   >   - Schedules follow the format "Days, HH:MM AM/PM - HH:MM AM/PM"
   >     (e.g., "Fridays, 3:30 PM - 5:00 PM")
   >   - max_participants is always a positive integer
   >   - Descriptions are 1-2 professional sentences
   >   - The same email must never appear twice in an activity's participants list
   ```

3. Review the file Copilot creates.

4. If Copilot's output is missing any conventions, provide follow-up feedback to refine it. Remember — Copilot keeps the conversation history, so you can iterate!

5. **Accept the changes** and save the file.

   > ❕ **Important:** The file must be at exactly `.github/copilot-instructions.md` at the workspace root. If Copilot placed it somewhere else, move it.

### Activity: Test your repository instructions ✅

Now let's verify that Copilot actually uses the instructions you just created.

1. Make sure you are in **Agent** mode in Copilot Chat.

2. Ask Copilot a question that exercises your conventions:

   > ![Static Badge](https://img.shields.io/badge/-Prompt-text?style=social&logo=github%20copilot)
   >
   > ```prompt
   > What conventions does this project have for activity names,
   > student emails, and the schedule format?
   > ```

3. Copilot should respond with the rules you defined — Title Case names, `@githubcopilot.edu` emails, and the `"Days, HH:MM AM/PM - HH:MM AM/PM"` schedule format.

4. Check the **References** section at the bottom of Copilot's response. You should see `.github/copilot-instructions.md` listed, confirming it was used.

   <details>
   <summary>Don't see the reference? 🔍</summary>

   - Make sure the file is saved at exactly `.github/copilot-instructions.md` (not in a subfolder).
   - Restart VS Code if the file was just created.
   - Verify the setting **"Enable custom instructions"** is checked in VS Code settings (search for `copilot instructions`).

   </details>

   **🎯 Goal: Copilot references your instruction file and follows the project conventions. ✅**

---

### Step 2: Path-Specific Custom Instructions

Great work setting up repository-wide custom instructions! Now let's tackle a more targeted scenario.

🐛 **THE ACTIVITIES DATA WILL DRIFT OVER TIME** 🐛

Repository-wide instructions are great for general guidance, but they live as paragraphs of prose — easy to overlook for a focused task. When teachers or admins add new entries to [`app/backend/data/activities.json`](../app/backend/data/activities.json), they might pick the wrong schedule format, forget the `@githubcopilot.edu` domain, or write a name in lowercase. We want **automatic, targeted enforcement** every time anyone edits the activities data file.

That's where **path-specific custom instructions** come in.

### 📖 Theory: Path-Specific Custom Instructions

Instruction files (`*.instructions.md`) provide Copilot with targeted guidance for **specific files or directories**. Unlike repository-wide instructions that apply everywhere, these use the `applyTo` field in the [frontmatter](https://jekyllrb.com/docs/front-matter/) with [glob syntax](https://code.visualstudio.com/docs/editor/glob-patterns) to target specific paths.

VS Code looks for `*.instructions.md` files in the `.github/instructions/` directory by default. When Copilot works on a file that matches the glob pattern, the instructions are **automatically attached** — no manual action required.

> 💡 **Tip:** Instructions should focus on **HOW** a task should be done — the guidelines, standards, and conventions for that particular part of the codebase.

### Activity: Create activities-data-specific instructions with Copilot 🤖

Instead of writing the instruction file from scratch, let's ask **Agent Mode** to create it for us — while also teaching it what the file needs to contain.

1. Open **Copilot Chat** and switch to **Agent** mode.

2. Drag [`app/backend/data/activities.json`](../app/backend/data/activities.json) into the chat as context, then ask Copilot to generate the instruction file:

   > ![Static Badge](https://img.shields.io/badge/-Prompt-text?style=social&logo=github%20copilot)
   >
   > ```prompt
   > Create a file at .github/instructions/activities-data.instructions.md
   > It should have an applyTo frontmatter targeting "app/backend/data/**/*.json"
   > and contain rules for entries in activities.json:
   > - Activity keys must be in Title Case (e.g., "Chess Club", not "chess club"
   >   or "Chess Club (JSON)" with extra suffixes)
   > - Each entry must have: description, schedule, max_participants, participants
   > - description: 1-2 professional sentences, no slang
   > - schedule: format "Days, HH:MM AM/PM - HH:MM AM/PM" (e.g., "Fridays, 3:30 PM - 5:00 PM")
   > - max_participants: positive integer (not a string)
   > - participants: list of student emails using the @githubcopilot.edu domain
   > - No duplicate emails inside a single activity's participants list
   ```

3. Review the file Copilot creates. It should have a YAML frontmatter block with `applyTo: "app/backend/data/**/*.json"` and a set of clear rules.

4. **Accept the changes** and save the file.

### Activity: Clean up the existing data with Copilot 🧹

Now the instruction file exists and automatically applies to any file under `app/backend/data/**/*.json`. Let's put it to the test on the data you already have.

1. Open [`app/backend/data/activities.json`](../app/backend/data/activities.json) in VS Code. Depending on what state you left it in after Lab 01, the keys might still have a `"(JSON)"` suffix or other inconsistencies.

2. Open **Copilot Chat** in **Agent** mode.

3. With the JSON file open, ask Copilot to normalize it:

   > ![Static Badge](https://img.shields.io/badge/-Prompt-text?style=social&logo=github%20copilot)
   >
   > ```prompt
   > Review this file and bring every entry in line with the project standards.
   > Don't change participant emails or schedules that are already valid.
   ```

4. Observe how Copilot references `.github/instructions/activities-data.instructions.md` in its response references — that's confirmation the path-specific instructions were applied automatically.

5. Review the proposed changes (activity keys should be in clean Title Case, no `(JSON)` suffixes, schedules in the canonical format, etc.).

6. **Accept the changes**, save the file, and refresh the browser. The activities list should still load — now with cleaned-up names.

### Activity: Add (and fix) a broken entry 🔧

Real-world data files drift because contributors paste in whatever they have. Let's simulate that.

1. Open [`app/backend/data/activities.json`](../app/backend/data/activities.json) and paste the following entry inside the top-level object (mind the trailing comma):

   ```jsonc
   "drama club": {
     "description": "drama!",
     "schedule": "tuesday afternoons",
     "max_participants": "15",
     "participants": ["alex", "alex", "jamie@gmail.com"]
   }
   ```

   This entry breaks **every** convention you defined: lowercase key, vague description, free-form schedule, `max_participants` as a string, duplicate participant, and an external email domain.

2. Save the file. **Do not** try to fix it yourself.

3. In **Copilot Chat** Agent Mode, with the JSON file open, ask:

   > ![Static Badge](https://img.shields.io/badge/-Prompt-text?style=social&logo=github%20copilot)
   >
   > ```prompt
   > The new "drama club" entry doesn't follow our standards. Fix it.
   > ```

4. Verify Copilot:

   - Renames the key to `Drama Club` (Title Case).
   - Expands the description into 1–2 professional sentences.
   - Rewrites the schedule into the canonical format (e.g., `"Tuesdays, 3:30 PM - 5:00 PM"`).
   - Converts `max_participants` from `"15"` to the integer `15`.
   - Removes the duplicate `alex` and replaces both invalid emails with the `@githubcopilot.edu` domain.

5. **Accept the changes** and verify the new card appears correctly in the browser.

   **🎯 Goal: A non-compliant entry pasted by a hypothetical teammate is automatically fixed by Copilot, guided by your path-specific instructions. ✅**

<details>
<summary>Having trouble? 🤷</summary>

- Make sure the instruction file is at `.github/instructions/activities-data.instructions.md`.
- The `applyTo` field must be `"app/backend/data/**/*.json"` — check for typos.
- Restart VS Code if the instructions don't seem to apply.
- If the website doesn't update, make sure you saved the JSON file and that the `uvicorn` server is running with `--reload`.

</details>

---

## Phase 2: Reusable Prompt Files

Now that activities follow a consistent structure, you want to make it easy to **add new activities** without remembering every field, format, or convention. This is a perfect scenario for a **prompt file** — a reusable slash command that automates repetitive workflows.

### 📖 Theory: What are Prompt Files?

Prompt files (`*.prompt.md`) define reusable prompts that appear as **slash commands** (`/`) in Copilot Chat. They can reference other workspace files (like data files and templates) to provide context.

| Aspect | Details |
| ------ | ------- |
| **File extension** | `.prompt.md` |
| **Default location** | `.github/prompts/` directory |
| **Invocation** | Type `/prompt-name` in the Copilot Chat input |
| **Context** | Can reference files using markdown links or `#file:` syntax |
| **Scope** | Reusable by anyone who clones the repository |

> 💡 **Tip:** Use prompt files to define repeatable tasks and workflows. Focus on **WHAT** needs to be done. Reference instructions for the **HOW**.

See the [VS Code Docs: Prompt Files](https://code.visualstudio.com/docs/copilot/copilot-customization#_prompt-files-experimental) page for more information.

### Activity: Create the new activity prompt with Copilot 🤖

Let's use **Agent Mode** to help us write the prompt file itself — prompt files are just markdown, and Copilot is great at writing markdown!

1. Open **Copilot Chat** in **Agent** mode.

2. Ask Copilot to create the prompt file:

   > ![Static Badge](https://img.shields.io/badge/-Prompt-text?style=social&logo=github%20copilot)
   >
   > ```prompt
   > Create a reusable prompt file at .github/prompts/new-activity.prompt.md
   > that automates adding a new extracurricular activity.
   > It should:
   > - Have frontmatter with `agent: agent`, a description, and an argument-hint
   > - Step 1: Gather activity info from the user if not provided
   >   (name, description, schedule, max_participants)
   > - Step 2: Append a new top-level entry to app/backend/data/activities.json
   >   following the format of existing entries
   >   (initialize `participants` as an empty array)
   > - Reference activities.json using a relative markdown link
   ```

3. Review the file Copilot creates.

4. **Accept the changes** and save the file.

### Activity: Test the new activity prompt 🧪

1. Open **Copilot Chat** in **Agent** mode.

2. Type `/new-activity` in the chat input. You have two options:

   - Type just `/new-activity` without details — Copilot will ask what the activity should be.
   - Include the details directly: `/new-activity Robotics Club, Tuesdays 4-6 PM, 18 students, build and program LEGO Mindstorms robots`

   <details>
   <summary>💡 Activity ideas to try</summary>

   ```text
   Robotics Club
   ```

   ```text
   Photography Club
   ```

   ```text
   Debate Team
   ```

   ```text
   Astronomy Club
   ```

   ```text
   Cooking Class
   ```

   </details>

3. Watch Copilot work. It should:

   - Ask you for any missing fields (or use the values you provided).
   - Append a new entry to `app/backend/data/activities.json`.
   - Follow the format of existing entries (Title Case key, `participants: []`, etc.).

4. The path-specific instructions from **Step 2** also apply here automatically — so the new entry will follow the conventions even if your prompt didn't mention them all.

5. Refresh the browser. The new activity should appear on the page.

   > 🪧 **Note:** Your prompt file defined **what** to do (add a new activity), and the path-specific instruction file from Step 2 enforced **how** to do it (formatting rules). The two work together!

<details>
<summary>Activity not showing on the website? 🔍</summary>

- Make sure the `uvicorn` server is running with `--reload`.
- Verify the JSON file is still valid (no missing commas, no duplicate keys).
- Check that the new key was added as a top-level property of the root object.

</details>

**🎯 Goal: A single slash command adds a fully compliant new activity to the data file in seconds, without having to remember any of the formatting rules. ✅**

### Activity: Your turn — write another prompt file ✈️

The school portal could automate more than activity creation. Try writing **one** of the following prompt files yourself:

- **Option A:** `/remove-activity` — removes an activity by name from `activities.json` (and prompts the user to confirm before deleting an activity that still has participants).
- **Option B:** `/withdraw-participant` — removes a student email from an activity's `participants` list, given an activity name and email.
- **Option C (advanced):** `/list-activities-by-day` — reads `activities.json` and returns a markdown table of activities grouped by the day of the week they meet, without modifying any files.

**Requirements:**

- The prompt file must be in `.github/prompts/` with the `.prompt.md` extension.
- It must include `agent: agent`, a `description`, and an `argument-hint` in the frontmatter.
- It must reference `app/backend/data/activities.json` so Copilot knows where to look.
- It should respect the conventions from your repository and path-specific instructions.

---

## Phase 3: Custom Agents

So far you've taught Copilot the project's rules (instructions), automated repeatable workflows (prompt files), and you're about to install on-demand expertise (agent skills). But every time you start a chat you still pick the model, allow or restrict tools, and re-explain the persona you want. **Custom agents** bundle all of that — instructions, allowed tools, model preference, and even handoffs to other agents — into a single selectable persona.

In this phase you'll build two agents that work together: an `activities-planner` that can only **read** the codebase and propose a plan, and an `activities-implementer` that picks up that plan and actually edits `activities.json`.

### 📖 Theory: What are Custom Agents?

A custom agent is a Markdown file with YAML frontmatter that defines a reusable Copilot persona. Once defined, it shows up in the agents dropdown next to **Ask** and **Agent** modes.

| Aspect | Details |
| ------ | ------- |
| **File extension** | `.agent.md` (VS Code also detects plain `.md` inside `.github/agents/`) |
| **Default location** | `.github/agents/` (workspace) or `~/.copilot/agents/` (user profile) |
| **Invocation** | Select it from the agents dropdown in Copilot Chat, or type `/agents` to manage them |
| **Scope** | Workspace agents are committed to the repo and shared with the team |

Here's how custom agents relate to what you've already built:

| Building block | Answers… | Example |
| -------------- | -------- | ------- |
| **Instructions** | *How* should things be done? | "Activity names must be Title Case" |
| **Prompt files** | *What* repeatable task should run? | `/new-activity` appends an entry |
| **Custom agents** | *Who* is doing the work, and *with which tools*? | `activities-planner` (read-only) vs `activities-implementer` (edits files) |
| **Agent Skills** | What *expertise* should I apply, only when relevant? | The `pdf` skill loads only when the task involves PDFs |

**Key frontmatter fields:**

| Field | Purpose |
| ----- | ------- |
| `name` | Display name in the agents dropdown (defaults to filename) |
| `description` | One-line placeholder shown in the chat input |
| `tools` | **Allow-list** of tools the agent can use. Leave unset for defaults, `[]` for no tools |
| `model` | Preferred model (string or prioritized array) |
| `handoffs` | Optional list of follow-up buttons that transition to another agent with a pre-filled prompt |
| `target` | `vscode` (default, IDE) or `github-copilot` (Cloud agent) |

> 💡 **Tip:** `tools` is an **allow-list**. Restricting it to read-only tools is the simplest way to guarantee an agent can't accidentally modify your code.

> 📚 Full reference: [Custom agents in VS Code](https://code.visualstudio.com/docs/copilot/customization/custom-agents).

### Activity: Create the `activities-planner` agent 🗺️

This agent reads the codebase and drafts a plan for any change to `activities.json`, but it **cannot edit files**. At the end of the plan, it offers a handoff to the implementer.

1. Open **Copilot Chat** and run the **`Chat: New Custom Agent`** command from the Command Palette (`⇧⌘P` / `Ctrl+Shift+P`). Alternatively, type `/agents` in the chat input and choose **Configure Custom Agents → New Agent (Workspace)**.

2. Choose **Workspace** as the location and name it `activities-planner`. VS Code creates the file at `.github/agents/activities-planner.agent.md`.

3. Replace the scaffolded content with the following (or ask Copilot in Agent mode to do it for you):

   ```markdown
   ---
   name: activities-planner
   description: Drafts a plan for changes to activities.json without editing files.
   tools: ['search', 'codebase', 'usages']
   handoffs:
     - label: Implement this plan
       agent: activities-implementer
       prompt: Implement the plan above, following all project conventions.
   ---

   # Activities Planner

   You help maintain the school activities portal. Your job is to **plan** changes to
   [activities.json](../../app/backend/data/activities.json) — never to edit files.

   When asked to add, remove, or modify an activity:

   1. Read the current state of `activities.json` and any relevant instructions.
   2. Produce a numbered plan describing the exact change (key, fields, values).
   3. Reference the project conventions (Title Case keys, `@githubcopilot.edu` emails,
      schedule format, etc.).
   4. End with a one-line summary and offer the **Implement this plan** handoff.

   Do **NOT** call any editing tools. If asked to edit, refuse and suggest using the
   handoff instead.
   ```

4. Save the file.

### Activity: Create the `activities-implementer` agent 🛠️

This agent takes a plan and applies it to `activities.json`, reusing the `/new-activity` prompt and the path-specific instructions you built earlier.

1. Run **`Chat: New Custom Agent`** again, name it `activities-implementer`, and save it as `.github/agents/activities-implementer.agent.md`.

2. Replace the content with:

   ```markdown
   ---
   name: activities-implementer
   description: Applies activity changes to activities.json following project conventions.
   ---

   # Activities Implementer

   You apply approved changes to
   [activities.json](../../app/backend/data/activities.json).

   - For **new activities**, follow the workflow in
     [new-activity.prompt.md](../prompts/new-activity.prompt.md).
   - For **edits or removals**, modify `activities.json` directly and respect the
     rules in [activities-data.instructions.md](../instructions/activities-data.instructions.md)
     (these apply automatically to the JSON file).
   - Always confirm the change with a short summary of what was added, modified, or removed.
   ```

3. Save the file.

   > 🪧 **Note:** This agent doesn't set a `tools` field, so it inherits the default Agent-mode tools (including editing). The path-specific instructions from Phase 1 Step 2 still apply automatically — the agent doesn't need to re-list those rules.

### Activity: Test the handoff 🔁

1. Open the agents dropdown in Copilot Chat. You should see `activities-planner` and `activities-implementer` alongside the built-in agents.

   <details>
   <summary>Don't see them? 🔍</summary>

   - Confirm the files are at `.github/agents/activities-planner.agent.md` and `.github/agents/activities-implementer.agent.md`.
   - Run **`Chat: Diagnostics`** from the Command Palette to inspect agent loading errors.
   - Restart VS Code if the files were just created.

   </details>

2. Select **`activities-planner`** and send this prompt:

   > ![Static Badge](https://img.shields.io/badge/-Prompt-text?style=social&logo=github%20copilot)
   >
   > ```prompt
   > We want to add a Chess Tournament Team that meets Wednesdays after school.
   > Draft the plan.
   > ```

3. Verify the planner's behavior:

   - It produces a numbered plan referencing the project conventions.
   - **No files were edited** (check the Source Control view — there should be no changes).
   - A handoff button labeled **"Implement this plan"** appears at the end of the response.

4. Click the **Implement this plan** handoff button. VS Code switches to `activities-implementer` with the prompt pre-filled.

5. Submit the handoff prompt. The implementer appends the new entry to `app/backend/data/activities.json`, automatically following every Phase 1 convention (Title Case key, canonical schedule, `participants: []`, etc.).

6. Refresh the browser. The new **Chess Tournament Team** card appears on the page.

   **🎯 Goal: A read-only planner agent drafts a compliant plan and hands it off to an implementer agent that ships the change — all from a single dropdown selection. ✅**

> 🪧 **Note:** Custom agents in VS Code can do much more — scoped hooks, MCP servers, cloud (`target: github-copilot`) execution, and self-referential subagents. See the [official documentation](https://code.visualstudio.com/docs/copilot/customization/custom-agents) for the full picture.

Next, you'll install on-demand domain expertise with **Agent Skills**.

---

## Phase 4: Agent Skills

So far you've taught Copilot about your project (instructions), automated workflows (prompt files), and defined personas (custom agents). All of these are **always-on** — they're injected into every request that matches their scope. But what about specialized expertise you only need occasionally? Generating a PDF, calling a payments API, debugging a memory dump? Loading that kind of knowledge into every prompt would burn context for no reason.

That's the niche **Agent Skills** fill: bundles of **expert knowledge plus supporting files** that Copilot loads **only when the task calls for them**.

### 📖 Theory: What are Agent Skills?

An agent skill is a folder containing a `SKILL.md` file (and optionally helper docs, scripts, or templates). Copilot reads each skill's frontmatter at startup, and at request time it decides — based on the skill's `description` and your prompt — whether to pull the rest of the skill into context.

| Aspect | Details |
| ------ | ------- |
| **File name** | `SKILL.md` (one per skill, inside its own folder) |
| **Discovery locations** | `.github/skills/<name>/` (workspace), `~/.copilot/skills/<name>/` (user), or inside a plugin's `skills/` folder |
| **Frontmatter** | `name` and `description` (required). `license` and others are optional |
| **Content** | Markdown — best practices, code samples, decision trees, do's/don'ts |
| **Bundled assets** | Reference docs (`reference.md`, `forms.md`), runnable `scripts/`, templates — anything the skill author wants Copilot to read or execute |
| **Loaded when** | Copilot judges the user's task matches the skill's `description` |

#### How skills differ from everything else you've built

| Customization | When it activates | Best for |
| ------------- | ----------------- | -------- |
| **Repository instructions** | Every request in the repo | Project-wide conventions |
| **Path-specific instructions** | Files matching `applyTo` glob | Format rules for a specific area |
| **Prompt files** | User invokes `/name` explicitly | Repeatable workflows the user triggers |
| **Custom agents** | User picks the agent from the dropdown | Persistent persona + tool restrictions |
| **Agent skills** | Copilot auto-loads when task matches `description` | Specialized expertise that's only sometimes relevant |

#### Progressive disclosure: the killer feature

A skill can be huge — Anthropic's official `pdf` skill includes `SKILL.md`, `reference.md`, `forms.md`, and a `scripts/` directory totaling thousands of lines. None of that consumes context until you ask a PDF-related question. When you do, Copilot reads `SKILL.md` (the entry point), and `SKILL.md` itself tells Copilot when to read the deeper files (e.g., "If the user needs to fill out a PDF form, read FORMS.md"). This is called **progressive disclosure** and it's why skills scale where instructions can't.

> ❕ **Important:** The `description` is **the most critical field**. Copilot uses it to decide whether to load the skill. Be specific and include keywords matching the tasks you want to trigger on. A vague description means the skill never activates.

> 💡 **Tip:** Skills can reference other workspace files via plain markdown links, and they can include executable `scripts/` that Copilot can invoke through the terminal. That's what lets a skill go beyond "extra instructions" into "domain capability."

See the [VS Code Docs: Agent Skills](https://code.visualstudio.com/docs/copilot/customization/agent-skills) page for the full reference, and browse [`github.com/anthropics/skills`](https://github.com/anthropics/skills) for production-quality examples.

### Activity: Install the Anthropic PDF skill 📥

Rather than write a skill from scratch, let's **install a real one** — Anthropic's `pdf` skill — and use it to generate a PDF directly from `activities.json`. This mirrors how most teams will consume skills in practice: pick one off the shelf, drop it into the right folder, done.

1. **Download the skill** from the [`anthropics/skills`](https://github.com/anthropics/skills/tree/main/skills/pdf) repo. The fastest way is to clone the repo into a temp folder and copy just the `pdf/` directory:

   ```bash
   git clone --depth 1 https://github.com/anthropics/skills.git /tmp/anthropics-skills
   mkdir -p .github/skills
   cp -r /tmp/anthropics-skills/skills/pdf .github/skills/pdf
   ```

   > 🪧 **License note:** The Anthropic PDF skill is shipped under its own `LICENSE.txt` (see the file inside the skill folder). Review it before redistributing the skill outside your own use.

2. **Verify the final layout** matches:

   ```text
   .github/skills/pdf/
   ├── LICENSE.txt
   ├── SKILL.md
   ├── forms.md
   ├── reference.md
   └── scripts/
   ```

3. **Install the Python libraries** the skill relies on so Copilot can actually run the code it suggests:

   ```bash
   pip install reportlab pypdf pdfplumber
   ```

4. **Restart VS Code** (or reload the window) so Copilot picks up the new skill.

5. **Confirm the skill is loaded.** Open Copilot Chat in **Agent** mode and ask:

   > ```prompt
   > What skills do you have available?
   > ```

   Copilot should list `pdf` in its response. If it doesn't appear, reload the window (`Ctrl+Shift+P` → **Developer: Reload Window**) and try again.

### Activity: Generate a PDF roster from activities.json 📄

Now let's put the skill to work on real project data.

1. Open **Copilot Chat** in **Agent** mode.

2. Ask Copilot to generate a one-page PDF roster for a single activity:

   > ![Static Badge](https://img.shields.io/badge/-Prompt-text?style=social&logo=github%20copilot)
   >
   > ```prompt
   > Read app/backend/data/activities.json and generate a one-page PDF roster
   > for "Chess Club" at output/chess-club-roster.pdf. Include the activity
   > name as the title, the description, the schedule, max participants,
   > and the list of currently signed-up student emails. Make it look like
   > something a teacher would actually print.
   ```

3. Watch Copilot's response. You should see it:

   - Reference the `pdf` skill in its tool/skill usage indicators.
   - Read `app/backend/data/activities.json`.
   - Write a short Python script that uses `reportlab` (the library `SKILL.md` recommends for creating PDFs) and run it.
   - Produce `output/chess-club-roster.pdf`.

4. Open the resulting PDF. You should have a real, printable roster — generated end-to-end by Copilot using the skill's expertise.

   **🎯 Goal: A real PDF file is generated from `activities.json` by Copilot, using the `pdf` skill's bundled knowledge without you writing any PDF code. ✅**

### Activity: Push the skill further 🚀

Skills shine when one prompt triggers multiple techniques from the bundled knowledge. Try one of these:

- **Bulk generation:** *"Generate a one-page roster PDF for every activity in `activities.json`, saved under `output/rosters/`."*
- **Merged handbook:** *"Generate a single multi-page PDF handbook at `output/activities-handbook.pdf` with a cover page, table of contents, and one page per activity."* (Exercises both `reportlab` for creation and `pypdf` for merging.)
- **Read-back:** *"Extract the text from `output/activities-handbook.pdf` and list every activity name you find."* (Exercises `pdfplumber` for extraction — proves the skill covers both produce and consume.)

Notice how the **same skill** powers all three tasks because `SKILL.md` documents *when* to reach for which library.

<details>
<summary>Skill not being applied? 🤷</summary>

- Confirm the path is exactly `.github/skills/pdf/SKILL.md` at the workspace root.
- Run **`Chat: Diagnostics`** from the Command Palette — the `pdf` skill should be listed.
- If Copilot writes PDF code but ignores the skill's guidance, mention PDFs more explicitly in your prompt (the `description` triggers on PDF-related keywords).
- Restart VS Code if you just copied the skill in.

</details>

> 💡 **Instructions, prompt files, agents, or skills?** Use **instructions** for rules that should always apply. Use **prompt files** for repeatable workflows the user explicitly triggers. Use **custom agents** for personas with specific tool restrictions or handoffs. Use **skills** for specialized expertise — including bundled scripts and reference docs — that should only load when the task calls for it.

Next, you'll break out of Copilot's text-only world and let it call **real tools** against your project data with **MCP**.

---

## Phase 5: Model Context Protocol (MCP)

Up to now, every customization you built (instructions, prompt files, agents, skills) lives **inside Copilot's world** — it's text the model reads. But what if you want Copilot to do something it physically cannot do on its own? Read a live database. Query an internal API. Compute statistics from data it can't see in the editor.

That's where **MCP** comes in.

### 📖 Theory: What is MCP?

**Model Context Protocol (MCP)** is "USB-C for AI tools" — a standard way for an AI client (Copilot Agent) to talk to a server process that exposes **tools**, **resources**, and **prompts**.

```text
┌──────────────┐      MCP       ┌──────────────────────┐
│ Copilot Chat │  ◀──────────▶  │ MCP Server           │
│  (client)    │   stdio/JSON   │ list_activities()    │
└──────────────┘                │ get_signups_count()  │
                                └──────────────────────┘
```

Two properties worth internalizing:

- **Each MCP server runs as a separate process.** That's the security boundary. Copilot can't accidentally read your filesystem because of an MCP bug — only what the server explicitly exposes.
- **You register servers in `.vscode/mcp.json`.** VS Code launches them on demand, you accept a trust prompt the first time, and the tools show up in **Configure Tools** in the chat input.

> 💡 **Tip:** You can also browse a registry of community MCP servers at [`github.com/mcp`](https://github.com/mcp) — GitHub, Postgres, Sentry, Playwright, Microsoft Learn, and many more.

### Activity: Build your own MCP server 🛠️

You're going to expose the school activities (the same data that powers the website) as an MCP server, so that Copilot can answer questions like *"which activity is closest to full?"* by calling real tools instead of guessing.

1. **Install the MCP Python SDK** into your virtual environment:

   ```bash
   pip install "mcp[cli]>=1.0"
   ```

2. **Create the server directory at the workspace root** (not inside `app/`):

   ```bash
   mkdir -p mcp_servers
   touch mcp_servers/__init__.py
   ```

   > 🪟 **PowerShell:** `touch` is not available. Use `New-Item` instead:
   > ```powershell
   > New-Item mcp_servers/__init__.py
   > ```

3. **Create the server file at `mcp_servers/school_activities_server.py`.** You can write it yourself or ask Copilot in **Agent** mode:

   > ![Static Badge](https://img.shields.io/badge/-Prompt-text?style=social&logo=github%20copilot)
   >
   > ```prompt
   > Create mcp_servers/school_activities_server.py — a FastMCP Python server
   > named "school-activities" that reads app/backend/data/activities.json
   > and exposes two tools:
   >  - list_activities() -> list[str]: return the activity names
   >  - get_signups_count(activity: str) -> int: return how many students
   >    have signed up for that activity. Raise ValueError if unknown.
   > Run the server with mcp.run() under `if __name__ == "__main__"`.
   > Use pathlib to resolve activities.json relative to the workspace root.
   > ```

   The result should look similar to:

   <details>
   <summary>Expected content 📄</summary>

   ```python
   """School activities MCP server — exposes the activities data file to
   MCP-aware clients (e.g., GitHub Copilot Agent Mode)."""

   import json
   from pathlib import Path

   from mcp.server.fastmcp import FastMCP

   ACTIVITIES_FILE = (
       Path(__file__).resolve().parent.parent
       / "app" / "backend" / "data" / "activities.json"
   )

   mcp = FastMCP("school-activities")


   def _load() -> dict:
       with open(ACTIVITIES_FILE, "r", encoding="utf-8") as f:
           return json.load(f)


   @mcp.tool()
   def list_activities() -> list[str]:
       """Return the names of all extracurricular activities."""
       return list(_load().keys())


   @mcp.tool()
   def get_signups_count(activity: str) -> int:
       """Return the number of students signed up for a given activity.

       Raises ValueError if the activity is unknown.
       """
       data = _load()
       if activity not in data:
           raise ValueError(f"Unknown activity: {activity!r}")
       return len(data[activity].get("participants", []))


   if __name__ == "__main__":
       mcp.run()
   ```

   </details>

   > 💡 **Tip:** The docstrings on each `@mcp.tool()` are how Copilot decides when to call them. Keep them short, specific, and accurate — they earn their keep.

4. **Register the server in `.vscode/mcp.json`** (create the file if it doesn't exist). Ask Copilot to do it, or paste:

   ```jsonc
   {
     "servers": {
       "school-activities": {
         "type": "stdio",
         "command": "${workspaceFolder}/venv/bin/python",
         "args": ["-m", "mcp_servers.school_activities_server"]
       }
     }
   }
   ```

   > 🪟 **Windows:** change `"command"` to `"${workspaceFolder}/venv/Scripts/python.exe"`. Everything else is identical.

5. **Start the server.** Open the Command Palette → **MCP: List Servers** → select `school-activities` → **Start Server**. Accept the trust prompt the first time.

6. **Verify the tools are wired up.** Open **Configure Tools** in the chat input. You should see `list_activities` and `get_signups_count` listed under `school-activities`.

### Activity: Drive the MCP server with prompts 🎯

1. **Single tool call:**

   > ![Static Badge](https://img.shields.io/badge/-Prompt-text?style=social&logo=github%20copilot)
   >
   > ```prompt
   > Use the school-activities MCP server to list all activities.
   > ```

   Copilot should call `list_activities()` once and return the names.

2. **Chained tool calls — the aha moment:**

   > ![Static Badge](https://img.shields.io/badge/-Prompt-text?style=social&logo=github%20copilot)
   >
   > ```prompt
   > Which activity is closest to full? Use the school-activities MCP server.
   > ```

   To answer, Copilot must call `list_activities()` once, then `get_signups_count(...)` for each result, then compare against `max_participants` (which it can see by reading `activities.json`). The full chain of tool calls is visible inline in the transcript.

3. **Negative case — see a tool error surface gracefully:**

   > ![Static Badge](https://img.shields.io/badge/-Prompt-text?style=social&logo=github%20copilot)
   >
   > ```prompt
   > Get the signup count for "Knitting Club" using the school-activities MCP.
   > ```

   `Knitting Club` doesn't exist; the tool raises `ValueError`; Copilot reports it gracefully (and may suggest calling `list_activities()` first).

**🎯 Goal: The chat transcript shows real MCP tool calls in order, and Copilot's final answer is grounded in the data your server exposed — not invented. ✅**

<details>
<summary>Server not starting? 🤷</summary>

- Make sure `mcp[cli]>=1.0` is installed in the same venv your `.vscode/mcp.json` `command` points to.
- Try running the server manually from the workspace root: `python -m mcp_servers.school_activities_server`. If it errors out, fix the error first.
- Check that `app/backend/data/activities.json` exists and is valid JSON.
- Restart VS Code if the server was just registered.

</details>

### Activity (optional): Consume an off-the-shelf MCP from a registry 🌐

Building your own MCP is the load-bearing skill — but most teams will spend more time **consuming** community MCPs than authoring them. The **Microsoft Learn MCP** is a great example: it gives Copilot grounded access to official Microsoft documentation.

1. Install the Microsoft Learn MCP from the [MCP registry](https://github.com/mcp/microsoftdocs/mcp). You can use the one-click **Install** button on the registry page, or add it manually to `.vscode/mcp.json`:

   ```json
   {
     "servers": {
       "microsoft-learn": {
         "type": "http",
         "url": "https://learn.microsoft.com/api/mcp"
       }
     }
   }
   ```

   > 💡 **Note:** The Microsoft Learn MCP uses HTTP transport — no local process to install or manage. VS Code connects directly to the hosted endpoint.

2. Verify it's running (**MCP: List Servers**), then try:

   > ![Static Badge](https://img.shields.io/badge/-Prompt-text?style=social&logo=github%20copilot)
   >
   > ```prompt
   > Search Microsoft docs and explain how MCP works in GitHub Copilot.
   > Cite the exact learn.microsoft.com URLs you used.
   > ```

   Copilot fetches real documentation pages and cites them — answers are grounded in source material, not just training data.

> 🪧 **Note:** The MCP world is large. We only covered **tools** today; MCP also supports **resources** (files/URIs the model can read) and **prompts** (reusable prompt templates the server exposes). Browse the [MCP registry](https://github.com/mcp) for hundreds of community servers — GitHub, Postgres, Sentry, Playwright, and more.

---

## Phase 5b: Agent Hooks (Preview)

Everything you've built so far — instructions, prompt files, agents, skills, MCP servers — guides Copilot through text. Agent Hooks let you go one step further: **run real shell code** at defined points in the agent lifecycle, so validation and context injection happen automatically, not just when you remember to ask.

> ⚠️ **Preview feature.** Agent Hooks require VS Code 1.100+ and are gated behind the `chat.useCustomAgentHooks` setting. Behavior may change before GA.

### 📖 Theory: Hook Basics

| Concept | Details |
| ------- | ------- |
| **Lifecycle events** | `UserPromptSubmit`, `PostToolUse`, `PreToolUse`, `AgentStart`, `AgentStop`, `SubagentStart`, `SubagentStop`, `Error` |
| **Scope** | **Workspace** — `.github/hooks/*.json`, loaded for every chat session in the repo; **Agent** — `hooks:` block in `.agent.md` frontmatter, loaded only when that agent is active; **Personal** — user-settings hooks, outside this lab |
| **Input** | The hook process receives a JSON object on **stdin** with fields like `hookEventName`, `tool_name`, `tool_input`, `tool_response`, and `prompt` |
| **Output** | Write JSON to **stdout** with `hookSpecificOutput.additionalContext` to inject context, `systemMessage` for a user-visible message, or `continue: false` to abort |
| **Exit codes** | `0` = success; non-zero = hook error (Copilot surfaces a warning) |

Two hooks will enforce quality in this project:

1. **`PostToolUse` (workspace scope)** — After every file edit, validate that `activities.json` is still valid JSON and follows the schema. Fires for all agents.
2. **`UserPromptSubmit` (agent scope, `activities-implementer` only)** — Before Copilot calls any tool, check whether the prompt mentions a quoted activity name that doesn't exist yet, and offer to create it with `/new-activity`.

---

### Activity 1 — Workspace-scoped PostToolUse validation hook

**Goal:** Automatically catch `activities.json` schema errors the moment any agent edits the file.

#### Step 0 — Enable the preview setting

*Agent Hooks is a preview feature that must be explicitly opted in to. Adding this setting activates hook loading for the workspace.*

1. Open `.vscode/settings.json` at the workspace root. If the file doesn't exist yet, create it.

2. Add or update the file so it contains:

   ```json
   {
     "chat.useCustomAgentHooks": true
   }
   ```

3. Save the file and **reload the VS Code window** (`Ctrl+Shift+P` → *Developer: Reload Window*) so the new setting takes effect.

#### Step 1 — Create the validation script

*The hook runs a Python script — keeping the logic in a `.py` file instead of an inline shell command makes it easy to test, version, and extend.*

1. Create the required folder structure inside `.github/`:

   ```
   .github/
   └── hooks/
       └── scripts/
   ```

   You can do this from the VS Code Explorer (right-click → **New Folder**) or from a terminal:

   ```bash
   mkdir -p .github/hooks/scripts
   ```

2. Inside `.github/hooks/scripts/`, create a new file called **`validate_activities.py`** and paste the following content:

   ```python
   #!/usr/bin/env python3
   """PostToolUse hook: validate activities.json after any file edit.

   VS Code sends a JSON object via stdin describing the tool call that just
   finished. This script checks whether activities.json is still valid JSON
   and follows the expected schema, then writes feedback to stdout that Copilot
   injects into the conversation context.
   """
   import json
   import pathlib
   import sys

   data = json.loads(sys.stdin.read())

   # Only run when the agent edited a file — ignore unrelated tool calls
   if data.get("tool_name") not in (
       "editFiles",
       "create_file",
       "replace_string_in_file",
       "insert_edit_into_file",
   ):
       sys.exit(0)

   activities_file = pathlib.Path("app/backend/data/activities.json")
   if not activities_file.exists():
       sys.exit(0)

   try:
       activities = json.loads(activities_file.read_text(encoding="utf-8"))
   except json.JSONDecodeError as exc:
       print(
           json.dumps(
               {
                   "hookSpecificOutput": {
                       "hookEventName": "PostToolUse",
                       "additionalContext": (
                           f"⚠️ activities.json is invalid JSON after your edit: {exc}. "
                           "Fix the syntax before continuing."
                       ),
                   }
               }
           )
       )
       sys.exit(0)

   errors = []
   for name, entry in activities.items():
       if not isinstance(entry.get("max_participants"), int):
           errors.append(f"'{name}': max_participants must be an integer, not a string.")
       if not isinstance(entry.get("participants"), list):
           errors.append(f"'{name}': participants must be an array.")

   if errors:
       print(
           json.dumps(
               {
                   "hookSpecificOutput": {
                       "hookEventName": "PostToolUse",
                       "additionalContext": (
                           "⚠️ activities.json has schema issues after your last edit: "
                           + "; ".join(errors)
                           + " Fix these before continuing."
                       ),
                   }
               }
           )
       )
   ```

   What the script does:
   - **Ignores** tool calls that are not file edits (searches, MCP calls, etc.) so it stays silent when not needed.
   - **Reports** a clear error message if the JSON syntax is broken after an edit.
   - **Checks** every entry for the two most common schema violations: `max_participants` as a string and `participants` as a non-array.

#### Step 2 — Create the workspace hook configuration file

*VS Code discovers workspace hooks from any `*.json` file inside `.github/hooks/`. Separating concerns into named files (one per guard) keeps hook configuration readable as the project grows.*

1. Inside `.github/hooks/`, create a new file called **`activities-guard.json`** and paste the following content:

   ```json
   {
     "hooks": {
       "PostToolUse": [
         {
           "type": "command",
           "command": "python .github/hooks/scripts/validate_activities.py",
           "windows": "python .github/hooks/scripts/validate_activities.py"
         }
       ]
     }
   }
   ```

   > 💡 The `windows` key provides the same command and ensures the correct Python executable name is used on Windows. On macOS/Linux the `command` key is used.

2. Verify the final directory layout looks like this:

   ```
   .github/
   └── hooks/
       ├── activities-guard.json        ← discovered automatically by VS Code
       └── scripts/
           └── validate_activities.py
   ```

#### Step 3 — Test the validation hook

*Deliberately breaking the file confirms the hook fires and that Copilot self-corrects from the injected context — that's the only way to trust automation.*

1. Open `app/backend/data/activities.json`.
2. Change any `max_participants` value from a number to a string (e.g., `"20"`).
3. Save the file.
4. Open GitHub Copilot Chat and send any message (e.g., "What activities are available?").
5. Observe that Copilot's next response acknowledges the schema issue and offers to fix it.
6. Revert your deliberate change.

---

### Activity 2 — Agent-scoped UserPromptSubmit hook

**Goal:** When a user asks `activities-implementer` about an activity that doesn't exist yet, have Copilot proactively offer to create it — before attempting any tool call that would fail.

#### Step 1 — Understand why UserPromptSubmit, not PostToolUse

*`PostToolUse` fires only after a tool completes successfully. If the MCP server raises a `ValueError` for an unknown activity, there is no "success" event — the hook never fires. `UserPromptSubmit` fires before any tool is called, making it reliable for proactive checks.*

The hook reads `prompt` from the stdin JSON and scans for quoted strings (e.g., `"Knitting Club"`). If a quoted name is absent from `activities.json`, it injects context telling Copilot to inform the user and offer the `/new-activity` workflow.

#### Step 2 — Create the hook script

*Keeping the detection logic in a `.py` file makes it easy to test independently and extend with additional rules — for example, checking for valid schedule formats in the prompt.*

1. Inside `.github/hooks/scripts/`, create a new file called **`check_activity_exists.py`** and paste the following content:

   ```python
   #!/usr/bin/env python3
   """UserPromptSubmit hook (scoped to activities-implementer):

   Detect quoted activity names in the user's prompt that don't exist in
   activities.json and inject a suggestion to create them with /new-activity.

   VS Code sends the prompt text via stdin as a JSON object. If the prompt
   contains quoted strings (e.g., "Knitting Club") that are not in the data
   file, this script outputs an additionalContext field that Copilot uses to
   guide its response before any tool call is made.
   """
   import json
   import pathlib
   import re
   import sys

   data = json.loads(sys.stdin.read())
   prompt = data.get("prompt", "")

   activities_file = pathlib.Path("app/backend/data/activities.json")
   if not activities_file.exists():
       sys.exit(0)

   try:
       activities = json.loads(activities_file.read_text(encoding="utf-8"))
   except json.JSONDecodeError:
       sys.exit(0)

   # Match quoted strings in the prompt, e.g.: "Knitting Club"
   quoted_names = re.findall(r'"([^"]+)"', prompt)
   missing = [n for n in quoted_names if n not in activities]

   if missing:
       names = ", ".join(f'"{{n}}"' for n in missing)
       print(
           json.dumps(
               {
                   "hookSpecificOutput": {
                       "hookEventName": "UserPromptSubmit",
                       "additionalContext": (
                           f"The following activities were not found in activities.json: {names}. "
                           "Inform the user that the activity does not exist. "
                           "Ask whether they would like to create it. "
                           "If they agree, use the /new-activity workflow to add it "
                           "with all required fields."
                       ),
                   }
               }
           )
       )
   ```

   Key points about this script:
   - The regex `r'"([^"]+)"'` matches **straight double-quoted** strings in the prompt (e.g., `"Knitting Club"`).
   - It cross-references the matched names against the top-level keys in `activities.json`.
   - Only missing names trigger output — existing names pass silently, so Copilot is not interrupted unnecessarily.

#### Step 3 — Add the hook to the agent frontmatter

*Agent-scoped hooks are declared directly in the `.agent.md` frontmatter. This scope ensures the "offer to create" suggestion appears only when the user is talking to `activities-implementer`, not in every chat window.*

1. Open `.github/agents/activities-implementer.agent.md`.

2. The file starts with a YAML frontmatter block between `---` markers. Add the `hooks:` block so the complete frontmatter looks like this:

   ```yaml
   ---
   name: activities-implementer
   description: Applies activity changes to activities.json following project conventions.
   hooks:
     UserPromptSubmit:
       - type: command
         command: "python .github/hooks/scripts/check_activity_exists.py"
         windows: "python .github/hooks/scripts/check_activity_exists.py"
   ---
   ```

3. Save the file. Leave everything below the closing `---` (the agent body) exactly as it is.

#### Step 4 — Test the hook

*An end-to-end test confirms the complete chain: user prompt → hook fires → Copilot receives context → response guides the user correctly.*

1. Switch to the **`activities-implementer`** agent in GitHub Copilot Chat.
2. Send:
   ```
   How many students signed up for "Underwater Basket Weaving"?
   ```
3. Copilot should reply that `"Underwater Basket Weaving"` doesn't exist and ask if you'd like to create it.
4. Reply **yes** — Copilot should invoke the `/new-activity` prompt file to walk you through adding it.
5. Once added, repeat the original question — this time Copilot should answer normally.

<details>
<summary>🔧 Troubleshooting</summary>

- **Hook not firing?** Verify `"chat.useCustomAgentHooks": true` is in `.vscode/settings.json` and reload the VS Code window.
- **Python not found?** Make sure `python` is on your `PATH`. On some systems you may need `python3` — update both the `command` and `windows` fields accordingly.
- **No output even when a name is missing?** Check that you used straight double quotes (`"`) in your prompt, not curly/smart quotes.
- **Script errors?** Run the script manually: `echo '{"prompt": "\"Knitting Club\""}' | python .github/hooks/scripts/check_activity_exists.py` — you should see JSON on stdout.

</details>

---

## Phase 6: Plugins

You've now built a small toolbox of customizations: a path-specific instruction file, a prompt file, two custom agents, an agent skill, and an MCP server. Each one lives in a different folder under `.github/`, `.vscode/`, or `mcp_servers/`. That's fine for your own workspace — but how do you **share** all of it with a teammate?

That's what **plugins** are for: a single directory that bundles all of these customizations into one distributable package.

### 📖 Theory: What are Plugins?

VS Code Copilot plugins are **bundles** that can include any combination of:

1. **Slash commands** — custom `/` commands in chat
2. **Agent skills** — on-demand instructions and scripts
3. **Custom agents** — the `.agent.md` files you created in Phase 3
4. **Hooks** — shell commands that fire at agent lifecycle points
5. **MCP servers** — the `.mcp.json` entries you built in Phase 5

You already built pieces #3 and #5 individually. Plugins wrap them for **distribution and discovery**.

**Standard plugin directory layout:**

```text
my-plugin/
├── plugin.json              # Required — plugin identity and pointers
├── agents/
│   └── reviewer.agent.md    # Custom agents go here
├── skills/
│   └── tester/
│       └── SKILL.md         # Agent skills go here
├── hooks.json               # Hook configuration (optional)
└── .mcp.json                # MCP server definitions (optional)
```

**The `plugin.json` manifest** has these fields:

| Field | Required | Purpose |
| ----- | -------- | ------- |
| `name` | ✅ | Kebab-case identifier. Only lowercase letters, numbers, and hyphens. **No slashes or colons** — invalid names silently fail to load. |
| `description` | | Brief description (max 1024 chars) |
| `version` | | Semantic version (e.g., `1.0.0`) |
| `author` | | Object with `name` (required), `email`, `url` |
| `agents` | | Path to agent directory (defaults to `agents/`) |
| `skills` | | Path to skill directory (defaults to `skills/`) |
| `mcpServers` | | Path to MCP config file or inline definitions |
| `hooks` | | Path to hooks config file or inline hooks object |

> ⚠️ **Plugins are in preview.** They are gated by the `chat.plugins.enabled` setting, which is often **managed at the organization level**. If it's grayed out in your VS Code settings, contact your admin.

> 🪧 **Security note:** Plugins can contain hooks and MCP servers that execute code. Always review the contents of a plugin before installing.

### Activity: Package your work as a plugin 📦

You'll bundle the **`activities-implementer`** custom agent (from Phase 3) and the **`school-activities`** MCP server (from Phase 5) into a single `my-school-plugin/` directory.

1. **Create the plugin root** at the workspace root:

   ```bash
   mkdir my-school-plugin
   ```

2. **Create `my-school-plugin/plugin.json`** with the manifest:

   ```json
   {
     "name": "my-school-plugin",
     "description": "Activities implementer agent and school-activities MCP server for the GitHub Copilot High School app",
     "version": "0.1.0",
     "author": {
       "name": "Your Name"
     },
     "agents": "agents/",
     "mcpServers": ".mcp.json"
   }
   ```

3. **Copy the custom agent into the plugin:**

   ```bash
   mkdir my-school-plugin/agents
   cp .github/agents/activities-implementer.agent.md \
      my-school-plugin/agents/activities-implementer.agent.md
   ```

   > 💡 **Note:** You're moving a copy of the agent from Phase 3 into the plugin structure. The file stays identical — same frontmatter, same system prompt.

4. **Create `my-school-plugin/.mcp.json`** to reference the MCP server you built in Phase 5:

   ```json
   {
     "mcpServers": {
       "school-activities": {
         "type": "stdio",
         "command": "python",
         "args": ["-m", "mcp_servers.school_activities_server"]
       }
     }
   }
   ```

   > 🪧 **Key differences from `.vscode/mcp.json`:**
   >
   > - Top-level key is `mcpServers` (not `servers`).
   > - `command` is simply `"python"` (not an absolute venv path). In a real distributed plugin you'd either bundle a runtime, document a Python version requirement, or use `npx` for a JavaScript MCP server. For this lab, assume the user has Python in their PATH.

**🎯 Goal: A self-contained `my-school-plugin/` directory at the workspace root with a valid `plugin.json` manifest, the custom agent inside `agents/`, and the MCP server definition in `.mcp.json`. ✅**

### Install & verify (read-through) 🔍

> 📖 **Read-through** — how VS Code would discover and load your plugin.

Plugins are in preview and the local-install flow is fiddly. Here's what the path looks like end-to-end so you know where this is going.

**Enabling plugin support.** Add to `.vscode/settings.json`:

```json
{
  "chat.plugins.enabled": true
}
```

If this setting is grayed out, your org admin controls it.

**Registering the plugin path for local development:**

Add this to `.vscode/settings.json` (alongside `chat.plugins.enabled`):

```json
{
  "chat.pluginLocations": {
    "${workspaceFolder}/my-school-plugin": true
  }
}
```

**What you'd see after reloading the window:**

1. The `activities-implementer` agent shows up in the agent picker.
2. **MCP: List Servers** lists `school-activities` (auto-started by the plugin).
3. `list_activities` and `get_signups_count` appear in **Configure Tools**.

**Testing the plugin from a separate workspace:**

`chat.pluginLocations` accepts absolute paths, so you can load the plugin into any workspace without moving or copying any files. This lets you simulate exactly what another developer would experience after installing it.

1. Open VS Code in a **new, empty folder** (File → Open Folder → pick or create an empty directory).
2. Create `.vscode/settings.json` in that folder with an absolute path to your `my-school-plugin/` directory. Replace the placeholder with the actual path on your machine:

```json
{
  "chat.plugins.enabled": true,
  "chat.pluginLocations": {
    "/absolute/path/to/your/my-school-plugin": true
  }
}
```

> 🪟 **Windows example:** `"C:\\Users\\you\\projects\\customize-copilot\\my-school-plugin": true`
> 🍎 **macOS/Linux example:** `"/home/you/projects/customize-copilot/my-school-plugin": true`

3. Reload the window (**Developer: Reload Window** from the Command Palette).
4. Verify in the new workspace:
   - `activities-implementer` appears in the agent picker (@ menu in the Chat view).
   - **MCP: List Servers** shows `school-activities` started.
   - **Configure Tools** lists `list_activities` and `get_signups_count`.
5. Ask `@activities-implementer` a question — it should answer using tools from the MCP server even though the new workspace contains no source code at all.

> **Note:** The `command: "python"` in `my-school-plugin/.mcp.json` relies on `python` being on the system PATH. If the server fails to start in the new workspace, use the absolute path to the venv interpreter instead (e.g. `C:\\path\\to\\your\\.venv\\Scripts\\python.exe` on Windows or `/path/to/your/.venv/bin/python` on macOS/Linux).

**Alternative: publish to Git and install from source**

If `chat.pluginLocations` doesn't load the plugin (it's a preview feature and may behave differently across VS Code versions), publish the plugin to a Git repository and install it properly instead:

1. Initialize a Git repo inside `my-school-plugin/`, commit the files, and push to a new remote:

   ```bash
   cd my-school-plugin
   git init
   git add .
   git commit -m "initial plugin"
   gh repo create my-school-plugin --public --source=. --remote=origin --push
   ```

2. In any workspace, open the Command Palette and run **Chat: Install Plugin From Source**, then enter your repository URL:

   ```
   https://github.com/YOUR-USERNAME/my-school-plugin
   ```

3. VS Code clones the plugin into its own cache (`%APPDATA%\Code\agentPlugins\` on Windows) and registers it globally — no path configuration needed. The plugin will now appear in **Extensions: Agent Plugins** and work in every workspace.

> **Why this matters:** `chat.pluginLocations` is a development-only shortcut. Publishing to Git is how your plugin would actually reach other developers — they install it once and it works everywhere on their machine, isolated from your source files.

**Where this is going (production):**

In production you'd publish the plugin to a Git repository and share it via:

- **Plugin marketplaces** — e.g., [`github/awesome-copilot`](https://github.com/github/awesome-copilot). Users browse and install from the **Agent Plugins** view in the Extensions sidebar.
- **Direct Git URL** — users install with **Chat: Install Plugin From Source** from the Command Palette.

Local `chat.pluginLocations` is mostly for development and testing — once your plugin is ready, you push it to a Git repo and share the URL.

**🎯 Goal: You understand how plugins bundle agents + MCP servers + skills + hooks into a single distributable package, what the `plugin.json` manifest looks like, and how VS Code discovers and loads them. ✅**

---

## Congratulations! 🎉

You've completed **Lab 02 — Customizing GitHub Copilot**! Here's a recap of what you learned:

| Phase | What You Did |
| ----- | ------------ |
| **Phase 1 · Step 1** | Set up custom instructions: organization (theory), repository-wide (hands-on), and personal (theory) |
| **Phase 1 · Step 2** | Built path-specific custom instructions for `app/backend/data/**/*.json` and used them to clean and fix activity entries |
| **Phase 2** | Created a reusable prompt file (`/new-activity`) to automate adding new activities |
| **Phase 3** | Defined two custom agents (`activities-planner` read-only and `activities-implementer`) and chained them with handoffs |
| **Phase 4** | Installed Anthropic's `pdf` agent skill and used it to generate real PDFs (rosters, handbook) directly from `activities.json` |
| **Phase 5** | Wrote and registered the `school-activities` MCP server in Python and drove it from Copilot with chained tool calls |
| **Phase 5b** | Created a workspace-scoped JSON validation hook and an agent-scoped `UserPromptSubmit` hook on `activities-implementer` that detects unknown activities and offers to create them via `/new-activity` |
| **Phase 6** | Bundled the custom agent + MCP server into a `my-school-plugin/` plugin package ready for distribution |

### Key Takeaways

- **Custom instructions** eliminate repetitive guidance — set them once, benefit every time.
- **Three levels** let you tailor Copilot at the organization, repository, and personal scope.
- **Two types of repository instructions**: repository-wide (`copilot-instructions.md`) and path-specific (`*.instructions.md`).
- **Priority order**: Personal > Repository > Organization — personal preferences always win.
- **Path-specific custom instructions** (`*.instructions.md`) target only the files that need them using glob patterns.
- **Prompt files** (`*.prompt.md`) package multi-step workflows into reusable slash commands.
- **Custom agents** (`*.agent.md`) bundle persona, tool restrictions, model preference, and handoffs into one selectable role — and compose with instructions, prompt files, and agent skills.
- **Agent Skills** (`SKILL.md`) bundle on-demand expertise plus supporting files (scripts, references) that Copilot loads only when the task matches the skill's `description` — enabling progressive disclosure of large knowledge bases.
- **MCP servers** expose data and capabilities Copilot can't reach on its own — each one is an isolated process with its own tool surface.
- **Agent hooks** fire at defined lifecycle points and execute real shell code — deterministic automation that complements natural-language instructions.
- **Plugins** bundle agents, MCP servers, skills, and hooks into a single distributable package — the unit of sharing across teams and the community.
- **Agent Mode** can create the instruction files, prompt files, agents, agent skills, MCP servers, and plugin manifests themselves — let Copilot do the heavy lifting!
