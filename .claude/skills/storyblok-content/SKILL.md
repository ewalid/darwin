---
name: storyblok-content
description: >
  Trigger: "set up the Storyblok space for [account]", "create these
  stories/components in [space]", rebrand/restructure/populate an
  existing demo space. Writes CMS content only (stories, components,
  assets, experiment results): dry-run → explicit OK → write → verify.
  Never clones a frontend, never edits Vue/Nuxt/React, never deploys
  Vercel — if they need a customer-specific site or a layout the starter
  cannot render, that is `custom-storyblok-demo` (standup) or work in
  the already-cloned demo repo. MCP first, Management API fallback.
---

# storyblok-content

## Scope — this is Darwin's one opt-in, product-specific skill

Darwin ships company- and product-agnostic (CLAUDE.md "Identity"). This
skill is the sanctioned exception: it's only relevant if the operator's
company uses **Storyblok** as its CMS (whether that's the product they
sell, or just the CMS they run). If the operator's product/stack has
nothing to do with Storyblok, this skill simply doesn't apply — it makes
no assumption that anyone selling anything uses it. Everything else in
Darwin stays product-neutral.

## Status (2026-07-22)

Two live paths now, checked in this order every time — never assume
which one is available, this varies by *where* Darwin is running,
not by session:

1. **Storyblok MCP connector.** the operator confirmed Claude Code (run
   directly on their machine, outside Cowork) has a working Storyblok
   MCP connection. If running there, check for it first — discover
   the actual tool names at runtime (same rule as Gong in CLAUDE.md
   guardrail 5, never assume a name). If found, use it directly for
   all read/write calls below; no network-egress concern applies,
   since the call isn't going through Cowork's sandbox proxy at all.
2. **Management API token (fallback).** the operator provided a Storyblok
   Management API personal access token. Stored at `storyblok.env`
   in the repo root (gitignored via the existing `*.env` pattern —
   confirmed with `git check-ignore -v` before writing the token in,
   per CLAUDE.md guardrail 10) as `STORYBLOK_MANAGEMENT_TOKEN`.
   **Never commit this file, never paste the token into chat or into
   any tracked file again.**

   Tested from *inside a Cowork sandbox*: `curl` to
   `https://mapi.storyblok.com/v1/spaces/` was blocked by the
   sandbox's own network proxy — `403 Forbidden`,
   `X-Proxy-Error: blocked-by-allowlist`. Not a token problem, not a
   missing-connector problem — Cowork sandboxes can only reach an
   admin-controlled domain allowlist, and `storyblok.com` isn't on
   it (as of 2026-07-21).

   This path works with no admin action needed as long as it's run
   somewhere with normal network access — the operator's own terminal, or a
   Claude Code session on their machine (the repo lives at
   `~/dev/darwin`, and `storyblok.env` already exists there with the
   token in it). It would also work directly inside Cowork if a
   Team/Enterprise org Owner adds `storyblok.com` /
   `mapi.storyblok.com` to Cowork's network allowlist (Admin settings
   → Capabilities) — not done as of this writing, not blocking since
   path 1 exists now.

Net effect: this skill is completable *today* from Claude Code (path
1, or path 2 with normal network access) even though a Cowork
session on its own is still blocked on path 2 specifically.

## This skill does not touch a frontend

It writes **into a Storyblok space**. It does not clone a template repo,
edit Vue/Nuxt/React/CSS, or run `vercel --prod`. Those belong to
`custom-storyblok-demo` (first standup of a customer frontend) or to the
already-cloned demo repo (later code changes).

If the operator asks for a layout the current frontend cannot render
(a new block type, a full-bleed video hero that needs a new component),
say so and route there. Authoring content cannot invent a renderer.
Rebranding, folder architecture, site-config copies, Dimensions clones,
and fake experiment charts **are** this skill.

**MCP first, Management API (`storyblok.env`) second, CLI third.** MCP
has no `updateSpace` — language codes and Dimensions folder mapping are
**UI** (Settings → Internationalization / Dimensions). Folders, stories,
components, and duplicates go through MCP.

## Space architecture (before any write, when the ask is structure)

When the ask is regions, brands, shared content, or translations — **plan
the tree in chat and get OK** before creating folders. Do not invent a
frontend routing scheme.

Standing facts (docs, not guesses):

- Storyblok does **not** auto-copy or auto-translate a new story into
  other languages. Field-level stores sibling keys (`field__i18n__code`)
  on **one** story. Folder-level is separate trees. Dimensions **links
  clones across top-level folders only** — locale folders nested under
  a brand folder cannot be Dimension roots. Clone / merge / overwrite
  match the same relative path in another root folder.
- **Folder-per-locale (Dimensions) needs the space-level
  `use_translated_stories` setting OFF.** With it ON, a `?language=<code>`
  fetch of a story that lives in a locale FOLDER 404s — the API expects
  field-level translations the folder content doesn't have. This 404'd
  published locales on two separate builds before it was traced to one
  space setting.
- **`with_slug` (and any folder-path lookup) resolves to the FOLDER, not
  its startpage.** Set a field on the id it returns and the value lands on
  the folder story, not the page a visitor sees — invisible on the rendered
  site. Target the startpage's own id for content writes (a residence field
  was once set on the folder instead of the page this way).
- **`Blocks` fields are not translatable.** Same blok tree across
  field-level locales; different structure per market needs folders +
  Dimensions.
- **Shared-across-brands content that still differs by market** lives
  under each Dimension root (`en-gb/_shared/…`), referenced by brand
  stories — not one global story unless copy is identical everywhere.
- Duplicating a folder tree is **per-story** (`duplicateStory`). There
  is no recursive tree duplicate. Duplicates get **new UUIDs**;
  reference fields still point at the source — remap or the copies
  resolve the wrong brand.
- **CMS schema ≠ frontend code.** The Visual Editor lists Block Library
  entries. The app renders whatever is registered under the same
  `component` string. Creating a Vue/React file does not create a CMS
  block; creating a CMS block does not ship a renderer. Schema sync
  between spaces is CLI (`components pull` / `push`), not git-magic.
- **GraphQL** is a read-only Content Delivery API (typed queries from
  component names, e.g. `default-page` → `DefaultpageItem`). It does
  not write content. Pricing lists it on Premium/Elite; REST CDN remains
  the fuller API. Do not treat GraphQL as a reason to start a custom
  frontend.

## What it will do

1. **Check for the MCP connector** (see Status, path 1). If present,
   all steps below go through it. If absent, fall back to path 2
   (Management API + token).
2. **Dry-run preview.** Given a target space and a set of
   stories/components to create or update (from a demo-setup script,
   an account brief, or a direct request), produce a full preview of
   exactly what would be written — story slugs, component structure,
   field values — without writing anything.
3. **Explicit confirmation.** Show the dry-run to the operator, wait for an
   explicit OK (CLAUDE.md guardrail 3). If the target is a production
   space (not a dev/test space), require a second, explicit
   confirmation naming the space — guardrail 3 is stricter there.
4. **Re-fetch immediately before any update-in-place write.** For any
   `updateStory` / `updateComponent` call (or MCP equivalent), fetch
   that exact story/component fresh right before writing — never
   reuse content held from earlier in the session or conversation,
   even content fetched moments ago. These endpoints replace their
   target in full; stale content silently clobbers anything changed
   in between by a human (e.g. a manual Visual Editor edit made while
   Darwin was mid-task) or by another process. This applies per-write,
   not once per task — if a task does three sequential updates to the
   same story, re-fetch before each one, not just the first.
5. **Write.** Execute the writes (create/update stories, components,
   assets) via the MCP connector's tools if available, otherwise via
   `https://mapi.storyblok.com/v1/spaces/{space_id}/stories` and
   related endpoints, authenticated with `STORYBLOK_MANAGEMENT_TOKEN`
   from `storyblok.env`.
6. **Verify.** Re-fetch what was written and confirm it matches the
   dry-run exactly — same principle as the deals-dashboard's
   `update_artifact` → `verify_artifact` pattern. Report any mismatch
   rather than assuming success from a 200 response alone.
7. **Then open it in the Visual Editor and confirm it is editable.** A
   re-fetch only proves the API accepted the write. It cannot see an
   empty story picker, a field the schema never declared, a component
   missing from the "+" menu, or a required-field error blocking Save —
   every one of which renders perfectly on the page. Check that each
   field shows its value, that the block can be added from the picker,
   and that the story saves. See "Schema semantics that break the
   editor, not the page" below for what to look for.
8. **Announce** what was written, in chat — never silent.

## Schema semantics that break the editor, not the page

Every item here produces the same signature: the write succeeds, the page
renders correctly, and the CMS is quietly broken — an empty picker, a
missing field, a page that refuses to save. None of it is visible from the
rendered site, which is why each one below cost real time on a multi-brand
build (2026-07-28/29) and several were only noticed when the operator
pointed at a screenshot. The commercial stake is the same every time: a
demo whose whole argument is "your marketers can edit this themselves"
fails at exactly the moment someone tries.

- **A reference field stores a uuid only when `use_uuid: true`.** An
  `options`/`option` field with `source: internal_stories` stores the
  numeric story id by default; `resolve_relations` only works with uuids.
  Author uuids without the flag and the frontend resolves everything
  while the picker shows an empty "Choose one or more…" — 18 of 21
  reference fields in one space were in this state, all rendering fine.
  Set `use_uuid: true` whenever content holds uuids, and check that no
  existing field stores numeric ids before flipping it.
- **A reference field returns an array even at `max_options: 1`.**
  Reading `.content` off the field instead of `field[0]` gives
  `undefined` with no error. One such line silently disabled every
  banner placement in a space at once, so the feature looked unbuilt
  rather than broken.
- **`component_group_uuid` is not `component_group_whitelist`.** The
  first is which folder a component lives in. The second is what a
  *bloks field* accepts, and setting it at component level does nothing.
  Since a page-body field typically whitelists one group, an ungrouped
  component renders where a script put it and is absent from the "+"
  menu — it cannot be added to any page, with no error to say why.
- **A story-type multilink is validated on `id`, not `cached_url`.** A
  link with a slug and an empty `id` renders correctly and makes every
  page holding it unsaveable ("Link in <field> is required"). If the
  helper that builds links omits the uuid, this returns after every
  script run — fix it at the source, then sweep the space once.
- **A destination with a query string must stay `linktype: "url"`.**
  Give it a story `id` and Storyblok resolves it to the bare story on
  the next fetch; a frontend that prefers the resolved value then
  silently drops the `?q=` filter the link existed for.
- **Content can hold fields the schema does not declare.** They render
  and the editor shows no field at all, so the value looks hardcoded.
  Orphaned content is worse than a missing feature because it looks
  deliberate. After authoring, diff the story's content keys against the
  component's schema keys and reconcile in whichever direction is right.
- **`resolve_relations` has to be passed to the fetch *and* the bridge.**
  Omit it on the bridge options and a reference resolves on load, then
  un-resolves on the first keystroke — the section vanishes for the
  person editing it, which reads as the CMS corrupting their work.
- **A schema `default_value` applies only to newly created stories.** On
  an existing space the field reads empty until someone saves it, which
  is indistinguishable from a broken field. Write the effective value
  into existing stories explicitly rather than relying on the default.
- **`translatable: true` is per-field, and a `bloks` field cannot carry
  it.** Translation happens on the fields of the nested components, so a
  headline is translated through the headline segment's own text field,
  never through the section that contains it. Collect the components a
  page actually uses at every depth and flag those; flag only the top
  level and every headline stays in the source language.
- **An asset field needs `id` and `meta_data`, not just `filename`.**
  Without them the editor offers no focal-point picker and there are no
  source dimensions to compute a crop from, so the feature looks
  unimplemented. A bare URL written by a script is not a library asset.
- **Experiment results are pushed in, not read out.** The Results tab
  says "push results from your analytics tool" and means it literally:
  `POST /experiments/:id/results` with a `charts` array (bar/line/text).
  Probing for a GET endpoint returns 404 and invites the wrong
  conclusion — that results can only come from Storyblok's own analytics
  — which sends the deliverable to the wrong place entirely. `started_at`
  is server-controlled and cannot be backdated, so any figures spanning
  more days than the experiment has run must be labelled illustrative,
  inside the panel rather than only in the deck.
- **Decide draft-versus-published deliberately, then verify the state.**
  Publishing is right when a live URL is the deliverable; staying draft
  is right when the point is to show an editorial workflow. Either way
  the two look identical in the Visual Editor, which reads draft, so
  confirm the state you intended rather than the one you assumed — and
  on a client-rendered frontend confirm it from the rendered DOM, not a
  status code.

- **Cloudinary (or any image CDN) `f_auto` / `q_auto` on SVG logos breaks
  them** in the browser. Prefer Storyblok CDN assets for demo logos; if an
  external SVG URL is unavoidable, strip those transforms before render or
  at write time. A white box / broken-image icon in a testimonials row is
  usually this, not a missing component.
- **Self-serve landing-page demo beat:** create a dedicated folder + root
  content type whose `body` bloks field has `restrict_components: true` and
  a tight whitelist (no commerce/blog/experiment blocks), a default
  **Campaign LP** preset with a full starter body, and one published
  example page. That is the guardrail story — schema enforce, not training.


## Guardrails

- Never write to a Storyblok space without the dry-run → OK step,
  regardless of which path (MCP or API) or which environment
  (Cowork or Claude Code) it's run from (CLAUDE.md guardrail 3).
- **Never clone a frontend, edit app code, or deploy hosting from this
  skill.** If the ask is a custom site or a layout the starter cannot
  render, stop and route (see "This skill does not touch a frontend").
- Production spaces need a second, explicit confirmation naming the
  space — a dev/test space does not.
- The token lives only in `storyblok.env` (gitignored). Never commit
  it, never echo it into chat, never write it into any other file.
- This skill has no bearing on RFP/security-question answers
  (`rfp-answer`) — that's a knowledge-library lookup, not a content
  API call, even once this skill is live.
- **Never reuse held-in-context story/component content across
  multiple writes in the same task** — see step 4. A full-content
  overwrite built from anything but a fresh fetch is a real risk on
  every single write, not just the first one in a task.
- **When extending an existing component's schema to add a new
  field or feature, touch only the new field(s).** Never add
  restrictions or `conditional_settings` to a *pre-existing* field to
  support the new feature — put any new conditional-visibility logic
  on the new field(s) only, and leave every pre-existing field
  byte-for-byte as it was. Adding a `filetypes` restriction plus a
  `conditional_settings` hide-rule to an existing image field (to
  support a new "toggle between image and video" feature) once broke
  that field's focal-point/crop editor in the Visual Editor — caught
  only because the operator noticed the hero looked different and the focal
  point stopped saving. The fix was to restore the pre-existing field
  to its exact original definition and move the conditional logic
  onto the new field instead. Diff discipline on schema edits is not
  optional: a schema update should read as "N new fields added," never
  as changes to lines that were already there.
- **Never accept a proxy for the surface a claim is about.** An HTTP
  200 proves nothing on a client-rendered frontend — every route returns
  200 with an empty shell, so a status code once stood in as evidence a
  page existed, and would later have "confirmed" the exact opposite
  about a draft being hidden. A screenshot can be a paint-timing race. A
  hidden or zero-width browser pane reports `innerWidth: 0`, which makes
  every responsive media query evaluate false and every layout
  measurement meaningless — one such reading looked exactly like the bug
  being chased. Name the surface, then check that surface.
- **Author through the path the operator will demo, or say plainly that
  it collides.** Content set via the API and content set through an
  editor picker are two write paths to one field. An operator pressing
  Done in a picker once silently replaced an API-populated product list
  with unrelated items (2026-07-27). If a field will be demoed by hand,
  set it by hand — or state up front that the API value disappears the
  moment someone touches that field.
