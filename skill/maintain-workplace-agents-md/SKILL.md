---
name: maintain-workspace-agents-md
description: |
  Update or audit the workspace-level AGENTS.md and the matching
  docs/repo-inventory.md. Use when the user says "update AGENTS.md",
  "audit AGENTS.md", "new repo in workspace", "we changed conventions",
  "agents.md is stale", "review workspace conventions", or any prompt
  that introduces a new sibling repo, retires one, or changes a
  cross-cutting rule. Designed a flat workspace. The workflow 
  generalises to any workspace folder that has a top-level AGENTS.md
  plus a docs/repo-inventory.md.
---

# maintain-workspace-agents-md

## Purpose

Keep the workspace-level `AGENTS.md` and `docs/repo-inventory.md`
accurate, current, and useful. These files are the front door for every
agent session in the workspace; if they're stale, every session pays
the cost.

The maintenance pattern is **detect drift → propose a diff → let the
user accept**. No silent writes. No bulk rewrites. Small, reviewable
changes only.

## Files this skill owns

- `<workspace-root>/AGENTS.md` — behavioural rules, cross-cutting
  cheatsheet, tooling expectations.
- `<workspace-root>/docs/repo-inventory.md` — the per-repo table:
  what it is, stack, cross-repo touchpoints.

If either file is missing, stop and ask before creating.

## When to use

Trigger phrases:
- "update AGENTS.md" / "audit AGENTS.md" / "AGENTS.md is stale"
- "new repo in workspace" / "we added <repo>"
- "<repo> was removed/archived"
- "we changed how we …" (when "how we" refers to a workspace
  convention)
- "review workspace conventions"
- "repo-inventory is out of date"

Don't use for **per-repo** `AGENTS.md` files — those are owned by the
repo and edited via that repo's normal workflow.

## Workflow

### Step 1 — Locate

```bash
# From the user's cwd, walk up to find the workspace AGENTS.md.
# In this user's setup the workspace root is
#   /Users/user/workspace
# but detect it generically: the workspace root is the first
# ancestor that contains BOTH AGENTS.md AND multiple sibling repos
# (not a single repo).
```

Confirm with the user if ambiguous. Don't assume the cwd is the
workspace root.

### Step 2 — Detect drift

Run these checks. Report results in a short markdown table. Do not
modify anything yet.

**Filesystem vs inventory**:
```bash
# Top-level directories in workspace, excluding hidden + node_modules.
ls -F <workspace-root> | grep '/$' | sed 's|/$||'
```

Then for each directory, check:
- Is it backticked in `AGENTS.md`?
- Is it backticked in `docs/repo-inventory.md`?
- Is it a git repo (look for `.git/`)?
- If a git repo, what's its remote? (`git -C <dir> remote -v`)

Classify each directory:
- **`missing-from-inventory`** — present on disk, not in repo-inventory,
  IS a git repo. Likely needs an inventory row.
- **`missing-from-agents`** — present on disk, not mentioned in
  `AGENTS.md`, IS a git repo. May need a cross-cutting-cheatsheet entry.
- **`inventoried-but-gone`** — listed in inventory, not on disk. Either
  the user removed it (drop the row) or didn't clone it yet (leave but
  flag).
- **`scratch-not-listed`** — present on disk, not a git repo, not in
  AGENTS.md "scratch dirs" list. Add to the scratch dirs list.
- **`ok`** — everything aligns.

**Behavioural-rules check** (lightweight, no auto-detection):
- Ask the user: "Any new ways of working since AGENTS.md was last
  updated? E.g. new MCP server, new subagent, new skill, retired
  convention?"
- Read the latest friction-analysis report if it exists at
  `~/agent-usage-analysis-*.md`. New friction patterns may
  warrant a new rule under "How the agent should behave".

### Step 3 — Propose a diff

Output as a single markdown block listing the proposed changes, grouped
by file. Each change is one of:

- **Add inventory row** (for a `missing-from-inventory` repo):
  ```
  | `<repo>` | <one-line description> | <stack> | <notes / cross-repo arrows> |
  ```
  Source the description from: the repo's README first line, then
  `package.json#description` / `go.mod` package comment, then
  `AGENTS.md` if the repo has one.
- **Remove inventory row** (for `inventoried-but-gone`).
- **Add scratch dir** to the AGENTS.md "scratch / notes" bullet.
- **Add cross-cutting cheatsheet line** (when a new repo plausibly
  belongs in one of the existing axes — checkout, paywall, gateway,
  shared lib, etc.).
- **Add or amend a "How the agent should behave" subsection** (only
  with explicit user direction — these aren't drift, they're decisions).

**Show the diff. Wait for confirmation.** Do not apply.

### Step 4 — Apply

After confirmation, edit the files in place using minimal `edit` calls
(prefer single `oldString`/`newString` swaps over rewrites). Preserve:

- Existing voice (terse, second-person-implicit, lowercase headings
  except where they're already title-case).
- Existing section ordering. Add new bullets to existing lists in
  alphabetical or logical-grouping order, not at the bottom.
- Markdown line-width (~80 chars where possible; not hard-wrapped).
- The "Notes" column convention in `docs/repo-inventory.md`
  (`-> <repo>` for cross-repo arrows, separate sentences for stack
  quirks).

### Step 5 — Verify

After writing:
- Re-read both files end-to-end. Confirm they still parse as markdown
  and that no section was accidentally truncated.
- Count top-level sections in AGENTS.md (should match before + new
  sections added).
- Print the changed file paths and a one-line summary
  ("AGENTS.md: +2 scratch dirs; repo-inventory.md: +1 row for
  `<repo>`").

## Style rules for AGENTS.md

These match the file's existing voice; deviating breaks the document's
scannability.

- **Imperative second person.** "Run X", "Don't Y", not "The agent
  should X".
- **Bold key verbs and rule-of-thumb statements.** Single sentence,
  high-density.
- **List items, not paragraphs**, for rules. Paragraphs only for
  short framing under a heading.
- **Cross-repo arrows** use `->` (ASCII), not `→`.
- **No emoji.** Ever.
- **No "we" or "you" mixed inconsistently** — the document uses both
  but is consistent within a section. Match the surrounding section.
- **Each rule should be testable** — "Never run deploys without
  confirmation" is testable; "Be careful with deploys" is not.

## Style rules for repo-inventory.md

- One row per repo. Columns: `Repo | What it is | Stack | Notes`.
- Sections grouped by purpose (`Checkout / offers / subscription
  funnel`, `Commerce services`, `Gateway / routing / infra`,
  `Front-end / UI / shared libs`, `Salesforce / MuleSoft integrations`,
  `Templates`, `Test / observability / catalogue / utility`).
- New repos go into the section that best matches their purpose; don't
  create new sections without checking with the user.
- "Notes" column: stack quirks, ownership, cross-repo arrows (`->`),
  whether the repo has its own AGENTS.md.

## Anti-patterns

- Auto-applying changes without showing a diff first.
- Rewriting whole sections when a one-line addition would do.
- Adding behavioural rules ("agents should X") without the user
  explicitly asking for the rule. Drift detection covers the
  inventory; behavioural rules are decisions, not drift.
- Adding a `CHANGELOG` / `CHANGES` / `HISTORY` section to AGENTS.md.
  The file is forward-looking guidance; git history covers the audit
  trail.
- Adding emoji or "Last updated: …" headers. Both age the file
  visibly without adding value.
- Editing per-repo AGENTS.md files from this skill. Those belong to
  the repo and are out of scope.
- Bulk-fetching every repo's README/package.json speculatively. Only
  fetch when a row needs writing.
- Inventing cross-repo touchpoints. If the relationship isn't
  evidence-supported (grep, git history, the user's word), don't
  write it.

## Quick reference

| Drift type | Source of truth | Action |
|---|---|---|
| New repo on disk | `ls <workspace>` vs grep | Propose inventory row |
| Inventoried repo gone | grep vs `ls` | Ask: removed or not yet cloned? |
| New scratch dir | `ls` + no `.git/` | Add to "scratch / notes" list |
| Convention changed | User input only | Update "How the agent should behave" |
| New cross-repo coupling | User input + grep | Add line under "Cross-cutting change cheatsheet" |
| Per-repo AGENTS.md drift | Out of scope | Refer user to that repo |

## Periodic audit prompt

When invoked without a specific change, run a full audit:

```
1. Detect filesystem-vs-inventory drift.
2. Ask: "Any conventions changed since the file was last edited?"
3. Read the friction-analysis report if present and propose at most
   ONE new behavioural rule based on it.
4. Show the combined diff.
5. Apply on confirmation.
```

A good cadence is monthly or after any week with >5 new sessions in
the workspace.
