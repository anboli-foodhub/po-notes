# po-notes

A Claude Code skill that turns a JIRA ticket's technical change into a short, plain-business summary a product owner (or anyone non-technical on the thread) can read in 30 seconds — then posts it as a ticket comment once you approve it.

## Why

Engineers are good at describing *what changed in the code*. POs need *what changed for the user/business*, with no file names, function names, or jargon. This skill drafts that translation from the ticket plus the actual code diff (when one exists), using a template you fully control, and never posts anything without your explicit yes.

## Install

```
/plugin marketplace add anboli-foodhub/po-notes
/plugin install po-notes@anboli-po-notes
```

Or clone and install locally:

```bash
git clone git@github.com:anboli-foodhub/po-notes.git
```
```
/plugin marketplace add /path/to/po-notes
/plugin install po-notes@anboli-po-notes
```

## Setup

This skill talks to JIRA via MCP tools. Before first use:

1. If you have a custom JIRA MCP server, it's tried first automatically — no setup needed beyond having it configured.
2. Otherwise it falls back to Atlassian's official MCP connector, which needs a `cloudId`. Open `skills/po-notes/SKILL.md` and replace `<YOUR_ATLASSIAN_CLOUD_ID>` (it appears twice) with your own — call `getAccessibleAtlassianResources` once if you don't know it.

## Usage

```
/po-notes PROJ-1234
```

The skill is marked user-only (`disable-model-invocation: true`) — Claude won't trigger it on its own, since it posts a comment to a real ticket.

## Example

```
/po-notes PROJ-1234
```

1. The skill fetches ticket `PROJ-1234` and, if a matching branch/diff exists in the current repo, reads the real changes.
2. It drafts a six-section summary:

   > **What's changing:** Customers who abandon checkout now get a reminder email 2 hours later.
   > **Why:** Reduces cart abandonment without needing manual follow-up.
   > **Rollout:** Ships behind a feature flag, off by default — enabled per-region after this release.
   > **If it fails:** No reminder is sent; checkout itself is unaffected either way.
   > **Traceability:** Each reminder is logged and visible in the customer's activity history.
   > **Scope:** Covers abandoned-checkout reminders only — does not cover cart-recovery discounts.

3. It shows you the draft and asks: *"Does this look good? Reply **yes** to post it to PROJ-1234, **edit** to modify, or **no** to cancel."*
4. On **yes**, it posts the comment to the ticket and confirms.

## Customizing the template per project

Copy `skills/po-notes/template.md` to `<your-project>/.claude/skills/po-notes/template.md` and edit the sections — add, remove, or rename them to match your team's conventions. The skill picks up the project-level copy automatically; no changes to `SKILL.md` needed.

## License

MIT
