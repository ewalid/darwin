# Integrations

Sources: validated RFP submissions, 2026-07-30 and 2025-11-19. Trust:
`validated`.

## Forms handling PII — don't store it in Storyblok
Form *configuration* (fields, CTAs, submission behaviour) lives as a Story,
referenced by a form Block/component on the page. But when a user actually
submits the form, the data should go **directly to the customer's own
receiving system** (CRM/MAP/backend) — the validated guidance is explicit:
**"We do not recommend storing form submission data containing PII within
Storyblok."** Use this as the precise answer to "how do you handle PII from
form submissions," rather than implying Storyblok is a data store for it.

## Procurement route: AWS Marketplace
Storyblok licenses can be procured via **AWS Marketplace**, including
Private Offers, which Storyblok's team has extensive experience running.
Selling points worth citing when the prospect is an AWS customer: the
subscription can count toward an existing AWS Private Pricing Agreement/
committed spend; procurement is simplified (added to the existing AWS
account, no new contract/legal cycle); and Storyblok can sometimes help
secure AWS promotional credits depending on contract value and eligibility
(requires the prospect's AWS Customer ID to pursue). Cite this whenever an
RFP specifically asks about marketplace/procurement-vehicle options.

## ⚠️ First question, every single time
**Does our product have a native/first-party integration with the
prospect's own product?** If the prospect is a software vendor, check the
app directory and changelog for *their company name* before anything else.

Real case (2026-07): the prospect was itself a software vendor in a
category we already integrate with. A first-party plugin connecting their
own platform to our editor existed, shipped, and was documented publicly —
letting content teams browse, filter and embed the prospect's own product
data by channel/locale/category into any story, exposed through the
delivery API for front-end rendering. **No competitor in that evaluation
had a native integration with the prospect's own platform.**
It was the strongest differentiator in the entire response and the
first-pass research missed it completely, because it wasn't a requirement
line item. Requirement-driven research structurally cannot find this.

## Forms / CRM / MAP — the "no native form builder" answer
There is no dedicated form-builder app. That is **not** a dead end, and
should not be answered as "requires custom development" without naming
the route. Two proven paths:
- **(a) Lowest effort:** embed the prospect's existing marketing-automation
  forms directly in the front end via their JS embed. Existing forms keep
  working as-is, zero CMS dependency, zero re-plumbing of lead routing.
- **(b) More editor control:** model form fields as reusable components
  (text input, dropdown, checkbox) that editors assemble visually;
  front-end handles validation and submission to the MAP/CRM/any endpoint.

Answer as **Supported with configuration**, presenting both paths.

## Analytics / tag management / consent
GA4, GTM, Hotjar, Cookiebot and equivalents are front-end script/tag
embeds in the app shell — identical to any headless front end, no CMS-side
blocker. Existing GTM containers are retained. Answer: Supported with
configuration.

Known third-party gotcha worth flagging proactively: Hotjar's own GA4
integration does not work when GA4 is deployed via GTM. Unrelated to us,
but flagging it earns credibility.

## Commerce / PIM / TMS ecosystem — and the plan floor it sets
First-party plugins exist for a long list of PIM, commerce and TMS
platforms, plus a documented e-commerce integration-plugin pattern for
anything else. **Always check the live app directory, the changelog AND the
pricing-page integration matrix** rather than relying on a remembered list.

Two things the pricing matrix tells you that the app directory doesn't:
1. Nearly every PIM/commerce/TMS connector is **Premium/Elite gated**
   (only Figma, Netlify, Vercel are on all tiers; Bynder, Cloudinary,
   Optimizely, Semrush, Shopify, Slack from Growth up). If the prospect's
   integration is central to the deal, it sets a **pricing floor** — raise
   that early, not in negotiation.
2. It is a fast way to check whether an integration with the prospect's own
   platform exists at all — the 2026-07 miss was listed right there in the
   integrations section of the public pricing page.
See `pricing-licensing.md` for the full gating list.

## FlowMotion is the first-party middleware answer
Before reaching for third-party middleware, note FlowMotion — Storyblok's
own automation/orchestration product — advertising **500+ pre-built
integrations** across CMS, CRM, marketing, commerce and analytics, plus
workflow automation with approval steps, AI-driven asset tagging, and
download/access approval flows. In validated submissions it carried most of
the "can you integrate with X / automate Y" answers.
Two caveats that keep it honest: it is **separately priced** (Core/Advanced/
Pro, metered by workflow executions — see `pricing-licensing.md`), and using
it is implementation effort, not a toggle.

## No native CRM or marketing-automation connectors
Validated answer, stated as *partial* both times: there is **no native CRM
connector**, and **no native pre-built connector for SFMC, Braze, HubSpot,
Klaviyo or Adobe Campaign** specifically. Integration is achievable via
API/webhooks, FlowMotion, or middleware — and needs a **per-tool technical
check** rather than a blanket yes. Say exactly that.

Also worth knowing: Storyblok is a listed **Adobe Commerce/Magento
Technology Partner**, and supports **OIDC-based SSO** for editors (not only
SAML) — relevant when a prospect's CIAM stack is OIDC-standard.

## Extensibility fallback — and its hard limit
Official App Directory + first-party Plugin SDK (custom field types, Space
Plugins, OAuth-based) + webhooks + Management API. Third-party middleware
(Zapier, n8n, Pipedream) covers anything without a native app.

⚠️ **There is no customer-controlled server and no plugin-hosting
mechanism.** Storyblok plugins are self-hosted elsewhere and **run only
inside the editor, never on the public site.** So "migrate our existing
.NET/Java application into the CMS" is a genuine **no** — the app must be
rehosted as its own service/container/serverless function, a real
architectural change for the prospect. Embedding it into the new frontend
once rehosted is standard. This came up verbatim in a validated grid; don't
soften it.

---

## Added from the 2026-09-15 architecture RFI (`validated`)

### FlowMotion — what it actually is
Workflow automation add-on **built on top of the open-source n8n engine**,
with custom first-party nodes and integrations for permission management
within an organization. **Separately licensed.** Workflows are **exportable**.

The portability answer to give: it is proprietary in the sense that it wraps
n8n with our nodes, but **the customer owns its configuration and outputs,
workflows export, and dropping it changes nothing structural — no
lock-in.** Same for Multi-Space Content Distribution, whose output is
ordinary stories.

**Where FlowMotion genuinely earns its place** — these are exactly the
lifecycle gaps named in `editorial-experience.md`, so pitch it against them
rather than generically: scheduled unpublish/expiry, content-review
reminders, automated brand-guideline or SEO-readiness checks on content, and
scheduled audit-log export.

Neither FlowMotion nor Multi-Space Content Distribution is **required** for
a standard architecture. Saying "not required" about your own add-on is what
makes the rest of the portability answer believable.

### Two complementary integration patterns — the reusable framing
1. **The frontend calls external APIs directly, alongside the CMS** at render
   time.
2. **External data is surfaced inside the editing interface** via field
   plugins, tool plugins or space plugins, so editors can select external
   records and reference them from editorial content.

Naming both matters: (1) alone sounds like "the CMS does nothing here";
(2) is what makes editors' experience of external data actually good.

### Custom editor extensions — ownership
Field/tool plugins and space apps are a **standard frontend codebase in the
customer's or partner's own repo**; we host only the manifest and runtime,
registered via the Partner Portal / App Store manifest. Useful in any
"who owns the extension code" or exit question.
