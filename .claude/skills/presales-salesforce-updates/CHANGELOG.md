# Changelog — Presales: Salesforce Updates

All notable changes to this skill. Newest first.

Semantic versioning:

- **major** — changed process flow, or a new required connector
- **minor** — new steps, gates, or fields
- **patch** — corrections and clarifications

When amending this skill, add an entry here and update `version` in the SKILL.md
frontmatter in the same change.

---

## 1.0.0 — 2026-09-16

First release. Owner: an SE colleague, Solutions Engineering.

**Process flow**

Seven steps: preflight → collect → map → contribution gate → reconcile → draft → review →
write. Steps 3 and 6 are hard gates.

**Scope**

- Writes only to `Solution_Engineer_Notes__c`. MEDDPICC+ component fields are read-only —
  several are half-owned by the AE.
- Scoped to the SE running the skill, resolved at runtime via `getUserInfo`. No hardcoded
  user IDs.

**Validated against live data (9–16 Sep 2026)**

- Gong attributes participation reliably and states non-contribution explicitly. The
  contribution gate at step 3 is built on this behaviour.
- `ask_account` returns calls for the *account*, not for the SE — hence the gate.
- Calendar `list_calendars` requires broader OAuth scope than this skill needs; preflight
  probes with `list_events` instead.
- Gong requires a `workspace` argument. Storyblok has two (`Sales Division Workspace`,
  `Executive Calls Workspace`); both are queried before concluding no calls exist.
- Account name lookups are frequently ambiguous ("Amazon" → 10 CRM accounts, including
  duplicates). Resolved by asking the SE for the `crmId`, never by guessing.
- Empty runs are a valid outcome and are reported with a per-company reason.

**Org constraints recorded**

- `Solution_Engineer_Notes__c` caps at 20,000 chars — under a third of `Manager_Notes__c`
  (65,536). With a prepend-only convention this will eventually fill; the skill stops
  rather than truncating.
- No field named "Technical Criteria" exists, though the SE Playbook (Sept 2026) gates
  Qualification and Proposal exit on it. Closest are `Decision_Criteria__c` and
  `Scorecard_Technical_Decision_Criteria__c`.
- `Technical_Solution_Confirmation__c` is labelled "Demo Prep Completed" and is a picklist
  of demo types, not the checkbox the Playbook describes.
- Field labels do not predict API names — see the reference table in SKILL.md.

**Conventions**

- Entry header `DD/MM II` / `MM/DD II`, locale-ordered, SE initials required (two SEs can
  work one opportunity). Existing field convention wins over locale.
- Prepend-only running log, matching the `Manager_Notes__c` house convention.
- Plain text only — the field renders markdown literally.

**Known gaps at release**

- Steps 4–7 are **not yet validated end-to-end** against a live Salesforce write. The
  `sobject-reads` connector is read-only; a write connector is required and was not
  available at time of release. Until then, step 7 queues and hands over paste-ready text.
- Step 6 persistence depends on an Artifact capability that is verified at runtime;
  falls back to a spreadsheet where unavailable.
