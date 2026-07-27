# Apps Script Hello World

[![CI](https://github.com/amalfi-group/apps-script-hello-world/actions/workflows/ci.yml/badge.svg)](https://github.com/amalfi-group/apps-script-hello-world/actions/workflows/ci.yml)

[日本語](README.ja.md)

A minimal Google Apps Script function built on the
[Apps Script Fleet](https://github.com/h13/apps-script-fleet) template
("1 repo = 1 function"). It returns a hello-world greeting — and doubles as
the fleet's canary: the first repo to verify the keyless CI/CD credential
flow end to end.

## How Deployment Works

| Action | Result |
| --- | --- |
| Open a PR | CI runs (lint + typecheck + tests, 80% coverage) |
| Push to `dev` | CD deploys to the **development** GAS project |
| Push to `main` | CD deploys to the **production** GAS project |

TypeScript in `src/` is bundled by Rollup into `dist/` and pushed with clasp.
Only top-level functions in `src/index.ts` are callable from Apps Script.

### GAS Projects

| Environment | Script |
| --- | --- |
| development | [script.google.com/d/1pqOIAwMX8TelAY47GfcPZluTLFW3ioGBTnXVffr_BVzsQoHJhDYMxpLq](https://script.google.com/d/1pqOIAwMX8TelAY47GfcPZluTLFW3ioGBTnXVffr_BVzsQoHJhDYMxpLq/edit) |
| production | [script.google.com/d/1HQJM7HR13mtZMTyvv9PTcnT1P8SmqemInmSwv7LI4xwZgUP5sZ3OIVac](https://script.google.com/d/1HQJM7HR13mtZMTyvv9PTcnT1P8SmqemInmSwv7LI4xwZgUP5sZ3OIVac/edit) |

## Credentials (keyless CI)

CI holds **no long-lived clasp secrets**. At deploy time the workflow
authenticates to Google Cloud via **Workload Identity Federation (OIDC)** and
fetches the fleet's shared clasp credentials from **Secret Manager** in the
central GCP project. Rotation and audit happen centrally — nothing to update
in this repo. Details:
[secret-manager.md](https://github.com/h13/apps-script-fleet/blob/main/docs/secret-manager.md).

Per-repo configuration lives in the GitHub environments `development` /
`production` (`CLASP_JSON` secret + `DEPLOYMENT_ID` variable), set up once by
`./scripts/init.sh`.

## Development

```bash
pnpm install
pnpm run check      # lint + typecheck + test
pnpm run build      # bundle to dist/
```

- `src/greeting.ts` — the hello-world logic
- `src/index.ts` — GAS entry points (`doGet`, `getMessage`)
- `test/` — Jest tests

Tooling (CI/CD, lint, build config) is managed by the upstream template and
kept up to date by weekly [template sync](.github/workflows/sync-template.yml)
PRs — see `.templatesyncignore` for what is synced.
