# RFP answer library — index

Two layers:

**`answers/<category>.md`** — tracked in git. Reusable product-positioning
answers, scrubbed of every client/stakeholder/deal specific (guardrail 6).
This is the moat: each RFP should leave this library better than it found it.

**`validated-submissions/`** — **gitignored** (2026-07-30, guardrail 10).
The real, as-submitted documents. They name real clients, stakeholders,
partner firms and commercial specifics, so they never go to git — but they're
the highest-trust source there is, because they're what the operator actually
put their name on. Read the most recent one for a category before drafting
anything new.

## Trust ranking for anything in here
1. **The operator / the owning internal team** — the only way to settle an
   internal fact the public web doesn't cover (see lesson 8).
2. **`official-docs`** — the public pages and fact sheet. Best for plan
   gating, quotas, SLA, published capabilities.
3. **`validated`** — survived the operator's review and went out the door.
   Excellent for *framing* and commercial judgement, and it carries real
   nuance the docs don't. But **not infallible on facts**: of the five
   validated submissions harvested so far, three contained a wrong or
   misattributed certification claim and one mis-stated a licensing detail.
   Trust the framing; verify the figures — and see standing lesson 9 below.
   ✅ The 2026-09-15 architecture RFI is the first to get the AWS-certificate
   attribution **right**, and it also resolved the 14-vs-30-day backup
   contradiction outright — so the newest submission is currently the most
   factually reliable one in here. It still gave two different SOC 2 Type II
   dates in two sections, so "verify the figures" stands.

## Categories
| File | Covers |
|---|---|
| `platform-architecture.md` | API/headless architecture, environments, dev workflow, hosting, extensibility |
| `editorial-experience.md` | Visual Editor, components, workflows, versioning, scheduling, AI authoring |
| `localisation.md` | i18n model, fallbacks, TMS/AI translation |
| `seo-ai-discoverability.md` | Meta/canonical/hreflang, structured data, sitemaps, redirects, LLM/AI readiness |
| `personalisation-experimentation.md` | A/B testing, audience targeting, variants |
| `integrations.md` | Forms/CRM/MAP, analytics, consent, PIM/commerce, middleware patterns |
| `security-compliance.md` | Certifications, SSO/SCIM, RBAC, residency, SLA, backups, DR |
| `migration.md` | Migration tooling/methodology, SEO preservation, freeze/rollback |
| `dam-assets.md` | Asset Manager, Image Service, asset governance/distribution, DAM partials |
| `company-credentials.md` | Company facts, funding, partnerships, case studies, references |
| `pricing-licensing.md` | Licensing structure, plan gating, TCO framing |
| `api-limits-webhooks.md` | Rate limits, quotas, `cv` caching, CDN layering, webhook guarantees/security, API versioning & deprecation |
| `multi-space-governance.md` | Space/folder topology, schema-as-code, propagation, compatibility gates, blast radius, rollback, component lifecycle, environments |
| `ai-governance.md` | AI capability register, providers, training/retention/residency, BYOAI, MCP server, opt-out, IP position |
| `observability-telemetry.md` | Logs, metrics, audit events, alerting, SIEM export, the named telemetry gaps |
| `exit-portability.md` | Export formats, S3 Backups exclusions, exit runbook, SDK lock-in inventory |

## Standing lessons (learned the expensive way — read before drafting)
1. **Plan tier gates features — read `pricing-licensing.md` FIRST, before
   answering any capability question.** It holds the full transcribed
   public plan matrix (tier names, quotas, and the ~third of the feature
   set that is Premium+/Elite-only). Never answer "supported out of the
   box" without naming the plan; a capability answer without its tier is
   half an answer and creates commercial risk downstream. "Unlimited" is
   usually an Elite word — Premium often has a real numeric cap. The
   pricing page's own integration matrix is also the fastest way to check
   whether an integration with the prospect's platform exists at all.
2. **Check whether the product natively integrates with the prospect's own
   product.** See `integrations.md` — this was missed once on an RFP where a
   first-party integration with the prospect's own platform existed and was
   the single strongest differentiator available. Requirement-driven research
   will never surface it, because it's never a line item.
3. **A "gap" is usually a path.** Before writing "Requires custom
   development," ask what the *actual delivery route* is on the target stack.
   Naming the route ("embed the existing forms directly — zero CMS
   dependency") is both more accurate and more useful than naming the
   absence.
4. **Answer the metric, not just the feature.** If the RFP states success
   metrics, tie capabilities back to them explicitly.
5. **A well-written "partial" beats a padded "yes".** The most reusable
   content in this library came from the *partial* rows of a validated
   requirements grid — each one names the precise limit and then the bridge
   ("no native X; achievable via Y, which is additional implementation").
   Copy that shape. Never inflate a partial into a yes; never leave a
   partial as a bare no.
6. **Some answers are legitimately "not ours".** Frontend/implementation-
   partner responsibilities (PWA shell, responsive build, WCAG at site
   level, lazy loading, code splitting) should be stated as such, paired
   with what the platform *does* contribute. A validated grid used this
   framing consistently across ~250 requirements without losing the deal.
7. **Cross-reference instead of repeating.** In a large grid, answering
   "same pattern as TB0024" is normal, accepted practice and keeps the
   response readable. Write the canonical answer once.
8. **"Not on the website" ≠ "not true" — and over-correcting is its own
   error.** Public pages are the right way to *check* a claim, never the way
   to *settle* one about our own company. Compliance reports, internal
   roadmap status and commercial terms are routinely unpublished by design.
   Real case (2026-08-03): three official pages didn't mention a
   certification, so this library briefly asserted we didn't hold it — the
   operator corrected it (we hold Type I, Type II in progress). That
   understatement would have damaged us in a security review just as much as
   the overstatement it was "fixing". When the web is silent on an internal
   fact, write "confirm with <team>" and ask a human — don't infer absence.
9. **A wrong certification claim isn't a one-off — it recurred, verbatim,
   in a second validated submission.** The exact "hosted on AWS which has
   [SOC 2, FedRAMP, PCI DSS L1...]" paragraph — whose actual subject is
   AWS's certificates, not Storyblok's own — was copied into a validated
   RFP response a second time (2025-11-19), still attributing AWS's
   certifications to Storyblok. Treat this specific paragraph as
   contaminated wherever it's found in source material, and rewrite it
   explicitly as "our infrastructure provider AWS holds..." every time,
   rather than assuming a previously-validated document already fixed it.
10. **Hosting-model answers can go stale fast.** One validated submission
    stated flatly "no dedicated/private-cloud option, no customer-controlled
    server." A later validated submission (2025-11-19) offered a genuine
    **BYOC (customer-controlled cloud)** option — full backend hosting in
    the customer's own AWS/GCP/Azure account, still Storyblok-managed.
    Re-check this specific claim against the latest validated source or the
    account team before repeating an older "no such option" answer.
11. **A validated submission can be prose, not a grid — match what's asked.**
    The Nissan submission (2025-11-19) was a narrative response organised
    around the RFP's own "five key dimensions" (migration, integration,
    operating model, product vision, AI strategy) rather than a line-item
    requirements grid. Confirms standing lesson from `rfp-answer`'s own
    guidance: read the incoming document's own structure and vocabulary
    before drafting, rather than defaulting to a grid format.

12. **Answer in the format the question mandates.** An architecture-grade
    RFI often states a required artefact per question — *"Format:
    component-level responsibility matrix covering configuration,
    monitoring, incidents, capacity, security, recovery and cost"*,
    *"Format: failure-mode matrix showing retained functionality, cache
    behaviour, recovery steps and maximum content staleness"*. That line is
    a compliance requirement, not a suggestion: produce a matrix when a
    matrix is asked for, with **exactly the named columns**, and a sequence
    diagram, risk matrix or worked example when those are asked for.
    Answering a mandated matrix in prose loses points no amount of substance
    recovers. This is standing lesson from `rfp-answer` step 1 ("match the
    document's own vocabulary") extended from *wording* to *artefact shape*.

13. **When they send an architecture diagram, answer against their drawing,
    layer by layer, naming their components.** The validated response walked
    the prospect's own CDN / WAF / load balancer / compute / cache stack and
    said what the CMS does and does not touch at each layer. Describing our
    architecture instead makes the evaluator do the mapping — and they score
    what they can see.

14. **Split responsibility explicitly: us / them / their partner / their
    cloud.** A component-level RACI across configuration, monitoring,
    incidents, capacity, security, recovery and cost is the most reusable
    single artefact in the library. And **name the genuinely *shared* rows**
    (we sign the webhook, they verify it; we issue the preview token, they
    validate it server-side) — shared rows are what make the matrix read as
    honest rather than defensive.

15. **Name the gap, then the mitigation, in the same breath — never one
    without the other.** "No automatic webhook retry" alone is a scoring
    loss; "no automatic retry, so treat the payload as a notification, fetch
    current state on receipt, acknowledge in under a second and run a
    scheduled reconciliation job that bounds staleness to the poll interval"
    is an architecture answer. The same pattern applies to: no replay
    protection, no SIEM connector, no tested exit runbook, no scheduled
    unpublish, no deprecation flag. This is standing lesson 3 ("a gap is
    usually a path") at architecture scale — and lesson 5's "well-written
    partial" is the same instinct at requirement scale.

16. **"Not in scope of a CMS" is a legitimate classification.** Semantic
    markup, captions/transcripts and link-purpose quality were each answered
    that way in a validated accessibility matrix — paired with what the
    platform *does* contribute and who owns the rest. Three of six rows, and
    it strengthened the response rather than weakening it. Extends standing
    lesson 6.

17. **Correct an outdated premise rather than answering it.** One question
    referenced a "15-space target model"; the commercial scope had since
    moved to a 10-space rate-card structure, and the answer opened with a
    "Note on scope" saying so before answering. Evaluators reuse old drafts —
    answering a stale premise as written propagates the error into the
    contract.

18. **Label what is an assumption, out loud.** The validated response
    repeatedly wrote "working assumption, not a formally scoped number",
    "discussed on calls, not signed off", "illustrative sizing only, not
    contractual". This costs nothing, prevents a discussion note hardening
    into a commitment, and reads as rigour. Do it for every number that came
    from a call rather than a document.

19. **Consultative asides are worth more than the answer they sit next to —
    and they must be *specific*.** What landed in this submission: flagging
    that AI alt-text sends the *image itself* outside the managed-provider
    boundary given the prospect's sensitivity about its imagery; that their
    own timeline put migration too late and dry runs should start during
    the build; that a load test needs 15 working days' notice and should be
    arranged ahead of their known seasonal peak; that URL changes threaten
    organic rankings right before that same peak. Each is one sentence,
    unasked, and directly actionable. Look for the equivalent in every deal:
    where does *this* prospect's stated context collide with a real property
    of the platform or the plan?

20. **Worked examples beat descriptions.** This response used a worked
    failure example (timestamped, step by step, ending in a design
    implication), a worked deployment example (a paired schema + component
    change promoted region by region), and a worked cost example (editors ×
    hours × days × saves/hour × calls, with low/expected/high columns and an
    explicit "illustrative, not contractual" label). Each converts an
    assertion into something an evaluator can check.

21. **The GTC and Order Form are a source, and a distinct one.** Breaking-
    change notice periods, objection windows, termination remedies, service
    credits and overage all live there — not in the docs and not in the
    fact sheet. Cite the contractual floor as the commitment and any
    generous precedent as *observed practice on one change*, explicitly not a
    committed term. See `api-limits-webhooks.md` and `company-credentials.md`.
