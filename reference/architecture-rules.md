# Architecture rules

Read these every time you build or extend the theme. Every module ships under these rules.

## 1. Tokens are the only source of truth

- Brand decisions (color, font, spacing, radius, padding) live in `fields.json` at the theme root.
- `<theme>/css/theme-overrides.css.html` is the **only** place those tokens become CSS.
- It emits one `<style>` block with `:root { --aci-…: {{ theme.x }}; }` — see [theme-fields-spec.md](theme-fields-spec.md) for the full variable map.
- `theme-overrides.css.html` is included from the theme's base layout (`templates/layouts/base.html`) inside `<head>` so every page picks it up.

```html
<!-- templates/layouts/base.html -->
<!DOCTYPE html>
<html lang="{{ html_lang }}">
  <head>
    {% include "../theme-overrides.css.html" %}
    {{ require_css(get_asset_url('../css/main.css')) }}
    {{ standard_header_includes }}
  </head>
  <body>
    {% block body %}{% endblock %}
    {{ require_js(get_asset_url('../js/main.js')) }}
    {{ standard_footer_includes }}
  </body>
</html>
```

**Never** copy a `{{ theme.x }}` HUBL expression into a module's CSS. Modules read the resulting CSS variable.

## 2. CSS placement

| Layer                 | File                                | What goes here                                                          |
|-----------------------|-------------------------------------|-------------------------------------------------------------------------|
| Token emitter         | `css/theme-overrides.css.html`      | `:root { --aci-… }` driven by `theme.x`                                  |
| Global styles         | `css/main.css`                      | Reset, typography baseline, `.aci-container`, `.aci-section`, `.aci-btn`, utility classes, header/footer, table baseline, form baseline |
| Module-scoped styles  | `<module>/module.css`               | Layout for that module only; class names prefixed with module slug      |
| Inline `style="…"`    | `module.html`                       | **Only** for one-off per-instance dynamic values (e.g. `min-height: {{ module.min_height }}vh`). Never for theme tokens. |

**Forbidden**:
- `<style>` tags inside `module.html`
- Hardcoded hex / px / font name in `module.css`
- `style="background:#E74C3C"` anywhere — use `style="background: var(--aci-primary)"` *if absolutely needed*, but prefer a class.

## 3. JS placement

- All shared behavior lives in `<theme>/js/main.js` and is loaded once per page.
- Modules emit `data-aci-*` attributes on their root element. `main.js` delegates from `document` and dispatches based on those attributes.
- A module **may** ship `module.js` for genuinely module-internal init (e.g. a chart instance keyed by id), but only when extracting it to `main.js` would be awkward.

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

window.addEventListener('scroll', () => {
  const header = document.querySelector('.aci-header');
  if (header) header.classList.toggle('scrolled', window.scrollY > 32);
}, { passive: true });
```

## 4. Buttons are global

- Two button styles: Primary, Secondary. Defined once in `fields.json` (Primary Button, Secondary Button groups).
- One CSS rule pair in `main.css` consumes the variables:

```css
.aci-btn { display:inline-block; cursor:pointer; text-decoration:none;
           transition:all .25s ease; font-family:var(--aci-font-primary);
           font-weight:600; line-height:1; }
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
.aci-btn--secondary { /* same shape, --aci-btn-secondary-* */ }
.aci-btn--secondary:hover { /* same */ }
```

- Every module CTA exposes `style: choice [primary, secondary]`.
- Module HTML renders: `<a class="aci-btn aci-btn--{{ btn.style }}" href="{{ btn.url.href }}">{{ btn.label }}</a>`.
- **No module redefines `.aci-btn--primary` / `--secondary`** or ships its own `.my-module-btn { … }` styling.

## 5. Mobile rules

- All multi-column layouts collapse at 768px. CSS:

```css
@media (max-width: 768px) {
  .aci-two-col, .aci-split-5050, .aci-cards { grid-template-columns: 1fr; }
  .aci-two-col[data-media] .aci-two-col__media { order: -1; }   /* media on top */
  .aci-hero { min-height: 60vh; }
  h1 { font-size: calc(var(--aci-h1-size) * 0.65); }
  h2 { font-size: calc(var(--aci-h2-size) * 0.7); }
}
```

- Hero text scales down on mobile via `clamp()` or `calc()` against the `--aci-h1-size` variable.

## 6. Field naming

- All `fields.json` field names: `snake_case`.
- All HTML class names: `kebab-case`, prefixed `aci-` (or your theme prefix). BEM-ish: `.aci-block__element--modifier`.
- All CSS variables: `--aci-…` — same prefix.
- HUBL local vars: `_lower_snake` (leading underscore optional).

## 7. Accessibility baseline

- Every interactive element keyboard-focusable (`a`, `button`, `[role="button"][tabindex="0"]`).
- Visible focus states: `:focus-visible { outline: 2px solid var(--aci-primary); outline-offset: 2px; }`.
- Every image module exposes `alt`. Decorative images: instruct editor to leave alt blank.
- Color contrast at default values: text vs. background ≥ 4.5:1; large text ≥ 3:1. The default token set in [theme-fields-spec.md](theme-fields-spec.md) meets this.
- Form fields: `label` always present; `aria-describedby` to help text; error states use `--aci-danger`.

## 8. Performance baseline

- Lazy-load all images below the fold (`loading="lazy"` on `<img>`).
- Defer all non-critical scripts (`defer` on `<script>` tags). `main.js` is deferred.
- Preconnect to Google Fonts: `<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>` in base layout.
- Inline critical CSS (the `theme-overrides.css.html` block) — keeps tokens render-blocking-free since they're already inline.
- No third-party scripts beyond what HubSpot's `standard_header_includes` brings in, unless the user explicitly asks (analytics, chat widgets, Chart.js for the chart module).

## 9. Editor experience

- Fields grouped logically. Use `expanded: true` only on the most commonly edited group.
- `help_text` on any non-obvious field (especially video IDs).
- `default` values render a usable on-brand layout if the editor changes nothing.
- `inline_help_text` for one-line guidance under the label.
- `visibility` rules hide irrelevant fields based on type pickers (e.g. only show `video_id` when `background_type == video` AND `video_source != hubspot_file`).

## 10. Non-destructive policy

If extending an existing theme:
- Do **not** rename existing modules. Add new ones, even if a similar one exists with worse architecture.
- Do **not** delete or rename existing `fields.json` keys — page editors reference them; pages will break.
- New theme tokens may be **added** to `fields.json` freely; just don't remove old keys.
- New modules may shadow the look of older ones — page editors keep using the older modules until manually migrated.
- The agent surfaces a "migration notes" section at the end if it sees old modules that should eventually be retired, but does **not** auto-migrate.

## 11. Build order

When scaffolding from scratch, the agent works in this order:

1. `theme.json`
2. `fields.json` (full panel)
3. `css/theme-overrides.css.html` (token emitter)
4. `css/main.css` (typography baseline, container, section, button, header, footer, table, form, utilities)
5. `js/main.js` (header scroll, accordion, tabs, modal, slider, toast)
6. `templates/layouts/base.html`
7. `templates/partials/header.html`, `partials/footer.html`
8. Modules in batches: hero & content → navigation → interactive → forms → data display → media → feedback
9. Templates: `home.html`, `landing-page.html`, `system/*`, `blog/*`
10. `images/theme-thumb.png` placeholder
11. Final validation pass against the checklist in [SKILL.md](../SKILL.md)
