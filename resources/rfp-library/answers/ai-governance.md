# AI governance

Source: 2026-09-15 architecture RFI (`validated`). AI-governance sections
are now standard in enterprise RFIs and security reviews — expect a
capability register with provider, data flow, retention, training policy,
residency, human review, auditability, opt-out, IP position and pricing.
Answer it as a register, not as prose.

## Capability register
| Capability | Provider / model | Data flow |
|---|---|---|
| **AI Translations** | OpenAI, Google Gemini or Anthropic Claude — **customer-selectable** — or BYO provider | Field text + configured system prompt/brand rules sent **as-is**; no anonymisation or PII-scrubbing layer. No space metadata or user email sent |
| **AI Alt Text** | same | ⚠️ **The image itself** is sent, not just a text prompt |
| **Ideation Room** | same | Draft text and prompts sent |
| **AI SEO app** | same | Story content sent |
| **AI Branding** | same | Brand voice/tone/style config sent as context alongside the content prompt |
| **MCP Server** | whatever external agent the customer connects | **Reverse direction** — an external agent connects *into* the platform rather than content being sent out. A tiered execution model (read-only / mutating / destructive) scopes what a connected agent can do |
| **BYOAI** | the customer's own contracted provider + API key | Request goes directly under the customer's key; **nothing logged on our side for that traffic** |

Everything above except BYOAI is normally **included** on enterprise
packages with a custom AI credit allocation. **BYOAI is a separately
priced, annually billed add-on — not bundled with the base licence.**
Confirm it as a firm entitlement on the order form rather than asserting it.

## The three answers that matter in a security review

**1. Training.** Input is **not used to train the AI features or the
underlying models.** Flat, clean, reusable.

**2. ⚠️ Residency — the answer people get wrong.** AI processing is **not
confined to the space's selected data-residency region.** A space set to EU
residency still sends AI Input to the AI providers (OpenAI Ireland, Google
Cloud EMEA, Anthropic Ireland) and their respective processing locations
when AI features are enabled on it. Say this out loud, with links to each
provider's own subprocessor list. A prospect who has bought EU residency
and then discovers this later has a real grievance; a prospect told upfront
usually just disables AI on the sensitive space. This is the single most
important disclosure in the whole AI section.

**3. Opt-out is granular and real.** AI features can be enabled or disabled
at **either space level or organisation level** (Settings → AI Settings),
so one sensitive space can have AI off entirely while it stays available
elsewhere. Disabling stops further data sharing immediately, and data is
only sent **"when actively prompted by a user," never in the background.**

## Controls, limits, IP
- **Human review:** the AI Terms require the customer to ensure appropriate
  human oversight before relying on, using or publishing any Output. Output
  lands as a **draft field — nothing auto-publishes** without an editor
  explicitly saving/publishing.
- ⚠️ **Auditability gap, state it:** the standard content Activity Log
  records the resulting edit/publish, but there is **no dedicated AI-specific
  log of which feature or prompt was used.** If a prospect needs a
  prompt-level AI audit trail, that is a genuine gap, not a partial.
- **IP position:** customer retains ownership of Input; we claim no
  ownership of Output; Output **"may not be unique"** — no exclusivity
  guarantee. Quote the terms rather than paraphrasing.
- **Subprocessors:** OpenAI Ireland, Google Cloud EMEA and Anthropic Ireland
  appear in the DPA subprocessor annex **only if the customer opts into AI
  features.** Customers get a **14-day objection window** on subprocessor
  changes. That conditionality is a genuinely good answer — use it.

## BYOAI — what it actually changes
BYOAI changes the **data-processing relationship**, not just the billing:
requests go directly under the customer's own contracted key, governed by
the customer's own agreement with that provider, with nothing logged on our
side for that traffic. Retention and residency become "whatever the
customer's vendor contract specifies."

This makes BYOAI the natural answer to *any* prospect whose blocker is
"our data must not reach a third-party model under your contract" — route
them to BYOAI rather than to a flat no or to disabling AI entirely.

## The consultative move that landed
When a prospect handles genuinely sensitive material, **name the specific
feature that carries the exposure rather than answering the section
generically.** In the validated submission that was: *AI Alt Text sends the
image itself outside the managed-provider boundary — if images of that
sensitivity should not leave, disable that feature specifically, or run it
under BYOAI.* One sentence, unprompted, directly actionable. That is what
separates an AI register that reads as compliance paperwork from one that
reads as expertise.

Look for the equivalent in every deal: which single AI feature touches the
prospect's most sensitive data class, and what is the precise remedy.

## MCP server — a native capability, not a frontend pattern
The first-party hosted MCP server (OAuth or token auth) lets AI tooling
query and act on live content through the Management API, permission-scoped
per connected agent via the tiered execution model. It is worth separating
explicitly from the "AI features" rows above: **no content is sent to our
managed AI providers through it**, data stays within our infrastructure and
whatever agent the customer connects, and a third-party model is only
involved if the connected agent itself uses one under the customer's own
arrangement. Standard Activity Log records any resulting
create/update/publish/delete, same as an editor change.

⚠️ MCP pricing/entitlement was **not confirmed** in the sources reviewed for
that submission — check the order form rather than assuming it is included.
