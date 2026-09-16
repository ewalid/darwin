# Personalisation & experimentation

Source: validated RFP submission, 2026-07-30. Trust: `validated`.

## A/B testing — status corrected 2026-07-30
Native **A/B Testing is GA** with its own product landing page, included at
no extra cost on **Premium and Elite — and available on no self-service
tier at all** (confirmed against the public pricing matrix 2026-07-30). Editors create Experiments, define
Variants (multiple versions of a story), and surface Results from their
existing analytics stack inside the CMS. Marketer-operable end to end:
variant creation, experiment setup, results review, no developer involved.

⚠️ **History worth knowing:** a July 2026 research pass found this as a
"Labs" changelog entry and consequently hedged the answer down to
"Supported with configuration — new, don't over-claim." By the validated
submission it was GA with a dedicated LP. Lesson: a changelog/Labs entry is
a *point-in-time* signal — check for a product landing page and current
plan-inclusion before downgrading a feature's maturity in an answer.

## Audience targeting / personalisation — the honest split
There is **no native audience-segmentation engine**. Audience targeting is
delivered by integrating a specialised tool (VWO, Optimizely; also Segment,
Dynamic Yield for delivery-side variants). The division of labour to state:
**content variants are managed in the CMS; targeting logic lives in the
specialised tool.** Frame it as MACH best-of-breed — each tool doing what it
does best — rather than as a missing feature, but never claim native
segmentation.

Wiring up targeting keys and page instrumentation is genuine
implementation work; "Requires custom development" is the fair answer for a
full audience-variant requirement (F19-style), while marketer-led A/B
testing itself is now OOTB.

## Don't overstate the gap either — simpler cases have a native path
**Native content variants plus returning-visitor detection** cover simpler
personalisation cases with no partner platform. It's profile/interest/
AI-based *targeting* that needs one. Validated framing: simple cases native,
sophisticated targeting via partner.

## AI-driven intent/behavioural experiences — the repeated honest answer
Requirements phrased as "contextual search driven by intent, supported by
AI", "dynamic page content based on click behaviour and navigation", or
"conversational recommendation" were each answered *partial* in a validated
grid with one consistent line: **achievable via custom development or a
partner platform, grounded on Storyblok's structured, API-first content.**
Reusable sentence — honest, doesn't concede the deal, and puts the
structured-content foundation forward as the enabler.

## Audience creation from a campaign ID (common demo ask)
Handled by the marketing-automation layer (HubSpot, Segment, etc.), not the
CMS. API + webhooks enable bi-directional flow so campaign data moves
between systems.

## Roadmap adjacency
Vector-based semantic content intelligence (Strata) is roadmap, not GA —
mention as direction of travel only, clearly labelled.

---

## Added from the 2026-09-15 architecture RFI (`validated`) — experimentation in a cached architecture

The question that separates a real answer from a feature list: **how does
experimentation coexist with a fully cached SSG/ISR front end?** Variant
allocation has to run per request, so a route carrying a live experiment
**cannot be served from a single pre-rendered, identical-for-everyone cache
entry**. Two patterns, both compatible:

- **(a) Render the experiment route dynamically** on every request.
  Simplest; costs one extra API round-trip on a cache-cold experiment-config
  fetch.
- **(b) Bucket earlier and vary the CDN-cached HTML on the `visitor_id`
  cookie**, serving a distinct cached HTML variant per bucket. This is the
  pattern **our own marketing site uses today** — cite that, it is concrete
  proof rather than theory.

Frame it explicitly as **an implementation decision for their team, not a
constraint we impose.** That sentence is what stops a cache-conscious
architect scoring experimentation as a performance risk.

### Consent — a clean "not ours, and here's why that's right"
⚠️ **There is no built-in consent gate** for the `visitor_id` cookie or the
exposure/conversion calls. Both the classification (functional vs.
analytics) and the consent timing are implemented in the customer's own
frontend against their own CMP. We are **not in that decision path** — which
is the correct architecture, since consent policy is theirs to set, not
ours to assume.

### Result data ownership — a strong anti-lock-in answer
Experiment and variant **content** lives as standard stories (exportable —
see `exit-portability.md`). Experiment **results** (statistical
significance, conversion data) live and are computed **entirely in the
customer's own analytics platform**; our Results tab only *displays* what it
reads back via the Management API. **No lock-in, no ownership of the
underlying measurement data.** Lead with this whenever a procurement or
data-governance reviewer touches experimentation.
