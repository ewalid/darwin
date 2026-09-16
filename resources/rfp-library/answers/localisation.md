# Localisation

Source: validated RFP submission, 2026-07-30. Trust: `validated`.

## Model
Native internationalisation with two approaches: **field-level
translation** and **multi-tree / folder-level (Dimensions)** for markets
needing structurally different content rather than translations. Locale-aware
preview in the Visual Editor with an in-editor language switcher;
locale URL structure (`/fr/`, `/de/`) is a standard front-end i18n pattern.
Translatable slugs supported.

## Fallback behaviour — precise answer
The Content Delivery API takes a `fallback_lang` parameter: an untranslated
field returns the fallback locale's value, then the default language if
that is also missing. Fallback is configurable **per API request at the
locale level**.

⚠️ **Per-field fallback granularity** (different fallback rules for
different fields on the same page) is **not** natively supported. Answer
"Supported with configuration" and state the limit honestly — an earlier
draft answered this vaguely ("should be re-confirmed") which is worse than
a precise partial yes.

## Translation workflows — two paths, both real
- **Native AI Translations** — 35+ languages directly in the editor. Fast,
  in-platform, no vendor contract needed. Often overlooked; lead with it.
- **External TMS** — official integrations with Lokalise (bidirectional
  sync), Smartling, Localazy, plus language export/import (XML/JSON)
  compatible with tools like Trados. **Gating: the Smartling integration
  and Language Export/Import are Premium/Elite only**; Dimensions
  (multi-tree) and translatable slugs are available from Growth up;
  translatable asset metadata is Premium/Elite.

Present both. Enterprise localisation questions usually assume a TMS is
mandatory; showing a credible no-vendor path is a differentiator.

## People-based translation via a customer's chosen vendor (not just TMS apps)
A validated submission described integrating with a **customer-nominated
human-translation vendor** (e.g. Claytablet) that isn't one of the official
app-directory TMS integrations above — the pattern generalises to any
similar vendor, and different vendors can be used per market (brand +
country) in the same deployment. The mechanism: source-locale content is
exported programmatically to the vendor via the **Management API's
Translation Endpoints**; the vendor's translated content is re-imported via
the same endpoints, populating the target locale on the existing Story
within the relevant Space. Manual export/re-import via CSV is also
supported for vendors without an API integration. Use this answer whenever
an RFP names a specific human-translation vendor that isn't in the app
directory — the API-first pattern still applies generically.

---

## Added from the 2026-09-15 architecture RFI (`validated`)

### The three models, as a chooser
| Model | How it works | Best fit |
|---|---|---|
| **Field-level** | One story holds every language version; editors toggle a per-field translation icon; structure identical across languages | Locales sharing the same page structure |
| **Folder-level** | Separate folders per language, each with its own story tree | Locales needing structural differences *within* a shared space |
| **Space-level** | Separate spaces per market/brand, independent settings/roles/content | Large-scale autonomous regional/brand operations |

Present it as a chooser, not a list — the question behind "how do you do
multi-language" is almost always "how much structural divergence can each
market have", and this table answers that directly.

### ⚠️ The limit to name before a prospect designs around it
**Blocks and Group fields are not field-level translatable.** Where a whole
*block structure* must differ per market, the answer is folder-level
translation or Dimensions — not field-level. This is the single most common
modelling mistake in a multi-market build, and naming it upfront reads as
experience rather than as a caveat.

### Dimensions — the precise mechanics
The Dimensions app links alternative versions of a story across top-level
(market/locale) folders, with **per-field control over what's duplicated
versus left alone**:
- **Clone** — creates a duplicate in a target folder
- **Merge** — copies structure into linked stories while **preserving local
  edits**
- **Overwrite** — replaces structure and content fully
Plus a "force overwrite by merge" option and **per-block exclusion** for
fine-grained control over what stays local. Linked alternates can carry an
entirely different structure from each other.

### Cross-*space* sharing is a different mechanism with a hard constraint
Sharing content across **brand spaces** uses **Multi-Space Content
Distribution** (clone, then push updates as overwrite or merge). ⚠️ It is
**same-region only, does not clone assets, and runs no automated
compatibility checks** — a copy tool, not a sync pipeline. See
`multi-space-governance.md`.

### hreflang — the division of responsibility
We do **not** generate hreflang tags. Dimensions links each story's
alternate-language versions, and that relationship plus each locale's slug
is exposed via the API (alternates are part of the story JSON); the
**hreflang `<link>` tags are rendered by the frontend** from that data.
Same CDN/rendering division as everywhere else — state it consistently
across the whole response rather than only here.

### Fallback, restated
`fallback_lang` renders a backup language's content for untranslated fields
rather than blank content, when using field translation. **It defaults to
the space's default language if not defined.** A fallback-language setting
can bring a space live before full translation completes — worth
volunteering as a go-live accelerator in a phased rollout.

### Commercial framing
**Locales, countries and markets are not priced units.** On enterprise
packages, unlimited locales, Dimensions, Translatable Slugs,
Export/Import Translatable Fields and Multi-Space Content Distribution are
included rather than separately priced add-ons. Against competitors who
price per locale or per market, say this explicitly — it is a real
differentiator that a capability-only answer buries.
