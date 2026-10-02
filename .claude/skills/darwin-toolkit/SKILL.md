---
name: darwin-toolkit
description: >
  Trigger during darwin-setup on a new computer, on request to install or
  verify Darwin's external toolkit, before a frontend landing-page/redesign
  task that needs the Taste skill, when a React project needs a real 3D
  interface, when OmniRoute could provide model routing or AI tools, or when a
  task needs external public data. Installs or connects supported capabilities
  safely: the design-taste-frontend skill, React Three Fiber when the project
  is compatible, OmniRoute through an official Cursor plugin if one exists or
  otherwise MCP, and the public-apis/public-apis catalogue as a discovery
  index with independent provider verification.
---

# darwin-toolkit

## Purpose

Make four optional capabilities reproducible on any computer running this
Darwin repository:

1. `design-taste-frontend` for marketing pages, landing pages, portfolios and
   visual redesigns.
2. React Three Fiber for justified 3D experiences in React applications.
3. OmniRoute for model routing and its agent tools.
4. `public-apis/public-apis` for discovering external data sources.

This is a bootstrap and routing skill. It does not copy third-party
documentation into Darwin, commit credentials, or assume that a capability is
installed merely because another computer had it.

## Run modes

- **Bootstrap:** during `darwin-setup`, check the global/project capabilities
  and offer to install/connect anything missing. React Three Fiber is only
  checked inside a relevant application, never installed into Darwin itself.
- **Task-triggered:** check only the capability relevant to the current task.
- **Audit:** report installed, reachable, missing and blocked items without
  changing the machine.

Installing software, installing a plugin, or changing Cursor's global MCP
configuration requires one concise preview and the operator's approval.
Read-only discovery and health checks do not.

## 1. Taste skill

### When to use it

Use `design-taste-frontend` for landing pages, portfolios, marketing surfaces
and visual redesigns. Do not force it onto dashboards, dense admin UI, data
tables or multi-step product workflows; the skill explicitly excludes those.

### Check and install

1. Check the available skills and `npx skills list --json` for
   `design-taste-frontend`.
2. If present, read its `SKILL.md` before the frontend task and follow it.
3. If absent, preview this project-level install and ask for approval:

   ```bash
   npx skills add Leonxlnx/taste-skill \
     --skill design-taste-frontend \
     --agent cursor \
     --yes
   ```

4. Verify with `npx skills list --json` and read the installed `SKILL.md`.
5. Inspect `git status`. The installer may create `.agents/` and
   `skills-lock.json`; never sweep them into an unrelated commit. Explain what
   appeared and stage them only when the operator explicitly wants the
   third-party skill or its lockfile versioned.

Do not reproduce or paraphrase the Taste rules here. The installed skill is
the source of truth.

## 2. React Three Fiber

Official documentation:
[`r3f.docs.pmnd.rs`](https://r3f.docs.pmnd.rs/getting-started/installation).

### When to use it

Use React Three Fiber (R3F) when all of these are true:

- the application uses React;
- 3D materially improves the requested experience, rather than decorating it;
- the interaction needs a scene graph, camera, lighting, models or
  frame-driven animation that normal CSS/DOM cannot express well;
- the bundle, rendering and mobile-performance cost is acceptable.

Do not install it during general Darwin setup. Do not use it in Vue/Nuxt:
R3F is a React renderer. For non-React projects, select a framework-native
renderer or Three.js directly after checking the existing stack. Do not rewrite
a working application into React merely to use R3F.

### Install safely

1. Read the target application's `package.json` and determine its React major,
   package manager and rendering framework.
2. Preserve the existing package manager and lockfile.
3. Pair versions correctly:
   - React 18 → `@react-three/fiber@8`
   - React 19 → `@react-three/fiber@9`
4. Preview the dependency change, then install in the target application:

   ```bash
   pnpm add three @react-three/fiber
   ```

   Use the project's actual package manager. Add `@react-three/drei` only when
   a required abstraction justifies it; it is not part of the base install.
5. For Next.js, add `three` to `transpilePackages` only if untranspiled
   ecosystem add-ons require it. Do not change configuration speculatively.

### Build and verification rules

- Keep the canvas in a client-only leaf; do not force the page tree to become
  client-rendered.
- Lazy-load non-critical scenes and large models.
- Reserve canvas dimensions to prevent layout shift.
- Dispose geometries, materials, textures and listeners.
- Honor reduced motion and provide a useful static fallback.
- Optimize GLB/GLTF assets and textures before shipping.
- Test the actual interaction on desktop and mobile, including low-power
  behavior, resizing, context loss and keyboard/pointer alternatives.
- Check bundle impact and Core Web Vitals. A visually impressive scene that
  breaks the primary content or demo path fails.
- Reuse the Taste skill's design direction, but do not add 3D merely because
  the library is available.

## 3. OmniRoute

### Choose the supported integration at runtime

Use this order:

1. If OmniRoute tools already exist in the current tool catalogue, inspect
   their schemas and use them.
2. Check whether an **official OmniRoute Cursor plugin** currently exists.
   Verify its publisher/source against `diegosouzapw/OmniRoute`; never install
   a similarly named third-party package. If an official plugin exists and
   exposes the required tools, preview its permissions and ask before
   installation.
3. Otherwise use OmniRoute's built-in MCP server. MCP configuration is not the
   same thing as installing a Cursor plugin.

The public repository currently documents Cursor as an OpenAI-compatible
client and MCP client. Never claim a Cursor plugin exists without runtime
evidence.

### Prefer one remote instance for several computers

For the same routing, providers, budgets and usage history on every computer,
prefer one secured, always-on OmniRoute instance over separate local installs.
Ask for:

- the HTTPS or Tailnet base URL, without secrets;
- whether the MCP endpoint is enabled with `streamable-http`;
- a dedicated MCP API key carrying only the scopes needed. Remote MCP access
  requires `manage`; do not reuse a broad personal/admin credential.

Merge an entry into the operator's global `~/.cursor/mcp.json`; never replace
existing servers:

```json
{
  "mcpServers": {
    "omniroute": {
      "url": "https://<host>/api/mcp/stream",
      "headers": {
        "Authorization": "Bearer ${env:OMNIROUTE_MCP_TOKEN}"
      }
    }
  }
}
```

The token belongs in the OS keychain/environment as
`OMNIROUTE_MCP_TOKEN`, never in this repository, `memory.md`, chat output or
the JSON file. Use HTTPS or a Tailnet, not a bare public HTTP endpoint.

### Local fallback

If no remote instance exists and the operator wants a local installation:

1. Preview and ask before `npm install -g omniroute`.
2. Start/configure OmniRoute according to its current official documentation.
3. Merge this stdio entry into `~/.cursor/mcp.json`:

   ```json
   {
     "mcpServers": {
       "omniroute": {
         "type": "stdio",
         "command": "omniroute",
         "args": ["--mcp"]
       }
     }
   }
   ```

4. Provider credentials remain in OmniRoute's encrypted store or the OS
   keychain. Never move them into Darwin.

### Verify the real surface

After Cursor reloads:

- discover the OmniRoute MCP namespace dynamically;
- run a health/read-only tool;
- list models or routing combos;
- if inference is needed, make one harmless test request and report the chosen
  provider/model;
- do not mutate providers, combos, budgets, keys or routing policy unless the
  task explicitly asks for it.

Configuring Cursor's **model picker/chat** through OmniRoute is separate from
MCP. Use `omniroute setup-cursor` for current instructions; it prints manual
Cursor steps because Cursor stores that configuration in opaque local state.
Do not edit Cursor's internal SQLite.

## 4. Public API discovery

### Correct role

[`public-apis/public-apis`](https://github.com/public-apis/public-apis) is a
lead-generation catalogue for APIs, not an authority on reliability, accuracy,
licensing, security or current availability. The catalogue's MIT license does
not license any listed API or its data.

Use it when a demo/prototype needs public external data such as weather,
geocoding, currency, transport, public records or safe test data. Do not use it
when an existing connected system is the source of truth, and never send
customer-confidential or personal data to a discovered API.

### Discovery workflow

1. Fetch the current README from GitHub with `gh`; do not depend on
   `api.publicapis.org`:

   ```bash
   gh api repos/public-apis/public-apis/contents/README.md \
     -H 'Accept: application/vnd.github.raw+json'
   ```

2. Search only the relevant category and keywords. Shortlist at most three
   candidates.
3. Prefer official government, standards-body, first-party or established
   open-data providers over aggregators and self-promotional listings.
4. For each candidate, open the provider's own current documentation and
   verify:
   - endpoint and response shape;
   - authentication and secret placement;
   - free-tier limits, quotas and rate limits;
   - commercial and sales-demo usage rights;
   - data license, attribution and retention requirements;
   - HTTPS, CORS and whether calls must be server-side;
   - freshness, geographic coverage and deletion/deprecation notices;
   - privacy implications and prohibited data.
5. Make one harmless live request. A documentation page returning 200 does not
   prove the API endpoint works.
6. Recommend one candidate and up to two alternatives. Mark anything not
   verified as `need validation`.

### Demo integration rules

- Prefer a dated, attributed fixture captured from a verified response when
  live updates add no demo value.
- Use a live dependency only when live behavior materially improves the demo.
- Put keys and non-CORS calls server-side.
- Add timeouts, caching, loading/empty/error states and a deterministic
  fallback.
- Never make the critical demo path depend solely on an unproven free API.
- Label simulated or cached data honestly; never present it as live.

### Output

Report:

- recommended API and why;
- alternatives;
- provider documentation link;
- auth, cost, limits, CORS and usage/license constraints;
- live verification performed and date;
- proposed client/server integration;
- reliability risk and fallback.

## Completion report

End bootstrap/audit with:

- Taste: installed and readable | missing | blocked
- React Three Fiber: not applicable | existing and version-compatible |
  installed and verified | blocked
- OmniRoute: plugin or MCP path, local/remote, health result | missing | blocked
- Public APIs: catalogue reachable; no API selected unless the current task
  required one
- any per-machine action still required, without printing secrets
