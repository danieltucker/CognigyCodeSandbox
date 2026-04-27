# Roadmap

Planned features for the Cognigy Code Node Sandbox. Items are listed roughly in order of priority, but nothing here is scheduled.

---

## Planned

### Named state saves

Add the ability to save and restore named snapshots of the full sandbox state (code, input, context, and profile). Currently only a single state is persisted to `localStorage`. Named saves would allow:

- Multiple test scenarios stored side by side (e.g. "happy path", "missing account", "timeout response")
- Quick switching between scenarios without manually re-entering data
- A simple save/load UI in the top bar or a dedicated panel

---

### Auto theme mode

Extend the current Night / Day toggle to include a third **Auto** option that follows the operating system's `prefers-color-scheme` preference, switching automatically when the system theme changes. The manual Night and Day overrides would remain available.

---

### Share via URL

Encode the full sandbox state — code, input, context, and profile — into a shareable URL. Opening the URL in a browser restores the exact configuration. Intended use cases:

- Linking a specific test case directly from a Cognigy Code Node comment (`// debug: <url>`)
- Sharing a failing scenario with a colleague without exporting or copy-pasting JSON
- Bookmarking reproducible test cases

The encoding would use URL-safe base64 or LZ-string compression to keep URLs reasonably short. No server required — the state lives entirely in the URL fragment or query string.

---

### LLM: document code

Connect to an LLM (configurable API key, stored in `localStorage`) to automatically generate documentation for the code in the editor. Output would include:

- A plain-English summary of what the code does
- A list of context variables it reads and writes
- A list of input fields it depends on
- Any side effects (api calls, profile updates, etc.)

Useful for handing off Code Nodes to other developers or documenting an existing agent.

---

### LLM: review code

Connect to an LLM to review the code in the editor against Cognigy best practices. Output would flag issues such as:

- Direct context mutation instead of `api.addToContext()`
- Missing null checks on `context.*` or `input.*` values
- Synchronous patterns that would break in an async context
- Hardcoded values that should come from context or input
- General code quality and readability improvements

Results would appear in a new **Review** tab in the Output pane alongside the existing Context Writes, Console, and Full State tabs.

---

## Completed

### Syntax-highlighted editors

The Code Node pane and all three data panels (input, context, profile) use [Ace Editor](https://ace.c9.io/) loaded via CDN — no build step required.

The code editor provides JavaScript syntax highlighting, line numbers, active-line highlight, bracket matching, and auto-indent. The JSON panels provide syntax coloring with keys, string values, numbers, and booleans each styled distinctly using the Cognigy brand palette. Both dark and light themes apply to all editor instances simultaneously.

See [README.md](README.md) for the full current feature set.
