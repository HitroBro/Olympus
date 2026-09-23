# Hermes Backend Plugin Compatibility

Olympus uses a static dashboard tab plus an optional Python FastAPI backend in
`dashboard/plugin_api.py`.

Current Hermes versions intentionally **do not auto-import Python backend routes
from user-installed or project dashboard plugins**. This is a security boundary:
non-bundled dashboard plugins may provide static UI assets, but their Python
`api` files are ignored.

The Hermes guard lives in `hermes_cli/web_server.py`:

```python
_NON_BUNDLED_PLUGIN_SOURCES = frozenset({"user", "project"})
```

When a plugin is installed under:

```text
$HERMES_HOME/plugins/olympus/dashboard/
```

Hermes treats it as a user dashboard plugin. The `/olympus` tab can load, but
backend routes such as these will return `404`:

```text
/api/plugins/olympus/health
/api/plugins/olympus/overview
/api/plugins/olympus/tuning
```

This is expected on hardened Hermes builds unless Olympus is bundled into Hermes
or Hermes adds an explicit trusted backend-plugin model.

## Supported Modes

| Mode | Static tab | `/api/plugins/olympus/*` backend | Status |
| --- | --- | --- | --- |
| User dashboard plugin under `$HERMES_HOME/plugins` | Yes | No | Supported path (frontend-only) |
| Bundled Hermes plugin under `hermes-agent/plugins/olympus/dashboard` | Yes | Yes | Future option; not in scope |
| Future trusted backend-plugin API | Yes | Yes | Requires upstream Hermes design/security approval |
| Separate local service + static plugin | Yes | External only | Possible, but adds auth/daemon burden |

## Recommended Architecture

Olympus is a **frontend-only user dashboard plugin**. The static user-plugin
mode is the supported experience; new frontend work consumes only first-party
Hermes dashboard APIs through the plugin SDK's `fetchJSON`. See
[`BACKEND_COMPATIBILITY_DECISION.md`](BACKEND_COMPATIBILITY_DECISION.md).

Bundling Olympus into Hermes or upstreaming a trusted backend-plugin model
remains a possible future option. `dashboard/plugin_api.py` stays in the repo
as a reference/prototype for that path but is not supported for user-plugin
installs on hardened Hermes.

## Smoke-Test Interpretation

If `/olympus` loads but live security/performance smoke tests fail with `404` on
`/api/plugins/olympus/*`, diagnose backend mounting before debugging frontend
code.

Run:

```bash
npm run test:compat
```

Expected result on current user-plugin installs:

```text
backendMounted: false
likelyReason: current Hermes refuses non-bundled dashboard plugin Python backends
```

That means the compatibility boundary is working as designed. It does **not**
mean `dashboard/plugin_api.py` failed to compile.

## Product Boundary

Olympus should keep linking to Hermes-owned pages for actual mutations:
profiles, skills, cron, gateways, MCP, memory, config, and keys. Backend support
is for read-only synthesis, scoring, privacy-redacted evidence, and tuning
recommendations.
