# Exit, portability & reconstruction

Source: 2026-09-15 architecture RFI (`validated`). Distinct from
`migration.md`, which covers migrating *in*. This covers the
exit/lock-in/reversibility questions that procurement and architecture
reviewers ask, and that a headless CMS should win comfortably — provided
the gaps are named rather than glossed.

## The ownership statement (lead with this)
**The customer owns and controls all of it.** The platform stores the live
current state inside each space, but the portable, versioned source of
truth exists in **code, in a customer-controlled git repo**. There is **no
proprietary template language, no compiled artefact only we can read, and
no server-side rendering logic** — everything is plain JSON or TypeScript,
retrievable through a documented endpoint or CLI command.

That sentence does more work than any feature list. Use it verbatim.

## Ownership / export matrix
| Artefact | Format | Exported via | Notes |
|---|---|---|---|
| Stories (content) | JSON, nested block tree | S3 Backups app, `storyblok stories pull`, Management API | Includes translated slugs, releases, scheduled content |
| Component schemas & validation rules | JSON (`components.json`) or typed TS via `@storyblok/schema` | S3 Backups app, `components pull`, the customer's codebase | Includes component groups, presets, tags |
| Datasources (incl. design-token references) | JSON or CSV | CLI `datasources pull`, CSV import/export | |
| Roles | JSON | Management API | |
| Workflows | JSON | S3 Backups app, Management API | ⚠️ **Not available via CLI pull commands** — S3 Backups is the confirmed export path |
| **Assets (binaries)** | original files | `storyblok assets pull`, or the community Assets Backup Script | ⚠️ **The S3 Backups app stores only asset *references*, not binaries** |
| Redirects | JSON, as an ordinary story/datasource | same as stories | ⚠️ No first-class redirect object — **must be modelled from day one** as a content type/datasource of source→target pairs. A build requirement, not an exit-time feature |
| Audit / activity history | JSON via Management API `/activities`, convertible to CSV | scripted export | Not a one-click UI export; paginated at 100 items/page |
| References between stories | UUID references embedded in the content JSON | same as stories | Must be remapped under a different space (new UUIDs) — our own migration guidance recommends a **lookup table mapping legacy IDs to new UUIDs** |
| Custom editor config (field/tool plugins, space apps) | JS/HTML/CSS bundle | standard frontend codebase; we host only the manifest and runtime | Customer or partner repo |
| Deployment configuration | CLI invocations in a pipeline definition | n/a — pipeline code the customer writes | |

## ⚠️ S3 Backups app — the exclusions, named
**Explicitly excluded: story version history, presets, space settings,
users, and actual asset binaries.** These require the CLI/Management API
path instead. Name the exclusions proactively; a prospect who discovers
them during an exit dry-run will remember that you didn't.

## Rate / time estimates
- Management API defaults to **6 req/s per space** (raisable on request for
  a scoped export effort) — this bounds schema/config/audit export.
- **Already-published content pulls far faster via the CDA** (up to ~1,000
  req/s for cached responses). Route the bulk of an export there.
- Dependencies: Node.js + the CLI (open source, free) and a Management API
  token with read scopes. For S3 Backups, an S3 bucket in the customer's own
  account — support can configure this for them at additional cost rather
  than requiring them to stand up S3 themselves.

## Costs
CLI and Management API access are **free and included, with no per-call or
bandwidth surcharge documented.** The S3 Backups app is included with daily
backups — **no separate licence fee, only the customer's own S3 storage
cost.** Commercial terms confirm all content/assets/schemas/configurations
are exportable at no additional cost, with exit/migration support included.

## ⚠️ The honest gap — and how to turn it into a deliverable
**There is no pre-assembled, dry-run-validated exit runbook.** The
capability exists today as *separable tools*, not as one documented
end-to-end procedure. Saying so costs nothing and buys credibility; then
propose building and dry-running one jointly before go-live:

1. Provision S3 + IAM role; enable S3 Backups (daily) as the running
   structural backup of stories/schemas/datasources/roles/workflows.
2. Schedule `assets pull` (or the Assets Backup Script) for asset binaries,
   then `components` / `datasources` / `stories pull`.
3. Execute a **full dry-run reconstruction into a scratch space**:
   components push → assets push → datasources push → stories push,
   remapping legacy UUIDs via a lookup table, then manually re-create roles
   and workflows.
4. **Time and validate actual throughput against the customer's real story
   and asset counts**, replacing the theoretical estimate with a measured
   one.

Offering step 4 unprompted is what makes this read as an operating
commitment rather than a feature answer.

## Continuous-ownership practice (worth volunteering)
Nothing in the platform enforces this automatically, so say it plainly:
treat running `components pull` / `datasources pull` / `stories pull` into
the customer's own repo **on every schema change** as standing practice.
That way the exit position is continuously true rather than reconstructed
under pressure.

## SDK / lock-in inventory
Typical footprint in a React/Next.js build: `@storyblok/react` (data
fetching, richtext rendering, component mapping), `@storyblok/js` (API
client, bundled via the above), the **Bridge script (preview-only, no
production footprint)**, the `cv` cache-versioning parameter, and the asset
CDN's image-transform URL convention.

- **Licence: all packages are open source under MIT.**
- ⚠️ **No formal LTS or deprecation-window commitment is published for these
  packages** — say so; the API-level commitments in
  `api-limits-webhooks.md` are the real contractual floor.
- **Replacement effort:** content is delivered as documented JSON, so none
  of it is structural lock-in. The API client is a thin wrapper replaceable
  with a standard `fetch`. **Richtext rendering is the most substantial
  piece to replace — comparable in scope to any structured rich-text CMS.**
  The Bridge is preview-only. Component mapping and URL conventions are
  application code and migrate with the application.

That last paragraph is the model answer for a lock-in question: name the
one genuinely non-trivial item, size it against the category rather than
against zero, and dismiss the rest with reasons.
