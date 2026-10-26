# Project Architecture — Summary

Static landing page for the Chat to Markdown extension. No application runtime:
a file tree is baked into an nginx image by CI, a bot commits the image tag into a
Helm chart, and ArgoCD reconciles it. No backend, no database, no server-side code
that executes user input.

**Components** (→ `PROJ-LAYOUT.md` for directories):

| Component | Purpose |
|---|---|
| `web/` | Landing page, privacy policy, robots/llms/sitemap, icons |
| `web/nginx.conf` | Routing, 7-day asset cache, security headers, `/nginx-health` |
| `Dockerfile` | `nginx:alpine`, stage `production`, copies `web/` to docroot |
| `helm/chatup` | Chart values (domain, CI-bumped tag, TLS) + vendored `static-site` subchart |
| `.github/workflows/ci.yml` | PR build+smoke; `main` build, smoke, push `sha-7`, bump chart |

**Flow:** PR → build+smoke → merge → push `main` → `sha-<7>` image on
`ops.noizu.com` → bot commits tag → ArgoCD syncs → nginx pod.

**Topology:** 1 replica, ClusterIP, Ingress class nginx with Cloudflare-only
source-range whitelist; TLS Secret `chatup-therobot-codes-tls` materialised by an
InfisicalSecret from `/apps/tls/chatuptherobotcodes`; private registry pull via
`ops-registry-secret`.

**Key decisions:** no JS framework; vendored `static-site` subchart (a
`file://../../../../components` dependency escaped the repo and broke ArgoCD);
Cloudflare-only origin; CI owns the image tag (never hand-edit, never `:latest`).
