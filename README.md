# Cognigy Code Node Sandbox

A single-file browser tool for writing, testing, and debugging Cognigy Code Node JavaScript without deploying to a live agent. Paste your code, configure the input state, run it, and inspect what changed — all locally.

Built by **Daniel Tucker, Senior Solutions Architect — NiCE ProServ**.

For planned features, see [ROADMAP.md](ROADMAP.md).

---

![App overview](images/app-view.png)

---

## What it does

Cognigy Code Nodes execute JavaScript with access to several injected objects: `api` / `actions`, `context`, `input`, `profile`, and `moment`. Testing this code normally requires a live Cognigy session with real conversation state. This sandbox simulates that environment in the browser so you can iterate quickly.

**You can:**
- Write and run Code Node JS in a syntax-highlighted editor with line numbers, bracket matching, and auto-indent
- See every `api.addToContext()`, `setContext()`, `deleteContext()`, and `updateProfile()` call recorded and displayed
- Inspect the full `input`, `context`, and `profile` state after execution
- See `console.log`, `console.warn`, and `console.error` output in a dedicated console tab
- Define the complete `input` object as JSON — including `input.data`, meta fields like `sessionId`, and extension results like `input.getAccountStatus`
- Paste code and get a modal prompt if it references `context.*`, `input.data.*`, or `input.*` values not yet defined — with a one-click auto-add

---

## Usage

Open `cognigy-node-sandbox.html` directly in a browser. No build step, no server, no dependencies to install.

### Layout

The workspace is split into three resizable panes:

| Pane | Purpose |
|---|---|
| **Code Node** | Write or paste your Code Node JavaScript here |
| **Data** | Set the runtime state: `input`, `context`, and `profile` |
| **Output** | View context writes, console output, and full post-run state |

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

![Error handling](images/error-handling.png)

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

### context

The context object as it exists when the Code Node begins executing. Set any context variables your code reads here.

### profile

The contact profile. Populated into `profile` and writable via `api.updateProfile()`.

---

## Persistence

All panels and the code editor save automatically to `localStorage` on every keystroke. Content is restored on page load with no manual save step.

**Clear** (top bar) resets the code editor to empty and all data panels to `{}`, and removes the saved state from `localStorage`.

---

## Theme

Click the **Night / Day** toggle in the top bar to switch between dark and light modes. Both themes use the Cognigy brand palette and apply to all editors.

---

## Keyboard shortcuts

| Shortcut | Action |
|---|---|
| `⌘ Enter` / `Ctrl Enter` | Run code |
| `Tab` | Indent (handled natively by the editor) |

---

## Limitations

- **No async/await support** — the sandbox executes code synchronously. Code that relies on `await` or returns a Promise will not behave as expected.
- **No real HTTP calls** — `api.httpRequest()` and similar are not implemented. Simulate HTTP responses by pre-populating `input.data` in the Input panel.
- **No NLU** — intent matching, slot filling, and NLU results are not simulated.
- **Extension node results** — results from Cognigy extension nodes (e.g., `input.getAccountStatus`) must be manually added as top-level keys in the **Input** panel.
- **Single execution** — the sandbox runs your code once per click. Multi-turn conversation state must be set up manually between runs.
