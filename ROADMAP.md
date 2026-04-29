# Roadmap

Planned features for the Cognigy Code Node Sandbox. Items are listed roughly in order of priority, but nothing here is scheduled.

---

## Planned

### Share via URL

Encode the full sandbox state — code, input, context, and profile — into a shareable URL. Opening the URL in a browser restores the exact configuration. Intended use cases:

- Linking a specific test case directly from a Cognigy Code Node comment (`// debug: <url>`)
- Sharing a failing scenario with a colleague without exporting or copy-pasting JSON
- Bookmarking reproducible test cases

The encoding would use URL-safe base64 or LZ-string compression to keep URLs reasonably short. No server required — the state lives entirely in the URL fragment or query string.

---

## Completed

### LLM prompt tools

Two buttons in the Code Node panel header — **Document with LLM** and **Review with LLM** — each build a ready-to-use prompt with the current code embedded and copy it to the clipboard. Paste into any LLM (claude.ai, Copilot, etc.) to get results immediately. No API key or direct LLM connection required.

**Document with LLM** returns the code unchanged in structure with a plain-English summary comment block at the top and concise inline comments added where the logic is non-obvious.

**Review with LLM** returns a structured review covering direct context mutation, missing null checks, hardcoded values, error handling risks, and general code quality — with specific findings, locations, and recommended fixes.

---

### Saves and scenario testing

A full test-case management system accessible from the **Saves** button in the top bar.

**Code saves** store a named JavaScript implementation. **Scenarios** are named snapshots of `input`, `context`, and `profile` attached to a code save, allowing multiple data configurations to be tested against the same code without re-entering data.

**Batch run** — each save with at least one scenario shows a **▶ Run all** button. Results are displayed in a summary view with pass/fail per scenario, context write counts, console log counts, and expandable detail.

**Assertions** — an always-visible panel at the top of the Output pane where expected post-run values are defined (e.g. `context.handover = true` or `context.locale`). Assertions are evaluated after every run — single or batch — and reported inline with the actual value observed. A batch scenario is only marked passing if the code runs clean and all assertions pass.

Each row in the Context Writes tab includes a **+ assert** button that creates or updates an assertion pre-populated with the written path and value. Clicking an existing assertion's text opens it for inline editing — press Enter to save or Escape to cancel.

Saves and assertions are stored separately in `localStorage` and are not affected by **Clear**.

---

### Theme persistence and Auto mode

The theme toggle cycles through three modes — **Night**, **Day**, and **Auto** — and persists the selection across page refreshes via `localStorage`. Auto mode follows the operating system's `prefers-color-scheme` preference and updates immediately when the system theme changes, with no page reload required.

---

### Syntax-highlighted editors

The Code Node pane and all three data panels (input, context, profile) use [Ace Editor](https://ace.c9.io/) loaded via CDN — no build step required.

The code editor provides JavaScript syntax highlighting, line numbers, active-line highlight, bracket matching, and auto-indent. The JSON panels provide syntax coloring with keys, string values, numbers, and booleans each styled distinctly using the Cognigy brand palette. Both dark and light themes apply to all editor instances simultaneously.

See [README.md](README.md) for the full current feature set.
