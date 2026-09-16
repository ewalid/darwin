# Platform architecture & developer experience

Sources: validated RFP submissions, 2026-07-30 and 2025-11-19
(automotive/AEM-migration, multi-region enterprise). Trust: `validated`.

## Core
API-first/headless: REST Content Delivery API (primary), Management API,
official SDKs and webhooks on **all** tiers. The **GraphQL** Content
Delivery API is **Premium/Elite only** — don't present it as universally
available. Multi-tenant SaaS is the default, and there is still **no
mechanism for hosting customer application code** (see `integrations.md`
for the plugin-hosting limit and what it means for "move our app into the
CMS" requirements) — strong separation needs are normally met via separate
spaces + folder structure + fine-grained access control, not dedicated
infrastructure.

⚠️ **Correction (2025-11-19 validated submission): a BYOC option exists.**
An earlier note here said flatly "no dedicated single-tenant or
private-cloud option, no customer-controlled server." A later validated
submission offers two deployment models: **Public Cloud (SaaS)** —
standard, fully Storyblok-hosted — and **Customer-Controlled Cloud (BYOC)**
— full backend hosting inside the *customer's own* AWS account, still
managed by Storyblok; GCP or Azure hosting can be offered as an option too.
For a prospect with sovereignty/infrastructure-ownership requirements this
is a real answer, not a flat no — but treat it as an enterprise/custom-deal
capability to confirm with the account team before quoting, not a
self-service option.

## Multi-region / multi-brand architecture pattern (enterprise)
Recommended pattern for a large multi-market rollout: **one regional Space
per geography** (e.g. one Space per major region, shared across the
markets in it), with individual markets organized as **folders within that
Space**. A separate **dedicated dev/test Space** hosts centrally-built
components and structural changes, which are then deployed to the regional
Spaces via the CLI. The **Dimensions app** lets a Story's content be cloned
into other market folders while retaining market-specific overrides —
this is the mechanism for "global consistency, regional autonomy."
Component/feature availability can be scoped per-Space and per-market via
roles and permissions. Cite this whenever an RFP asks how the platform
supports multiple regions, brands or country-specific styling at once.

## Scaling — visibility answer
Backend systems run on redundant EC2 instances across multiple AWS regions
and availability zones via AWS Elastic Container Service, with load
balancing inside the VPC and auto-scaling for traffic spikes. Use this when
an RFP specifically asks for "visibility of scaling capability" rather than
just asserting "it scales."

## Transactional vs. non-transactional content — the architecture answer
Storyblok's content model can reference (not duplicate) external
transactional data: a component defines the API call/parameters needed, and
at request time the frontend unifies Storyblok's content/config with
live data from the customer's own systems (pricing, availability, user
info) via API calls — avoiding data duplication and keeping the CMS as the
single source of truth for the *marketing* content only. For pages that are
almost entirely transactional (e.g. a configurator or checkout flow), the
CMS may not be involved at all past a hand-off point in the user journey.
Frame the split as "CMS content vs. transactional data" rather than
implying the CMS stores transactional state.

## Token scoping for multi-market access control
Access tokens can be scoped to a region or folder (not just a whole Space),
letting a central team grant a market or agency editor access only to their
own folder/Story while enforcing global consistency centrally. Cite
alongside the RBAC/custom-roles material in `security-compliance.md` for
"how do you restrict who edits what by country/brand/market" questions.

## Framework integration
Official Next.js SDK with dedicated docs and Visual Editor support; large
production deployment base. Framework-agnostic beyond that — any front end
can consume the delivery API.

## Environments & dev workflow
Environments are modelled as **separate spaces** (e.g. dev / staging /
production), promoted via CLI/Management API. **Self-service plans include
exactly 1 space**, so any real environment requirement implies
Premium/Elite — state this rather than describing environments as freely
available. **Pipelines / content staging is also Premium/Elite only**
(5 stages on Premium, unlimited on Elite). Page History gives content
versioning out of the box. Schema-as-code (TypeScript, version-controlled)
covers content-model-in-CI.

**Be precise about the CI/CD boundary:** app-code CI/CD lives in the
customer's own repo and hosting pipeline, not in the CMS; the CMS provides
webhooks to trigger external builds. Answering "Supported out of the box"
for CI/CD overstates it — "Supported with configuration" plus this
boundary sentence is the accurate answer.

## Hosting
- **Vercel** — official integration, available on all plan tiers,
  dedicated docs, production customers to cite. High confidence.
- **Containerised hosting (GCP Cloud Run, etc.)** — architecturally
  compatible since the CMS is API-first and hosting-agnostic, but **no
  official partnership or dedicated integration.** State the confidence
  difference between these two plainly rather than blurring them; a
  prospect naming both will notice.

## Extensibility
Official App Directory + first-party Plugin SDK, webhooks, Management API.
Hosted MCP Server shipped for agentic/programmatic workflows.
**Plugin gating: Field Plugins on all tiers; Space Plugins and Tool
Plugins are Premium/Elite only.**

---

# Added from the 2026-09-15 architecture RFI (`validated`)

> The multi-region note above is superseded for large estates by
> **`multi-space-governance.md`**, which carries the full topology,
> propagation, compatibility-gate and blast-radius answers. API quotas,
> caching and webhook behaviour now live in **`api-limits-webhooks.md`**.

## "Does your CMS fit our target architecture?" — the answer shape
When a prospect sends their own architecture diagram, **answer layer by
layer against their drawing, naming their components**, rather than
describing ours. The validated opener: *"[Product] fits the proposed
architecture as designed"*, then a bullet per layer — frontend autonomy,
content APIs, SDKs and tooling, delivery services, cache invalidation,
visual editor, third-party integration, identity, managed operations,
deployment model, API connectivity, licensing scope, commercial scope.

The two closing bullets matter as much as the technical ones:
- **Licensing scope** — call out which elements of *their* diagram are
  licensing choices rather than technical limitations (SSO, GraphQL, custom
  roles and approval workflows, unlimited locales, the higher SLA tier), and
  confirm they're already in what's quoted.
- **Commercial scope** — API request volume and traffic are contracted per
  plan, so their traffic model should be sized against the agreed quota;
  their hosting, build, egress and third-party services remain their costs.

## Responsibility boundaries — the RACI answer
Enterprise architecture RFIs ask *"define your responsibility boundaries"*
and expect a **component-level matrix across configuration, monitoring,
incidents, capacity, security, recovery and cost** — one row per component,
naming the customer, their implementation partner, their cloud provider and
us separately.

We own all seven columns for exactly **four components**: the **Content
Delivery API, Management API, our CDN, and the Visual Editor / preview
bridge (as a product)**. Everything from the customer's webhook endpoint
outward — their revalidation service, hosting stack, cache layer and ops
tooling — is theirs.

Three rows are genuinely **shared**, and saying so is what makes the matrix
credible rather than defensive:
- **Preview bridge security** — we issue the signed preview parameter; **they
  must validate it server-side.**
- **Webhook signing** — we sign; **they verify and handle replay protection.**
- **Identity** — they own their IdP tenants and any cross-tenant sync; we own
  the single org-level SSO connection and RBAC enforcement.

Also split **webhook emission** (ours: fired, signed, logged) from **webhook
receipt, queueing, retry and reconciliation** (theirs — *there is no
automatic retry from our side*). Collapsing those two into one row is where
responsibility answers go wrong.

## Outage behaviour — the failure-mode matrix
"What still works if X is down" is a standard enterprise question and the
answer is strong, provided one distinction is made honestly:

| Unavailable | Customer-facing | Editorial |
|---|---|---|
| **Content Delivery API** | ⚠️ **ISR and pre-rendered routes keep serving** from cache. **Pure SSR routes call the CDA on every request and will fail or need a stale fallback.** This distinction is the whole answer — do not blur it | Draft authoring and publish still succeed (Management API is separate); new content just doesn't reach visitors yet |
| **Management API** | Fully unaffected | No save/publish; unsaved edits must be re-entered |
| **Our CDN** | Warm edge keeps serving; otherwise falls through to origin — slower, functional | Unaffected |
| **Asset / image service** | Page content renders; uncached images fail; new transformations fail. **Not a staleness case — a binary availability gap.** Self-heals on restore, no republish needed (asset URLs are stable) | Upload/browse degraded |
| **Visual Editor** | Fully unaffected | WYSIWYG lost, **standard form-based editing remains fully available**, and the Management API is reachable directly by script/CLI |
| **Preview bridge** | Fully unaffected | Live in-context sync stops; authors can still save, publish and view the preview deployment by URL |
| **Webhook service** | Site keeps serving last-cached content | ⚠️ Publish succeeds but the revalidation trigger never arrives — **staleness is unbounded in the worst case** unless a time-based revalidate interval or a reconciliation job exists as a safety net |

The webhook row is the one that needs the mitigation attached (see
`api-limits-webhooks.md`); leaving it as a bare "unbounded" is a scoring
loss, and hiding it is worse.

## Environment topology — what is a space and what isn't
- **Development** and **Production (one per region/brand group)** are
  long-lived **spaces**.
- **Staging is not a separate space** — it is an **Environment layered on
  top of Production**, cloned from Production's current content and
  component structure at creation, persisting indefinitely. It rehearses a
  change against production-like content.
- **Preview is not a space either** — it is content fetched in the
  framework's preview mode via a Preview token, mixing draft and published
  content. **Content-only staging, distinct from schema staging.**

See `multi-space-governance.md` for the Environment licensing trap (**one
per organisation, not per space**) and the promotion workflow's known
limitations.

## Honest platform boundaries worth stating upfront
- **API connectivity is public HTTPS with token-based auth** (configurable
  token scope and expiry). ⚠️ **Private connectivity — VPC peering,
  PrivateLink — is not part of the standard offering.** A security-led
  enterprise will ask; answer it directly.
- **No first-party CI/CD product.** The CLI and Management API are the only
  two mechanisms for driving schema/config state programmatically; the
  customer's own pipeline wraps them.
- **Custom domains per production space** are achievable but are a
  **customer-built CDN layer pointed at our origin**, not an automatic
  per-space feature. Configured in space settings at no additional platform
  cost; DNS cutover is theirs.
- **A space's region is fixed at creation and content cannot be moved
  between regions afterwards.** The most consequential irreversible decision
  in the whole topology — surface it during scoping, not after.
