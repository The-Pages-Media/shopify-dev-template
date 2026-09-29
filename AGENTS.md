# Instructions for AI Agents

This file is for **any** AI coding agent operating in this repository (Claude Code, Cursor, Codex, OpenClaw, etc.). These rules are mandatory. Where the GitHub plan supports branch protection, the server enforces them. Where it doesn't, review enforces them, and following them is a condition of your work being merged.

## Before writing any code

Read, in this order:

1. `CLAUDE.md`: core directives, file order, theme facts, brand reference. **If it still contains `TODO(discovery)` markers, stop.** Run `docs/theme-onboarding.md` (or ask the person you work for whether to) before writing feature code.
2. The pattern doc for the layer you're touching:
   - `docs/liquid-patterns.md`
   - `docs/css-patterns.md`
   - `docs/javascript-patterns.md`
3. `docs/development-workflow.md`: the branching, PR, and release process you must follow
4. `docs/tech-debt.md`: so you don't "fix" catalogued debt as a side effect

## Non-negotiable rules

1. **Never commit or push to `main` or `development`.** `main` is the live theme (a merge is a deploy). All work happens on a `feature/short-description` branch created off `development`, delivered as a pull request into `development`. Reading from `main` is fine; writing to it is not.
2. **Never merge pull requests.** Humans review and merge.
3. **Never deploy.** Do not run `shopify theme push` or `shopify theme share`, and do not edit any theme through admin APIs. The GitHub integration deploys on merge; you never do.
4. **Never modify** `.github/` workflows, `CODEOWNERS`, rulesets, or this file.
5. **Build order: HTML → Liquid → CSS → JavaScript.** Server-render with Liquid first; use native theme features (settings, blocks, section groups, the Section Rendering API) before custom code; style with CSS; JavaScript only for genuinely dynamic behavior. Never use JS for something CSS can do.
6. **Native JavaScript only.** Never add a JS library, and never write new code against one the theme already loads (jQuery, Alpine, Swiper, a framework layer from an earlier agency build). Use web components and platform APIs.
7. **Reuse before you write.** Check the pattern docs for an existing snippet, token, utility class, custom element, helper, or event that does the job. The smallest diff that meets the brief wins.
8. **Mark every piece of custom code** with a `MARK:-` comment (`{% # MARK:- ... %}` for a single line and `{% comment %} MARK:- ... {% endcomment %}` for multi-line Liquid, `# MARK:- ...` inside `{% liquid %}`, `/* MARK:- ... */` in CSS, `// MARK:- ...` in JS).
9. **Prefix every new custom file, element, class, and event with `tpm-`** (`sections/tpm-<feature>.liquid`, `assets/tpm-<feature>.js`, `<tpm-thing>`, `.tpm-thing__part`, `tpm:<domain>:<action>`). Do not rename existing customs in unrelated PRs.
10. **Accessibility and performance are acceptance criteria**, not nice-to-haves: semantic HTML, keyboard support, visible `:focus-visible` states, `prefers-reduced-motion` guards, width/height on images, targets at least 44×44px on touch, no render-blocking additions, and no assets loaded from a third-party CDN.
11. **Do not hand-edit vendor, compiled, or app-owned files** beyond a marked, surgical change. For this theme: TODO(discovery): list them (e.g. the vendor `theme.js`/`theme.css`, minified vendor bundles, `blocks/ai_gen_block_*`, app snippets like `klaviyo-*`). Override from outside (`custom.css`, `custom.js`, section-scoped code) whenever possible.
12. **Liquid variables:** initialize with a default, then conditionally reassign. Never `assign x = a and b`; compound boolean assignment breaks the page.
13. **Every PR adds an `### Unreleased` changelog entry** in `README.md` and describes how to test the change on the dev theme. Version numbers are assigned at release time, never on a feature branch.
14. **Never use a Shopify preview theme as a transport layer.** If a branch is connected to a development theme for QA, only make editor changes that belong to the feature. `shopify[bot]` commits carrying unrelated content will be rejected in review.
15. **Keep the docs true.** If your work adds or contradicts a convention in `/docs`, update the doc in the same PR with a `file:line` citation, or state "Docs impact: …" in the PR description.

If a task appears to require breaking any rule above, stop and raise it in the PR or with the person you are working for. Don't work around it.
