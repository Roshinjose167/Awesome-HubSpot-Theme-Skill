# Architecture rules

Read these every time you build or extend the theme. Every module ships under these rules. They are not preferences — they are what the OneIMS production catalog (Aagard, KineticGPO, Ondex/ACI, Meridian IT, Minitab, Smartguard, Devpulse, Logistics, Healthcare, Legal Law, MyGovBill, …) has converged on after dozens of revisions.

## 1. Tokens are the only source of truth

- Brand decisions (color, font, spacing, radius, padding) live in `fields.json` at the theme root.
- `<theme>/css/theme-overrides.css.html` is the **only** place those tokens become CSS.
- It emits one `<style>` block with `:root { --aci-…: {{ theme.x }}; }` — see [theme-fields-spec.md](theme-fields-spec.md) for the full variable map.
- `theme-overrides.css.html` is included from the theme's base layout (`templates/layouts/base.html`) inside `<head>` so every page picks it up.

### The token-bridge contract (KineticGPO pattern — the cleanest in production)

Write the emitter in **two steps**: HubL `{% set %}` locals first, then one clean `:root` block. Open the file with a header comment that documents the contract:

```hubl
{#
  Token bridge — the SINGLE connection between Design Manager → Theme Settings
  and the CSS variables every module/template consumes.
  Loaded AFTER main.css so this :root block OVERRIDES the hardcoded defaults
  baked into main.css :root. Do not run a CSS formatter on this file.
#}
{% set font_display = "'" ~ theme.typography.display_font.font ~ "', " ~ theme.typography.display_font.fallback %}
<style>
:root {
  --aci-primary: {{ theme.global_colors.primary_color.color }};
  --aci-font-display: {{ font_display }};
  /* … */
}
</style>
```

Rules that follow from the contract:

- `main.css` may ship a hardcoded `:root` fallback block with the same variable names — the theme-overrides emitter **wins by load order**, so a HubL failure can never leave the site unstyled. Document the load order with a comment next to the `<link>`/include lines in `base.html`.
- **Emit from field `name`s, never field `id`s** (`theme.spacing.max_width`, not `theme.spacing.spc_max_width`).
- **Never let a CSS formatter touch a HubL-bearing stylesheet** — Prettier rewrites `{% set %}` into invalid `{ % set % }` and silently kills the whole settings panel (this shipped on Ondex). Add these files to `.prettierignore`.
- **Path-diff before done:** every `theme.<group>.<field>` read in the emitter must exist in `fields.json`; every `var(--x)` consumed anywhere must be declared. Three production themes shipped silent drift here (see [upload-validation.md §19–21](upload-validation.md)).
- Scope theme-settings typography to **bare elements only** (`h1–h6`, `body`) — component headings (`.section-title`, `.hero h1`) keep their design values and win on specificity, so enabling a setting causes zero visual change until an editor retunes it.
- Reuse the `color()` macro (converts a HubSpot color field to `rgba()` and defaults opacity to `1` when unset — avoids the `rgba(x,y,z,)` output bug). It must be `{% import %}`ed into every stylesheet that uses it; imports don't cascade through `main.css`.

```html
<!-- templates/layouts/base.html — note the ../../ paths: this file is two levels below theme root -->
<!DOCTYPE html>
<html lang="{{ html_lang }}">
  <head>
    {% include "../../css/theme-overrides.css.html" %}
    {{ require_css(get_asset_url('../../css/main.css')) }}
    {{ standard_header_includes }}
  </head>
  <body>
    {% block body %}{% endblock %}
    {{ require_js(get_asset_url('../../js/main.js')) }}
    {{ standard_footer_includes }}
  </body>
</html>
```

**Never** copy a `{{ theme.x }}` HUBL expression into a module's CSS. Modules read the resulting CSS variable.

## 2. CSS placement

| Layer                 | File                                | What goes here                                                          |
|-----------------------|-------------------------------------|-------------------------------------------------------------------------|
| Token emitter         | `css/theme-overrides.css.html`      | `:root { --aci-… }` driven by `theme.x`, wrapped in `<style>`           |
| Global styles         | `css/main.css`                      | Reset, typography baseline, `.aci-container`, `.aci-section`, `.aci-btn`, utility classes, header/footer, table baseline, form baseline |
| Module-scoped styles  | `<module>/module.css`               | Layout for that module only; class names prefixed with module slug      |
| Inline `style="…"`    | `module.html`                       | **Only** for one-off per-instance dynamic values driven by editor input (e.g. `min-height: {{ module.min_height }}vh`). Never for theme tokens. |

**Forbidden** (this is the most-corrected rule in the OneIMS history; every theme review has caught at least one of these):
- `<style>` tags inside `module.html`
- `<script>` tags inside `module.html` — sole exception: `<script type="application/ld+json">` structured-data blocks (FAQ schema etc.)
- Hardcoded hex / px / font name inside `module.css`
- `style="background:#E74C3C"` anywhere — use a class. Inline `style="…"` is allowed **only** for one-off per-instance dynamic values driven by editor input (e.g. `min-height: {{ module.min_height }}vh`) — never for theme tokens.

Per-module CSS files (`module.css`) are HubSpot-auto-scoped, so they are safe to use freely — but still reference `var(--aci-*)` only.

## 3. JS placement

- All shared behavior lives in `<theme>/js/main.js` and is loaded once per page (via `require_js` in the base layout).
- Modules emit `data-aci-*` attributes on their root element. `main.js` delegates from `document` and dispatches based on those attributes.
- **"Add the markup; do not add init code"** (KineticGPO BUILD-CONVENTIONS): `main.js` self-initialises on markup attributes/classes (`data-animate`, `data-count`, `.faq-item`, `.marquee-track`, slider tracks, …). A new module that reuses an existing behavior ships zero JS.
- A module **may** ship `module.js` for genuinely module-internal init (e.g. a chart instance keyed by id) — but only when extracting it to `main.js` would be awkward.
- Module `module.html` files do **not** contain `<script>` tags.
- **Idempotent init, always.** The HubSpot page editor re-renders module DOM at will; any init that runs twice must be a no-op. The Meridian "Partners 2026" guard is the canonical idiom:

```js
containers.forEach(function (c) {
  if (c.dataset.aciBound) return;
  c.dataset.aciBound = '1';
  // …init once…
});
```

```js
// js/main.js (sketch)
document.addEventListener('click', (e) => {
  const accordionTrigger = e.target.closest('[data-aci-accordion-trigger]');
  if (accordionTrigger) toggleAccordion(accordionTrigger);

  const tabTrigger = e.target.closest('[data-aci-tab-trigger]');
  if (tabTrigger) switchTab(tabTrigger);

  const modalOpen = e.target.closest('[data-aci-modal-open]');
  if (modalOpen) openModal(modalOpen.dataset.aciModalOpen);
});

// Sticky header — hide on scroll-down, show on scroll-up.
// OneIMS canonical pattern (Aagard, Minitab, MyGovBill).
let _lastY = 0;
window.addEventListener('scroll', () => {
  const header = document.querySelector('.aci-header');
  if (!header) return;
  const y = window.scrollY;
  header.classList.toggle('scrolled', y > 32);
  header.classList.toggle('scrolled-down', y > _lastY && y > 80);
  _lastY = y;
}, { passive: true });
```

```css
/* main.css */
.aci-header { transition: transform .3s ease; }
.aci-header.scrolled-down { transform: translateY(-100%); }
```

## 3b. Animations — CSS-first, no GSAP/Lenis

In April 2026 the team **removed GSAP and Lenis from every KineticGPO variant and from Aagard** ("removed heavy GSAP and Lenis dependencies, converted animations and preloaders to more performant CSS animations with vanilla js triggers"). Do not reintroduce animation libraries. The replacement stack:

1. **A CSS keyframe library** (Animista-derived `animista.css`): `fade-in`, `fade-in-top/right/bottom/left`, `scale-in-*`, `slide-in-blurred-*` — each keyframe mapped to a class of the same name.
2. **The markup contract**: `data-animate="fade-in-bottom" data-animate-delay="350"`. Stagger in loops: `data-animate-delay="{{ 200 + loop.index0 * 150 }}"`.
3. **One IntersectionObserver** in `main.js` (`threshold: 0.15`); elements already in the viewport at load trigger immediately instead of being observed:

```js
const animateObserver = new IntersectionObserver((entries) => {
  entries.forEach((entry) => {
    if (!entry.isIntersecting) return;
    const el = entry.target;
    const delay = el.getAttribute('data-animate-delay');
    if (delay) el.style.animationDelay = delay + 'ms';
    el.classList.remove('is-pending');
    el.classList.add('animated', el.getAttribute('data-animate'));
    el.addEventListener('animationend', () => {
      el.classList.remove('animated', el.getAttribute('data-animate'));
      el.classList.add('animation-complete');   // frees :hover transitions
      el.style.animationDelay = '';
    }, { once: true });
    animateObserver.unobserve(el);
  });
}, { threshold: 0.15 });
```

4. **Editor-safety inversion (the load-bearing detail):** the default CSS state is **VISIBLE**. JS adds `.is-pending` (hidden) only to nodes present at initial mount — so content is never stuck at `opacity: 0` without JS, in the HubSpot editor, or when the editor re-injects module DOM:

```js
document.querySelectorAll('[data-animate]:not(.animated):not(.animation-complete)')
  .forEach((el) => el.classList.add('is-pending'));
```

5. **Belt-and-braces editor guard** in the token emitter:

```hubl
{% if is_in_editor %}
  [data-animate], [data-reveal], .is-pending {
    opacity: 1 !important; transform: none !important;
    animation: none !important; transition: none !important;
  }
{% endif %}
```

6. Scroll handlers are `requestAnimationFrame`-throttled with `{ passive: true }`; autoplaying sliders/marquees respect `prefers-reduced-motion` and pause on touch/hover.
7. Preloaders (if the brief demands one) are CSS-only with a JS progress gate — never a GSAP timeline. Hide them entirely under `is_in_editor`.

If a client explicitly requires GSAP, load it from `theme.json` `js_assets` with explicit `priority` ordering (the Ondex pattern) — never per-module `<script>` tags.

## 3c. The section shell — standard `spacing` + `background` groups

KineticGPO standardized two groups injected into nearly every section-level module (28 and 25 modules respectively). Ship these on every new section module:

```jsonc
{ "type": "group", "name": "spacing", "label": "Section spacing",
  "children": [
    { "type": "number", "name": "pad_top",    "label": "Padding top",    "suffix": "rem", "min": 0, "step": 0.5, "default": 6 },
    { "type": "number", "name": "pad_bottom", "label": "Padding bottom", "suffix": "rem", "min": 0, "step": 0.5, "default": 6 } ] },
{ "type": "group", "name": "background", "label": "Section background",
  "children": [
    { "type": "choice", "name": "bg_type", "label": "Background type",
      "choices": [["none","None"],["color","Color"],["gradient","Gradient"],["image","Image"],["video","Video"]],
      "default": "none" },
    { "type": "color", "name": "bg_color", "label": "Color",
      "visibility": { "controlling_field": null, "controlling_field_path": "background.bg_type",
                      "controlling_value_regex": "color", "operator": "MATCHES_REGEX" } },
    { "type": "image", "name": "bg_image", "label": "Image",
      "visibility": { "controlling_field": null, "controlling_field_path": "background.bg_type",
                      "controlling_value_regex": "image|video", "operator": "MATCHES_REGEX" } }
    /* + gradient pair, video file, overlay color/opacity — same visibility idiom */ ] }
```

Note the **`controlling_field_path`** (dotted path) form — this is what resolves group-child siblings reliably; plain `controlling_field` is the one that breaks in nested repeaters. Render with one shared ~10-line HubL preamble at the top of `module.html` producing `class="has-section-bg"` + inline padding style + optional `.section-bg-media` / `.section-bg-overlay` divs — identical across modules so editors get one mental model.

Also standard on section-level modules: an optional `anchor_id` text field for deep links:

```hubl
<section {% if module.anchor_id %}id="{{ module.anchor_id|escape_attr }}"{% else %}id="aci-{{ name }}"{% endif %} class="…">
```

## 4. Buttons are global

- Two button styles: Primary, Secondary. Defined once in `fields.json` (Primary Button, Secondary Button groups).
- One CSS rule pair in `main.css` consumes the variables:

```css
.aci-btn { display:inline-flex; align-items:center; justify-content:center; gap:.5rem;
           cursor:pointer; text-decoration:none; transition:all .25s ease;
           font-family:var(--aci-font-primary); font-weight:600; line-height:1;
           border-style: solid; }
.aci-btn--primary {
  background: var(--aci-btn-primary-bg);
  color: var(--aci-btn-primary-text);
  border: var(--aci-btn-primary-bw) solid var(--aci-btn-primary-border);
  border-radius: var(--aci-btn-primary-radius);
  padding: var(--aci-btn-primary-py) var(--aci-btn-primary-px);
}
.aci-btn--primary:hover {
  background: var(--aci-btn-primary-bg-h);
  color: var(--aci-btn-primary-text-h);
}
.aci-btn--secondary {
  background: var(--aci-btn-secondary-bg);
  color: var(--aci-btn-secondary-text);
  border: var(--aci-btn-secondary-bw) solid var(--aci-btn-secondary-border);
  border-radius: var(--aci-btn-secondary-radius);
  padding: var(--aci-btn-secondary-py) var(--aci-btn-secondary-px);
}
.aci-btn--secondary:hover {
  background: var(--aci-btn-secondary-bg-h);
  color: var(--aci-btn-secondary-text-h);
}
```

- Every module CTA exposes `style: choice [primary, secondary]`.
- Module HTML renders: `<a class="aci-btn aci-btn--{{ btn.style }}" href="{{ btn.url.href }}">{{ btn.label }}</a>`.
- **No module redefines `.aci-btn--primary` / `.aci-btn--secondary`** or ships its own `.my-module-btn { … }` styling.

### Class naming is per-project — adopt, never mix

`.aci-btn--*` and the `aci-` prefix are this skill's **greenfield defaults**, not a universal law. Production themes each carry their own established family — Aagard: `.btn .btn-red/-outline/-white/-dark` (and explicitly bans `.aci-btn--*`); KineticGPO: `.btn-primary/-secondary/-outline/-outline-white`; Ondex: `.btn` + `{{ module.button_style|default('btn-primary') }}`; Minitab: flat `.blue_solid`-family + pill `.mt-radius-button .mt-btn-*`. The rule that is universal: **one global button family, defined once, chosen per-CTA via a `choice` field — and when extending an existing theme you adopt ITS family verbatim. Never introduce a second button system.**

Two collision warnings from production:
- If a theme has a global `[class^="btn-"]` rule, never name an unrelated helper class `btn-*` — it will be styled as a button (KineticGPO BUILD-CONVENTIONS).
- On greenfield builds, replace the `aci-` prefix with the client slug (`kgpo-`, `mt-`, …) so two themes never collide in a shared portal.

## 5. Mobile rules

- All multi-column layouts collapse at **767px** (universal OneIMS convention — matches HubSpot's mobile preview width). CSS:

```css
@media (max-width: 767px) {
  .aci-two-col, .aci-split-5050, .aci-cards { grid-template-columns: 1fr; }
  .aci-two-col[data-media] .aci-two-col__media { order: -1; }   /* media on top */
  .aci-hero { min-height: 60vh; }
  h1 { font-size: calc(var(--aci-h1-size) * 0.65); }
  h2 { font-size: calc(var(--aci-h2-size) * 0.70); }
}
```

- Hero text scales down on mobile via `clamp()` or `calc()` against the `--aci-h1-size` variable.
- **Mobile nav**: the logo always remains a working `<a href="/">` even when the menu drawer is open. Don't disable or swap the link in the menu-open state.
- **Horizontal scroll**: if a module uses off-screen animation or wide images, ensure the page wrapper clips overflow: `body { overflow-x: hidden; }` or `<main style="overflow-x:hidden; width:100%; max-width:100%">`.

## 6. Container width — global setting, not per-module px

Themes ship a global `layout_structure.container.max_width` (default `1300`). The token becomes `--aci-container-max`. All modules read it:

```css
.aci-container { max-width: var(--aci-container-max); margin-inline: auto; padding-inline: var(--aci-container-px); }
```

When an editor asks to change site-wide width, they change one setting — every module updates. Never hardcode `max-width: 1300px` inside a module. (This is one of the most-repeated corrections on Aagard.)

## 7. Field naming

- All `fields.json` field names: `snake_case`.
- All HTML class names: `kebab-case`, prefixed `aci-`. BEM-ish: `.aci-block__element--modifier`.
- All CSS variables: `--aci-…` — same prefix.
- HUBL local vars: `_lower_snake` (leading underscore optional).
- **Avoid HubSpot-reserved field `name` values**: `label`, `name`, `body`, `type`, `id`, `css`, `styles`, `child_css`, `overrideable`. Use a prefix (`aci_label`, `aci_body`, `aci_title`) or an alternative (`heading_text`, `body_content`). Full list and fixes in [upload-validation.md](upload-validation.md).

## 8. Accessibility baseline

- Every interactive element keyboard-focusable (`a`, `button`, `[role="button"][tabindex="0"]`).
- Visible focus states: `:focus-visible { outline: 2px solid var(--aci-primary); outline-offset: 2px; }`.
- Every image module exposes `alt`. Decorative images: instruct editor to leave alt blank.
- **Body text default color is off-black**, never `#000000`. Use `#111111` / `#2F3437` / `#333333`. Line-height ≥ 1.6.
- Color contrast at default values: text vs. background ≥ 4.5:1; large text ≥ 3:1.
- Form fields: `<label>` always present; `aria-describedby` to help text; error states use `--aci-danger`.

## 9. Performance baseline

- Lazy-load all images below the fold (`loading="lazy"` on `<img>`).
- Defer all non-critical scripts (`defer` on `<script>` tags). `main.js` is deferred.
- Preconnect to Google Fonts: `<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>` in base layout.
- Inline critical CSS (the `theme-overrides.css.html` block) — keeps tokens render-blocking-free since they're already inline.
- No third-party scripts beyond what HubSpot's `standard_header_includes` brings in, unless the user explicitly asks (analytics, chat widgets, Chart.js for the chart module).
- Optional `favicon.svg` with embedded CSS for light/dark mode (single file, no `.ico` second copy):

```html
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 32 32">
  <style>
    .fg { fill: #111111; }
    @media (prefers-color-scheme: dark) { .fg { fill: #FFFFFF; } }
  </style>
  <path class="fg" d="…"/>
</svg>
```

## 10. Editor experience

- Group fields logically. Use `expanded: true` only on the most commonly edited group (typically Global Colors or Layout & Structure).
- `help_text` on any non-obvious field (especially video IDs, JSON inputs, raw URLs).
- `inline_help_text` for a one-line nudge under the label.
- `default` values render a usable on-brand layout if the editor changes nothing.
- `visibility` rules hide irrelevant fields based on type pickers (e.g. only show `video_id` when `background_type == video` AND `video_source != hubspot_file`).
- For color or styling fields inside a repeater, put the field **inside** the repeater item — not at group level — so each row can differ.
- For content-heavy repeaters (testimonials, kiosk models, case studies), add an optional `item_id` text field labelled "Internal ID (optional)".
- For icon selection, ship a `choice` between `icon_library` and `custom_svg`, with conditional `icon` picker and `file` upload.
- When a module update makes the module disappear from the editor preview (HubSpot caches), refresh the page and wait 30–60s for the indexer. This is a HubSpot quirk, not a bug in the module.
- **Repeater rows must show meaningful names** in the editor sidebar: set `occurrence.sorting_label_field` to the item's title-ish child field. Ship a populated top-level `"default": [ {...}, {...} ]` array on every repeater so the module renders complete out of the box.
- **Label fields by role, not just name** (KineticGPO convention — the best editor-UX idea in the catalog): `"Red — primary CTA / accent"`, `"Slate — secondary text / meta"`, `"3XL (H2 / Section Titles)"`. Add group-level `help_text` explaining what the group feeds.
- **Zero mandatory fields.** Never set `required: true`; give defaults and guard empty states in the template.

## 10b. Render safety — three HubSpot quirks that corrupt output

- **Rich text renders inside `<div>`, never `<p>`.** HubSpot richtext output already emits its own `<p>` tags; wrapping the field in a `<p>` produces stray empty paragraphs (Aagard fix).
- **`rgba()` backgrounds need a hex fallback declared first** — HubSpot sometimes strips alpha channels; `background: #0B1D3A; background: rgba(11,29,58,.85);` survives, the lone rgba doesn't (Aagard fix).
- **`content.widgets.<module>.body.<field>` only contains fields the editor actually touched.** Every feed/listing read of widget data needs `|default(...)` or it blows up on untouched instances (KineticGPO fix).

## 10c. Sections (`sections/*.html`) are standard, not optional

Meridian ships 15, Aagard 17, KineticGPO 3. A section file is a **bare `{% dnd_section %}` with no wrapping `dnd_area`**, annotated:

```html
<!--
  templateType: section
  label: Hero Carousel
  isAvailableForNewContent: true
  screenshotPath: ../images/section-previews/hero-carousel-preview.png
  description: "Section with a hero carousel"
-->
{% dnd_section full_width=true, vertical_alignment='MIDDLE' %}
  {% dnd_module path='../modules/hero-carousel.module' %}{% end_dnd_module %}
{% end_dnd_section %}
```

Ship a `images/section-previews/` screenshot per section — sections without previews look broken in the editor picker. `dnd_section` accepts only `background_color`, `background_image`, `background_linear_gradient`, `margin`, `padding`, `vertical_alignment`, `full_width`, `max_width`.

## 11. Blog templates — patterns the team has corrected

- **Blog search form** — submits to `/hs-search-results` with `method="get"`. Query param `name="term"` (not `q`). Hidden `<input type="hidden" name="type" value="BLOG_POST">` and optionally `LISTING_PAGE`. Do not include `SITE_PAGE` for blog-only search.
- **Featured + grid sections under search** — when search filter is active, flatten the featured layout (2fr/1fr) to the same grid (e.g. 3-col) as the rest of the listing. Toggle via a `.is-searching` class set by the form, not visibility rules.
- **Date display** — date only (no time). Right-aligned, consistent across featured and grid. `{{ content.publish_date|format_date('long') }}` or `{{ content.publish_date|formatdate('%B %d, %Y') }}`.
- **Pagination orphans** — avoid layouts that leave a single orphaned card on the last page. Adjust per-page count or hide the featured slot on page 2+ so all paginated pages have a uniform grid.
- **Video featured image** — HubSpot blog doesn't support video as featured image. Generate a still frame from the video and use that.
- **Related posts** — use `blog_recent_posts(group, limit)`. `related_blog_posts(...)` does NOT exist. For truly related posts on a post page, filter recent posts by the current post's tag.

## 12. Non-destructive policy

If extending an existing theme:
- Do **not** rename existing modules. Add new ones, even if a similar one exists with worse architecture.
- Do **not** delete or rename existing `fields.json` keys — page editors reference them; pages will break.
- New theme tokens may be **added** to `fields.json` freely; just don't remove old keys.
- New modules may shadow the look of older ones — page editors keep using the older modules until manually migrated.
- The agent surfaces a "migration notes" section at the end if it sees old modules that should eventually be retired, but does **not** auto-migrate.

## 13. Dual-path mirroring (multi-environment themes)

Some clients (e.g. Minitab) have parallel theme folders — one live (production HubSpot upload), one test (theme previewer). The agent must mirror every change to both folders unless the user explicitly says "only the live one" or "only the test one". Preserve directory structure across both paths.

## 14. Build order — phased, brand-first

The agent works in three phases. Phase boundaries are **hard stops** — even in auto mode the agent pauses and asks.

### Phase 0 — Brand intake (no files written)

Run the brand intake described in [SKILL.md](../SKILL.md) Phase 0. Collect: theme identity, colors (primary/secondary/tertiary + status + body text + backgrounds), typography (fonts, weights, sizes, line-height, eyebrow style, link colors), buttons (primary + secondary + optional tertiary, all hover variants), layout (max width, padding, section spacing, mobile breakpoint), assets (logo light/dark, favicon, brand patterns), forms/tables/blog overrides, motion preferences, constraints. Echo back what was captured, flag every default applied because the user left a field blank, and **wait for explicit confirmation**.

### Phase 1 — Theme Settings foundation (ship, then STOP)

Build only the brand foundation and pause for review:

1. `theme.json` — client name/label/version/author/screenshot path; `responsive_breakpoints: [{ "name": "mobile", "mediaQuery": "(max-width: 767px)" }]`
2. `fields.json` — every group's default values are the **client's** values from Phase 0, not skill defaults
3. `css/theme-overrides.css.html` — `<style>:root { --aci-…: {{ theme.x }} }</style>` token emitter wired to every fields.json group
4. `css/main.css` — typography baseline, `.aci-container`, `.aci-section`, `.aci-btn--primary` / `.aci-btn--secondary` (token-driven), `.aci-header` skeleton, form baseline, table baseline, utilities
5. `js/main.js` — event-delegation skeleton (header scroll-direction toggle, `data-aci-*` click delegator)
6. `templates/layouts/base.html`
7. `templates/partials/header.html`, `partials/footer.html`
8. `images/theme-thumb.png` placeholder; optional `favicon.svg` with `@media (prefers-color-scheme: dark)`

**STOP. Print the file tree. Echo the brand tokens that landed in `fields.json`, flag any defaults applied. Ask the user to upload to HubSpot and verify the brand renders correctly before proceeding.** This stop is blocking even under auto/yolo mode.

### Phase 2 — Modules + templates (after foundation approval)

9. Modules in batches: hero & content → navigation → interactive → forms → data display → media → feedback
10. Templates: `home.html`, `landing-page.html`, `system/*`, `blog/blog-listing.html`, `blog/blog-post.html`
11. Optional `sections/*.html` if the brief implies drag-drop sections
12. Final validation pass against the checklist in [SKILL.md](../SKILL.md)
