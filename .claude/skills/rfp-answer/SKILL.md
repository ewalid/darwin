---
name: rfp-answer
description: >
  Trigger: "answer this RFP", "help with this RFP question", or
  "harvest this RFP" after a submission closes. Two directions:
  RETRIEVE (match incoming RFP questions against known answers, adapt,
  flag gaps) and HARVEST (fold new/validated answers back into the operator's
  own local library afterward). Primary knowledge source is a real,
  live, company-wide Notion database (see below) — not empty, not
  hypothetical. Product/feature facts for RFPs come from the product's
  official docs and the company knowledge base — never from a
  content-management API (a CMS content connector like the one
  `storyblok-content` uses is unrelated to RFP answers).
---

# rfp-answer

## Status (2026-07-21)

Mechanism drafted, and the primary source is real: there's a live
"RFP Answer Library (POC)" database in the company Notion (its
collection id is in local `memory.md`; ~19 Q&A rows as of 2026-07-21 —
checked directly, not assumed) that someone else on
the team already started. Every row's Status is currently "needs
review," not "approved" — treat accordingly (see "Trust levels"
below). The operator is also going to seed `resources/rfp-library/` with their
own validated RFP files once they have them (their first live RFP deals are
both live "In progress" right now) — that becomes a second,
higher-trust local source layered on top of the shared one.

## Where the answers come from, in trust order

1. **The operator's own local library** — the highest-trust source,
   and **no longer empty (seeded 2026-07-30)**. Two layers:
   - `resources/rfp-library/validated-submissions/` (gitignored) — real
     as-submitted documents. Read the most recent relevant one **before
     drafting anything**. Higher trust than official docs, because it
     carries both the facts and how the operator chooses to frame them
     commercially, already through review.
   - `resources/rfp-library/answers/<category>.md` (tracked) — reusable
     positioning harvested from those submissions, scrubbed of client
     specifics. Includes plan-gating tables and a "standing lessons"
     list in `_index.md` that exists because of real misses; read it.
     As of 2026-09-15 there are 16 category files; five of them
     (`api-limits-webhooks`, `multi-space-governance`, `ai-governance`,
     `observability-telemetry`, `exit-portability`) came out of a single
     architecture-grade RFI and are the ones to reach for on any technical
     or enterprise-architecture evaluation.
1b. **The contract itself — the GTC, the SLA exhibit and the deal's Order
   Form.** A distinct source, not a subset of the docs. Breaking-change
   definitions and notice periods, customer objection windows, termination
   remedies, service-credit bands, overage pricing, and which entitlements
   are actually bought for *this* account all live here and nowhere else.
   Never improvise a contractual number from a marketing page; never quote a
   generous precedent as if it were a committed term (see the
   30-day-vs-6-months example in `answers/api-limits-webhooks.md`).
2. **The product's official public docs** (the operator's product and its
   docs URL are recorded in `memory.md`) — the
   authoritative source for product/feature facts. Ranks above the
   library in #3 (confirmed by the operator, 2026-07-21): docs are official,
   the POC library is not.
3. **The shared company Notion library** ("RFP Answer Library (POC)",
   collection id in local `memory.md`) — **not an
   official resource.** A coworker put it together on their own
   initiative; it's real, usable content (19 rows, genuinely
   well-written) but every row's Status is "needs review," and the operator
   confirmed it has no official standing — if it conflicts with the
   official docs, the docs win. Use it, but always surface the Status
   alongside the answer and never present a "needs review" row as
   equivalent to an approved one. **Read-only** for now — this is
   someone else's resource; writing back to it (marking rows
   "approved," adding new ones) needs an explicit ask from the operator first,
   unlike their own Accounts DB (CLAUDE.md guardrail 4 is about the operator's
   own Notion content, not someone else's unofficial database).
4. **Other sanctioned research, when 1-3 don't cover it** (The operator
   explicitly authorized these, 2026-07-21):
   - **The rest of the company Notion**, starting from the RFP Answer
     Library page as an anchor and branching out via `notion-search`
     — other teams (product, security, RevOps) may have written
     official material that isn't in the docs or the POC library.
   - **Slack search** (not just #se-requests/#se-sgm) — past threads
     where a similar question was actually answered.
   - **Google Drive** — security whitepapers, SOC2/ISO reports,
     one-pagers.
   Anything sourced this way is labeled `[research: <source>, needs
   SME confirmation before submitting]` — never presented as
   equivalent to a validated or official-docs answer.
5. **Nothing else.** If none of the above cover a question, mark it
   `needs SME input` — never fabricated from general CMS-market
   knowledge. This mirrors CLAUDE.md guardrail 5 (never invent
   Salesforce/Gong content) applied to product/security facts.

**A CMS content connector is not relevant here.** A connector like
the one `storyblok-content` uses manages content *inside* a customer's
CMS space — stories, components. It has no bearing on "does the product
support SAML SSO" or "what's the uptime SLA." Those are answered from
the knowledge sources above, never from a content-management API.

## Trust levels — always shown in the draft answer

- `validated` — from the operator's own local library.
- `official-docs` — from the product's official docs. Authoritative; still worth
  a quick sanity check for anything contractual, but not a guess.
- `needs review (unofficial POC)` — from the coworker's shared Notion
  library. Real content, no official standing, status is literally
  "needs review" — flag this explicitly, don't smooth it over, and
  defer to official docs if the two ever disagree.
- `research: <source>` — from wider Notion/Slack/Drive, needs SME
  confirmation before it goes in a real submission.
- `needs SME input` — nothing found anywhere; a real gap, not a guess.

## Why the local library isn't a vector DB

The operator asked directly whether their own library should be a vector
database instead of plain markdown. Decision: no, for this scale —

- **Realistic corpus size.** Even harvested over years, one SE's
  validated answers is realistically dozens to a few hundred entries —
  small enough for category + keyword search with Claude reading full
  matched files for real semantic judgment. A vector index buys
  nothing extra at this size, and the shared Notion library above is
  already a real structured DB for the company-wide content anyway.
- **No persistent infra to run.** This repo's actual persistence is
  git + the filesystem. A markdown library is itself the durable
  store — no embeddings pipeline to keep alive or re-run after edits.
- **Auditability.** Every local answer is a readable file with
  `git log` history — needed for anything that ends up in a
  legally-reviewed RFP submission. A vector store's nearest-neighbor
  match is a worse fit for "show me exactly which answer this came
  from and who approved it."

If the local library ever grows into the thousands of entries, revisit
this — that's a real threshold, not a permanent ruling.

## Steps — RETRIEVE ("answer this RFP")

0. **Two research passes, not one — run BOTH.** This is the single
   biggest lesson from the 2026-07 RFP that a colleague's assistant
   materially out-answered (see "Why that response was better" below).
   - **Requirement-driven** (the obvious pass): answer every numbered
     requirement in the document.
   - **Account-driven** (the pass that gets skipped, and where the deal
     is actually won): ask what makes *this* product uniquely right for
     *this* prospect — things no requirement line item will ever ask for.
     Non-negotiable checks:
     a. **Does our product natively integrate with the prospect's own
        product?** If the prospect is a software vendor, search the app
        directory, changelog and partner pages for *their company name*
        before anything else. On the 2026-07 RFP a shipped first-party
        plugin connecting the prospect's own platform to our editor
        existed, was publicly documented, and no competitor in the
        evaluation had an equivalent — the strongest differentiator
        available, and requirement-driven research walked straight past
        it because it wasn't a line item.
     b. Is the prospect an existing partner/customer/integration in any
        capacity already?
     c. Which case studies match this prospect's *profile* (industry,
        current CMS, locale count, team shape) rather than just being
        impressive?
     d. If the RFP states success metrics, tie capabilities back to each
        metric explicitly.
1. Take the incoming requirement set (from the document itself — see the
   document-production section below for keeping their format). **Match the
   response vocabulary the document itself uses** — it varies, and inventing
   your own categories is a compliance failure: a prose RFP used
   *Supported out of the box / with configuration / requires custom
   development / not supported*, while a requirements grid used a simple
   *Yes / Partial / No* comply column plus a comments field. Read the
   column headers and legend before answering, and if the grid has a
   separate English column alongside another language, fill both.
1b. **Obey the format the question mandates.** Architecture-grade RFIs
   often state a required artefact per question — *"Format: component-level
   responsibility matrix covering configuration, monitoring, incidents,
   capacity, security, recovery and cost"*, *"Format: failure-mode matrix
   showing retained functionality, cache behaviour, recovery steps and
   maximum content staleness"*, *"Format: SLA/SLO table, service-credit
   terms, exclusions and dependencies"*. That is a compliance requirement,
   not a suggestion. Produce the named artefact with **exactly the named
   columns**, and produce a sequence diagram, risk matrix or worked example
   when those are what's asked for. Answering a mandated matrix in prose
   loses points that no amount of substance recovers. This is step 1's
   "match their vocabulary" extended from *wording* to *artefact shape*.
2. For each requirement, check in trust order: the operator's local
   library (validated submissions first, then category answers) → the
   product's official docs → the shared (unofficial) Notion library →
   other sanctioned research → `needs SME input`. If official docs and
   the POC library disagree, go with the docs and note the discrepancy
   rather than silently picking one.
3. **Always resolve the plan tier — read
   `resources/rfp-library/answers/pricing-licensing.md` before answering
   any capability question.** It holds the transcribed public plan matrix:
   real tier names, per-tier quotas, and the roughly one-third of the
   feature set that is Premium+/Elite-only. A capability answer without its
   plan gate is half an answer and creates commercial risk downstream.
   Also check the pricing page's **integration matrix** — it is the
   fastest single check for whether an integration with the prospect's own
   platform exists (see step 0a), and it reveals when a must-have
   integration sets a pricing floor. Re-fetch the live page if the local
   transcription looks stale; the fact-sheet filename is date-stamped.
4. **Before writing "Requires custom development" or calling something a
   gap, find the delivery route on the prospect's actual target stack.**
   A gap named without a route reads as a missing feature and loses points
   on the (usually heavily-weighted) platform-capabilities criterion. Real
   examples from the 2026-07 RFP where a hedged "gap" answer was corrected
   to a concrete path: forms → embed the prospect's existing MAP forms
   directly in the front end (zero CMS dependency) *or* model fields as
   components; non-developer redirects → CMS story as the data store *plus*
   the host's own dashboard-driven redirect config; canonical/hreflang →
   the framework's native metadata API, i.e. framework-native work rather
   than bespoke development. Be honest about what isn't native; be
   specific about how it actually gets delivered.
5. **Don't downgrade a feature's maturity off a changelog entry alone.**
   A "Labs"/changelog mention is a point-in-time signal. Check for a
   product landing page and current plan inclusion before hedging — one
   answer was written as "new, don't over-claim" when the feature was
   already GA and included on two plan tiers.
5b. **Never infer a negative from silence.** If the public sources don't
   mention something about our own company, that is *not* evidence we lack
   it — compliance reports, certification progress, internal roadmap status
   and commercial terms are routinely unpublished on purpose. Write
   "confirm with <team>" and ask the operator; do not write "we do not
   have X" on the basis of an absent web page. (2026-08-03: three official
   pages omitted a certification, a draft concluded we didn't hold it, and
   the operator corrected it — we hold Type I with Type II in progress.
   Understating our own posture in a security review is as damaging as
   overstating it.)
6. **Exhaust the sanctioned sources before writing `needs SME input`.**
   That tag is for genuine gaps, not for facts that are merely
   inconvenient to find. Company headcount, customer/enterprise counts,
   funding, certifications, support-tier response times, roadmap items and
   partnership credentials are all normally obtainable from public pages,
   the trust centre, the public roadmap, or sales-enablement material in
   Drive/Notion — trust-order step 4 exists precisely for this. On the
   2026-07 RFP, eight `needs SME input` flags were written where every
   single one was answerable; they are now recorded in
   `resources/rfp-library/answers/company-credentials.md`.
7. **Pull the live deal context, don't rely on the account brief.** Read
   the account's own Slack channel, recent email and any Gong calls for
   the *current* state before drafting. The 2026-07 RFP turned on facts
   that lived only there — the proposed implementation partner had
   dropped out, and the two teams had agreed to evaluate the platform
   first and select a partner afterwards. A draft written without that
   reads as evasive on the partner question; with it, it reads as aligned.
   The channel ID is in the account's local notes; guardrail 8 (Slack
   outranks Notion) applies.
8. Adapt whatever's found to the prospect's context — never paste verbatim
   if it doesn't actually fit.
9. Return a draft with every answer tagged by trust level so the operator
   knows what's safe to submit as-is versus what needs a second look.

## Architecture-grade RFIs — the additional moves (2026-09, validated)

Some RFIs are written by an enterprise architect, not a procurement team:
they send their own target-architecture diagram, ask for responsibility
boundaries, failure modes, blast radius, telemetry, AI governance and an
exit runbook, and mandate an artefact format per question. That document
type has its own playbook, learned from a validated submission that landed
well.

- **Answer against their drawing, layer by layer, naming their components.**
  Open with a direct verdict — *"[Product] fits the proposed architecture as
  designed"* — then one bullet per layer of *their* stack (frontend
  autonomy, content APIs, SDKs, delivery services, cache invalidation,
  visual editor, third-party integration, identity, managed operations,
  deployment model, API connectivity), closing with **licensing scope** and
  **commercial scope**. Describing our architecture instead makes the
  evaluator do the mapping, and they score what they can see.
- **Produce a component-level responsibility matrix** (us / them / their
  implementation partner / their cloud provider) across configuration,
  monitoring, incidents, capacity, security, recovery and cost. **Name the
  genuinely shared rows** — we sign the webhook, they verify it; we issue
  the preview token, they validate it server-side; they own their IdP
  tenants, we own the single org SSO connection. Shared rows are what make
  the matrix read as honest rather than defensive. Split webhook *emission*
  from webhook *receipt/retry/reconciliation*; collapsing them is where
  responsibility answers go wrong.
- **Never state a gap without its mitigation in the same breath.** "No
  automatic webhook retry" alone is a scoring loss. "No automatic retry, so
  treat the payload as a notification, fetch current state on receipt,
  acknowledge in under a second, and run a scheduled reconciliation job that
  bounds staleness to the poll interval" is an architecture answer. Same for
  no replay protection, no SIEM connector, no tested exit runbook, no
  scheduled unpublish, no deprecation flag. This is step 4 at architecture
  scale.
- **"Not in scope of a CMS" is a legitimate classification** when paired
  with what the platform *does* contribute and who owns the rest. Three of
  six rows in a validated accessibility matrix used it, and it strengthened
  the response.
- **Separate "licensing choice" from "technical limitation" explicitly.**
  Listing SSO, GraphQL, custom roles, unlimited locales and the higher SLA
  tier as *licensing choices already reflected in what's quoted* turns five
  potential gap-scores into a commercial non-issue in one sentence.
- **Correct an outdated premise rather than answering it.** One question
  referenced a scope figure the commercial conversation had already moved
  past; the answer opened with a "Note on scope" before answering.
  Evaluators reuse old drafts — answering a stale premise propagates the
  error into the contract.
- **Label every assumption out loud**: "working assumption, not a formally
  scoped number", "discussed on calls, not signed off", "illustrative
  sizing only, not contractual". Costs nothing, stops a discussion note
  hardening into a commitment, and reads as rigour.
- **Use worked examples**: a worked failure example (timestamped, step by
  step, ending in a design implication), a worked deployment example (a
  paired schema + frontend change promoted region by region), a worked cost
  example (with low/expected/high columns and an explicit "illustrative"
  label). Each converts an assertion into something the evaluator can check.
- **Cross-reference aggressively** ("full architecture in A16", "see A05",
  "same division of responsibility as elsewhere in this response"). Confirms
  standing lesson 7 holds for prose responses, not just grids — and the
  *consistency* of a repeated framing across 30 answers is itself scored.
- **Find the consultative aside, and make it specific.** Where does this
  prospect's stated context collide with a real property of the platform or
  the plan? Examples that landed: an AI feature that sends the image itself
  outside the managed-provider boundary, given their sensitivity about
  imagery; their own timeline putting migration too late, so dry runs should
  start during the build; a load test needing 15 working days' notice ahead
  of their known seasonal peak; URL changes threatening organic rankings
  right before that peak. One sentence each, unasked, directly actionable.

## Why that response was better (2026-07, worth re-reading before each RFP)

A colleague's assistant produced the validated version of an RFP response
Darwin had drafted. It was ~39% longer and better on substance, not
formatting — the structure, co-branding and compliance work carried over
essentially unchanged. What it did that Darwin hadn't:

- Found the first-party integration with the prospect's own product.
- Answered every fact Darwin had punted to `needs SME input`.
- Resolved plan-tier gating on every capability.
- Converted "gap" answers into concrete delivery routes.
- Carried live deal context (partner situation) into the narrative.
- Added consultative judgment the RFP never asked for — e.g. flagging
  that the prospect's own timeline put migration too late and dry runs
  should start during the build phase. Reads as expertise, not scope creep.
- Answered "two contactable references" as a genuinely different ask from
  "three case studies", with a "why this maps to you" paragraph each.
- Closed with a useful-links appendix (docs, roadmap, changelog, FAQ,
  fact sheet, T&Cs) — cheap to add, gives the evaluator somewhere to go.
- Stripped all internal scaffolding (trust tags, draft notices) for the
  final version — those are Darwin's working aids, never the deliverable.

## Steps — producing an actual response document (not just chat text)

Once real answers exist (RETRIEVE done) and the operator wants an actual
file, not just a chat draft:

1. **Default to annotating the prospect's own document in place**, when
   they sent one (a Word/PDF/etc., as opposed to a spreadsheet grid they
   want filled a specific way): keep every original section, heading,
   table, and word of theirs untouched; insert the operator's answers as
   extra columns appended onto their own requirement tables, and as
   clearly-labeled response blocks right after their narrative sections.
   Never rebuild it as a separate from-scratch proposal unless asked —
   confirmed with the operator 2026-07-22 that the annotated-in-place
   version is what's actually wanted, not a restructure.
2. **Ask the operator first, before checking anything** (see CLAUDE.md
   guardrail 11): does this response need to actually follow the
   operator's brand guidelines throughout, or does it just need the
   prospect's own document format kept, with a co-branded logo at most?
   Most RFP responses are the second case (step 1 above) — don't spend
   tokens reading brand assets that turn out not to be needed. Only if
   the answer is "yes, follow them" or a logo is genuinely needed: real
   logo files and a local guideline-doc snapshot live in
   `resources/brand-guidelines/` if the operator has added them — but
   treat that local copy as possibly stale. If a `SOURCES.md`-style note
   in that folder points at a live source (typically a Notion page),
   check that directly rather than the snapshot, especially for fonts —
   Notion tends to hold the current type/token detail that a one-time
   PDF export won't get updated with. If the response document combines
   the operator's own logo with the prospect's (a cover page, for
   example), pull both logos from real sources (the prospect's from
   their own document, the operator's from `resources/brand-guidelines/`)
   and follow whatever co-branding rule the actual guidelines specify
   (safe area, separator style) — never invent a lockup.
3. Keep new table columns/response blocks visually distinct enough to
   spot (a labeled box, or matching the original document's own header
   color for a seamless look — ask the operator which they prefer,
   don't assume) but never touch the original content itself.
4. Every trust-level tag and internal note (open questions, NEEDS SME
   INPUT flags) stays in the draft and gets called out explicitly as
   "delete before sending" — this is a working draft, not the final
   submission.

## Steps — HARVEST ("harvest this RFP")

1. After a submission (or a won/lost outcome), take the final
   operator-approved answers. **Also harvest whenever a validated or
   externally-proofed version comes back** — that version is now the
   highest-trust source and supersedes Darwin's draft, even mid-cycle.
2. Save the document itself to
   `resources/rfp-library/validated-submissions/` as
   `YYYY-MM-DD_<Account>-<Topic>_VALIDATED.<ext>` — **gitignored**, it
   names real clients/stakeholders/partners (guardrail 10).
3. Then extract the *reusable* positioning into
   `resources/rfp-library/answers/<category>.md`, scrubbed of every
   client/stakeholder/deal specific so those files stay safe to track in
   git (guardrail 6). Update `_index.md`. This is the compounding step —
   skipping it means the next RFP relearns everything.
4. **If the validated version differs from Darwin's draft, diff it and
   record why in the category file**, not just the corrected fact. "Plan
   tier was missing" / "a gap was really a delivery route" / "punted to
   needs-SME-input when it was public" are reusable lessons; the fact
   alone is not. `_index.md`'s "standing lessons" list is exactly this.
   Where the diff reveals a process gap rather than a content gap, that's
   a `darwin-improve` trigger — take it.
5. This is local harvesting — separate from the shared Notion library,
   which stays read-only unless the operator says otherwise.
6. If the deal's outcome is known, log why it was won/lost in
   `resources/rfp-library/answers/won-lost-notes.md` — gitignored
   (2026-07-21, guardrail 10), it will name real deals/customers, same
   class of data as `accounts/`.
7. Announce what was added/updated in chat.

## Guardrails

- Never answer a compliance/security-grade question from anything
  other than the sources listed above, in trust order — never from
  general training-data knowledge about CMS products.
- Never present a "needs review" or "research"-sourced answer as
  equivalent to a validated one — the trust-level tag is not optional.
- Never write to the shared company "RFP Answer Library (POC)" DB
  without the operator explicitly asking for that — it's read-only by default.
- `won-lost-notes.md` is customer data — local-only, gitignored, never
  pushed even though this repo is private (same rule as `accounts/`).
- the operator's local category answer files are reusable product-positioning
  content, not tied to one customer's identity — those stay tracked in
  git, unlike `won-lost-notes.md`.
