# PO Notes Template
#
# This is the DEFAULT template used by /po-notes.
#
# HOW TO CUSTOMISE PER PROJECT:
#   Copy this file to <your-project>/.claude/skills/po-notes/template.md
#   Edit the sections below — the skill will pick up the project-level copy automatically.
#   You can add, remove, or rename any section to match your team's conventions.
#
# PLACEHOLDER REFERENCE:
#   {{WHATS_CHANGING}}   — one or two sentences, plain business language, no file/function/table names
#   {{WHY}}              — the problem/impact this solves, in user- or business-facing terms
#   {{ROLLOUT}}          — how it ships: flag/toggle default state, phased rollout, or "ships directly" if no flag
#   {{IF_IT_FAILS}}      — what happens on failure, whether anything is blocked or lost, and the fallback
#   {{TRACEABILITY}}     — how someone can confirm it happened (audit log, notification, dashboard, report)
#   {{SCOPE}}            — what this does and does NOT cover, named in plain terms
#
# RULES FOR FILLING THESE IN (read before drafting):
#   - No file names, function names, class names, table/column names, repo names, or code snippets.
#     A PO should never see "OrderProcessor.php" or "calculate_discount_total" — describe the
#     user-visible behaviour and business outcome only.
#   - No internal jargon (cron, Lambda, endpoint, API token) unless there is truly no plain-English
#     substitute — prefer "a background process" over "a cron", "the system" over "the service".
#   - Each section is 1-3 sentences. This is a status update a PO reads in 30 seconds, not a design doc.
#   - If a section genuinely doesn't apply (e.g. no rollout flag was used), say so plainly — "Ships
#     directly with this release, no toggle." — don't leave it blank or pad it.

---

**What's changing:** {{WHATS_CHANGING}}

**Why:** {{WHY}}

**Rollout:** {{ROLLOUT}}

**If it fails:** {{IF_IT_FAILS}}

**Traceability:** {{TRACEABILITY}}

**Scope:** {{SCOPE}}
