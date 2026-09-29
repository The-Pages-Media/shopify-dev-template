# Theme Onboarding — First-Pull Discovery

Run this once, when we take on a theme we haven't worked in. It turns the starter docs into a verified map of *this* theme, so every later change can reuse what the theme already has and stay small.

**Why it matters:** our goal in someone else's theme is the smallest footprint. That only works if we know which snippets, tokens, utility classes, custom elements, helpers, and events already exist. Discovery is where we find them. Skipping it leads to a second add-to-cart helper, a fourth breakpoint, and a new hex color that already exists as a token.

**Done means:** `grep -rn "TODO(discovery)[:]" CLAUDE.md AGENTS.md README.md docs .github .theme-check.yml` returns nothing, and every claim in the docs cites `file:line`.

---

## 1. Bootstrap the repo

1. Create the client repo from this template (keep `CLAUDE.md`, `AGENTS.md`, `docs/`, `.github/`, `.theme-check.yml`, `.shopifyignore`, `.gitignore`, `shopify.theme.toml`, `README.md`).
2. Set the store handle in `shopify.theme.toml` and replace the placeholders in `README.md` (store name, domain, preview URL).
3. Pull the live theme into the repo root: `shopify theme pull -e dev --live`. This is the one time we pull code from Shopify into Git. After this, code only moves through Git.
4. **Record the vendor theme name and version now**, in `CLAUDE.md`, from `config/settings_schema.json` (`theme_info`) and anything like `themeName`/`themeVersion` in `layout/theme.liquid`. Step 5 overwrites `theme_version` with our release number, so this is the last chance to read the vendor's from that file.
5. Set `theme_version` in `config/settings_schema.json` to `1.0.0` and make sure the README changelog's top entry is `### v1.0.0 - YYYY-MM-DD`. From here on, those two numbers always match (see `docs/development-workflow.md` step 8).
6. Commit to `main` as the initial import ("Initial import of <Theme> vX.Y.Z from the live theme") and push. Create `development` from `main` and push it. This bootstrap is the only direct commit to `main`; the "no direct commits" rule applies from the next commit on.

## 2. Connect Shopify and GitHub

1. In the Shopify admin, **Online Store → Themes → Add theme → Connect from GitHub**: connect `main`. Shopify creates a new theme from the branch and does not replace the live one.
2. Compare that theme with the live theme in preview. If the merchant edited content since the pull, pull those changes into `main` before continuing. At an agreed time, **publish the `main`-connected theme**. From then on, a merge into `main` is a deploy, and the merchant's theme-editor edits come back as `shopify[bot]` commits.
3. Connect `development` to a second theme named `[DEV-TR] Development theme` (or the store's naming) for QA.
4. If the store can't work this way (for example, the merchant won't let us publish a connected theme yet), write down the actual deploy path in `docs/development-workflow.md` and `CLAUDE.md` before any feature work.

## 3. Wire up GitHub

1. `.github/CODEOWNERS`: confirm the owner handle. Adjust the structural paths to this theme's token/variable snippets (found in step 5).
2. Create a `no-release` label (used by `changelog-check.yml` to opt a PR out).
3. On GitHub Team or higher, import `.github/rulesets/branch-protection.json` (**Settings → Rules → Rulesets → Import**) and enable "Do not allow bypassing." On GitHub Free, record in `docs/development-workflow.md` that enforcement is by convention and review.
4. Run the **Release** workflow once by hand (`workflow_dispatch`) to tag `v1.0.0` from the README changelog.

## 4. Theme Check baseline

```bash
shopify theme check --fail-level error
```

Most vendor themes carry inherited error-level offenses. Record each one in `.theme-check.yml` so CI is green on the inherited tree and fails only on **new** breakage:

- Prefer **file-scoped `ignore` lists per check**. That keeps the check live for every other file, so a new syntax error in a new section still fails.
- Demote a check to `severity: warning` only when the offenses are spread across the whole tree (typical: `MatchingTranslations` across locale files).
- Write the measurement date and counts in the file's header comment and in the "Theme Check baseline" section of `docs/liquid-patterns.md`.

When you're done, `shopify theme check --fail-level error` must exit 0.

## 5. Discovery passes

Do these on a `feature/theme-onboarding` branch off `development`. Each pattern doc opens with a **First-pull discovery** section that has the exact searches to run and the tables to fill. Work through them in this order, because later passes build on earlier ones:

1. **Liquid** — `docs/liquid-patterns.md`: layout map, section/snippet taxonomy, reusable snippets, image rendering path, metafield namespaces, app-owned files.
2. **CSS** — `docs/css-patterns.md`: where CSS lives, the token system and its format, canonical breakpoints, utility and component classes, the focus/motion baseline.
3. **JavaScript** — `docs/javascript-patterns.md`: asset inventory and edit policy, script loading order, the global namespace and helpers, **every custom element and custom event the theme defines or listens to**, cart and fetch patterns, theme-editor hooks.
4. **Brand reference** in `CLAUDE.md`, from `config/settings_data.json` `current`.
5. **Tech debt** in `docs/tech-debt.md`: log anything found during passes 1–3 (third-party CDN loads, dead assets, hand-edited vendor files, `{% include %}`, `img_url`, broken references). Log it with evidence; don't fix it.
6. **Prior customizations:** if `git log` has history, or the theme has a `custom.css`/`custom.js`, list what was customized and by whom. Add `MARK:-` comments to prior custom code only as you touch it later, not in a sweep.

AI agents can run passes 1–3 in parallel, since each only reads the theme and writes its own doc. Merge the results and resolve any contradictions between docs before opening the PR.

**Rules for discovery writing:**

- **Verify, don't assume.** Every claim cites `file:line` or a grep count. "The theme uses BEM" isn't enough. Write "BEM, e.g. `.product-card__title` (`assets/base.css:812`)."
- **Record what to reuse, not just what exists.** The most useful output is the list of existing events, helpers, tokens, and classes that new work should use instead of writing its own.
- **Name the "do not imitate" patterns.** Legacy or poor patterns in the theme (unguarded defines, `querySelectorAll` sweeps, hardcoded hex, foreign breakpoints) get listed so nobody copies them.
- **Discovery doesn't change theme code.** The onboarding PR only touches docs and config.

## 6. Ship it

1. Fill in `Last Updated` in `CLAUDE.md`.
2. Confirm the "done means" grep above returns nothing.
3. Add an `### Unreleased` changelog entry ("House conventions and first-pull discovery docs; no theme code changed"), open a PR into `development`, then release it as a patch (`v1.0.1`) following `docs/development-workflow.md`.

Keep this file after onboarding. It records how discovery was done and is the runbook if the theme gets a vendor upgrade (which calls for redoing passes 1–3 on the parts that changed).
