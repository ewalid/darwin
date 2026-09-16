# Security & compliance

Sources, in order of authority:
1. **The operator / the owning internal team** — settles anything about our
   own company that the website doesn't cover (see the SOC 2 entry).
2. Three official pages checked **2026-08-03** — `/trust-center`,
   `/trust-center/security`, `/enterprise-security` — plus the official public
   fact sheet (`official-docs`). Note `/trust-center/security` is visibly the
   **oldest** (see the retention and TLS discrepancies below); prefer
   `/trust-center`.
3. Two validated RFP submissions (`validated`) — useful, but each contained
   at least one wrong certification claim, so don't treat them as final.

## Certifications — the answer to give
- **SOC 2 Type I — held. Type II in progress** (operator-confirmed
  2026-08-03; not on the public pages because Type I reports are shared under
  NDA, not marketed).
- **ISO 27001 certified** — the certificate PDF is publicly downloadable
  from storyblok.com/enterprise-security, and the Trust Center links TÜV.
  (A validated submission adds: TÜV Rheinland, valid to 2028-03-03.)
- **TISAX** — listed under ISMS Policies & Certifications on the Trust
  Center.
- **GDPR compliant**, and **self-certified under the EU-U.S. Data Privacy
  Framework** (Trust Center, Data Protection & Privacy).
- OWASP secure-coding adherence, AWS WAF, AWS GuardDuty (AI-based intrusion
  detection), Detectify continuous automated security testing, regular
  internal *and* third-party penetration testing, dedicated incident
  response team with severity-based escalation and root cause analysis.

### ✅ SOC 2 — SETTLED 2026-08-03 by the operator (authoritative)
**Storyblok holds SOC 2 Type I. Type II is in progress.**

Confirmed directly by the operator, who works at the company. This is the
answer to use. It also means:
- An earlier validated submission's **"SOC 2 Type I, never claim Type II"** was
  **correct**.
- The requirements grid's **"SOC 2 Type II certifications"** was **wrong** —
  Type II is in progress, not held. That answer went out in a real document
  and is worth flagging to whoever wrote it.
- The Type II error is still best explained by the misread described below.

**How to answer it:** ISO 27001 (certified, certificate publicly available) +
TISAX + GDPR + EU-U.S. Data Privacy Framework + **SOC 2 Type I**, with Type II
described honestly as *in progress* — never as held, and never with a promised
completion date unless the security team gives one. A SOC 2 Type I report is
typically shared under NDA rather than published, which is why it doesn't
appear on the marketing pages.

### Where the "Type II" error comes from — still worth knowing
`/trust-center/security` reads: *"The solution is hosted on Amazon AWS in
Frankfurt/Germany **which has** various security certificates like: SOC
1/SSAE 16/ISAE 3402, **SOC 2**, SOC 3, FISMA/DIACAP/FedRAMP, DOD CSM 1-5,
PCI DSS Level 1, ISO 9001/ISO 27001, ITAR, FIPS 140-2, MTCS Level 3."*

The subject of that sentence is **AWS**, not Storyblok — it's the hosting
provider's certificate list. Reading it as Storyblok's own inflates the
company's posture (SOC 2 → SOC 2 *Type II*, FedRAMP, PCI DSS L1…). Useful
practice: cite AWS's certifications separately and explicitly as **the
hosting layer's**, which is both accurate and reassuring in a security review.

⚠️ **This exact mistake recurred, verbatim, in the 2025-11-19 validated
submission** — the same AWS-certificate paragraph was reproduced under
Storyblok's own "Data Protection" security answer with no clarification
that the subject is AWS, not Storyblok. Two data points now, not one: this
is a standing risk in how the source material gets copied into RFP
responses, not a one-off slip. **Always rewrite this paragraph explicitly
as "our infrastructure provider AWS holds..." before it goes in a real
submission** — don't copy it as-is even from a previously-validated
document.

### ⚠️ Lesson recorded: absence from the website ≠ absence of the certification
On 2026-08-03 this file briefly asserted "Storyblok has **no** SOC 2
certification of its own" on the basis that three official pages didn't
mention it. That was an over-correction — and a worse error than the one it
was fixing, because it would have had us *understate* our compliance posture
to a security-conscious prospect.

Not-published-publicly is not the same as not-true. Compliance reports
(SOC 2 especially) are routinely NDA-only and deliberately unmarketed. For
any **internal fact about our own company**, the operator or the owning team
outranks the absence of a public web page. Search the website to *check* a
claim; ask a human to *settle* one.

## SSO / provisioning / token scoping
SSO: Auth0, Okta, Microsoft Entra ID, Google Workspace, OneLogin,
JumpCloud, Salesforce, SAML 2.0 and SAML 1.0. SCIM 2.0 provisioning via
Okta and Entra ID (auto-provision, role sync, de-provision).

**Scoped Personal Access Tokens (PATs)** — apply least-privilege to
integrations, automation workflows and developer tooling by limiting a token
to only the permissions it needs. Worth naming in any API-security answer.

Internal network access is public/private key pair over SSH only; access is
monitored by automated tooling with brute-force mitigation, and every content
change is logged to the user activity event stream.

**Plan gating:** SSO, SCIM and **Forced** 2FA are all **Premium/Elite
only** — not on any self-service tier. Plain 2FA is on all tiers.

✅ **"Is SSO an add-on?" — resolved 2026-08-03.** It is **included in the
user licence** on Premium and Elite. The official fact sheet's legend
distinguishes ✔ (included) from $ (additional fee), and SSO carries ✔ on
both enterprise tiers; its Security Access row reads Premium = "Standard
roles +SSO". A validated submission had called SSO an Enterprise add-on —
that was imprecise. See `pricing-licensing.md` for the genuine paid add-ons.

## RBAC / audit
Admin + Editor roles on all plans. **Custom roles are Premium/Elite only
(10 on Premium, unlimited on Elite).** Scoping by space, language, content
type and workflow stage comes with custom roles.

**Folder / block / datasource / asset-folder scoping and Access Token
Scopes are Elite ONLY** (the "Additional Access Controls" row), as is
restricted IP range and security audit. An earlier draft presented
fine-grained scoping as generally available — it isn't. Activity Logs are
encrypted (AES-256 at rest, TLS 1.3 in transit) and access-restricted.
**Retention: 1 day Starter / 30 days Growth & Growth Plus / 180 days
Premium / unlimited Elite** — "retained indefinitely on Enterprise" is
wrong, only Elite is unlimited.

## Change management (useful in security questionnaires)
Documented process for customer-impacting changes: **peer-reviewed**
(technical review required), **tested** (verified not to adversely impact
security), and **approved** (all changes authorised for oversight of business
impact). Unusual/malicious traffic is detected by CloudWatch alarms with
notification to a responsible employee.

## Contracting detail worth knowing (Trust Center)
There are **two separate GTCs**: one for enterprise customers in the
**Americas** region, and one for global self-service customers plus
enterprise customers in **all other regions**. Point procurement at the
right one. Separate terms also exist for AI-powered features. DSA (EU
2022/2065) and DMCA compliance are both stated.

## Honest limit: no customer-facing infrastructure monitoring
**Server/infra-level metrics, tracing and alerting dashboards are not
exposed to customers** — that is Storyblok's internal ops concern. Content-
level auditing is covered (Activity Log on all plans; longer retention on
enterprise tiers), and Storyblok monitors its own infrastructure, but a
prospect asking for a self-service CPU/memory/response-time dashboard gets
a *partial*, not a yes. The public status page plus the uptime SLA are the
customer-visible surface. Validated answer, twice.

## Residency / DR / SLA
EU residency by default (AWS Frankfurt, Germany). **Standard selectable
regions: EU, US, Canada, Australia** — dedicated AWS infrastructure, per the
Trust Center and the official fact sheet.

**China IS officially offered — correction, 2026-08-03.** An earlier note
here said China wasn't available because the Trust Center lists only the
four standard regions. The official fact sheet is explicit: **"Additional
Data Center (China)"** is a **paid add-on on both Premium and Elite**, with
dedicated Mainland China infrastructure, **an ICP license already in place**
for international customers, and a custom domain (`app.storyblokchina.cn`).
This is a real, sellable answer for a prospect with a China requirement —
just priced separately. Lesson: the Trust Center lists the *standard*
regions; the fact sheet lists the *purchasable* ones.

**Region choice ("Preferred Data Center Selection") is a paid add-on on
Premium and included on Elite** — not a free universal setting.

DPA is published publicly and also downloadable in-app. Hot-standby read
replica with automatic failover; point-in-time restore within the
transaction-log window (see the retention caveat below); regular backup
testing. **Self-managed S3 backups to
the customer's own bucket are Premium/Elite** (weekly on Premium, daily on
Elite); a **Managed Backup Service** is a paid add-on on both (Premium
weekly/180 days, Elite daily/up to 7 years). Full programmatic export via
the APIs at any time (data-portability answers).

⚠️ **Backup retention: the two official pages disagree — 14 vs 30 days.**
- `/trust-center` (newer): **"14-Day Transaction Log Retention** — restore to
  any point in time within the last 14 days."
- `/trust-center/security` (older): daily S3 backup "with a changelog for a
  **retention period of 30 days**. Restoring is possible at any point in time
  within 30 days."

`/trust-center/security` shows clear staleness tells — it still says APIs use
**TLS 1.2** and carries a note about TLS 1.1 being disabled "after
01.12.2021", while the main Trust Center states **TLS 1.3**. So the 14-day
figure is probably current and the 30-day page probably out of date — but
**don't state either as fact in a security review without confirming**;
retention windows are exactly the kind of number a regulated prospect will
hold you to. Also documented: **monthly recovery tests** covering
point-in-time database recovery and asset recovery via S3 versioning.

**Likely reconciliation (2025-11-19 validated submission read both figures
side by side):** the same document stated **30-day retention for daily S3
backups** *and*, separately, **14-day point-in-time restore via the
transaction log**. Read together, these look like **two different
mechanisms, not a contradiction**: daily S3 backups are retained 30 days
(coarser-grained, disaster-recovery snapshots), while the transaction log
supports point-in-time restore for the most recent 14 days (finer-grained,
any-timestamp restore). If this reading is right, the honest answer to a
security questionnaire is "30-day backup retention; 14-day point-in-time
restore granularity" rather than picking one number — but this is still an
inference, not an operator-confirmed fact, so **confirm with the
infrastructure team before it goes in a real submission.**

## Vulnerability management MTTR (mean time to resolve), by severity
From a validated submission's stated remediation targets, tested via
vulnerability scans, screenings, penetration tests and failover tests:
**Critical — 48 hours · High — 5 days · Medium — 10 days · Low — 14 days.**
Vulnerabilities that can't be fully fixed are risk-assessed and either
mitigated to an acceptable level or formally accepted, captured in an
internal risk portfolio. Patch management is split into two tracks:
libraries/dependencies (Dependabot-scanned continuously) and OS/environment
(checked against AWS's release cadence, compatibility-tested before
upgrading affected instances).

## Data processor vs. controller — the precise GDPR framing
Storyblok acts as a **processor** for personal data that customers include
in their own content (typically marketing/editorial content meant for
publication). For personal data tied to **access credentials** (e.g.
employee accounts used to log into the platform), Storyblok acts as a
**controller**. Use this distinction rather than a blanket "we are a
processor" when a security review asks about GDPR roles.

## Accessibility (WCAG)
Two dimensions, and they must be separated:
1. **Delivered front-end** — conformance is fully achievable through the
   implementation; the CMS does not block it. It is the implementation
   team's responsibility, not a platform guarantee.
2. **The editor app itself** — targets WCAG 2.2 AA, independent audit by
   AxessLab completed, full-time Accessibility Engineer on staff, design
   system audited, a small number of edge cases remain. The official
   published statement uses the words "partially compliant"; the validated
   framing is "largely compliant, audit completed, edge cases remain."
   Use the validated framing, but never claim full conformance.

---

# Added from the 2026-09-15 architecture RFI (`validated`)

## ✅ Backup retention — the 14-vs-30 contradiction is RESOLVED
The "likely reconciliation" inferred above is **confirmed**. A validated
submission states both figures side by side as **two distinct,
separately-retained mechanisms**:
- **Transaction-log point-in-time restore = 14 days**
- **Daily encrypted snapshots (AES-256/KMS) = 30 days, retained separately**

So the answer to give is **"30-day backup retention; 14-day point-in-time
restore granularity"** — not a choice between the two numbers. This is no
longer an inference. Backups are stored in **S3, in the same region as the
organisation**, and deleted on termination subject to the backup cycle
completing.

## Underlying infrastructure — the detail that answers DR questions properly
Space content is stored in **PostgreSQL on RDS, with real-time replication
to a second Availability Zone (RPO ≈ 0, RTO 15 minutes)**, plus **daily
backups shipped to a separate AWS region entirely** (e.g. Ireland for a
Frankfurt-primary EU space).

⚠️ **The honest nuance that wins the DR question:** for an **AZ/infrastructure
failure**, failover is automatic and AWS-managed. For **logical or data
corruption, the replica reflects the corruption too** — so recovery requires
a **point-in-time restore to a new instance, not a failover**. Exact
RPO/RTO for that path is **not published; estimate ~1 hour and say it is an
estimate.** Long-term/compliance restore from snapshot: ~1 hour. Naming the
distinction between infrastructure failure and logical corruption is what
separates a real DR answer from a marketing one.

## ⚠️ SLA scope — narrower than people assume
The contractual **99.99% annual average uptime is scoped specifically to the
Content Delivery API in the GTC.** It does **not** separately name:
- **CDN & asset delivery** — same infrastructure as the CDA, but not
  separately covered by the commitment
- **Management API / Visual Editor** — same core platform infra; **not
  published as a separate contractual number**, confirmed on request at
  contracting
- **Publishing latency** (webhook fire → cache invalidation) — **no published
  contractual number**; an architectural characteristic, not plan-gated
- **Preview availability** — the preview experience renders the *customer's
  own* deployment in an iframe; our role is limited to issuing and
  validating the signed preview token. We do not monitor or commit an SLA
  figure for it

**Service credits:** the Trust Center wording is that customers *"may be
eligible for a pro-rated refund credit for the next billing cycle depending
on your selected enterprise plan"* when the uptime commitment is missed.
**Exact credit percentages per shortfall band are in the Order Form / SLA
exhibit** — point procurement there rather than quoting a number.

**Dependencies to state:** our SLA covers our own service boundary. It
cannot cover the customer's DNS, CDN, WAF, load balancer, cache layer or
network. **The composite end-to-end figure depends on both SLAs together
and we cannot guarantee it.** Say this explicitly — an enterprise reviewer
asking for an "end-to-end availability commitment" is testing whether you
will overclaim.

## Support response targets (enterprise package)
**Critical 2 hours · High 12 hours · Medium 24 hours · Low next business
day.** Channels: Help Center, Live Chat, and a dedicated **Incident Email
for critical issues.**

## ⚠️ Identity — the material limitation, and its workaround
**SSO is configured at the organization level: one organization holds one
identity-provider connection and one IdP tenant ID at a time.** Native
federation of **multiple separate IdP tenants** (e.g. one per region) into a
single organization is **not supported.** For any multinational with
regional Entra/Okta tenants this is a real constraint — lead with it rather
than letting it surface in implementation.

Two workaround patterns, both validated:
| Pattern | How it works |
|---|---|
| **Hub-tenant (recommended)** | The customer designates one tenant as the identity hub and uses **cross-tenant synchronization** (Entra: requires Entra ID P1 per synced user) to sync users from regional tenants into it. One SAML app configured in the hub against our Entity ID/Reply URL; **SCIM connects to the hub only** |
| **Separate-org** | One organization per region, each federated to its own tenant. Cross-region content sharing then uses space-level collaborator access or Multi-Space Content Distribution instead of shared identity. More admin overhead |

**Federation detail:** SAML 2.0 and SAML 1.0; the organization owns and
manages its own connection, metadata, domain mapping and attribute
settings in-app.

**SCIM limits to state:** supported against **Microsoft Entra ID and Okta
only**; **one active SCIM token per organization**; provisions user
create/update, group-driven space assignment and enable/disable; ⚠️ **does
not provision organization owners/admins** — those stay manually managed.

**Joiner/mover/leaver** (hub pattern): created in the regional tenant →
cross-tenant sync to hub → SCIM creates the user and assigns role/spaces
from group membership. Mover = group membership change propagates. Leaver =
disabled in the home tenant → propagates → SCIM disables access. Walk this
through as a flow; identity reviewers score the *flow*, not the feature list.

## Role model, restated cleanly
| Role | Scope | Configurable |
|---|---|---|
| Owner | Full space control incl. deletion, billing, ownership transfer | Fixed |
| Admin | Day-to-day admin, space config, all features in a space incl. deletion | Fixed |
| Editor | Content creation/management only | Fixed |
| **Custom roles** | Scoped per space to stories/components/assets/datasources | Fully configurable; **mappable to an IdP group via External ID** |

The External-ID group mapping is the detail that makes "how do roles map
across countries and brands" a one-sentence answer.

## Visual preview security — the end-to-end flow
Asked routinely, and the answer is more rigorous than most people expect:
1. Editor authenticates to the org (password or SAML SSO); the editor only
   loads spaces/stories their role grants.
2. Opening a story loads the space's configured preview URL in an iframe
   with `_storyblok` (story ID) and
   `_storyblok_tk[space_id|timestamp|token]`, where token is a hash
   combining space ID, timestamp and preview token.
3. **The customer's route handler recomputes the SHA1 with its own preview
   token and compares, rejecting any timestamp older than a configured
   limit (e.g. one hour) — that is the replay protection.**
4. Once verified, the app calls the CDA with `version: draft` using the
   preview token — **the only general-purpose token type returning
   unpublished content.**
5. Draft Mode sets a signed, httpOnly session cookie so subsequent requests
   skip re-verification and force dynamic rendering instead of cached output.
6. **Cache prevention:** draft/preview responses must be excluded from the
   customer's CDN and served `private`/`no-store` — a customer-owned config
   item, since our CDN never sits in this path.
7. Each space supports **multiple named preview URLs** (local, staging,
   production), selectable from a Visual Editor dropdown.

⚠️ **The critical shared-responsibility point:** preview traffic originates
from each editor's own browser, so **there is no fixed IP to allowlist.**
**Server-side validation of `_storyblok_tk` is the entire control — and it
is the customer's to implement. Unvalidated, the preview environment is
protected by URL obscurity alone.** Recommend on top: a
`frame-ancestors https://app.storyblok.com` CSP, `noindex`/robots exclusion
on the preview route, and the customer's own network control (WAF rules or
signed cookies) so token validation is the second line rather than the only
one. Stating this plainly in a security review is far better than letting
it be discovered.

**Token lifetimes:** preview access token has **no fixed expiry**, is
revoked/regenerated on demand with immediate effect, and **multiple tokens
can be active for zero-downtime rotation.** The `_storyblok_tk` signature is
not separately revocable — it derives from the active preview token, so
revoking that invalidates all its signatures.

## Certifications — refinements
- **SOC 2 Type I:** report held, **KPMG, ISAE 3000, latest completed dated
  2025-07-15.** That auditor/standard/date detail is new and worth having.
- **SOC 2 Type II: the audit has been completed; the report is awaited.**
  This is a *step forward* from "in progress" but still **not held** — do
  not claim it. ⚠️ The same validated document gave **two different expected
  dates in two sections ("H2 2026" and "Q4 2026")**. Don't quote either
  without confirming with the security team; if a date is needed, ask.
- **ISO 27001 (TÜV Rheinland), TISAX, GDPR/UK GDPR with signed DPA and SCCs**
  — unchanged.
- ✅ **Standing lesson 9 held.** This submission cited AWS's certifications
  **correctly and explicitly as the infrastructure provider's** — "our
  infrastructure provider (AWS) separately holds SOC 1/2/3, FedRAMP, PCI DSS
  Level 1, ISO 9001/27001… cited as such rather than as Storyblok's own."
  The fix is now demonstrated in a real submission; copy *this* phrasing.
- **Penetration testing:** regular external third-party testing; **a summary
  report can be provided under NDA.** Offer that proactively.

## Data protection — residency, transfers, tenancy
- **Content, assets, user profiles and telemetry** are stored in our AWS
  infrastructure in the space's selected region: **EU (default), US, Canada,
  Australia, or dedicated China infrastructure — selectable per space.**
  ⚠️ **Published-content delivery is CDN-distributed globally regardless of
  the space's region** — state this; a residency reviewer will ask.
- **Encryption:** AES-256 at rest; TLS in transit, **TLS 1.2 minimum, TLS 1.3
  supported.**
- **Tenant isolation:** **logical multi-tenancy** — segregated by
  organization/space ID within shared infrastructure. **No dedicated
  single-tenant deployment** on the standard SaaS model. Say it plainly (and
  see the BYOC note in `platform-architecture.md` for the enterprise
  exception).
- **International transfers:** Standard Contractual Clauses under GDPR
  Chapter V. **Subprocessor list (DPA Annex 3):** AWS EMEA SARL (Luxembourg
  entity; datacenters in EU/US/Canada/Australia) and Tiptap GmbH (Germany) —
  **plus OpenAI Ireland, Google Cloud EMEA and Anthropic Ireland *only if*
  the customer opts into AI features.** Customers get a **14-day objection
  window** on subprocessor changes.
- ⚠️ **AI processing is not confined to the space's residency region** — see
  `ai-governance.md`. This is the most consequential residency disclosure in
  the whole response.
- **Support-access data:** access limited to what is reasonably necessary
  per the DPA; retention and deletion for this category are **not published
  in customer-facing docs** — say so rather than guessing.
- **Telemetry retention:** Activity/audit log unlimited on top tier;
  ⚠️ **webhook delivery-log retention window is not separately published —
  a genuine gap, flag it.**
- **On termination:** we return, delete or block access to Customer Data
  (including backups) unless legally required to retain, per the DPA.

## Accessibility — the enforcement matrix (supplements the section above)
Classify each requirement rather than answering "partially". The validated
classification:

| Requirement | Classification | Mechanism |
|---|---|---|
| **Alternative text** | **Enforced** | Settings → Asset Library → Alt Text can be set to **Required**, blocking save/use of an image without alt text. ⚠️ **Configured per space** — each space needs it set individually, not once centrally. AI-assisted alt-text suggestions speed authoring |
| **Invalid component combinations** | **Enforced** | Component/group allowlisting restricts what can nest inside a Blocks/Rich Text field — a disallowed combination **can't be inserted**, not flagged after the fact |
| **Headings** | **Partially enforced** | The Rich Text toolbar can be configured **per field** to limit which heading levels an editor can insert. Validating heading *order* across an assembled page is page composition — a frontend concern |
| **Semantic structure** | **Not in scope of a CMS** | Schemas and required fields enforce content structure, which is what feeds semantic HTML; producing final markup is a frontend responsibility in any headless architecture. Recommend Lighthouse/axe-core in their CI |
| **Captions / transcripts** | **Not in scope of a CMS** | A media-production and video-hosting function. Achievable as a schema with a required caption/transcript field if they want it modelled alongside content |
| **Link purpose** | **Not in scope of a CMS** | Whether link text reads well is a copywriting judgment — **no CMS validates this natively.** Say so; don't invent a partial |

**None of the above carries an additional cost.** And note the editor app
and website **target WCAG 2.2 AA** (see the framing rules in the
Accessibility section above — "largely compliant, audit completed, edge
cases remain", never full conformance).

**"Not in scope of a CMS" is a legitimate and well-received classification**
when it is paired with what the platform *does* contribute and who owns the
rest. Three of six rows used it here without weakening the response.
