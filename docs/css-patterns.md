# CSS Patterns — [Store Name] (TODO(discovery): vendor theme + version)

CSS before JavaScript, always: hover, toggle, show/hide, sticky, and scroll-snap behavior belong in `:hover`, `:checked`, `:has()`, `<details>`, media queries, and the theme's existing utilities before any script. Never CSS for what HTML already does. **The smallest footprint in CSS means reusing the theme's own tokens, classes, and breakpoints.** A new rule that restates an existing utility, a new hex color that already exists as a token, or a new breakpoint the theme doesn't use is a defect. All new custom CSS starts with `/* MARK:- description */`.

---

## First-pull discovery

> Run these on the first pull (see `docs/theme-onboarding.md`), then fill in every `TODO(discovery)` section below with `file:line` citations and counts.

**Where CSS lives and how it loads:**

```bash
ls -la assets/*.css* ; grep -rnE 'stylesheet_tag|<link[^>]+stylesheet' layout sections snippets
grep -rlE '\{%-? ?style ?-?%\}|<style' sections snippets | wc -l        # section-scoped style blocks
grep -rl '{% stylesheet %}' sections snippets blocks                  # concatenated section stylesheets
ls package.json tailwind.config.* postcss.config.* 2>/dev/null        # any build pipeline?
```

**The token system** (where settings become custom properties, and their format):

```bash
grep -rlE -- '--[A-Za-z0-9_-]+:[[:space:]]*\{\{' snippets layout assets     # the settings → token file(s)
grep -rhoE -- '--[A-Za-z0-9_-]+:' snippets/css-variables.liquid 2>/dev/null | sort -u
grep -rhoE 'rgba?\(var\(--[A-Za-z0-9_-]+' assets | sort | uniq -c | sort -rn | head   # RGB-triplet tokens?
```

**Existing design patterns to reuse:**

```bash
# Canonical breakpoints, by frequency — use only these
grep -rhoE '@media[^{]+' assets sections snippets | sed -E 's/[[:space:]]+/ /g' | sort | uniq -c | sort -rn | head -20

# Most-defined classes (layout wrappers, buttons, grids, utilities, visibility helpers)
grep -hoE '^\.[A-Za-z][A-Za-z0-9_-]*' assets/*.css* | sort | uniq -c | sort -rn | head -60
grep -hoE '\.(visually-hidden|sr-only|page-width|container|btn|button|rte|grid|hidden|small--hide|medium-up--hide)[A-Za-z0-9_-]*' assets/*.css* | sort -u

# Stacking order (headers, drawers, modals, sticky bars)
grep -rhoE 'z-index:[[:space:]]*-?[0-9]+' assets sections | sort | uniq -c | sort -rn

# Fonts actually loaded (weights!) and where from
grep -rnE '@font-face|font_face|font_url|fonts\.googleapis|fonts\.shopifycdn' layout snippets assets
```

**Accessibility and hygiene baseline** (record the counts; new code must beat them):

```bash
for p in ':focus-visible' 'prefers-reduced-motion' '!important' 'hover: hover' 'outline: ?(none|0)'; do
  printf '%-24s %s\n' "$p" "$(grep -rhoE -- "$p" assets sections snippets | wc -l)"; done
grep -rhoE '#[0-9a-fA-F]{6}\b|#[0-9a-fA-F]{3}\b' assets/*.css* | sort | uniq -c | sort -rn | head   # hardcoded hex
grep -rn 'MARK:-' assets/*.css* | wc -l
```

If you can get a clean copy of the same vendor theme version, diff its stylesheets against the pulled theme to find earlier hand edits.

---

## Where CSS lives

TODO(discovery): one row per layer, with status.

| Layer | File | Status |
|---|---|---|
| Global stylesheet | `assets/…` — loaded at `layout/theme.liquid:NN` | Vendor — **frozen** |
| Our global overrides | `assets/custom.css` (if present; loaded after vendor?) | Ours |
| Design tokens | `snippets/css-variables.liquid` (or equivalent) | Settings → tokens |
| Fonts | `snippets/…` | `font-display`? preconnect? |
| Section-scoped | `{% style %}` in N sections, raw `<style>` in N | Per-instance values |

## Where OUR CSS goes (house rule, any theme)

1. **Section-local styles** → that section's `{% style %}` block, scoped to `#shopify-section-{{ section.id }}`, marked `/* MARK:- */`. It's removed along with the section. `{% style %}` output is inlined per instance, so keep it to per-instance *values* (custom properties), not long rule sets.
2. **Small shared rules for one section type** (up to ~100 lines) → `{% stylesheet %}` inside the section. **Trade-off:** Shopify concatenates every `{% stylesheet %}` into one file loaded on **every page**. That's fine for small static rules, not for a big feature or anything page-specific.
3. **Cross-page overrides and new global styles** → the theme's hand-editable custom stylesheet (`assets/custom.css` or equivalent), each block starting `/* MARK:- <feature> */`. It must load **after** the vendor stylesheet so plain specificity wins without `!important`. If no such file exists, create `assets/tpm-custom.css`, load it after the vendor CSS in the layout, and record it above.
4. **A large self-contained feature** → `assets/tpm-<feature>.css`, loaded from the layout `<head>` behind a template/section condition. A `stylesheet_tag` emitted mid-body is render-blocking for everything after it; that's acceptable only for a below-the-fold section on a single template.
5. **Never** edit the vendor stylesheet for new work. If a vendor rule must change, override it from the custom stylesheet and cite the vendor `file:line` in the `MARK:-` comment.

## Design tokens

TODO(discovery): the token format (plain values used as `var(--x)`, or RGB triplets used as `rgb(var(--x))` / `rgb(var(--x) / 0.5)`), then list the tokens by group with `file:line`:

- **Colors:** …
- **Layout / spacing / gutters / page width:** …
- **Typography:** families, base size, heading scale, weights. Use size arithmetic from tokens (`calc(var(--base-size) * 0.875)`), not literal px.
- **Shape / misc:** button radius, border widths, shadows, icon stroke.

**Known token quirks:** TODO(discovery): tokens that are misnamed, duplicated, dead, or not what their name says. Record them; don't "fix" them in passing.

New CSS takes colors, type, and spacing from tokens. **Never a new hex value or literal px size when a token exists.** A per-feature value that merchants control becomes a `--tpm-*` custom property set from section settings.

## Breakpoints

TODO(discovery): the theme's media queries from the search above, by frequency, plus any visibility helper classes tied to them.

| Query | Count | Helper class |
|---|---|---|
| `(max-width: NNNpx)` | N | `.…--hide` |

Use only these. **Never introduce a foreign breakpoint** (a 767 in a 749/750 theme, a 1024 in a 989/990 theme). Watch for off-by-one overlaps (`max-width: 768px` with `min-width: 768px`).

## Class naming

TODO(discovery): the theme's naming convention (BEM `block__element--modifier`, utility classes, a Tailwind layer?), its layout wrappers, buttons, grid, rich text, visually-hidden helper, and state classes (`.is-active`, `[aria-expanded]`, `[hidden]`). Reuse these instead of writing equivalents.

**New custom blocks use the `tpm-` prefix** in the theme's convention: `.tpm-look-row__thumbnail`, `.tpm-look-row--compact`. Style custom elements by class, not tag name. Don't extend a legacy utility framework the theme is moving off.

## Section-scoped styling (canonical)

Section settings become custom properties on the section root; rules only ever consume `var()`:

```liquid
{% style %}
  /* MARK:- Look row: merchant colors from section settings */
  #shopify-section-{{ section.id }} {
    --tpm-look-accent: {{ section.settings.accent_color }};
    --tpm-look-gap: {{ section.settings.gap }}px;
  }
{% endstyle %}
```

Rules that read those properties live in `{% stylesheet %}` (small) or the feature file (large). Conditional whole declarations from Liquid and media queries inside the block are fine.

## Images the CSS depends on

TODO(discovery): does the theme hide images until JS reveals them (fade-in on scroll, lazy-load classes, AOS)? Do aspect-ratio boxes use `aspect-ratio` or padding hacks? Anything cloned or injected by JS may never get the reveal class. Record the sanctioned fix (render server-side, or scope the reveal inside our block), and never apply it globally.

## Hover, focus, motion — new code must beat the theme's baseline

TODO(discovery): the baseline counts from the hygiene search (`:focus-visible`, `prefers-reduced-motion`, `!important`, `(hover: hover)`, `outline: none`), plus any theme-level animation toggle (e.g. a body attribute) and the theme's easing and duration values.

Acceptance criteria for everything we add (any theme):

- **Focus:** every new interactive element gets a visible `:focus-visible` rule that doesn't depend on theme JS (such as a "tab pressed" class).
- **Motion:** anything beyond a simple opacity fade gets a `@media (prefers-reduced-motion: reduce)` guard, plus the theme's own animations-off hook if it has one. Animate `transform`/`opacity` only; never `box-shadow`, `width`, `top`, or other layout properties. Match the theme's durations and easing.
- **Touch:** interactive targets at least 44×44px (a smaller visible control can sit inside a 44px hit area). Put meaningful hover effects inside `@media (hover: hover) and (pointer: fine)`.
- **Contrast:** text meets WCAG AA against its background. Check merchant-controlled color pairs in the setup doc if nothing validates them.
- **`!important`:** only to beat an inline style or app CSS, with the reason in the `MARK:-` comment.
- **Newer features** (`color-mix()`, container queries, `:has()`, `@layer`): write down the support decision, or provide a fallback declaration first or wrap the feature in `@supports`.
- **Layout stability:** reserve space for anything that loads late (images, app widgets, injected rows) so it doesn't cause CLS.

## Do NOT imitate

Universal anti-patterns: hardcoded hex where a token exists; literal px type sizes; foreign breakpoints; `!important` without a reason; animating `box-shadow` or layout properties; hover effects with no `(hover: hover)` guard; 32px touch targets; `scroll-snap-type: mandatory` fighting JS scroll; a global override of a vendor reveal/opacity rule; styling custom elements by tag name.

TODO(discovery): theme-specific instances, with `file:line`, so nobody copies them.
