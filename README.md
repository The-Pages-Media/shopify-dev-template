# [Store Name]

[store-handle].myshopify.com ([store-domain].com)

TODO(discovery): vendor theme + version base, customized by The Pages Media. Start with [CLAUDE.md](CLAUDE.md) for conventions and the [/docs](docs/) pattern files before writing code. AI agents and contractors: read [AGENTS.md](AGENTS.md) first.

> **Starting a new theme from this template?** Follow [docs/theme-onboarding.md](docs/theme-onboarding.md): pull the live theme, connect GitHub, record the Theme Check baseline, and run the first-pull discovery that fills in every `TODO(discovery)` marker. Delete this note once onboarding is merged.

## Preview URL

`https://[store-domain].com/?preview_theme_id=[THEME_ID]`

## Requirements

It is recommended for this project to use Shopify CLI

- [Shopify CLI Reference](https://shopify.dev/themes/tools/cli)
- [Commands](https://shopify.dev/docs/themes/tools/cli/commands)
- [Upgrade](https://shopify.dev/docs/themes/tools/cli/commands#upgrade)
- [Environments](https://shopify.dev/docs/themes/tools/cli/environments)

## Get Started

Set the store handle in `shopify.theme.toml` (no `.myshopify.com` needed), then run with the environment:

`shopify theme dev -e dev`

If working out of the customizer and needing to save updated sections, pass flags before the environment to avoid clashes:

`shopify theme dev --theme-editor-sync -e dev`

## Workflow

All work goes through `feature/*` branches off `development` and pull requests into `development`. `main` is the live theme: it's connected to production through the Shopify GitHub integration, so a merge into `main` is a deploy and automatically creates a tagged GitHub Release from the changelog below. No direct pushes to `main` or `development`, no exceptions (humans or AI agents). Full process: [docs/development-workflow.md](docs/development-workflow.md).

## Documentation

- [CLAUDE.md](CLAUDE.md): core directives, file order, theme facts, brand reference
- [AGENTS.md](AGENTS.md): mandatory rules for AI coding agents and contractors
- [docs/theme-onboarding.md](docs/theme-onboarding.md): first-pull discovery runbook
- [docs/liquid-patterns.md](docs/liquid-patterns.md): section anatomy, naming, snippets, schemas, images, metafields
- [docs/css-patterns.md](docs/css-patterns.md): where CSS lives, tokens, breakpoints, naming, scoping, focus and motion
- [docs/javascript-patterns.md](docs/javascript-patterns.md): asset edit policy, theme globals and events, web components, cart patterns
- [docs/development-workflow.md](docs/development-workflow.md): branching, PRs, releases, documentation pattern, contractor handoff, agent rules
- [docs/tech-debt.md](docs/tech-debt.md): verified inherited debt
- [docs/reviews/](docs/reviews/): review records, one file per review round
- [docs/specs/](docs/specs/): feature briefs and specs
- Merchant setup guides: `docs/<feature>-setup.md` (list each one here as it's added)

## Changelog

Original base of theme is TODO(discovery): vendor theme + version. Feature branches add notes under `### Unreleased`; the release PR into `main` renames that heading to the version and bumps `theme_version` in `config/settings_schema.json` to match.

### v1.0.0 - YYYY-MM-DD

- Initial import of the live theme, with house conventions and CI
