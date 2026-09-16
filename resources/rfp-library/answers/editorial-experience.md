# Editorial experience

Source: validated RFP submission, 2026-07-30. Trust: `validated`.

## The differentiator, stated as such
The Visual Editor is the primary differentiator in the headless CMS
market: **true in-context, real-time visual editing with click-to-edit on
the live page — not a side-panel form view.** Say this explicitly; it's
the single most decision-relevant sentence for a marketing-team buyer.

Adoption proof: one B2B SaaS customer reported ~1 hour to full team
adoption; another 80% faster page creation. Use these against any
"90% of marketers publishing without central support" style target.

## Capabilities + plan gates
- **Components/blocks** — typed, nestable, reusable; defined once, reused
  across content types. Unlimited and composable by design, **with no
  per-entry pricing impact** — worth stating for large sites (2,000+
  pages, 20+ content types), it's a real cost differentiator.
- **Visual editing/preview** — live via the JS bridge (SSG/SPA/SSR),
  multiple preview environments, in-editor locale switcher.
- **Versioning** — Page History with who/when, compare + one-click revert.
- **Scheduling** — single-story scheduling from Growth up (2 scheduled
  stories on Growth/Growth Plus, 100 on Premium/Elite). **Releases** for
  coordinated multi-story publishing: **20 on Premium, unlimited on
  Elite**, not available below Premium. Release merging Premium/Elite.
- **Workflows** — default draft→review→ready-to-publish on all plans.
  **Custom Workflows are Premium/Elite: 2 on Premium, unlimited on
  Elite** (stages themselves unlimited on both). Per-stage role
  restriction depends on custom roles — also Premium/Elite.
- **Assets** — Asset Library + image service (on-the-fly resize/crop,
  WebP/AVIF, responsive sizing), CDN-delivered.

## AI authoring suite (in production — not roadmap)
AI content generation and AI Translations are on **all** tiers; AI alt text
from Growth up. **AI SEO and Basic AI Branding are Premium/Elite.
Advanced AI Branding and bring-your-own-model (Custom AI) are Elite only.**
Default model options: OpenAI only on Starter, OpenAI + Gemini above.

AI usage is metered in **AI credits** with monthly caps per tier (25k
Starter → 200k Growth Plus → custom on Premium/Elite) — a real consumption
line item, not an unlimited feature. Distinguish shipped features from
roadmap items.

---

## Content lifecycle gaps — added from the 2026-09-15 architecture RFI (`validated`)
An enterprise governance section will ask about the *whole* lifecycle, not
just draft→publish. Four of those stages have **no native control**, and the
validated answer names each gap with its delivery pattern rather than
hedging. Reuse this shape:

| Stage | Position |
|---|---|
| **Emergency publication** | ⚠️ No dedicated "emergency" control. Pattern: a **break-glass role with publish rights on *every* workflow stage** (not just the terminal one), so a nominated user can bypass the sequence; a scheduled story can also be force-published via "Publish it now". Restrict per space/content type; **every use is attributed in the Activity Log** — say that, it is what makes the pattern acceptable to a governance reviewer |
| **Expiry / embargo** | ⚠️ Story Scheduling exposes `publish_at` only — **there is no `unpublish_at`, so no native scheduled unpublish.** Deliverable via a workflow-automation add-on, or a custom date field used as a `filter_query` at fetch time so only content whose expiry is in the future is returned |
| **Archival** | ⚠️ No dedicated "archived" state. Pattern: a custom "Archived" workflow stage combined with unpublishing — **content and version history are retained indefinitely** |
| **Periodic review / freshness** | ⚠️ No native review-reminder or freshness scheduler. Deliverable via a scheduled API request filtering on `updated_at_lt` beyond a chosen date, flagging stale stories for review |

## Workflow capabilities that *are* native and worth claiming
- **Default 3-stage workflow on all tiers:** Drafting → Reviewing → Ready to
  Publish, and **only "Ready to Publish" permits publication** — that is the
  native approval gate, not an add-on. New stories can start at the default
  stage or with no workflow, configurable.
- **Custom stages carry their own edit *and* publish rights plus an assignee
  that triggers an email notification.**
- ⚠️ **Workflows are scoped to specific content types** — so a mandatory
  Legal stage can apply *only* to legally-sensitive content types while
  routine content stays fast. This is a strong, under-used answer to
  "can approval requirements vary by content risk?"
- **Language-based workflow option** tracks publish status per translation
  independently — e.g. English in Reviewing while German is still in
  Drafting, within the same story.
- **Workflow state changes emit webhooks** — the hook for feeding approval
  events into downstream automation.
- **Comments/discussions:** field-level discussions in the Visual Editor, a
  dedicated Comments tab, team mentions surfaced on the dashboard, and
  Management API access to discussions and comments.
- Workflows are configured per space and **reproducible across spaces via
  the Management API** — they are *not* natively distributed as a single
  cross-space definition, so script them from the repo (see
  `multi-space-governance.md`).

## Audit evidence for a workflow question — the three-layer answer
1. **Workflow Stage Changes** — `/v1/spaces/:space_id/workflow_stage_changes/`
   records every stage transition with `user_id`/`created_at`, per story.
2. **Activity Log** — exportable via `/activities/` to CSV, unlimited
   retention on the top tier.
3. **Version History** — every save and stage change with timestamp and
   author, restorable, with a **colour-coded diff between versions**.

Naming all three separately is what makes a compliance reviewer stop asking.
