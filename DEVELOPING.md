# Developing for IN-CORE

Work on an IN-CORE client or service, **or build your own client**. Everything talks to the same REST services.

Scientists writing notebooks or analyses: see the [Development Guides](https://tools.in-core.org/doc/incore/dev_guides.html) in the IN-CORE manual instead.

## Before you start

Discuss your idea/contribution with the community on [Slack](https://in-core.slack.com) (`#developer`) or [GitHub Discussions](https://github.com/orgs/IN-CORE/discussions).
Can't? Email incore-dev@lists.illinois.edu — we'll tell you the next steps, including access to the dev environment.

## Repositories

| Repo | What | Stack | Setup |
|------|------|-------|-------|
| [incore-services](https://github.com/IN-CORE/incore-services) | Data, DFR3, hazard, space, semantics, project services | Java 11+, Gradle, MongoDB | [CONTRIBUTING](https://github.com/IN-CORE/incore-services/blob/develop/CONTRIBUTING.md) |
| [incore-ui](https://github.com/IN-CORE/incore-ui) | Web tools (Data/Hazard/DFR3 viewers) | React/Redux, webpack, Node ≥ 20 | [CONTRIBUTING](https://github.com/IN-CORE/incore-ui/blob/develop/CONTRIBUTING.md) |
| [incore-studio](https://github.com/IN-CORE/incore-studio) | Projects, workflows, hazard creation | React + TypeScript, esbuild, Node ≥ 20 | [CONTRIBUTING](https://github.com/IN-CORE/incore-studio/blob/develop/CONTRIBUTING.md) |
| [incore-playbook](https://github.com/IN-CORE/incore-playbook) | Resilience playbook library | React/Redux, webpack | [CONTRIBUTING](https://github.com/IN-CORE/incore-playbook/blob/develop/CONTRIBUTING.md) |
| [Joplin](https://github.com/IN-CORE/Joplin-Playbook) · [Galveston](https://github.com/IN-CORE/Galveston-Playbook) · [SLC](https://github.com/IN-CORE/SLC-Playbook) Playbook | Testbed apps using incore-playbook | React | see incore-playbook |
| [pyincore](https://github.com/IN-CORE/pyincore) · [pyincore-viz](https://github.com/IN-CORE/pyincore-viz) | Python client + analyses / visualization | Python ≥ 3.11, conda | README |
| [incore-docs](https://github.com/IN-CORE/incore-docs) | IN-CORE manual | Jupyter Book | [README](https://github.com/IN-CORE/incore-docs/tree/main/manual_jb) |
| [incore-helm](https://github.com/IN-CORE/incore-helm) | Kubernetes deployment | Helm | README |

## Pick a backend

| Backend | URL | Auth | When |
|---------|-----|------|------|
| Dev (shared) | `https://dev.in-core.org` | Keycloak, approved account | Default for client work |
| Prod | `https://tools.in-core.org` | Keycloak | Scripts/backends only (see CORS) |
| Local | `http://localhost:8080` | `x-auth-*` headers, no Keycloak | Changing services, offline work. Setup, web-client login and seed data: [incore-services CONTRIBUTING](https://github.com/IN-CORE/incore-services/blob/develop/CONTRIBUTING.md) |

Web clients read the backend from `INCORE_REMOTE_HOSTNAME`. pyincore: `IncoreClient("<url>")`.

Dev data is shared and may be reset — don't rely on it persisting.

## CORS (browser clients only)

Scripts, notebooks, pyincore and curl are **not** affected.

| Backend | Policy |
|---------|--------|
| **Prod** | Accepts browser requests only from IN-CORE's own web apps; third-party origins (incl. `localhost`) are not added. Shipping a web client against prod? Call IN-CORE from your own backend/proxy (server-to-server has no CORS). |
| **Dev** | `http://localhost:3000` and `:5000` are allowed. Need another origin? See [Before you start](#before-you-start). |
| **Your own services** | Set `SERVICES_ALLOW_ORIGIN` (comma-separated), `SERVICES_ALLOW_METHODS`, `SERVICES_ALLOW_HEADERS` on every service. Helm: `cors.origin` / `cors.methods` / `cors.headers` (see `values-incore-dev.yaml` in incore-helm). |

## Build your own client

**Endpoints** (`<host>` = `https://dev.in-core.org` or `https://tools.in-core.org`)

| Service | Base URL | Examples |
|---------|----------|----------|
| Data | `<host>/data/api` | `/datasets`, `/datasets/{id}/blob` |
| Hazard | `<host>/hazard/api` | `/earthquakes`, `/tornadoes`, `/hurricanes`, `/hurricaneWindfields`, `/floods`, `/tsunamis` |
| DFR3 | `<host>/dfr3/api` | `/fragilities`, `/mappings`, `/repairs`, `/restorations` |
| Space | `<host>/space/api` | `/spaces` |
| Semantics | `<host>/semantics/api` | `/types` |
| Project | `<host>/project/api` | `/projects` |

Full reference (Swagger): https://tools.in-core.org/doc/api/

**Auth** — Keycloak realm `In-core`, client `react-auth`:

```bash
TOKEN=$(curl -s -X POST https://dev.in-core.org/auth/realms/In-core/protocol/openid-connect/token \
  -d grant_type=password -d client_id=react-auth \
  -d username="$INCORE_USER" -d password="$INCORE_PW" | jq -r .access_token)

curl -H "Authorization: Bearer $TOKEN" "https://dev.in-core.org/data/api/datasets?limit=5"
```

| Client type | How to authenticate |
|-------------|---------------------|
| Script / backend service | Password grant above, or use pyincore (`IncoreClient` + `DataService`, `HazardService`, …) as an SDK |
| Browser app (SPA) | OIDC auth-code + PKCE (e.g. `keycloak-js`, `oidc-client-ts`; see `incore-studio/src/app.config.js`). Redirect URI must be registered — see [Before you start](#before-you-start). |

**Steps**

1. Get dev access ([Before you start](#before-you-start)).
2. Prototype against dev with the token flow above.
3. GET works for public data; POST/DELETE create resources owned by your user.
4. Going to prod: browser apps go through your own backend/proxy (see CORS); register your redirect URI for login; switch `<host>` to `https://tools.in-core.org`.

## Pull requests

- Branch from `develop`, PR back to `develop`. Link the issue (`Fixes #123`).
- Update `CHANGELOG.md` (Keep a Changelog format).
- Lint and tests pass (see each repo's CONTRIBUTING).
