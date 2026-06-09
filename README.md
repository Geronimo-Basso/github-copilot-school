# GitHub Copilot SDK Lab: Build, Ship, and Iterate an Assistant

Last revised: 2026-06-08T11:25:26.100+02:00

This guide turns Phase 1 of the Mergington lab into an end-to-end workflow:

1. Build a GitHub Copilot SDK assistant for the Mergington FastAPI app.
2. Use Copilot CLI interactively to summarize, review, and prepare the change.
3. Publish the work to GitHub with `git` and `gh`.
4. Create a PR and issue.
5. Use GitHub Copilot Coding Agent through issue assignment or PR comments.

Replace these placeholders everywhere:

- `OWNER/REPO` — your GitHub repository, for example `octocat/mergington-sdk-lab`.
- `feature/sdk-assistant` — the feature branch used in this guide.
- `ISSUE_NUMBER` — the GitHub issue number you create.
- `PR_NUMBER` — the GitHub pull request number you create.

> Human review gate: Copilot can help write code, descriptions, and follow-up commits. A human still reviews every diff, checks behavior, and performs the merge.

---

## 1. Prerequisites

You need:

- GitHub account with Copilot enabled.
- GitHub CLI (`gh`) installed.
- Copilot CLI available and authenticated for interactive prompts.
- Python 3.9+.
- Permission to create or attach a GitHub repository.
- GitHub Copilot Coding Agent enabled for the repository or organization if you want to run Phase D.

No Node.js setup is required. The Mergington frontend is plain HTML, JavaScript, and CSS.

---

## 2. Prepare the GitHub repository

The lab uses the `Geronimo-Basso/github-copilot-101` repository as the starting point. You will fork it into your own GitHub account so that you have full write access and can push branches, open PRs, and use the Coding Agent freely.

### 2.1 Authenticate

```powershell
gh auth login --web
gh auth status
```

### 2.2 Fork the repository

Create a personal fork under your own GitHub account:

```powershell
gh repo fork Geronimo-Basso/github-copilot-101 --clone --remote
```

This command forks the repository, clones it locally, and automatically sets up two remotes: `origin` pointing at your fork and `upstream` pointing at the original. You can call your fork whatever you like — rename it on GitHub afterwards if you prefer a different name.

Navigate into the cloned folder:

```powershell
cd github-copilot-101
```

Validate that GitHub can see your fork:

```powershell
gh repo view
```

> From this point forward, `OWNER/REPO` means **your fork** (e.g. `octocat/github-copilot-101`). Substitute your actual username wherever you see `OWNER/REPO`.

### 2.3 Create the working branch

All your changes will live on a feature branch so that the PR and Coding Agent workflows in later phases work correctly:

```powershell
git checkout -b feature/sdk-assistant
```

---

## 3. Phase A — Build with the Copilot SDK

### 3.1 Install the SDK

From the Mergington app folder (relative to the repository root):

```powershell
cd github-copilot-101\lab-6-github-sdk-spec\mergington-app
```

Edit `requirements.txt` and add:

```text
github-copilot-sdk
```

Then install:

```powershell
pip install -r requirements.txt
python -c "from copilot import CopilotClient; print('Copilot SDK import OK')"
```

### 3.2 Playground file

Create the playground at:

```text
scripts\sdk_playground.py
```

### 3.3 First message

Use this baseline shape: start the client, create one session, send one message, and stop the client.

Replace `scripts/sdk_playground.py` with:

```python
import asyncio

from copilot import CopilotClient
from copilot.session import PermissionHandler


async def main() -> None:
    client = CopilotClient()
    await client.start()
    try:
        session = await client.create_session(
            on_permission_request=PermissionHandler.approve_all,  # ⚠️ Lab only; inspect permissions in production.
            model="gpt-4.1",
        )
        response = await session.send_and_wait("Tell me a three-sentence story about a curious cat.")
        print(response.data.content)
    finally:
        await client.stop()


if __name__ == "__main__":
    asyncio.run(main())
```

Run:

```powershell
python scripts\sdk_playground.py
```

### 3.4 Streaming events

Streaming is opt-in with `streaming=True`. Subscribe with `session.on(handle_event)`, then call `session.send_and_wait(...)`.

```python
import asyncio

from copilot import CopilotClient
from copilot.session import PermissionHandler
from copilot.generated.session_events import SessionEventType


async def main() -> None:
    client = CopilotClient()
    await client.start()
    try:
        session = await client.create_session(
            on_permission_request=PermissionHandler.approve_all,  # ⚠️ Lab only; inspect permissions in production.
            model="gpt-4.1",
            streaming=True,
        )

        done = asyncio.Event()

        def handle_event(event) -> None:
            if event.type == SessionEventType.ASSISTANT_MESSAGE_DELTA:
                print(event.data.delta_content, end="", flush=True)
            elif event.type == SessionEventType.SESSION_IDLE:
                done.set()

        session.on(handle_event)
        send_task = asyncio.create_task(
            session.send_and_wait("Tell me a three-sentence story about a curious cat.")
        )
        await done.wait()
        await send_task
        print()
    finally:
        await client.stop()


if __name__ == "__main__":
    asyncio.run(main())
```

`ASSISTANT_MESSAGE_DELTA` carries text in `event.data.delta_content`. `SESSION_IDLE` means the session finished processing the current prompt.

### 3.5 Multi-turn conversation

So far each script has created a fresh session, sent one prompt, and exited. The session-as-conversation idea hasn't really been tested. Before adding custom tools and making things interesting, we'll spend one step on the conversation primitive itself: a small **REPL** in the terminal where one session lives across many prompts.

The point is simple: a session holds the running history of messages — your prompts and the assistant's replies — so the model can resolve pronouns, follow-up references, and "yes do that"-style answers. We're stepping back from streaming for this one to keep the loop body tight; streaming returns in Step 6.3 once the rest of the moving parts are in place.

Replace `scripts/sdk_playground.py` with:

```python
import asyncio

from copilot import CopilotClient
from copilot.session import PermissionHandler


async def main() -> None:
    client = CopilotClient()
    await client.start()
    try:
        session = await client.create_session(
            on_permission_request=PermissionHandler.approve_all,
            model="gpt-4.1",
        )

        print("Type a message and press Enter. Type 'quit' to exit.\n")
        while True:
            try:
                prompt = input("you> ").strip()
            except (EOFError, KeyboardInterrupt):
                print()
                break
            if not prompt:
                continue
            if prompt.lower() in {"quit", "exit"}:
                break

            response = await session.send_and_wait(prompt)
            print(f"assistant> {response.data.content}\n")
    finally:
        await client.stop()


if __name__ == "__main__":
    asyncio.run(main())
```

Run it from `mergington-app/`:

```bash
python scripts/sdk_playground.py
```

Try a couple of follow-ups that depend on prior turns (pronouns, "make it shorter", "use the same tone"). Quit, restart, and ask the recall question first — confirm the memory is gone. The context lives in the session, not the model.

> 📌 **What is the session actually holding?** Conceptually, the message history: every prompt you sent plus every assistant reply, in order. When you ask a follow-up, the model sees that whole transcript as context — which is why pronouns and references work. Once tools are added, the session also holds the tool-call history (which tools were called, with what arguments, what they returned). Same idea, more channels.

> **One client, one session, many sends.** The pattern `client.start()` → `create_session()` → loop of `session.send_and_wait()` → `client.stop()` is the canonical shape for any interactive Copilot SDK app. The web chat in Step 7 is the same pattern with `input()` replaced by an HTTP request and `print()` replaced by an SSE stream.

### 🛠️ Step 3.6 — Custom tools, including a write tool

Streaming and multi-turn are about *talking* to the model. Tools are what make the model *do* things in your world. This is the step where the SDK stops feeling like a fancier chat library and starts feeling like an agent framework — the model can decide, on its own, to call functions you wrote, look at what they returned, and use that to shape its next reply.

We'll do this in three passes. **3.6.1** adds a warm-up tool — `get_temperature(city)` — so the moving parts of a custom tool are visible on a tiny example. **3.6.2** swaps in the two real tools the Mergington integration in Step 7 needs: `list_activities` (a **read** tool) and `register_kid` (a **write** tool), wired into the multi-turn REPL with a before/after snapshot so the mutation is visible. **3.6.3** layers streaming back on top so you can watch the model decide-call-respond live.

**Step 3.6.1 — A warm-up tool: `get_temperature`**

Every custom tool, no matter how fancy, is four things:

1. A **name** the model can reference.
2. A **description** so the model knows *when* to call it.
3. A **parameter schema** (a Pydantic model) so the SDK can validate arguments before your code ever runs.
4. A **handler** — an `async` function that does the work and returns a value the model can read.

`get_temperature` is the smallest possible example of all four. It returns a hardcoded number — there's no real API call — so nothing about the lesson is hidden behind networking. The script drives the tool with a small fixed list of prompts (known city, follow-up requiring reasoning over the returned number, unknown city) so you can read each turn against the data.

Replace `scripts/sdk_playground.py` with:

```python
import asyncio

from copilot import CopilotClient
from copilot.session import PermissionHandler
from copilot.tools import define_tool
from pydantic import BaseModel, Field


# Hardcoded "weather data". A real version would call an API.
FAKE_TEMPS_C: dict[str, float] = {
    "madrid": 22.5,
    "london": 14.0,
    "reykjavik": 4.5,
    "buenos aires": 27.0,
}


class TemperatureParams(BaseModel):
    city: str = Field(description="City name in English, e.g. 'Madrid' or 'London'.")


@define_tool(
    description="Get the current temperature in Celsius for a given city. "
                "Returns a number, or an error string if the city is unknown."
)
async def get_temperature(params: TemperatureParams) -> str:
    key = params.city.strip().lower()
    if key not in FAKE_TEMPS_C:
        known = ", ".join(sorted(FAKE_TEMPS_C))
        return f"Error: no data for {params.city!r}. Known cities: {known}."
    return f"{FAKE_TEMPS_C[key]} °C"


async def main() -> None:
    client = CopilotClient()
    await client.start()
    try:
        session = await client.create_session(
            on_permission_request=PermissionHandler.approve_all,
            model="gpt-4.1",
            tools=[get_temperature],
        )

        prompts = [
            "What's the temperature in Madrid right now?",
            "And in Reykjavik? Should I bring a coat?",
            "How about Tokyo?",
        ]
        for prompt in prompts:
            print(f"\n>>> {prompt}")
            response = await session.send_and_wait(prompt)
            print(response.data.content)
    finally:
        await client.stop()


if __name__ == "__main__":
    asyncio.run(main())
```

Run it from `mergington-app/`:

```bash
python scripts/sdk_playground.py
```

The numbers in the answers should match `FAKE_TEMPS_C` for the known cities — that only happens if the tool actually ran. For the unknown city, watch how the model handles the error string the tool returned.

**Step 3.6.2 — The real tools: `list_activities` and `register_kid`**

Same mechanics, more interesting payload. Now we register two tools that operate on a small in-memory copy of the Mergington activities data, and combine them with the multi-turn REPL pattern from Step 5 so you can talk to the assistant the way a parent will in Step 7.

The two tools:

- `list_activities()` — **read.** Returns every activity with schedule and spots left.
- `register_kid(name, activity)` — **write.** Appends a kid to an activity's participant list. Returns a confirmation string, or an error string if the activity doesn't exist.

The error case is intentional: rather than raising, the tool returns a string the model can see. That way, if the user asks to sign Maya up for *"Chest Club"* (typo), the model gets the error back, can apologize, and re-prompt — instead of crashing the script. A `deepcopy` snapshot at the start and a before/after print after the client shuts down make the mutation visible at the end of the run.

Replace `scripts/sdk_playground.py` with:

```python
import asyncio
from copy import deepcopy

from copilot import CopilotClient
from copilot.session import PermissionHandler
from copilot.tools import define_tool
from pydantic import BaseModel, Field


# Tiny in-memory activities dict — same shape as backend/data/activities.json,
# trimmed down so the script stays self-contained.
activities: dict[str, dict] = {
    "Chess Club": {
        "description": "Learn strategies and compete in chess tournaments",
        "schedule": "Fridays, 3:30 PM - 5:00 PM",
        "max_participants": 12,
        "participants": ["michael@mergington.edu", "daniel@mergington.edu"],
    },
    "Programming Class": {
        "description": "Learn programming fundamentals and build software projects",
        "schedule": "Tuesdays and Thursdays, 3:30 PM - 4:30 PM",
        "max_participants": 20,
        "participants": ["emma@mergington.edu", "sophia@mergington.edu"],
    },
    "Gym Class": {
        "description": "Physical education and sports activities",
        "schedule": "Mondays, Wednesdays, Fridays, 2:00 PM - 3:00 PM",
        "max_participants": 30,
        "participants": ["john@mergington.edu", "olivia@mergington.edu"],
    },
}


@define_tool(description="List every activity with schedule and spots left.")
async def list_activities() -> list[dict]:
    return [
        {
            "name": name,
            "description": d["description"],
            "schedule": d["schedule"],
            "spots_left": d["max_participants"] - len(d["participants"]),
        }
        for name, d in activities.items()
    ]


class RegisterKidParams(BaseModel):
    name: str = Field(description="The kid's name or email, e.g. 'Maya' or 'maya@mergington.edu'.")
    activity: str = Field(description="Exact activity name, e.g. 'Chess Club'.")


@define_tool(
    description="Sign a kid up for one activity. Returns a confirmation, "
                "or an error message if the activity does not exist."
)
async def register_kid(params: RegisterKidParams) -> str:
    if params.activity not in activities:
        available = ", ".join(activities.keys())
        return f"Error: no activity named {params.activity!r}. Available: {available}."
    activities[params.activity]["participants"].append(params.name)
    return f"Signed up {params.name} for {params.activity}."


async def main() -> None:
    before = deepcopy(activities)
    client = CopilotClient()
    await client.start()
    try:
        session = await client.create_session(
            on_permission_request=PermissionHandler.approve_all,
            model="gpt-4.1",
            tools=[list_activities, register_kid],
        )

        print("Type a message and press Enter. Type 'quit' to exit.\n")
        while True:
            try:
                prompt = input("you> ").strip()
            except (EOFError, KeyboardInterrupt):
                print()
                break
            if not prompt:
                continue
            if prompt.lower() in {"quit", "exit"}:
                break

            response = await session.send_and_wait(prompt)
            print(f"assistant> {response.data.content}\n")
    finally:
        await client.stop()

    print("--- Participants before vs. after ---")
    for name in activities:
        print(f"{name}:")
        print(f"  before: {before[name]['participants']}")
        print(f"  after : {activities[name]['participants']}")


if __name__ == "__main__":
    asyncio.run(main())
```

Run it from `mergington-app/`:

```bash
python scripts/sdk_playground.py
```

Drive a short conversation: ask what's available, sign two different kids up for two different activities, then ask a follow-up referring to *"the second one"* without restating which activity you meant. Type `quit` when done. The before/after print confirms whether both kids made it into the participants list — proof the model didn't just describe the world, it *changed* it.

> **Read tools vs. write tools — and why permissions matter.** From the SDK's side, `list_activities` and `register_kid` are registered the same way. From your side they are not the same thing at all. A read tool returning bad data is annoying; a write tool firing incorrectly is a bug your users see. In production you'd swap `PermissionHandler.approve_all` for a callback that confirms before any tool with side effects runs. We stay on `approve_all` in this lab because the side effect (mutating an in-memory dict) is harmless, but Step 1's preview interaction is already a useful design exercise: *should* the parent be asked to confirm before the kid is signed up? The SDK gives you the hook to do it.

**Step 3.6.3 — Bring streaming back**

With both tools working, layer streaming on top so you can watch the model think in real time as it decides whether to call a tool, calls it, and uses the result to shape its answer. This is the final playground state — streaming, multi-turn, two real tools — that Step 3.7's integration will mirror.

The wiring is the same as Step 3.4 (`streaming=True`, an event handler, an `asyncio.Event`), now living alongside the tools and the REPL. Inside the loop, `done.clear()` resets the event for each turn, the send is kicked off as a task, and the main coroutine waits on the event for `SESSION_IDLE` before reading the next prompt.

Replace `scripts/sdk_playground.py` with:

```python
import asyncio
from copy import deepcopy

from copilot import CopilotClient
from copilot.session import PermissionHandler
from copilot.generated.session_events import SessionEventType
from copilot.tools import define_tool
from pydantic import BaseModel, Field


activities: dict[str, dict] = {
    "Chess Club": {
        "description": "Learn strategies and compete in chess tournaments",
        "schedule": "Fridays, 3:30 PM - 5:00 PM",
        "max_participants": 12,
        "participants": ["michael@mergington.edu", "daniel@mergington.edu"],
    },
    "Programming Class": {
        "description": "Learn programming fundamentals and build software projects",
        "schedule": "Tuesdays and Thursdays, 3:30 PM - 4:30 PM",
        "max_participants": 20,
        "participants": ["emma@mergington.edu", "sophia@mergington.edu"],
    },
    "Gym Class": {
        "description": "Physical education and sports activities",
        "schedule": "Mondays, Wednesdays, Fridays, 2:00 PM - 3:00 PM",
        "max_participants": 30,
        "participants": ["john@mergington.edu", "olivia@mergington.edu"],
    },
}


@define_tool(description="List every activity with schedule and spots left.")
async def list_activities() -> list[dict]:
    return [
        {
            "name": name,
            "description": d["description"],
            "schedule": d["schedule"],
            "spots_left": d["max_participants"] - len(d["participants"]),
        }
        for name, d in activities.items()
    ]


class RegisterKidParams(BaseModel):
    name: str = Field(description="The kid's name or email, e.g. 'Maya' or 'maya@mergington.edu'.")
    activity: str = Field(description="Exact activity name, e.g. 'Chess Club'.")


@define_tool(
    description="Sign a kid up for one activity. Returns a confirmation, "
                "or an error message if the activity does not exist."
)
async def register_kid(params: RegisterKidParams) -> str:
    if params.activity not in activities:
        available = ", ".join(activities.keys())
        return f"Error: no activity named {params.activity!r}. Available: {available}."
    activities[params.activity]["participants"].append(params.name)
    return f"Signed up {params.name} for {params.activity}."


async def main() -> None:
    before = deepcopy(activities)
    client = CopilotClient()
    await client.start()
    try:
        session = await client.create_session(
            on_permission_request=PermissionHandler.approve_all,
            model="gpt-4.1",
            tools=[list_activities, register_kid],
            streaming=True,
        )

        done = asyncio.Event()

        def handle_event(event) -> None:
            if event.type == SessionEventType.ASSISTANT_MESSAGE_DELTA:
                print(event.data.delta_content, end="", flush=True)
            elif event.type == SessionEventType.SESSION_IDLE:
                done.set()

        session.on(handle_event)

        print("Type a message and press Enter. Type 'quit' to exit.\n")
        while True:
            try:
                prompt = input("you> ").strip()
            except (EOFError, KeyboardInterrupt):
                print()
                break
            if not prompt:
                continue
            if prompt.lower() in {"quit", "exit"}:
                break

            done.clear()
            print("assistant> ", end="", flush=True)
            send_task = asyncio.create_task(session.send_and_wait(prompt))
            await done.wait()
            await send_task
            print("\n")
    finally:
        await client.stop()

    print("--- Participants before vs. after ---")
    for name in activities:
        print(f"{name}:")
        print(f"  before: {before[name]['participants']}")
        print(f"  after : {activities[name]['participants']}")


if __name__ == "__main__":
    asyncio.run(main())
```

Run it from `mergington-app/`:

```bash
python scripts/sdk_playground.py
```

✅ Run the same kind of conversation as Step 3.6.2 — confirm the responses now stream in token-by-token, and the before/after print at the end still shows both kids were signed up. The write tool worked even while streaming. This final playground state — streaming, multi-turn, two real tools — is what you'll wire into the Mergington app in Step 3.7.

> **Why streaming matters for tool calls.** When the model decides to call a tool, you see it "thinking" before the tool executes. When the tool returns, you see the model incorporate the result into its answer, live. That visibility is the difference between a frozen spinner and a usable assistant — and it's what makes Step 3.7's chat panel feel responsive.

### 🛠️ Step 3.7 — Leverage Copilot to integrate the SDK into the Mergington app

This is the integration step — and it's also the step where we stop hand-holding. You've spent the previous six steps walking a single playground script through the SDK's surface area. The final state of `mergington-app/scripts/sdk_playground.py` — streaming, multi-turn, the two real tools — is your reference implementation. You don't need a teacher to retype that into FastAPI for you — you need to **use Copilot itself** to do the wiring, the way you would on a real task at work.

So Step 3.7 reads more like a ticket than a tutorial. We'll describe **what** to build, **what "done" looks like**, and **how to prompt Copilot** to get there. The code lives in your editor, not on this page.

**Goal — what you're shipping**

A parent visits the Mergington site, sees the existing activities and signup form at the top of the page, and below them finds a new chat panel. They type a request in plain language — *"Sign Maya up for Chess Club and tell me when it meets."* — and an answer streams back word-by-word. If they asked for a signup, the activities cards above re-render to show the new participant the moment the answer finishes.

Three concrete deliverables get you there:

1. **A new module `backend/assistant.py`** that owns the SDK lifecycle (`CopilotClient` started lazily, stopped on shutdown) and defines the same two tools you already wrote — `list_activities` and `register_kid` — but pointed at the **real** `activities` dict imported from `backend.app`, not a toy dict. Plus a `stream_answer(prompt)` async generator that bridges streaming SDK events to whatever the HTTP layer needs.
2. **A streaming endpoint `POST /assistant/stream`** on the existing FastAPI app, returning a `text/event-stream` `StreamingResponse` whose body is the SSE-framed output of `stream_answer`. Plus a shutdown hook that calls `client.stop()` so the SDK process is cleaned up when uvicorn exits.
3. **A chat panel in `frontend/index.html`** (a small `<section>` with a prompt input and an answer area) and the JS in `frontend/app.js` that POSTs to the endpoint, reads the SSE stream chunk-by-chunk, appends each chunk to the answer area, and calls the existing `fetchActivities()` once the stream ends so any write-tool side effects (like a new kid signed up) refresh the activities cards on screen.

**Acceptance criteria — how you know you're done**

- The new chat panel is visible below the existing signup form on `http://localhost:8000`.
- Asking *"Which activities still have spots?"* streams an answer that names the real activities from `backend/data/activities.json` with correct spot counts — proving the tool is reading live data, not a hardcoded list.
- Asking *"Sign Maya up for Chess Club."* streams a confirmation, and after the stream completes, your frontend code calls `fetchActivities()` to refresh the cards on screen — the Chess Club card re-renders with one more participant. Reloading the page keeps Maya in the list (the write tool mutated the live dict, same one the GET endpoint serves).
- Asking for a non-existent activity gets an apology in plain language, not a 500 error — your tool returned an error string and the model relayed it.

---

## 4. Phase B — Prepare the delivery with Copilot CLI or VS Code Chat

So far you've built a working feature locally. Before pushing it to GitHub, this phase shows you one of the most underrated uses of Copilot: **using it as a thinking partner at commit time**, not just when writing code.

Most developers context-switch between their editor and a blank terminal when they need to write a commit message or a PR description. Copilot lets you stay in the flow — you ask it to look at what you actually changed and help you communicate it clearly. This matters for two reasons:

- **Quality gate.** Asking Copilot to summarize your diff before you commit forces you to read the diff *with* an explanation alongside it. Risky areas surface naturally. It's a lightweight self-review.
- **Drafting speed.** Commit messages, PR bodies, and issue descriptions are high-value text that most engineers write under time pressure and under-invest in. Copilot turns a 30-second prompt into a solid first draft you can edit in seconds.

This is not about automating your Git workflow. You will still read the output, decide what to keep, and type the final commands yourself. Copilot is the drafting layer, not the decision layer.

> **Two ways to run these prompts.** Choose whichever fits your flow — both give you the same quality of answer:
>
> - **Copilot CLI** — open a terminal and type `gh copilot suggest` or `gh copilot explain`. Best if you prefer staying in the terminal.
> - **VS Code Copilot Chat** — open the Chat panel (`Ctrl+Alt+I`) and type the prompt directly. VS Code Chat has access to your open workspace files and the Source Control diff, so it can answer with full codebase context without any extra setup. Best if you are already in the editor.

### 4.1 Review your changes before committing

Check what you have changed so far:

```powershell
git status
git add .
git diff HEAD
```

Now ask Copilot to read and explain those changes. This is your self-review step — use the output to spot anything you want to fix before committing.

**Option A — Copilot CLI:**

```text
Review my current git diff for the Mergington Copilot SDK assistant. Summarize the files changed, behavior added, and risky areas I should manually verify.
```

**Option B — VS Code Chat** (`Ctrl+Alt+I`):

Open the Source Control view (`Ctrl+Shift+G`) so your diff is loaded, then send this in the Chat panel:

```text
#changes Review the staged and unstaged changes in this workspace for the Mergington Copilot SDK assistant. Summarize the files changed, behavior added, and any risky areas I should verify before committing.
```

Read the summary carefully. If anything looks wrong or incomplete, fix it before moving on.

### 4.2 Draft the commit message

Once you are happy with the code, ask Copilot to draft the commit message. It has seen your diff and can describe what the change does concisely.

**Option A — Copilot CLI:**

```text
Draft a concise commit message for a feature branch named feature/sdk-assistant. Mention the SDK playground, FastAPI streaming endpoint, frontend stream consumer, and read/write tools.
```

**Option B — VS Code Chat:**

```text
#changes Draft a concise conventional commit message for these changes. Mention the SDK playground, FastAPI streaming endpoint, frontend stream consumer, and read/write tools.
```

Edit the suggested message to match your voice, then commit and push:

```powershell
git add .
git commit -m "feat: add Copilot SDK assistant"
git push -u origin feature/sdk-assistant
```

### 4.3 Draft the issue body

Before creating the issue in the next phase, ask Copilot to help draft a well-structured issue body. This ensures the issue description clearly communicates the problem, proposed solution, and acceptance criteria.

**Option A — Copilot CLI:**

```text
Draft a GitHub issue body for a feature request. The issue should describe updating the page styling with summer colors and adding a View Participants button to activity cards. Include a user story, problem statement, proposed solution, acceptance criteria checklist, and out of scope section.
```

**Option B — VS Code Chat:**

```text
Draft a GitHub issue body for improving activity cards with summer styling and participant visibility. Include a user story, problem statement, proposed solution split into two parts, acceptance criteria as a checklist, and out of scope items.
```

Save the output — you will paste it into the `gh issue create` command in the next section.

---

## 5. Phase C — Create the PR and tracking issue

Create the tracking issue so you have `ISSUE_NUMBER` before writing the PR body:

```powershell
gh issue create --repo OWNER/REPO --title "Improve activity cards with summer styling and participant visibility" --body "## User story
As a student or parent, I want the page to have brighter, more summery colors and the ability to see who is registered for each activity, so the experience is more engaging and informative.

## Problem
1. The current colors (dark blue #1a237e) are too formal and don't convey the energy and fun of school activities.
2. There is no way to see who is already signed up for an activity before registering.

## Proposed solution

### Part 1: Summer color palette
Update the color palette in \`styles.css\` with more summery tones:
- Primary colors: turquoise, coral, sunny yellow
- Lighter and warmer backgrounds
- Maintain good readability and contrast

### Part 2: View participants button
Add a button on each activity card that displays the list of registered participants.

## Acceptance criteria
- [ ] Header with gradient or summer-themed color
- [ ] Buttons with more vibrant colors
- [ ] Activity cards with colorful borders or accents
- [ ] Each activity card has a 'View Participants' button
- [ ] Clicking the button shows the list of registered emails/names
- [ ] The participant list can be closed/collapsed
- [ ] Works without page reload (JavaScript handles the toggle)
- [ ] Maintain accessibility (WCAG AA contrast)

## Out of scope
- Changes to the existing API routes
- Backend logic changes
- New Python dependencies"
```

Record the issue number as `ISSUE_NUMBER`.

---

## 6. Phase D — Use GitHub Copilot Coding Agent

Use Coding Agent for small, reviewable follow-up work. Do not ask it to merge, bypass review, or make security decisions alone.

### Assign the issue to `@copilot`

1. Open the issue in GitHub.
2. In **Assignees**, search for `@copilot`.
3. Assign the issue to `@copilot`.

> **Model selection:** When assigning the issue or adding comments, you can select which model the Coding Agent should use. Consider your token budget and task complexity when choosing — simpler tasks may work well with lighter models, while complex implementations may benefit from more capable ones. This helps optimize token consumption across your organization.

> **Note:** If the Coding Agent does not begin working shortly after assignment, you may trigger it by adding a comment mentioning `@copilot` with a specific request. You can also use subsequent comments to provide feedback on completed work, request modifications, or assign additional tasks within the same issue. You can see if it has started working in the **Agents** tab

4. Make the request specific in the issue body or a new comment. Start with the color update:

```text
@copilot please implement Part 1 of this issue.

Update the color palette in styles.css with summer-themed colors (turquoise, coral, sunny yellow). The header should have a gradient or vibrant summer color. Buttons should use more vibrant tones. Activity cards should have colorful borders or accents. Ensure all color changes maintain WCAG AA contrast accessibility.
```

5. Once the color changes are complete, add a follow-up comment for the participant visibility feature:

```text
@copilot please implement Part 2 of this issue.

Add a 'View Participants' button to each activity card in index.html and app.js. When clicked, the button should toggle a list showing all registered participants for that activity. Use JavaScript to show/hide the list without page reload. The button text should change to 'Hide Participants' when expanded.
```

6. If needed, add additional comments to refine the implementation:

```text
@copilot please refine the View Participants feature. Display participant emails in a cleaner bulleted list format. Add a subtle animation when the list expands/collapses. Ensure the button styling matches the new summer color palette.
```

7. Review the agent-created PR or commits before approving. **Important:** Change the PR base branch to `feature/sdk-assistant` before merging:
   - Open the PR on GitHub
   - Click "Edit" next to the PR title
   - Change the base branch from `main` to `feature/sdk-assistant`
   - Confirm the change

If `@copilot` is not assignable, confirm Coding Agent is enabled for `OWNER/REPO`.

### Review and validate locally

Now that the new features are implemented, pull the changes and validate them locally:

1. Pull the latest changes from the feature branch:

```powershell
git checkout feature/sdk-assistant
git pull origin feature/sdk-assistant
```

2. Restart uvicorn to see the new colors and buttons:

```powershell
cd app
uvicorn backend.app:app --reload
```

3. Open `http://localhost:8000` in your browser and verify:
   - The header has summer-themed colors (turquoise, coral, or gradient)
   - Buttons have more vibrant colors
   - Activity cards have colorful borders or accents
   - Each activity card has a "View Participants" button
   - Clicking the button shows/hides the participant list
   - Colors maintain good readability and contrast

### Merge to main

If everything looks good, merge the `feature/sdk-assistant` branch to `main`:

```powershell
git checkout main
git merge feature/sdk-assistant
git push origin main
```

Or merge via GitHub:

```powershell
gh pr create --repo OWNER/REPO --base main --head feature/sdk-assistant --title "feat: Copilot SDK assistant with summer styling" --body "Merges all SDK assistant features and UI improvements to main."
gh pr merge PR_NUMBER --repo OWNER/REPO --squash
```

---

## 7. Safety notes

- Do not commit secrets, tokens, passwords, private keys, or live connection strings.
- Do not paste secrets into Copilot CLI, GitHub issues, PR bodies, or `.squad` notes.
- Treat `PermissionHandler.approve_all` as a local development shortcut only.
- Write tools such as `register_kid` need explicit authorization rules before production.
- User identity is not automatically available to permission callbacks; thread it through deliberately if access control depends on the current user.
- Copilot-generated code can be wrong. Review, test, and simplify before merging.

---

## 8. Troubleshooting

### `gh auth status` fails

Run:

```powershell
gh auth login --web
```

Then retry:

```powershell
gh repo view OWNER/REPO
```

### `gh pr create` cannot find a repository

Confirm the fork and remote are set up correctly:

```powershell
git remote -v
gh repo view
```

If `origin` is missing, re-run the fork command from **section 2.2**:

```powershell
gh repo fork Geronimo-Basso/github-copilot-101 --clone --remote
```

### SDK import fails

From `mergington-app`, confirm `requirements.txt` includes `github-copilot-sdk`, then run:

```powershell
pip install -r requirements.txt
python -c "from copilot import CopilotClient; print('Copilot SDK import OK')"
```

### Stream does not appear token by token

Check:

- `streaming=True` is passed to `create_session(...)`.
- `session.on(handle_event)` is registered before `session.send_and_wait(...)`.
- The endpoint returns `StreamingResponse(..., media_type="text/event-stream")`.
- The frontend reads `response.body.getReader()` and parses `data:` lines.

### Chat signup does not refresh the UI

Check:

- `register_kid` mutates the same `activities` dictionary imported from `backend.app`.
- The frontend calls `fetchActivities()` after receiving `data: [DONE]`.
- Missing activities return an error string instead of raising an exception.

### `@copilot` is not assignable

Check repository and organization settings for GitHub Copilot Coding Agent availability. If it is disabled, use normal PR review comments and keep the human review gate.

---

## 9. Completion checklist

- [ ] Repository exists at `OWNER/REPO` and `gh repo view OWNER/REPO` succeeds.
- [ ] Work is on `feature/sdk-assistant`.
- [ ] `scripts\sdk_playground.py` uses `CopilotClient`, `PermissionHandler`, `SessionEventType`, `define_tool`, and Pydantic schemas correctly.
- [ ] Playground uses `client.start()`, `client.stop()`, `client.create_session(...)`, `session.on(...)`, and `session.send_and_wait(...)`.
- [ ] FastAPI exposes `POST /assistant/stream`.
- [ ] Shutdown hook calls `await stop_client()`.
- [ ] Frontend consumes SSE and refreshes activities after `[DONE]`.
- [ ] PR `PR_NUMBER` exists.
- [ ] Issue `ISSUE_NUMBER` exists and is linked.
- [ ] Coding Agent workflow was tested by assigning an issue to `@copilot` or by commenting `@copilot` on PR `PR_NUMBER`.
- [ ] Human reviewed all changes before merge.
