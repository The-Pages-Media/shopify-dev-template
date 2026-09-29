# Development Workflow & Branching Strategy

> This document governs all development work on the [Store Name] Shopify theme. It applies to **all contributors: agency developers, contractors, and any AI-assisted tools** (Claude Code, Cursor, Codex, or anything else operating under repo credentials). If a tool can push a commit, this document applies to it.

---

## Branch overview

| Branch | Purpose | Deployed via |
|--------|---------|--------------|
| `main` | **Production.** Connected to the live theme through the Shopify GitHub integration, so a merge into `main` is a deploy. Merchant edits in the live theme editor are written back to `main` as `shopify[bot]` commits. | Shopify GitHub integration (automatic on merge) |
| `development` | Integration and QA branch, connected to the `[DEV-TR] Development theme` through the same integration. All feature work is reviewed and QA'd here before it reaches production. | Shopify GitHub integration (automatic on merge) |
| `feature/*` | Individual units of work: bugs, features, content changes. Short-lived, one piece of work per branch, branched off `development`. | Not deployed. QA on a local `shopify theme dev` preview or a per-branch development theme (see [Preview themes](#preview-themes-and-the-github-integration)). |
| `hotfix/*` | Critical production fixes that can't wait for the `development` cycle. Branched off `main`. | PR into `main`, then merged back into `development`. |

> **`main` is the live theme.** The integration writes merchant content edits (copy, images, theme-editor settings) back to `main` automatically, so `main` is always current and there's no separate "pull first" step. The flip side: **`development` must absorb `main` before every release PR**, or the release merge will fight the merchant's latest edits.
>
> If this store deploys differently (see `docs/theme-onboarding.md` step 2), replace this section with the real deploy path before any feature work.

---

## The core rule

**No one pushes directly to `main` or `development`. All changes go through a feature branch and a pull request**, however small. That applies to agency staff, contractors, and every AI agent, with no exceptions.

Reading from `main` is always fine: branch from it for hotfixes, diff against it, pull it for context. **Writing to it goes through a PR.**

> **Enforcement:** TODO(discovery): the GitHub plan this repo is on. On GitHub Team or higher, the ruleset in [`.github/rulesets/branch-protection.json`](../.github/rulesets/branch-protection.json) enforces this rule server-side. On GitHub Free, it's enforced by convention, PR review, and `CODEOWNERS` review requests. See [Enforcement](#enforcement).

---

## Workflow step by step

### 1. Start from a scoped task
Every branch corresponds to one agreed piece of work: a GitHub issue, a Basecamp task, or a written brief. Confirm the scope (and any hours cap) before starting. If the brief locks an architecture or marks something as closed, don't reinterpret it; ask before deviating.

### 2. Create a feature branch off `development`

```bash
git checkout development
git pull origin development
git checkout -b feature/short-description
```

Examples: `feature/collection-image-tiles`, `feature/42-pdp-price-display` (include the issue number when one exists).

### 3. Do the work, commit to the feature branch
Keep commits focused and describe what changed. Follow the repo conventions: [`CLAUDE.md`](../CLAUDE.md), [`docs/liquid-patterns.md`](liquid-patterns.md), [`docs/css-patterns.md`](css-patterns.md), [`docs/javascript-patterns.md`](javascript-patterns.md). Every piece of custom code gets a `MARK:-` comment; new custom files use the `tpm-` prefix. If the work adds or contradicts a convention, update the pattern doc in the same branch.

Run `shopify theme check` before opening the PR. New or changed files must be clean. Pre-existing offenses elsewhere are catalogued in [`.theme-check.yml`](../.theme-check.yml) and aren't yours to fix in a feature PR.

### 4. Add an `Unreleased` changelog entry
Every feature PR adds its notes under an `### Unreleased` heading at the top of the Changelog in `README.md`. **Don't pick a version number on a feature branch.** The version is assigned at release time (step 8), so parallel branches never fight over it.

```markdown
## Changelog

### Unreleased

- Collection image tiles under the collection banner, linking to subcollections and filters
  - New `sections/tpm-collection-tiles.liquid`, `assets/tpm-collection-tiles.css`; setup in `docs/collection-tiles-setup.md`
  - Reviewer verification: tiles render on a collection with the metafield set, scroll arrows work by keyboard, no console errors

### v1.2.0 - YYYY-MM-DD
...
```

See [Documentation pattern](#documentation-pattern) for how to write the entry.

### 5. Open a PR → `development`
Fill in the [PR template](../.github/pull_request_template.md): link the task, summarize the change, list the files changed, include desktop and mobile screenshots or a recording, state the commands run and their results, name known gaps, and confirm the `Unreleased` changelog entry. Contractor PRs follow the [handoff package](#contractor-handoff-package) below.

### 6. QA and approval on `development`
Merging to `development` deploys to the `[DEV-TR] Development theme`. Test there: mobile and desktop, console clean, Liquid renders, theme editor settings behave. The designated reviewer approves. Requested changes go back to the same feature branch and PR.

### 7. Absorb `main` before releasing
`main` accumulates `shopify[bot]` commits from merchant edits in the live theme editor. Before a release PR:

```bash
git checkout development && git pull origin development
git merge origin/main   # resolve conflicts if any — merchant content wins unless the change was intentional
git push origin development
```

Sanity check: `git merge-base --is-ancestor origin/main development` must exit 0. If it doesn't, `main` has commits `development` hasn't absorbed.

### 8. Release PR: `development` → `main`
On the release branch (a short `release/vX.Y.Z` branch off `development`, or directly on `development` if nothing else is in flight):

1. Rename `### Unreleased` to `### vX.Y.Z - YYYY-MM-DD` in `README.md`, and merge the accumulated bullets into a coherent entry.
2. Bump `theme_version` in `config/settings_schema.json` to the same number (no `v` prefix). The two must always match. [`changelog-check.yml`](../.github/workflows/changelog-check.yml) fails the PR if they differ or if an `Unreleased` heading is still present.
3. Open the PR into `main`. It needs at least one human approval.

Semantic versioning: **patch** = bugfixes, copy, small CSS; **minor** = new features or notable layout work; **major** = rebuilds, base-theme upgrades, breaking structural changes.

### 9. Merge = deploy = release
Merging the PR into `main` does three things at once:

- The Shopify GitHub integration pushes `main` to the live theme.
- [`release.yml`](../.github/workflows/release.yml) reads the top changelog entry, creates the `vX.Y.Z` tag, and publishes a GitHub Release with that entry as its notes. Never tag by hand.
- The theme card in the Shopify admin shows the new `theme_version`.

Because the merge is the deploy, **the PR approval is the production checkpoint.** Review the full diff, confirm QA on `development` happened, and merge during hours when someone can watch the live site.

### 10. Close the loop
Merge `main` back into `development` if the release branch diverged. Post the release to the task or Basecamp thread with a merchant-facing summary (see [Documentation pattern](#documentation-pattern)).

---

## Documentation pattern

Every change leaves a trail that a developer and the merchant can each read without the other. Each artifact has one home:

| Artifact | Audience | Where | Written when |
|---|---|---|---|
| Changelog entry | Both | `README.md` → `### Unreleased`, versioned at release | Every PR |
| Release notes | Both | GitHub Release, generated from the changelog by `release.yml` | Automatically on merge to `main` |
| Merchant setup guide | Merchant | `docs/<feature>-setup.md` (from [`_feature-setup-template.md`](_feature-setup-template.md)) | Any feature that needs admin setup: metafields, metaobjects, theme settings, app config |
| Merchant release summary | Merchant | Task / Basecamp thread | After each release: plain language, grouped by what the merchant can now do, with links to setup guides |
| Conventions | Dev | `CLAUDE.md`, `docs/*-patterns.md` | When a convention is discovered or changes |
| Specs and briefs | Dev | `docs/specs/<feature>/` | Before non-trivial work |
| Review records | Dev | `docs/reviews/YYYY-MM-DD-<feature>-round-N.md` | Each review round |
| Inherited debt | Dev | `docs/tech-debt.md` | When debt is found (log it, don't fix it in passing) |

**Changelog entry shape.** The top bullet says what changed **in plain language a merchant would understand** and why. Sub-bullets cover the files, sections, and settings touched (for developers) and a **"Reviewer verification:"** line saying what to check on the dev theme. Because the top bullets are merchant-readable, the GitHub Release doubles as the merchant's record, and the release summary can be lifted from it.

---

## Preview themes and the GitHub integration

The Shopify GitHub integration can connect **any** branch to a development theme, and it writes theme-editor changes back to that branch as `shopify[bot]` "Update from Shopify" commits. That's useful for QA and dangerous for hygiene:

- **Prefer `shopify theme dev`** (`shopify theme dev -e dev`, or `--theme-editor-sync` when you need to save editor changes) for day-to-day work. Nothing is written to Git without you.
- If you connect a feature branch to a development theme for QA, **every setting you touch in that theme editor becomes a commit on your branch.** Only make editor changes that belong to the feature. Never use a connected preview theme for unrelated content edits (product swaps, copy changes), because they'll ship with the feature.
- Bot commits on a feature branch are noise in review. Keep them minimal, call out any intentional ones in the PR description, and never describe settings changes as excluded when the branch history shows otherwise.
- Shopify is a QA surface, not a transport layer. Code moves between people through Git and PRs, never through theme pulls.

---

## Content updates (non-code changes)
Merchant edits in the live theme editor are expected and fine. They reach Git automatically as `shopify[bot]` commits on `main`. Nobody needs to run `shopify theme pull` for content.

---

## Hotfix process

```bash
git checkout main && git pull origin main
git checkout -b hotfix/short-description
# fix, versioned changelog entry (patch bump), theme_version bump
```

PR directly into `main`, one approval, merge (which deploys and releases), then immediately merge `main` back into `development`.

---

## Contractor handoff package

Each contractor PR is a review input. It must include:

- A summary of the changed behavior and how it maps to the brief.
- Files changed, with a sentence on anything touched outside the expected scope.
- Screenshots or a recording of desktop and mobile states.
- Commands run and their results (`shopify theme check` at minimum).
- Known gaps and any design ambiguity encountered.
- Confirmation of: `Unreleased` changelog entry, `MARK:-` comments on all custom code, `tpm-` prefix on new files, no unrelated content or settings changes in the branch history, and a setup guide for anything the merchant must configure.

Reviews are recorded in [`docs/reviews/`](reviews/) (one file per round, `YYYY-MM-DD-<feature>-round-N.md`) so each revision can be checked against what was asked. Those files are Git-tracked and excluded from the theme upload via `.shopifyignore`.

Check-in expectation: for work longer than a few hours, send a short update at each natural stage boundary (markup done, styling done, behavior done), and never go more than about four hours without one. A finished PR shouldn't be the first time the reviewer sees the approach.

---

## Enforcement

### What is in place on every plan
- **`CODEOWNERS`** ([`.github/CODEOWNERS`](../.github/CODEOWNERS)) routes structural paths (`/layout/`, `/config/`, the token snippets, `/docs/`, `CLAUDE.md`, `AGENTS.md`, `.github/`) to the project lead. GitHub requests that review automatically on every PR touching them.
- **CI status checks**: [`theme-check.yml`](../.github/workflows/theme-check.yml) runs `shopify theme check --fail-level error` on every PR into `development` and `main`; [`changelog-check.yml`](../.github/workflows/changelog-check.yml) guards release PRs into `main`; [`release.yml`](../.github/workflows/release.yml) tags and publishes on merge.
- **[`.theme-check.yml`](../.theme-check.yml)** records the inherited error baseline so CI is green on the current tree and fails only on **new** breakage. Never silence further checks there to get a PR green; fix the offense or raise it in the PR.

### What needs GitHub Team
Branch protection (require a PR, one approval, no admin bypass, required status checks, required code-owner review) isn't available on private repos under GitHub Free. On Team or higher, import [`.github/rulesets/branch-protection.json`](../.github/rulesets/branch-protection.json) (**Settings → Rules → Rulesets → Import**) and enable "Do not allow bypassing." Without that, admins and any agent running under admin credentials can still push straight to `main`.

### Credentials
- Contractors and agents get the least access that lets them work: a collaborator seat with write access to open PRs, never admin.
- Only the project lead holds Shopify CLI or admin credentials for the live store. The integration deploys; people don't push.

---

## Rules for AI agents (all of them)

AI tools are legitimate contributors here **inside the same boundaries as everyone else**. See [`AGENTS.md`](../AGENTS.md) for the mandatory list.

**Agents may:**
- Read anything, including `main`
- Create `feature/*` branches off `development`
- Commit to feature branches and open PRs targeting `development`, when the person they work for has asked them to

**Agents must never:**
- Commit or push to `main` or `development`
- Merge any PR
- Run `shopify theme push` / `shopify theme share` against any theme, or edit the live theme through admin APIs
- Modify `.github/` workflows, `CODEOWNERS`, or ruleset configuration

---

## Quick reference

```
Scoped task agreed
    ↓
feature/* branch off development
    ↓
Work + MARK:- comments + tpm- prefix + docs updated + README "Unreleased" entry
    ↓
PR → development (review + QA on the [DEV-TR] Development theme)
    ↓
Merge main into development (absorb merchant content)
    ↓
Release: Unreleased → vX.Y.Z, bump theme_version, PR → main
    ↓
Merge → live theme deploys → tag + GitHub Release created automatically
    ↓
Merge main back into development, post the merchant summary, close the task
```
