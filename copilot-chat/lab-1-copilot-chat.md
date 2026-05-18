# Lab 01 — Copilot Chat

Explore **GitHub Copilot Chat** — your AI pair programmer that lives right inside the IDE. Ask it to explain code, fix bugs, write tests, and more.

## What You'll Learn

Today's goal will be to learn about GitHub Copilot Chat, the AI-powered UI that GitHub offers, and how to get the most out of the three different modes it provides:

- **Ask Mode** — Onboard to a project, explore code, and get answers
- **Plan Agent** — Design an implementation plan before writing any code
- **Agent Mode** — Let Copilot autonomously fix bugs, add features, and iterate on code
- **Inline suggestion controls** — Accept the full suggestion, just the next word, or cycle through alternatives
- **Next Edit Suggestions** — Let Copilot predict and chain your follow-up edits
- **Model selection** — Choose Auto or a specific language model, and adjust reasoning effort when available
- **Chat power-ups** — Use `#` context references, `/` slash commands, and `@` chat participants

By the end of this lab you will have used all three modes to fix a bug, add new features, and write tests for a FastAPI web application.

## Step 1: Hello Copilot

Welcome to your **"Getting Started with GitHub Copilot"** exercise!

In this exercise, you will be using different GitHub Copilot features to work on a website that allows students of GitHub Copilot High School to sign up for extracurricular activities. 🎻 ⚽️ ♟️

### 📖 Theory: Getting to know GitHub Copilot

GitHub Copilot is an AI coding assistant that helps you write code faster and with less effort, allowing you to focus more energy on problem solving and collaboration.

GitHub Copilot has been proven to increase developer productivity and accelerate the pace of software development. For more information, see [Research: quantifying GitHub Copilot’s impact on developer productivity and happiness in the GitHub blog.](https://github.blog/news-insights/research/research-quantifying-github-copilots-impact-on-developer-productivity-and-happiness/)

As you work in your IDE, you'll most often interact with GitHub Copilot in the following ways:

| Interaction Mode          | 📝 Description                                                                                                                 | 🎯 Best For                                                                                                     |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------- |
| **⚡ Inline suggestions** | AI-powered code suggestions that appear as you type, offering context-aware completions from single lines to entire functions. | Completion of the current line, sometimes a whole new block of code                                             |
| **💭 Inline Chat**        | Interactive chat scoped to your current file or selection. Ask questions about specific code blocks.                           | Code explanations, debugging specific functions, targeted improvements                                          |
| **💬 Ask Mode**           | Optimized for answering questions about your codebase, coding, and general technology concepts.                                | Understanding how code works, brainstorming ideas, asking questions                                             |
| **🤖 Agent Mode**         | Recommended default mode for most coding tasks: autonomous edits, tool use, and follow-through until the task is done.         | Daily coding tasks, from scoped fixes to larger multi-file implementation work                                   |
| **🧭 Plan Agent**         | Optimized for drafting a plan and asking clarifying questions before any code changes are made.                                | When you want a reviewed plan first, then hand off to implementation                                            |

As you work, you'll find GitHub Copilot can help out in several places across the `github.com` website and in your favorite coding environments such as VS Code, Jet Brains, and Xcode!

> [!TIP]
> You can learn more about current and upcoming features in the [GitHub Copilot Features](https://docs.github.com/en/copilot/about-github-copilot/github-copilot-features) documentation.

### Activity: Get a project intro from Copilot Chat

Let's start up our development environment, use copilot to learn a bit about the project, and then give it a test run.

1. Start by cloning this repository into your local machine.

2. If in Visual Studio Code, in the left sidebar, click the extensions tab and verify that the `GitHub Copilot` and `Python` extensions are installed and enabled.

1. At the top of VS Code, locate and click the **Toggle Chat icon** to open a Copilot Chat side panel.

   > 🪧 **Note:** If this is your first time using GitHub Copilot, you will need to accept the usage terms to continue.

1. Make sure you are in **Ask Mode** for our first interaction.

1. Enter the below prompt to ask Copilot to introduce you to the project.

   > ![Static Badge](https://img.shields.io/badge/-Prompt-text?style=social&logo=github%20copilot)
   >
   > ```prompt
   > Please briefly explain the structure of this project.
   > What should I do to run it?
   > ```

   Now add the src folder as a context and repeat the same question. 

   Also, you can use the #codespace, that will allow copilot to see the whole repo content and its codebase.

1. Now that we know a bit more about the project, let's actually try running it!

### Activity: Use Copilot to help remember a terminal command 🙋

Great work! Now that we are familiar with the app and we know it works, let's ask copilot for help starting a branch so we can do some customizing.

1. In VS Code's bottom panel, select the **Terminal** tab and on the right side click the plus `+` sign to create a new terminal window.

   > 🪧 **Note:** This will avoid stopping the existing debug session that is hosting our web application service.

1. Within the new terminal window use the keyboard shortcut `Ctrl + I` (windows) or `Cmd + I` (mac) to bring up **Copilot's Terminal Inline Chat**.

1. Let's ask Copilot to help us remember a command we have forgotten: creating a branch and publishing it.

   > ![Static Badge](https://img.shields.io/badge/-Prompt-text?style=social&logo=github%20copilot)
   >
   > ```prompt
   > Hey copilot, how can I create and publish a new Git branch called "accelerate-with-copilot"?
   > ```

   > 💡 **Tip:** If Copilot doesn't give you quite what you want, you can always continue explaining what you need. Copilot will remember the conversation history for follow-up responses.

1. Press the `Run` button to let Copilot insert the terminal command for us. No need to copy and paste!

1. After a moment, look in the VS Code lower status bar, on the left, to see the active branch. It should now say `accelerate-with-copilot`. If so, you are all done with this step!

## Step 2: Getting work done with Copilot

In the previous step, GitHub Copilot was able to help us onboard to the project. That alone is a huge time saver, but now let's get some work done!

:bug: **THERE IS A BUG ON THE WEBSITE** :bug:

We’ve discovered that something’s off in the signup flow.
Students can currently register for the same activity **more than once**! Let’s see how far Copilot can take us in uncovering the cause and shaping a clean fix.

Before we dive in, a quick primer on how Copilot works. 🧑‍🚀

### 📖 Theory: How Copilot works

In short, you can think of Copilot like a very specialized coworker. To be effective with them, you need to provide them background (context) and clear direction (prompts). Additionally, different people are better at different things because of their unique experiences (models).

- **How do we provide context?:** In our coding environment, Copilot will automatically consider nearby code and open tabs. If you are using chat, you can also explicitly refer to files.

- **What model should we pick?:** For our exercise, it shouldn't matter too much. Experimenting with different models is part of the fun! That's another lesson! 🤖

- **How do I make prompts?:** Being explicit and clear helps Copilot do the best job. But unlike some traditional systems, you can always clarify your direction with followup prompts.

### Activity: Use Copilot to fix our registration bug :bug:

1. Let's ask Copilot to suggest where our bug might be coming from. Open the **Copilot Chat** panel in **Ask mode** and ask the following.

   > ![Static Badge](https://img.shields.io/badge/-Prompt-text?style=social&logo=github%20copilot)
   >
   > ```prompt
   > Students are able to register twice for an activity.
   > Where could this bug be coming from?
   > ```

1. Now that we know the issue is in the `backend/app.py` file and the `signup_for_activity` method, let's follow Copilot's recommendation and go fix it (semi-manually). We'll start with a comment and let Copilot finish the correction.
   1. Open the `backend/app.py` file.

      > 💡 **Tip:** If Copilot mentioned `backend/app.py` in chat, you can click the file directly in the chat view to open it.

   1. Near the bottom of the file, find the `signup_for_activity` function.

   1. Find the comment line that describes adding a student. Above this is where it seems logical to do our registration check.

   1. Enter the below comment and press enter to go to the next line. After a moment, temporary shadow text will appear with a suggestion from Copilot! Nice! :tada:

      Comment:

      ```python
      # Validate student is not already signed up
      ```

   1. Press `Tab` to accept Copilot's suggestion and convert the shadow text to code.

   <details>
   <summary>Example Results</summary><br/>

   Copilot is growing every day and may not always produce the same results. If you are unhappy with the suggestions, here is an example of a valid suggestion result we produced during the making of this exercise. You can use it to continue forward.

   ```python
   @app.post("/activities/{activity_name}/signup")
   def signup_for_activity(activity_name: str, email: str):
      """Sign up a student for an activity"""
      # Validate activity exists
      if activity_name not in activities:
         raise HTTPException(status_code=404, detail="Activity not found")

      # Get the activity
      activity = activities[activity_name]

      # Validate student is not already signed up
      if email in activity["participants"]:
        raise HTTPException(status_code=400, detail="Student is already signed up")

      # Add student
      activity["participants"].append(email)
      return {"message": f"Signed up {email} for {activity_name}"}
   ```

   </details>

### Activity: Take control of inline suggestions ⌨️

When Copilot shows shadow text in the editor, you don't have to take it or leave it as a whole. You can accept just a piece of it, or ask for a different suggestion. This is useful when Copilot is _almost_ right but you only want part of what it proposed.

#### 📖 Theory: Accepting full, partial, or next-word suggestions

While a ghost-text suggestion is visible, you can:

- **Accept the full suggestion** — press `Tab`.
- **Accept just the next word** — press `Cmd + →` (macOS) or `Ctrl + →` (Windows / Linux). Useful when you want to keep typing but borrow the next token from Copilot.
- **Cycle through alternatives** — use `Option + ]` / `Option + [` (macOS) or `Alt + ]` / `Alt + [` (Windows / Linux) to see other suggestions Copilot has for the same spot.
- **Dismiss the suggestion** — press `Esc`.

> [!TIP]
> If you want to accept a suggestion one line at a time, you can bind a keyboard shortcut to the `editor.action.inlineSuggest.acceptNextLine` command from **Keyboard Shortcuts**.

#### Try it

1. Open `backend/app.py` and scroll to the bottom of the file.

1. On a new line inside the `signup_for_activity` function (or just below it), start typing a comment that gives Copilot something to predict, for example:

   ```python
   # Return the list of participants for a given activity
   def 
   ```

1. Wait for the shadow text to appear. **Do not press `Tab` yet.**

1. Press `Cmd + →` (macOS) or `Ctrl + →` (Windows / Linux) once or twice and notice how only the next word is added each time.

1. With a suggestion still visible, press `Option + ]` / `Alt + ]` to cycle to an alternative suggestion, then `Option + [` / `Alt + [` to go back.

1. When you find one you like, press `Tab` to accept the rest. If none of them fit, press `Esc` to dismiss everything.

   **🎯 Goal: Be comfortable accepting only part of a suggestion and switching between alternatives instead of taking whatever Copilot shows first. ✅**

> [!NOTE]
> Feel free to undo (`Cmd/Ctrl + Z`) any throwaway code you added during this activity before moving on.

### Activity: Follow Copilot's next edit suggestions ➡️

Inline suggestions complete code _where your cursor is_. **Next Edit Suggestions (NES)** go a step further: after you make a change, Copilot predicts the _next_ edit you'll probably want to make — even if it's somewhere else in the file — and offers to jump there and apply it for you.

#### 📖 Theory: What are Next Edit Suggestions?

When NES has a suggestion, you'll see:

- A small **arrow indicator in the gutter** (left margin) pointing at the line Copilot wants to edit.
- A **preview** of the proposed change at that location.

The keyboard flow is simple:

- Press `Tab` to **jump** to the suggested edit (if your cursor isn't already there).
- Press `Tab` again to **accept** it.
- Press `Esc` to **dismiss** it.

This makes repetitive edits — like renaming a variable in three places, updating callers after changing a signature, or fixing a series of similar typos — feel almost automatic.

> [!NOTE]
> NES can be toggled with the setting `github.copilot.nextEditSuggestions.enabled`. If you don't see arrow indicators, check that this setting is enabled in your VS Code settings.

#### Try it

1. Open `backend/app.py`.

1. Pick any small but **repeatable** edit. A good one: rename the function parameter `email` to `student_email` in the `signup_for_activity` function signature.

1. Make the change in the signature only. **Don't update the other usages yourself.**

1. Look at the gutter (left margin) for a small arrow indicator on the next line that still references `email`.

1. Press `Tab` to jump to that suggestion, then `Tab` again to accept it. Repeat until all references are updated.

   **🎯 Goal: Use `Tab` to chain through Copilot's predicted follow-up edits instead of finding and fixing each one manually. ✅**

> [!NOTE]
> When you're done experimenting, undo (`Cmd/Ctrl + Z`) the rename so the rest of the lab continues to work with the original parameter name.

### Activity: Let Copilot generate sample data 📋

In new project developments, it's often helpful to have some realistic looking fake data for testing. Copilot is excellent at this task, so let's add some more sample activities and introduce another way to interact with Copilot using **Inline Chat**

**Inline Chat** and the **Copilot Chat** panel are similar, but differ in scope: Copilot Chat handles broader, multi-file or exploratory questions; Inline Chat is faster when you want targeted help on the exact line or block in front of you.

1. Near the top of the `backend/app.py` file (about line 23), find the `activities` variable, where our example extracurricular activities are configured.

1. Highlight the entire `activities` dictionary by clicking and dragging your mouse from the top to the bottom of the dictionary. This will help provide context to Copilot for our next prompt.

1. Bring up Copilot inline chat by using the keyboard command `Ctrl + I` (windows) or `Cmd + I` (mac).

   > 💡 **Tip:** Another way to bring up Copilot inline chat is: `right click` on any of the selected lines -> `Open Inline Chat`.

1. Enter the following prompt text and press enter or the **Send** button on the right.

   > ![Static Badge](https://img.shields.io/badge/-Prompt-text?style=social&logo=github%20copilot)
   >
   > ```prompt
   > Add 2 more sports related activities, 2 more artistic
   > activities, and 2 more intellectual activities.
   > ```

1. After a moment, Copilot will directly start making changes to the code. The changes will be stylized differently to make any additions and removals easy to identify. Take a moment to inspect and verify the changes, and then press the **Keep** button.

   <details>
   <summary>Example Results</summary><br/>

   Copilot is growing every day and may not always produce the same results. If you are unhappy with the suggestions, here is an example result we produced during the making of this exercise. You can use it to continue forward, if having trouble.

   ```python
   # In-memory activity database
   activities = {
      "Chess Club": {
         "description": "Learn strategies and compete in chess tournaments",
         "schedule": "Fridays, 3:30 PM - 5:00 PM",
         "max_participants": 12,
         "participants": ["michael@githubcopilot.edu", "daniel@githubcopilot.edu"]
      },
      "Programming Class": {
         "description": "Learn programming fundamentals and build software projects",
         "schedule": "Tuesdays and Thursdays, 3:30 PM - 4:30 PM",
         "max_participants": 20,
         "participants": ["emma@githubcopilot.edu", "sophia@githubcopilot.edu"]
      },
      "Gym Class": {
         "description": "Physical education and sports activities",
         "schedule": "Mondays, Wednesdays, Fridays, 2:00 PM - 3:00 PM",
         "max_participants": 30,
         "participants": ["john@githubcopilot.edu", "olivia@githubcopilot.edu"]
      },
      "Basketball Team": {
         "description": "Competitive basketball training and games",
         "schedule": "Tuesdays and Thursdays, 4:00 PM - 6:00 PM",
         "max_participants": 15,
         "participants": []
      },
      "Swimming Club": {
         "description": "Swimming training and water sports",
         "schedule": "Mondays and Wednesdays, 3:30 PM - 5:00 PM",
         "max_participants": 20,
         "participants": []
      },
      "Art Studio": {
         "description": "Express creativity through painting and drawing",
         "schedule": "Wednesdays, 3:30 PM - 5:00 PM",
         "max_participants": 15,
         "participants": []
      },
      "Drama Club": {
         "description": "Theater arts and performance training",
         "schedule": "Tuesdays, 4:00 PM - 6:00 PM",
         "max_participants": 25,
         "participants": []
      },
      "Debate Team": {
         "description": "Learn public speaking and argumentation skills",
         "schedule": "Thursdays, 3:30 PM - 5:00 PM",
         "max_participants": 16,
         "participants": []
      },
      "Science Club": {
         "description": "Hands-on experiments and scientific exploration",
         "schedule": "Fridays, 3:30 PM - 5:00 PM",
         "max_participants": 20,
         "participants": []
      }
   }
   ```

   </details>

1. You can now go to your website and verify that the new activities are visible.

### Activity: Use Copilot to describe our work 💬 (Optional)

Nice work fixing that bug and expanding the example activities! Now let's get our work committed and pushed to GitHub, again with the help of Copilot!

1. In the left sidebar, select the `Source Control` tab.

   > 💡 **Tip:** Opening a file from the source control area will show the differences to the original rather than simply opening it.

1. Find the `app.py` file and press the `+` sign to collect your changes together in the staging area.

1. Above the list of staged changes, find the **Message** text box, but **don't enter anything** for now.
   - Typically, you would write a short description of the changes here, but now we have Copilot to help out!

1. To the right of the **Message** text box, find and click the **Generate Commit Message** button (sparkles icon).

1. Press the **Commit** button and **Sync Changes** button to push your changes to GitHub.

## Step 3: Plan your implementation with the Planning Agent 🧭

So far we've explored the project and made small, manual fixes. Before we hand the keyboard over to a more autonomous Copilot, let's slow down for one round and work like architects: define a strong approach first, then hand it off for implementation. This gives us better clarity, fewer surprises, and cleaner results. 🧪

### 📖 Theory: What is Copilot Plan Agent?

Copilot [Plan Agent](https://code.visualstudio.com/docs/copilot/agents/planning) helps you design a solution before any code is changed.

Instead of jumping straight into edits, it researches your request, asks clarifying questions, and drafts an implementation plan you can refine.

#### Plan Agent (at a glance)

| Aspect | 🧭 Plan Agent |
| --- | --- |
| Purpose | Creates a structured implementation plan before coding starts. |
| Context gathering | Uses read-only research to understand requirements and constraints. |
| Collaboration style | Asks clarifying questions, then updates the plan using your answers. |
| Iteration | Supports multiple refinement passes before implementation. |
| Safety | Does not edit files until you approve the plan and hand off to **Agent Mode**. |
| Handoff | **Start implementation** button hands off the approved plan to **Agent Mode** for coding. |

> [!TIP]
> You can start from a high-level request and then add constraints and details in follow-up prompts.

### ⌨️ Activity: Plan and implement backend tests

Your backend still has zero test coverage. Use **Plan Agent** to create a plan, answer questions, and then launch implementation.

1. Open the **Copilot Chat** panel and switch to **Plan Agent**.

1. Let's start with a broad prompt and Copilot will help us fill in the details:

   > ![Static Badge](https://img.shields.io/badge/-Prompt-text?style=social&logo=github%20copilot)
   >
   > ```prompt
   > I want to add backend FastAPI tests in a separate tests directory.
   > ```

1. Wait for Copilot to generate its first plan. If it asks you any questions, answer them to the best of your ability. 

   > 🪧 **Note:** Don't worry about getting it perfect, you can always refine the plan later.

1. You can refine the plan and provide additional details in follow up prompts

   Here are some examples:

   > ![Static Badge](https://img.shields.io/badge/-Prompt-text?style=social&logo=github%20copilot)
   >
   > ```prompt
   > Let's use the AAA (Arrange-Act-Assert) testing pattern to structure our tests
   > ```

   > ![Static Badge](https://img.shields.io/badge/-Prompt-text?style=social&logo=github%20copilot)
   >
   > ```prompt
   > Make sure we use `pytest` and add it to `requirements.txt` file
   > ```


1. Review the proposed plan and when you are happy with it, click **Start implementation** to hand off to **Agent Mode**.

   Notice that clicking the button switched from **Plan** to **Agent Mode**. This is your first peek at Agent Mode — we'll explore it in depth in the next step.

1. Watch Copilot implement the plan you just created. It may ask for permissions to run certain tools (e.g., run commands or create virtual environments). Approve these permissions so it can continue working.

1. Review the changes and make sure tests run successfully. If needed, continue guiding until implementation is complete.

   **🎯 Goal: Get all tests passing (green) before you move on. ✅**

   > 🪧 **Note:** Agent Mode may complete this in one pass, or it may need follow-up prompts from you.

## Step 4: Engage Hyperdrive - Copilot Agent Mode 🚀

You just saw a glimpse of Agent Mode at the tail end of the plan handoff. Now let's drive it directly to add new features across the codebase. 🚀

### 📖 Theory: What is Copilot Agent Mode?

Copilot [agent mode](https://code.visualstudio.com/docs/copilot/chat/chat-agent-mode) is the next evolution in AI-assisted coding. Acting as an autonomous peer programmer, it performs multi-step coding tasks at your command.

Copilot Agent Mode responds to compile and lint errors, monitors terminal and test output, and auto-corrects in a loop until the task is completed.

#### Agent Mode (at a glance)

| Aspect | 👩‍🚀 Agent Mode |
| --- | --- |
| Autonomy and planning | Breaks down high-level requests into multi-step work and iterates until the task is complete. |
| Context gathering | Uses your current context and can discover additional relevant files when needed. |
| Tool use | Selects and invokes tools automatically; you can also direct tools with mentions like `#codebase`. |
| Approval and safety gates | Sensitive actions can require approval before execution, helping you stay in control. |

#### 🧰 Agent Mode Tools

Agent mode uses tools to accomplish specialized tasks while processing a user request. Examples of such tasks are:

- Finding relevant files to complete your prompt
- Fetching contents of a webpage
- Running tests or terminal commands

#### 🔐 Approvals and permission levels

Because Agent Mode can run tools on your machine (edit files, run terminal commands, call MCP servers), VS Code asks for **approval** before sensitive actions. You stay in control of what runs.

**Per-tool approvals.** When Agent Mode wants to invoke a sensitive tool (for example, run a terminal command), a confirmation appears in the chat. Next to **Continue** there is a dropdown that lets you choose the scope of your approval:

| Scope | What it does |
| --- | --- |
| **Allow in this Session** | Auto-approves this tool for the rest of the current chat session. |
| **Allow in this Workspace** | Auto-approves this tool whenever you work in this workspace. |
| **Always Allow** | Auto-approves this tool everywhere, across all workspaces. |

You can also open the **Tools** picker in the chat input to enable or disable specific tools, and run **Chat: Reset Tool Confirmations** from the Command Palette to clear previous "Always allow" decisions.

**Session permission level.** On top of per-tool approvals, each chat session has a **permission level** (set from the permissions picker in the Chat view). It controls how much autonomy the agent has overall:

| Level | What it does |
| --- | --- |
| **Default Approvals** | Uses your approval settings. Only read-only and safe tools run without asking; sensitive ones prompt you. |
| **Bypass Approvals** | Auto-approves all tool calls. The agent may still ask clarifying questions. |
| **Autopilot (Preview)** | Auto-approves tool calls **and** auto-responds to questions, so the agent keeps working until the task is done. |

> [!TIP]
> For this lab, stay on **Default Approvals**. It lets you see what Copilot is about to do — especially terminal commands — and approve each one consciously. Save Bypass and Autopilot for tasks you trust in a safe environment (a sandbox, a throwaway branch, or a container).

Now, let's give **Agent Mode** a try on its own! 👩‍🚀

### Activity: Use Copilot to add a new feature! :rocket:

Our website lists activities, but it's keeping the guest list secret 🤫 

Let's use Copilot to change the website to display signed up students under each activity!

1. At the bottom of Copilot Chat window, use the dropdown to switch to **Agent** mode.

1. Open the files related to our webpage then drag each editor window (or file) to the chat panel, informing Copilot to use them as context.

   - `frontend/app.js`
   - `frontend/index.html`
   - `frontend/styles.css`

   > 🪧 **Note:** Adding files as context is optional. If you skip this, Copilot Agent Mode can still use tools like `#codebase` to search for relevant files from your prompt. Adding specific files helps point Copilot in the right direction, which is especially useful in larger codebases.

   > 💡 **Tip:** You can also use the **Add Context...** button to provide other sources of context items, like a GitHub issue or the results of a terminal window.

1. Ask Copilot to update our project to display the current participants of activities. Wait a moment for the edit suggestions to arrive and be applied.

   > ![Static Badge](https://img.shields.io/badge/-Prompt-text?style=social&logo=github%20copilot)
   >
   > ```prompt
   > Hey Copilot, can you please edit the activity cards to add a participants section.
   > It will show what participants that are already signed up for that activity as a bulleted list.
   > Remember to make it pretty!
   > ```

   After Copilot finishes work, you are in control of what changes get to stay. 

   Using the **Keep** buttons shown below, you can accept/discard all changes or review and decide change by change. This can be done either from the chat panel view or while inspecting each edited file.

1. Before we simply accept the changes, please check our website again and verify everything is updated as expected. 
   
   Here is an example of an updated activity card. You may need to restart the app or refresh the page.

   > 🪧 **Note:** Your activity card may look different. Copilot won't always produce the same results.

1. Now that we have confirmed our changes are good, use the panel to cycle through each suggested edit and press **Keep** to apply the change.

   > 💡 **Tip:** You can accept the changes directly, modify them, or provide additional instruction to refine them using the chat interface.

### Activity: Use Agent mode to add functional "unregister" buttons

Let's experiment with some more open-ended requests that will add more functionality to our web application.

If you don't get the desired results, you can try other models or provide follow-up feedback to refine the results.

1. Make sure your Copilot is still in **Agent** mode.

1. Click on the **Tools** icon and explore all Tools currently available to Copilot Agent Mode.

1. Time for our test! Let's ask Copilot to add functionality for removing participants.

   > ![Static Badge](https://img.shields.io/badge/-Prompt-text?style=social&logo=github%20copilot)
   >
   > ```prompt
   > #codebase Please add a delete icon next to each participant and hide the bullet points.
   > When clicked, it will unregister that participant from the activity.
   > ```

   The `#codebase` tool is used by Copilot to find relevant files, code chunks that are relevant to the task at hand.

   > 🪧 **Note:** In this lab we explicitly include the `#codebase` tool to get the most repeatable results.
   > Feel free to try the prompt **without** `#codebase` and observe whether Agent Mode decides to gather broader project context on its own.

1. When Copilot is finished, inspect the code changes and the results on the website. If you like the results, press the **Keep** button. If not, try providing Copilot some feedback to refine the results.

   > 🪧 **Note:** If you don't see updates on the website, you may need to restart the code.

1. Ask Copilot to fix a registration bug.

   > 💡 **Tip:** We recommend testing the registration flow yourself so you can clearly see the before/after changes behavior.

   > ![Static Badge](https://img.shields.io/badge/-Prompt-text?style=social&logo=github%20copilot)
   >
   > ```prompt
   > I've noticed there seems to be a bug.
   > When a participant is registered, the page must be refreshed to see the change on the activity.
   > ```

1. When Copilot is finished, inspect the results and validate the registration flow on the website.

   If you like the results, press the **Keep** button. If not, try providing Copilot some feedback.

## Step 4.5: Tune Copilot for the task 🛠️

Before you fly solo, let's add one more practical skill: choosing how much thinking power Copilot should use.

Different tasks need different tradeoffs. A quick syntax question should feel fast. A multi-file refactor, bug investigation, or testing strategy may benefit from a stronger reasoning model or higher thinking effort.

### 📖 Theory: Models, Auto, and reasoning effort

In Copilot Chat, the selected language model affects the kind of response you get. Some models are optimized for fast, lightweight help; others are better for deeper reasoning, debugging, architecture, or agentic coding tasks.

You can usually start with **Auto**. In VS Code, Auto model selection lets Copilot choose an appropriate available model for the current chat request. This is a good default when you do not have a strong reason to pick a specific model.

Choose a specific model when you want more control. For example, you might pick a faster model for small edits and quick explanations, or a stronger reasoning model for multi-step debugging, refactoring, planning, or codebase-wide analysis.

Some reasoning models also let you configure **thinking effort**. Higher effort can help with complex tasks, but may take longer. Lower effort is useful when the task is straightforward and you want a faster response.

> [!NOTE]
> The exact model list depends on your Copilot plan, organization policies, VS Code version, and which models are currently available. If you do not see the same options as your neighbor, that is normal.

### Activity: Compare Auto and a specific model

1. Open the **Copilot Chat** panel.

1. In the chat input area, find the **model picker**. It is usually shown near the chat input, alongside the current model name.

1. Select **Auto** if it is available.

1. Open the model picker and select a reasoning model, if one is available.

1. Look for a submenu or arrow next to the model name that lets you configure **Thinking Effort**.


## Step 4.6: Power-ups — context, slash commands, and chat participants 🧩

Beyond plain prompts, Copilot Chat understands a few special "shortcuts" you can drop into the input box. They help you point at the right context, trigger a common task, or hand the conversation off to a specialist.

There are three families to know:

- `#` — **context references**: attach files, folders, problems, or terminal output to the prompt.
- `/` — **slash commands**: trigger a built-in action like explaining code or generating tests.
- `@` — **chat participants**: route the question to a specialist (e.g., GitHub, Azure, your workspace).

> [!NOTE]
> Which references, commands, and participants are available depends on your VS Code version, installed extensions (e.g., GitHub Pull Requests, Azure), and Copilot plan. Type `#`, `/`, or `@` in the chat input to see what is offered to you.

### 🔗 Context references (`#`)

Use these to tell Copilot _what_ to look at. 5 useful ones:

| Reference | What it does |
| --- | --- |
| `#codebase` | Lets Copilot search across your whole workspace for relevant files. |
| `#file:<name>` | Attaches a specific file as context (e.g., `#file:app.py`). |
| `#selection` | Uses the code you currently have selected in the editor. |
| `#terminalLastCommand` | Sends the most recent terminal command and its output — great for debugging failures. |
| `#problems` | Includes the current errors and warnings from the Problems panel. |

### ⚡ Slash commands (`/`)

Use these to trigger a common task without writing a full prompt. 5 useful ones:

| Command | What it does |
| --- | --- |
| `/explain` | Explains the selected code or file. |
| `/fix` | Proposes a fix for the selected code or current error. |
| `/tests` | Generates unit tests for the selected code. |
| `/doc` | Adds documentation comments to the selected code. |
| `/new` | Scaffolds a new project or file based on your description. |

### 🧑‍🚀 Chat participants (`@`)

Use these to hand the prompt to a specialist that knows about a specific tool or domain. 5 useful ones:

| Participant | What it's good for |
| --- | --- |
| `@workspace` | Questions about your current project's code and structure. |
| `@vscode` | Questions about VS Code itself — settings, commands, keybindings. |
| `@terminal` | Help crafting or understanding terminal commands. |
| `@github` | Questions about a GitHub repo, issues, or pull requests (requires the GitHub Pull Requests extension). |
| `@azure` | Help with Azure services, deployments, and configuration (requires the Azure extension). |

### Try one

Pick any one of the above and run a quick prompt in **Ask Mode**, for example:

> ![Static Badge](https://img.shields.io/badge/-Prompt-text?style=social&logo=github%20copilot)
>
> ```prompt
> #terminalLastCommand why did this fail and how do I fix it?
> ```

**🎯 Goal: Know that `#`, `/`, and `@` exist, and feel comfortable typing them to discover what's available in your setup. ✅**


## Step 5: Fly Solo — Put It All Together 🧑‍✈️

In the previous steps, we walked through each Copilot mode with guided prompts. Now it's time to take the training wheels off! In this step you will implement new features and improve the codebase using GitHub Copilot as your companion — but **we will not give you the exact prompts**. Think about what you've learned so far and craft your own prompts to get the job done.

> 💡 **Tip:** Remember that the key to great Copilot results is clear context and explicit direction. Don't hesitate to iterate — if the first result isn't perfect, refine your prompt or provide follow-up instructions.

### 📖 Theory: Prompt Crafting — Thinking Like a Copilot Whisperer

Before you dive in, here are some principles to keep in mind when writing your own prompts:

| Principle | 📝 Description | 🎯 Example |
| --- | --- | --- |
| **Be specific** | Tell Copilot exactly what you want, where, and how. | _"Read activities from `data/activities.json` instead of the hardcoded dictionary in `app.py`"_ |
| **Provide context** | Attach relevant files or use `#codebase` so Copilot knows the full picture. | Drag `app.py` and `activities.json` into the chat panel. |
| **Break it down** | Large tasks work better as a series of smaller, focused requests. | First move the data, then add new entries, then verify. |
| **Iterate and refine** | Copilot remembers the conversation — follow up if the result needs adjustments. | _"That looks good, but also add error handling for a missing JSON file."_ |
| **Choose the right mode** | Use **Ask** to explore, **Agent** to implement, and **Plan** to architect. | Use Ask Mode to learn about Swagger, then Agent Mode to add annotations. |

---

### Activity: Replace hardcoded data with a JSON file 📂 (Step 5.1)

Currently, all the activity data lives as a hardcoded Python dictionary inside `backend/app.py`. In a real application this data would come from a database, a configuration file, or an external API. For our project, we already have a JSON file waiting at `backend/data/activities.json` — we just need to wire it up!

#### Part A: Move the data source to JSON

1. Start by exploring the current state of things. Open `backend/app.py` and `backend/data/activities.json` side by side so you can see both files.

   > 💡 **Tip:** You can use **Ask Mode** first to understand how the hardcoded data is currently structured and what changes would be needed to load it from a file instead.

1. Open the **Copilot Chat** panel and switch to **Agent** mode.

1. Add both `backend/app.py` and `backend/data/activities.json` as context by dragging them into the chat panel.

1. Now, craft a prompt that asks Copilot to:
   - Load the activities data from the `data/activities.json` file instead of the hardcoded dictionary.
   - Make sure the JSON file is read at application startup.
   - Remove or replace the old hardcoded `activities` dictionary.

   > 🪧 **Note:** We are intentionally not providing the exact prompt here. Think about what information Copilot needs to accomplish this task. Remember the principles from the theory section above!

1. Review the changes Copilot proposes. Pay attention to:
   - Is the file path correct and relative to where the app runs?
   - Does it handle the case where the JSON file might not exist?
   - Is the JSON structure compatible with how `activities` was used before?

1. If everything looks good, press **Keep**. If not, provide follow-up instructions to refine the result.

1. **Test it!** Restart your backend server and verify the website still loads all activities correctly.

   **🎯 Goal: The app should work exactly as before, but now reading data from `activities.json`. ✅**

   > 🪧 **Note:** If you run into import errors or file-not-found issues, ask Copilot to help you debug! This is a great opportunity to practice providing error context in your prompts.

#### Part B: Expand the activities catalog

Now that our data lives in a JSON file, it's much easier to manage. Let's make our school's extracurricular catalog look more impressive by adding more activities.

1. Open `backend/data/activities.json` in the editor.

1. Think of a prompt that asks Copilot to add more activities to the JSON file. Consider:
   - How many new activities do you want?
   - What categories should they cover? (sports, arts, academics, technology, etc.)
   - Should they have realistic schedules, descriptions, and participant limits?

1. Use either **Inline Chat** (`Cmd + I` / `Ctrl + I`) with the file selected, or **Agent Mode** in the chat panel — whichever you prefer.

1. After Copilot generates the new activities, review them for consistency:
   - Do they follow the same JSON structure as the existing entries?
   - Are the schedules realistic and non-overlapping?
   - Do the `max_participants` values make sense for each activity type?

1. Restart the backend and refresh the frontend to verify the new activities appear on the website.

   **🎯 Goal: Your activities page should now show a rich catalog of diverse extracurricular options. ✅**

   > 💡 **Tip:** If the frontend layout looks cramped or broken with more activities, you can ask Copilot to adjust the CSS grid or card layout to better accommodate the larger list.

---

### Activity: Add a withdraw feature 🚪 (Step 5.2)

So far, our app only lets students sign up for activities. But what happens when a student needs to drop an activity due to a scheduling conflict or a last-minute emergency? It would be great to let students withdraw so that another student can take their place.

This is a **full-stack feature** — you'll need changes in both the backend and the frontend. Here's what needs to happen:

#### 📖 Understanding the requirements

Before writing any code, let's break down what this feature needs:

| Component | What's Needed |
| --- | --- |
| **Backend** | A new API endpoint that accepts an activity name and a student email, then removes that student from the activity's participant list. |
| **Frontend (JS)** | A function that calls the new backend endpoint when a student clicks a "withdraw" or "unregister" button. |
| **Frontend (HTML/CSS)** | A UI element (button or icon) next to each participant or on the activity card that triggers the withdraw action. |

> 💡 **Tip:** This is a great candidate for **Plan Agent**! You can first ask Copilot to create an implementation plan, review it, and then hand off to Agent Mode for the actual coding.

#### Part A: Plan the implementation

1. Open the **Copilot Chat** panel and switch to **Plan Agent**.

1. Craft a prompt describing the withdraw feature. Be sure to mention:
   - The need for a new backend endpoint (think about what HTTP method makes sense — `DELETE`? `POST`?).
   - The frontend needs a way to trigger the withdrawal.
   - The UI should update immediately without requiring a page refresh.

1. Wait for Copilot to generate a plan. Review it carefully:
   - Does the plan cover both backend and frontend changes?
   - Does it include proper error handling (e.g., what if the student isn't registered)?
   - Does it mention updating the UI after a successful withdrawal?

1. If you have feedback, refine the plan with follow-up prompts. For example:
   - _"Use a DELETE endpoint for the withdrawal"_
   - _"Show a confirmation before withdrawing"_
   - _"Make sure the participant list updates in real time"_

#### Part B: Implement the feature

1. Once you're happy with the plan, click **Start implementation** to hand off to **Agent Mode**.

1. Watch Copilot make changes across multiple files. It should modify at least:
   - `backend/app.py` — new endpoint
   - `frontend/app.js` — new JavaScript function
   - `frontend/index.html` and/or `frontend/styles.css` — UI updates

1. When Copilot finishes, review all the proposed changes file by file.

   > 🪧 **Note:** Pay special attention to the backend endpoint. Make sure it validates that:
   > - The activity exists
   > - The student is actually registered before trying to remove them
   > - Appropriate HTTP status codes are returned for each error case

1. **Test the full flow:**
   - Sign up a student for an activity
   - Verify they appear in the participant list
   - Withdraw that student using the new feature
   - Verify they are removed from the list
   - Try withdrawing a student who isn't registered and check the error handling

   **🎯 Goal: Students can sign up for and withdraw from activities, with the UI updating in real time. ✅**

   <details>
   <summary>Stuck? Here are some hints</summary><br/>

   - For the backend, a `DELETE /activities/{activity_name}/withdraw?email=...` endpoint is a clean approach.
   - In the frontend, you likely need an event listener on each participant's remove button that calls `fetch()` with the `DELETE` method.
   - Don't forget to re-render the activity card after a successful withdrawal — reuse or refactor the existing rendering logic.

   </details>

---

### Activity: Document your API with Swagger 📜 (Step 5.3)

We've done a lot of work building out our API — we have endpoints for listing activities, signing up, and now withdrawing. But how would another developer know how to use our API? That's where **API documentation** comes in.

#### 📖 Theory: What is Swagger / OpenAPI?

FastAPI comes with built-in support for the [OpenAPI specification](https://swagger.io/specification/) (formerly known as Swagger). This means your API automatically gets interactive documentation — no extra setup required!

| Feature | 📝 Description |
| --- | --- |
| **Swagger UI** | An interactive web page where you can see all endpoints and test them directly from the browser. Available at `/docs`. |
| **ReDoc** | An alternative documentation view with a clean, readable layout. Available at `/redoc`. |
| **OpenAPI Schema** | A machine-readable JSON specification of your API. Available at `/openapi.json`. |

> 🪧 **Note:** By default, FastAPI generates documentation from your Python function signatures and docstrings. The more detail you add to your code, the better the documentation becomes!

#### Part A: Explore the current documentation

1. Make sure your backend server is running.

1. Open your browser and navigate to `http://localhost:8000/docs` (adjust the port if needed).

1. Take a moment to explore the current Swagger UI:
   - What endpoints are listed?
   - How much detail does each endpoint show?
   - Are request/response models clearly described?

1. You can also use **Ask Mode** in Copilot to learn more about Swagger and what makes good API documentation:

   > 🪧 **Note:** This is a great example of using **Ask Mode** for learning rather than coding. Not everything needs Agent Mode!

   Think of questions like:
   - _"What is Swagger and how does FastAPI use it?"_
   - _"What are best practices for documenting FastAPI endpoints?"_
   - _"How do I add request/response models, descriptions, and examples to my endpoints?"_

#### Part B: Enhance the endpoint documentation

Now let's improve the API documentation by adding richer metadata to our endpoints.

1. Switch to **Agent Mode** in the Copilot Chat panel.

1. Add `backend/app.py` as context.

1. Craft a prompt asking Copilot to improve the Swagger documentation. Think about adding:
   - Descriptive **summary** and **description** fields for each endpoint
   - Proper **response models** using Pydantic `BaseModel` classes
   - **Example values** for request parameters
   - **HTTP status code descriptions** (e.g., 200, 400, 404)
   - **Tags** to group related endpoints together (e.g., "Activities", "Registration")

   > 💡 **Tip:** You don't have to do everything at once. Start with one improvement (like adding response models), verify it works in the Swagger UI, and then ask for more enhancements.

1. After Copilot makes changes, restart the backend and revisit `http://localhost:8000/docs`.

1. Compare the documentation before and after. You should see:
   - Richer descriptions for each endpoint
   - Clearly defined request and response schemas
   - Example values that make it easy to test endpoints directly from the Swagger UI

   **🎯 Goal: Your Swagger documentation should clearly describe every endpoint, its parameters, possible responses, and include helpful examples. ✅**

   <details>
   <summary>Example improvements to look for</summary><br/>

   Here are some things that make Swagger documentation shine:

   - **Tags:** Endpoints grouped by category (e.g., `Activities`, `Registration`)
   - **Response models:** Instead of just returning a raw dict, define Pydantic models like `ActivityResponse`, `SignupResponse`, etc.
   - **Status codes:** Each endpoint documents what happens on success (200/201), bad request (400), and not found (404).
   - **Descriptions:** Each endpoint has a clear one-liner summary and a longer description explaining its behavior.
   - **Examples:** Request and response examples so developers can quickly understand the expected format.

   </details>

#### Part C: Validate and test from Swagger UI

1. With your improved documentation in place, use the **Swagger UI** at `/docs` to test each endpoint interactively:
   - Try listing all activities
   - Sign up a student for an activity
   - Withdraw a student from an activity
   - Attempt invalid operations (e.g., signing up twice, withdrawing someone who isn't registered)

1. Verify that the error responses match the documentation — the status codes and error messages should be consistent.

   > 💡 **Tip:** If you find any inconsistencies between the actual API behavior and the documentation, ask Copilot to fix them. This is a common real-world task — keeping docs and code in sync!

## Step 6: Finally commit time

Ask github copilot chat to do a recap of everything you have done so far, you can commit this if you would like too or just keep it as a personal summary. 

## Congratulations! 🎉

You've completed Lab 02! Here's a recap of what you learned:

- **Ask Mode** — Used Copilot Chat to onboard to a new project, understand its structure, and recall terminal commands
- **Inline Suggestions & Inline Chat** — Fixed a duplicate-registration bug with code completions and generated sample data using inline chat
- **Plan Agent** — Designed a testing strategy collaboratively before handing off implementation to Agent Mode
- **Agent Mode** — Let Copilot autonomously add a participants display, unregister buttons, and fix a UI refresh bug across multiple files
- **Inline suggestion controls** — Accepted full, partial, and next-word suggestions, and cycled through alternatives
- **Next Edit Suggestions** — Used `Tab` to chain through Copilot's predicted follow-up edits across the file
- **Model selection** — Compared Auto with a specific model and adjusted thinking effort when available
- **Chat power-ups** — Explored `#` context references, `/` slash commands, and `@` chat participants

You're now equipped to use all three Copilot Chat modes in your daily workflow. Head over to the other labs in this repository to keep exploring!

