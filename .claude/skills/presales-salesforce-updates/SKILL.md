---
name: presales-salesforce-updates
description: Draft Storyblok SE Notes for Salesforce opportunities from recorded Gong calls, with an interactive review step before anything is written. Use when an SE or presales engineer needs to update SE Notes, catch up on CRM hygiene, or write up their calls.
version: 1.0.0
---

# Presales — Salesforce Updates

**Version 1.0.0** · see `CHANGELOG.md` · Storyblok Solutions Engineering

Drafts **Solutions Engineer Notes** for Salesforce opportunities from recorded Gong calls.
Scoped to the SE running it. Nothing is written without per-opportunity approval.

---

## Process flow

```
0  PREFLIGHT ........ verify connectors, resolve SE, set lookback window
1  COLLECT .......... Calendar → candidate companies
2  MAP .............. Gong → CRM account → CRM deal → SF opportunity
3  GATE ............. did THIS SE contribute?  ── no ──▶ report, do not draft
4  RECONCILE ........ opportunity team + Assigned SE
5  DRAFT ............ SE Notes entry (prepend, dated, initialled)
6  REVIEW ........... interactive card, SE approves per opportunity
7  WRITE ............ Salesforce ── fail ──▶ queue + scheduled retry
```

Stop conditions are absolute. Steps 3 and 6 are gates: if a gate does not pass, the
opportunity is reported, not written.

---

## Connectors

| Purpose | Connector | Required |
|---|---|---|
| Call list | Google Calendar (`list_events`) | Degrades |
| Call content, CRM mapping | Gong (`ask_deal`, `ask_account`, `generate_brief`) | Yes |
| Opportunity reads | Salesforce (`soqlQuery`, `getObjectSchema`, `getUserInfo`) | Yes |
| Role evidence | Gong `get_transcript` | Optional |
| Field writes | Salesforce write connector | Degrades |
| Retrying writes | Scheduled tasks | Only on failure |

Resolve tools by function name at runtime. MCP server UUIDs are per-person — never copy a
prefixed tool name from these instructions or another SE's session.

---

## Step 0 — Preflight

Run every time. Connectors disconnect and tokens expire between runs.

| Check | Probe | On failure |
|---|---|---|
| Salesforce | `getUserInfo` | **Stop.** Report and exit. |
| Gong | `ask_account` on any known account | **Stop.** Report and exit. |
| Calendar | `list_events`, 1-day window | **Continue.** Ask the SE to name accounts; state the list is manual. |
| SF write | Deferred to step 7 | **Continue.** Queue at step 7. |

Do not use `list_calendars` as the Calendar probe — it requires broader scope than this
skill needs and fails on a correctly-configured connector.

Name any missing connector specifically and state the remedy. Never exit with a generic
error, and never proceed silently to an empty result.

### Resolve the SE

From `getUserInfo`, take `userId`, `displayName`, `email`, `timeZoneIana`, `localeCode`.
Never hardcode a user ID.

Derive **initials** from `displayName` — "Jane Doe" → `JD`.

### Set the lookback window

Prompt for the number of days prior to today. Default 7; always allow an override. Accept
"3", "since Friday", "last 10 days".

Compute in `timeZoneIana`, not UTC. Confirm the resolved window in one line before
proceeding, and state the last window covered if known.

### Resolve the Gong workspace

Gong requires a `workspace` argument when the account has more than one. Storyblok has two:

- `Sales Division Workspace` — deal calls
- `Executive Calls Workspace` — leadership calls

**Query both** before concluding there were no calls. A single-workspace query returns zero
rather than erroring.

---

## Step 1 — Collect candidate companies

Call Calendar `list_events` over the window. Keep events that:

- have ≥1 attendee whose email domain ≠ the SE's domain, **and**
- the SE organised or accepted (skip declined), **and**
- are not internal meetings that happen to include a guest.

Exclude known non-customer domains: `*.greenhouse.io` (recruiting), `regus.com` (rooms),
and partner/agency domains.

Group by external domain → one candidate company each. Carry attendee names forward.

**This is a candidate list, not a list of calls to write up.** The filter is lossy in both
directions:

- Customer-titled meetings with only internal attendees are real deal work but produce no
  recorded customer call.
- AE-organised customer calls may never appear on the SE's calendar.

When Calendar yields few or no candidates, **ask the SE which accounts they worked**. Gong
is the authority on what was recorded; Calendar only seeds the search.

Never source this list from Salesforce ownership — that skips the opportunities whose
records are wrong, which is the reason this skill exists. Do not rely on SF `Event`/`Task`
records; Gong→SF activity sync is not reliably populated.

---

## Step 2 — Map to opportunities

Per candidate company:

1. `ask_account` with the company name and window, `includeSources: true`. Ask which deals
   had call activity.
2. On `CRM_AMBIGUOUS_ENTITY` — the response lists candidates with `crmId` and
   `lastActivity`. **Ask the SE which one.** Do not pick by recency or closest name;
   duplicate accounts with near-identical names are common.
3. Take the CRM deal ID forward to step 3. Prefer IDs over names — `ask_deal` name lookup
   is exact-match.
4. Hydrate from Salesforce once Gong has identified the opportunity:

```sql
SELECT Id, Name, StageName, CloseDate, Amount, Owner.Name, Account.Name,
       Assigned_Developer_Relations__r.Name, Solution_Engineer_Notes__c, LastModifiedDate
FROM Opportunity WHERE Id = '<crmIdFromGong>'
```

Use Gong's CRM mapping, not `Account.Name LIKE` matching — the latter breaks on
subsidiaries and trading names.

**Report, don't guess:**

| Condition | Report as |
|---|---|
| No deal for the account | Unlinked — call may predate opportunity creation |
| Several deals | Ask which one |
| Calendar meeting with no Gong recording | Unrecorded — SE may need to write from memory |

---

## Step 3 — Contribution gate

`ask_account` and `ask_deal` return calls for the account or deal, **not calls this SE was
on**. A calendar invite is not participation.

Per call, ask Gong who attended from the vendor side and what this specific SE contributed.
Then gate:

| Gong response | Action |
|---|---|
| Names the SE with specific contributions | **Pass** — draft at step 5 |
| Names the SE attending, no substantive contribution | **Stop.** Report "attended, no SE contribution recorded" |
| Does not mention the SE | **Stop.** Colleague's call — report whose |
| Vague on who did what | Try `get_transcript` on the call IDs; still unclear → report inconclusive, let the SE decide |

Never draft a note crediting the SE with work Gong does not evidence. Where another SE led,
offer to flag it to them.

Also ask what was discussed, decided and committed to — that content feeds step 5.

`generate_brief` is an alternative where the workspace has published briefs; an invalid
`briefName` returns the available list.

### Empty runs are valid

Report the reason per company and stop. Do not manufacture notes.

Legitimate causes: internal prep/debrief only; SE attended but did not lead; partner or
agency meeting rather than an opportunity; AE-organised call absent from the SE's calendar.

---

## Step 4 — Reconcile opportunity team

```sql
SELECT Id, Name, Assigned_Developer_Relations__c, Assigned_Developer_Relations__r.Name
FROM Opportunity WHERE Id = '<oppId>'

SELECT Id, UserId, User.Name, TeamMemberRole, OpportunityAccessLevel
FROM OpportunityTeamMember WHERE OpportunityId = '<oppId>'
```

| State | Condition | Action |
|---|---|---|
| Aligned | On team as `Solutions Engineer` **and** Assigned SE set | None |
| Partial | One set, not the other | Propose the missing one |
| Wrong role | On team, different or null role | Propose correction |
| Absent | Neither | Propose adding as `Solutions Engineer` |

Read the `TeamMemberRole` picklist at runtime; role definitions change.

Every non-aligned opportunity is **updated or explicitly flagged** — never silently passed
over. Surface it on the step-6 card with Gong call links as evidence.

- Write access → propose, approve at step 6, then write.
- No write access → flag, and offer one bulk RevOps request covering all flagged
  opportunities.

Team changes are always proposals. They affect reporting and credit; Playbook Ch. 0
requires joint AE/SE/manager review before any split or credit decision.

---

## Step 5 — Draft the note

### Target field

Write **only** to `Solution_Engineer_Notes__c`. Never to MEDDPICC+ component fields — several
are half-owned by the AE.

### Prepend, newest first

Read the current value, then write `<new entry>\n\n<existing value>`. Never append, never
replace.

### Entry header

`DD/MM II` or `MM/DD II` — zero-padded, space, SE initials.

| `localeCode` | Format | 16 Sept, Jane Doe |
|---|---|---|
| `en_US` | `MM/DD II` | `09/16 JD` |
| all others | `DD/MM II` | `16/09 JD` |

Initials are required — two SEs can work the same opportunity.

If the field already contains entries with a consistent date convention, **match those**
rather than the SE's locale. Use locale only when the field is empty. State the format used
at step 6.

### Format

Plain text only — this is a plain textarea; `##`, `**bold**` and `|tables|` render
literally.

- One blank line between entries
- `CS:` current status · `NS:` next steps
- Inline labels, one per line: `Risk:` `Signer:` `Procurement:` `Partner:`
  `Path to close:` `Exec Sponsor:` `Stack:` `Calls:`

```
16/09 JD
CS: Completed architecture review with platform team - Bedrock endpoint confirmed feasible
NS: Scoping PoT for DAM integration, session booked 18/09
Stack: AEM 6.5 on-prem, Cloudinary DAM, Adobe Analytics
Technical champion: <name> (Principal Engineer)
Risk: Security model only permits Bedrock, needs custom endpoint - 3 weeks quoted by customer
Calls: 11/09 Architecture deep-dive <gong link>
```

Keep it to a log entry, not a report.

### Content to cover

| Cover | Scope |
|---|---|
| Identified pain | Technically validated |
| Initial use case | Feasibility — buildable, demo-able |
| Technical champion | Named, plus credibility with their peers |
| Technical decision criteria | SE's half only |
| Technical approval steps | Security review, technical sign-off |
| Security / InfoSec status | SOC2, questionnaires |
| Competitive positioning | Technical differentiation |
| Technical urgency | EOL dates, compliance deadlines, incumbent renewal |
| Metrics | M1 from case studies, M2 from BVA/ROI on customer data |

Do **not** cover Economic Buyer or EB Insights.

### Character cap

`Solution_Engineer_Notes__c` caps at 20,000. Check existing length before writing. If the
new entry would exceed it, **stop and tell the SE** — never truncate. Offer to archive the
oldest entries.

### Stage flags

Stages: `Upcoming` `Discovery` `Qualification` `Validate Problem & Impact` `EB Go/No-GO`
`Proposal` `Negotiation` `Finalizing`.

| Condition | Flag |
|---|---|
| Stage = `Qualification` | Urgent — cannot exit without SE Notes |
| Stage = `Validate Problem & Impact` | Heaviest SE load: TFW, BVA, POV |
| Stage ≥ `Proposal` | Expects SE fields complete incl. `Technical_Win_Achieved__c` |
| Stage ≥ `Proposal` and SE work requested | Outside documented model (Appendix 2) — flag back |
| PoC referenced, no manager approval evident | Hard gate — flag |
| PoT referenced in territory account, no approval | Hard gate — flag |

---

## Step 6 — Review card

Publish a private Artifact page. It appears as a card in Claude; the card opens a hosted,
interactive page that persists across sessions. Private to the SE unless they share it.

Load the `artifact-capabilities` skill first and use a persistence capability. **Verify the
capability is available before promising persistence**; if not, fall back to a spreadsheet.

One card per opportunity:

- Opportunity name, stage, close date, AE owner
- Team reconciliation state + proposed fix, individually toggleable
- Draft entry in an **editable** field, exactly as it will be written
- Existing note text read-only beneath, showing what it prepends to
- Gong call links
- Stage flags
- Approve / hold toggle
- Flagged items: unrecorded meetings, unlinked accounts, cap warnings, date format used

Nothing is written until that opportunity is approved.

---

## Step 7 — Write and retry

Check write access before promising it. The `sobject-reads` connector is read-only.

A failed write is **pending, never dropped**:

1. **Persist the queue** — opportunity ID, exact text, approved team change, target field,
   approval timestamp. Store in the step-6 artifact state or a working-folder file that
   survives the session.
2. **Report failures immediately**, with cause. Never report success for a queued item.
3. **Schedule a retry** via scheduled tasks — a few hours for transient errors (auth,
   locks, timeouts); next morning for anything needing an admin.
4. **Re-verify on retry** — re-read the field; if an entry with the same date+initials
   header exists, mark done rather than duplicating.
5. **Escalate after 3 attempts** — stop, tell the SE, hand over paste-ready text.

Distinguish permission and validation errors from transient ones; they will not self-heal.

With no write connector at all, queue everything and provide paste-ready text immediately.

---

## Org field reference

Labels do not predict API names. Resolve via `getObjectSchema` at runtime.

| Label | API name | Type |
|---|---|---|
| Solutions Engineer Notes | `Solution_Engineer_Notes__c` | textarea, 20,000 |
| Technical Win Achieved | `Technical_Win_Achieved__c` | boolean |
| Demo Prep Completed | `Technical_Solution_Confirmation__c` | picklist, demo types |
| Assigned Solution Engineer | `Assigned_Developer_Relations__c` | User reference |
| Manager Notes | `Manager_Notes__c` | textarea, 65,536 |
| Identified Pain | `Top_Priorities_Challenges__c` | textarea |
| Initial Use Case | `Customer_Use_Case__c` | textarea |
| Decision Criteria | `Decision_Criteria__c` | textarea |
| Decision Process | `Mutual_Close_Plan__c` | textarea |
| Paper Process | `Paper_Process__c` | textarea |
| Champion | `Champion__c` / `Champion_Name__c` | reference / text |
| Champion Insights | `Champion_Insights__c` | textarea |
| Economic Buyer | `Contact__c` | reference |
| Economic Buyer Insights | `Economic_Buyer_Description__c` | textarea |
| Compelling Event | `Compelling_Event__c` | textarea |
| Metrics | `Metrics__c` | textarea |

**No "Technical Criteria" field exists**, though the Playbook gates Qualification and
Proposal exit on it. Cover technical criteria in SE Notes and flag the gap.

Opportunity team role picklist: `Solutions Engineer`, `Opportunity Owner`,
`Customer Success Manager`, `Account Executive`.

---

## Guardrails

**Factual only**

- Every statement must trace to something said in a call summary or transcript.
- Never infer pain, metrics, champions, competitors, budgets, timelines, risks or next
  steps to fill a slot. An absent section is correct; an invented one is a fabricated CRM
  record others will act on.
- Attribute: "customer quoted 3 weeks", not "3 weeks".
- "Not discussed" ≠ "no issue". Silence on security is not a clean security review.
- Mark thin claims for SE confirmation rather than smoothing them into confident prose.
- Report inconclusive role attribution as inconclusive.

**Writes**

- Only `Solution_Engineer_Notes__c`. Never MEDDPICC+ component fields.
- Prepend-only. Overwriting destroys history predating the current SE.
- Team changes are proposals, approved per opportunity.
- Never report a write as done unless it succeeded.

---

## Requesting changes

This skill is maintained centrally and versioned. If it does not work for your accounts or
your region, open it in Claude and describe the problem — include:

- **Which step** (0–7) and what you expected
- **What happened** — exact error text, or the wrong output
- **Connector** involved, and whether preflight passed
- **Account or opportunity** it occurred on, if specific

Claude can amend the skill and bump the version. Do not edit your local copy silently —
changes made locally are lost when a new version is distributed, and divergent copies make
team issues hard to diagnose.

Version history is in `CHANGELOG.md`. Semantic versioning: **major** for a changed process
flow or new required connector, **minor** for new steps, gates or fields, **patch** for
corrections and clarifications.
