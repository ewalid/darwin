# Darwin — Skills Reference

Darwin is a personal AI assistant for a Solutions Engineer, built on Claude
Cowork. It runs the operational layer around a sales cycle — briefings,
account intelligence, demo prep, RFP answers, CRM/Notion hygiene, call
coaching — so the SE can focus on selling and engineering.

This doc explains every skill: when it fires, what it does, and what it needs.
It's the starting point if you're a colleague picking Darwin up.

> **New to Darwin? Start here.** The very first time you run Darwin on a fresh
> clone, it detects there's no local memory yet and runs **`darwin-setup`**
> automatically — it introduces itself and interviews you to make the repo
> yours (your name/role, your company, the product you sell, your team,
> your connectors, your Notion/Slack IDs).
> Nothing below is tuned to you until that runs. See `darwin-setup` at the end.

## How skills work

Each skill is a folder under `.claude/skills/<name>/SKILL.md`. A skill is a
focused playbook with a clear trigger, one job, and built-in guardrails. Darwin
picks the right one from what you ask — you don't invoke them by name (though
you can). The hard rules every skill obeys live in `CLAUDE.md`; design
rationale is in `HANDOFF.md`; rolling state is in `memory.md` (local-only).

Connectors used across the skills: Google Calendar, Gmail, Slack, Notion,
Google Drive, GitHub, and — if connected — Gong. Salesforce and Gong content
is pasted in when those aren't connected; Darwin never invents CRM/call content.

---

## Daily & weekly rhythm

### `daily-briefing`
**Trigger:** "morning brief", "daily brief", or the start of a session with no
other ask. **Does:** one scannable snapshot of the day — demos first, then
deadlines (today, then this week), Slack (needs-reply + worth-claiming),
relevant email, todos, and a one-liner. Read-only; pulls Calendar + Notion +
Gmail + both Slack channels over the last-working-day window. **Also runs**
as a scheduled task each weekday morning.

### `weekly-review`
**Trigger:** "weekly review", "how's the pipeline looking", or Monday cadence.
**Does:** a pipeline/account-health pulse across all live accounts — what
moved, what's stalling, what's due. Plain chat text, never an artifact.
Distinct from `monthly-review` (which audits Darwin itself, not the pipeline).

### `todo-sync`
**Trigger:** "sync my todos", or as a step inside a briefing/review when a new
personal action item surfaces. **Does:** creates and checks off items on the
one real "✅ To Do" checklist on your Notion space page. Personal action items
only — account-specific next-steps stay on the account rows, never here.

---

## Account intelligence & the deal pipeline

### `process-customer`
**Trigger:** "process [account]", "brief me on [account]". **Does:** builds or
refreshes the full account brief before a demo, call, or RFP push — and
rewrites the entire Notion Accounts DB row (Stage, Priority, Notes, AE, Next
call, Deadline, MEDDPICC) from real evidence (Slack, Drive, Gong-if-connected,
pasted Salesforce/Gong extracts). It asks once for the evidence it needs, then
hard-stops until you provide it — it never analyzes a guess. The full brief
lives in a local-only account file; the chat reply leads with a plain-language
"who is this company" before any field-level diff.

### The deals dashboard (`walid-deals-dashboard` — a live artifact, not a skill)
A self-contained Cowork page you re-open anytime; it pulls fresh data on load.
Three views: **Your Pipeline** (your Notion deals, as a Kanban by Stage), **Team
Pipeline** (last-7-day #se-requests claimed by your SE peers, grouped by who
claimed each), and **Worth Claiming** (unclaimed requests from your AE pod).
Each card links straight to its Slack thread so you read the real context
before acting. It never sends Slack messages or acts on your behalf.

---

## Call handling

### `post-call-update`
**Trigger:** "debrief [account]", or right after a call. **Does:** one debrief
in, four places updated out — a new row in the Notion Debriefs DB, the Accounts
DB row (Stage / Next call / Deadline / MEDDPICC), the account's local debrief
file, and memory. If Gong MCP is connected it can pull the transcript directly;
otherwise you paste it or just tell Darwin what happened. Either way it
confirms the outcome with you before writing a Stage change. Never fabricates
outcomes.

### `call-coach`
**Trigger:** "coach me on [account] call", "how'd I do?". **Does:** turns a call
transcript into concrete, quotable feedback — 3-5 things done well, 3-5 to
improve (each tied to an actual line), and one focus for next time. Pulls the
transcript from Gong if connected, otherwise from what you paste. Feeds a
private coaching log that compounds over time and is never surfaced anywhere
customer-facing.

---

## Demo preparation

### `build-deck`
**Trigger:** "build a deck for [account]". **Does:** produces a `.pptx` by
adapting the closest-fitting real example deck to the account's brief — it
never invents slide types outside the proven template, and demo slides stay
blank (that's the live demo, not scripted content). Confirms the demo-station
shortlist with you before building. Needs an account brief from
`process-customer` first.

### `demo-script`
**Trigger:** "script the [account] demo". **Does:** the talk track you actually
speak — Tell-Show-Tell per demo station, with likely objections and responses.
Chains off the brief and the deck. Confirms the outline/framing before
finalizing.

### `demo-setup`
**Trigger:** "set up the demo for [account]". **Does:** the concrete
demo-environment setup script — for a headless-CMS product that's
spaces/folders, components, front-end wiring, permissions, data caveats;
for a different product, its equivalents. The written script is the
deliverable. If the product exposes a connector or API it can also execute
it (dry-run → your OK → write → verify), but that's a bonus, not a dependency.

---

## Storyblok demo work — pick the right skill first

Two different jobs. The word "custom" is not enough to choose.

| They need | Skill |
|---|---|
| A **customer-specific frontend** — clone/scaffold a repo, change how blocks render, deploy | `custom-storyblok-demo` |
| **CMS content only** — rebrand, folders, site-config, pages from existing blocks, experiment charts, same starter frontend | `storyblok-content` |
| A **written setup script** before either of the above | `demo-setup` |

If the frontend repo already exists and they want more Vue/CSS, that is
work in that repo — not a re-run of the standup skill, and not a CMS
write. Content cannot invent a renderer the frontend does not have.

---

## Heavy artillery

### `rfp-answer`
**Trigger:** "answer this RFP", "help with this RFP question", or "harvest this
RFP" after a submission closes. **Does:** two directions — RETRIEVE (match
incoming questions against known answers, adapt, flag gaps) and HARVEST (fold
new validated answers back into a local library). Every answer carries an
explicit trust tag (validated / needs-review / research / needs-SME-input) —
never smoothed over, because a wrong compliance claim in a submission is a
real problem. Official product docs outrank any unofficial answer library.
*Status: mechanism ready; the validated library is still being seeded.*

### `custom-storyblok-demo` *(opt-in — Storyblok CMS only)*
Also tied to Storyblok specifically — ignore it unless your company uses
Storyblok. **Trigger:** they need a **customer-specific frontend**
("custom site", clone the template, a layout the starter cannot draw).
The word "custom" alone is not enough — rebranding an existing space is
`storyblok-content`. **Does:** Path A duplicates a starter template repo
(from local memory); Path B scaffolds via Storyblok CLI. Then both: own a
private repo, apply template fixes, deploy to Vercel, wire preview URLs.
Ask frontend-vs-content before Path A/B. Never creates a would-be-blank
CMS space (asks the operator).

### `storyblok-content` *(opt-in — Storyblok CMS only)*
Darwin is otherwise product-agnostic; this is the one skill tied to a
specific product. Ignore it unless your company uses **Storyblok** as its
CMS. **Trigger:** "set up the Storyblok space", create stories/components,
rebrand/restructure/populate. **Does:** CMS writes only (MCP or Management
API), dry-run → OK → write → verify. Never clones a frontend or deploys
Vercel. If they need a renderer the starter does not have, route to
`custom-storyblok-demo` or the already-cloned demo repo.

### `component-reuse`
**Trigger:** the end of every custom demo build, a `darwin-improve`
retrospective, or "what can we reuse / should this go in the SE team
repo". **Does:** spots components that recur across demos, records them in
a local, git-ignored registry, and writes a generalisation plan per
candidate (schema, data contract, starting files, effort). **Plan only:**
it never implements, and it never opens a PR, pushes or merges on the SE
team repo — only the operator does that.

---

## Darwin maintaining itself

### `darwin-improve`
**Trigger:** you correct Darwin, state a preference ("always/never…"), or Darwin
catches its own mistake. **Does:** turns that friction into a permanent fix in
the right place (a hard rule → `CLAUDE.md`; a procedure → the relevant skill;
reference material → `resources/`), commits it as `improve: …`, and logs it.
So nothing has to be corrected twice.

### `monthly-review`
**Trigger:** "run a monthly review", "how are you doing overall", or monthly
cadence. **Does:** mines memory and the changelog for *recurring* patterns
across corrections — a friction that keeps reappearing signals a rule that
needs to change, not another one-off patch — and runs `darwin-improve` on what
it finds. Audits Darwin's own behavior, not the pipeline.

### `darwin-setup`
**Trigger:** automatically on first run in a repo with no `memory.md` (a new
computer, account, or clone); also "set up Darwin", "onboard me". **Does:**
introduces Darwin, then interviews you to rebuild the repo for *you* — your
identity/role, your team structure (it doesn't assume the sales-engineering
shape), which connectors are actually live (it tests them), where your real
data lives (your Notion DB, Slack channel IDs — not the previous owner's),
which guardrails to keep, and your voice preferences. It then rewrites the
identity/config files from scratch rather than inheriting someone else's.

---

## Privacy & sharing note

This repo is safe to share with colleagues: it contains **zero real client or
account names**. All customer evidence, per-account files, the rolling memory,
and private coaching notes are git-ignored and stay local to each person's
machine. `darwin-setup` regenerates the personal files fresh for whoever runs
it. See `CLAUDE.md` guardrail 6 for the full rule.
