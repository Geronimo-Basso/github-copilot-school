# GitHub Copilot SDK Lab 4: Build, Ship, and Iterate an Assistant

The lab runs in two sessions and four phases:

- **Session 1: Phase A — GitHub Copilot SDK.** Learn the SDK in *pure isolation* with one playground Python script (no web server, no frontend), then wire the concepts into the existing GitHub Copilot High School activities app.
- **Session 2: Phases B-D — Publish and iterate with Copilot.** Review and describe your work from VS Code Chat, publish the branch, open a PR, create follow-up issues, then use GitHub Copilot Coding Agent for a small autonomous iteration.

---

## 1. Prepare the GitHub repository

The lab uses the `Geronimo-Basso/github-copilot-school` repository as the starting point. You will fork it into your own GitHub account so that you have full write access and can push branches, open PRs, and use the Coding Agent freely.

### 1.1 Fork the repository

Go to the repository on GitHub at https://github.com/Geronimo-Basso/github-copilot-school and create a personal fork under your own GitHub account using the GitHub UI. Click the "Fork" button in the top-right corner and follow the prompts.

Once forked, clone your fork locally:

```powershell
git clone https://github.com/YOUR_USERNAME/github-copilot-school.git
cd github-copilot-school
```

Add the upstream remote to track the original repository:

```powershell
git remote add upstream https://github.com/Geronimo-Basso/github-copilot-school.git
git remote -v
```

### 1.2 Create the working branch

All your changes will live on a feature branch so the publish and follow-up agent workflows in later phases stay isolated and reviewable:

```powershell
git checkout -b feature/sdk-assistant
```

### Prerequisites

Before Phase A, confirm your environment:

- Python 3.10+ installed.
- A virtual environment is created and activated in the repository.

```powershell
python -m venv .venv
# Windows PowerShell
.venv\Scripts\Activate.ps1
# macOS/Linux
source .venv/bin/activate

pip install -r requirements.txt
```

---

## 2. Phase A — Build with the Copilot SDK

### What the Copilot SDK is, and what it isn't

The Copilot SDK is a library that lets your application **talk to the same agent runtime that powers Copilot CLI and the cloud Coding Agent**, from your own code. You import it, create a `CopilotClient`, open a session, and send messages — much like you'd use an HTTP client or a database driver.

What that buys you is more than a chat-completion call:

- **The agent loop is built in.** The SDK doesn't hand you a single model response. It hands you a *session* that can reason, decide to call one of your tools, observe the result, decide whether to call another, and finally answer. That loop is the same one that runs behind `copilot` in your terminal.
- **Tools are first-class.** You register Python functions as tools. The model gets their names, descriptions, and parameter schemas, and decides when to call them. The SDK handles the wire format and the routing. Crucially, tools can *read* data **or** *mutate* it — the model can take actions, not just answer questions.
- **Streaming is built in.** Subscribe to a session event stream and you get token-by-token deltas, tool-call events, session-idle signals, and the rest — without rolling your own SSE parser.

What the Copilot SDK *isn't*:

- It's **not a raw chat-completion API**. There's a session abstraction in the middle and a real agent loop around it.
- It's **not** the Copilot extension/skills API that runs inside `github.com`. Those are different surfaces for embedding into Copilot Chat itself.
- It's **not multi-language-only in name.** The SDK ships for Node.js / TypeScript, Python, Go, .NET, and Java. We're using **Python** in this lab because the GitHub Copilot High School backend is FastAPI.

### Core concepts used in this lab

Five concepts cover everything in Phase 1.

**`CopilotClient` and sessions.** `CopilotClient` is the long-lived handle to the Copilot CLI process. You `await client.start()` once when the application starts, and `await client.stop()` once when it shuts down. From the client you create **sessions** with `client.create_session(...)`. A session is one conversation: it holds the message history, the registered tools, the model choice, and the streaming configuration. Multi-turn = same session across requests; one-shot = new session each time.

**Permission handler.** Every session needs a permission handler. When the model wants to call a sensitive operation (a tool that mutates state, a shell command, a write to disk), the SDK asks your handler whether to allow it. For the lab we use the built-in `PermissionHandler.approve_all`, which approves everything without prompting. In production you'd write a callback that prompts a user or checks a policy — especially relevant for write tools like the `register_kid` you'll build in Step 6.

**Streaming events.** A session emits a stream of events. You subscribe with `session.on(handler)`. The two we care about in this lab:

- `ASSISTANT_MESSAGE_DELTA` — a chunk of the assistant's reply. The chunk text is in `event.data.delta_content`.
- `SESSION_IDLE` — the session has finished processing the latest message and is waiting for the next one. This is your "done streaming" signal.

The full enum lives in `copilot.generated.session_events.SessionEventType`.

**Custom tools — read and write.** A custom tool is a Python function (sync or async) that the model is allowed to call. You decorate it with `@define_tool(description=...)`, define its parameters with a Pydantic `BaseModel`, and pass it to the session via `tools=[my_tool]`. The model sees the description and schema, and decides when to call it. A **read** tool (like `list_activities`) returns data and has no side effects. A **write** tool (like `register_kid`) mutates application state — appending a kid to an activity's participant list, sending an email, writing to a database. From the SDK's perspective both look the same; from your application's perspective the difference is enormous, which is exactly why the permission handler exists.

### What you can build with the SDK

Knowing the moving parts is one thing — it's worth pausing on **what shape of product** they add up to. The four concepts above (sessions, permission handler, streaming, custom tools) are the same building blocks across very different applications. Three patterns are worth having in your head before you write a line of code.

**1. In-app AI assistants.** This is the GitHub Copilot High School case you're about to build: an existing web or desktop product gains a Copilot-driven chat surface that doesn't just answer questions about the domain — it can act on it. Streaming gives the assistant a responsive feel; custom tools let it sign a kid up, cancel a booking, or file a ticket; the permission handler is the seam where you decide which of those actions go through silently and which need a human in the loop. Most teams reach for the SDK here first because the agent loop is already wired — you only have to write the tools.

**2. Internal chatops and bots.** A single bot in Slack or Teams that wraps a pile of internal tooling — deploy a service, query a dashboard, open a Jira, restart a worker — behind plain English. The SDK's agent loop is what turns "ship the API gateway to staging" into the right sequence of tool calls; the permission handler is where your org policy lives ("who is actually allowed to deploy to prod?"). Streaming is less critical here than in a chat UI, but the multi-turn session model is what lets a user say "do that again but for the EU region" without restarting from scratch.

**3. Workflow and back-office automation.** Unattended agents that sit between an inbox, a queue, or a webhook and your internal systems. An incoming support request, an expense submission, a renewal notification — the agent reads it, calls tools to look up the relevant records, takes an action, and either replies or files a follow-up. The permission handler matters most in this shape: the agent is acting without a human watching each call, so the policy you encode there is effectively the safety net.

Across all three, the value the SDK offers is the same package — **agent loop + tools + permission gate** — pre-built and battle-tested. Anywhere you'd otherwise be hand-writing that loop, the SDK already has it.

---

### Part 2 — Hands-on

### 🛠️ Step 1 — What you'll build

Before any code, picture the finished interaction so you know what you're building toward.

A parent visits the GitHub Copilot High School site. Below the existing list of activities and the signup form, there's a new chat panel. They type:

> **"Sign my daughter Maya up for Chess Club, and tell me what days it meets."**

The assistant streams its answer back in real time, and two things happen under the hood. The model **calls your `register_kid` tool**, which appends `Maya` to Chess Club's participant list in the running app — the next time anyone reloads the activities page, Maya is there. It **also calls your `list_activities` tool** to ground the schedule and spot count in live data, instead of hallucinating one.

That's the integrated end state. To get there, you'll first walk one playground script through every concept the SDK exposes — first message, streaming, a multi-turn REPL, then custom tools (a tiny warm-up tool followed by the two real GitHub Copilot High School tools) — and only then wire it all into the app.

### 🛠️ Step 2 — Install the SDK

The first move is to pull the SDK into the project and confirm it imports cleanly. The app itself stays untouched for now.

1. **Open** `requirements.txt` in your editor. It currently lists:

   ```text
   fastapi
   uvicorn
   httpx
   watchfiles
   ```

2. **Add a line** for the SDK so the file reads:

   ```text
   fastapi
   uvicorn
   httpx
   watchfiles
   github-copilot-sdk
   ```

3. **Open a terminal at the workspace root** and install:

   ```bash
   pip install -r requirements.txt
   ```

### 🛠️ Step 3 — Send your first message

The smallest useful SDK program: start the client, open a session, ask one question, print the answer, shut down. No tools, no streaming, no FastAPI — just a complete one-shot round-trip you can read top-to-bottom.

This is the baseline shape of every later step: an `async def main()` that brackets a `client.start()` / `client.stop()` pair, with a session and one or more `send_and_wait` calls in between. Each subsequent step replaces the entire file — no diffs to chase.

Before creating the file, make sure the path exists:

```bash
mkdir -p app/scripts
```

Use a model available in your Copilot environment. The examples use `gpt-4.1`; if it is unavailable in your account, remove the `model` parameter and use the default configured model.

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
        response = await session.send_and_wait("What is 2 + 2? Answer in one short sentence.")
        print(response.data.content)
    finally:
        await client.stop()


if __name__ == "__main__":
    asyncio.run(main())
```

Run it from `app/`:

```bash
python scripts/sdk_playground.py
```

The first run may take a few seconds because the SDK is spawning the local `copilot` process behind the scenes.

> **`send_and_wait` versus streaming.** `send_and_wait(prompt)` blocks until the model is completely done and returns the final response object. It's the simplest possible call shape. Step 4 swaps it for the event-driven version where you see tokens as they're produced — same session API, different consumption pattern.

### 🛠️ Step 4 — Stream the response

`send_and_wait` is fine for one-line answers, but for anything longer, watching nothing happen for several seconds feels broken. This step rewrites the playground so it prints chunks as they arrive — the same shape as the live typing in Copilot Chat. 

The mechanic: pass `streaming=True` to `create_session`, register a callback with `session.on(...)`, kick `send_and_wait` off as a background task, and use an `asyncio.Event` to bridge the callback world back to the main coroutine. The callback prints deltas as they arrive and flips the event on `SESSION_IDLE`.

Replace `scripts/sdk_playground.py` with:

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
            on_permission_request=PermissionHandler.approve_all,
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

Run it from `app/`:

```bash
python scripts/sdk_playground.py
```

✅ The response should arrive incrementally, token-by-token. If everything shows up at once, your terminal may be line-buffering — that's a terminal-emulator quirk, not a script bug.

### 🛠️ Step 5 — Multi-turn conversation

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

Run it from `app/`:

```bash
python scripts/sdk_playground.py
```

Try a couple of follow-ups that depend on prior turns (pronouns, "make it shorter", "use the same tone"). Quit, restart, and ask the recall question first — confirm the memory is gone. The context lives in the session, not the model.

> 📌 **What is the session actually holding?** Conceptually, the message history: every prompt you sent plus every assistant reply, in order. When you ask a follow-up, the model sees that whole transcript as context — which is why pronouns and references work. Once tools are added, the session also holds the tool-call history (which tools were called, with what arguments, what they returned). Same idea, more channels.

> **One client, one session, many sends.** The pattern `client.start()` → `create_session()` → loop of `session.send_and_wait()` → `client.stop()` is the canonical shape for any interactive Copilot SDK app. The web chat in Step 7 is the same pattern with `input()` replaced by an HTTP request and `print()` replaced by an SSE stream.

### 🛠️ Step 6 — Custom tools, including a write tool

Streaming and multi-turn are about *talking* to the model. Tools are what make the model *do* things in your world. This is the step where the SDK stops feeling like a fancier chat library and starts feeling like an agent framework — the model can decide, on its own, to call functions you wrote, look at what they returned, and use that to shape its next reply.

We'll do this in three passes. **6.1** adds a warm-up tool — `get_temperature(city)` — so the moving parts of a custom tool are visible on a tiny example. **6.2** swaps in the two real tools the app integration in Step 7 needs: `list_activities` (a **read** tool) and `register_kid` (a **write** tool), wired into the multi-turn REPL with a before/after snapshot so the mutation is visible. **6.3** layers streaming back on top so you can watch the model decide-call-respond live. 

**Step 6.1 — A warm-up tool: `get_temperature`**

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

Run it from `app/`:

```bash
python scripts/sdk_playground.py
```

The numbers in the answers should match `FAKE_TEMPS_C` for the known cities — that only happens if the tool actually ran. For the unknown city, watch how the model handles the error string the tool returned.

**Step 6.2 — The real tools: `list_activities` and `register_kid`**

Same mechanics, more interesting payload. Now we register two tools that operate on a small in-memory copy of the GitHub Copilot High School activities data, and combine them with the multi-turn REPL pattern from Step 5 so you can talk to the assistant the way a parent will in Step 7.

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

Run it from `app/`:

```bash
python scripts/sdk_playground.py
```

Drive a short conversation: ask what's available, sign two different kids up for two different activities, then ask a follow-up referring to *"the second one"* without restating which activity you meant. Type `quit` when done. The before/after print confirms whether both kids made it into the participants list — proof the model didn't just describe the world, it *changed* it.

> **Read tools vs. write tools — and why permissions matter.** From the SDK's side, `list_activities` and `register_kid` are registered the same way. From your side they are not the same thing at all. A read tool returning bad data is annoying; a write tool firing incorrectly is a bug your users see. In production you'd swap `PermissionHandler.approve_all` for a callback that confirms before any tool with side effects runs. We stay on `approve_all` in this lab because the side effect (mutating an in-memory dict) is harmless, but Step 1's preview interaction is already a useful design exercise: *should* the parent be asked to confirm before the kid is signed up? The SDK gives you the hook to do it.

**Step 6.3 — Bring streaming back**

With both tools working, layer streaming on top so you can watch the model think in real time as it decides whether to call a tool, calls it, and uses the result to shape its answer. This is the final playground state — streaming, multi-turn, two real tools — that Step 7's integration will mirror.

The wiring is the same as Step 4 (`streaming=True`, an event handler, an `asyncio.Event`), now living alongside the tools and the REPL. Inside the loop, `done.clear()` resets the event for each turn, the send is kicked off as a task, and the main coroutine waits on the event for `SESSION_IDLE` before reading the next prompt.

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

Run it from `app/`:

```bash
python scripts/sdk_playground.py
```

✅ Run the same kind of conversation as Step 6.2 — confirm the responses now stream in token-by-token, and the before/after print at the end still shows both kids were signed up. The write tool worked even while streaming. This final playground state — streaming, multi-turn, two real tools — is what you'll wire into the app in Step 7.

> **Why streaming matters for tool calls.** When the model decides to call a tool, you see it "thinking" before the tool executes. When the tool returns, you see the model incorporate the result into its answer, live. That visibility is the difference between a frozen spinner and a usable assistant — and it's what makes Step 7's chat panel feel responsive.

### 🛠️ Step 7 — Leverage Copilot to integrate the SDK into the GitHub Copilot High School app

This is the integration step — and it's also the step where we stop hand-holding. You've spent the previous six steps walking a single playground script through the SDK's surface area. The final state of `app/scripts/sdk_playground.py` — streaming, multi-turn, the two real tools — is your reference implementation. You don't need a teacher to retype that into FastAPI for you — you need to **use Copilot itself** to do the wiring, the way you would on a real task at work.

So Step 7 reads more like a ticket than a tutorial. We'll describe **what** to build, **what "done" looks like**, and **how to prompt Copilot** to get there. The code lives in your editor, not on this page.

**Goal — what you're shipping**

A parent visits the GitHub Copilot High School site, sees the existing activities and signup form at the top of the page, and below them finds a new chat panel. They type a request in plain language — *"Sign Maya up for Chess Club and tell me when it meets."* — and an answer streams back word-by-word. If they asked for a signup, the activities cards above re-render to show the new participant the moment the answer finishes.

Three concrete deliverables get you there:

1. **A new module `app/backend/assistant.py`** that owns the SDK lifecycle (`CopilotClient` started lazily, stopped on shutdown) and defines the same two tools you already wrote — `list_activities` and `register_kid` — but pointed at the **real** `activities` dict imported from `app/backend/app.py`, not a toy dict. Plus a `stream_answer(prompt)` async generator that bridges streaming SDK events to whatever the HTTP layer needs.
2. **A streaming endpoint `POST /assistant/stream`** in `app/backend/app.py`, returning a `text/event-stream` `StreamingResponse` whose body is the SSE-framed output of `stream_answer`. Plus a shutdown hook that calls `client.stop()` so the SDK process is cleaned up when uvicorn exits.
3. **A chat panel in `app/frontend/index.html`** (a small `<section>` with a prompt input and an answer area) and the JS in `app/frontend/app.js` that POSTs to the endpoint, reads the SSE stream chunk-by-chunk, appends each chunk to the answer area, and calls the existing `fetchActivities()` once the stream ends so any write-tool side effects (like a new kid signed up) refresh the activities cards on screen.

**Acceptance criteria — how you know you're done**

- The new chat panel is visible below the existing signup form on `http://localhost:8000`.
- Asking *"Which activities still have spots?"* streams an answer that names the real activities from `backend/data/activities.json` with correct spot counts — proving the tool is reading live data, not a hardcoded list.
- Asking *"Sign Maya up for Chess Club."* streams a confirmation, and after the stream completes, your frontend code calls `fetchActivities()` to refresh the cards on screen — the Chess Club card re-renders with one more participant. Reloading the page keeps Maya in the list (the write tool mutated the live dict, same one the GET endpoint serves).
- Asking for a non-existent activity gets an apology in plain language, not a 500 error — your tool returned an error string and the model relayed it.

Run and verify Step 7 immediately after integration:

```powershell
cd app
uvicorn backend.app:app --reload
```

Then open `http://localhost:8000` and validate the acceptance criteria above.


---

## 3. Phase B — Prepare the delivery in VS Code Chat

So far you've built a working feature locally. Before pushing it to GitHub, this phase shows you one of the most underrated uses of Copilot: **using it as a thinking partner at commit time**, not just when writing code.

Most developers context-switch between their editor and a blank terminal when they need to write a commit message or a PR description. Copilot lets you stay in the flow — you ask it to look at what you actually changed and help you communicate it clearly. This matters for two reasons:

- **Quality gate.** Asking Copilot to summarize your diff before you commit forces you to read the diff *with* an explanation alongside it. Risky areas surface naturally. It's a lightweight self-review.
- **Drafting speed.** Commit messages, PR bodies, and issue descriptions are high-value text that most engineers write under time pressure and under-invest in. Copilot turns a 30-second prompt into a solid first draft you can edit in seconds.

This is not about automating your Git workflow. You will still read the output, decide what to keep, and publish the branch yourself. Copilot is the drafting layer, not the decision layer.

### 3.1 Review your changes before committing

Now ask Copilot to read and explain those changes. This is your self-review step — use the output to spot anything you want to fix before committing.

Send this in the Chat panel:

```text
#changes Review the staged and unstaged changes in this workspace for the GitHub Copilot High School Copilot SDK assistant. Summarize the files changed, behavior added, and any risky areas I should verify before committing.
```

Read the summary carefully. If anything looks wrong or incomplete, fix it before moving on.

### 3.2 Draft the commit message and PR summary

Once you are happy with the code, ask Copilot to draft the commit message. It has seen your diff and can describe what the change does concisely.

```text
#changes Draft a concise conventional commit message for these changes. Mention the SDK playground, FastAPI streaming endpoint, frontend stream consumer, and read/write tools.
```

Then draft the PR summary while the same diff context is still fresh:

```text
#changes Draft a pull request title and a short PR description for these changes. Focus on the SDK playground, the FastAPI streaming endpoint, the frontend stream consumer, and the read/write tools.
```

Edit both drafts to match your voice, then commit and push using your preferred method:

- Terminal: `git push -u origin feature/sdk-assistant`
- VS Code Source Control: **Publish Branch**

If push fails, authenticate with GitHub and retry.

### 3.3 Draft the follow-up issue

Before creating the follow-up issues in the next phase, ask Copilot to help draft smaller issue bodies. This keeps each issue focused and avoids stuffing unrelated work into one oversized request.

```text
Draft a GitHub issue body for improving activity cards with summer styling only. Include a user story, problem statement, proposed solution, acceptance criteria as a checklist, and out of scope items.
```

```text
Draft a second GitHub issue body for improving activity cards with participant visibility only. Include a user story, problem statement, proposed solution, acceptance criteria as a checklist, and out of scope items.
```

Keep both drafts nearby. In the next phase you will either hand them to Copilot in agent mode or paste them into GitHub if your current setup cannot create the issues directly.

---

## 4. Phase C — Open the PR and create the follow-up issue

This phase is about publishing the work, not memorizing `gh` syntax. Keep VS Code as the primary surface and let Copilot help with the GitHub artifacts.

### 4.1 Open the pull request

If you have not pushed the branch yet, do that first from Source Control or with `git push -u origin feature/sdk-assistant`.

Then ask Copilot in Agent mode to help you open the PR using the draft from Phase 3. A good prompt is:

```text
Use the published feature/sdk-assistant branch to open a pull request into main for this repository. Use the PR title and summary we drafted from #changes, and keep the description concise and reviewer-friendly.
```

If your GitHub-connected setup cannot open the PR directly from chat, open the PR in GitHub and paste in the title and description Copilot already drafted.

### 4.2 Create the follow-up issues

Now ask Copilot in Agent mode to create the two follow-up issues for the UI polish tasks:

```text
Create a GitHub issue in this repository titled "Improve activity cards with summer styling" using the first issue draft we just wrote. Keep it focused on colors, spacing, and visual polish, and leave participant visibility out of scope.
```

```text
Create a second GitHub issue in this repository titled "Add participant visibility to activity cards" using the second issue draft we just wrote. Keep it focused on the View Participants interaction and leave styling changes out of scope.
```

If your setup cannot create issues directly from chat, create the issues in the GitHub web UI and paste in the drafts from Phase 3.

Record both issue numbers or URLs. You will use the first issue in the next phase, then decide whether to hand off the second one after that.

---

## 5. Phase D — Use GitHub Copilot Cloud Coding Agent

Use Cloud Coding Agent for small, reviewable follow-up work. 

GitHub Docs for the cloud agent workflow: https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-cloud-agent

### Assign the styling issue to `@copilot`

1. Open the issue in GitHub.
2. In **Assignees**, search for `@copilot`.
3. Assign the issue to `@copilot`.

> **Model selection:** When assigning the issue or adding comments, you can select which model the Cloud Coding Agent should use. Consider your token budget and task complexity when choosing — simpler tasks may work well with lighter models, while complex implementations may benefit from more capable ones. This helps optimize token consumption across your organization.

> **Note (requires docs verification):** Use the optional prompt during assignment to give Copilot its initial instructions. After Copilot opens a PR, use `@copilot` comments on that PR to request refinements or follow-up changes. 

> **UI review tip:** If the change is visual, ask Copilot to include screenshots in the PR or attach screenshots as visual context in the prompt. That makes the UI changes easier to review directly from the PR.


4. If needed, add additional comments to refine the styling implementation:

```text
@copilot please refine the styling work. Tighten the color contrast, keep the layout readable on small screens, and make the summer palette feel cohesive across the page.
```

5. Review the agent-created branch, commits, or PR before approving anything. Focus on scope control, accessibility, and whether the UI changes still match the acceptance criteria in the issue.

### Optional follow-up: participant visibility issue

If you want to continue after the styling issue lands, assign the second issue to `@copilot` and use this prompt:

```text
@copilot please implement the participant visibility issue.

Add a 'View Participants' button to each activity card in index.html and app.js. When clicked, the button should toggle a list showing all registered participants for that activity. Use JavaScript to show/hide the list without page reload. The button text should change to 'Hide Participants' when expanded.
```

### Review and validate locally

Now that the new features are implemented, validate them locally before you decide what to keep.

1. Bring the agent's changes into your local workspace using your normal Git or PR checkout workflow.

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

### Final Conclusion

You now have the full loop: build the assistant in isolation, wire it into the app, review the diff in VS Code Chat, publish the work, and hand off small follow-up tasks to GitHub Copilot Cloud Coding Agent. The main takeaway is that Copilot can help at every stage of the workflow, but the human still decides what ships, what gets reviewed, and when the work is done.
