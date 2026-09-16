# Migration

Sources: validated RFP submissions, 2026-07-30 and 2025-11-19
(large-scale AEM→Storyblok, automotive). Trust: `validated`.

## Real tooling — name it, don't describe it generically
- **Official WordPress importer** exists as a public repo starting point.
  Check for an official importer for the prospect's *current* CMS before
  describing migration as bespoke scripting.
- **CLI v4** + Management API: scripted, re-runnable jobs with dry runs,
  idempotency guarantees, and manifest-based reference mapping — designed
  exactly for the "multiple rehearsal passes before cutover" pattern most
  RFPs require.
- **Schema-as-code package**: content model defined in TypeScript,
  version-controlled, CI/CD-friendly, pushed repeatedly to a staging space
  for validation. Strong answer for "scripted and repeatable."
- **Figma plugin** — generates Block (component) schemas directly from
  Figma designs, speeding up the "design system refresh" portion of a
  component migration. Cite for "how do you migrate/rebuild components"
  questions.

## Timeline-setting — a real number, honestly caveated
A validated submission for an enterprise-scale AEM migration stated
**12–16 months** for a migration of that size, while being explicit that
Storyblok **cannot guarantee a full timeline** because it depends mostly on
the resources the implementation partner commits. Use this framing rather
than inventing a number: give a range tied to a comparable scale, and
attribute the variability to partner resourcing, not to the platform.

## Distinguishing "coupled" vs "decoupled" legacy code — determines the route
A validated answer categorised the prospect's existing frontend code into
three types, which is a reusable way to scope any migration's code-side
effort:
1. **Legacy code tightly coupled to the old CMS's templates** — automated
   migration is impossible; it needs full decommission and rewrite against
   the new decoupled frontend (React/Vue/Next.js) consuming the Content
   Delivery API, with business logic extracted into separate microservices.
2. **Existing decoupled code (e.g. micro-frontends already served via
   APIs)** — does **not** need a rewrite, only an integration: build an API
   adapter that swaps the old CMS's calls for Storyblok's Content Delivery
   API, remap data consumption, and add the Storyblok Bridge for Visual
   Editor support.
3. **New frontend code built from scratch** — built directly against
   Storyblok's API using a modern headless stack, no legacy constraint.
Naming which bucket a piece of code falls into is more credible than a
blanket "will require custom development."

## Automated vs. manual migration — the criteria, not just the split
- **Automated-migration criteria:** standard/well-defined content types
  with a consistent, predictable structure; structured single-value data
  fields (titles, dates, short text); and — critically — a stated
  assumption that source data is relatively "clean" (limited unstructured
  HTML embeds, limited inconsistent formatting).
- **Manual-migration criteria:** complex/highly-custom content with
  non-standard structure or deeply nested relational data; content flagged
  during audit for data-quality/governance issues; and high-value,
  high-risk assets (e.g. checkout flows, regulatory pages) where an error
  has real business impact — these get manual verification even if they
  *could* technically be automated.
Naming these criteria (not just "some things are automated, some aren't")
reads as a real methodology rather than a hedge.

## Vendor-vs-customer responsibility split (RACI-style) — reusable answer
When an RFP asks "who does what" during migration/implementation:
- **Storyblok's responsibilities:** platform provisioning and maintenance
  (features, updates, infrastructure availability); technical guidance via
  Solutions Engineering to the customer's chosen implementation partner
  (architecture, best practices, issue resolution); documentation and
  training for both the customer's internal teams and their partner.
- **Customer / implementation-partner responsibilities:** all development
  and integration work (frontend build, backend integration, custom
  feature configuration); project management and QA of the final solution;
  and ongoing content management once live.
State this split explicitly rather than letting "who builds what" stay
implicit — it heads off a later dispute about scope.

## Phased multi-region rollout — a pattern worth reusing
For a large multi-market rollout, the validated recommendation was to scope
year one around a small number of regional Spaces (e.g. three) plus one
dedicated development Space, rather than all regions/markets at once. Benefits
named: concentrates resources, lets the team refine the operating model and
gather feedback before a wider rollout, minimises risk, and gives a proven
model to replicate. Full rollout (e.g. ten total Spaces) follows once the
model is validated. Use this shape whenever a prospect's own rollout could
plausibly be phased rather than big-bang.

## Methodology skeleton
1. Discovery & content audit — full page/template/content-type inventory,
   localised variants, URL audit against current sitemap + analytics.
2. Content model design — translate the old flexible-content structures
   into components; proof-of-modelling on one representative page first
   (this usually maps directly to the RFP's own demo requirement).
3. Scripted migration — importer/CLI/API, multiple dry passes before
   cutover, sized against the stated freeze window.
4. Redirects & SEO preservation — 301 map every indexed URL, metadata and
   structured-data parity checks, CWV validation, analytics QA.
5. Launch & hypercare — DNS cutover, defined hypercare window, decommission
   the old platform.

## Content freeze — don't just call it "not a platform feature"
It *is* a process commitment, but the platform can enforce it operationally:
lock editors to a read-only workflow stage, or temporarily restrict
publishing permissions via role config for the freeze period. Answer
**Supported with configuration**, not "Not supported."

## Risk mitigation plan
Owned by the implementation partner as a methodology deliverable — but
enumerate the platform's contributions rather than deferring wholesale:
version history + content rollback, automated daily backups with
point-in-time restore, hot-standby replica for HA, published incident
response process.

## Consultative flag worth making
If a prospect's timeline puts content migration in a single late phase,
recommend beginning dry-run migrations during the *build* phase instead —
it surfaces content-model problems early. Flagging a realistic risk in
their own plan reads as expertise, not obstruction.

---

## Added from the 2026-09-15 architecture RFI (`validated`)

> Exit / reversibility / lock-in questions now have their own file:
> **`exit-portability.md`**. This section covers migrating *in*.

- **Model the target schema once and propagate to every space via
  CLI/Management API *before* content migration begins**, so every
  brand/region space starts aligned. Getting this order wrong is the most
  expensive mistake in a multi-space migration.
- Bulk-import legacy content per space via the CLI. **`@storyblok/migrations`
  handles HTML/Markdown-to-richtext conversion and reference remapping** —
  name the package; "we have migration tooling" is weaker than naming it.
- For external translation-provider round-tripping: **Export Translatable
  Fields (XML/JSON) → provider → Import Translatable Fields** (or the
  scripted Management API equivalent).
- Once populated, **Multi-Space Content Distribution** pushes shared content
  out to sibling spaces, which then diverge locally. ⚠️ Same-region only,
  no assets, no compatibility checks.
- **Throughput ceiling:** bulk publish is bounded by the **Management API's
  6 req/s per space** — raisable on request for a scoped migration. Batch
  migrations to respect it and run them outside peak hours. Quote this when
  asked for a migration timeline; it is the real constraint, not tooling.
- **Reference remapping:** stories reference each other by **UUID embedded in
  the content JSON**, and UUIDs change under a different space. Our own
  migration guidance recommends a **lookup table mapping legacy IDs to new
  UUIDs** — say this before a customer discovers it mid-cutover.
- **Redirects must be modelled as content from day one** (a content
  type/datasource of source→target pairs). There is no first-class redirect
  object, so this is a build requirement, not something retrofittable at
  cutover. Pair it with the **301 map built from the legacy sitemap and
  server logs** (see `seo-ai-discoverability.md`).
- ⚠️ **Webhooks do not always fire reliably on direct scripted/bulk
  Management API writes** (see `api-limits-webhooks.md`). Migration pipelines
  should either use the standard publish action or handle revalidation
  separately — otherwise a migrated site looks stale for no visible reason.
- **Consultative move that landed:** when the prospect's own stated timeline
  puts migration too late, say so, and recommend **dry runs starting during
  the build phase** rather than after it. Flagging a risk in *their* plan
  that the RFP never asked about reads as expertise, not scope creep.
