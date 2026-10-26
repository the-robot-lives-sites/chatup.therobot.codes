# Project Layout — Summary

Static landing page for the *Chat to Markdown* extension: nginx Alpine image
built from `web/`, deployed by a vendored Helm subchart. No application code, no
database. 28 tracked files.

```
chatup.therobot.codes/
├── .github/workflows/ci.yml    # PR build+smoke; main → sha-* push + chart bump
├── Dockerfile                  # nginx:alpine, stage `production`
├── Makefile                    # sandbox at 127.0.0.1:8787
├── README.md                   # what/why, deploy + TLS provisioning notes
├── CHANGELOG.md                # dated work log
├── web/                        # baked into the image
│   ├── index.html              # landing page (inline CSS, JSON-LD, gtag)
│   ├── privacy.html            # privacy policy (store listing URL)
│   ├── nginx.conf              # routes, caching, headers, /nginx-health
│   ├── llms.txt robots.txt sitemap.xml
│   └── icons/ apple-touch-icon.png favicon.png
├── helm/chatup/                # chart: values (CI-bumped image tag, TLS) + vendored static-site subchart
├── docs/                       # LAYOUT / ARCH / SCHEMA / THREAT-MODEL (+ summaries)
└── .meta/pointers.yaml         # doc-pointer store
```

Setup-critical: `static-site.image` is CI-managed; origin TLS certs are
provisioned in the monorepo, not here. No `.envrc` exists.
