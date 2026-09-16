---
name: custom-storyblok-demo
description: >
  Trigger: "let's create a custom [demo] for [customer]" AND they need a
  customer-specific frontend (clone/scaffold a repo, change how blocks
  render, deploy). The word "custom" is NOT enough on its own — if they
  only want to rebrand, restructure folders, or author pages in an
  existing space on the unchanged starter frontend, that is
  `storyblok-content`, not this skill. Distinct from `demo-setup`
  (writes the planning script). End-to-end pipeline: Path A duplicates a
  starter template repo the operator already uses; Path B scaffolds via
  the Storyblok CLI. Then both: clone+own a private repo, apply known
  template fixes, deploy to Vercel, wire preview URLs. Template-specific
  specifics live in local memory, never in this file.
---

# custom-storyblok-demo

## What it does

Turns "let's create a custom for [customer]" into a deployed, CMS-wired demo
**with its own frontend**, end to end and mostly hands-off. Distinct from
`demo-setup` (planning/script) and `storyblok-content` (CMS content only —
no repo, no Vue/React, no Vercel). First real run 2026-07-24; reworked
leaner the same day — see "Efficiency notes".

## Fork first — custom frontend vs content-only

"Custom demo" is ambiguous. **Ask this before Path A/B**, in the same
batched turn as the other human-gated questions:

- **Custom frontend (this skill):** they need a customer-specific site —
  duplicate or scaffold a repo, change how components render (new blocks,
  layouts the starter cannot draw), deploy that frontend. Words that
  belong here: custom site, custom frontend, clone the template, a hero
  / layout / component the starter does not have.
- **Content-only (`storyblok-content`):** same starter frontend, different
  CMS payload — rebrand, folders, site-config, Dimensions clones, pages
  built from existing blocks, fake experiment results. **Stop this skill
  and run `storyblok-content`.** Do not clone a repo.
- **Frontend already cloned, more code work:** stay in that demo repo.
  Do not re-run this standup pipeline. Do not try to invent a renderer
  by writing CMS content.

The shape: **the human-gated bits go up front** (a starter template choice
and, if that template needs a pre-seeded CMS space, its creation — Darwin
can't seed one that renders), then everything after runs without hand-holding:
clone+own the repo, apply any known fixes, deploy to Vercel via CLI, set env
vars via CLI, wire both preview URLs into the CMS.

> **Template-specific details are local-only.** Which starter repo the
> operator uses, its known bugs/fixes, and any CMS-seeding quirks are
> confidential to the operator's setup and live in `memory.md` (gitignored) —
> **never** name a specific private repo, its files, or its fix list in this
> tracked skill. This file stays a generic process; memory supplies the
> specifics at run time.

## Two template paths — decide this FIRST

- **Path A — the operator has a starter/template repo they duplicate for
  demos.** Ask which one (the operator's usual default may be recorded in
  local `memory.md` — check there first; if found, confirm rather than
  re-ask). Use it as the base in step 3. Any template-specific fixes or a
  required pre-seeded CMS space are properties of *that* template — pull those
  from local memory, don't hardcode them here.
- **Path B — no such template, or the operator can't access one.** Ask which
  **framework/language** they want (Next.js, Nuxt, Astro, SvelteKit, etc.),
  then scaffold with the **Storyblok CLI** (see step 3, Path B). This produces
  a different boilerplate each time, so any Path-A fix list does NOT apply —
  verify what actually reproduces rather than editing files that may not exist.

Confirm which path applies before step 3. If the operator's memory records a
default template, that settles it (Path A); otherwise ask.

## Prerequisites

- GitHub CLI (`gh`) authenticated as the operator's own account.
- Vercel CLI authenticated (`npx vercel whoami`) — the deploy is done via CLI,
  not the dashboard.
- **Path A:** access to whatever starter template repo the operator names.
  **Path B:** the Storyblok CLI — npm package `storyblok` (`npm i -g storyblok`,
  or `npx storyblok`); `storyblok create` scaffolds a framework project and can
  seed a matching space.
- Storyblok Management API token in `storyblok.env` (`STORYBLOK_MANAGEMENT_TOKEN`),
  and/or the official Storyblok MCP connector (now permanently configured —
  HTTP server at `https://mcp.labs.storyblok.com/mcp`, registered via
  `claude mcp add --transport http --scope user`; it needs a **Personal Access
  Token**, not the space Management token, in the Bearer header, and its tools
  only load after a session restart following the add). The MCP has `getSpace`
  (read) but **no space-settings write op** — environments/preview URLs must be
  set via Management API curl regardless (see step 9).

## Steps

The human-gated step (1) goes first, so anything the operator must do resolves
while Darwin works — by the time the token is needed (step 6), it's ready.

1. **FIRST, get the human-gated answers up front** — in one batched turn,
   before touching code.
   - **Path A: ask which starter template repo** to duplicate (check
     `memory.md` for a recorded default first and just confirm it). If that
     template needs a pre-seeded CMS space that Darwin can't create itself
     (see the space guardrail), **ask the operator to create it** and to do
     any one-time space setup their process requires; don't block on their
     confirmation until step 6.
   - **Path B: ask which framework/language** instead — the Storyblok CLI's
     `storyblok create` scaffolds and seeds the space itself in step 3, so no
     separate manual space-creation ask is needed.
   - **Both paths: ask which Shopify store** — reuse the shared SE-demo
     sandbox used elsewhere, or a customer-specific one? (The shared sandbox
     is a generic multi-tenant store with no vertical-specific catalog — fine
     as a placeholder, but confirm it suits the demo.) Batch every applicable
     ask into one turn, don't drip them out.

2. **Resolve the customer's local working directory.** Check `~/dev/accounts/`
   for a folder that's already a variant of the customer's name — different
   casing, spacing, or an abbreviation (that directory already has
   inconsistent naming: `joué club`, `Group`, `implid-demo`/`implid-careers`
   as separate folders for one account). If something matches, use it. Only
   create `~/dev/accounts/<Customer>/` if genuinely nothing matches — **ask
   the operator to confirm** on any ambiguity. Never silently create a second
   folder for the same customer under a different spelling. The
   same discipline covers the whole build: **one demo = one repo + one space
   + one deploy.** Don't start a second, parallel build of the same demo on a
   different space/repo/Vercel URL — the Yugo demo ran as two overlapping
   builds on two deploys, never reconciled, and the duplication was a top
   token sink. If a second one already exists, stop and reconcile to one
   before continuing.

3. **Get the code into an owned private repo** — path-dependent:

   **Path A (duplicate the operator's starter template).** Clone it straight
   into the working dir, drop the template remote, create+push the private
   repo in one `gh` call — one clone, no scratchpad, no re-clone:
   ```
   git clone <template-repo-the-operator-named> \
     ~/dev/accounts/<Customer>/<customer>-storyblok-demo
   cd <that dir>
   git remote remove origin
   gh repo create <customer>-storyblok-demo --private --source=. \
     --remote=origin --push
   ```
   `--push` creates AND pushes in one step, and the working dir is already the
   clone — the first run's scratchpad + re-clone was pure waste.

   **Path B (Storyblok CLI scaffold).** Scaffold with the CLI in the framework
   the operator chose (step 1), then own it the same way:
   ```
   npx storyblok create ~/dev/accounts/<Customer>/<customer>-storyblok-demo \
     --template <framework>          # verify current flags: storyblok create --help
   cd <that dir>
   git init && git add -A && git commit -m "Initial scaffold"   # if create didn't
   gh repo create <customer>-storyblok-demo --private --source=. \
     --remote=origin --push
   ```
   `storyblok create` also scaffolds/links a Storyblok space for that
   boilerplate — so on Path B the space is handled here. CLI flags change
   across versions (the `--token`/`--skip-space` flags are recent) — check
   `storyblok create --help` rather than trusting the syntax above verbatim.

   **Both paths:** never use GitHub's native Fork — it defaults to public and
   carries visible "forked from …" lineage, wrong for a client demo repo
   (operator decision, 2026-07-24: "a private repo, so I can work on it
   without touching the team's template"). Repo name: lowercase-kebab,
   `<customer>-storyblok-demo`.

4. **Apply known template fixes, then audit the component library.** If the
   operator's starter template has recurring boot-breaking bugs, the fix list
   lives in local `memory.md` (Darwin loads it each session) — apply those
   fixes fresh on every new customer repo, and **re-verify each still
   reproduces** before editing (the upstream template may have been patched).
   On Path B skip the known list — a fresh boilerplate won't have those files.

   **Then run these three audits on either path, before authoring any content.**
   Both problems are invisible until an editor hits them live, which is the
   worst time to find out:

   a. **Unguarded field access → editor 500s.** Grep the component library for
      fields read without optional chaining:
      ```
      grep -rnE "blok\.[a-zA-Z_]+\.(length|filename|alt|items)" \
        --include="*.vue" <components-dirs> | grep -v "?\."
      ```
      A block added in the Visual Editor arrives with **only `component` and
      `_uid`** — every array and asset field is undefined — so
      `blok.cards.length` throws and takes down the whole preview with a 500.
      The effect is that a marketer cannot add those blocks at all, which
      breaks the "non-technical staff can edit this" pitch the demo exists to
      make. Fix with optional chaining across the library (a mechanical
      regex pass), then `npm run build` to confirm nothing broke. Watch for
      half-guards the regex leaves behind, e.g. `blok.form?.items[0].id`
      still throws — it needs `?.items?.[0]?.id`.

   b. **Hardcoded root paths → multi-site breakage.** If the space uses a
      folder per brand, grep for route/story lookups that assume a
      single-site space at the root (product routes, template lookups,
      internal links). Anything comparing `slug[0]` to a fixed folder name,
      or fetching `<folder>/<story>` from the root, resolves to the wrong
      place — or nothing — once content lives under `<brand>/...`. Resolve
      the folder from the current route instead, in one shared composable
      that every call site uses.

      **Same trap on the locale axis, and it is the expensive one.** A
      folder-per-locale (Dimensions) site whose frontend hardcodes the
      locale list (`LOCALE_FOLDERS = ['en-us','en-gb']`) 404s the moment an
      editor adds a locale folder from the CMS — the demo's own "add a
      market yourself" pitch breaks at the worst moment. Derive the locale
      list dynamically (space `language_codes` via `cdn/spaces/me`, or the
      top-level folders) so a CMS-only locale add needs zero frontend code.
      And **lock the locale model before building the frontend**:
      folder-per-locale (Dimensions) vs field-level i18n couples directly to
      the routing, so switching after the frontend exists rewrites both the
      tree and the router — the single biggest time sink on the Yugo build,
      which decided the model twice.

   c. **Asset & interactive rendering — components must honour the editor's
      own controls.** Whenever a component renders a Storyblok `asset`/
      `multiasset` or a dynamic element, check three things — each was a real,
      separately-shipped bug in one build:
      - **Focus point must drive the image.** With `object-cover`, never
        hardcode `object-position`. Convert the asset's `image.focus`
        ("x1xy1:x2xy2", original pixels) into a CSS `object-position` computed
        from the image's natural size, recomputed on `@load` and on a `watch`
        of the focus. Otherwise the editor's focus-point control silently does
        nothing — a "why doesn't this work" the operator hits every time.
      - **Images need a height or they collapse.** An `<img class="h-full">`
        in a wrapper with no defined height renders ~80px tall. Give the
        wrapper an explicit/responsive height (e.g. `h-72 md:h-[500px]`) or an
        aspect ratio; don't rely on `h-full` alone.
      - **Dynamic `:is` needs a resolved component, not a string.**
        `<component :is="'NuxtLink'">` renders a dead `<nuxtlink>` tag with no
        `href`. Use `resolveComponent('NuxtLink')` (or real `<NuxtLink>` /
        `<div>` branches). Same for any component name passed to `:is`.
      After any such change, verify on the DEPLOYED page that the rendered
      `object-position` / height / `href` actually reflect the intent — a
      successful build is not proof (guardrail 12).

   d. **Components you BUILD yourself — test every option/variant they expose,
      on the real surface.** A component with a variant field (`media_type`
      image/video/youtube, `layout`, `card_style`, columns…) is only as done
      as the combination you actually tried; wiring and verifying one path is
      not the component. Enumerate the field's values and click each on the
      DEPLOYED page — especially the headline use. Real miss (Boulanger,
      2026-08-26): a `page-hero` media picker was tested only as YouTube on an
      *inline* featured block, so the **full-bg + video/youtube** path was
      never wired and the hero's own primary use silently rendered nothing
      until the operator hit it. Build a quick matrix (layout × media_type,
      light/dark bg, empty vs filled) and run it before declaring done. And
      **before inventing a pattern the operator judges on feel** (a media/
      header picker, a hero), check prior demo repos first — the operator
      pointed at Yugo's media header as the reference; a proven pattern they
      already liked beats a fresh invention (`~/dev/accounts/*/` and the Yugo
      repo).

   Commit and push the fixes.

5. **`npm install`, then check `git diff` before committing.** Benign lockfile
   metadata churn (stale `peer`/`extraneous` flags, the package-name field) is
   fine to commit — but watch for silent dependency *version* bumps (an
   install once bumped `swiper` to a new major). Revert
   `package.json`/`package-lock.json` if the diff includes version changes that
   weren't the intent; commit if it's only metadata.

6. **Once the space exists, retrieve its Preview token yourself** — don't make
   the operator hunt for it. Via Management API:
   `GET /v1/spaces` to find the space by name → `GET /v1/spaces/<id>/api_keys/`
   → the token whose `access` is `"private"` is the draft-capable Preview
   token (verify if unsure: it can fetch `?version=draft`; the `public` one
   returns Unauthorized on draft). Then write `.env` in the project root:
   - `STORYBLOK_TOKEN` = that private/Preview token.
   - `SHOPIFY_DOMAIN` / `SHOPIFY_TOKEN` = per the step-1 answer.
   - Leave any optional integration vars (analytics/personalization SDKs, PIM
     connectors, etc.) commented out unless that station is part of the demo.
   Confirm `.env` is gitignored (`git check-ignore -v .env`).

7. **Deploy to Vercel via CLI — not the dashboard.** The CLI path is fully
   automatable and is the default now (operator, 2026-07-24: "could have
   deployed yourself and add the variable yourself"). Netlify remains an
   untried fallback — only offer it if the operator asks. Sequence:
   ```
   npx vercel link --yes --project <customer>-storyblok-demo --scope <team>
   # add all vars to all three environments, non-interactively:
   for ENV in production preview development; do
     npx vercel env add STORYBLOK_TOKEN  "$ENV" --value "<token>"  --yes
     npx vercel env add SHOPIFY_DOMAIN   "$ENV" --value "<domain>" --yes
     npx vercel env add SHOPIFY_TOKEN    "$ENV" --value "<token>"  --yes
   done
   npx vercel --prod --yes    # builds WITH the env vars already set
   ```
   Doing env-vars-before-first-`--prod` avoids the dashboard flow's
   "redeploy needed after adding vars" gotcha entirely. If a project already
   exists, `vercel link` finds it; if `link` prints a `next[]` hint asking for
   `--scope`, re-run with the scope it names. The final `--prod` output gives
   both the deployment URL and the aliased
   `https://<customer>-storyblok-demo.vercel.app`.

8. **Verify the DEPLOYED URL renders real content — cheaply. Skip localhost
   entirely** (operator, 2026-07-24: "don't run it... you don't have to check
   that it works"). The deployed check is the real gate and it also confirms
   the space isn't empty. Do it with ONE lightweight check: `get_page_text`,
   or a single `innerText.includes('...')` via the JS tool. **Never use
   `read_network_requests`** on a Nuxt/Vite frontend like this — it inlines
   fonts as base64 `data:` URIs and the dump is enormous (the big token sink
   on the first run). Avoid screenshots unless a text check is ambiguous; a
   first "blank" check is usually a paint-timing race — re-check text.

9. **Wire the space `domain` AND both preview environments** via Management
   API (the MCP has no space-settings write op). Solutions Demo spaces ship
   with `domain` = `https://<hash>.me.storyblok.com/` — that host is a
   Shopify theme stub and the Visual Editor iframe shows
   **`page.liquid is missing`**. Setting `environments` alone is NOT enough:
   VE still defaults to `domain` (hit on two demo builds,
   2026-09-08 and 2026-09-15). Always PUT both in one call, Vercel first:
   `PUT /v1/spaces/<id>` with
   `{"space":{"domain":"https://<customer>-storyblok-demo.vercel.app/",
     "environments":[
     {"name":"Vercel","location":"https://<customer>-storyblok-demo.vercel.app/"},
     {"name":"dev","location":"https://localhost:3000/"}]}}`.
   Plain URLs, nothing appended — a `?token=...&path=...` query-param token
   *overrides* the env-var fallback, so pasting the wrong one there (e.g. a
   Shopify token by mistake) silently breaks auth (happened 2026-07-24).
   localhost is wired as "dev" for convenience but is NOT launched or
   verified by this skill. After the PUT: hard-refresh VE and confirm the
   location dropdown shows Vercel, not `me.storyblok.com`.

## Working with the Storyblok Management API / MCP

Mechanics learned the hard way — none of these are obvious from the tool
descriptions.

- **`page.liquid is missing` = wrong space `domain`, not a Liquid bug.**
  SE demo spaces default `domain` to `*.me.storyblok.com` (Shopify stub).
  Custom frontends must overwrite `domain` to the Vercel URL in the same
  PUT as `environments` (step 9). Environments alone leave VE on the stub.

- **Placeholder images must be uploaded as real CMS assets.** Frontends that
  run images through an image-service helper append a transform suffix
  (`/m/1200x0/filters:...`) to whatever URL they're given, which produces a
  broken image for any external host (`placehold.co` and friends). Generate
  simple placeholders locally and upload them: create the asset record, POST
  the file to the returned signed S3 URL, then finalize. Scripting that loop
  directly against the Management API is far faster than one MCP call per
  step per image.
- **Bulk writes get rate-limited (HTTP 429).** Anything looping over more than
  a handful of records (datasource entries, stories, assets) needs a small
  delay (~0.35s) plus retry-with-backoff on 429. Write the loop to be
  **idempotent and resumable** — check what already exists first — so a
  mid-run 429 can simply be re-run rather than creating duplicates.
- **Some move operations report failure but succeed.** `moveStory` /
  `bulkMoveStories` have returned an error ("Body is unusable: Body has
  already been read") while the move actually completed server-side. Don't
  retry blindly and don't assume failure — **re-read the affected stories and
  check their `full_slug`/`parent_id`** before doing anything else.
- **Check staleness cheaply before an update-in-place.** `updateStory`
  replaces `content` in full, so the re-fetch rule matters — but a full
  re-fetch is expensive. Read just `story.updated_at` with field selection
  and compare it to the timestamp the last write returned. Match → the held
  copy is current. Differ → someone edited in the editor, so re-fetch the
  whole thing and merge. This caught a real conflict twice in one session
  (an operator edit, and products added via the picker).
- **Content written by API bypasses editor-side validation.** Component
  whitelists and required fields aren't enforced, so a page can render
  perfectly while being un-addable or un-editable in the UI. After authoring,
  open the page in the Visual Editor at least once — rendering correctly is
  not evidence that it is editable. The inverse also happens: a field can
  exist in a story's content while the component schema never declared it, so
  it renders and the editor shows no field for it at all. `storyblok-content`
  carries the full list of schema semantics that produce this signature —
  read it before authoring, not after the operator finds an empty picker.
- **A datasource-backed `options` field is a reliable stand-in for a flaky
  picker app.** If a commerce picker plugin only loads part of the catalog
  (so recently-added products can't be found), sync the catalog into a
  datasource as name/value pairs and point a native multi-select at it. The
  editor gets real search over everything, in a chosen order. Say plainly
  that the datasource is a snapshot needing re-sync, and that a
  collection/category-driven field avoids the staleness entirely.

## Efficiency notes (what changed after the first run, and why)

- **Human-gated asks go first** so they overlap the code work instead of
  blocking at the end.
- **One clone, no scratchpad** (`gh repo create --source=. --push`) — the
  first run cloned twice.
- **Vercel via CLI**, env-vars-before-first-deploy — no dashboard, no
  Netlify-vs-Vercel question, no post-hoc redeploy.
- **No localhost run/verify** — the deployed URL is the single render gate.
- **Cheap verification only**: `get_page_text`/`innerText`, never
  `read_network_requests` (base64 fonts) and rarely a screenshot.
- **Don't churn MCP `search`** for space-settings writes — they aren't in the
  connector; go straight to Management API curl for token + environments.

## Guardrails

- Tokens (Storyblok Preview, Shopify Storefront) live only in that project's
  own local `.env`, gitignored by the template itself — never in the darwin
  repo, never in any tracked file. A Storefront token is public/read-only by
  design and safe to receive in chat; nothing else is.
- **Template-specific specifics stay in local memory, never in this tracked
  file** — which starter repo the operator uses, its exact files/bugs/fixes,
  and any CMS-seeding quirks are confidential to the operator's setup. This
  skill names none of them; it reads them from `memory.md` at run time.
- **Never create a CMS space that would render blank.** If the operator's
  template needs a space pre-seeded with matching component schemas, Darwin
  cannot produce that via a raw "create space" API call — a blank space fails
  silently (renders nothing, no error). Ask the operator to create/seed it and
  wait for confirmation. (Path B is different: the Storyblok CLI legitimately
  scaffolds a seeded space for its own boilerplate — fine, it's not blank.)
  On either path, once the space exists, retrieving its Preview token IS
  Darwin's job (step 6) — don't make the operator fetch and paste it.
- The render-verification gate is the **deployed URL**, not localhost — and it
  must be a real content check (text actually present), not just a 200 or "no
  crash," because an empty space renders cleanly with zero content. Skip
  launching/verifying localhost entirely (operator preference, 2026-07-24).
  Keep verification cheap: `get_page_text`/`innerText`, never
  `read_network_requests` (inlined base64 fonts). **Prefer DOM assertions over
  screenshots** — a `querySelectorAll` check of section order, headings and
  link hrefs is cheaper, more precise, and more reliable than an image.
  Screenshots repeatedly came back blank mid-session from paint timing; treat
  one blank frame as a rendering race, not evidence of breakage, and confirm
  with a text check before chasing a non-existent bug.
- **Publish, don't just save.** Once the frontend serves published content to
  normal visitors, anything created or updated via the API sits in draft and
  is invisible on the live URL, while looking perfect in the Visual Editor
  (which reads draft). Pass the publish flag on create/update, and before
  handing over, list the folder's stories and confirm none are still
  unpublished. Duplicated stories inherit the original's published state;
  folders themselves always read as unpublished, which is normal.
- **Reference galleries can belong to a different template.** A component
  screenshot gallery shipped with a replication skill was captured from an
  older, since-abandoned starter template, so it did not reflect how this
  template's components actually render. Treat such galleries as indicative
  only — **the authoritative answer is the component source in the repo being
  built**. Reading the Vue file is what revealed that one card component
  renders its image as a full-bleed background with overlaid white text,
  which no gallery PNG would have shown.
- **Read each message for which mode is active before assuming a default.**
  The operator has, in the same session, asked to type every command
  themselves *and* asked for direct edits/commits ("do it yourself") at
  different points — don't assume one mode holds for an entire session
  (2026-07-24).
- Treat "abort" / "NO NO" as an immediate hard stop — revert whatever's
  in-progress without a clarifying question first, then ask what to do next
  (2026-07-24).
- A Shopify Storefront token can read products/collections and drive a mock
  cart, but cannot create/edit them — that needs a separate Admin API token
  (or just creating products by hand in the Shopify admin UI, which needs no
  token and is fast enough for a dozen demo products). Don't conflate the two.
- Some templates drive certain content blocks from a **live** Shopify catalog
  via a custom CMS field plugin (an editor picks real products in the Visual
  Editor). Mocking product data in code silently breaks those blocks. If the
  operator's template has such blocks (tracked in local memory), use real
  Shopify products rather than mocks. Generic caveat here; specifics in memory.
- **Building a custom field plugin: use the official CLI, never hand-inject
  it via the Management API.** Scaffold with `npx @storyblok/field-plugin
  create`, build, and deploy it — the CLI registers `body`/`compiled_body`/
  `space_ids` correctly. Writing those fields directly onto a `field_type`
  via MAPI produces a plugin that silently never renders in the editor (the
  Yugo `pms-picker` burned a long session exactly this way). A field plugin
  also **cannot be verified headlessly** — it only renders inside the open
  editor — so debugging needs the operator's editor console, and you keep a
  datasource-backed `options` fallback ready (see the picker stand-in above)
  so a residence/room choice is never lost while the plugin is in doubt.
