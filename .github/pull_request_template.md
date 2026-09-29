## Task

<!-- Issue / Basecamp task / brief this PR delivers. `Closes #123` when there is an issue. -->

## What changed and why

<!-- Plain-language summary a merchant could follow, then the technical detail. -->

## How to test on the dev theme

<!-- Steps a reviewer follows: URLs, sections, settings to toggle, viewports. -->

## Screenshots

<!-- Desktop and mobile, before/after where it helps. -->

## Commands run

<!-- e.g. `shopify theme check --fail-level error` → exit 0 -->

## Known gaps / docs impact

<!-- Anything unfinished, any design ambiguity, and which /docs files changed (or why none needed to). -->

## Checklist

- [ ] Branch is `feature/*` off `development` and this PR targets `development` (hotfixes: `hotfix/*` off `main`, targeting `main`)
- [ ] `### Unreleased` changelog entry added to `README.md` (no version number on feature branches)
- [ ] Reused existing snippets, tokens, classes, helpers, and events where they fit; the diff is the smallest one that meets the brief
- [ ] All new/modified custom code carries a `MARK:-` comment; new files, elements, classes, and events use the `tpm-` prefix
- [ ] Follows `docs/liquid-patterns.md`, `docs/css-patterns.md`, `docs/javascript-patterns.md`: HTML → Liquid → CSS → JS, theme tokens (no new hex), canonical breakpoints only
- [ ] Native JS only: no new libraries, and no new code built on libraries the theme already loads
- [ ] No assets from third-party CDNs; no edits to vendor, compiled, or app-owned files beyond a marked, surgical change
- [ ] Accessibility held or improved: keyboard support, `:focus-visible`, `prefers-reduced-motion`, 44×44px targets, alt text, announced state changes
- [ ] Performance held or improved: nothing render-blocking, `defer` on scripts, lazy images with width/height, assets load only where used
- [ ] Tested on mobile and desktop; browser console clean; Liquid renders on the dev theme
- [ ] Merchant setup guide added or updated in `docs/<feature>-setup.md` if the feature needs admin configuration
- [ ] No unrelated theme-editor/content changes in the branch history (`shopify[bot]` commits called out if intentional)

## Author

- [ ] Human
- [ ] AI agent (which: <!-- Claude Code / Cursor / Codex / other -->): I confirm this PR was created per `AGENTS.md` (feature branch only, no merges, no deploys)
