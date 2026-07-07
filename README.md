# GitHub API File-Append Workaround (n8n)

An n8n workflow that solves a small but annoying problem: **GitHub's Contents API has no "append" operation.** Every file update is a full overwrite — so appending a single line to a file means read → decode → modify → write, every time.

This project also (for fun) drives that same commit mechanism on a schedule to draw a wave pattern into the GitHub contribution graph.

<img width="888" height="534" alt="image" src="https://github.com/user-attachments/assets/d7d839d4-0b32-44b5-8e2c-a45876891bd9" />

## The problem

GitHub's `PUT /repos/{owner}/{repo}/contents/{path}` endpoint (which n8n's GitHub node wraps) always requires:
- the **full file content**, base64-encoded
- the current file's `sha`, to avoid overwriting someone else's concurrent change

There's no diff/patch endpoint for file contents — a commit always points at a complete blob. So "append a line" has to be built manually.

## The solution

**Read → Decode → Append → Write**, using three nodes:

1. **Get a file** (GitHub node, resource: File, operation: Get)
   Pulls the current file. Returns the content as **raw base64** in `content`.

2. **Edit Fields** (Set node)
   Decodes the base64 and appends the new line:
   ```
   {{ $json.content.base64Decode().replace(/\n+$/, '') + "\n" + "your new line here" + "\n" }}
   ```
   n8n expressions include `.base64Decode()` / `.base64Encode()` string helpers — no need to reach for `Buffer` or `atob`, which aren't available in the expression sandbox anyway.

3. **Edit a file1** (GitHub node, resource: File, operation: Edit)
   Writes the combined content back as **plain text** — the node handles the base64 encoding on commit automatically, so no manual re-encode step is needed on the way out.

### Gotcha worth noting

It's easy to accidentally decode the same string twice (e.g. if you cache a decoded value in one field and then call `.base64Decode()` on it again downstream) — decoding plain text as if it were base64 silently produces garbled bytes instead of throwing an error. Keep exactly one decode call between the raw `content` field and the final write.

## Bonus: painting the contribution graph

The same append-and-commit mechanism, run on a daily Schedule Trigger gated by a sine-wave check in a Code node, causes commits to land only on specific days — tracing a wave shape across the contribution graph over time instead of a normal every-day pattern.

```
Schedule Trigger (daily) → Code node (wave gate) → Get a file → Edit Fields → Edit a file1
```

The Code node computes which day-of-week should be "on" for the current week based on a sine function, and returns `[]` (stopping the workflow) on days that don't match.

## Setup

- GitHub credentials (Personal Access Token with repo write access) configured in n8n
- Target repo + file path set in both GitHub nodes
- Adjust the wave parameters (`periodWeeks`, `amplitude`, `centerRow`, `invert`) in the Code node to taste

## Why this exists

Mostly as a small, self-contained example of working around an API limitation cleanly, plus a slightly ridiculous use of the same mechanism to draw on a contribution graph.
