# API limits, caching & webhooks

Source: 2026-09-15 architecture RFI (`validated`) — the deepest technical
submission in the library. Cross-check quotas against the current public
docs/fact sheet before quoting; rate limits are the kind of number a
technical evaluator will hold you to.

## Rate limits — the table to reuse
| API | Normal limit | Notes |
|---|---|---|
| Content Delivery API — **cached** (current `cv`, from CDN) | up to **1,000 req/s** | virtually all production traffic should land here |
| CDA — uncached, single story or listing ≤25 items | 50 req/s | |
| CDA — uncached, listing 26–50 | 15 req/s | limit scales *down* as page size grows |
| CDA — uncached, listing 51–75 | 10 req/s | |
| CDA — uncached, listing 76–100 | 6 req/s | |
| Preview (draft) requests | same ceilings as uncached CDA | draft is never CDN-cacheable |
| **Management API** | **6 req/s** | governs migration tooling, content-ops scripts, SIEM pulls, bulk publish. Can be raised on request — a starting constraint, not a hard one |
| GraphQL | 100 points/s, query-complexity weighted; cached queries cost 0 | |
| Image Service | no separate published ceiling; resize caps at 4,000×4,000px | first request generates + caches the variation |

All APIs return **429** on breach. Official guidance is client-side
exponential backoff; **no `Retry-After` header is documented** — the
first-party JS client retries on a fixed 300ms delay up to 10 times.

**No "Search API" exists** — filtering runs through the CDA's
`filter_query`/`search` params or GraphQL, under the limits above.

## Hard ceilings worth naming
- Max story JSON response: **5 MB** (all plans).
- Resolvable links per request: **500** (CDA v2) / **100** (GraphQL).
- Listing page size: default 25, **max 100** (matches the rate bands above).
- Max asset upload: **5 GB** on enterprise packages.
- Platform ceiling: **250 spaces per organisation** — well above realistic
  enterprise estates, so it is almost never the real constraint; what the
  customer contracts for is.
- **Publishing throughput is bounded by the 6 req/s Management API limit.**
  That is the practical ceiling on bulk-publish and migration tooling speed.

## Latency — the honest answer
**No per-endpoint p50/p95/p99 percentiles or benchmarking methodology are
published.** The only externally committed performance figure is the uptime
SLA. Two routes to a number a prospect can actually rely on, and this
framing has survived review:
1. In an SSG/ISR architecture the latency that matters to an end user is
   *their own CDN's* edge latency on an already-built page — measurable by
   the customer, largely independent of us. Our API latency only matters at
   build/revalidation time.
2. The customer can run **their own load test against our infrastructure**
   under an extended support package, coordinated **at least 15 working
   days in advance** with the infrastructure team. Offer this proactively
   when the prospect has a known seasonal peak.

## The `cv` cache-version mechanism — reusable explanation
Story JSON is cache-partitioned by the **`cv` (cache version) query
parameter, not by a `Cache-Control` TTL**. Requests carrying the current
`cv` serve straight from the CDN (the ~1,000 req/s path); a stale or
missing `cv` is transparently redirected to the current version. Publishing
**advances the space's `cv` automatically — there is no manual purge step.**

Treat `cv` as part of the *application's* cache key, not something left to
the HTTP layer. Recommended fetch key shape:
`storyblok:{space_id}:{slug}:{locale}:{cv}`.

Consequences to state plainly:
- Each space has its own `cv` counter — a natural top-level cache partition,
  one brand's cache cannot leak into another's.
- **Locale is field-level translation within a space, so it must be added
  explicitly to the application cache key.**
- Omit `cv` and you drop from ~1,000 req/s (cached) to 6–50 req/s (uncached,
  hits origin). This is the single most consequential implementation detail
  in a high-traffic build.
- Asset URLs are **versioned/content-hashed** — a replaced asset gets a new
  URL, so assets are cacheable up to a year with no purge step.

## Our CDN sits *parallel to*, not behind, the customer's CDN
In a headless build there are typically four independent cache layers:
browser → customer's CDN (their rendered HTML) → the framework's data
cache / ISR output (inside their compute) → **our CDN (raw CDA responses
and assets, fetched server-side during render)**. Our CDN is never proxied
through theirs and is invisible to it. Say this explicitly — evaluators
routinely assume the two are stacked, and the invalidation story only makes
sense once they aren't.

Corollary for cost questions: because content is fetched **server-side at
render/regeneration time**, visitor traffic volume does not drive API
consumption — only **publish frequency and editor activity** do. This
reframes "what happens at our traffic peak" from a scary question into a
non-issue, and it is worth modelling as a worked calculation (editors ×
hours/day × working days × saves/hour × calls/save, plus a buffer) rather
than asserted.

## Webhooks — capabilities and the gaps, stated honestly
This is the area where the validated answer wins points by being blunt.

| Property | Behaviour |
|---|---|
| Signing | `webhook-signature` header = **HMAC-SHA1 of the raw, unparsed body**, keyed with the endpoint's configured secret. Verify with constant-time comparison |
| Payload | Lightweight event metadata only (`action`, `space_id`, `story_id`, `full_slug`, a text line) — **not** full story content |
| Secret availability | Part of webhook config on enterprise packages; **task-triggered webhooks (fired via the Management API rather than a real content event) do not support a secret** |
| Network hardening | **Static egress IPs published per region** for WAF/network allowlisting — available on request. Name this; it is a genuine enterprise reassurance |
| Delivery guarantee | **Single attempt, at-most-once per endpoint. No automatic retry** on non-2xx, timeout or connection error |
| Ordering | **No cross-event ordering guarantee** — a rapid unpublish-then-publish can arrive out of order |
| Idempotency | No built-in key beyond `story_id`/`space_id`. Multiple configured endpoints each receive independent deliveries — expected, not a bug |
| Replay protection | **None built in.** The signature covers only the body — no timestamp, no nonce — so a captured (request, signature) pair stays valid indefinitely |
| Secret rotation | Updatable via Management API; **no documented rotation procedure and no dual-secret overlap window** — a change takes effect on the very next delivery |
| Timeout | 120s |
| Delivery history | Settings → Webhooks → View Logs: status, response body, full payload per run. **In-app only — no export endpoint identified** |
| Replay | **No one-click resend.** Force redelivery by re-triggering the publish action, or script it against the Management API |

**The mitigation pattern that turns all of the above into a good answer —
reuse this wording:** treat the payload as a *notification, not data*. On
receipt, verify the signature, then fetch the current story and `cv` from
the API and act on that. This makes a replayed payload an idempotent no-op
and removes the ordering problem at the same time. Pair it with: the
receiving endpoint should acknowledge fast (**return 202 immediately,
process asynchronously**) so a slow handler is never misread as a failure,
and a **scheduled reconciliation job** comparing the space's `cv` against
the last-built value bounds worst-case staleness to the poll interval
rather than leaving it open-ended.

⚠️ **Known sharp edge:** the webhook fires reliably from the standard
publish action (Visual Editor, or the ordinary Management API publish
endpoint used through the editorial flow). **Content pushed via direct
scripted/bulk Management API calls has not always fired the corresponding
webhook reliably** — migration and content-ops automation should either use
the standard publish action or handle revalidation separately. Flag this
before a customer designs a bulk pipeline around webhook-driven
revalidation.

## API lifecycle & deprecation — the contractual answer
- Breaking changes ship as a **new major API version rather than an
  in-place change**. CDA v1 and v2 coexist; no announced EOL for v1. SDKs
  follow semver. The **Management API has a single version — there is no v2.**
- **GTC commitments** (quote these, not observed practice):
  - *Definition:* removal or material decrease in main functionality of the
    latest APIs without a suitable replacement, or a change that would make
    an interfacing external system non-operational.
  - *Notice:* **at least 30 days** before implementation; previous major kept
    unchanged for **3 months**, then deprecated.
  - *Remedy:* customer may object within 20 days of notice; if unresolved
    within 30 days, they may terminate and receive a pro-rated refund of
    unused fees.
- Observed practice has sometimes run far longer than the floor — the
  unscoped-PAT sunset gave ~6 months with 90/30/7-day reminders. **Present
  that as observed practice on one change, never as a committed term.** The
  30-day GTC clause is the actual floor.
- Notification channels: public changelog with RSS, email + in-app
  notification for breaking changes, status page with email subscription,
  plus the CSM on enterprise packages.

### Material breaking changes, rolling 24-month window (as at 2026-09)
| Announced | Effective | Change |
|---|---|---|
| 2024-07-08 | 2024-08-31 | Vue 2 SDK end of support |
| 2024-07-08 | 2024-12-31 | Nuxt 2 SDK end of support |
| **2026-05-27** | 2026-05-29 creation retired / **2026-11-30 existing tokens revoked** | **Unscoped Personal Access Tokens sunset** — all Management API tokens now require explicit space/permission scoping |

⚠️ The PAT sunset is **live and time-sensitive**. Any new build's migration
and automation tooling must use scoped tokens (or the newer OAuth 2.0
scoped grants) from day one. Raise it proactively in any RFP involving
programmatic content operations — it is a real dated migration obligation,
not a footnote.
