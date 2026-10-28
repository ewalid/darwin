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

## First five minutes — before anything else

Two failures repeat across builds and both happen in the first minutes, so this
block runs before the kickoff checklist, in this order:

1. **Invoke this skill through the Skill tool**, visibly. Reading the SKILL.md
   and following it silently is what the operator called "you didn't use the
   skills" (2026-10-23).
2. **Inspect the space with the Storyblok MCP** (`getSpace`, `listStories`,
   `listManagementComponents`), not curl. See "MCP gaps" below for the few
   things that need the Management API.
3. **Point the editor at the frontend NOW, not at step 9.** Pick the Vercel
   project name at once (`<customer>-storyblok-demo`, URL
   `https://<customer>-storyblok-demo.vercel.app/`) and PUT the space `domain`
   + environments to that predicted URL immediately, before any code exists.
   A 404 there is harmless; the Shopify stub is not. Five builds hit
   `page.liquid is missing` (2026-09 to 2026-10-23) because the domain was set
   hours in, after the operator had already opened the Visual Editor on the
   stub and the editor had remembered that location.
4. **Tell the operator, in the kickoff message:** "If you already opened the
   Visual Editor, pick 'Vercel' in its location dropdown (next to the
   desktop/mobile icons) and hard-refresh." And ask them to log in **inside
   the browser pane** (it has its own session, separate from their browser),
   so the editor can actually be checked (guardrail 12).

## Kickoff checklist — one message, then build

A full same-day build (2026-10-06) was fast but still lost its time in the
same places: decisions Darwin made alone that were the operator's, a router
built for one locale axis, a field contract applied after the fact, and a
hero rebuilt instead of reused. Do these up front:

1. **Ask the operator-owned decisions in ONE early message, and keep building
   whatever doesn't depend on them while waiting.** Never "lock" these alone,
   even under time pressure. Give the evidence and a recommendation, then let
   the operator choose:
   - **Locale model:** field-level translation, market folders (Dimensions),
     or both. **Whatever the answer, build the router for both axes from
     day one** (`/de/...` = folder, `/de/fr/...` = translation of an `fr/` page,
     `_storyblok_lang`). It is cheap, and switching later rewrites the tree and
     the router.
   - **Page types:** which folder gets a locked template, and which pages
     (home!) stay on the flexible page type. Lock only what was asked for.
   - **Design direction and site chrome:** mirror the prospect's live chrome
     (announcement bar, nav, footer). Add no chrome they don't have (e.g. a
     language switcher) unless asked.
   - **Browser pane logged into the CMS,** so the Visual Editor can actually
     be checked (guardrail 12).
2. **Use the Storyblok MCP for every CMS operation it covers** (inspecting the
   space, folders, stories, workflows, workflow stage changes, assets). Fall
   back to Management API scripts only for what the MCP lacks: space settings
   (`domain`, environments, languages), space roles, and the idempotent
   schema / preset / datasource scripts. Say which path you used and why. The
   operator corrected a build that inspected the space with curl while the MCP
   was connected (2026-10-23).
   **Announce reuse before porting it.** When you take plumbing from a previous
   demo, say in the kickoff message which demo and which parts (router,
   wrapper, hero, scripts), and that its brand, copy and content scripts get
   stripped. Porting silently reads as "why are you using another customer's
   stuff?" (2026-10-23).
3. **Start from proven components, not a blank file.** Check the local
   reuse registry (`resources/component-reuse/registry.md`) for the hero,
   image/video media, rich-text body, locale router, landing-page template,
   announcement bar and build scripts. The operator names past demos as the
   reference ("like we did for X"); the registry says where X lives.
4. **Apply the component contract on the FIRST schema push,** not after
   feedback: rich-text body copy, image-or-video media, default presets,
   help text on every option, theme colour roles (see "Standing build
   defaults").
5. **Verification mechanics:**
   - Navigate with a cache-buster (`?v=n`) after each deploy. A stale bundle
     looks exactly like a failed fix.
   - Run the URL matrix for routing, including the editor's translation path
     `<lang>/<full_slug>`.
   - Run the block × theme contrast matrix and a 375px width check.
   - Re-read the space settings at the end.
   - Run `vercel deploy` with a 5-minute timeout or in the background (the
     2-minute default cuts it off).
6. **Brief research agents with facts, not guesses.** Read the space name
   and the prospect's site before describing the prospect in a subagent
   prompt.
7. **Parallel builder agents: give each a disjoint file list** (blocks +
   their schema modules), one reference block to copy, and the contract.
   Two traps from a build that ran three agents at once (2026-10-23):
   - **Renaming a ported prefix:** never blanket-`sed` a short prefix that
     collides with Tailwind utilities (`pl-` is also padding-left). Match
     the identifier forms explicitly (`Pl[A-Z]`, `pl-<component>`,
     `--pl-`), then lint.
   - **The starter's pre-commit hook lints the whole working tree,**
     including agents' unfinished files. Commit when agents finish. If you
     must commit mid-flight, lint only your staged files, commit with
     `--no-verify`, say so, and let the final commit go through the hook.
   - **Assert every scripted edit matched.** The starter's `eslint --fix`
     reformats CSS and Vue files, one-line rules included, so a later
     `str.replace` against the text you originally wrote silently does
     nothing. Check the match count (`assert s.count(old) == 1`, or `re.subn`
     and check `n`), or use the Edit tool, which fails loudly. In regex
     replacement strings, never escape `.`: `"\\.bt-wrap {"` writes a
     literal backslash into the selector. (2026-10-24: three edits no-op'd
     and one wiped the mobile gutter before a deployed check caught it.)

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
   inconsistent naming: accented vs unaccented spellings, a bare `Group`,
   and per-site suffixes as separate folders for one account). If something
   matches, use it. Only
   create `~/dev/accounts/<Customer>/` if genuinely nothing matches — **ask
   the operator to confirm** on any ambiguity. Never silently create a second
   folder for the same customer under a different spelling. The
   same discipline covers the whole build: **one demo = one repo + one space
   + one deploy.** Don't start a second, parallel build of the same demo on a
   different space/repo/Vercel URL — one demo ran as two overlapping
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
      tree and the router — the single biggest time sink on one custom build,
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
      - **YouTube behind content needs cropping.** Even with `controls=0`, an
        autoplaying embed shows its title bar, play/skip buttons and logo for
        the first seconds. Size the iframe to cover its box (container
        query units: `max(100cqw, 177.78cqh)` × `max(100cqh, 56.25cqw)`),
        then oversize it ~1.3× inside an `overflow: hidden` box, add
        `pointer-events: none`, and judge it on a screenshot taken ~9s after
        load, not 2s. Accept a link or a bare ID (parse `watch?v=`,
        `youtu.be/`, `/embed/`, `/shorts/`).
      After any such change, verify on the DEPLOYED page that the rendered
      `object-position` / height / `href` actually reflect the intent — a
      successful build is not proof (guardrail 12).

   d. **Components you BUILD yourself — test every option/variant they expose,
      on the real surface.** A component with a variant field (`media_type`
      image/video/youtube, `layout`, `card_style`, columns…) is only as done
      as the combination you actually tried; wiring and verifying one path is
      not the component. Enumerate the field's values and click each on the
      DEPLOYED page — especially the headline use. Real miss (an FR retail demo,
      2026-08-26): a `page-hero` media picker was tested only as YouTube on an
      *inline* featured block, so the **full-bg + video/youtube** path was
      never wired and the hero's own primary use silently rendered nothing
      until the operator hit it. Build a quick matrix (layout × media_type,
      light/dark bg, empty vs filled) and run it before declaring done. And
      **before inventing a pattern the operator judges on feel** (a media/
      header picker, a hero), check prior demo repos first — the operator
      pointed at an earlier demo's media header as the reference; a proven
      pattern they already liked beats a fresh invention
      (`~/dev/accounts/*/` and prior demo repos).

   e. **Components someone else built (another agent, a parallel session,
      a Cursor build) — audit field by field before the dry-run.** Inherited
      work looks finished in the CMS and can be dead on the page. One demo
      morning (2026-10-27) found a form whose `width` field the renderer never
      read, an invisible submit button, a `required` Choices field that made
      every text field unsaveable, and page templates that rendered blank when
      empty. Three checks:
      - **Every schema field is read by its renderer.** Grep each field name
        of the component's schema in its Vue file; a field nobody reads is a
        control that silently does nothing. (The schema-vs-content validator
        only checks the other direction.)
      - **Theme tokens never resolve to `undefined`.** If the layout copies
        CMS colours into CSS variables, each one falls back to the default
        when the CMS value is empty (`?.color || defaults[key]`), or every
        section on that colour turns transparent. Check with
        `getComputedStyle(document.documentElement).getPropertyValue('--x')`.
        Fix it once in the layout, never per component.
      - **Change each field and watch the page change**, per block, on the
        real surface (audit d, applied to code you didn't write).

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

9. **Re-assert the space `domain` AND both preview environments** (first set
   in "First five minutes", step 3) via Management
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

   **The PUT response is not proof — the setting can silently revert.**
   On a freshly provisioned Solutions Demo space the whole settings object
   (`domain`, `environments`, `languages`) came back to its original values
   about an hour after a PUT that returned 200 with the new values: the
   provisioning job finishing late, or a Settings page opened earlier and
   saved afterwards with a stale form. The operator then opened the VE on
   `page.liquid is missing` — the third build to hit it (2026-10-06).
   **Second cause, same symptom: the editor remembers its preview location
   per space in the browser.** If the operator opened the Visual Editor
   before the PUT, the editor keeps showing the old `me.storyblok.com`
   location even though `domain` and `environments` are correct server-side
   (confirmed 2026-10-23: settings untouched in the audit log, fixed by
   picking "Vercel" in the editor's location dropdown). Before re-PUTting,
   read the space via the MCP (`getSpace` + `listActivities` type Space): if
   the settings are right and unchanged, tell the operator to pick the Vercel
   location in the dropdown and hard-refresh. Say this proactively in the
   hand-over, because the space is usually opened before the domain is set.
   So: (1) wait ~60s after the PUT and **GET the space again** — `domain`,
   `environments`, `options.languages`; (2) **re-GET it once more right
   before handing over** (it is a close-out gate, see below); (3) if it
   reverted, re-PUT and re-read; and tell the operator not to save the
   space Settings page from a tab opened before the change.

## Standing build defaults — never make the operator ask for these

Two full retail builds (a jewellery storefront, then a two-brand toy
retailer) each needed the operator to restate the same expectations mid-build
("style tabs on every component", "deploy and push", "don't leave locales
pending", "page.liquid again", "no session 2"). They are defaults now; apply
them without being asked and say in the plan that they are applied.

**Scope and cadence**
- Build the whole agreed scope before the demo. Do not split work into a
  "session 2" or "P2 if there's time" unless the operator proposes it — one
  build was re-planned mid-night because the split was Darwin's idea, not
  theirs.
- After every change: commit, `git push`, `vercel deploy --prod`, verify on
  the deployed URL. A repo with no remote is a defect — create the private
  GitHub repo at standup (one earlier demo lived only on one laptop, with no
  remote, and its previous copy was lost).
- Shared working trees: other sessions may commit in the same repo. Stage
  only your own paths (never `git add -A`), `git pull --rebase` before every
  push, and flag any file you did not write instead of committing it.

**Component contract (every block Darwin builds)**
- **Two tiers. Default to the simple one for anything marketers will edit,
  and name the tier in the kickoff message.** The operator judged the full
  tier "too complex for non-technical people" and asked for something fresh
  and simpler (2026-10-24).
  - **Simple (default):** General with a named **Layout** option first
    ("Text with image beside it", not `split`); **Style** = background as a
    colour *role* (cream / sand / purple / orange / navy / image; text colour
    is derived from it, so no colour pickers and nothing unreadable),
    space above / below, width, alignment, headline size and style;
    **Visibility** = hide on mobile / desktop, show from / until, anchor.
    Fields that only matter in one case are hidden with `conditional_settings`
    (`validated_object` + `equals` / `not_equals`), e.g. the background image
    only appears when Background = Image. Every block also gets a display
    name, description, icon and colour, plus a block-picker preview image
    captured from the component library page. Set that image on the block's
    **preset** (`image`) as well as the component: the picker's hover preview
    shows the preset image, so a component-only image never appears
    (verified in the editor, 2026-10-24).
  - **Full (on request, or for a technical audience):** General, **Style**
    (background colour/custom/image/video with overlay, text colour, inner
    and **outer spacing**, alignment, width, shape, title size), **Mobile**
    (hide, spacing, alignment, title size; mobile-only, never changes
    desktop), **Planning** (visible from / until). All brand tokens.
- One shared wrapper renders the Style/Mobile tabs; one shared composable
  renders Planning: hidden on the live site outside its window, always visible
  with a dated badge in the Visual Editor, `?preview_date=` to simulate "now".
- Nested blocks accepted by several hosts (a marketing tile in a catalogue
  grid AND in a home collection) must honour their fields in **every** host —
  a tile's Position/Width worked in one grid and was silently ignored in the
  other until the operator hit it in the editor.
- **Body copy is rich text.** Descriptions, intros, answers and paragraphs
  use a `richtext` field with a trimmed toolbar (bold, italic, link, lists).
  Titles, labels and one-liners stay `text`. Render through one shared
  wrapper that also accepts a legacy plain string, and migrate existing
  content in place rather than re-seeding it (operator preference, 2026-10-06).
- **Media fields take image or video** (`filetypes: images, videos`), rendered
  by one shared component. Vue sets `muted` as a property, so autoplay needs
  `el.muted = true` and an explicit `play()` on mount and on asset change.
- **Fields that only apply to some variants are never `required`.** Hide
  them with `conditional_settings` instead: a required field is enforced on
  API writes even when hidden, so a Choices field required on every form
  field blocked saving plain text fields (2026-10-27).
- **Every page template renders a visible empty state** for missing media,
  bio or video ("Add a video in Storyblok", a silhouette with "Photo coming
  soon"), never a blank area. A page created from a preset must look like it
  is waiting for content, not broken.
- **Every block ships a default preset.** A block added in the editor
  arrives empty (only `component` + `_uid`); a default preset makes it
  arrive with placeholder copy and a brand image. Re-check `preset_id`
  after re-running the schema script.
- **Theme tokens are colour roles, not fixed colours.** Before calling a
  themed library done, render every block × every theme on a throwaway draft
  page and measure text contrast against its real background (AA: 4.5, or
  3 for 24px+), then delete the page. One audit found an invisible primary
  button, an invisible quote and an invisible highlight that way.
- **Unlayered CSS beats Tailwind 4 utilities.** A custom class that sets
  `display` (a button) silently overrides `hidden` / `md:inline-flex`.
  Put component classes in `@layer components`, and check the page
  width equals the viewport at 375px.
- **Global resets go in at zero specificity.** A layout-wide reset such as
  `.site a { color: inherit }` (0,1,1) out-ranks every single-class
  component colour, so link-based buttons and tiles silently lose their
  text colour (dark text on a brand-colour button, 2.94 contrast, found
  only by the contrast matrix, 2026-10-28). Write resets as
  `.site :where(a)`.
- **No opacity-based scroll reveals.** An `animation-timeline: view()`
  fade that starts at `opacity: 0` leaves everything below the fold
  blank in full-page captures and in any frame that never scrolls (the
  editor iframe, thumbnails). Premium polish = hover states, shadows,
  spacing, never content that starts hidden.
- **Fullscreen sections are not cards.** Keep their viewport edges square.
  In nested sticky layouts, apply the fixed-header offset exactly once: if
  the parent panel already sticks below the header, its sticky child uses
  `top: 0`, not the header height again. Include a short-desktop gate around
  1024×500 and measure that the full copy and CTA remain inside the viewport;
  a standard laptop screenshot does not catch this class of clipping.
- **Pages accept nested blocks and atoms too**, not only sections (operator
  preference, 2026-10-23): the flexible page's body whitelists the sections,
  items and atoms groups, items render in a page-level frame, and site-config
  items live in their own group so they never show in a page's picker.
- Equal-size cards by default; "featured" layouts only when there are more
  items than columns.
- Schema scripts are the single source of truth. Never patch a schema by hand
  outside the script — re-running it silently reverts the patch (a font
  option added ad hoc disappeared on the next run).

**Language**
- Lock the locale model at kickoff and finish it. If an AI-translation beat is
  plausible, keep the languages in the space and mark copy fields
  `translatable` from day one — languages removed "for simplicity" had to be
  re-added and 180 fields flipped the night before a demo.
- Do not pre-fill translations, and do not pre-build the page a live beat
  will create (AI page generation, live translation). Those are the operator's
  moves in the room; a pre-made copy spoils the beat and leaves the same page
  twice in the tree. Prepare the brief/prompt instead, and verify the
  mechanism on a throwaway you remove afterwards (2026-10-06).
- **Market folders AND field-level translation in one space** (Dimensions
  folders `fr/ de/ es/` whose pages also get translated field by field): the
  Visual Editor previews a translation as `<lang>/<full_slug>`, e.g.
  `de/fr/landing-pages/x`. A router that reads the first segment only as a
  market folder looks for `de/fr/...` and 404s. Read a language code followed
  by a market folder as "translation of that page", honour `_storyblok_lang`,
  and strip the `de/` that Storyblok already adds to resolved links, or nav
  links come out as `/de/de/fr` (2026-10-06).
- Field-level translation needs `use_translated_stories` **off** (on: both
  `?language=` and `<lang>/<slug>` 404). Internal links must carry the
  language prefix; brand/folder roots derived from `full_slug` must strip it.

**Design and content**
- Pull the real brand tokens (colours, fonts, SVG logo) from the prospect's
  live site before designing. Then show 3–4 visual directions as a quick
  inline mockup and let the operator pick or write their own — design
  direction is the operator's call, and "ugly" came back twice when it wasn't
  checked.
- Never use campaign banners with text baked into the image as hero/tile
  visuals — they look pasted-in and low-res. Use product packshots in a framed
  "product box" treatment, or clean imagery, with real editable copy.
- Commerce listing pages: filters in a sticky left sidebar (collapsible groups,
  counts, active chips, mobile drawer), not a pill bar above the grid.
- No real catalogue → a mock PIM: a JSON file of the prospect's own public
  products + API routes + datasources synced for editor pickers. Verify each
  product fact against its source page (an age range was wrong until checked).
- Location/segment content (a store, a region) → a personalisation wrapper
  with one variant per segment, a fallback ("choose your store") or hide mode,
  and an editor preview switcher.

**Surfaces and drafts**
- Ask at kickoff (with the other human-gated asks) for the operator to log the
  browser pane into the CMS, so the Visual Editor can actually be checked
  (guardrail 12). Two builds shipped with "not verified in the editor".
- **Blocking:** that login ask goes in the kickoff message, every build,
  not "when needed". A third build (2026-10-27) shipped schema tabs, presets
  and the A11y tab unchecked in the editor because the ask was never made.
- If the browser pane refuses the deploy domain (it refuses `*.vercel.app`),
  verify the live site in Claude in Chrome. Never report localhost as the
  live check.
- Before any write, check `unpublished_changes`. If the operator has a draft in
  progress, patch only the target block inside that draft, save without
  publishing, and say so — never overwrite or publish their draft.

**Close-out**
- **Hand-over block (always in the final message):** live URLs; "In the
  Visual Editor, the location dropdown must show 'Vercel' (pick it and
  hard-refresh if you see `page.liquid is missing`)"; what was verified on
  which surface; what was NOT verified (say "Visual Editor not checked by me"
  if the pane was never logged in); live beats left for the operator.
- **Independent visual reviewer before hand-over.** A subagent that did not
  build anything compares each home to the live site at 1440 and 375 and
  scores it; fix its high/medium items in one batch and re-measure. On the
  2026-10-23 build it caught the mobile and header gaps (6/10 → fixed).
- **Component library page** per site (and one in the group/master folder):
  every block with its main variants under numbered dividers, `noindex`. The
  operator uses it to see the whole kit quickly; build it once, duplicate it
  with the MCP's `duplicateStory`.
- Re-read the space settings (`GET /v1/spaces/<id>`) as the last step before
  handing over: `domain` and the first environment must be the Vercel URL,
  and the languages must be the locked set. Never report "preview wired"
  from the PUT response alone (step 9).
- Run a schema-vs-content validator before calling content done (undeclared
  fields, `max_length`, required, bloks whitelists, publish state) — a script
  for this lives in the latest demo's build folder (path in local memory).
  Re-run it after every content write, not once at the end: it is what
  catches a block placed in a locked template's list that the template's
  own whitelist forbids (renders fine, cannot be added in the editor).
- Run the URL matrix with **one page of every content type**, articles
  included, after the content exists. Page-level code that walks
  `story.content` must check `Array.isArray`: the same field name is a
  block list on one root type and rich text on another (`body`), and a
  spread over the rich-text doc took down every article page with a 500
  (2026-10-28).
- Finish every custom build by running **`component-reuse`**: log what this
  build created that recurs across demos, and update the plans. Plan only;
  never a PR or merge on the SE team repo.

## Working with the Storyblok Management API / MCP

**MCP gaps (checked 2026-10-23): the only reasons to use the Management API.**
Everything else (stories, folders, publish, duplicate, workflows and stage
changes, assets, components read) goes through the MCP.

| Need | MCP | Use |
|---|---|---|
| Space settings write (`domain`, environments, languages, `use_translated_stories`) | read only (`getSpace`) | MAPI `PUT /spaces/<id>` |
| Space API keys (Preview token) | none | MAPI `GET /spaces/<id>/api_keys` |
| Space roles | none | MAPI `/space_roles` (script) |
| Experiments: create, activate, results | none | MAPI `/experiments` (script); results accept `bar`, `line`, `text` only |
| Bulk component schemas + presets | per-call only | idempotent schema script (single source of truth) |

Say in one line which row applies when you use the Management API.

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
- **Content through the MCP means full-story rewrites.** `updateStory` has
  no partial patch, so every later fix resends the whole page. Run the
  independent visual review and the validator on the FIRST content pass,
  then batch all fixes for a story into one write; don't trickle small
  edits (an SEO title, one block) into separate rewrites.
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

- **Folders and start pages.** Creating a folder with `default_root` can
  auto-create an empty `<folder>/<folder>` start-page story — list the folder
  after creating it and remove empties. Setting `is_startpage` on a `home`
  story renames its `full_slug` to `<folder>/` and breaks any route that fetches
  `<folder>/home`.
- **MAPI/MCP write validation is partial and type-strict.** Story writes
  reject empty required fields (so a "block added empty" test page can only
  hold blocks without required fields; presets cover the rest), `number`
  fields must be sent as strings (`"8"`, not `8`), and a story inside a
  workflow can't be published via API until it reaches a stage that allows
  publishing (move it with `createWorkflowStageChange` first). The MCP's
  `updateStory` also takes `ai_translate_language`, a backup path for a
  live AI-translation beat.
- **Schema before content, and stop on failure.** A schema script writes
  and re-fetches the schema first, and exits if any schema write failed,
  before it writes stories. One run wrote a story after its schema update
  returned 422 and left a half-written, unpublished draft behind
  (2026-10-27).
- **Asset filenames are lowercased** on upload — look assets up
  case-insensitively.
- **Space languages: send them top-level, then re-read.** `PUT /spaces/<id>`
  with `{"space":{"options":{"languages":[...],"use_translated_stories":false}}}`
  returns 200 and changes nothing. The working shapes are
  `{"space":{"languages":[{"code":"de","name":"German"}]}}` and
  `{"space":{"use_translated_stories":false}}`. Check `options.languages`
  in the response, not the status code (2026-10-27).
- **A market folder named like a language code is read as a language.** The
  starter's `getLanguage` treats the first URL segment as a language when it
  matches a space language code, so `es/home` becomes "home in Spanish" and
  404s. Before building market folders (`fr/`, `es/`, `de/`), remove those
  codes from the space languages, or name the folders differently.
- **Space-setting changes and the CDN cache.** After changing a space setting
  (languages, translated stories), cached 404s keep being served until the
  cache version bumps — republish any clean story to force it.

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
- **Don't churn MCP `search`** for the rows in the MCP-gaps table; go straight
  to the Management API for exactly those, and use the MCP for everything else.

## Visual fidelity on high design-language-gap sites

When the ask is "make it look like their live site" (high design-language gap),
the expensive failure mode is **inventing a coherent aesthetic** (e.g. a dark
editorial blog) and polishing brand hex on top — then learning from the
operator's side-by-side screenshots that the source was light all along.

**Before any Vue/React restyle of a marketing surface:**
1. Open source + demo at the **same viewport**.
2. Write a **≤5-row structural delta table** (page/section bg, chrome, meta
   row, sidebar, body invert / typography role). **Dark vs light is
   first-class**, not a color tweak.
3. Change only those deltas. Do **not** ship a "match brand colors" commit
   on a surface that failed the structure compare — accurate tokens on the
   wrong layout make fiction look intentional.
4. **One surface to PASS** before the next (home → listing → detail → LP).
   Scope sprawl without a lock is what turns a short CSS job into a day.

Prefer the team `replicate-site` / `replicate-section` quality gate
(screenshot → reviewer) when the operator invoked it. Freehand restyles
still owe the delta table.

**Hard ban — content-swap is not fidelity.** When design-language gap is
HIGH (or the operator asked for the live site look), do **not** ship a
"real" marketing surface by pasting prospect copy into the starter
showcase blocks (`hero-section`, `image-text-section`, sparkle frames,
Brand New Day layouts). That fails the structure compare even if the
words match. Build `sw-*` (or equivalent) Vue that mirrors the source
layout, or run `/replicate-section` section-by-section. Operator
correction 2026-10-26 (Succession home).

**Assets:** lock **one** delivery path at kickoff (prefer the CMS asset
library / Storyblok CDN for demos). Do not mid-flight hop through a
third-party image host and back — each hop forces re-wire, re-publish,
and re-check. If a third-party URL must be used temporarily, never leave
Cloudinary-style `f_auto` / `q_auto` transforms on **SVG** logos (they
corrupt or 404); strip transforms or re-host the SVG on the CMS CDN.


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
  one build's custom picker plugin burned a long session exactly this way).
  A field plugin
  also **cannot be verified headlessly** — it only renders inside the open
  editor — so debugging needs the operator's editor console, and you keep a
  datasource-backed `options` fallback ready (see the picker stand-in above)
  so a residence/room choice is never lost while the plugin is in doubt.
