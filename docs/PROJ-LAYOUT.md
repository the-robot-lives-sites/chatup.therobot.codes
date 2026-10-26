# Project Layout

`chatup.therobot.codes` is a **small, single-purpose repository**: a hand-authored
static landing page for the *Chat to Markdown* browser extension, plus the Docker
image definition and the Helm chart that deploys it. There is no application
source code, no database, and no runtime service beyond nginx serving files.

The whole tracked surface is 28 files; the tree below is complete (nothing is
collapsed or extracted into `layout/` — the repo does not need it).

```
chatup.therobot.codes/
├── .github/
│   └── workflows/
│       └── ci.yml              # PR build+smoke; main build-push sha-* image; chart tag bump
├── .dockerignore               # excludes helm/, docs, .git, *.md, Makefile from the image
├── .gitignore                  # .DS_Store only
├── CHANGELOG.md                # dated, PR-cited work log
├── Dockerfile                  # nginx:alpine, final stage named `production`
├── Makefile                    # `sandbox` / `sandbox-stop` local docker targets (port 8787)
├── README.md                   # entry point: what/why, sandbox, CI-driven deploy, TLS provisioning
├── web/                        # everything baked into the image → /usr/share/nginx/html
│   ├── index.html              # landing page (inline CSS, JSON-LD, gtag)
│   ├── privacy.html            # hosted privacy policy (Chrome Web Store listing URL)
│   ├── nginx.conf              # server block → /etc/nginx/conf.d/default.conf
│   ├── llms.txt                # machine-readable product summary for AI crawlers
│   ├── robots.txt              # allow-all + sitemap pointer
│   ├── sitemap.xml             # two URLs: / and /privacy
│   ├── apple-touch-icon.png    # 180px touch icon
│   ├── favicon.png
│   └── icons/                  # icon16.png, icon48.png, icon128.png (extension icon reuses)
├── helm/
│   └── chatup/
│       ├── Chart.yaml          # chart metadata; explains the vendored-subchart decision
│       ├── values.yaml         # domain, image tag (CI-bumped), TLS/Infisical wiring
│       └── charts/
│           └── static-site/    # VENDORED reusable nginx static-site subchart
│               ├── Chart.yaml
│               ├── values.yaml # replicas, resources, ingress class/annotations, TLS defaults
│               ├── .helmignore
│               └── templates/
│                   ├── _helpers.tpl        # names/labels + Cloudflare-only IP whitelist
│                   ├── deployment.yaml     # 1 replica, probes on `/`
│                   ├── service.yaml        # ClusterIP
│                   ├── ingress.yaml        # host rule, TLS, source-range whitelist
│                   └── tls-secret.yaml     # InfisicalSecret → kubernetes.io/tls Secret
├── docs/                       # this documentation set
│   ├── PROJ-LAYOUT.md          # this file
│   ├── PROJ-LAYOUT.summary.md
│   ├── PROJ-ARCH.md
│   ├── PROJ-ARCH.summary.md
│   ├── PROJ-SCHEMA.md
│   ├── PROJ-SCHEMA.summary.md
│   ├── THREAT-MODEL.md
│   └── THREAT-MODEL.summary.md
└── .meta/
    └── pointers.yaml           # doc-pointer store (file-kind anchors on the load-bearing files)
```

## Key Files Requiring Setup or Care

| File | Action |
|------|--------|
| `helm/chatup/values.yaml` `static-site.image` | **Do not edit by hand.** CI's `bump-chart` job rewrites the `sha-*` tag on every `main` push. |
| TLS origin certs | One-time, in the **monorepo** (`terraform/cloudflare/origin-certs-chatup`), not here. Writes `.secrets/tls/chatuptherobotcodes/{cert,key}.pem`, then `infisical-populate-secrets --include apps-tls-chatuptherobotcodes`. |
| `helm/chatup/charts/static-site/` | Vendored copy, not fetched by `helm dependency update` — it is shared by other static sites and can drift; see `PROJ-ARCH.md`. |
| `.envrc` | **Absent** — the repo has no direnv/env file and needs no secrets at build time. |

## Notes

- `web/nginx.conf` is a `.conf` file, so it cannot carry a doc-pointer marker
  (`.conf` is not a scanner-registered extension); it is anchored here instead.
- `.claude/` is present locally (worktrees dir) but is not tracked and is excluded
  from all scans.
