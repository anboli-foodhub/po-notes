---
name: po-notes
description: This skill should be used when the user wants a plain-business, non-technical summary of a JIRA ticket's change — suitable for a product owner or anyone non-technical on the thread — drafted from the ticket and/or the actual code diff, then optionally posted as a ticket comment after approval. Invoked as /po-notes <TICKET-ID>.
disable-model-invocation: true
---

# PO Notes Skill

Produce a short, non-technical summary of what a ticket's change means for the product — written for a PO (and readable by anyone else non-technical on the thread) — then post it to the ticket after the user approves.

## Setup (do this once before first use)

This skill calls JIRA through MCP tools. Depending on what you have configured:

- **A custom JIRA MCP server** (e.g. your own `jira-analyser`-style tool) — used first if available, by ticket ID lookup.
- **Atlassian's official MCP connector** (`mcp__claude_ai_Atlassian_Rovo_*`) as a fallback — this needs a `cloudId`. If you don't know yours, call `getAccessibleAtlassianResources` once and reuse the returned `cloudId` everywhere below instead of hardcoding one.

Replace `<YOUR_ATLASSIAN_CLOUD_ID>` in Step 2 and Step 7 below with your own, or wire up the equivalent call to fetch it dynamically.

## How to invoke

```
/po-notes PROJ-1234
/po-notes PROJ-1234 --template=.claude/skills/po-notes/template.md
```

The `--template` flag is optional. When omitted the skill resolves the template in this order:
1. `.claude/skills/po-notes/template.md` (project-level override)
2. `~/.claude/skills/po-notes/template.md` (user-level default — this file's sibling)

This is the **pluggable** hook: drop a `template.md` inside any project's `.claude/skills/po-notes/` to tailor the section set per project without touching this skill.

---

## Step-by-step execution

### 1 — Parse input
Extract the TICKET-ID from the first argument. If none is supplied, ask for it and stop.

### 2 — Fetch the ticket (with fallback)
Try your custom JIRA MCP first, if one is configured:
```
mcp__jira-analyser__fetch_jira_ticket  { "ticket_id": "<TICKET-ID>" }
```
**If that returns "not found" or isn't configured**, fall back to the Atlassian connector:
```
mcp__claude_ai_Atlassian_Rovo_2__getJiraIssue  { "cloudId": "<YOUR_ATLASSIAN_CLOUD_ID>", "issueIdOrKey": "<TICKET-ID>", "fields": ["summary","description","issuetype","status"] }
```
Extract: title, description, acceptance criteria, ticket type.

### 3 — Gather the actual change (if available)
If the current repo has commits for this ticket on the branch (branch name or commit messages containing the TICKET-ID), read:
```bash
git log main..HEAD --oneline --no-merges
git diff main..HEAD --stat
```
and read the key changed files to understand the real behaviour change. If there is no matching branch/diff in the current working directory (e.g. the ticket isn't implemented here, or this is being run from an unrelated repo), fall back to the ticket's description/acceptance criteria alone — do not block on missing code.

### 4 — Load the template
Read the resolved `template.md`. It carries the placeholder reference and the drafting rules (no file/function/table names, no internal jargon, 1-3 sentences per section) — follow those rules exactly regardless of how technical the underlying change is.

### 5 — Draft the content
Translate the change into the template's sections, in plain business language:
- **What's changing** — the user/business-visible behaviour, not the mechanism.
- **Why** — the problem or impact being addressed.
- **Rollout** — flag default state / phased rollout / "ships directly", whichever is true.
- **If it fails** — what breaks or doesn't, and the fallback.
- **Traceability** — how someone (support, QA, the PO) can confirm it happened.
- **Scope** — what's included and explicitly excluded, in plain terms.

If a detail genuinely isn't knowable from the ticket or the diff (e.g. no rollout mechanism exists), say that plainly rather than inventing one.

### 6 — Show draft and ask for approval
Display the full draft in a markdown block.

Then ask:
> "Does this look good? Reply **yes** to post it to {TICKET-ID}, **edit** to modify, or **no** to cancel."

### 7 — Handle approval
- **yes** → post the comment. Try your custom JIRA MCP first (e.g. `mcp__jira-analyser__add_jira_comment`); if it errors or isn't configured, fall back to `mcp__claude_ai_Atlassian_Rovo_2__addCommentToJiraIssue` with `cloudId: "<YOUR_ATLASSIAN_CLOUD_ID>"`, `issueIdOrKey: "<TICKET-ID>"`, `contentFormat: "markdown"`. Confirm success either way.
- **edit** → apply the user's corrections, show the revised draft, go back to step 6.
- **no** → abort with "Notes discarded. Nothing was posted to JIRA."

---

## Rules

- Never post to JIRA without explicit approval in step 6.
- No file names, function/class names, table or column names, repo names, or code in the output — ever. If a draft section accidentally contains one, rewrite it before showing the user (e.g. don't say "updated `OrderProcessor.php`'s `calculate_discount_total`" — say "updated how discounts are calculated for orders").
- Keep each section to 1-3 sentences — this is a scannable status update, not a design doc or QA test plan.
- If the ticket can't be fetched from either source, say so and ask the user to paste the title/description instead of guessing.
