---
name: flow
description: Explain how a feature works end to end in a Python web repo (Django/Flask/FastAPI) as a visual infographic (stages, tables, branches, code on click) using `lenscheck flow` — AST trace + Claude explanation. Use on /flow <words>, "flow of X", "how does X work end to end", "show me the steps of X". `/flow list` opens the catalog of every entrypoint and saved journey.
---

# /flow

Mental model: **lenscheck finds, Claude explains, the page draws.** One API's flow is traced directly; a
multi-step thing (a wizard, "the 5 steps") is a **journey** — a small YAML naming which steps call which
APIs, in user order. Journeys are saved and reused; pages are cached.

## Paths
- lenscheck: `LC="${LENSCHECK:-lenscheck} flow"` (set `LENSCHECK="python3 /path/to/cli.py"` to run from a checkout)
- journeys: `${FLOW_JOURNEYS:-$HOME/.lenscheck/journeys}/<repo>/<name>.yaml` — YAML only, never HTML
- pages: `$HOME/.cache/lenscheck/flows/<repo>/` — they embed your source code: never commit them, never publish them
- `<repo>` = the repo folder name; open pages with `xdg-open` (Linux) or `open` (macOS)

## Steps
0. **Setup — ask before the first trace.** If `<journeys dir>/repos.yaml` exists, show it and ask "same repos?";
   otherwise ask in ONE message (all optional except one backend repo; default = the current git root):
   - **Backend repos** — paths (`path@branch` allowed). The first is `<repo>`, where the entry or journey lives.
   - **Frontend repos** — paths of the web / mobile apps that call these APIs.
   - **App paths** — folders inside a backend repo to focus on (e.g. `apps/orders`), or "whole repo".

   Save the answers to `<journeys dir>/repos.yaml` (`backend: [...]`, `frontend: [...]`, `apps: {<repo>: [...]}`).
   How each answer is used:
   - Other backend repos → `--crossrepo` (and a journey's `crossrepo:` key), only when the flow writes tables they
     read, calls their routes or dispatches their tasks; skip them for single-repo CRUD.
   - App paths are **never** passed as `<repo>` — lenscheck needs the whole repo to resolve models and imports.
     Use them to filter the catalog, to prefer candidates whose location is under them before asking `--pick`,
     and to limit what you read when drafting a journey.
   - Frontend repos are not traced by lenscheck; read them for step order and for which API each screen calls
     (step 4b), and cite them in the journey's `source:`.
1. `/flow list` → `$LC <repo> --catalog --journeys <journeys dir> --html <pages dir>/` → open `<pages dir>/index.html`
   (with app paths: also say which catalog groups match them).
2. **Saved journey** whose file name or `title` matches the words → say up front that a fresh explanation is one
   `claude -p` call (~3–5 min, then cached) → `$LC <repo> --journey <file> --html <pages dir>/ --explain`
   → refresh the index (step 1, don't open it) → open the page.
3. **One entrypoint** (the words name one API / command / task and don't describe a multi-step thing) →
   `$LC <repo> "<words>"`; exit 2 = ambiguous: show the ranked list, ask which, rerun with `--pick N`;
   then same as step 2 with the query instead of `--journey`.
4. **Multi-step thing with no saved journey** ("creation", "wizard", "N steps", "setup", "onboarding") → draft one:
   a. `$LC <repo> --catalog --json <scratch>/catalog.json`; read the relevant app groups (summaries, writes).
   b. Find the order in code: step/progress helpers (`step_progress`, `steps = [`), "step N" in API docs/summaries,
      URL grouping, and the frontend repos: wizard/route order and the API calls each screen makes
      (`fetch(`, `axios.`, API client modules).
   c. Map each step to the endpoints that WRITE the data that step's completion needs — read the code, never
      map from URL names alone. Use `Handler.method`; `Handler@file` if two definitions share a name.
   d. Write a DRAFT YAML (`name`, `title`, `source:` = the evidence with file:line, `steps`) and show it. **Wait for yes.**
   e. On yes: save it to `<journeys dir>/<repo>/<name>.yaml` and run step 2. If the journeys dir is a git repo,
      offer to commit that one file; commit only on yes.
5. Reply short: the page path, the stages in 3–6 bullets, explain stats (refs kept / dropped / unverified),
   what wasn't verified, fresh or cached.

## Rules
- An entry that no longer resolves (a renamed view) → show the error, propose the fixed entry, ask before editing.
- One `--explain` per flow; it re-explains by itself when the code changes. `--refresh-explain` only on request.
- `<instance>` / `(one of)` / `✂` in a page are honest gaps of the static trace — say so, don't paper over them.
