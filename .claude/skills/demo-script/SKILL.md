---
name: demo-script
description: >
  Trigger: "script the [account] demo", "write a demo script for [X]".
  Produces the talk track the operator actually speaks during a demo — chains
  off `accounts/<customer>/brief.md` and the deck built by `build-deck`.
---

# demo-script

## What it does

The deck (`build-deck`) has blank "Demo" slides by design — this skill
writes what the operator actually says and does at each one: Tell-Show-Tell
per station, anticipated objections pulled from `resources/battle-cards/`,
and a pre-call verify checklist. Output lives in
`accounts/<customer>/demo-script-<date>.md`, not on the slides themselves.
Always in English regardless of the deck's language — this is an
internal working doc, not customer-facing (style-guide.md's FR/EN rule:
decks match the customer's language, internal notes/briefs stay English).

## Prerequisites

- `accounts/<customer>/brief.md` (from `process-customer`) — the
  script is critiqued against this account's actual priorities and
  MEDDPICC gaps, not generic product talking points.
- Ideally the deck itself (from `build-deck`), so the script's demo
  stations match the deck's actual station names/order/slide numbers
  exactly — if no deck exists yet, ask whether to build one first or
  script against a station list the operator gives directly.

## Steps

1. **Read the brief and the deck's demo stations** (confirm station
   names/order/slide numbers from the actual built deck — don't assume
   they match whatever the brief implied, since naming can change
   during `build-deck`). If no deck exists yet, ask the operator for the
   station list directly.

2. **Draft the outline first**, before writing full prose: for each
   station, the Tell (opening)/Show/Tell (closing) angle and which
   specific pain point or MEDDPICC element it answers.

3. **Confirm the outline with the operator before writing the final doc.**
   This is a judgment call about framing and emphasis (what to lead
   with, what to downplay) — not a factual lookup, so it needs a real
   checkpoint, not an assumption. (Learned 2026-07-20, first real FR run: a
   first version was written end-to-end with no chance to adjust
   framing before it was finalized.)

4. **Write the full script** once confirmed. **Default format: terse
   bullets and sub-bullets, no prose paragraphs** — the operator reads it
   live, mid-demo (stated 2026-09-30). Timings per station, clicks as
   sub-bullets, one-line tells.
   - **Tell (opening)**: 2-3 sentences setting up the problem/context —
     tied to something specific from the brief (a stated pain point, a
     constraint from discovery), not a generic product pitch.
   - **Show**: the concrete sequence of actions the operator will actually
     perform in the product — numbered steps, not vague ("open the Visual
     Editor, drag in a Teaser component, publish").
   - **Tell (close)**: the "so what" — the specific business outcome
     this ties back to for this account (their KPI/pain, not a generic
     benefit).

5. **Anticipated objections** — check `resources/battle-cards/` for
   this account's known competitors/objection patterns. If
   battle-cards is empty or has nothing relevant, say so rather than
   inventing objections; ask the operator what they've actually heard from this
   account so far (Salesforce/Gong context they'd need to paste in
   manually — never invented).

6. **Verify-before-call checklist** — concrete things to check live
   before the call, not assumed: demo environment data is current, any
   account-specific config still working, whether stakeholders
   confirmed for the call match who the script assumes is in the room,
   any open dependencies (unconfirmed date, unlocked attendee list,
   mock-vs-live data caveats to remember to say out loud).

7. **Save** to `accounts/<customer>/demo-script-<date>.md`:
   ```
   # <Account> — Demo Script — <date>
   Deck: <link/filename from build-deck, if any>

   ## Station 1: <name>
   Tell (open): ...
   Show: 1. ... 2. ... 3. ...
   Tell (close): ...

   ## Anticipated objections
   - <objection> — <response>, source: battle-cards/<file> or "the operator,
     <date>" if given directly, or "need validation" if neither.

   ## Verify before the call
   - [ ] ...
   ```

8. **Suggest the logical next step**: run through it once with the operator
   before the call, or flag if `resources/battle-cards/` is thin and
   could use populating.

## Guardrails

- Never invent objections or competitor claims not sourced from
  battle-cards or something the operator directly told you — label the
  source on every objection, or mark "need validation."
- Never fabricate Salesforce/Gong-derived context (CLAUDE.md guardrail
  5) — if the operator hasn't pasted it in, don't assume it.
- Always confirm the outline (step 3) before finalizing — this is the
  one skill in the demo-prep chain that's pure judgment/framing, not a
  factual writeup, so it gets a real checkpoint every time.
- This is customer-facing prep material, not a deliverable sent to the
  customer — but should still follow style-guide.md tone since it may
  shape what the operator actually says live.
