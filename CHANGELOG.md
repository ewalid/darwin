# CHANGELOG

All notable changes Darwin makes to itself, this repo, or the operator's connected
tools (Notion, scheduled tasks, artifacts) are logged here — newest first.
Session-level narrative detail lives in `memory.md` (local-only, git-ignored);
this file is the short, skimmable, **name-free** "what changed and when" record
that is safe to share with colleagues. Update it alongside any `improve:` commit.

- improve: custom-storyblok-demo — added audit 4d: components you build yourself must
  be tested across EVERY option/variant they expose (layout × media_type, etc.) on the
  deployed surface, not just the one path you tried; and consult prior demo repos for a
  proven pattern before inventing one. (From a full-bg-hero video/YouTube miss.)

- **process-customer:** Ran on a new inbound French retail account (~EUR120k, RFP stage, replacing a legacy enterprise CMS) — first full run; created the Notion Accounts DB row from scratch (Type/Stage/Priority/AE/dates/MEDDPICC/Notes) and wrote its brief. Same AE-conflict pattern as the 2026-08-06 case recurred: a Salesforce-bot Slack post named one person as Opportunity Owner (outbound sourcing) while a different person is operationally running the deal — left AE as Unassigned/need validation pending the operator's confirmation rather than guessing. Also note: the account's own name is a common noun in its language, which produced false-positive CRM matches during lookup — worth a quick sanity check on ambiguous account names generally.
- **improve:** Folded the custom-frontend build's lessons into the existing Storyblok skills — no new skill (the CMS-only/custom-frontend split already exists, and a mature `replicate-site` lives in the SE team repo). `custom-storyblok-demo`: derive the locale list dynamically so a CMS-added locale doesn't 404, and lock the i18n/tree model before building (routing couples to it — it was the biggest time sink); build field plugins via the official CLI, never hand-inject `body`/`compiled_body` via MAPI (silent non-render), keep a datasource `options` fallback; one demo = one repo/space/deploy (a demo once ran as two overlapping builds). `storyblok-content`: folder-per-locale needs space `use_translated_stories` OFF (else `?language=` 404s); `with_slug` returns the folder, not its startpage. `darwin-improve`: search every skill location (incl. the team repo's `replicate-*`) before creating a new skill; whole-engagement retrospectives are a valid trigger.
- **improve:** Storyblok demo work splits cleanly: CMS-only (folders, Dimensions, i18n, stories — `storyblok-content`, MCP first) vs a custom frontend (`custom-storyblok-demo`). `storyblok-content` now has an architecture-before-write pass (Dimensions = top-level folders only; no auto-translate on create; CMS schema does not sync from git). `demo-setup` asks that fork before listing Vercel/repo steps. No new skill — the two already existed; routing was the gap.
- **improve:** daily-briefing gains a read-only "calls to debrief" detection step (step 5) + output section, plus the manual "sweep yesterday's calls" trigger — surfaces last-working-day Gong calls with no matching Debriefs-DB row and OFFERS to run post-call-update per call (detection only; never pulls, summarizes, or writes — those stay inside post-call-update behind operator confirmation). Kept out of the brief's write path so daily-briefing stays read-only by design.
- **improve:** custom-storyblok-demo step 4 gains a third component audit — asset & interactive rendering: image focus point must drive `object-position` (never hardcode it), images need a defined wrapper height (`h-full` alone collapses to ~80px), and dynamic `:is` needs a resolved component not a string (`:is="'NuxtLink'"` renders a dead tag).

Convention (guardrail 6): this file is tracked in git and MUST contain zero real
client/account names, deal figures, or real colleague/operator names — accounts
are referred to generically ("an RFP deal", "an FR demo account") and people as
roles ("the operator", "an AE", "an SE colleague"). Real names live only in
local, git-ignored files (`memory.md`, `resources/people.md`). (Tightened
2026-07-23 when the repo went fully agnostic — previously colleague names were
allowed here.)

## 2026-08-06 (process-customer: new inbound retail demo account; improve: AE-conflict + Slack-bot Amount evidence)

- Ran `process-customer` on a new inbound EMEA retail account (~EUR41.7k
  ARR, early Discovery stage) — first full run for this account. Notion
  Accounts DB row created from scratch (Type, Stage, Priority, Owner,
  AE, Next call, Deadline, MEDDPICC, Notes); the AE select list needed a
  new option added via ALTER COLUMN (same known-issue class as
  guardrail 8's history).
- Real friction: a Salesforce-bot Slack notification named one person as
  "Opportunity Owner" at deal-creation time, but the live Salesforce
  record and the operator both confirmed a different person is the
  actual AE — bot notifications can reflect an earlier sourcing/BDR
  assignment rather than the current owner. `process-customer/SKILL.md`
  Step 3's AE guidance now says to prefer a direct CRM record or the
  operator's own confirmation over a bot's "Opportunity Owner" field
  alone when the two disagree.
- Also generalized Step 1's Slack scan: the Salesforce-bot's
  opportunity-notification posts carry a real Amount field, usable as
  ACV/TCV evidence even without checking Salesforce directly — this
  session the bot's figure matched a real SF screenshot exactly.
- Full brief written to the account's local-only file; MEDDPICC
  currently Confirmed only on Decision Criteria/Identify Pain/
  Competition, with Economic Buyer and Champion still open gaps.

## 2026-08-06 (improve: todo-sync becomes dynamic — suggests removals too, not just additions)

- todo-sync previously only appended new personal to-do items and checked
  off completed ones; it never removed a stale/no-longer-relevant item,
  only flagged it in chat. The operator asked for the routine to
  dynamically suggest items to add OR remove, then actually update the
  list in the same pass.
- `todo-sync/SKILL.md`: added a real removal path (step 4) alongside
  check-off — a line is deleted only when there is a stated reason
  (cancelled, superseded, handled elsewhere), always announced in the
  same output as the write. Guardrails updated to allow justified
  removals while still forbidding guesses.
- `daily-briefing/SKILL.md`: Step 5 (todo sync) now explicitly runs
  live inside the briefing, not just as a mention; TODOS output section
  gained Added:/Removed: lines.
- A recurring scheduled brief (weekdays morning) was set up to run this
  routine automatically going forward.

## 2026-08-04 (process-customer: a partner-sourced account, reopened after stalling)

- Ran a full `process-customer` pass on an account with no prior Notion row.
  The operator's reply to the Step-2 hard stop wasn't Salesforce detail but a
  key strategic fact instead (the deal had previously stalled over a partner
  alignment question and has just reopened) - treated as satisfying the hard
  stop and proceeded on that basis rather than re-asking.
- Hit the known missing-AE-select-option issue again (guardrail 8's
  documented class of bug): added the option via `notion-update-data-source`
  before the page write would succeed.
- **Caught own mistake before announcing**: first pass wrote all
  evidence-bearing MEDDPICC elements (Confirmed + Partial) into the Notion
  multi-select, which contradicts this file's own 2026-07-23/08-04 written
  convention (Confirmed only; Partial/Gap detail stays in the local brief).
  Corrected before reporting out. Generalizing: check this file for standing
  conventions on a skill's *output shape* before the first write, not just
  for facts about the account.

## 2026-08-04 (process-customer: a large, well-funded retail account)

- Ran a full `process-customer` pass on a retail account whose Notion row
  had been created from a Slack-evidence sweep only (no Gong/Salesforce
  reviewed yet). Gong was connected this session and, combined with
  public web research, resolved several open "need validation" items
  from that earlier pass (stakeholder roles, and a copy-paste anomaly in
  an AE's template email that turned out to be nothing). Rewrote the
  whole Notion row (guardrail 4) and the local account brief.
- Re-applied the MEDDPICC multi-select convention noted here on
  2026-07-23: tag only Confirmed elements in the Notion property,
  Partial/Gap detail stays in the local brief only.
- **Caught and fixed a path-convention mix-up**: `device_bash`'s working
  directory (the VM sandbox root) is not the same as the real device
  path used by `device_stage_files`/`device_commit_files` — a `~`-relative
  `mkdir` inside `device_bash` created a stray, harmless directory
  outside the actual mounted repo instead of inside it. Caught before
  anything was written there; cleaned up safely since it never touched
  a real user file. Worth remembering for any future local file writes
  through the device bridge.

## 2026-08-04

- Added a local-only `resources/case-study-database.md` to the knowledge base:
  the company's marketing-owned case study database (URL, data source id,
  property schema, views) plus the internal six-section win case study video
  format. Gitignored **at creation**, before any content went in — guardrail 10
  applied proactively rather than reactively for the first time.
- Ran a full cross-source evidence sweep on a won deal (Gong, Gmail, Slack,
  Salesforce-for-Slack, the demo deck, local account notes) and produced a
  bullet-form internal win case study video outline in that account's
  local-only folder. Two unevidenced facts left as explicit VALIDATE flags
  rather than filled in.
- Learned: Gong's `ask_deal`/`ask_account` require the `workspace` parameter
  on accounts with more than one workspace — the first call fails with
  `WORKSPACE_NOT_FOUND` and lists the valid names.
- Learned: the Storyblok Management API is unreachable from a cloud sandbox
  session (connection failure, not auth) — `storyblok-content` needs a session
  running on the operator's own machine.

## 2026-08-04 (RFP library: second validated automotive/AEM-migration harvest)
- `resources/rfp-library/answers/` (8 category files): harvested a second
  validated RFP submission (automotive, large-scale AEM migration). Added a
  standing lesson that the AWS-certificate-list misattribution error has now
  recurred twice — treat it as a contaminated paragraph in source material,
  not a one-off. Corrected a stale "no dedicated/private-cloud option" claim
  (a real BYOC option exists). Proposed a reconciliation of the 14- vs
  30-day backup-retention conflict (two different mechanisms, not a
  contradiction) — flagged as inference pending confirmation, not settled.
  Added migration methodology detail, AI-credit token mechanics, AWS
  Marketplace procurement route, GDPR processor/controller framing, and
  named onboarding packages.
- One case-study reference tightened to remove framing that sat too close
  to the source submission's real prospect identity, per guardrail 6.

## 2026-07-30 (first Closed Won recorded; new win/revenue tracking + live impact dashboard)
- First deal reached Closed Won. Recorded it end-to-end from real sources only:
  the Salesforce closed-won Slack notification for the booked annual figure, and
  Gong (1 call + 23 emails) for contract term and total contract value. Salesforce
  still isn't directly connected; the SF-to-Slack bot is the usable substitute for
  closed-won facts and is worth treating as a standing source.
- Notion Accounts DB gained seven properties so money and outcome are trackable at
  all: `Outcome` (Won/Lost/Open/No decision), `ACV (EUR)`, `TCV (EUR)`,
  `Contract years`, `Closed date`, `Partner`, `Competition beaten`. Every existing
  row got an explicit Outcome so win-rate has a real denominator instead of an
  inferred one.
- New Notion database **🏆 Wins** (one row per Closed Won deal, full anatomy: role,
  partner, competition beaten, why-we-won, milestone, SF opp reference) plus a
  **📈 My Impact** summary page — revenue per year, TCV, win rate, open pipeline,
  and a written note on how the figures are sourced.
- New Cowork artifact **impact-dashboard**: revenue-per-year bar chart (annual vs
  total contract value), win-rate donut, wins detail, pipeline by stage, all live
  from the two Notion databases on every open. It also derives its own data-integrity
  flags rather than trusting the rows: annual×term ≠ total-contract-value mismatches,
  deadlines inside 21 days, and next-call dates that have gone past. Verified live —
  both Notion queries succeed and return the expected shapes.
- **Notion API limitation confirmed and worth not rediscovering:** `FORMULA(...)`
  columns using `repeat()` are rejected with a bare "Type error with formula",
  including with no string concatenation and against a paren-free property name.
  This is the same class of gap as the chart-DSL failure logged on day one — Notion
  visuals over the API are limited to callouts, tables and pre-rendered bar text;
  anything genuinely charted belongs in an artifact. Two attempts, then stopped
  guessing rather than burning calls on syntax roulette.
- Honesty notes carried into the deliverables rather than smoothed over: a real
  ~2% discrepancy between annual-figure×term and the stated total contract value is
  flagged in the win record, on the summary page, and automatically by the dashboard;
  a 100% win rate off a single close is labelled as the thin number it is; and open
  deals with no sourced amount read "need validation" instead of carrying an estimate.

## 2026-07-29 (improve: Storyblok schema semantics that break the editor, and a standing verify-the-real-surface rule)
- Audited two long demo-build sessions on request. The dominant finding was one
  repeating failure, not a list of bugs: content written through the API renders
  perfectly on the page and is quietly broken in the CMS — an empty story
  picker, a component absent from the "+" menu, a field the schema never
  declared, a link type that makes the page unsaveable, a facet that had never
  returned a result. Across the two builds the operator raised it as "I can't
  do this myself in the CMS" four separate times, each after the rendered site
  had been checked and reported as working.
- `storyblok-content` (135 → 238 lines): new section documenting twelve of
  these semantics with the editor-side symptom stated first, since that is how
  each will be recognised next time — reference fields needing `use_uuid`,
  reference fields returning arrays at `max_options: 1`, `component_group_uuid`
  vs `component_group_whitelist`, multilink `id` vs `cached_url`, query-string
  destinations having to stay url-type, content holding fields the schema never
  declared, `resolve_relations` needing both the fetch and the bridge,
  `default_value` applying only to new stories, `translatable` being per-field
  and never on a bloks field, asset fields needing `id` + `meta_data` for the
  focal-point picker, the Experiments results-push endpoint, and deciding
  draft-vs-published deliberately. Plus a new step 7 (open it in the editor and
  confirm it is editable — a re-fetch cannot see any of the above) and two
  guardrails: never accept a proxy for the surface a claim is about, and author
  through the path that will be demoed or say plainly that the two write paths
  collide.
- `custom-storyblok-demo`: one existing bullet extended to name the inverse
  case and point at the above, rather than restating it.
- `CLAUDE.md`: new guardrail 12 — verify on the surface the operator will
  actually use. Promoted to a standing rule rather than another skill patch
  because the same correction recurred in several different forms, which is the
  compounding signal `darwin-improve` describes.
- Also noted, not fixed: the richest Storyblok gotchas file on the machine
  lives in a skill that is in no git repo, and already documented four of the
  behaviours above — including an API endpoint that got rediscovered by probing
  during this build. Nothing routes a lesson between that skill and Darwin.

## 2026-07-27 (improve: custom-storyblok-demo gains a component audit + API mechanics, after a full demo build)
- Built an entire branded multi-site demo on the pipeline (folder per brand,
  themed site-config, homepage rebuilt against a live reference site, campaign
  page, product template, collection-driven and hand-picked product blocks).
  Folded the reusable lessons back into the skill: (1) step 4 now requires two
  audits of the component library *before* authoring content — unguarded field
  access, which 500s the whole preview the moment an editor adds a block and
  silently destroys the "non-technical staff can edit this" pitch, and
  hardcoded root paths, which break every product route in a folder-per-brand
  space; (2) a new "Working with the Management API / MCP" section covering
  placeholder images having to be uploaded as real CMS assets (image-service
  helpers mangle external URLs), rate-limit-safe idempotent bulk writes, move
  operations that report failure while succeeding, checking `updated_at` with
  field selection as a cheap staleness guard before an update-in-place,
  API-written content bypassing editor validation, and using a
  datasource-backed multi-select when a commerce picker plugin is unreliable;
  (3) guardrails on publishing (API writes sit in draft and are invisible on a
  published-content frontend) and on preferring DOM assertions to screenshots.
  Also recorded that a reference component gallery shipped with a replication
  skill was captured from a different, abandoned template — the component
  source in the repo is authoritative. Template-specific fix lists stay in
  local memory per the confidentiality split.

## 2026-07-24 (improve: keep confidential template specifics out of git — generify custom-storyblok-demo)
- The `custom-storyblok-demo` skill had been authored around a specific
  starter template that is confidential to the operator's setup, and its
  identifiers had landed in tracked files (and pushed history). Generified the
  skill, `SKILLS.md`, and these changelog entries so git names no specific
  template, its files, or its fix list — the process stays generic (Path A =
  duplicate whatever starter repo the operator names; Path B = Storyblok CLI
  scaffold), and all template-specific specifics now live only in local,
  gitignored `memory.md`, loaded at run time. Also purged the identifiers from
  past commit history (history rewrite + force-push) since the darwin repo is
  public. Standing rule reinforced (CLAUDE.md guardrail 6): confidential
  operator/setup specifics never go in tracked files.

## 2026-07-24 (improve: custom-storyblok-demo reworked leaner after its first real end-to-end run)
- Ran the skill end-to-end for the first time (repo → fixes → deploy → CMS
  wiring, all working), then audited it for speed/token cost and applied the
  fixes back: (1) reordered so the human-gated step(s) go FIRST and overlap
  the code work, instead of blocking at the end; (2) collapsed a wasteful
  two-clone + scratchpad dance into a single clone with
  `gh repo create --source=. --push`; (3) switched the deploy from a guided
  dashboard flow to the Vercel CLI (`vercel link` / `env add` / `--prod`),
  setting env vars before the first deploy so there's no post-hoc redeploy,
  and dropped the Netlify-vs-Vercel question (Vercel default; Netlify an
  untried fallback); (4) stopped launching/verifying localhost — the deployed
  URL is now the single render-verification gate, per operator preference;
  (5) token discipline: verify with a cheap `get_page_text`/`innerText` check,
  never `read_network_requests` on a Nuxt/Vite frontend (inlined base64 fonts
  — the biggest token sink on the first run), and don't churn MCP `search` for
  space-settings writes the connector doesn't expose. Also recorded that the
  official Storyblok MCP connector (`https://mcp.labs.storyblok.com/mcp`) is
  now permanently configured.

## 2026-07-24 (improve: new custom-storyblok-demo skill, split out of storyblok-content)
- Split the demo-bootstrap runbook out of `storyblok-content` into its own
  dedicated skill, `custom-storyblok-demo` — operator's explicit call, to keep
  "writes content into a space that already works" and "stands up a brand-new
  customer's demo end to end" as separate skills. Added a Path B (Storyblok
  CLI scaffold) for operators without their own starter template. Space
  handling: never create a space that would render blank (ask the operator to
  seed it), then fetch its Preview token automatically rather than making them
  paste it. Template-specific specifics kept in local memory, not in git.

## 2026-07-24 (improve: storyblok-content gained then shed a demo-bootstrap runbook)
- First real run of standing up a Storyblok demo frontend end to end (repo →
  Vercel → env vars → live). Captured the reusable, non-confidential lessons
  (own a private repo rather than pushing to a shared template; env vars
  needed locally vs. on the host; read-only Shopify Storefront token vs.
  write-capable Admin token; watch `npm install` for silent dependency
  version bumps). This later moved into the dedicated `custom-storyblok-demo`
  skill; template-specific quirks live in local memory.

## 2026-07-23 (improve: process-customer retro after an FR demo went well)
- process-customer/SKILL.md Step 1: added Gong as an explicit self-research
  source with a concrete rule — never rely on thematic/synthesized Gong
  answers alone, always also pull the raw call list (dates + attendees)
  before writing any "first meeting" framing. A real miss this cycle: a
  thematic query didn't surface that the AE had already run several
  presentations over many months before the SE's first cycle; corrected
  after the fact, should have been right the first time.
- process-customer/SKILL.md Step 1: added public web research (the
  account's own site(s), corporate structure, industry, visible tech
  signals) as a standing source, done proactively rather than only when
  asked — also noted that Drive already holds real BDR/AE account-research
  docs for other accounts using a reusable template, worth checking for
  the account in hand before researching from scratch.
- memory.md "Local config": recorded the Gong workspace name to default to,
  so a future session doesn't rediscover it via a failed call.

## 2026-07-23 (improve: safe slide-deletion recipe for build-deck)
- build-deck/SKILL.md: deleting a pptx slide via `sldIdLst.remove()`
  alone leaves an orphaned relationship that can make python-pptx write
  a duplicate part filename on save (silent corruption, only a
  `zipfile` UserWarning as a clue). Documented the correct two-step
  deletion (`prs.part.drop_rel(rId)` + `sldIdLst.remove()`) and a
  mandatory post-save duplicate-part check before ever delivering a
  deck built by slide surgery.

## 2026-07-23 (build-deck run on an FR multi-brand demo account)
- First real build-deck run: adapted one existing example deck's three
  native station slots in place (its own editorial-experience station,
  its own structured-content-reuse station, and one repurposed
  station retexted using a different existing deck's real personalization/
  AI content) rather than inventing anything. Dropped the optional
  technical-topics/placeholder section for a first-SE-cycle account.

## 2026-07-23 (process-customer run on an FR multi-brand demo account)
- Ran process-customer on a partner-sourced FR demo account ahead of a
  first-meeting custom demo. Gong is connected via a deal-intelligence MCP
  (ask_deal/ask_account/generate_brief, multi-workspace — pass the workspace
  name) — pulled the discovery call directly per guardrail 5 instead of asking
  the operator to paste it; Salesforce still not connected (operator pasted the
  opp notes). Wrote the local-only brief and rewrote the whole Notion row
  (Stage/Priority/AE/Next call/Notes/MEDDPICC). MEDDPICC multi-select convention
  used = tag only the Confirmed elements; Partial/Gap detail lives in brief.md.

## 2026-08-03 (targeted audit of the RFP library's unverified claims)
- Provenance scan showed only two of ten category files were officially
  grounded; the rest rested on validated submissions alone — the same
  weakness that let a wrong certification claim propagate. Ran a targeted
  verification of the four riskiest instead of a full re-audit.
- **Found the official public fact sheet**, which turned out to be the
  missing source and produced three corrections plus new commercial facts:
  - **A dedicated China data centre IS offered** — a paid add-on on both
    enterprise tiers, with an ICP license already in place and a custom
    domain. A note added the same week had wrongly said China wasn't
    available, based on the Trust Center listing only the four *standard*
    regions. The Trust Center lists standard regions; the fact sheet lists
    purchasable ones.
  - **The "is SSO an add-on?" conflict is resolved**: the fact sheet's legend
    separates included-in-licence from additional-fee, and SSO is included on
    both enterprise tiers. The validated submission was imprecise.
  - The broken-link checker's plan gate was wrong here (stated one tier too
    low).
  - New: a **usage-breach surcharge** applies after two consecutive months
    over the API/traffic limit — relevant to any cost-predictability answer.
    Also the self-managed vs managed backup split, and the four legal
    entities.
- **Company scale figures confirmed as genuinely unpublished.** Checked the
  about page, the press page and the fact sheet: no headcount, customer
  count, enterprise-customer count, user count or total-funding figure
  appears on any of them. A validated submission asserted all five; a
  third-party estimate contradicted the headcount. The file now refuses to
  carry a number and instead routes the question to sales enablement or the
  AE, while keeping what *is* citable (founding year, founders, entities,
  named investors, a corroborated funding round, published leadership team).
  Also flagged that a stated ARR ambition must never be quoted as revenue.
- Migration tooling claims all verified as real (official importer repo,
  CLI v4 pull/push, schema-as-code package) — with a note that the
  schema-as-code package is very new, so it gets the same
  don't-oversell-maturity treatment as other recent features.

## 2026-08-03 (operator settles the certification question; a lesson about inferring negatives)
- The operator confirmed the actual position: **SOC 2 Type I is held, Type II
  is in progress.** That is now the answer in the library, with Type II always
  described as in progress and never with a completion date unless the
  security team supplies one.
- **This corrected an over-correction of mine.** Earlier the same day I had
  concluded from three official pages that no SOC 2 certification existed at
  all. Wrong, and arguably worse than the error I was fixing: understating our
  own compliance posture to a security-conscious prospect is as damaging as
  overstating it. Type I reports are shared under NDA, not marketed — which I
  had actually hypothesised and then abandoned in favour of a firmer claim
  than the evidence supported.
- Recorded as a standing rule in both the library index and the RFP skill:
  **never infer a negative from silence.** Public pages are for *checking* a
  claim, not *settling* one about our own company; when the web is silent on
  an internal fact, write "confirm with <team>" and ask a human.
- Also demoted the library's trust ranking for validated submissions. They had
  been ranked above official docs; of the three harvested so far, two carried
  a wrong certification claim and one mis-stated a licensing detail. New
  order: the operator/owning team, then official docs, then validated
  submissions — trust their framing, verify their figures.
- Kept the genuinely useful finding: the sprawling certificate list on the
  oldest security page belongs to the **hosting provider**, not us, which is
  the likely origin of the erroneous Type II claim in a shipped document.
  Cite the hosting layer's certifications separately and explicitly.

## 2026-08-03 (SOC 2 root cause found: it is the hosting provider's certification, not ours)
- Checked a third official security page and found the source of the error.
  Its data-protection section lists a long certificate set — SOC 1/2/3,
  FedRAMP, PCI DSS L1, ISO 9001/27001, FIPS 140-2 and more — but the sentence
  subject is **the AWS region the platform is hosted in**, not the platform
  vendor. Someone read that list as the vendor's own, which is almost
  certainly how "holds ISO 27001, SOC 2 Type II certifications" got into a
  validated submission; the other submission's softer "Type I" looks like the
  same misread.
- Standing rule tightened accordingly: claim ISO 27001, TISAX, GDPR and the
  EU-U.S. Data Privacy Framework as the vendor's own; cite the hosting
  provider's certifications separately and explicitly as the *hosting layer's*
  (legitimate and often reassuring in a security review); never present SOC 2
  of any type as the vendor's own certification.
- **Un-resolved a figure I had marked resolved:** the two official pages
  disagree on transaction-log/backup retention (14 vs 30 days). The 30-day
  page carries clear staleness tells — it still documents TLS 1.2 and a TLS
  1.1 deprecation dated 2021, where the main page says TLS 1.3 — so 14 is
  probably current, but the file now says confirm before quoting rather than
  asserting either. Retention windows are exactly what a regulated prospect
  holds you to.
- Also captured for security questionnaires: scoped personal access tokens
  for least-privilege integration access, the documented
  reviewed/tested/approved change-management process, monthly recovery tests
  covering point-in-time database and asset restore, and SSH-key-only
  internal network access.

## 2026-08-03 (SOC 2 conflict resolved against official docs — do not claim it)
- Checked the two authoritative public security pages directly. The Trust
  Center's ISMS section lists **ISO 27001 and TISAX only**, and the dedicated
  enterprise-security page (titled "ISO 27001 Certified CMS") lists exactly
  ISO 27001, OWASP, GDPR, WAF and AWS GuardDuty. **SOC 2 appears nowhere, in
  any form.** So neither validated submission's claim holds up: one asserted
  Type I, the other Type II — and the first had explicitly warned that third
  parties keep wrongly claiming Type II.
- Standing rule recorded: answer certification questions with ISO 27001 +
  TISAX + GDPR / EU-U.S. Data Privacy Framework, and say SOC 2 report
  availability will be confirmed in writing. A report may well exist under
  NDA and simply not be marketed — which is why it needs confirming rather
  than asserting. Escalated to the security team rather than resolved from
  documents.
- Two other facts settled from the same pages: transaction-log retention is
  **14 days** (the earlier 30-day figure was wrong), and officially listed
  data-residency regions are **US, Canada, Germany, Australia** — China is
  not listed as a standard region, so an earlier internal note claiming it
  is now marked unconfirmed.
- New material picked up: self-certification under the EU-U.S. Data Privacy
  Framework, a publicly downloadable ISO 27001 certificate, and **two
  separate GTCs** (Americas enterprise vs. global/self-service and all other
  regions) — point procurement at the right one.

## 2026-07-30 (English is the working language; two more validated RFPs harvested)
- **CLAUDE.md Voice**: English regardless of the input language. Source
  material may arrive in any language; reading it in that language is fine,
  but answers, summaries and notes are English unless the operator asks
  otherwise. The deliberate exception stays: a customer-facing deliverable
  in the customer's own language, confirmed with the operator first, with
  internal notes still in English.
- Two further validated submissions added by the operator and harvested —
  both **requirements grids** (~250 rows total) rather than prose RFPs, i.e.
  the other major RFP format. New `dam-assets.md` category file (Asset
  Manager, image service, asset governance/distribution, and the two honest
  DAM partials). Existing files gained: first-party automation product as
  the middleware answer with 500+ pre-built integrations; no native CRM or
  marketing-automation connectors (per-tool check required); no
  customer-controlled server and no plugin hosting, so "move our app into
  the CMS" is a real no; no customer-facing infra monitoring dashboard;
  native broken-link/404 checker; two genuinely platform-side Core Web
  Vitals levers; simpler personalisation cases that *are* native; and the
  reusable framing for AI/intent requirements.
- **New standing lessons in `_index.md`** drawn from the grids: a
  well-written "partial" beats a padded "yes" (name the limit, then the
  bridge); some answers are legitimately "not ours" and should say so while
  naming what the platform does contribute; cross-reference rather than
  repeat in large grids.
- `rfp-answer/SKILL.md`: match the response vocabulary the document itself
  uses — a prose RFP's four-option scheme and a grid's Yes/Partial/No comply
  column are different, and inventing categories is a compliance failure.
- **Escalated, not resolved: a SOC 2 contradiction between two validated
  submissions** (one says Type I and warns against claiming Type II; the
  other claims Type II; the public trust centre mentions neither). Recorded
  with an instruction to cite only the certification all sources agree on
  until the security team confirms.

## 2026-07-30 (improve: real plan/licensing knowledge, after plan-gating turned out to be a systematic weakness)
- The operator pointed out the obvious root cause behind the plan-gating
  misses: no structured knowledge of the product's own plan tiers existed
  anywhere in the repo. Fixed by transcribing the full public pricing-page
  comparison matrix plus the internal Premium-vs-Elite solutions guide into
  `resources/rfp-library/answers/pricing-licensing.md` — real tier names,
  per-tier quotas, the full feature-gating split, support tiers, integration
  gating, and the separately-priced automation product line.
- **This corrected real errors in category files committed the same day**,
  which is exactly why the reference was needed: log/version retention is
  180 days on the mid enterprise tier and unlimited only on the top one
  (previously written as "indefinite on Enterprise"); fine-grained folder/
  asset scoping and token scopes are top-tier-only, not general; the GraphQL
  API, pipelines, space/tool plugins and SSO/SCIM are enterprise-gated, not
  universal; region choice is a paid add-on on the mid tier; several
  "unlimited" capabilities have real numeric caps below the top tier; and an
  automation product described as roadmap is in fact already sold.
- `rfp-answer/SKILL.md` step 3 now requires reading that file *before*
  answering any capability question, and points at the pricing page's
  integration matrix as the fastest check for whether an integration with
  the prospect's own platform exists.
- Also logged an unresolved conflict rather than papering over it: a
  validated submission called SSO an add-on while the public matrix lists it
  as included on both enterprise tiers.

## 2026-07-30 (improve: rfp-answer gets account-driven research; RFP library seeded for real)
- An externally-proofed version of an RFP response came back materially
  better than Darwin's draft (~39% longer, same structure/formatting —
  the delta was all substance). Diffed both and rebuilt the skill around
  the gaps.
- `rfp-answer/SKILL.md`: RETRIEVE now runs **two research passes** —
  requirement-driven *and* account-driven. The account-driven pass leads
  with "does our product natively integrate with the prospect's own
  product?", which is where the biggest miss came from (a shipped
  first-party integration with the prospect's own platform, publicly
  documented, no competitor equivalent — invisible to requirement-driven
  research because it was never a line item). Also added: always resolve
  the plan tier behind a capability answer; find the delivery route before
  calling anything a gap; don't downgrade feature maturity off a changelog
  entry alone; exhaust sanctioned sources before writing `needs SME input`;
  pull live deal context from Slack/email/calls rather than trusting the
  account brief. Plus a "why that response was better" section to re-read
  before each RFP.
- HARVEST rewritten so the library actually compounds: validated documents
  are saved (gitignored) and the reusable positioning extracted into
  tracked category files, with the *reason* a draft differed recorded
  alongside the corrected fact.
- `resources/rfp-library/` genuinely populated for the first time: 10
  category answer files (tracked, client-scrubbed) covering architecture,
  editorial, localisation, SEO/AI, personalisation, integrations, security,
  migration, credentials and licensing — including a plan-gating table,
  since missing plan tiers was a systematic weakness. New
  `validated-submissions/` folder, **gitignored at creation** (guardrail
  10) since it holds real as-submitted documents.
- `_index.md` rewritten with the two-layer model, a `validated` trust
  level ranked above official docs, and a standing-lessons list.

## 2026-07-23 (improve: ask before checking brand guidelines; Notion is the live source)
- CLAUDE.md guardrail 11 revised: ask the operator first whether a new
  document needs to actually follow brand guidelines, or just needs the
  input file's own format kept with information added — avoids reading
  brand assets (tokens) when they turn out not to be needed. When
  guidelines genuinely apply, a local snapshot in
  `resources/brand-guidelines/` is treated as possibly stale — a
  `SOURCES.md`-style note pointing at a live source (typically a Notion
  page) is checked directly instead, especially for fonts.
- `resources/style-guide.md` and `rfp-answer/SKILL.md` updated to match
  (ask-first branching, live-source-over-snapshot rule).

## 2026-07-23 (improve: brand-guidelines compliance for document creation)
- New CLAUDE.md guardrail 11: before creating any document that will
  carry the operator's own branding, check `resources/brand-guidelines/`
  (real logo + brand guideline doc, gitignored, local-only) rather than
  guessing or reusing a color sampled from someone else's document.
  `build-deck` is exempt — it already inherits real branding from the
  operator's own example decks.
- `resources/style-guide.md` gets a matching pointer to the same folder
  (kept generic — no real brand specifics tracked in git).
- `rfp-answer/SKILL.md` gains a new "producing an actual response
  document" section: default to annotating a prospect's own RFP document
  in place (their wording untouched, answers appended as extra columns/
  labeled blocks) rather than rebuilding a separate proposal, and check
  brand guidelines before styling any co-branded cover page.

## 2026-07-23 (Darwin goes agnostic)
- **Darwin is now company- and product-agnostic.** The tracked repo no
  longer assumes the operator works at or sells any particular product.
  CLAUDE.md "Identity" rewritten as a template; operator name/company/
  product/team now live only in local, git-ignored files (`memory.md`,
  `resources/people.md`), written by `darwin-setup`.
- Genericized "the product we sell" (was Storyblok) across all operating
  skills, `style-guide.md`, `SKILLS.md`, and the guardrails. The deck
  template keeps its reusable *structure*; product-specific slide content
  now comes from the operator's own example decks, not hardcoded.
- `storyblok-content` kept as the one sanctioned exception — reframed as
  an opt-in "only if your company uses Storyblok CMS" integration.
- Operator referred to generically ("the operator") in operating files;
  gendered pronouns neutralized. Real colleague names removed from
  operating skills (they reference `resources/people.md` instead).
- `resources/people.md` now git-ignored (real team = local
  customization); tracked `resources/people.template.md` added. Product
  brand assets under `resources/brand-guidelines/` git-ignored too.
- Historical docs (this file, HANDOFF, ROADMAP) keep the reference
  operator's build history — framed as history, not a hardcoded default.

## 2026-07-22
- `storyblok-content` gained two new guardrails from a real content
  session on a demo space (via Claude Code, not this repo's Cowork
  session): (1) re-fetch a story/component fresh immediately before
  every update-in-place write, never reuse content held from earlier
  in the task — a full-content overwrite built on stale content nearly
  clobbered manual Visual Editor edits made in between writes; (2) when
  extending an existing component schema for a new feature, touch only
  the new field(s) — adding a restriction to a pre-existing field broke
  its focal-point/crop editor, caught only because the visual result
  looked off. Fixed live on the affected space, then generalized here.
- The operator: Claude Code (run on their own machine, outside Cowork) has a
  working Storyblok MCP connector. `storyblok-content` rewritten to
  check for it first at runtime (never assume tool names — same rule
  as Gong), falling back to the Management API token path if absent.
  Skill is now completable from Claude Code today, even though a
  Cowork sandbox alone still can't reach `mapi.storyblok.com`.
- README intro re-expanded (~1.4k chars) after the title-rename commit
  left it shorter than the photo again — text/image height parity
  restored.

## 2026-07-21 (Darwin's namesake, a signature)
- The operator: Darwin is named after their cat — a black cat with pointy ears
  and huge round eyes, "the sweetest cat ever." New CLAUDE.md
  "Signature" section: a small ASCII cat that opens `darwin-setup`'s
  introduction and every "routine" skill's chat output (`daily-briefing`,
  `weekly-review`, `monthly-review`), always as the very first thing.
  The operator dropped the real photo in at the repo root; moved it to
  `resources/darwin.jpg` and it's now shown at the top of `README.md`
  above the ASCII mascot.

## 2026-07-21 (repo made shareable: Gong MCP wired in conditionally, no client names in git)
- Gong support made conditional across the call-consuming skills
  (`call-coach`, `post-call-update`, `process-customer`) and CLAUDE.md
  guardrail 5: if a Gong MCP connector is present in the session, pull
  transcripts directly (discover the actual tool names at runtime — they
  vary by connector version, never assume); if not, fall back to the operator
  pasting transcripts, exactly as before. `darwin-setup` now tests/asks
  for Gong alongside the other connectors. (At the time of writing, no
  Gong MCP tools were actually resolvable in-session — the wiring is
  forward-compatible, same pattern as storyblok-content.)
- **Repo scrubbed of all real client/account names** so colleagues can
  use Darwin: `memory.md` is now git-ignored (local-only, regenerated
  fresh per person by `darwin-setup` — same model as `accounts/`), and
  every tracked file (this CHANGELOG, HANDOFF, ROADMAP, skills, resources,
  CLAUDE.md) was rewritten to generic account descriptors. Guardrail 6
  tightened to make "zero real client names in git" a hard rule, not a
  preference. (Git *history* still contains prior names — rewriting that
  is a separate manual step, noted for the operator.)

## 2026-07-21 (improve: process-customer chat reply must lead with company brief)
- The operator: "Never forget to add a brief about the company" — a real run
  wrote a full "Who they are" section into the local brief.md, but the
  chat reply itself skipped straight to the Notion diff, so the operator had
  to ask "what is this company?" afterward.
- Fixed in `process-customer/SKILL.md` Step 5: new explicit point 2
  (chat reply must lead with the 2-4 sentence company brief before any
  field-level diff), renumbered the rest. Applies to every future run.

## 2026-07-21 (process-customer: first real run)
- Ran `process-customer` on a live RFP deal for the first time — wrote
  its local-only `accounts/<account>/brief.md`. No Gong/Salesforce
  extracts existed (it's an RFP) — all evidence from Slack (a
  deal-specific channel + a bid-function channel + #se-requests + DMs)
  and Drive (the requirements matrices + a colleague's draft response).
- Notion row: Deadline corrected to the real submission deadline
  (the prior date was wrong). Notes rewritten to reflect real progress
  (large draft response already mostly resolved, in human-review).
  MEDDPICC left empty — all 8 elements landed Gap/Partial, none
  Confirmed (expected for a partner-mediated RFP with no direct contact).
- Surfaced two unresolved inconsistencies to the operator rather than
  auto-fixing (a deal-value conflict across two Slack sources, and a
  possible naming confusion in their own shorthand).

## 2026-07-21 (accounts/ folder was leaking customer names into git history)
- The operator asked whether the account folder scaffold was necessary. It
  wasn't: only 3 of the real account folders even had a tracked
  `.gitkeep`, and those paths put real customer names into git history
  via the directory name itself even though contents were empty — a
  guardrail-6 violation. Removed the customer-named paths, replaced with
  a single generic `accounts/.gitkeep`, and tightened `.gitignore` to
  `accounts/*` + `!accounts/.gitkeep` so no future account folder can be
  committed by name.

## 2026-07-21 (new: darwin-setup — portability to a new computer/account)
- Built `darwin-setup`: the interview that rebuilds this repo for
  whoever is actually running Darwin. Wired to fire automatically —
  CLAUDE.md has a "First run" check at the top ("does `memory.md` exist?
  if not, this is an uninitialized copy") that runs `darwin-setup`
  before responding to anything else, introduction first. Covers: who
  the person is, team structure (generalized — doesn't assume the
  AE-pod/SE-colleague shape), which connectors are live (tested, not
  assumed), where the real data lives (Notion DB, Slack channel IDs),
  which guardrails to keep vs. change, voice preferences. Rewrites
  CLAUDE.md Identity/Voice/guardrails, `resources/people.md`, and resets
  `ROADMAP.md`/`memory.md` — doesn't merge with the previous config.

## 2026-07-21 (storyblok-content: token in, blocked by sandbox network allowlist)
- The operator provided a Storyblok Management API personal access token,
  stored gitignored at `storyblok.env` (confirmed ignored before writing
  it in). Test call from this Cowork sandbox to `mapi.storyblok.com` was
  blocked by the sandbox's own network proxy allowlist — a sandbox
  network-access problem, not a token or connector one. Two documented
  ways around it: run from the operator's own terminal (no sandbox), or get
  `storyblok.com` added to the org's network allowlist.

## 2026-07-21 (Slack>Notion rule; Claim button removed; AE schema fixed; pipeline scan rebuilt)
- New standing rule (CLAUDE.md guardrail 8, broadened from a
  Notion-dates-only rule): for Deals, Slack outranks Notion. Notion is
  manually-maintained and can be stale, incomplete, or structurally
  wrong. When they conflict, Slack is evidence to fix Notion, never the
  reverse.
- Dashboard: removed the one-click "Claim" button (auto-wrote to Notion)
  from Worth Claiming cards, replaced with a "View Slack thread" link —
  the operator wants to read the actual thread before any action. Real trigger:
  a request looked claimable from the message text, but the thread showed
  the AE saying they didn't need SE support. Same link added to Team
  Pipeline cards. `claimDeal()` removed as dead code.
- Notion Accounts DB: the "AE" select property only had two configured
  options — most of the AE pod (several AE-pod members
  Strindevall) were never selectable values at all. Likely root cause of
  an earlier "why does everything show one AE" symptom. Added the missing
  three, then corrected one deal's AE from blank to the value the operator
  identified directly.
- Pipeline scan rebuilt: Team Pipeline and Worth Claiming were two
  separate batch-AI extractions over the whole #se-requests dump;
  replaced with one unified 7-day scan that parses each message
  deterministically (regex against the Slack bot's regular field format)
  and, only for messages with a thread, runs one small scoped AI call per
  thread asking whether the SE peers claimed it. Routing is binary
  per-message: claimed by one of those three -> Team Pipeline, else ->
  Worth Claiming. Claimed-but-untracked deals auto-add to Notion. Caught
  mid-build via `verify_artifact`: `update_artifact` had silently dropped
  `slack_read_thread` from the tool allowlist — always re-verify after
  updating an artifact.
- Team Pipeline row rebuilt: filtered to the AE pod, grouped by SE owner
  (real Notion user IDs resolved via `notion-get-users`) with dismiss (×)
  buttons persisted via localStorage.
- Found a real, live, company-wide "RFP Answer Library (POC)" Notion DB
  (unofficial — a coworker built it) and rewrote `rfp-answer` to use it
  as a read-only source, ranked below official Storyblok docs, above
  general research; every answer carries a trust-level tag.
- Confirmed via the MCP registry there is no Storyblok connector at all;
  `storyblok-content` updated to say so plainly.

## 2026-07-21 (two corrections: RFP trust order, Team Pipeline silent drop)
- `rfp-answer`: the shared POC library is unofficial — official Storyblok
  docs now rank above it, and if they disagree the docs win.
- Team Pipeline was silently dropping any deal with a blank/non-pod AE
  because the SQL filter excluded it. Fixed: fetch all deals, filter in
  JS, flag excluded deals by name + actual AE value in a banner.

## 2026-07-21 (Team Pipeline switched from Notion to Slack #se-requests)
- Per the operator: Team Pipeline no longer reads the Notion Accounts DB (too
  sparse) — it scans #se-requests directly, reads each request's thread
  to determine who claimed it, and groups by claimer.
- Re-added a `colorForName` helper accidentally deleted in an earlier edit.

## 2026-07-21 (fixed dashboard "still loading" hang)
- Root cause: a 30-day Slack scan + sequential per-thread claim-checks
  could take minutes, and a transient Claude API 529 overload was
  silently swallowed as "zero results." Fixed: window trimmed, thread
  checks parallelized and capped, and a `looksLikeApiError()` check now
  surfaces AI-call failures as a visible banner. (Later superseded by the
  full pipeline-scan rebuild above.)

## 2026-07-21 (Phase 4 finished; Phase 5 drafted and honestly flagged as blocked)
- `todo-sync` skill built: reconciles personal action items onto the
  single "✅ To Do" checklist block on the operator's Notion space page.
  Deliberately doesn't touch account-specific next-steps.
- Dashboard gained a "Team Pipeline" row alongside the renamed "the operator's
  Pipeline" row.
- Worth Claiming widened from 24h to 7 days, and now reads each
  candidate's Slack thread to filter out already-claimed requests.
- `rfp-answer` mechanism drafted (RETRIEVE + HARVEST against a markdown
  library, explicitly not a vector DB — documented why). Not live: the
  library is empty. `won-lost-notes.md` gitignored + untracked.
- `storyblok-content` mechanism drafted (dry-run → confirm → write →
  verify). Blocked on an execution path.
- Notion views renamed for clarity: chart view → "📊 Analytics (charts)",
  board view enriched + renamed "🗂️ Sales Pipeline (Board)", "By status"
  → "By AE" (it groups by AE, not Stage).

## 2026-07-20 (Kanban rebuild — Cowork artifact + Notion board view)
- `walid-deals-dashboard` rebuilt as an actual Kanban board: deals
  grouped into columns by real Notion Stage values. Any owned deal with
  no Stage set is flagged in a banner rather than silently dropped.
- Notion board/analytics/By-AE views renamed and enriched (see above).

## 2026-07-20 (deals dashboard artifact: 24h Slack window + visual redesign)
- "Worth Claiming" Slack scan windowed to the last 24h instead of a flat
  message-count pull.
- Full visual redesign — gradient hero with stat tiles, priority-colored
  card accents, pill tags. Behavior unchanged (Notion write on Claim,
  Slack reply copy-only, never auto-sent).

## 2026-07-20 (build-deck first run: FR demo deck; gitignore gap fixed)
- **First real `build-deck` run** — a French-language demo deck for an FR
  retail account, adapted from the closest example deck, saved local-only.
  3 demo stations picked from `style-guide.md`'s real theme vocabulary
  (not invented), translated to French; scaffolding slides excluded.
- **`improve: broaden accounts/ gitignore to all files, not just .md`** —
  the `.gitignore` rule didn't cover a new `.pptx` deck. Changed so the
  whole per-account directory stays local-only.

## 2026-07-20 (feat: deals dashboard artifact — Notion + Slack, claim-in-one-click)
- New Cowork artifact `walid-deals-dashboard`: "My Deals" (live Notion
  query) + "Worth Claiming" (Slack #se-requests AI-triaged for unclaimed
  AE-pod opportunities). Claim button created the Notion row and showed a
  copy-ready Slack reply rather than auto-sending. (Later replaced by the
  thread-link approach.)

## 2026-07-20 (feat: weekly-review built; weekly + monthly routines scheduled; no-artifact preference)
- **Standing preference**: no artifacts from routines — plain chat text.
  `daily-briefing/SKILL.md` docs fixed to match.
- **New skill: `weekly-review`** — pipeline/account-health pulse, distinct
  from `monthly-review`'s self-audit scope.
- **Both `weekly-review` and `monthly-review` scheduled** (Mondays 08:30,
  1st of month 08:00, Paris time).

## 2026-07-20 (improve: root-cause fix for the recurring gitignore pattern; monthly-review; demo-setup reframed)
- **New CLAUDE.md guardrail 10**: `.gitignore` check at file creation for
  anything under `resources/`/`accounts/` that could hold real customer/
  personal data — fixes the root cause behind three same-day incidents.
- **New CLAUDE.md Voice rule**: bullets vs. prose, generalized from the
  `call-coach` fix.
- **New skill: `monthly-review`** — pattern-mining across memory/CHANGELOG.
- **`demo-setup`/`ROADMAP` reframed**: the written setup script is the
  deliverable, connector execution is a future bonus, not a dependency.

## 2026-07-20 (post-call-update: a real deal backfilled from a transcript)
- Notion Debriefs DB: new row for a real call. Accounts DB MEDDPICC +
  Notes refreshed with what the call confirmed. The account's local-only
  `debriefs.md` created. Full evidence stays local per guardrail 6.

## 2026-07-20 (improve: call-coach chat output always mirrors bulleted log format)
- **`call-coach/SKILL.md`** new step 7: the chat reply always uses the
  same two-bullet-list structure as the `coaching-log.md` entry — the
  first real run went out as prose in chat despite the log being bulleted.

## 2026-07-20 (improve: untrack coaching-log.md from git)
- **`resources/coaching-log.md`** was tracked since the initial scaffold;
  once `call-coach` wrote a real entry it would have pushed customer
  quotes to GitHub. Added to `.gitignore`, untracked via `git rm
  --cached`. Content on disk unaffected.

## 2026-07-20 (improve: darwin-improve triggers more reliably)
- **CLAUDE.md "How I improve"** rewritten with concrete trigger signals
  (correction, stated preference, self-caught mistake, reflection
  question) instead of relying on the literal "Darwin, learn this"
  phrase; explicit instruction to reread the skill before acting; no
  exceptions to the `improve:` commit prefix.
- **`darwin-improve/SKILL.md`**: new step 1 (notice the trigger), step 2
  rereads the file's own steps before executing.

## 2026-07-20 (improve: permanent guard against the overwrite mistake recurring)
- **New CLAUDE.md guardrail 9**: before any bash/python write to a path
  the Edit tool refuses (currently `.claude/skills/`), manually
  `cat`/`git log --oneline -- <path>` first.
- `darwin-improve/SKILL.md` step 3 documents this failure mode directly.

## 2026-07-20 (fix: restored overwritten demo-script/demo-setup content)
- **Self-caught mistake**: an earlier "formalize as skills" pass
  overwrote real pre-existing Phase 3 content without reading it first.
  Restored and merged with the one thing the pass got right (explicit
  confirm-before-finalizing gates).

## 2026-07-20 (fix: title-slide AE/SE headshots; new photo resource)
- **New resource**: `resources/AEs & SEs/<Full Name>.jpeg` — headshots for
  deck title slides (gitignored). The operator added their own; others missing.
- **Bug fix**: all example decks carry 2 headshot PICTURE shapes on slide
  1, invisible to a text-only scan. The first FR deck had shipped with the
  source deck's original photos. `style-guide.md` + `build-deck/SKILL.md`
  updated to call out the headshot swap.

## 2026-07-20 (improve: confirmation checkpoints added to demo-prep skills)
- **`build-deck`** confirms the demo-station shortlist before building.
- **New skills `demo-script`/`demo-setup`** formalized from the first real
  runs, each with a confirmation checkpoint before finalizing.

## 2026-07-20 (Gmail added to daily-briefing; deals updated from email)
- **New connector wired in: Gmail** — `daily-briefing/SKILL.md` scans the
  inbox as a fourth source, read-only (guardrail 1). New "📧 EMAIL" section.
- Several live deals updated from email evidence (deadlines confirmed, a
  stage moved to Contracting, a stub-row account fleshed out with a
  confirmed SE + demo date). Real detail stays in local-only account files.

## 2026-07-20 (process-customer: renamed Phase A-E to explicit step names)
- Phases renamed for clarity (Self-research / Ask-hard-stop / Analyze /
  Write / Update-and-announce). No behavior change.

## 2026-07-20 (an RFP deal claimed — Notion row from a parallel session)
- A new RFP deal was claimed live off a briefing flag; its Notion row was
  created in a different session (repurposing a blank stub). Real evidence
  moved to the account's local-only notes; that session's memory entry had
  inline customer specifics, trimmed to meta-level.

## 2026-07-20 (process-customer: added a "Demo — what to show" section)
- Template gains a "Demo — what to show" section tying concrete use cases
  to real pain points/MEDDPICC gaps (not a generic feature tour). Applied
  to the first FR brief (local-only) and its Notion page.

## 2026-07-20 (process-customer: first real run + quality upgrade)
- **First real `process-customer` run** — an account brief written from
  real Salesforce + Gong extracts, Notion row rewritten. Customer
  specifics live only in the local-only file.
- **Policy fix (the operator's correction): customer data must be git-ignored.**
  `accounts/*/*.md` added to `.gitignore`; guardrail 6 codified.
- **`process-customer/SKILL.md` upgraded**: Step 2 asks for Salesforce +
  Gong + a deal-specific Slack channel ID; Step 4's brief template
  restructured (prospect intro, attendees, bigger MEDDPICC, concrete
  next-steps checklist).
- **Notion — "Pipeline board" view added** (Kanban by Stage); the FR
  account's Notion page got the brief + checklist pasted in.

## 2026-07-20 (account-switch recovery: git, scheduler, artifact, GitHub connector)
- **Git repo reconciled with the real GitHub history** (`ewalid/claude-os`,
  branch `main`) after the local repo was orphaned from a different Claude
  account on the same Mac. Restored `README.md`, gitignored the example
  decks (~120MB, local-only).
- **Scheduled task and live artifact recreated** under this account.
- **GitHub MCP connector connected** (OAuth); tools load on a fresh session.

## 2026-07-15 (process-customer: full-row rewrite)
- **`process-customer` now rewrites the ENTIRE Notion Accounts DB row**
  (Stage, Priority, Notes, AE, Next call, Deadline, MEDDPICC) from real
  Gong/Salesforce/Slack evidence — not just MEDDPICC. Removed the
  propose-diff-wait-for-OK gate per guardrail 4; writes directly and
  announces after. Never writes an unevidenced field, never touches
  human-only columns.

## 2026-07-15 (process-customer upgrade)
- **`process-customer` now does a real MEDDPICC analysis** against pasted
  extracts, tagging each of the 8 elements Confirmed/Partial/Gap; only
  Confirmed elements write to Notion (a downgrade still needs the operator's OK).

## 2026-07-15
- **Notion — Accounts DB restructured.** Split the ambiguous "Due date"
  (kept as legacy) into **Next call** and **Deadline**; added **AE** select
  and **MEDDPICC** multi-select properties. Populated the live rows'
  dates; AEs left unassigned where no source confirmed them.
- **Notion — new views**: "This week", "Demos upcoming", "Overdue", and a
  "Dashboard" tab (charts couldn't be scripted via the API — added by hand).
- **Notion — new "📞 Debriefs" database** created, related back to Accounts.
- **daily-briefing** updated to read the new fields; became a live Cowork
  artifact and was scheduled (weekdays 9am Paris).
- **Repo created**: private GitHub repo, scaffolded from HANDOFF.md.

## 2026-07-15 (later same day)
- **People model corrected**: `resources/people.md` distinguishes the AE
  pod from SE colleagues (the operator's SE colleagues) and
  the operator's manager (the operator's manager). Wired into the artifact's Slack triage.
- **Slack scan window widened** to "start of the last working day through
  now".
- **Deck format decided**: .pptx exports, not .odp.

## 2026-07-15 (Phase 2)
- **`darwin-improve`, `post-call-update`, `call-coach` skills drafted.**
  Phase 2 complete. Phase 3 next, blocked on Storyblok MCP + example decks.

## 2026-07-15 (Phase 3)
- **`resources/deck-examples/` populated** — 5 real demo decks (gitignored).
  Reverse-engineered the shared template into `style-guide.md`.
- **`build-deck`, `demo-script`, `demo-setup` skills drafted** — `demo-setup`
  blocked on the Storyblok connector.

## Known gaps / carried forward
- Notion "Analytics" view has no charts yet (API limitation).
- MEDDPICC not yet populated for most accounts — needs the operator's input or
  real extracts.
- `rfp-answer` library is empty — blocked on the operator seeding real content.
- `storyblok-content` can't run in-sandbox — network allowlist / run locally.
- `resources/battle-cards/` is empty — `demo-script`'s objection-sourcing
  has nothing to draw on yet.
- Colleague (AE/SE pod) names are still in tracked files by design — they
  aren't clients. `darwin-setup` regenerates them per person.
- Git *history* still contains pre-scrub client names — rewriting history
  (e.g. `git filter-repo`) is a separate manual step if the operator wants it.
