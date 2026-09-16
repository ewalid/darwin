# Multi-space governance — schema-as-code, propagation, blast radius

Source: 2026-09-15 architecture RFI (`validated`). This supersedes the
shorter multi-region note in `platform-architecture.md` as the canonical
answer for large multi-brand / multi-market estates.

## The topology answer
Two concerns are separate and must be explained as such:
- **Space** = the unit of schema, permissions, workflow, tokens and release
  cadence. A strong isolation boundary.
- **Folder** = the unit of content structure *inside* a space.

Recommended shape for a large estate: **one "Shared Component Library"
space holding canonical components, content types, validation rules and
datasources (no published content of its own), plus one space per
region-group, with brands as folders inside.** Each grouping carries a
production + non-production pair. A five-grouping estate therefore lands
around **10 spaces**, not one per brand.

Why region groups beat brand-per-space: a space's **region is fixed at
creation and content cannot subsequently be moved between regions**, so the
region boundary is the one that is genuinely irreversible. Brand is not —
brands can be folders. Lead with the irreversible constraint.

Escalate a brand to its own space only where it needs **independent access
control** — a market on dedicated infrastructure, or an acquisition not yet
integrated.

## What propagates, and by which mechanism
| Artefact | Mechanism | Crosses regions? |
|---|---|---|
| Component schemas, content types, validation rules | `storyblok schema push`, or `storyblok components push --space <target> --from <source>`, driven from CI | Yes — API-based, region-agnostic |
| Controlled vocabularies (age bands, codes, labels) | **Datasources**, synced via CLI | Yes |
| Design tokens | Datasources of token *names* bound to a select field | Yes |
| Workflows, stages, roles, permissions | Management API (`/workflows`, `/workflow_stages`, `/space_roles`) — configured per space, **not natively distributed as one cross-space definition**, so scripted from the repo and applied per space | Yes, when scripted |
| Shared **content** (group nav, group-wide policy pages) | **Multi-Space Content Distribution** — clone / overwrite / merge from a source space, with Content Association linking versions | ⚠️ **Within a single region only** |

⚠️ **Multi-Space Content Distribution is same-region only, does not clone
assets, and runs no automated compatibility checks.** Frame it as a *copy
tool, not a migration or sync pipeline*. For a genuinely cross-region
estate, shared content has to be handled another way — say so rather than
implying the feature covers it.

## Design tokens — the positioning that won
**Do not offer the CMS as the source of truth for design tokens.** Tokens
are not a CMS primitive. The validated answer: tokens live in the design
system repository and are consumed by the frontend; the CMS's role is to
expose the **brand-safe subset to editors as constrained options** — a
datasource of token names bound to a select field, so an editor can choose
"brand primary" but cannot type an arbitrary hex value. Each brand's token
set is applied at render time from a brand configuration story, which is
how *one* shared component library produces visually distinct brand output.

This is a strong answer precisely because it declines scope. Reuse the
shape whenever an RFP asks the CMS to own something it shouldn't.

## Schema-as-code — the CLI/API vocabulary
Schemas defined as typed TypeScript via the **`@storyblok/schema`** package
(`defineField()`, `defineBlock()`, `defineDatasource()`), held in a
**customer-owned git repo**. The spaces are downstream artefacts of that
repo. Commands worth naming precisely:

- `storyblok schema validate` — catches a malformed definition
- `storyblok types generate --space <id>` — TypeScript types from schema
- `storyblok schema push --dry-run` — exact diff, no writes
- `storyblok schema push` / `storyblok schema rollback`
- `storyblok migrations generate` / `migrations run --dry-run` /
  `migrations run` / `migrations rollback` (rollback files written under
  `.storyblok/migrations/<space-id>/`; an automatic pre-run snapshot is
  taken as part of the migration process)
- `storyblok components|datasources|stories|assets pull`

## The compatibility-gate ladder (reusable verbatim)
The real risk in a multi-space estate is **not drift between spaces — it is
drift between the content schema and the frontend component that renders
it.** A schema change every space accepts still produces a broken page if
the deployed component expects the previous shape. Because schema
definitions and the component library live in the same repo, that check
happens at build time:

1. `schema validate` — malformed definition
2. `types generate` — TS types from the new schema
3. **TypeScript compilation of the application — the decisive gate.** Any
   component reading a renamed or removed field fails the build
4. `schema push --dry-run` — exact diff of what would change, no writes
5. Push to the shared library space; run application build + E2E suite
6. Pilot region's non-production space, then its production space

Then classify the diff as **additive** (new optional field — safe),
**mutating** (rename, type change, tightened validation — needs a content
migration) or **destructive** (field/component removal — needs a
deprecation cycle).

## Expand-then-contract — the destructive-change discipline
A destructive schema change is **never pushed in one step**:
expand (add the new field, both shapes valid) → migrate the content →
deploy the frontend reading the new field → contract (remove the old field,
once no space still references it). Each phase is independently reversible,
and at no point is a deployed component reading a field that no longer
exists. Policy sentence worth quoting: *deprecate, migrate, then remove in
a later release — never remove a field in the same change that stops using
it.*

## Blast radius — the isolation matrix
| Risk | Isolation unit | Real blast radius |
|---|---|---|
| Erroneous publication | Story, then Space | One story, one space. Content never crosses spaces on its own |
| **Breaking schema change** | The owning space **and every space it is propagated to** | The genuine weak boundary — inherent to centrally governed components. This is *why* the staged-rollout ladder exists |
| Compromised editor credential | User account, scoped by role | Bounded by role — scopable to space, folder, story, language, workflow stage, block field, datasource, asset folder on top tiers |
| Compromised **delivery** token | Token type + space | Public token: published content, one space. Preview token: + drafts, one space. **Neither can write** |
| Compromised **Management API** token | Token scopes + selected spaces | Highest-severity category. Since the 2026-05 scoping change, bounded to granted spaces/scopes rather than everything |
| Excessive consumption | Per space, against the CDA | 429 for **that space only** — does not throttle siblings |
| Accidental space deletion | The Space | Privileged action, not available to Editor-level roles. Restore from backup (mind the documented exclusions) |
| Regional infrastructure failure | The space's region | Primarily authoring/Management API traffic; already-built ISR/pre-rendered pages keep serving. Pure-SSR routes are the exception |

Underlying infra to cite: content in **PostgreSQL on RDS, real-time
replicated to a second AZ (RPO ≈ 0, RTO 15 min), plus daily backups shipped
to a separate AWS region**. Note honestly that for *logical* corruption the
replica reflects the corruption too, so recovery is a **point-in-time
restore to a new instance, not a failover** — estimate ~1 hour, and say
the exact RPO/RTO is not published.

## Rollback — five independent levels
| What needs reverting | Mechanism |
|---|---|
| A schema change | `storyblok schema rollback`, or revert the commit and re-push |
| A content migration | `storyblok migrations rollback` from the written rollback files |
| An individual story | Version history restore (unlimited retention on top tier) |
| A component definition | Restore a previous component version via the Management API |
| The frontend | Standard deployment rollback — customer-owned, CMS-independent |
| An entire space | Backup restore — **mind the exclusions** (see `exit-portability.md`) |
| **An entire published release, as one unit** | ⚠️ **Not available as a single action.** Per-story version restore is the only mechanism |

**One architecture dependency worth stating:** after any schema or content
rollback, revalidation must be issued **through a shared cache handler** so
it reaches every application instance, not only the one that received the
request. Without that, a rollback appears to have partially succeeded.

## Component lifecycle & design-system enforcement
| Capability | Mechanism |
|---|---|
| Restrict which components an editor can insert | Allowlist on a Blocks or Richtext field — per field, inherited everywhere reused |
| Restrict rich-text formatting | `customize_toolbar` + explicit permitted-elements array; markdown can be disabled outright |
| Field-level validation | `required`, `max_length`, min/max item counts, number ranges — enforced **including on the Management API** |
| **Deprecate a component** | ⚠️ **No native "deprecated" lifecycle flag.** Pattern: remove from the allowlist(s) so it can't be inserted, plus an Internal Tag / `zz_deprecated_` naming convention as a visible signal. A convention the customer enforces, not a platform control |
| Discover usage | `GET /spaces/{id}/stories?contain_component={name}` — every story using it |
| Bulk-migrate off it | `storyblok migrations run` scoped to the component; auto-snapshots first; `--publish` controls auto-republish |

⚠️ **Sharp edge worth naming before a prospect designs around it: `regex`
pattern validation is enforced in the Visual Editor only — it is *not*
enforced on the Management API.** A scripted bulk import can therefore
write content that violates a regex rule. Every other listed validation
rule *is* enforced on the API. This is exactly the kind of precise partial
that earns credibility.

## Environments vs. spaces — don't conflate them
- **Environment** = a sandbox layered on top of an existing space, cloned
  from that space's current content and component structure at creation,
  persisting indefinitely. For rehearsing a change against production-like
  content.
- **One Environment is included per organisation, not per space.**
  Additional Environments are priced separately. State this — it is a
  common and expensive misreading.
- An Environment is **not** a substitute for a dedicated non-production
  space.
- ⚠️ An Environment copies content and component structure at creation but
  **not installed apps or space settings** — those need recreating for a
  fully representative test.
- There is **no UI tool to promote a change from an Environment to
  Production** — the same reviewed script/migration is run against each
  production space.

## Acquisition / new-brand onboarding — the scenario answer
The question "how fast can we onboard an acquired brand" splits cleanly:
- **Folder inside an existing space:** adds close to nothing on the platform
  side — existing space, existing roles, existing design system. Identity
  1–3 days, domains same day, roles hours, design system on the normal push
  cycle.
- **Its own space:** roughly **1–2 weeks**, driven by content complexity and
  testing, not by platform setup.
- In both cases **content migration from the legacy CMS is the dominant
  timeline driver**, not anything on our side. Say so — it sets an honest
  expectation and moves the conversation to scoping.
- Platform cost is **$0 either way within the contracted envelope**; it only
  becomes commercial if it pushes past the contracted space count.
