# Darwin — Solutions Engineer AI Assistant
<table>
<tr>
<td width="60%" valign="top">

**Darwin** — a personal AI assistant for a Solutions Engineer, built on Claude
Cowork. It runs the operational layer around a sales cycle — briefings,
account intelligence, demo prep, RFP answers, Notion hygiene, call coaching —
so the SE can spend that time selling and engineering, not assembling status
updates by hand.

It also runs on its own cadence, not just on request: a **daily briefing**
every morning, a **weekly review** of the whole pipeline, and a **monthly
self-audit** that fixes recurring friction at the root instead of patching it
twice. Underneath that cadence sit the skills doing the real work: building
account briefs from real evidence across calls, Slack, Notion, and email;
turning those briefs into brand-consistent decks and talk tracks; answering
RFPs against a trust-tagged answer library; coaching every call into concrete
feedback; and keeping a live deals dashboard current by reading Slack threads
directly instead of trusting stale CRM fields.


It also improves itself over time — when something breaks, the fix
gets written back into its own skills instead of being patched once and
forgotten.

Darwin is the sweetest cat ever, he's a very good boy.

A seasoned Solution Engineer.

</td>
<td width="40%">

<img src="resources/darwin.jpg" alt="Darwin">

</td>
</tr>
</table>

## Start here
- **[SKILLS.md](SKILLS.md)** — what Darwin can do: every skill, when it fires,
  what it needs. Read this first.
- **[CLAUDE.md](CLAUDE.md)** — Darwin's identity and hard guardrails (always
  loaded).
- **[HANDOFF.md](HANDOFF.md)** — design history and rationale.
- **[ROADMAP.md](ROADMAP.md)** — phased build plan and skill roster.
- **[CHANGELOG.md](CHANGELOG.md)** — what changed and when.

## Using Darwin on your own machine/account
On a fresh clone there's no `memory.md`, so Darwin runs **`darwin-setup`**
automatically the first time — it introduces itself and interviews you to make
the repo yours (identity, team, connectors, Notion/Slack IDs). Everything is
tuned to you from there.

## Privacy
This repo contains **zero real client or account names** and is safe to share.
Customer evidence, per-account files, rolling memory, and private coaching
notes are all git-ignored and stay local to each person's machine. See
`CLAUDE.md` guardrail 6.
