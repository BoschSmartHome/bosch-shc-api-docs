# AGENTS.md

## Project Focus
- This repository is a **static documentation site** for the local Bosch SHC API.
- Core artifacts are OpenAPI YAML files in `openapi/`, a Swagger UI page in `index.html`, and a Postman collection in `postman/`.

## Architecture and Data Flow
- `index.html` loads Swagger UI from CDN and renders the YAML selected in the `#select` dropdown.
- Each domain is maintained as a separate file: `openapi/*-local-openapi-v3.yml` (for example `openapi/MainResources-local-openapi-v3.yml`).
- API endpoints/ports are designed for local SHC communication:
  - REST: `https://{shcIp}:8444/smarthome` (see OpenAPI `servers`)
  - Client registration: `https://{{host}}:8443/smarthome/clients` (see Postman environment)
  - Public info: `https://{{host}}:8446/smarthome/public/information`
  - Long polling JSON-RPC: `https://{{host}}:8444/remote/json-rpc`

## Critical Workflows
- Local preview: a simple static server is enough; there is no `package.json` or build step.
- Deployment runs via GitHub Pages workflow `/.github/workflows/publish-openapi-docs.yml`.
- Auto-deploy triggers only on push to `master` with `openapi/**.yml` changes; changes to `index.html`/`css/` need manual `workflow_dispatch` or an additional OpenAPI file change.

## Repository-Specific Conventions
- API version is expected as header `api-version` (in OpenAPI as `apiVersionHeaderParam`, in Postman requests as a request header).
- Postman environment stores this value as `api_version` (currently `3.12` in `postman/Bosch Smart Home Controller.postman_environment.json`); keep values aligned during API updates.
- Client registration follows fixed fields and roles in the Postman collection:
  - `id`: `oss_{{client_id}}`
  - `name`: `OSS {{client_name}}`
  - `primaryRole`: `ROLE_RESTRICTED_CLIENT`
  - `Systempassword` header must be Base64-encoded.
- Long polling is the preferred integration path (see `best_practice/README.md` and Postman folder `Long Polling`):
  - `RE/subscribe` -> stores `shc_poll_id`
  - `RE/longPoll` with `shc_poll_id`
  - `RE/unsubscribe`
  - Subscription limits: max age 24h, inactivity timeout 5min.

## Change Patterns for Agents
- To add a new API domain:
  1. Create a new YAML file in `openapi/`.
  2. Add the matching dropdown option in `index.html` (otherwise it is not visible in Swagger UI).
  3. Optionally add matching requests in `postman/Bosch Smart Home Controller.postman_collection.json` and variables in the environment file.
- For device ID URLs, handle edge cases: encode `#` in child device IDs as `%23` (`CHANGELOG.md`).
- Keep terminology/compliance intact: non-commercial usage and OSS client naming constraints are defined in `README.md`.

## Relevant References
- `README.md` (product context + terms)
- `CHANGELOG.md` (API/domain release history and version-related migration hints)
- `best_practice/README.md` (polling, request rate, TLS host verification)
- `postman/README.md` (practical onboarding steps)
- `.github/workflows/publish-openapi-docs.yml` (deployment behavior)


