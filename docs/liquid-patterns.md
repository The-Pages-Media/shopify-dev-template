# Liquid Patterns — [Store Name] (TODO(discovery): vendor theme + version)

Philosophy: HTML → Liquid → CSS → JS. The Shopify way first: server-render, use native settings/blocks/section groups/metafields, and use the Section Rendering API before custom fetch-and-DOM code. **Reuse an existing snippet before writing markup.** The theme's product card, image, icon, and price snippets already solve most rendering problems, so rendering one of them is smaller than writing a new one. Every custom addition or modification carries a `MARK:-` comment.

---

## First-pull discovery

> Run these on the first pull (see `docs/theme-onboarding.md`), then fill in every `TODO(discovery)` section below with `file:line` citations and counts.

**Shape of the theme:**

```bash
for d in layout sections snippets blocks templates locales; do printf '%-10s %s\n' $d "$(ls $d 2>/dev/null | wc -l)"; done
ls sections/*.json                                         # section groups (header/footer/popups)
ls sections | sed -E 's/-.*//' | sort | uniq -c | sort -rn | head   # naming families (main-*, custom-*, …)
ls sections snippets assets | grep -iE 'custom|tpm'        # earlier customizations
```

**What to reuse** (the most-rendered snippets are the theme's building blocks):

```bash
grep -rhoE "render '[^']+'" sections snippets layout blocks | sort | uniq -c | sort -rn | head -40
grep -rhoE "render '[a-z0-9_-]*(image|media|icon|price|card|button)[a-z0-9_-]*'" sections snippets | sort | uniq -c | sort -rn
grep -rlE '\{% ?doc ?%\}' snippets | wc -l                 # snippets documented with {% doc %}
```

**Data the theme reads:**

```bash
grep -rhoE 'metafields\.[a-z0-9_]+\.[a-z0-9_]+' --include='*.liquid' . | sort | uniq -c | sort -rn
grep -rhoE 'metaobjects\.[a-z0-9_]+' --include='*.liquid' . | sort -u
grep -rl '"@app"' sections blocks                          # sections that accept app blocks
```

**Schema conventions:**

```bash
printf 'with presets: %s / %s\n' "$(grep -l '"presets"' sections/*.liquid | wc -l)" "$(ls sections/*.liquid | wc -l)"
grep -L 'shopify_attributes' $(grep -l '"blocks"' sections/*.liquid)   # sections with blocks but no shopify_attributes
grep -rhoE '"(enabled_on|disabled_on|limit|class|tag)"' sections/*.liquid | sort | uniq -c
grep -rhoE '"label": "t:' sections/*.liquid | wc -l ; grep -rhoE '"label": "[^t]' sections/*.liquid | wc -l
```

**Deprecated or risky patterns** (these go in `docs/tech-debt.md`, not fixed in a sweep):

```bash
grep -rn '{% include' layout sections snippets templates
grep -rnE '\| ?img_url' layout sections snippets | wc -l
grep -rn 'request.design_mode' sections snippets | head    # precedent for editor-only hints
```

---

## Mandatory Liquid rules (any theme)

1. Multi-line logic goes in a `{% liquid %}` block **at the top of the file**, not in scattered standalone `{% assign %}` tags.
2. **Initialize with a default, then conditionally reassign.** NEVER `assign x = a and b != blank`; compound boolean assignment breaks the page. Compound conditions inside `if`/`unless` are fine.
3. File order: Liquid assigns → CSS (`{% style %}`) → HTML → JS → `{% schema %}`.
4. Comments follow Liquid convention, and `MARK:-` always comes first so `grep -rn "MARK:-"` finds every one:
   - single line: `{% # MARK:- Short description %}`
   - multi-line: `{% comment %} MARK:- Heading line, then the explanation {% endcomment %}`
   - inside a `{% liquid %}` block: `# MARK:- ...` as a line
5. Don't use `| default:` to mask a data read; branch on the real value. It's fine for presentation fallbacks such as alt text. Remember `default` treats `false` and `0` as empty; use `allow_false: true` or a real branch when zero or false is a valid value.
6. `{% render %}` only, never `{% include %}`. Don't pass filtered values as `render` arguments (`alt: x | default: y` is a Theme Check `LiquidSyntaxError`); assign first.
7. Escape merchant text that lands in attributes or headings (`| escape`). Output from `richtext`/`inline_richtext` settings is already HTML.
8. Fixed UI copy uses `| t`; merchant-editable copy is a schema setting with a sensible English default.

## Section anatomy

TODO(discovery): how the theme's own sections are built. Wrapper element and attributes (`data-section-id`, `data-section-type`), layout wrapper class (`.page-width` or similar), heading markup, where section padding comes from (schema `class`, settings, or utility classes), and whether it uses `{% style %}` for per-instance values. Cite one representative vendor section and any existing custom sections, noting what to copy and what not to.

**New sections follow this model** (adjust wrapper and heading classes to the theme's own):

```liquid
{% # MARK:- tpm-<feature> — what it does %}
{% liquid
  assign show_cta = false
  if section.settings.cta_text != blank and section.settings.cta_link != blank
    assign show_cta = true
  endif
%}
{% style %}
  #shopify-section-{{ section.id }} { --tpm-accent: {{ section.settings.accent }}; }
{% endstyle %}
<div class="page-width" data-section-id="{{ section.id }}" data-section-type="tpm-<feature>">
  <h2>{{ section.settings.title | escape }}</h2>
  {% for block in section.blocks %}
    <div {{ block.shopify_attributes }}>…</div>
  {% endfor %}
  {% if request.design_mode %}<p>{{ 'sections.tpm_<feature>.editor_note' | t }}</p>{% endif %}
</div>
{% # Only if genuinely needed; prefer a custom element in assets/tpm-<feature>.js %}
<script src="{{ 'tpm-<feature>.js' | asset_url }}" defer="defer"></script>
{% schema %}…{% endschema %}
```

## Section / snippet naming taxonomy

TODO(discovery): one row per naming family found above.

| Name pattern | Meaning | Edit policy |
|---|---|---|
| `main-*` | `"main"` entry of a JSON template | Vendor: surgical edits, marked `MARK:-` |
| bare kebab (`rich-text`, `slideshow`, …) | Vendor marketing/utility sections | Vendor: same |
| `*-group.json` | Section groups | Editor-owned; don't hand-edit |
| earlier customs (`custom-*`, unprefixed) | Prior agency or earlier work | Ours: keep names, mark when touched |
| **`tpm-*`** | **All new custom sections/snippets/blocks/assets** | Ours |
| app snippets/blocks (`ai_gen_block_*`, `klaviyo-*`, analytics) | App- or AI-owned | Don't hand-edit; replace with a `tpm-*` file if changes are needed |

## Snippet conventions

- **Document params in a `{% doc %}` header** with `@param` (and `[optional]` params in brackets) for every new snippet.
- **Reuse first.** TODO(discovery): the snippets new work should render instead of writing markup: product card, collection card, image/media, price, icon(s), buttons, pagination, breadcrumbs, quantity input. Include each one's params.
- **Icons:** TODO(discovery): how the theme ships icons (one snippet per icon, a sprite, inline SVG). New icons: `snippets/tpm-icon-<name>.liquid`, `stroke`/`fill="currentColor"`, `aria-hidden="true"`, `focusable="false"`.

## Schema conventions (any theme)

- **Presets** on every merchant-addable section; none on `main-*` or template-bound sections.
- Group settings with `{"type": "header"}` dividers; give text settings a `"default"` so presets render non-empty.
- `"disabled_on": { "groups": ["header", "footer", …] }` on addable content sections; `"enabled_on"` for template-bound ones.
- **`{{ block.shopify_attributes }}` on every block wrapper**, so the editor can select blocks.
- `"limit": 1` on any section that fetches or does heavy per-instance work.
- Prefer native setting types (`color_scheme`, `image_picker`, `video`, `product`, `collection`, `metaobject`, `range`) over free text. Use `range` instead of free-typed numbers.
- Schema labels: follow the theme's convention (`t:` keys vs plain English). TODO(discovery): which it is, and whether translations are a priority for this store.

## Translations

TODO(discovery): storefront locales present, `*.schema.json` locales present, and how many `| t` uses exist. Reuse existing keys before adding. When adding a key, add it to every storefront locale. Editor-only notes behind `request.design_mode` may stay English.

## Image rendering

TODO(discovery): the theme's image path (an image snippet and its params, or direct `image_url | image_tag`), its lazy-loading default, `sizes`/`widths` handling, the placeholder pattern, and any reveal-on-scroll behavior that hides images inserted by JS.

House rules (any theme):

- `image_url | image_tag` (directly or through the theme's snippet), which emits `width`/`height`. **No new `| img_url`** and no hand-rolled `srcset`.
- `loading: 'lazy'` below the fold; `loading: 'eager'` plus `fetchpriority: 'high'` only for the LCP image. Always pass `sizes` when the image isn't full-width.
- Real `alt` text from the image's alt, falling back to the product or collection title. Use `alt=""` for decorative images.

## Metafields & metaobjects

TODO(discovery): every namespace found above.

| Namespace / object | Status | Where |
|---|---|---|
| `product.metafields.custom.*` | … | `file:line` |

Access pattern: assign at the top, `.value` on reference/list types, guard with `!= blank`. New custom data goes in a store-appropriate namespace (not the vendor's), and comes with a merchant setup guide in `docs/<feature>-setup.md` (see `docs/_feature-setup-template.md`). Definitions are store data: the theme can't create them, so the guide has to walk the merchant through creating them. Prototype new definitions on the client dev store (`-e dev`) before creating them on prod.

## `layout/theme.liquid` map

TODO(discovery): line-referenced map of the layout: preconnects, analytics, fonts, stylesheets and tokens, the inline JS globals, `content_for_header`, scripts, body attributes, skip link, section groups, `<main>`, templates, and modals. This is where you look to find where to hook anything global.

## Templates

TODO(discovery): count of JSON templates and alternates, Liquid templates (customers, gift card, `*.ajax` / `?view=` alternates), and any template that serves as a data source for other features.

## Theme Check baseline

TODO(discovery): CLI version, date measured, total files/errors/warnings, and each pre-existing error-level check with counts and files. `.theme-check.yml` records the same baseline so CI fails only on **new** error-level offenses. Never add to its ignore lists to get a PR green.

## `docs/` policy

- `docs/*-patterns.md`, `development-workflow.md`, `theme-onboarding.md`: conventions
- `docs/<feature>-setup.md`: merchant-facing setup guides, one per feature that needs admin setup
- `docs/specs/<feature>/`: briefs, specs, revisions, design references
- `docs/reviews/`: review records, `YYYY-MM-DD-<feature>-round-N.md`
- `docs/tech-debt.md`: verified inherited debt

`docs/` is Git-tracked and listed in `.shopifyignore`, so it's never uploaded to a theme.
