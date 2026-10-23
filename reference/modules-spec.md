# Modules — exact field spec for every module to scaffold

Each module lives at `<theme>/modules/<module-name>.module/` with these five files:

```
<module-name>.module/
├── module.html      ← HUBL render — uses theme CSS vars, NO inline <style> or <script>
├── module.css       ← per-module styles, references --aci-* vars only (HubSpot auto-scopes it)
├── module.js        ← optional; prefer event delegation in <theme>/js/main.js
├── fields.json      ← field definitions
└── meta.json        ← label, host_template_types, content_types, tags, help_text, categories
```

Field types referenced below: `text`, `richtext`, `image`, `file`, `url`, `color`, `font`, `number`, `boolean`, `choice`, `alignment`, `icon`, `form`, `menu`, `link`, `cta`, `simplemenu`, `gradient`, `group`, repeater via `occurrence`, `visibility` for conditional fields.

**Every CTA in every module** exposes `style: choice [primary, secondary]` and renders `<a class="aci-btn aci-btn--{{ btn.style }}" ...>`. No module redefines button styles. (In an existing theme, adopt its button family instead — see [architecture-rules.md §4](architecture-rules.md).)

**Reserved field-name reminder:** never name a field `label`, `name`, `body`, `type`, `id`, `css`, `styles`, `child_css`, `overrideable` (nor `class`, `style`, `content`, `tag`) — those values are rejected on upload. Use `heading_text`, `body_content`, `kind`, etc. (See [upload-validation.md](upload-validation.md).)

**Nested-repeater visibility reminder:** if a child field's visibility depends on a sibling **inside a doubly-nested repeater**, HubSpot may reject the upload (`"no controlling_field with path 'X' exists"`). Use an in-template `{% if %}` guard instead. For group children (single nesting), use `controlling_field_path` with the dotted path. (See [hubl-gotchas.md §19](hubl-gotchas.md).)

**Section-shell reminder:** every section-level module ships the standard `spacing` + `background` groups and an optional `anchor_id` field — exact JSON in [architecture-rules.md §3c](architecture-rules.md).

**Repeater conventions (KineticGPO/Meridian standard):**
- `occurrence.sorting_label_field` set to the item's title-ish child field, so editor rows have readable names.
- A populated top-level `"default": [ {...}, {...} ]` array — the module renders complete out of the box.
- **Zero `required: true` fields** anywhere; guard empty states in the template.
- Group naming convention across the catalog: `content` = editorial copy · `misc` = per-instance layout/style knobs · `style` = the STYLE-tab group (style field types only!) · `animation` = motion picker.

**"X + X Repeater" pairing:** when a card/tile is useful standalone AND in a grid, ship two modules sharing markup — a single-item module and a repeater version (Meridian pattern: `Card With Icon` + `Card With Icon Grid`).

---

## `meta.json` template (every module)

```json
{
  "label": "<Human Label>",
  "host_template_types": ["PAGE", "BLOG_POST", "BLOG_LISTING"],
  "content_types": ["PAGE", "BLOG_POST", "BLOG_LISTING"],
  "is_available_for_new_content": true,
  "global": false,
  "categories": ["BODY_CONTENT"],
  "tags": ["custom"],
  "help_text": "<one-line guidance shown in the module library>"
}
```

- `categories` must be from the HubSpot enum: `TEXT, COMMERCE, DESIGN, FUNCTIONALITY, FORMS_AND_BUTTONS, BODY_CONTENT, MEDIA, BLOG, SOCIAL`. **`BANNER_AND_CTA` is NOT valid** — use `TEXT` or `FORMS_AND_BUTTONS`. Always UPPERCASE (lowercase values are the legacy risky form found in old Aagard modules).
- For template-specific modules (e.g. blog hero), narrow `host_template_types` and `content_types` to `["BLOG_POST"]` or `["BLOG_LISTING"]`. Modules meant for landing pages include `"LANDING_PAGE"` in `content_types`.
- **Never copy a `module_id` into a new `meta.json`** — it's portal-assigned; omit it and let HubSpot set it on first upload.
- Optional extras seen in production: `"smart_type": "NOT_SMART"`, `inline_help_text`, and `css_assets`/`js_assets` with `autoload: true` for module-scoped vendor files.

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
    "default": { "color": "#1A1A1A", "opacity": 100 },
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
  { "type": "richtext", "name": "heading_text",       "label": "Title",   "default": "<h1>Headline</h1>" },
  { "type": "color",    "name": "title_color",        "label": "Title color",       "default": { "color": "#FFFFFF", "opacity": 100 } },
  { "type": "richtext", "name": "body_content",       "label": "Description",       "default": "<p>Supporting copy.</p>" },
  { "type": "color",    "name": "description_color",  "label": "Description color", "default": { "color": "#FFFFFF", "opacity": 90 } },

  { "type": "group", "name": "buttons", "label": "Buttons",
    "occurrence": { "min": 0, "max": 2, "default": 2 },
    "children": [
      { "type": "text",    "name": "button_label",   "label": "Label",   "default": "Get Started" },
      { "type": "url",     "name": "button_url",     "label": "URL" },
      { "type": "choice",  "name": "style",          "label": "Style",
        "choices": [["primary","Primary"],["secondary","Secondary"]], "default": "primary" },
      { "type": "boolean", "name": "open_in_new_tab","label": "Open in new tab", "default": false } ] },

  { "type": "alignment", "name": "content_align", "label": "Content alignment",
    "default": { "horizontal_align": "CENTER", "vertical_align": "MIDDLE" } },
  { "type": "number", "name": "min_height_vh", "label": "Min height (vh)", "default": 80, "min": 30, "max": 100 }
]
```

> Note the use of `heading_text` / `body_content` / `button_label` / `button_url` instead of `title` / `body` / `label` / `url` — keeps clear of HubSpot's reserved-name list while staying descriptive.

### `background-image-section.module`
Image (or color/gradient fallback), parallax toggle, overlay (color + opacity), inner rich-text slot.

---

## Content components

### `two-column-section.module`
- `media_position`: choice `left | right` (mobile always media-on-top via `order: -1` in CSS)
- `media_type`: choice `image | video`
- `media_image`: image (visible if image)
- `media_video_source`: choice `youtube_id | hubspot_file | vimeo_id | wistia_id` (visible if video)
- `media_video_value`: text ID, or `media_video_file`: file (conditional per source)
- Right column group: `tag` (text), `heading_text` (richtext + color override), `subtitle` (richtext + color override), `body_content` (richtext + color override), optional `button { button_label, button_url, style }`.

### `cards.module`
- `columns`: choice `2 | 3 | 4` (default 3)
- `card_click_behavior`: choice `entire_card | button_only` (default `entire_card`) — full-card click is the OneIMS default
- `cards`: repeater `{ icon (icon picker), image (optional), heading_text (text), description (richtext), link_url (url), link_label (text), button { button_label, button_url, style } (optional), item_id (text, optional "Internal ID") }`

The render wraps each card in `<a>` when `card_click_behavior == entire_card` AND the card has a primary link, so the whole card is clickable. Otherwise the inner CTA is the trigger.

### `feature-blocks.module`
- `layout`: choice `alternating | grid`
- `blocks`: repeater `{ icon, headline, body_content (richtext), link }`

### `content-block.module`
- `eyebrow` (text), `heading_text` (richtext), `body_content` (richtext), `align`: choice left/center/right.

### `media-object.module`
- `media_position`: left/right, `image`, `body_content` (richtext), `link`.

### `article-block.module`
- `source`: choice `blog | hubdb`, `blog_id` or `hubdb_table_id`, `count` (number), `columns` (2/3/4).
- HUBL: use `blog_recent_posts(group, limit)` — NOT `related_blog_posts`.

### `testimonials-grid.module`
- `columns` (2/3/4)
- `items`: repeater `{ quote (richtext), author_name (text), role (text), company (text), avatar (image), rating (number 0–5), item_id (text, optional) }`

### `testimonials-slider.module`
Same `items` shape; slider settings: `autoplay` (bool), `interval` (number ms, default 5000), `show_dots` (bool), `show_arrows` (bool). All slides equal height; content vertically centered (see Carousel pattern below).

### `pricing-tables.module`
- `columns` (2/3/4)
- `tiers`: repeater `{ tier_name (text), price (text), period (text), features (richtext), highlight (boolean), button { button_label, button_url, style }, item_id (text, optional) }`

### `stats-counter.module`
- `columns` (2/3/4)
- `items`: repeater `{ value (number), prefix (text), suffix (text), stat_label (text), icon (icon) }`
- `animate_on_scroll` (boolean, default true)

---

## Services-page set

### `services-overview-tabs.module`
- `tabs`: repeater `{ tab_icon (icon), tab_label (text), image (image), headline (richtext), body_content (richtext), button { button_label, button_url, style } (optional) }`
- `default_tab`: number (1-indexed)

### `process-accordion.module`
- `intro` (richtext, optional)
- `single_block_mode` (boolean): if true, the editor authors all steps as one rich-text block; if false, steps are a repeater.
- `steps`: repeater `{ heading_text (text), content (richtext) }` — visible when `single_block_mode == false`.
- `single_block_content` (richtext) — visible when `single_block_mode == true`.

### `credentials.module`
- `columns` (default 4)
- `items`: repeater `{ icon_image (image), credential_label (text), link (url, optional) }`

### `why-choose.module`
- `columns` (default 4 desktop, 2 tablet)
- `items`: repeater `{ icon (icon), heading_text (text), description (richtext) }` — default 8 items.

### `projects-gallery.module`
- `columns` (2/3/4)
- `items`: repeater `{ image, project_title (text), category (text), link (url) }`

### `customers-logos.module`
- `layout`: choice `grid | marquee`
- `logos`: repeater `{ logo_image (image), link (url, optional), alt_text (text) }`

### `contact-form.module`
- `layout`: choice `centered | split-with-image`
- `form_id`: form picker
- `heading_text` (text), `subheading` (richtext, optional), `intro` (richtext, optional)
- `image` (image, visible only when `layout == split-with-image`)

---

## Navigation

### `navbar.module`
- `logo` (image), `logo_link` (url, default `/`)
- `primary_menu`: menu picker
- `secondary_menu`: menu picker (optional)
- `cta_button`: group `{ button_label, button_url, style }`
- `sticky` (boolean, default true), `scroll_shrink` (boolean, default true)
- Mobile drawer: logo MUST remain a working link to homepage even when drawer is open.

### `mega-menu.module`
- `columns`: repeater `{ heading_text (text), links (repeater of { link_label, link_url }), featured_card { image, card_title, link } (optional) }`

### `breadcrumbs.module`
- `separator`: choice `/ | › | • | →`
- `show_home` (boolean)

### `pagination.module`
- `pagination_style`: choice `numbered | prev-next | load-more`

### `sidebar-nav.module`
- `menu`: menu picker
- `heading_text` (text, optional)

### `footer-nav.module`
- `columns`: repeater `{ heading_text (text), links (repeater of { link_label, link_url }) }`

---

## Interactive

### `button.module`
Single CTA. Fields: `button_label`, `button_url`, `style` (primary/secondary), `icon` (optional, icon picker), `open_in_new_tab`.

### `accordion.module`
- `allow_multiple_open` (boolean, default false)
- `items`: repeater `{ heading_text (text), content (richtext), default_open (boolean) }`

### `tabs.module`
- `orientation`: choice `horizontal | vertical`
- `default_tab` (number)
- `items`: repeater `{ tab_label (text), icon (icon, optional), content (richtext) }`

### `modal.module`
- `trigger_button`: group `{ button_label, button_url (optional), style }`
- `dialog`: group `{ heading_text (text), body_content (richtext), form_id (form picker, optional) }`
- `dismiss_on_overlay` (boolean, default true)

### `dropdown.module`
- `trigger_label` (text)
- `items`: repeater `{ link_label, link_url }`

### `tooltip.module`
- `trigger_text` (text), `tooltip_content` (text)
- `position`: choice `top | right | bottom | left`

### `slider.module` / `carousel.module`
- `items`: repeater `{ image, headline (richtext), body_content (richtext), button { button_label, button_url, style }, item_id (text, optional) }`
- `autoplay` (boolean), `interval` (number ms, default 5000), `loop` (boolean), `show_arrows` (boolean), `show_dots` (boolean)
- **Equal-height slides** — all slides share one `min-height` (e.g. `min-height: 360px`) or `aspect-ratio`. Content vertically centered with flexbox. Don't ship `height: auto` carousels (they jitter during autoplay).
- Icons inside slides centered (not top-aligned).

### `toggle-switch.module`
- `toggle_label` (text), `default_on` (boolean), `on_label` / `off_label` (text)
- For form-driven pages; emits `data-aci-toggle`.

---

## Forms & inputs (primitives)

Each consumes the **Forms** theme tokens.

### `input-field.module`
- `input_type`: choice `text | email | password | tel | url | number`
- `field_name` (text, required), `field_label` (text), `placeholder` (text), `required` (boolean), `help_text` (text)

### `textarea.module`
- `field_name`, `field_label`, `placeholder`, `rows` (number, default 4), `required`, `help_text`

### `select.module`
- `field_name`, `field_label`, `required`, `options`: repeater `{ option_label, option_value }`

### `checkbox.module`
- `field_name`, `field_label`, `default_checked` (boolean)

### `radio-group.module`
- `field_name`, `field_label`, `options`: repeater `{ option_label, option_value }`

### `search-bar.module`
- `placeholder` (text), `submit_label` (text), `action_url` (url)
- For blog search, default `action_url` to `/hs-search-results`, use `name="term"` for the input, and include hidden `type` inputs for `BLOG_POST` and `LISTING_PAGE`. (See [hubl-gotchas.md](hubl-gotchas.md).)

### `form-group.module`
- `legend` (text), child slot for inputs (drag-drop). On HubSpot, render via column group.

### `file-upload.module`
- `field_name`, `field_label`, `accept` (text, e.g. `.pdf,.jpg`), `multiple` (boolean), `help_text`

---

## Data display

### `table.module`
Custom HTML table builder. Fields:
- `headers`: repeater `{ header_text }`
- `rows`: repeater `{ cells: repeater { value (text or richtext) } }`
- `show_footer` (boolean), `footer`: repeater `{ footer_text }` (visible if true)
- Renders with `.aci-table` class consuming Tables tokens.

### `list.module`
- `list_style`: choice `unordered | ordered | check | arrow`
- `items`: repeater `{ content (richtext) }`

### `badges.module`
- `items`: repeater `{ badge_label (text), color_token: choice [primary, secondary, tertiary, success, warning, danger, info] }`

### `progress.module`
- `progress_label` (text), `value` (number), `max` (number, default 100), `color_token`

### `chart.module`
- `chart_type`: choice `bar | line | pie | doughnut | radar`
- `chart_title` (text), `data_json` (text — JSON of `{ labels: [], datasets: [] }`)
- Loads Chart.js from CDN once via `main.js`.

### `timeline.module`
- `items`: repeater `{ event_date (text), heading_text (text), body_content (richtext), icon (icon) }`
- `orientation`: choice `vertical | horizontal`

### `tags.module`
- `items`: repeater `{ tag_label, link (url, optional) }`

---

## Media

### `image.module`
- `image`, `alt_text`, `link` (url, optional), `lightbox` (boolean), `caption` (richtext, optional).

### `image-gallery.module`
- `layout`: choice `masonry | grid | slider`
- `columns` (2/3/4) for grid
- `items`: repeater `{ image, alt_text, caption }`

### `video-embed.module`
- `video_source`: choice `youtube_id | vimeo_id | wistia_id | hubspot_file`
- `video_value` (text id) or `video_file` (file picker), conditional via visibility
- `aspect_ratio`: choice `16:9 | 4:3 | 1:1 | 21:9` (default `16:9`)
- `autoplay` (boolean, default false) — per-instance module field. (If the brief wants a site-wide default, add a `video` group with an `autoplay` boolean to theme `fields.json` first and read it as `{{ theme.video.autoplay }}` — a documented behavioral exception to the tokens-only-in-emitter rule; never reference a `theme.*` path that isn't in `fields.json`.)
- `muted` (boolean, default true when autoplay is true)
- `loop` (boolean), `controls` (boolean)
- **Fixed height / aspect-ratio**, not `height: auto`.

### `icon-set.module`
- `columns` (2/3/4/6)
- `items`: repeater `{ icon_source: choice [library, custom_svg], icon (icon, visible if library), custom_svg_file (file, visible if custom_svg), label_text (text, optional), link (url, optional) }`

### Icon-source pattern (use anywhere an icon appears)

**Preferred (KineticGPO, post-upload-failure): no `icon` field at all.** Ship a `choice` of inline stroke-SVG glyph keys and render the SVG in `module.html` — version-independent, zero validation surface, zero external font dependency:

```jsonc
{ "type": "choice", "name": "glyph", "label": "Icon",
  "choices": [["arrow","Arrow"],["check","Checkmark"],["doc","Document"],["gear","Gear"]],
  "default": "arrow" }
```

If the editor genuinely needs a full icon library plus a custom-SVG path, ship the conditional trio — but note **HubSpot validates icon names against FontAwesome 5.0.10 regardless of the declared `icon_set`** (FA6-only names fail upload):

```jsonc
{ "type": "choice", "name": "icon_source", "label": "Icon source",
  "choices": [["library","Icon library"],["custom_svg","Custom SVG upload"]],
  "default": "library" },
{ "type": "icon", "name": "icon", "label": "Icon", "icon_set": "fontawesome-5.14.0",
  "visibility": { "controlling_field": "icon_source", "controlling_value_regex": "library", "operator": "EQUAL" },
  "default": {} },
{ "type": "file", "name": "custom_svg_file", "label": "Custom SVG", "picker": "file",
  "visibility": { "controlling_field": "icon_source", "controlling_value_regex": "custom_svg", "operator": "EQUAL" } }
```

`"default": {}` (empty object) is the Meridian-proven safe default. If you populate it, include all four keys with an **FA5-valid** name — partial defaults fail upload validation. (See [upload-validation.md §3](upload-validation.md).)

---

## Feedback / utility

### `loader.module`
- `variant`: choice `spinner | dots | bar`
- `color_token`: choice [primary, secondary, tertiary]

### `alert.module`
- `variant`: choice `info | success | warning | danger`
- `heading_text` (text), `body_content` (richtext), `dismissible` (boolean)

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
- Full-screen overlay block. `overlay_color` (color), `opacity` (number 0-100), `blur` (number px), child rich text.

---

## Encouraged extras (build only if industry/brief implies)

- `faq-schema.module` — accordion + JSON-LD output (uses `accordion`'s shape; emits a `<script type="application/ld+json">` block — the sole `<script>` permitted in `module.html`)
- `comparison-table.module` — multi-column feature comparison
- `lead-magnet-bar.module` — sticky bottom bar with form
- `newsletter-signup.module` — email + submit, wraps a HubSpot form
- `map-embed.module` — Google Maps iframe with address field
- `social-share.module` — share buttons for current URL
- `author-bio.module` — for blog post template
- `related-posts.module` — for blog post template (uses `blog_recent_posts(group, limit)`)

---

## Module rendering checklist (apply to every module)

- [ ] `module.html` has zero `<style>` and zero `<script>` tags (sole exception: `<script type="application/ld+json">` structured data).
- [ ] `module.css` references only `var(--aci-*)` for colors/fonts/spacing.
- [ ] All buttons render `.aci-btn.aci-btn--{primary|secondary}`; no module-local button styling.
- [ ] No field `name` value uses a HubSpot reserved word (`label`, `name`, `body`, `type`, `id`, `css`, `styles`, `child_css`, `overrideable`).
- [ ] All conditional fields use `visibility` rules with proper operators; nested-repeater conditionals use template `{% if %}` instead.
- [ ] Icon field `default` is either `{}` (empty object) or a complete FA5-valid 4-key object (`name`, `unicode`, `type`, `icon_set`) — never a partial.
- [ ] `meta.json` has valid `host_template_types` and `categories` from the enum.
- [ ] `meta.json` `is_available_for_new_content: true`.
- [ ] All images have alt fields.
- [ ] All interactive modules emit `data-aci-*` attributes; behavior lives in `main.js`.
- [ ] Mobile breakpoint at `767px` stacks all multi-column layouts; media-on-top in any media+content split.
- [ ] Carousel/slider items are equal-height; content vertically centered.
- [ ] Expandable cards trigger on the whole card, not only the icon.
- [ ] Color/style fields inside repeaters are at the item level, not group level.
- [ ] Repeater items with identity (testimonials, cases, models) expose optional `item_id` text field.
