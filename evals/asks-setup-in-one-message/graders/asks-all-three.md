---
type: llm
---

PASS if the final message asks the user, in that one message, for (1) the backend repo(s), (2) the frontend repo(s)
and (3) optional app folders / app paths to focus on.
FAIL if it asks only one or two of these, splits them into separate turns, or starts tracing without asking.
