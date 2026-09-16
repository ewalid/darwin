---
name: demo-setup
description: >
  Trigger: "set up the demo for [account]", "prep the demo environment
  for [X]". Produces the concrete demo-environment setup script for a
  demo — spaces/folders, components, front-end wiring, permissions,
  data caveats. That script IS the deliverable, full stop, not a
  placeholder for something else. If the operator's product has a content/config MCP connector
  (e.g. Storyblok CMS — see `storyblok-content`), steps 6-9 below
  describe how the same script could be executed directly instead of run by hand — a bonus, not the point.
---

# demo-setup

## What it does

Turns a deck's demo stations into a concrete, written setup script —
spaces/folders, components (with fields), front-end wiring, permissions,
data caveats — that the operator runs by hand ahead of a demo. This is a
planning/writing skill, like `demo-script`; it does not require a live
product connector to do its job.

The concrete vocabulary below (spaces, folders, components, a
content-delivery API) assumes a headless-CMS product — which is the
common case for the operator's product (read it from `memory.md`).
For a different kind of product, keep the *shape* of the script
(what each demo station must show, wired end to end) and swap the
CMS-specific nouns for that product's equivalents.

## Prerequisites

- The account's deck (`build-deck` output) and `accounts/<customer>/brief.md`
  — the script is driven by what the deck's stations actually need to
  show, not a generic space template.

## Steps

1. **Read the deck's stations and the brief's "Demo — what to show"
   section.** Every setup item should trace back to something a
   station actually needs to demonstrate — don't add generic product
   features that aren't part of this account's planned demo.

2. **Decide space structure** (one shared space vs. space-per-brand/
   entity, folder structure, shared vs. duplicated component library).
   **Flag this decision to the operator before finalizing** if the account has
   more than one brand/site involved — this is a structural call that
   affects how convincingly the demo proves a Decision Criterion (e.g.
   "single-environment multi-site management"), not just a formatting
   detail. (Learned 2026-07-20, first real FR run.)

3. **List components needed**, each tied to a specific station/pain
   point — fields, not just names, where that's known from the brief.

4. **Front-end wiring — and which skill actually builds it.** Flag
   explicitly whether the stations need:
   - **Content-only:** the shared Solutions Demo Environment frontend
     already renders every block; execution is `storyblok-content`
     (rebrand, folders, pages from existing components). No repo clone.
   - **Custom frontend (looks-like-the-prospect):** clone a private
     copy of the SE showcase template
     (`https://github.com/storyblok/storyblok-demo-showcase-se`), own
     it as `<customer>-storyblok-demo`, deploy, then author
     prospect-shaped pages on top. Execution is `custom-storyblok-demo`
     Path A (that skill does the clone/own/deploy; this skill only
     decides and scripts *what* the stations need). Default Path A
     starter lives in local `memory.md` — confirm it there if the
     operator's default has moved.
   Don't bury this in a wiring note. The operator's "custom demo" often
   means only the first case. Plus: what must be connected (e.g. a
   decoupled front-end via the Content Delivery API) for an
   architecture-focused station to work live, and a note to test the
   live-update loop before the call.

5. **Permissions/governance and AI features** — anything the demo needs
   to show on-screen to back up a talking point (e.g. a role/permission
   split for a security argument), not just discuss verbally.

6. **Flag data caveats explicitly** — mock/fictional data, no live
   client API, no finalized client UX/UI — as checklist items the operator
   verbally caveats during the demo, not hides.

7. **Save** to `accounts/<customer>/demo-setup.md` (local-only, per
   guardrail 6). This is the complete script — the operator runs it by hand.

8. **Report back**: what the script covers, station by station, and
   anything flagged as needing their confirmation or judgment.

## If the product has a content/config MCP connector (e.g. Storyblok)

The same script becomes directly executable instead of a manual
checklist — but the script itself doesn't change shape, only who runs
steps 3-6. If asked to execute directly:
- **Dry-run preview before any write** — produce a preview of exactly
  what will be created/changed and show it to the operator before touching
  the space. This is CLAUDE.md guardrail 3, not optional.
- **Get explicit confirmation before writing.** Production spaces get
  a second, separate confirmation — never treat a first OK as covering
  both.
- **Execute nothing beyond what was previewed**, then verify live
  (don't just trust the write succeeded — confirm the actual content
  is there and correct, and test the live-update loop specifically for
  any architecture-focused station).
- Never improvise this through browser automation or a guessed API
  call if the connector turns out to only be partially there — a
  misconfigured demo space is worse than none.

## Guardrails

- Never write to a live product environment (e.g. a Storyblok space)
  without a preview + explicit OK
  (CLAUDE.md guardrail 3) — no exceptions, no matter how small the
  change. This only applies if/when direct execution is ever attempted;
  it has no bearing on writing the script itself.
- **Never invent a space structure without checking whether the account
  has a multi-brand/multi-site Decision Criterion the structure itself
  needs to prove (step 2's confirmation gate exists for this).
- **Never skip the content-only vs custom-frontend flag in step 4.**
  Treating every "custom demo" as a repo clone wastes a cycle; treating
  a missing renderer as a CMS write fails silently in the editor.
- Never treat mock/fictional data as if it were live — always carried
  through as an explicit caveat, both here and in `demo-script`.
