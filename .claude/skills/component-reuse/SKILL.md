---
name: component-reuse
description: >
  Trigger: at the end of every custom demo build (custom-storyblok-demo,
  storyblok-content with new blocks), during a darwin-improve retrospective,
  or on request: "what can we reuse", "track reusable components",
  "should this go in the SE team repo", "plan the reusable [X]". Spots
  components built for one prospect that recur across demos, records them in
  a local registry, and writes a generalisation plan for each. PLAN ONLY:
  never implements, never opens a PR, never pushes or merges to the SE team
  repo — the operator does that, and only the operator.
---

# component-reuse

## What it does

Every custom demo ends up rebuilding the same capabilities under a new brand
(a product grid on a commerce feed, a sticky editorial scroll story, a
store locator, a scheduling badge…). This skill makes that visible:

1. **Scan** what the latest build created (new block schemas + their
   frontend components) and compare with previous demo repos.
2. **Record** each reusable capability in the local registry
   `resources/component-reuse/registry.md` (git-ignored: it names the real
   demos a component came from).
3. **Plan** how it would be generalised: for Darwin's own demo kit, or as a
   contribution candidate for the SE team's shared starter/template repo
   (which repo that is lives in local `memory.md`, never in this file).

It never writes the generalised component. Planning is the deliverable.

## Hard rules (never break these)

- **Never create a pull request, push a branch, merge, or open an issue on
  the SE team repo (or its starter/template repos) on your own initiative.**
  Only the operator does that. This holds even if a plan is "ready", even if
  the operator said "go ahead" about something else earlier, and even if a
  tool makes it one command away. If the operator explicitly asks for a PR,
  prepare the branch/diff locally and hand it over; the operator opens and
  merges it.
- **Plan, don't implement.** No generalised component code, no schema
  changes to the team template, no edits in the team repo's working tree.
  Implementation happens only when the operator explicitly asks for it, as a
  separate task.
- **Tracked files stay name-free** (CLAUDE.md guardrail 6): this SKILL.md
  describes capabilities generically. Real demo, account and repo names go
  only in the git-ignored registry.
- Don't duplicate what the team repo already has: before proposing a
  candidate, check the team repo's current components/skills (read-only) and
  mark "already exists upstream" instead of re-planning it.

## Steps

1. **Collect evidence** (read-only):
   - New/changed block schemas in the demo space(s) (MAPI list components with
     `fields`), and the matching frontend files in the demo repo(s).
   - Previous demo repos (the operator's accounts folder) for the same
     capability under another name.
   - The team starter/template repo, to see what already exists upstream.
2. **Score each candidate** in the registry:
   - **Recurrence**: how many demos needed it (1 = watch, 2 = candidate,
     3+ = strong candidate).
   - **Coupling**: what is prospect-specific (brand colours, copy, data
     source, hardcoded labels) vs generic.
   - **Dependencies**: commerce connector (real or mocked), external libs
     (map, animation), datasources, field plugins, shared wrappers
     (style/mobile/planning tabs).
   - **Target**: Darwin demo kit, or SE team template candidate.
3. **Write the plan** per candidate, in the registry:
   - Generic name + one-line purpose.
   - Proposed schema (fields, tabs, nested blocks), with what becomes a
     setting instead of hardcoded.
   - Data contract (e.g. a commerce adapter interface: `list(query)`,
     `get(sku)`, with Shopify / mock-JSON / PIM implementations).
   - Source files to start from (paths), known bugs/lessons from each demo.
   - Effort estimate (S/M/L) and open questions for the operator.
   - Status: `watch` → `candidate` → `planned` → `approved by operator` →
     `implemented (by operator's request)` → `contributed (by operator)`.
4. **Report** in chat: the new/updated candidates, their status, and the
   one or two worth doing next. Ask nothing to be merged.

## Registry format (`resources/component-reuse/registry.md`)

```
## <Generic capability name>  — status: <status>
- Purpose: …
- Seen in: <demo> (<repo path>/<file>), <demo> (…)   ← local-only names
- Generic vs specific: …
- Dependencies: …
- Target: Darwin demo kit | SE team template candidate
- Plan: schema … / data contract … / start from … / effort …
- Open questions: …
- Log: <date> — <what changed>
```

## Guardrails

- Read-only on every repo except Darwin's own registry file.
- No PR / push / merge / issue on the SE team repo — ever, unless the
  operator does it themselves.
- The registry is the source of truth; update entries in place, newest log
  line first, rather than appending duplicates.
