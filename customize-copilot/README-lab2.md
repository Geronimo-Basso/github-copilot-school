# Lab 02 — Copilot Custom Instructions

Teach **GitHub Copilot** to speak your project's language. Custom instructions, path-specific rules, reusable prompt files, custom agents, and agent skills turn Copilot from a generic assistant into a domain expert that knows your conventions by heart.

## Table of Contents

- [What You'll Learn](#what-youll-learn)
- [Setup](#setup)
- [Phase 1: Custom Instructions](#phase-1-custom-instructions)
  - [Step 1: Setting Up Custom Instructions](#step-1-setting-up-custom-instructions)
  - [Step 2: Path-Specific Custom Instructions](#step-2-path-specific-custom-instructions)
- [Phase 2: Reusable Prompt Files](#phase-2-reusable-prompt-files)
- [Phase 3: Custom Agents](#phase-3-custom-agents)
- [Phase 4: Agent Skills](#phase-4-agent-skills)
- [Congratulations! 🎉](#congratulations-)

## What You'll Learn

Today's goal is to learn how to **customize GitHub Copilot's behavior** so its output consistently matches your project standards — without repeating the same guidance in every prompt.

- **Custom Instructions** — Give Copilot project-wide, personal, and organization-level context
- **Path-Specific Custom Instructions** — Target conventions to specific files or directories
- **Prompt Files** — Build reusable slash commands that automate multi-step workflows
- **Custom Agents** — Define reusable personas with their own instructions, tool restrictions, and handoffs
- **Agent Skills** — Package domain expertise so Copilot writes better code in specialized areas

By the end of this lab you will have set up custom instructions at every scope, fixed non-compliant content in your activities data, built a prompt file to automate adding new activities, defined a pair of custom agents that hand off work to each other, and created an agent skill that layers expert defaults on top of that prompt — all on the FastAPI school activities website from Lab 01. ♟️ ⚽️ 🎻

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

So far you've taught Copilot the project's rules (instructions), automated repeatable workflows (prompt files), and you're about to give it expert defaults (agent skills). But every time you start a chat you still pick the model, allow or restrict tools, and re-explain the persona you want. **Custom agents** bundle all of that — instructions, allowed tools, model preference, and even handoffs to other agents — into a single selectable persona.

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
| **Agent Skills** | What *expertise* should I apply? | "Default `max_participants` to 20" |

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

Next, you'll layer **expert defaults** on top of everything you've built with **Agent Skills**.

---

## Phase 4: Agent Skills

You've customized how Copilot understands your project, automated content creation, and defined reusable personas. But what about the **defaults** Copilot uses when the user is vague? Right now, `/new-activity` asks for every field — that's safe, but for *most* clubs the school's defaults would do.

That's where **Agent Skills** come in. An agent skill is a small file of **expert knowledge** that Copilot reads before acting — like giving it a cheat-sheet from a senior teacher who knows what the school usually wants.

### 📖 Theory: What are Agent Skills?

An agent skill is a `SKILL.md` file that gives Copilot domain expertise. While instructions say "follow these rules" and prompts say "do this task", an agent skill says **"here is how an expert does it — apply this knowledge."**

| Aspect | Details |
| ------ | ------- |
| **File name** | `SKILL.md` |
| **Location** | Inside a folder under `.github/skills/` |
| **Frontmatter** | `name` and `description` — tells Copilot when to use it |
| **Content** | Best practices, defaults, and do's/don'ts |

> 💡 **Tip:** Keep agent skills focused on one area. A small, specific agent skill is more useful than a massive generic one.

> ❕ **Important:** The `description` field in the frontmatter is **the most critical part** of an agent skill. Copilot reads it to decide whether to load the agent skill for a given task. If the description doesn't mention the right keywords (e.g., "activity", "create", "school"), Copilot won't know when to apply it — and your agent skill will be ignored. Make the description specific and include the key terms that match the kind of tasks you want the agent skill to activate for.

See the [VS Code Docs: Agent Skills](https://code.visualstudio.com/docs/copilot/customization/agent-skills) page for more information.

#### Agent Skills Can Reference Other Files

Inside a `SKILL.md` file, you can link to other files in your workspace using regular markdown links — just like prompt files do. When Copilot loads the agent skill, it also reads the linked files for additional context.

This is powerful because it lets you **reuse** the prompt files you already built. For example, an agent skill can say: "When creating activities, follow the workflow in the `new-activity` prompt file, but also apply these extra defaults."

### Activity: Create the activity-creation agent skill 🧑‍🏫

Let's build an agent skill that layers school-specific defaults on top of the `/new-activity` prompt file from Phase 2.

1. Create the agent skill file:

   ```text
   .github/skills/activity-creation/SKILL.md
   ```

2. Fill in the placeholders (`##`, `enter-prompt-file-name`, `enter-path-to-prompt-file`) below with the correct agent skill name, description, and **relative path** to your prompt file:

   ```markdown
   ---
   name: ##
   description: '##. Use when ##'
   ---

   # Activity Creation Agent Skill

   When creating new activities, follow the workflow in
   [enter-prompt-file-name](enter-path-to-prompt-file) but apply
   these school defaults and rules:

   ## Defaults

   - If the user does not specify `max_participants`, default to **20**.
   - If the user does not specify a schedule, default to **`"Fridays, 3:30 PM - 5:00 PM"`**.

   ## Required Information

   - **Activity name**
   - **Description** (at least one full sentence)
   - If either of these is missing, do **NOT** proceed. Ask the user to provide them before creating anything.

   ## Validation

   - The activity name must be in Title Case.
   - The description must be at least 10 words.
   - `participants` must always start as an empty array.
   ```

   > 🚧 **Note:** Notice how the agent skill **links to the prompt file** instead of repeating its workflow. Copilot follows the prompt file's steps (gather info, append to JSON) while also applying the defaults and validation from the agent skill.

3. Save the file.

### Activity: See the agent skill in action 🚀

1. Open **Copilot Chat** in **Agent** mode.

2. Ask Copilot to create a new activity without specifying every field:

   > ![Static Badge](https://img.shields.io/badge/-Prompt-text?style=social&logo=github%20copilot)
   >
   > ```prompt
   > Create a new astronomy club activity where students learn about
   > constellations and use the school telescope.
   ```

3. Observe how Copilot behaves with the agent skill active:

   - It should **not** ask for `max_participants` (the agent skill defaults to 20).
   - It should **not** ask for a schedule (the agent skill defaults to `"Fridays, 3:30 PM - 5:00 PM"`).
   - It should follow the same workflow from the prompt file (append to `activities.json`, init `participants: []`).
   - The new entry should respect every path-specific rule from Step 2.

4. Refresh the browser and verify the new activity appears with the expected defaults.

   **🎯 Goal: The agent skill combines the `/new-activity` prompt workflow with school-specific defaults and validation — showing that agent skills can reference and extend existing files. ✅**

<details>
<summary>Agent Skill not being applied? 🤷</summary>

- Make sure the file is at exactly `.github/skills/activity-creation/SKILL.md`.
- The `description` in the frontmatter must mention keywords like `activity`, `create`, `school` so Copilot knows when to load it.
- Check that the relative path to the prompt file is correct (typically `../../prompts/new-activity.prompt.md`).
- Restart VS Code if the agent skill was just created.

</details>

> 💡 **Agents, prompt files, or skills?** Use custom agents when you need a persistent persona with specific tool restrictions, model preferences, or handoffs between roles. For one-off tasks that don't need tool restrictions, use [prompt files](https://code.visualstudio.com/docs/copilot/customization/prompt-files). For portable, reusable capabilities with scripts and resources, use [agent skills](https://code.visualstudio.com/docs/copilot/customization/agent-skills).

---

## Congratulations! 🎉

You've completed **Lab 02 — Copilot Custom Instructions**! Here's a recap of what you learned:

| Phase | What You Did |
| ----- | ------------ |
| **Phase 1 · Step 1** | Set up custom instructions: organization (theory), repository-wide (hands-on), and personal (theory) |
| **Phase 1 · Step 2** | Built path-specific custom instructions for `app/backend/data/**/*.json` and used them to clean and fix activity entries |
| **Phase 2** | Created a reusable prompt file (`/new-activity`) to automate adding new activities |
| **Phase 3** | Defined two custom agents (`activities-planner` read-only and `activities-implementer`) and chained them with handoffs |
| **Phase 4** | Built an agent skill that references the prompt file and layers school-specific defaults on top |

### Key Takeaways

- **Custom instructions** eliminate repetitive guidance — set them once, benefit every time.
- **Three levels** let you tailor Copilot at the organization, repository, and personal scope.
- **Two types of repository instructions**: repository-wide (`copilot-instructions.md`) and path-specific (`*.instructions.md`).
- **Priority order**: Personal > Repository > Organization — personal preferences always win.
- **Path-specific custom instructions** (`*.instructions.md`) target only the files that need them using glob patterns.
- **Prompt files** (`*.prompt.md`) package multi-step workflows into reusable slash commands.
- **Custom agents** (`*.agent.md`) bundle persona, tool restrictions, model preference, and handoffs into one selectable role — and compose with instructions, prompt files, and agent skills.
- **Agent Skills** (`SKILL.md`) give Copilot deep domain expertise so it writes higher-quality, specialized code.
- **Agent Mode** can create the instruction files, prompt files, agents, and agent skills themselves — let Copilot do the heavy lifting!
