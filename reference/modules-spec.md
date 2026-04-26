# Modules — exact field spec for every module to scaffold

Each module lives at `<theme>/modules/<module-name>.module/` with these five files:

```
<module-name>.module/
├── module.html      ← HUBL render — uses theme CSS vars, no inline <style>
├── module.css       ← per-module styles, references --aci-* vars only
├── module.js        ← optional; prefer global handler in <theme>/js/main.js
├── fields.json      ← field definitions
└── meta.json        ← label, host_template_types, tags, help_text
```

Field types referenced below: `text`, `richtext`, `image`, `file`, `url`, `color`, `font`, `number`, `boolean`, `choice`, `alignment`, `icon`, `form`, `menu`, `link`, `cta`, `simplemenu`, `gradient`, `group`, repeater via `occurrence`, `visibility` for conditional fields.

Every CTA in every module exposes `style: choice [primary, secondary]` and renders `.aci-btn.aci-btn--{style}`. **No module redefines button styles.**

---

## Hero / Background

### `hero-banner.module`

```jsonc
[
  { "type": "choice", "name": "background_type", "label": "Background type", "display": "select",
    "choices": [["image","Image"],["video","Video"],["color","Solid color"],["gradient","Gradient"]],
    "default": "color" },

  { "type": "image", "name": "background_image", "label": "Background image",
    "responsive": true, "resizable": true,
    "visibility": { "controlling_field": "background_type", "controlling_value_regex": "image", "operator": "EQUAL" } },

  { "type": "choice", "name": "video_source", "label": "Video source",
    "choices": [["hubspot_file","HubSpot file"],["youtube_id","YouTube ID"],["vimeo_id","Vimeo ID"],["wistia_id","Wistia ID"]],
    "default": "youtube_id",
    "visibility": { "controlling_field": "background_type", "controlling_value_regex": "video", "operator": "EQUAL" } },
  { "type": "file", "name": "video_file", "label": "Video file", "picker": "file",
    "visibility": { "boolean_operator": "AND", "criteria": [
      { "controlling_field": "background_type", "controlling_value_regex": "video", "operator": "EQUAL" },
      { "controlling_field": "video_source",    "controlling_value_regex": "hubspot_file", "operator": "EQUAL" } ] } },
  { "type": "text", "name": "video_id", "label": "Video ID",
    "help_text": "Paste only the ID (e.g. dQw4w9WgXcQ).",
    "visibility": { "boolean_operator": "AND", "criteria": [
      { "controlling_field": "background_type", "controlling_value_regex": "video", "operator": "EQUAL" },
      { "controlling_field": "video_source",    "controlling_value_regex": "(youtube_id|vimeo_id|wistia_id)", "operator": "MATCHES_REGEX" } ] } },

  { "type": "color", "name": "background_color", "label": "Background color",
    "default": { "color": "#201600", "opacity": 100 },
    "visibility": { "controlling_field": "background_type", "controlling_value_regex": "color", "operator": "EQUAL" } },

  { "type": "group", "name": "gradient", "label": "Gradient",
    "visibility": { "controlling_field": "background_type", "controlling_value_regex": "gradient", "operator": "EQUAL" },
    "children": [
      { "type": "color",  "name": "start", "label": "Start", "default": { "color": "#E74C3C", "opacity": 100 } },
      { "type": "color",  "name": "end",   "label": "End",   "default": { "color": "#0D899E", "opacity": 100 } },
      { "type": "number", "name": "angle", "label": "Angle (deg)", "default": 135, "min": 0, "max": 360 } ] },

  { "type": "boolean", "name": "overlay_enabled", "label": "Enable overlay", "display": "toggle", "default": true,
    "visibility": { "controlling_field": "background_type", "controlling_value_regex": "(image|video)", "operator": "MATCHES_REGEX" } },
  { "type": "color", "name": "overlay_color", "label": "Overlay color",
    "default": { "color": "#000000", "opacity": 50 },
    "visibility": { "controlling_field": "overlay_enabled", "controlling_value_regex": "true", "operator": "EQUAL" } },

  { "type": "text",     "name": "eyebrow",            "label": "Eyebrow", "default": "" },
  { "type": "richtext", "name": "title",              "label": "Title",   "default": "<h1>Headline</h1>" },
  { "type": "color",    "name": "title_color",        "label": "Title color",       "default": { "color": "#FFFFFF", "opacity": 100 } },
  { "type": "richtext", "name": "description",        "label": "Description",       "default": "<p>Supporting copy.</p>" },
  { "type": "color",    "name": "description_color",  "label": "Description color", "default": { "color": "#FFFFFF", "opacity": 90 } },

  { "type": "group", "name": "buttons", "label": "Buttons",
    "occurrence": { "min": 0, "max": 2, "default": 2 },
    "children": [
      { "type": "text",    "name": "label",          "label": "Label",   "default": "Get Started" },
      { "type": "url",     "name": "url",            "label": "URL" },
      { "type": "choice",  "name": "style",          "label": "Style",
        "choices": [["primary","Primary"],["secondary","Secondary"]], "default": "primary" },
      { "type": "boolean", "name": "open_in_new_tab","label": "Open in new tab", "default": false } ] },

  { "type": "alignment", "name": "content_align", "label": "Content alignment",
    "default": { "horizontal_align": "CENTER", "vertical_align": "MIDDLE" } },
  { "type": "number", "name": "min_height_vh", "label": "Min height (vh)", "default": 80, "min": 30, "max": 100 }
]
```

### `background-image-section.module`
Image (or color/gradient fallback), parallax toggle, overlay (color + opacity), inner rich-text slot.

---

## Content components

### `two-column-section.module`
- `media_position`: choice `left | right` (mobile always media-on-top)
- `media_type`: choice `image | video`
- `media_image`: image (visible if image)
- `media_video_source`: choice `youtube_id | hubspot_file | vimeo_id | wistia_id` (visible if video)
- `media_video_value`: text ID, or `media_video_file`: file (visible per source)
- Right column group: `tag` (text), `title` (richtext + color override), `subtitle` (richtext + color override), `body` (richtext + color override), optional `button { label, url, style }`.

### `cards.module`
- `columns`: choice `2 | 3 | 4` (default 3)
- `cards`: repeater `{ icon (icon picker), image (optional), title (text), description (richtext), link_url (url), link_label (text), button { label, url, style } (optional) }`

### `feature-blocks.module`
- `layout`: choice `alternating | grid`
- `blocks`: repeater `{ icon, headline, body (richtext), link }`

### `content-block.module`
- `eyebrow` (text), `title` (richtext), `body` (richtext), `align`: choice left/center/right.

### `media-object.module`
- `media_position`: left/right, `image`, `body` (richtext), `link`.

### `article-block.module`
- `source`: choice `blog | hubdb`, `blog_id` or `hubdb_table_id`, `count` (number), `columns` (2/3/4).

### `testimonials-grid.module`
- `columns` (2/3/4)
- `items`: repeater `{ quote (richtext), name, role, company, avatar (image), rating (number 0–5) }`

### `testimonials-slider.module`
Same `items` shape; slider settings: `autoplay` (bool), `interval` (number ms), `show_dots` (bool), `show_arrows` (bool).

### `pricing-tables.module`
- `columns` (2/3/4)
- `tiers`: repeater `{ name, price, period, features (richtext), highlight (boolean), button { label, url, style } }`

### `stats-counter.module`
- `columns` (2/3/4)
- `items`: repeater `{ value (number), prefix (text), suffix (text), label (text), icon (icon) }`
- `animate_on_scroll` (boolean, default true)

---

## Services-page set

### `services-overview-tabs.module`
- `tabs`: repeater `{ tab_icon (icon), tab_label (text), image (image), headline (richtext), body (richtext), button { label, url, style } (optional) }`
- `default_tab`: number

### `process-accordion.module`
- `intro` (richtext, optional)
- `steps`: repeater `{ title (text), content (richtext) }` — OR a single `content` richtext if the editor wants to author all 12 steps as one block (toggle field).

### `credentials.module`
- `columns` (default 4)
- `items`: repeater `{ icon_image (image), label (text), link (url, optional) }`

### `why-choose.module`
- `columns` (default 4 desktop, 2 tablet)
- `items`: repeater `{ icon (icon), title (text), description (richtext) }` — default 8 items.

### `projects-gallery.module`
- `columns` (2/3/4)
- `items`: repeater `{ image, title, category, link }`

### `customers-logos.module`
- `layout`: choice `grid | marquee`
- `logos`: repeater `{ logo_image (image), link (url, optional), alt (text) }`

### `contact-form.module`
- `layout`: choice `centered | split-with-image`
- `form_id`: form picker
- `heading` (text), `subheading` (richtext, optional), `intro` (richtext, optional)
- `image` (image, visible only when `layout == split-with-image`)

---

## Navigation

### `navbar.module`
- `logo` (image), `logo_link` (url)
- `primary_menu`: menu picker
- `secondary_menu`: menu picker (optional)
- `cta_button`: group `{ label, url, style }`
- `sticky` (boolean, default true), `scroll_shrink` (boolean, default true)

### `mega-menu.module`
- `columns`: repeater `{ heading (text), links (repeater of { label, url }), featured_card { image, title, link } (optional) }`

### `breadcrumbs.module`
- `separator`: choice `/ | › | • | →`
- `show_home` (boolean)

### `pagination.module`
- `style`: choice `numbered | prev-next | load-more`

### `sidebar-nav.module`
- `menu`: menu picker
- `heading` (text, optional)

### `footer-nav.module`
- `columns`: repeater `{ heading (text), links (repeater of { label, url }) }`

---

## Interactive

### `button.module`
Single CTA. Fields: `label`, `url`, `style` (primary/secondary), `icon` (optional, icon picker), `open_in_new_tab`.

### `accordion.module`
- `allow_multiple_open` (boolean, default false)
- `items`: repeater `{ title (text), content (richtext), default_open (boolean) }`

### `tabs.module`
- `orientation`: choice `horizontal | vertical`
- `default_tab` (number)
- `items`: repeater `{ label (text), icon (icon, optional), content (richtext) }`

### `modal.module`
- `trigger_button`: group `{ label, url (optional), style }`
- `dialog`: group `{ heading (text), body (richtext), form_id (form picker, optional) }`
- `dismiss_on_overlay` (boolean, default true)

### `dropdown.module`
- `trigger_label` (text)
- `items`: repeater `{ label, url }`

### `tooltip.module`
- `trigger_text` (text), `tooltip_content` (text)
- `position`: choice `top | right | bottom | left`

### `slider.module` / `carousel.module`
- `items`: repeater `{ image, headline (richtext), body (richtext), button { label, url, style } }`
- `autoplay` (boolean), `interval` (number), `loop` (boolean), `show_arrows` (boolean), `show_dots` (boolean)

### `toggle-switch.module`
- `label` (text), `default_on` (boolean), `on_label` / `off_label` (text)
- For form-driven pages; emits `data-aci-toggle`.

---

## Forms & inputs (primitives)

Each consumes the **Forms** theme tokens.

### `input-field.module`
- `type`: choice `text | email | password | tel | url | number`
- `name` (text, required), `label` (text), `placeholder` (text), `required` (boolean), `help_text` (text)

### `textarea.module`
- `name`, `label`, `placeholder`, `rows` (number, default 4), `required`, `help_text`

### `select.module`
- `name`, `label`, `required`, `options`: repeater `{ label, value }`

### `checkbox.module`
- `name`, `label`, `default_checked` (boolean)

### `radio-group.module`
- `name`, `label`, `options`: repeater `{ label, value }`

### `search-bar.module`
- `placeholder` (text), `submit_label` (text), `action_url` (url)

### `form-group.module`
- `legend` (text), child slot for inputs (drag-drop). On HubSpot, render via column group.

### `file-upload.module`
- `name`, `label`, `accept` (text, e.g. `.pdf,.jpg`), `multiple` (boolean), `help_text`

---

## Data display

### `table.module`
Custom HTML table builder. Fields:
- `headers`: repeater `{ label }`
- `rows`: repeater `{ cells: repeater { value (text or richtext) } }`
- `show_footer` (boolean), `footer`: repeater `{ label }` (visible if true)
- Renders with `.aci-table` class consuming Tables tokens.

### `list.module`
- `style`: choice `unordered | ordered | check | arrow`
- `items`: repeater `{ content (richtext) }`

### `badges.module`
- `items`: repeater `{ label (text), color_token: choice [primary, secondary, tertiary, success, warning, danger, info] }`

### `progress.module`
- `label` (text), `value` (number), `max` (number, default 100), `color_token`

### `chart.module`
- `chart_type`: choice `bar | line | pie | doughnut | radar`
- `title` (text), `data_json` (text — JSON of `{ labels: [], datasets: [] }`)
- Loads Chart.js from CDN once via `main.js`.

### `timeline.module`
- `items`: repeater `{ date (text), title (text), body (richtext), icon (icon) }`
- `orientation`: choice `vertical | horizontal`

### `tags.module`
- `items`: repeater `{ label, link (url, optional) }`

---

## Media

### `image.module`
- `image`, `alt`, `link` (url, optional), `lightbox` (boolean), `caption` (richtext, optional).

### `image-gallery.module`
- `layout`: choice `masonry | grid | slider`
- `columns` (2/3/4) for grid
- `items`: repeater `{ image, alt, caption }`

### `video-embed.module`
- `source`: choice `youtube_id | vimeo_id | wistia_id | hubspot_file`
- `value` (text id) or `file` (file picker), conditional via visibility
- `aspect_ratio`: choice `16:9 | 4:3 | 1:1 | 21:9`
- `autoplay` (boolean), `muted` (boolean), `loop` (boolean), `controls` (boolean)

### `icon-set.module`
- `columns` (2/3/4/6)
- `items`: repeater `{ icon (icon), label (text, optional), link (url, optional) }`

---

## Feedback / utility

### `loader.module`
- `variant`: choice `spinner | dots | bar`
- `color_token`: choice [primary, secondary, tertiary]

### `alert.module`
- `variant`: choice `info | success | warning | danger`
- `title` (text), `body` (richtext), `dismissible` (boolean)

### `toast.module`
- `default_message` (text), `variant`, `position`: choice `top-right | top-left | bottom-right | bottom-left`
- Triggered via `window.aciToast({ message, variant })` from `main.js`.

### `divider.module`
- `orientation`: choice `horizontal | vertical`
- `color_token`, `thickness` (number px), `spacing` (number px)

### `spacer.module`
- `axis`: choice `vertical | horizontal`
- `size` (number px), `mobile_size` (number px, optional)

### `overlay.module`
- Full-screen overlay block. `color` (color), `opacity` (number 0-100), `blur` (number px), child rich text.

---

## Encouraged extras (build only if industry/brief implies)

- `faq-schema.module` — accordion + JSON-LD output
- `comparison-table.module` — multi-column feature comparison
- `lead-magnet-bar.module` — sticky bottom bar with form
- `newsletter-signup.module` — email + submit, wraps a HubSpot form
- `map-embed.module` — Google Maps iframe with address field
- `social-share.module` — share buttons for current URL
- `author-bio.module` — for blog post template
- `related-posts.module` — for blog post template

---

## Module rendering checklist (apply to every module)

- [ ] `module.html` has zero `<style>` and zero `<script>` tags.
- [ ] `module.css` references only `var(--aci-*)` for colors/fonts/spacing.
- [ ] All buttons render `.aci-btn.aci-btn--{primary|secondary}`; no module-local button styling.
- [ ] All conditional fields use `visibility` rules with proper operators.
- [ ] `meta.json` has `host_template_types: ["PAGE", "BLOG_POST", "BLOG_LISTING"]` unless module is template-specific.
- [ ] `meta.json` `is_available_for_new_content: true`.
- [ ] All images have alt fields.
- [ ] All interactive modules emit `data-aci-*` attributes; behavior lives in `main.js`.
- [ ] Mobile breakpoint at 768px or below stacks all multi-column layouts; media-on-top in any media+content split.
