# Operational telemetry, logging & SIEM

Source: 2026-09-15 architecture RFI (`validated`). Expands the "no
customer-facing infrastructure monitoring" honest limit in
`security-compliance.md` into the full matrix an enterprise ops or SecOps
reviewer will ask for.

**The shape of the honest answer: content-level auditing is strong,
operational/infrastructure telemetry is thin, and there is no native
alerting or SIEM connector anywhere.** Lead with the strength, name the
gaps precisely, then give the integration route.

## Telemetry matrix
| Domain | What's captured | Retention | Export | Alerting |
|---|---|---|---|---|
| **APIs** | Aggregate API Traffic Statistics (request counts, bandwidth) per space, by day/month/year across CDA, Management, GraphQL and webhook payload traffic. ⚠️ **Not per-request or per-IP logs** — that granularity is not a product feature | not separately published | Management API traffic-statistics endpoints (pull, JSON, date-filterable) | **none native** |
| **Asset / storage traffic** | Bandwidth and volume per space | not published | Management API endpoint | none native |
| **AI credit consumption** | Per space and per organisation | not published | Statistics API / in-app view | none native |
| **Publishing & content changes** | **Activity Log** — story/asset/user/workflow/datasource events per authenticated user | **unlimited on top tier** | Management API `/activities` (documented CSV-export pattern) or in-app | none native |
| **Webhooks** | Per-webhook delivery log: status, response, payload, per run | not published | ⚠️ **In-app only** — no dedicated export/API endpoint identified | none native; failed deliveries need manual retry |
| **Preview** | ⚠️ No dedicated preview telemetry — server-side preview fetches are invisible to us, rolling up only into aggregate API stats. Editor-side actions appear as ordinary Activity Log entries | as Activity Log | as Activity Log | none native |
| **Identity & admin activity** | Activity Log captures user management and role/permission changes made via dashboard or Management API | as Activity Log | as Activity Log | none native |
| **Infrastructure metrics** (CPU, memory, response time) | ⚠️ **Not exposed through any documented API or dashboard** | — | — | — |
| **Platform status** | Incident/maintenance across monitored components | live | email, Slack or Microsoft Teams subscription | ✅ **native** — subscribe for incident created/updated/resolved |

## The three gaps to name before a SecOps reviewer finds them
1. **SCIM-driven provisioning/deprovisioning is recorded in a separate
   internal audit trail and is *not* surfaced in the customer-facing
   Activity Log** or any customer-accessible endpoint. A prospect whose
   joiner/mover/leaver evidence must be auditable end-to-end needs to know
   this — their IdP holds that evidence, we don't expose ours.
2. **SSO/SAML sign-in events are logged on the IdP side, not duplicated in
   our Activity Log.** Each user record carries a **last-sign-in-date
   field — a snapshot of the most recent login, not a per-event sign-in
   history.**
3. **No correlation IDs anywhere.** Entries carry user, story, space and
   timestamp fields only; there is no request ID to stitch a CMS event to a
   downstream trace. Don't imply otherwise.

## SIEM export — the integration answer
There is **no native SIEM connector.** The supported path is pulling the
Activity Log via the Management API and shipping entries into the
customer's own SIEM with a custom integration — a **one-time engineering
effort on their side, with no additional licensing charge for the API
access itself.**

Design guidance worth volunteering, because it turns a gap into advice:
the Management API defaults to **6 req/s per space**, so the pull job
should be a **scheduled, incremental pull filtered by timestamp since the
last run**, not a high-frequency poll. That ceiling **can be raised on
request** for higher-throughput cases — a starting constraint, not a hard
one.

## Framing that survived review
Don't apologise for the infrastructure-metrics gap — explain the boundary.
In a headless architecture the customer owns the entire serving path
(their CDN, compute, cache, WAF), and that is where the latency and error
telemetry that matters to their users actually lives, fully instrumented in
their own tooling. Our surface is content operations, and on that surface
the audit trail is complete and exportable. The public status page plus the
uptime SLA are the customer-visible platform-health surface.
