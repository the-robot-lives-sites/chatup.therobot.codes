# chatup.therobot.codes
<!-- 〚📁:39119938-5809-5276-9553-09973a7a3d44〛 README.md :: repo entry: what it is, sandbox, CI-driven deploy and TLS provisioning. -->

**Repo:** https://github.com/the-robot-lives/chatup.therobot.codes

Landing page for **Chat to Markdown** — a local-only browser extension that exports ChatGPT, Claude, Gemini, Le Chat, Open WebUI and more as clean Markdown or API-format YAML. Plugin: [the-robot-lives/chat-to-markdown](https://github.com/the-robot-lives/chat-to-markdown). Part of [The Robot Lives](https://therobotlives.com).

## What

A hand-authored static marketing/info site (no JS framework) served by an nginx Alpine image, with a privacy page, sitemap, robots.txt, and llms.txt.

## Why

Gives the Chat to Markdown extension a public home page under the therobot.codes domain without pulling a framework into a one-pager.

## Getting Started

Prerequisites: Docker.

```bash
make sandbox        # serve locally at http://127.0.0.1:8787
make sandbox-stop
```

**Deploy:** CI-driven. PRs target `develop`; `main` is CI/CD-only.

1. One-time origin TLS provisioning: local-state terraform in the monorepo at `terraform/cloudflare/origin-certs-chatup` (needs a TRL Cloudflare token with "SSL and Certificates: Edit"); writes `.secrets/tls/chatuptherobotcodes/{cert,key}.pem`, then `infisical-populate-secrets --include apps-tls-chatuptherobotcodes`. Zone SSL mode is Full (Strict); a 526 means the origin ingress cert doesn't include `chatup.therobot.codes`.
2. GitHub Actions on `main`: builds + smokes the nginx image (`/nginx-health`, fallback `GET /`), pushes `sha-<7>` to `ops.noizu.com` (never `:latest`), a bot commits the helm tag bump, ArgoCD auto-syncs.

Never bump chart tags, merge, or push `main` by hand.

## How It Works

- `web/` — static HTML + assets + nginx.conf, baked into the image
- `helm/chatup/` — Helm chart wrapping the vendored `static-site` subchart
- `Dockerfile` — nginx Alpine, tagged `ops.noizu.com/chatup.therobot.codes/web:sha-*`

## License

MIT © TheRobotLives
