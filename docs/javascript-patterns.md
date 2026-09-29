# JavaScript Patterns — [Store Name] (TODO(discovery): vendor theme + version)

JavaScript is the last resort: HTML → Liquid → CSS → JavaScript, and never JS for what CSS can do. When JS is warranted, **the first question is what the theme already provides**: a custom element, a global helper, a cart method, or an event to listen for or dispatch. The smallest correct change usually listens to an existing event or calls an existing helper instead of adding a new mechanism. Every new addition carries `// MARK:-`.

---

## First-pull discovery

> Run these on the first pull (see `docs/theme-onboarding.md`), then fill in every `TODO(discovery)` section below with `file:line` citations. Paths assume the repo root; add `blocks` if the theme has it. Ignore minified vendor bundles in counts unless noted.

**Runtime and frameworks.** Anything non-zero here gets noted in `CLAUDE.md` and in the asset inventory as **legacy: do not extend**. Agency-built themes often stack jQuery, Alpine, sliders, or a compiled framework layer. We don't build on any of it; new code is native.

```bash
grep -rlE 'jQuery|\$\(document\)|\$\.ajax|Alpine|x-data|Vue|React' --include='*.js' --include='*.liquid' assets layout sections snippets
ls assets/*.js | xargs wc -l | sort -rn          # size of every JS asset
grep -rn '<script' layout/theme.liquid           # load order, defer/async, inline globals
grep -rnE '<script[^>]+src=' layout sections snippets | grep -v asset_url   # third-party scripts → tech-debt.md
```

**What the theme exposes to reuse:**

```bash
# Custom elements the theme defines (names, and whether defines are guarded)
grep -rhoE "customElements\.define\([[:space:]]*['\"][a-z0-9-]+" assets sections snippets | sort | uniq -c
grep -rnE "customElements\.get\(" assets sections snippets | wc -l

# Custom events: dispatched, and listened to (skip native DOM events like click/change)
grep -rhoE "new (Custom)?Event\([[:space:]]*['\"\`][^'\"\`]+" assets sections snippets | sort | uniq -c | sort -rn
grep -rhoE "addEventListener\([[:space:]]*['\"\`][^'\"\`]+" assets sections snippets | sort | uniq -c | sort -rn

# Pub/sub bus (Dawn-family themes: PUB_SUB_EVENTS, subscribe/publish)
grep -rnE 'PUB_SUB_EVENTS|subscribe\(|publish\(' assets | head

# Global namespace and helpers
grep -rhoE 'window\.[A-Za-z_]+' assets layout snippets sections | sort | uniq -c | sort -rn | head -30
grep -rnE 'debounce|throttle|trapFocus|formatMoney|fetchConfig|getSectionsToRender' assets | head -30
```

**How the theme talks to Shopify:**

```bash
grep -rnE "/cart(/add|/change|/update|/clear)?(\.js)?|routes\.cart" assets snippets sections layout
grep -rnE "section_id=|sections=|\?view=" assets sections snippets   # Section Rendering API vs alternate templates
grep -rhoE "shopify:(section|block):[a-z]+" assets sections snippets | sort | uniq -c   # theme-editor hooks
```

**Prior customization and hygiene:**

```bash
grep -rn 'MARK:-' assets sections snippets layout | wc -l
grep -rn 'console\.log' assets sections snippets | grep -v '\.min\.js'
ls assets | grep -iE 'custom|tpm'                     # an existing hand-editable custom.js?
```

If you can get a clean copy of the same vendor theme version (Theme Store / theme library download), diff its `assets/*.js` against the pulled theme. That's the fastest way to find earlier hand edits in "vendor" files.

---

## Asset inventory & edit policy

TODO(discovery): one row per JS asset. Policy is one of **Frozen vendor** (read it, call it, listen to it, never edit), **Frozen compiled** (build artifact whose source isn't in the repo), **Surgical only** (marked in-place edits of existing behavior, when overriding from outside is impossible), or **Ours** (hand-editable).

| Asset | What it is | Policy |
|---|---|---|
| `assets/theme.js` (N lines) | Vendor core: namespace, custom elements, section registry | Frozen vendor |
| `assets/custom.js` | Ours, if present | Ours, with `// MARK:-` |

## Script loading

TODO(discovery): the order scripts load in `layout/theme.liquid` (inline globals → `content_for_header` → deferred vendor → deferred custom), which templates load extra scripts, and the "theme ready" signal if there is one. Note any body attributes that toggle behavior (e.g. an animations-disabled flag) and whether JS reads them.

**Where new JS goes (house rule, any theme):**

- **Small, section-local behavior** → `{% javascript %}` at the bottom of the section, before `{% schema %}`. The trade-off: Shopify concatenates every `{% javascript %}` block into one file loaded on **every** page. Guard for the element's absence and keep it small.
- **Large or template-specific feature JS** → `assets/tpm-<feature>.js`, loaded from the section with `<script src="{{ 'tpm-<feature>.js' | asset_url }}" defer="defer"></script>`, so it loads only where the section renders.
- **Cross-page behavior** → the theme's hand-editable custom file (`assets/custom.js` or equivalent), under a `// MARK:-` heading. If none exists, create `assets/tpm-custom.js`, load it `defer` after the vendor script in the layout, and record it here.
- **Never** a third-party CDN `<script>`. App SDKs that can't be self-hosted load `defer`/idle and are listed in `docs/tech-debt.md` if they're render-blocking.

## The theme's global contract

TODO(discovery): the global namespace (`window.theme`, `window.themeVariables`, `window.Shopify`, …) and what is on it: routes, translated strings, settings, config/breakpoints. **Use these instead of new globals or hardcoded URLs.** Record how to add a translated string for JS (e.g. extending the strings object from the section that needs it) so no one invents a second mechanism.

Helpers worth reusing, with `file:line`: debounce/throttle, focus trap, money formatting, cart get/change, drawer/modal open/close, section re-render, product-card re-init after injection, lazy/visible init, library loader.

## Custom elements the theme defines

TODO(discovery): every `customElements.define` from the search above. This is the first place to look before building UI.

| Element | Defined at | What it does | Reuse notes |
|---|---|---|---|
| `<example-element>` | `assets/theme.js:NNN` | … | Guarded? Config via `data-*`? |

## Custom event vocabulary

TODO(discovery): every event the theme dispatches or listens to, from the searches above. **Listening to or dispatching an existing event is almost always smaller than new code.** The rows that matter most: what fires after add-to-cart, what makes the cart drawer rebuild/open, what fires on cart change, variant change, collection filter/sort re-render, quick view/recommendations loaded, and theme ready.

| Event | Target · source | Detail | Use |
|---|---|---|---|
| `example:added` | `document`, bubbles (`assets/theme.js:NNN`) | `{ … }` | Dispatch after a programmatic add so the drawer rebuilds itself |

**Our events** are named `tpm:<domain>:<action>` and dispatched with `new CustomEvent(name, { bubbles: true, detail })`. Add each new one to this table.

---

## Writing new JS (any theme)

1. **Exhaust the platform first**: Liquid conditionals; CSS `:hover`/`:checked`/`:has()`/`<details>`/`<dialog>`; an existing helper, custom element, or event from the tables above. Render server-side, and fetch rendered HTML with the Section Rendering API before building JSON→DOM by hand.
2. **New interactive UI is a Web Component**, prefixed **`tpm-`**. Use a plain `HTMLElement`; read config from `data-*` attributes rendered by Liquid; bind listeners in `connectedCallback`; release them with one `AbortController` in `disconnectedCallback`; guard the define. This also makes the component safe when the theme editor re-renders it.

   ```javascript
   // MARK:- Look row — mounts Liquid-rendered markup and wires prev/next
   class TpmLookRow extends HTMLElement {
     connectedCallback() {
       this.controller = new AbortController();
       const { signal } = this.controller;
       this.querySelector('[data-next]')?.addEventListener('click', () => this.step(1), { signal });
       document.addEventListener('cart:updated', (event) => this.onCart(event.detail), { signal }); // use the theme's real event name
     }

     disconnectedCallback() {
       this.controller.abort();
     }

     step(direction) {
       const behavior = matchMedia('(prefers-reduced-motion: reduce)').matches ? 'auto' : 'smooth';
       this.scrollBy({ left: direction * this.clientWidth, behavior });
       this.dispatchEvent(new CustomEvent('tpm:look:changed', { bubbles: true, detail: { index: this.index } }));
     }
   }

   if (!customElements.get('tpm-look-row')) customElements.define('tpm-look-row', TpmLookRow);
   ```

3. **Bind by component or by delegation, never with a page-load `querySelectorAll` sweep.** Product cards, drawers, and sections are re-rendered by AJAX filtering and the theme editor, so nodes bound at `DOMContentLoaded` go dead. Use a component, or one `document`-level listener with `event.target.closest(...)`.
4. **Modern native JS only**: `const`/`let` (never `var`), `async/await`, optional chaining, `fetch`, web components. **Never add a library, and never extend one the theme already loads.** If the theme ships jQuery, Alpine, Swiper/Flickity, or an agency framework layer, new code still uses platform APIs: `<dialog>`, `<details>`, `popover`, CSS scroll-snap, `IntersectionObserver`, `fetch`. Don't stack another dependency on top. Calling a vendor helper that wraps a library is fine when it's the theme's own API (e.g. its slideshow initializer); instantiating the library directly or writing new code in its style is not. No new `window.*` globals (extend the theme's namespace or use `data-*`). No `console.log`; `console.warn`/`console.error` only on a genuine failure path.
5. **Accessibility is an acceptance criterion**: `aria-busy` on any async button; `aria-expanded`/`aria-controls` on disclosures; keyboard operability (arrow/Home/End on tab-like UIs, Esc on overlays); focus moved into and restored out of overlays (use the theme's focus trap); state changes announced through an `aria-live` region.
6. **Performance is an acceptance criterion**: `defer` on every script tag; `matchMedia('(prefers-reduced-motion: reduce)')` before any programmatic animation; `IntersectionObserver` (or the theme's visible-init helper) before fetching below-the-fold content; `width`/`height` on injected images; debounce input and resize handlers; batch DOM writes; cache repeated queries.
7. **Liquid → JS data** goes through `data-*` attributes or a `<script type="application/json">` island. Don't interpolate Liquid into JS logic.
8. **Naming**: elements `tpm-*`, events `tpm:<domain>:<action>`, files `assets/tpm-<feature>.js`.

## Cart & fetch patterns

TODO(discovery): how the theme adds to cart (form-urlencoded vs JSON, headers, how it surfaces a 422 error), what event it fires afterward, and how the cart drawer/notification refreshes. Name the method to call for re-reading the cart and for money formatting.

House rules (any theme):

- **Use the real product form** wherever one exists, and let the theme's own add-to-cart handle it.
- **Programmatic adds** (quick add, add-all) go through **one** shared `async` helper: JSON `{ items: [...] }` to the cart-add route from the theme's routes object; on `!response.ok` read `description` and show it; set and clear `aria-busy`; then **dispatch the theme's own "added" event** so the drawer rebuilds and opens itself. Never rebuild the drawer by hand. If a helper already exists, extend it; never add a second one.
- **Fetch rendered HTML with the Section Rendering API** (`?section_id=` or `?sections=`). **Never fetch a whole storefront page and scrape it.**
- **Prices**: render in Liquid when possible; in JS, use the theme's money formatter with its money format setting.

## Theme editor

TODO(discovery): how the theme re-initializes on `shopify:section:load/unload/select/deselect` and `shopify:block:select/deselect` (a registry keyed by `data-section-type`, per-component listeners, or nothing).

House rule: **new code does not register with the vendor's section registry.** Custom elements get editor safety automatically, because `connectedCallback`/`disconnectedCallback` re-run when the editor swaps section HTML. Add explicit `shopify:*` listeners only for stateful re-init, gated on `Shopify.designMode`.

## Do NOT imitate

Universal anti-patterns. Reject them in review wherever they appear:

- `DOMContentLoaded` + `querySelectorAll` binding sweeps (dead after AJAX or editor re-render)
- `var`, function-declaration IIFEs, unguarded `customElements.define`
- Fetching a full storefront page and rewriting theme markup at runtime (breaks silently when the card markup changes)
- Shadow DOM for themed UI (it walls the component off from the theme's tokens and `custom.css`); use light DOM
- Timer choreography (`setTimeout` inside `setInterval`) as animation; `location.reload()` as state refresh
- Inline `onclick`, third-party CDN scripts, `console.log`, document listeners with no teardown
- JS that reads or writes styles for something a class toggle plus CSS could do

TODO(discovery): theme-specific instances of the above, with `file:line`, so nobody copies them.
