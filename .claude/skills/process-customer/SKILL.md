---
name: process-customer
description: >
  Trigger: "let's process [account]", "process [account]", "brief me on
  [account]". Builds or refreshes the full account brief before a demo,
  call, or RFP push — and rewrites the ENTIRE Notion Accounts DB row
  (Stage, Priority, Notes, AE, Next call, Deadline, MEDDPICC) from real
  Salesforce/Gong/Slack evidence. The operator's manual notes are not the
  source of truth — this skill's analysis is. Has a hard stop in the
  middle — do not skip it.
---

# process-customer

## What it does

Produces `accounts/<customer>/brief.md`: the single document the operator should
be able to walk into a demo or call with. It also rewrites the account's
entire Notion Accounts DB row from whatever real Gong/Salesforce/Slack
evidence is available — not just the MEDDPICC column, the whole row.
Five phases, in order.

## Step 1 — Self-research (gather what I can find myself)

1. `memory.md` and any existing `accounts/<customer>/` files — don't
   re-derive what's already known, but don't treat old notes as
   authoritative either; they get superseded by this pass's analysis.
2. Notion Accounts DB row for this customer + any linked notes — this is
   what's about to be rewritten, so read the current state fully first.
   "Due date (legacy)" may still be set on old rows; treat it as
   unverified, never as the source of truth, and never write to it (it's
   deprecated — **Next call** / **Deadline** are the real fields).
3. Calendar — past and upcoming events with this customer/account.
4. Slack — search #se-requests, #se-sgm, and any other channel IDs
   the operator has shared, for threads mentioning this account. The
   Salesforce-for-Slack bot's opportunity-notification posts (in
   #se-requests and #sf-new-opps-notification) carry a real **Amount**
   field — capture it as deal-size evidence for the Notion ACV/TCV
   fields even when Salesforce itself isn't checked directly. (2026-08-06,
   an FR retail deal: the bot's Amount matched the operator's own SF
   screenshot exactly, to the euro — so it's a reliable substitute when
   the operator hasn't pasted a CRM extract yet.)
5. **Gong, if a connector is present this session** (discover the real
   tool names/workspace at runtime — don't assume; check `memory.md`'s
   "Local config" section for a workspace name that worked before, but
   confirm since it can be account-dependent). Pull real evidence, but
   never stop at one synthesized/thematic question — thematic answers
   ("what are the pains/criteria") can hide real relationship history.
   **Always also ask for the raw call list** (dates + who attended each
   one) before writing "first meeting" or similar framing anywhere.
   (2026-07-23, real miss: a thematic Gong query never surfaced that the
   AE had already run 4 presentations over 9 months before this cycle —
   an early brief wrongly called it a first-meeting account, only caught
   after a follow-up ask specifically for the call list. Corrected same
   session, but should have been right the first time.)
6. Google Drive — decks, docs, RFP files tied to this account.
7. Notion "📞 Debriefs" database — any past debrief rows related to this
   account (relation property).
8. **Public web research** — the account's own live site(s), corporate
   structure/ownership (parent/subsidiary brands, recent acquisitions),
   industry, and any visible tech-stack signals. Internal tools (Notion/
   Slack/Gong) capture the deal; they rarely capture "who this company
   actually is" — a plain web search fills that gap and is worth doing
   as standard practice, not just when the operator happens to ask for
   it. (2026-07-23: found via web search only after the operator asked
   directly — should have been proactive; also spotted that Google Drive
   already holds real BDR/AE account-research docs for OTHER accounts
   using a "Situation / Compelling Events / Tech Stack & Display Layer
   Info" template — worth checking whether one exists for the account
   being processed before doing this research from scratch.)

## Step 2 — Ask the operator once, then HARD STOP

Before writing anything, post ONE consolidated message to the operator asking
for three concrete things (not a vague "anything you have"):
1. **Salesforce** — opportunity notes, stage/close-date fields, next
   steps, whatever's in the CRM for this account.
2. **Gong** — call extract(s)/summary for this account (transcripts,
   call notes, next steps). If a Gong MCP connector is available this
   session, pull the account's calls directly instead of asking the operator
   to paste them (discover the actual Gong tool names at runtime — don't
   assume). Only ask them to paste if Gong isn't connected or the lookup
   comes back empty/ambiguous.
3. **A Slack channel ID for this specific account/deal**, if one exists
   (separate from the standing #se-requests/#se-sgm scan in Step 1 —
   deal-specific channels often carry the richest thread). If the operator
   doesn't have one or none exists, that's a fine answer — don't block
   on it, just proceed without it.

Also surface every inconsistency found in Step 1 (e.g. Notion date vs.
calendar mismatch, conflicting stage info, stale notes) in the same
message.

**Do not proceed to Step 3 until the operator replies.** This is a hard stop,
not a suggestion — the whole point of this skill is analyzing real
evidence instead of stale manual notes, so it can't run on nothing. (If
The operator pastes the evidence unprompted, before this message even goes out,
that satisfies the hard stop — no need to ask again.)

## Step 3 — Analyze the evidence

Once the operator's reply is in hand, extract everything relevant from the
real evidence (pasted Gong/Salesforce extracts, Slack threads, past
Debriefs DB rows, calendar context, anything the operator said directly).
Never invent signal from tools you can't actually read — Salesforce
isn't connected, and Gong only if its MCP is present this session.
Work only from what was pulled (Gong-if-connected), pasted, or said. For each
Notion field below, either derive a real value with its evidence, or
mark it "no change / need validation" if the evidence doesn't support
one:

- **Stage** — does the evidence show real progression (e.g. "moved to
  proposal," "verbal commit," "lost to <competitor>")?
- **Priority** — does the evidence justify a change (deal size, urgency,
  exec sponsorship signal)?
- **AE** — does the evidence name the actual AE on this deal? Never
  guess from role plausibility. Prefer a direct CRM screenshot/extract
  or the operator's own confirmation over a Slack bot's "Opportunity
  Owner" field alone if the two ever disagree — bot notifications can
  reflect an earlier sourcing/BDR assignment rather than the current
  owner (2026-08-06, an FR retail deal: an opp-creation bot post named one
  person, but the live SF screenshot plus the operator both confirmed a
  different person was the real AE).
- **Next call / Deadline** — the concrete next milestone with a real
  date, labeled explicitly as CALL or DEADLINE.
- **Notes** — a fresh, concise summary of where the deal actually
  stands, replacing the old Notes text (which may be stale/wrong —
  don't just append to it, rewrite it to reflect current reality).
- **MEDDPICC** — all 8 elements (Metrics, Economic Buyer, Decision
  Criteria, Decision Process, Paper Process, Identify Pain, Champion,
  Competition), each tagged:
  - **Confirmed** — specific quotable evidence + source.
  - **Partial** — a signal but incomplete; note what's missing.
  - **Gap** — no evidence either way.

## Step 4 — Write the brief

`accounts/<customer>/brief.md` — **local-only, never committed to git**
(see CLAUDE.md guardrail 6; `.gitignore` already excludes `accounts/*/*.md`).
Structured as:

```
# <Customer> — Account Brief
Last updated: <date>

## Who they are
2-4 sentences a rep could read cold, 30 seconds before a call: what kind
of company, industry, rough scale, and why they're evaluating a CMS
right now. Not a MEDDPICC field — just the "who is this" orientation
that should come before any deal mechanics.

## Snapshot
Type / Stage / Priority / AE / next milestone — labeled explicitly as
CALL or DEADLINE (never just "due date"), reflecting what was just
written to Notion in Step 5, not the pre-analysis state.

## Who's in the meeting
Explicit list for the next call: name, role, confirmed vs. inferred.
If sources disagree on who's attending, say so here directly — don't
bury it in "Need validation" at the bottom where it'll get missed right
before a call.

## MEDDPICC — what wins or loses this deal
All 8 elements, each with its status (Confirmed / Partial / Gap), the
quoted evidence or the specific gap, and the source (Gong extract,
Salesforce extract, Slack thread + link, or "the operator, <date>"). Then,
past the element-by-element list, three short closing paragraphs that
actually say something useful instead of just tagging elements:
- **What wins this deal** — the specific, concrete conditions that get
  this to Closed Won given the evidence so far (e.g. "closing the
  Champion gap by getting X personally invested" or "winning the
  architecture argument with the IT stakeholder").
- **What loses this deal** — the specific active or dormant risks that
  would sink it (a named competitor's strength, an unresolved technical
  objection, a stalled internal alignment issue) — not generic risk
  language.
- **The product's positioning right now** — an honest read: ahead, even, or
  behind, and why, given the competition, decision criteria, and
  champion status above. If it's genuinely unclear, say that too.

## Their priorities
What the customer actually cares about, per stakeholders and past calls.

## Demo — what to show
Concrete, specific use cases to actually demo — not a generic feature
tour. Each item should trace back to a real pain point, decision
criterion, or MEDDPICC gap found in Step 3, with the "why" stated
explicitly (e.g. "AI auto-metadata on upload — directly answers their
under-resourced-editorial-team pain point," not just "show AI features").
If there are multiple upcoming calls/demos at different depths (e.g. a
light discovery-call glimpse vs. a fuller contextualized demo later),
split the list by occasion and keep the near-term one deliberately light
if that's what the evidence calls for — matching the actual call's scope
beats cramming in everything at once. Flag anything that needs data the
team doesn't have yet (e.g. no live client API → use mock/fictional data
and say so, don't silently assume it'll be ready).

## Stakeholders
Names, roles, notes on each (technical buyer, economic buyer, etc.) —
this can overlap with "Who's in the meeting" but covers the full cast,
not just next-call attendees.

## Technical context
Stack, integration points, anything demo/RFP-relevant.

## Risks
Open objections, competitive threats, anything that could stall the deal.

## Next steps
A concrete action checklist, not a vague "verify" list. Should name real
next actions with what they chain into — e.g.:
- [ ] Confirm attendee list for <date>
- [ ] `build-deck` for the <date> call/demo
- [ ] `demo-script` for the same
- [ ] `demo-setup` / product demo-environment script (draft-only if the MCP
      connector is still absent — don't skip drafting just because it
      can't execute yet)
- [ ] Close the biggest MEDDPICC gap: <specific gap>
Tailor the actual items to what this account genuinely needs next —
don't include a deck/script item if there's no upcoming demo to prep for.

## Need validation
Anything Steps 1-3 couldn't confirm that ISN'T already covered above
(attendees have their own section now). Never silently dropped.

## Sources
Where every above section came from (Notion link, Slack thread, calendar
event, the operator's own input, Gong/SF extract) — so staleness is auditable later.
```

## Step 5 — Update Notion and announce

1. **Write every field derived in Step 3 directly to the Notion
   Accounts DB row** — Stage, Priority, Notes, AE, Next call, Deadline,
   MEDDPICC. Per CLAUDE.md guardrail 4, this is a direct edit: no
   proposal step, no waiting for an OK, on any field, including
   downgrading a previously-set MEDDPICC element or overwriting stale
   Notes — the analysis in Step 3 is the new source of truth. The two
   limits that still apply: never write a field Step 3 couldn't
   actually derive from real evidence (leave it as-is / need validation
   rather than guess), and never touch a human-only column.
2. **Lead the chat reply with the "Who they are" company brief** — 2-4
   plain-language sentences on who the account actually is (same content
   as the brief's own "Who they are" section), before any Notion diff.
   Never assume the operator (or whoever's reading the chat that session)
   already holds this in their head — the brief.md having this section
   isn't enough if the chat reply itself jumps straight to field-level
   diffs. (2026-07-21: the first real run skipped this in chat even
   though it was already in the written brief — the operator had to ask "what
   is this company?" after the fact.)
3. **Announce every field that changed** — old value → new value → the
   evidence it's based on, so the operator can spot-check anything that
   surprises them even though no approval gate blocked the write.
4. Update `memory.md` with what changed this session (including the
   full before/after on the Notion row and what it's based on), and add
   an entry to `CHANGELOG.md`.
5. Suggest the logical next step (e.g. "next: demo-script for the July
   21 call" or "the biggest MEDDPICC gap is Economic Buyer — worth
   confirming before the next call").

## Guardrails specific to this skill

- Never fabricate Salesforce/Gong content — if the operator doesn't provide
  it, the field is left as-is (or "need validation" in the brief), and
  the MEDDPICC element stays Gap. Not writing something is always
  safer than guessing it.
- Never mark a MEDDPICC element Confirmed, or change Stage/Priority/AE/
  Next call/Deadline, without a specific, traceable piece of evidence
  and its source in the brief's Sources section.
- Never skip the Step 2 hard stop — the whole row rewrite depends on
  having real evidence in hand first; without it there's nothing to
  analyze.
- Rewriting the whole row means old manual Notes/Stage/Priority values
  are superseded, not preserved — that's the point (The operator's manual
  notes are treated as unreliable by design). Always show the old ->
  new diff when announcing so nothing changes invisibly.
