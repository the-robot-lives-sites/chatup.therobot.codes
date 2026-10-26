# Project Schema — Summary

**No persistence layer.** No database, migrations, ORM, KV store, or queue — the
container serves immutable files. No ERD applies.

| Kind | Where | Doc form |
|------|-------|----------|
| Config | `helm/chatup/values.yaml` + vendored `static-site/values.yaml` | values tables + layering graph |
| Content/data | `web/*.html`, `llms.txt`, `robots.txt`, `sitemap.xml`, icons, `CHANGELOG.md` | inventory table |
| Interface | `web/nginx.conf` routes/headers, `.github/workflows/ci.yml` triggers/jobs | route + job tables |

**Config:** subchart defaults ← parent overrides ← CI `sed` bump of
`static-site.image` (`sha-<7>`). Site values: `domain: chatup.therobot.codes`,
`fullnameOverride: chatup`, port 80, `ops-registry-secret`, TLS Secret
`chatup-therobot-codes-tls` from Infisical `/apps/tls/chatuptherobotcodes`
(`CHATUP_TLS_CRT`/`CHATUP_TLS_KEY`).

**HTTP surface:** `/nginx-health` → `200 ok`; asset regex → 7-day cache;
`/` → `try_files $uri $uri.html $uri/index.html =404` (so `/privacy` → `privacy.html`);
headers `nosniff`, `SAMEORIGIN`, `strict-origin-when-cross-origin`; gzip on. No CSP,
HSTS, or Permissions-Policy.

**CI contract:** `push` main/stage + all PRs; env `REGISTRY=ops.noizu.com`,
`IMAGE=.../chatup.therobot.codes/web`; jobs `docker-build` → `smoke` (PR),
`build-push` (main) → `bump-chart` (`sha-<7>`, `[skip ci]`). Secrets
`REGISTRY_USER`/`REGISTRY_PASSWORD`. A `stage` push runs no jobs.
