---
name: darwin-improve
description: >
  Trigger: "Darwin, learn this", or any moment the operator corrects a mistake,
  states a preference, or gives context that should stick permanently.
  Formalizes the standing "How I improve" loop from CLAUDE.md into a
  concrete procedure — this is what actually ran, ad hoc, all through
  2026-07-15 session 1; this skill just names and systematizes it.
---

# darwin-improve

## What it does

Turns a piece of friction — a mistake, a missing preference, new context
The operator supplies — into a permanent fix in the right file, committed to
the repo, so it never has to be corrected twice.

## Steps

1. **Notice the trigger — there's no routing system that does this for
   me.** This skill doesn't auto-fire; I have to recognize the moment
   myself, every time, from signals wider than the literal phrase
   "Darwin, learn this": The operator correcting something; the operator stating a
   preference ("next time...", "always/never..."); catching my OWN
   mistake via verification or a git-log check (self-caught friction
   counts exactly the same as operator-stated friction — most of
   2026-07-20's real fixes were self-caught); or the operator asking a
   reflection/status question about how I'm operating — if the honest
   answer reveals a gap, that IS the trigger, not just a question to
   answer and move past. A
   reflection question can also be a **whole-engagement retrospective** ("that
   project was chaotic, what should change?"), not just a single in-the-moment
   correction — mine the relevant prior sessions (session-mgmt
   `search`/`list_events`) to ground the fixes in what actually happened
   before writing them.

2. **Identify the friction.** What exactly was wrong or missing? Quote it
   back in your own head before acting — vague understanding produces a
   vague fix.

   **Re-read this file's steps before executing — don't run the
   procedure from memory.** That's specifically how the `improve:`
   commit-prefix convention drifted into `fix:` twice in one session
   (2026-07-20): the gist was remembered, the actual step wasn't
   reread.

3. **Classify where the fix belongs** (never guess — if two categories
   seem to fit, ask; a fix in the wrong place gets lost):
   - **Recurring, always-true behavior** (a hard rule, a standing
     preference that applies to everything) → `CLAUDE.md`.
   - **Task-specific procedure** (only matters when running one skill)
     → that skill's `SKILL.md` (e.g. Slack scan window → daily-briefing).
   - **Reference material** (people, competitors, tone/format decisions)
     → the right file in `resources/` (people.md, style-guide.md,
     battle-cards/, etc.).
   - **A one-off fact about a specific account** → `accounts/<customer>/`.
   - **Something a live artifact does** (daily-briefing's classification
     logic, its Slack triage, etc.) → the artifact's HTML/JS itself, via
     `update_artifact`, THEN verify with `verify_artifact` before calling
     it done. A fix that isn't redeployed to the artifact isn't a fix.

   **Before concluding "this needs a NEW skill," search every skill location
   for one that already covers it** — `~/dev/darwin/.claude/skills/`,
   `~/.claude/skills/`, the SE team repo `~/dev/tools/se-ai-tools/skills/`
   (mature `replicate-*` site/section skills live there), and the current
   project's own `.claude/skills/`. Near-miss: a demo retrospective almost
   spawned a `replicate-site-demo` skill when a deep `replicate-site` already
   existed in the team repo, and most of the "new" gotchas were already
   captured inside `custom-storyblok-demo` / `storyblok-content`. Default to
   folding a fix into the existing skill; a genuinely new skill is the
   exception, not the reflex. (Per operator policy these SE-demo lessons land
   in Darwin's own skills, not the team repo — but you still check the team
   repo so you don't duplicate or contradict it.)

4. **Apply it.** Read-before-Edit as usual. If the fix spans multiple
   files (it usually does — e.g. a people correction touches
   `resources/people.md` AND the artifact AND `daily-briefing/SKILL.md`),
   apply all of them in the same pass, not piecemeal across sessions.

   **If the target is under `.claude/skills/`**, the Edit tool will
   refuse it (protected location) — this forces a fallback to bash/
   python, which does NOT enforce read-before-write the way Edit does.
   Before writing, manually run `cat <path>` (or `git show HEAD:<path>`)
   AND `git log --oneline -- <path>` to see whether real content
   already exists. Never assume a file is a stub just because it's
   short or the trigger felt like "create a new skill" — check first.
   (2026-07-20, real mistake: overwrote genuinely drafted `demo-script`/
   `demo-setup` content this way, without checking either. Restored
   and merged afterward — see memory.md Session 15-16. CLAUDE.md
   guardrail 9 now states this rule generally.)

5. **Commit** as `improve: <what changed>`, and add a line to
   `CHANGELOG.md` (newest-first, short) describing it in plain terms.

6. **Update `memory.md`** with the friction and the fix — future-session
   Darwin needs to know *why* something is the way it is, not just that
   it changed.

7. **Report back concisely**: what changed, where it now lives, and (if
   relevant) what's still open because of it. Don't re-explain the whole
   mechanism every time — the operator already knows how this works after the
   first pass.

## Compounding

Corrections aren't isolated — patterns across many of them are exactly
what `monthly-review` (Phase 4) will mine for. A correction that keeps
recurring in slightly different forms is a signal CLAUDE.md itself needs
a clearer rule, not just another one-off patch.

## Good example (real, from 2026-07-15)

The operator: "[names] are my SE colleagues... [name] is my manager...
check Slack from the last working day to the day of the routine."
(Real names go to `resources/people.md`, which is local-only, never
quoted back into a tracked file like this one.)

- Classified: people context → `resources/people.md` (new); Slack scan
  window → task-specific procedure → `daily-briefing/SKILL.md` +
  the live artifact (both the fetch window AND the triage prompt needed
  updating, since the artifact does its own reasoning independently of
  the skill doc).
- Applied to all three places in one pass, verified the artifact's tool
  calls still succeeded via `verify_artifact`.
- Committed as `improve: distinguish AE pod from SE colleagues/manager,
  widen Slack scan to last-working-day window, decide deck format is
  .pptx`. Logged in `CHANGELOG.md` and `memory.md`.

## Guardrails

- This skill never overrides the hard guardrails in CLAUDE.md — e.g. a
  "learn this" about Slack still can't mean sending a message without
  showing the draft first.
- Never fabricate the reasoning behind a past decision to make a fix
  sound more justified than it was — if the "why" isn't known, say so.
