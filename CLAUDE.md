# [Store Name] — Shopify Theme

Client theme repository for **[Store Name]** ([store-domain].com, `[store-handle].myshopify.com`). The theme is **TODO(discovery): vendor + theme name + version** (cite `config/settings_schema.json` `theme_info` and any `themeName`/`themeVersion` in `layout/theme.liquid`), customized by The Pages Media.

> **New theme?** If any `TODO(discovery)` marker remains in this file or in `/docs`, the first-pull discovery has not been finished. Run [docs/theme-onboarding.md](docs/theme-onboarding.md) before writing feature code. Our code is meant to sit on top of what the theme already provides, so we have to know what it provides first.

**What this theme actually is** (verified, not assumed; fill in during discovery, citing `file:line`):

- **Build tooling:** TODO(discovery): is there a `package.json`, a `src/` or `source/` folder, or compiled `*.bundle.*` / minified assets with no source in the repo? Name which assets are vendor or compiled and **frozen**, and where our code goes instead (Liquid files, section-scoped `{% style %}` / `{% javascript %}`, a hand-editable `custom.css`/`custom.js`, new `assets/tpm-*` files).
- **Frontend runtime:** TODO(discovery): native Web Components? jQuery? Alpine? A pub/sub bus? A section registry (`theme.sections.register`, `data-section-type`)? The global namespace (`window.theme`, `window.themeVariables`, …). Details go in `docs/javascript-patterns.md`.
- **Styling system:** TODO(discovery): where settings become CSS custom properties (`snippets/css-variables.liquid` or similar), token format (plain hex `var(--x)` vs RGB triplets `rgb(var(--x))`), class naming (BEM, utilities), canonical breakpoints. Details go in `docs/css-patterns.md`.
- **Deploy path:** TODO(discovery): confirm `main` is connected to the live theme through the Shopify GitHub integration (the default in [docs/development-workflow.md](docs/development-workflow.md)). If it isn't, record how the live theme is deployed and update the workflow doc to match.

## Core Directives

These apply to every task in this repo:

1. **Web standards first, in this order: HTML → Liquid → CSS → JavaScript.** Start with semantic, accessible markup. Add Liquid for data and server-side logic. Style with CSS. Reach for JavaScript last, and only for genuinely dynamic behavior.
2. **The Shopify way first.** Render server-side with Liquid wherever possible. Use native theme features (settings, blocks, section groups, metafields/metaobjects, the Section Rendering API) before writing custom code. Consider JavaScript enhancements only after the server-rendered version works. Fetching a whole storefront page and rebuilding its DOM in JS is the opposite of this rule.
3. **Leverage what exists and write the least code possible.** The goal is the smallest footprint in someone else's theme. Before adding anything, look for an existing snippet, token, utility class, custom element, theme helper, or custom event that already does the job, and use it. The pattern docs list what this theme provides; if they don't answer the question, search the theme before writing new code.
4. **Never write JavaScript for something CSS can do.** Hover states, transitions, accordions (`<details>`), sticky positioning, show/hide, animations: CSS first, always. Never write CSS for something HTML can do.
5. **New JS is modern native JavaScript**: `const`/`let` (never `var`), `fetch`, `async/await`, optional chaining, and **web components** for interactive UI. Use a plain `HTMLElement`, bind listeners in `connectedCallback`, clean them up with an `AbortController` in `disconnectedCallback`, and guard `customElements.define` with `customElements.get`. No new `window.*` globals. **Native JS even when the theme ships a library.** A theme or earlier agency build that loads jQuery, Alpine, Swiper, or a compiled framework layer is not permission to build on it. New code never adds a library and never extends one that's already there, so the theme doesn't pile more dependencies on top of the old ones. Existing library code stays as-is until it's deliberately replaced. Don't bind with `DOMContentLoaded` + `querySelectorAll`; it breaks on nodes inserted by AJAX or the theme editor.
6. **All assets are self-hosted. Never load functional assets from a third-party CDN.** JS, CSS, fonts, and images live in `/assets` and load via `asset_url`. A jsDelivr/unpkg/Google Fonts outage must never be able to break the storefront. The only exemptions are services that can't be self-hosted (analytics, Klaviyo, app SDKs), and those load deferred, never render-blocking. Shopify's own CDN is our infrastructure and doesn't count.
7. **Performance and accessibility are acceptance criteria, not nice-to-haves.** Every change should hold or improve both:
   - **Accessibility:** semantic elements, ARIA only where HTML falls short, full keyboard support, visible `:focus-visible` states, announced state changes (`aria-live`), touch targets at least 44×44px, text alternatives for images, and `| t` strings for UI copy.
   - **Performance:** no render-blocking additions, `defer` on every script, `loading="lazy"` plus `width`/`height` on below-the-fold images (CLS), `IntersectionObserver` before fetching below-the-fold content, animate `transform`/`opacity` only, and a `prefers-reduced-motion` guard on any motion. Assets load only on the templates that use them.
   - Most vendor themes have a thin baseline on focus states and reduced motion. New code must beat it, not match it.
8. **Mark all custom code with `MARK:-`.** Every piece of code we add or modify gets a comment starting with `MARK:-` so custom work is greppable later:
   - Liquid, single line: `{% # MARK:- Description of the customization %}`
   - Liquid, multi-line: `{% comment %} MARK:- Description … {% endcomment %}` (Liquid's own convention for block comments; `MARK:-` stays first so the grep still finds it)
   - Inside a `{% liquid %}` block: `# MARK:- ...` as a line
   - CSS: `/* MARK:- Description of the customization */`
   - JavaScript: `// MARK:- Description of the customization`
   - JSON templates don't support comments, so note customizations in the related section/snippet instead.

   To find all custom work: `grep -rn "MARK:-" .`

   Customizations made before this repo adopted the convention are **not** marked. When you find pre-existing custom code while debugging, add a `MARK:-` comment to it.
9. **Prefix new custom work with `tpm-`** (The Pages Media): `sections/tpm-<feature>.liquid`, `snippets/tpm-<name>.liquid`, `assets/tpm-<feature>.{css,js}`, custom elements `<tpm-thing>`, BEM blocks `.tpm-thing__part`, custom events `tpm:<domain>:<action>`, CSS custom properties `--tpm-*`. Any prefix says "custom"; ours says whose. Leave existing customs under other names alone in unrelated PRs.
10. **Don't hand-edit vendor, compiled, or app-owned files** beyond a surgical, `MARK:-` marked change. Override from outside (`custom.css`, `custom.js`, section-scoped code). The frozen-file list for this theme lives in `AGENTS.md` rule 11 and the pattern docs.

## File Order Within a Liquid File

1. Liquid variable assignments and logic (in a `{% liquid %}` block at the top)
2. CSS (`{% style %}`, or `{% stylesheet %}` for small section-local rules)
3. HTML markup with Liquid templating
4. JavaScript (`{% javascript %}` or a `defer` `<script src>` at the bottom)
5. `{% schema %}` last (sections)

**Liquid variable rule (mandatory):** always initialize with a default, then conditionally reassign. Never `assign x = a and b != blank`; compound boolean assignment breaks the page. (Compound conditions inside `if`/`unless` are fine.)

```liquid
{% liquid
  # MARK:- Show the CTA only when both text and link are set
  assign show_cta = false
  if section.settings.cta_text != blank and section.settings.cta_link != blank
    assign show_cta = true
  endif
%}
```

## Documentation

Theme-specific conventions live in `/docs`. Read the relevant file before working in that layer:

- [docs/theme-onboarding.md](docs/theme-onboarding.md): first-pull discovery runbook (run once per new theme)
- [docs/liquid-patterns.md](docs/liquid-patterns.md): section anatomy, naming taxonomy, snippets, schemas, images, metafields, layout map
- [docs/css-patterns.md](docs/css-patterns.md): where CSS lives, tokens, breakpoints, class naming, section scoping, focus and motion
- [docs/javascript-patterns.md](docs/javascript-patterns.md): asset inventory and edit policy, the theme's globals and events, web components, cart/fetch patterns
- [docs/development-workflow.md](docs/development-workflow.md): branching, PRs, releases, the GitHub integration, contractor handoff, and the rules **all** contributors (human and AI agents) must follow
- [docs/tech-debt.md](docs/tech-debt.md): verified catalog of inherited debt. Check it before "fixing" something odd, and never fold cleanup into unrelated PRs
- [docs/reviews/](docs/reviews/): review records, one file per round
- `docs/<feature>-setup.md`: merchant-facing setup guides (see [docs/_feature-setup-template.md](docs/_feature-setup-template.md))

**Keep the docs true.** After a task, check whether it invalidated or extended what `/docs` says. When you discover a theme convention, event, token, or gotcha while debugging, add it to the right doc with a `file:line` citation in the same PR. If you can't, say so in the PR description ("Docs impact: …"). Silence is the only wrong option.

## Brand Reference

TODO(discovery): values from `config/settings_data.json` (`current`). Always consume them through the theme's tokens, never as new hex:

- **Primary / buttons:** `#______` → `var(--______)`; button radius → `var(--______)`
- **Accent / sale / savings:** …
- **Body text / backgrounds / borders:** …
- **Typography:** font families and weights actually loaded (never ask for a weight that isn't loaded; the browser will synthesize it), base size token, heading scale token
- **Behavior settings that shape custom code:** cart type (drawer/page/notification) and the event that opens/refreshes it, quick shop/quick add, animation toggles exposed as body attributes
- **Canonical breakpoints:** the theme's own media queries, by frequency. Never introduce foreign ones.

## Workflow

- Develop with Shopify CLI: `shopify theme dev -e dev` (uses `shopify.theme.toml`)
- **All work happens on `feature/*` branches off `development`, delivered by pull request into `development`. Never commit to `main` or `development` directly.** `main` deploys to the live theme on merge and creates a tagged release from the README changelog. See [docs/development-workflow.md](docs/development-workflow.md).
- Every PR adds an `### Unreleased` changelog entry to `README.md`; the release PR assigns the version and bumps `theme_version` in `config/settings_schema.json`.
- Test on mobile and desktop viewports; check the browser console for errors before calling anything done
- Run `shopify theme check` before opening a PR. New and changed files must be clean; the inherited baseline is catalogued in `.theme-check.yml`
- Never run `shopify theme push` against any theme; the GitHub integration is the only deploy path

---

Last Updated: TODO(discovery): YYYY-MM-DD
