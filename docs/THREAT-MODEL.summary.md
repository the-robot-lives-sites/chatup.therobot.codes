# Threat Model — Summary

Static content site: no auth, no sessions, no user input handling, no data store.
Assets: integrity of the served bytes (public product face + Chrome Web Store
privacy-policy URL) and the deploy credential path (CI registry push, origin TLS key).

**Trust boundaries:** internet ↔ Cloudflare edge ↔ ingress ↔ nginx pod; CI ↔
registry ↔ chart repo ↔ ArgoCD; Infisical ↔ cluster Secret.

**Register:** 9 entries — 3 open, 2 partial, 4 accepted (T-004 tracked via T-001).

| ID | Sev | STRIDE | Finding | Status |
|----|-----|--------|---------|--------|
| T-001 | Medium | Repudiation / Info disc. | `gtag.js` (G-N67LR8EK2E) live on the page while the privacy page says "does not run analytics" | Open |
| T-002 | Medium | Tampering | No CSP + unsignatured third-party script | Open |
| T-003 | Medium | Tampering / Elevation | Mutable `sha-7` image tag, no digest pinning or signing | Partial |
| T-004 | Low | Info disclosure | Privacy page is the store-listing URL ⇒ accuracy is compliance surface | Accepted (as T-001) |
| T-005 | Low | DoS | 1 replica, small limits, no in-repo rate limit; relies on Cloudflare | Accepted |
| T-006 | Low | Tampering / Spoofing | No origin HSTS header | Partial |
| T-007 | Low | Tampering | Vendored `static-site` subchart can drift from shared original | Accepted |
| T-008 | Low | Info disclosure | `.gitignore` covers only `.DS_Store`; TLS key material generated in the monorepo | Open (preventive) |
| T-009 | Low | Spoofing | Bot commits unsigned, rest on branch protection | Accepted |

**In place:** Cloudflare-only ingress whitelist (full IPv4/IPv6 edge ranges), no
dynamic execution (`try_files` only), `nosniff` / `SAMEORIGIN` /
`strict-origin-when-cross-origin` headers, TLS + registry secrets out of repo
(Infisical operator, `ops-registry-secret`), `sha-<7>` tags never `:latest`,
image ships only `web/`.

**Residual:** availability and DDoS volume control delegated to Cloudflare; subchart
drift; unsigned commit provenance. Decision needed on T-001 and T-002.

**Out of scope:** the extension's own permissions/adapters (`the-robot-lives/chat-to-markdown`),
Cloudflare zone config, Ingress controller, ArgoCD, Infisical deployment.
