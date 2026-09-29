# Tech Debt & Cleanup Catalog

A verified inventory of debt inherited with this theme. **Nothing here is urgent.** This doc exists so cleanup happens deliberately, each item or group in its own scoped `feature/*` branch and PR tested on the dev theme, instead of silently or as a side effect of other work. Re-verify counts before deleting anything.

Priority key: **P1** = policy violation or user-facing risk, fix soonest · **P2** = wasted weight or requests, fix opportunistically · **P3** = tidiness, batch when convenient.

TODO(discovery): populate from the discovery passes in `docs/theme-onboarding.md`. Typical first-pull findings: third-party CDN scripts or styles (P1), render-blocking app scripts (P1), broken references or console errors (P1), dead assets with zero references (P2), `{% include %}` / `| img_url` usage (P3), hand-edited vendor files (record the edits so a theme upgrade can re-apply them).

## P1 — Policy violations and user-facing risk

| Item | Where | Verified | Fix |
|---|---|---|---|
| | | | |

## P2 — Wasted weight and requests

| Item | Where | Verified | Fix |
|---|---|---|---|
| | | | |

## P3 — Tidiness

| Item | Where | Verified | Fix |
|---|---|---|---|
| | | | |

## Exempt from the self-hosting rule

Services that can't be self-hosted and why (analytics, Klaviyo, app SDKs), and how each loads (deferred/idle). Shopify's own CDN is our infrastructure, not a third party.

---

When an item is done, note it in the PR and the README changelog, and delete its row here. When new debt turns up, add it with `file:line` evidence and a verification date.
