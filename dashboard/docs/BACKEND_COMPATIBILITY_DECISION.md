# Backend Compatibility Decision: Frontend-Only User Plugin

Status: decided
Date: 2026-09-23

## Context

Hardened Hermes serves static dashboard assets from non-bundled user/project
plugins but refuses to import their Python backend APIs (`plugin_api.py` is
ignored). See
[`HERMES_BACKEND_PLUGIN_COMPATIBILITY.md`](HERMES_BACKEND_PLUGIN_COMPATIBILITY.md)
for the boundary and [`STATIC_USER_PLUGIN_MODE.md`](STATIC_USER_PLUGIN_MODE.md)
for the shipped frontend fallback.

`TODO.md` item 5 required choosing one integration path before further feature
work:

1. Bundled Hermes PR (add Olympus under `hermes-agent/plugins/olympus/dashboard`).
2. Frontend-only user plugin against existing Hermes dashboard APIs.
3. Upstream trusted backend-plugin RFC.

## Decision

Olympus adopts **option 2: frontend-only user plugin** as the supported
integration path.

- The static user-plugin mode (shipped in `dashboard/dist/index.js`) is the
  primary experience, not a temporary degradation.
- New frontend work consumes only first-party Hermes dashboard APIs through
  `window.__HERMES_PLUGIN_SDK__.fetchJSON`. Auth/session handling stays owned
  by Hermes.
- `dashboard/plugin_api.py` remains in the repo as a reference/prototype for a
  possible future bundled or trusted-backend mode, but it is not a supported
  install path and receives no new feature work.
- Olympus features that cannot be built safely from existing Hermes dashboard
  APIs (readiness scoring, Trace Spine, Kanban synthesis, skill hygiene
  synthesis, config policy recommendations, operational evals, Metrics Spine)
  stay hidden or explicitly labelled as unavailable rather than being rebuilt
  in browser JavaScript.

## Why

- **Zero upstream dependency.** User dashboard plugins work on hardened Hermes
  today. No Hermes PR, RFC, or release cycle gates Olympus installs.
- **Security posture is preserved.** The frontend stays inside Hermes'
  session-token flow and never inspects config/env/transcript content directly.
  The privacy rules in `STATIC_USER_PLUGIN_MODE.md` remain the ceiling for
  what the frontend may read.
- **Bundling and the trusted-backend RFC remain open.** The decision does not
  block upstream work; if Hermes later gains a trusted backend-plugin model,
  the existing `plugin_api.py` collectors can be revived.

## Consequences

- Bundled/trusted backend documentation moves from "recommended path" to
  "future option." `HERMES_BACKEND_PLUGIN_COMPATIBILITY.md` reflects this.
- Panels in static mode must degrade honestly: hidden when there is no
  evidence, labelled when partially derivable. No mock or placeholder data.
- `npm run test:compat` remains the diagnostic for whether a given Hermes
  install mounted the backend; `backendMounted: false` is the expected result
  in supported installs.
- The visual fixture must keep covering the static user-plugin mode scenario:
  `npx playwright test tests/visual/olympus.spec.js -g "static user-plugin mode"`.

## Follow-ups

- Expand static-mode coverage where first-party Hermes APIs expose safe,
  count-level evidence (no labels, no content, no config values).
- Keep `plugin_api.py` compile-checked in `npm run verify` but out of the
  live-backend gates for user-plugin installs.
- If upstream Hermes publishes a trusted backend-plugin interface, re-evaluate
  reviving the backend path.
