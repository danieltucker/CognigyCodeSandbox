# Cognigy Code Node Sandbox

A single-file browser tool for writing, testing, and debugging Cognigy Code Node JavaScript without deploying to a live agent. Paste your code, configure the input state, run it, and inspect what changed — all in the browser, no server required.

Built by **Daniel Tucker, Senior Solutions Architect — NiCE ProServ**.

For planned features, see [ROADMAP.md](ROADMAP.md).

---

![App overview](images/app-view.png)

---

## What it does

Cognigy Code Nodes run JavaScript with access to injected objects: `api` / `actions`, `context`, `input`, `profile`, and `moment`. Normally, testing this code requires a live Cognigy session with real conversation state. This sandbox simulates that environment locally so you can iterate without deploying.

**You can:**
- Write and run Code Node JS in a syntax-highlighted editor with line numbers, bracket matching, and auto-indent
- See every `api.addToContext()`, `setContext()`, `deleteContext()`, and `updateProfile()` call recorded and displayed
- Inspect the full `input`, `context`, and `profile` state after execution
- See `console.log`, `console.warn`, and `console.error` output in a dedicated Console tab
- Define the complete `input` object as JSON — including `input.data`, meta fields like `sessionId`, and extension results like `input.getAccountStatus`
- Paste code and get a modal prompt listing any `context.*`, `input.data.*`, or `input.*` references not yet defined in the data panels — with a one-click **Auto-Add All**
- Save named Code Nodes and test them against multiple named data configurations (input / context / profile snapshots)
- Define output assertions and validate them on every run — single or batch

---

## Usage

Open `cognigy-node-sandbox.html` directly in a browser. No build step, no server, no dependencies to install.

### Layout

The workspace is split into three resizable panes:

| Pane | Purpose |
|---|---|
| **Code Node** | Write or paste your Code Node JavaScript here |
| **Data** | Set the runtime state: `input`, `context`, and `profile` |
| **Output** | View context writes, console output, full post-run state, and assertion results |

Drag the dividers between panes to resize them horizontally. Drag the **Context** or **Profile** section headers in the Data pane to resize those panels vertically.

### Running code

Click **Run** or press `⌘ Enter` (Mac) / `Ctrl Enter` (Windows/Linux).

### Code editor

The Code Node pane uses an embedded [Ace Editor](https://ace.c9.io/) with:

- JavaScript syntax highlighting
- Line numbers and active-line highlight
- Bracket matching and auto-indent
- Soft tabs (2 spaces)

The three data panels (input, context, profile) also use Ace in JSON mode, with keys, string values, numbers, and booleans each colored distinctly.

### Available globals

| Variable | Description |
|---|---|
| `api` | Cognigy API object (same as `actions`) |
| `actions` | Alias for `api` — paste real Code Node JS without renaming |
| `context` | Mutable context object, pre-populated from the Context panel |
| `input` | The complete input object, built from the Input panel |
| `profile` | Contact profile object, pre-populated from the Profile panel |
| `moment` | [Moment.js](https://momentjs.com/) 2.30 |
| `_` | [Lodash](https://lodash.com/) 4.17 |

### Supported `api` / `actions` methods

| Method | Behaviour in sandbox |
|---|---|
| `addToContext(path, value, mode)` | Writes to context; recorded in Context Writes tab |
| `setContext(path, value)` | Same as above |
| `getContext(path)` | Reads from the current context object |
| `deleteContext(path)` / `removeFromContext(path)` | Deletes from context; recorded |
| `resetContext()` | Clears all context keys; recorded |
| `updateProfile(updates)` | Merges into profile; recorded |
| `say(text)` | Logged to console tab as `[api.say]` |
| `output(obj)` | Logged to console tab as `[api.output]` |
| `log(msg)` / `logDebugMessage(msg)` | Logged to console tab |
| `logDebugError(msg)` | Logged to console tab as error |
| `setNextNode(nodeId)` | Logged to console tab |
| `stopExecution()` | Logged to console tab as warning |
| `setState(state)` / `getState()` | `setState` logged; `getState` returns null |
| `setAppState(key, val)` | Logged to console tab |
| `handover(config)` | Logged to console tab as warning |
| `completeGoal(goal)` | Logged to console tab |
| `base64Encode(str)` / `base64Decode(str)` | Functional |

### Output tabs

- **Context Writes** — every context operation in order, with path, operation type, value type, and value
- **Console** — all `console.*` output and `api.*` side-effect calls
- **Full State** — the complete `input`, `context`, and `profile` objects as they exist after the run

Runtime errors are surfaced in both the Context Writes and Console tabs with the full error message.

![Error handling](images/errors.png)

---

## Paste detection

When you paste code into the editor, the sandbox scans it for references to `context.*`, `input.data.*`, and `input.*` values that don't exist in the current data panels. If any are missing, a modal appears listing them with a single **Auto-Add All** button that inserts `null` placeholders at the correct paths.

![Variable detection modal](images/variable-detection.png)

Detection covers:
- `context.foo`, `context?.foo?.bar`, `context['key']`
- `api.getContext('some.path')`
- `input.data.foo`, `input?.data?.bar`
- `input.getAccountStatus?.result` and other `input.*` chains (including extension results)

After auto-adding, edit the placeholder values in the data panels before running.

---

## Saves

The **Saves** button in the top bar opens the Saves panel, where you can store named Code Node implementations and test them against multiple data configurations.

### Structure

Saves use a two-level hierarchy:

- **Code save** — the JavaScript for a specific Code Node, stored with a name (e.g. "Get Account Status")
- **Scenario** — a named snapshot of `input`, `context`, and `profile` attached to a code save (e.g. "Happy path", "Missing account")

This lets you test the same Code Node against multiple data configurations without re-entering data between runs.

### Creating a save

1. Write or paste your code into the Code Node editor
2. Click **Saves** in the top bar
3. Type a name and click **Save code** (or press Enter)

### Adding scenarios

1. Configure the Data panels for the test case you want to capture
2. Open **Saves**, find the relevant code save, type a scenario name in the input at the bottom of the card, and click **Add**

The current contents of all three data panels are snapshotted under that name. Repeat for each test scenario.

### Loading

| Action | What it restores |
|---|---|
| **Load code** | The saved JavaScript — data panels unchanged |
| **Load** (on a scenario) | The saved input, context, and profile — code unchanged |

### Batch run

Each save with at least one scenario shows a **▶ Run all** button. Clicking it runs the saved code against every scenario in sequence and displays a results summary with:

- A pass/fail badge per scenario
- Per-scenario counts for context writes, console logs, and assertions
- Expandable detail showing context writes, console output, and per-assertion outcomes

A scenario passes if it runs without a runtime error **and** all defined assertions pass. If no assertions are defined, a clean run is sufficient.

![Scenario run results](images/scenario-run.png)

Saves are stored in `localStorage` independently of the workspace state and are never reset by **Clear**.

---

## Assertions

The **Assertions** panel at the top of the Output pane lets you define expected post-run values. After every run — single or batch — each assertion is evaluated against the final state and shows a pass or fail result inline.

![Assertions panel](images/assertions.png)

### Syntax

| Expression | Passes when |
|---|---|
| `context.handover` | `context.handover` exists and is not null |
| `context.handover = true` | `context.handover` strictly equals `true` |
| `context.locale = "en-GB"` | `context.locale` strictly equals `"en-GB"` |
| `context.retryCount = 3` | `context.retryCount` strictly equals `3` |
| `profile.escalated = false` | `profile.escalated` strictly equals `false` |

Supported roots: `context.*` and `profile.*`. Supported value literals: `true`, `false`, `null`, numbers, quoted strings.

### Adding assertions from context writes

After a run, each row in the Context Writes tab shows a **+ assert** button. Clicking it creates an assertion pre-populated with the path and the actual value that was written — no typing required. If an assertion for that path already exists, it is updated to the new value.

### Editing assertions

Click any assertion's text to edit it inline. Press **Enter** to save or **Escape** to cancel.

### Single run

After each run, each assertion badge updates in place: `✓ <actual value>` or `✗ <actual value>`. The topbar status also reports the assertion tally: `OK - 2 writes · 2/2 assertions`.

### Batch run

Assertions are evaluated per scenario. A scenario is marked as failed if any assertion fails, even if the code ran without errors. The batch results view includes an Assertions section in each scenario's expanded detail, showing the actual value observed for each assertion.

Assertions persist in `localStorage` and apply across all runs until removed.

---

## Data panels

### input

The complete `input` object as it will exist when your code runs. Define any combination of fields in a single JSON object:

```json
{
  "text": "hello",
  "sessionId": "sandbox-session-001",
  "data": {
    "request": { "estimatedWaitTime": 420000 }
  },
  "getAccountStatus": {
    "result": { "account": { "accountId": "12345" } }
  }
}
```

Fields not specified fall back to these defaults at run time:

| Field | Default |
|---|---|
| `text` | `""` |
| `sessionId` | `sandbox-session-001` |
| `userId` | `sandbox-user-001` |
| `flowName` | `Main Flow` |
| `URLToken` | `dev` |
| `currentTime` | Auto-set to current time |
| `data` | `{}` |

Any field you define overrides its default. Use the `data` key for webhook payloads and HTTP node responses. Use top-level keys to simulate extension node results (e.g., `getAccountStatus`, `getOrderStatus`).

The **Gen IDs** button in the Input panel header generates fresh UUIDs for `userId` and `sessionId` if either is missing or null — useful when you want unique identifiers without typing them manually.

### context

The context object as it exists when the Code Node begins executing. Set any context variables your code reads here.

### profile

The contact profile. Populated into `profile` and writable via `api.updateProfile()`.

---

## Persistence

All panels and the code editor save to `localStorage` on every keystroke and are restored on page load — no manual save step needed.

**Clear** (top bar) empties the code editor, resets all data panels to `{}`, and removes the active workspace state from `localStorage`. Saves and assertions are not affected by Clear.

---

## Theme

Click the theme toggle in the top bar to cycle through three modes:

| Mode | Behaviour |
|---|---|
| **Night** 🌙 | Dark theme using the Cognigy brand palette |
| **Day** ☀️ | Light theme using the Cognigy brand palette |
| **Auto** ◑ | Follows the OS `prefers-color-scheme` setting, updating live when the system theme changes |

The selected mode persists across page refreshes. All editors switch theme simultaneously.

---

## Keyboard shortcuts

| Shortcut | Action |
|---|---|
| `⌘ Enter` / `Ctrl Enter` | Run code |
| `Tab` | Indent (handled natively by the editor) |

---

## Limitations

- **No async/await support** — execution is synchronous. Code using `await` or returning a `Promise` will not behave as expected.
- **No real HTTP calls** — `api.httpRequest()` and similar are not implemented. Simulate responses by pre-populating `input.data` in the Input panel.
- **No NLU** — intent matching, slot filling, and NLU results are not simulated.
- **Extension node results** — results from Cognigy extension nodes (e.g., `input.getAccountStatus`) must be added manually as top-level keys in the Input panel.
- **Single execution** — the sandbox runs code once per click. Multi-turn conversation state must be set up manually between runs.
