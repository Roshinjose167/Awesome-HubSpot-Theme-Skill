# HUBL gotchas

Each item here is something the OneIMS team has been burned by in a real client engagement. Don't rediscover these in production.

---

## 1. `related_blog_posts(...)` does NOT exist

Use `blog_recent_posts(group, limit)`. That's the actual function.

```hubl
{# WRONG — this function doesn't exist; HUBL will error #}
{% set related = related_blog_posts(content.blog.id, 3) %}

{# RIGHT #}
{% set recent = blog_recent_posts('default', 6) %}
{% for post in recent %}
  <a href="{{ post.absolute_url }}">{{ post.name }}</a>
{% endfor %}
```

For *truly related* posts on a single blog-post template, filter recent posts by the current post's tag:

```hubl
{% set candidates = blog_recent_posts('default', 20) %}
{% set out = [] %}
{% for post in candidates %}
  {% if post.id != content.id and post.topic_list|intersect(content.topic_list)|length %}
    {% set out = out + [post] %}
  {% endif %}
{% endfor %}
{% for post in out[:3] %}
  <a href="{{ post.absolute_url }}">{{ post.name }}</a>
{% endfor %}
```

---

## 2. Blog search form must submit (GET) to `/hs-search-results`

HubSpot ships a built-in search results page at `/hs-search-results`. Your blog search form must target it directly — a custom action won't work.

```html
<!-- RIGHT — works in any blog template -->
<form action="/hs-search-results" method="get" class="aci-search">
  <input type="text" name="term" placeholder="Search..." aria-label="Search">
  <input type="hidden" name="type" value="BLOG_POST">
  <input type="hidden" name="type" value="LISTING_PAGE">
  <button type="submit" class="aci-btn aci-btn--primary">Search</button>
</form>
```

- Query param is `name="term"`, NOT `q`.
- Include hidden `<input type="hidden" name="type" value="BLOG_POST">` (and optionally `LISTING_PAGE`).
- **Do NOT** include `SITE_PAGE` if the user only wants blog/listing hits — that pollutes results with marketing pages.

---

## 3. Nested-repeater visibility — use `{% if %}`, not `controlling_field`

When a child field lives **inside a doubly-nested repeater** and its visibility depends on a sibling at the same level, HubSpot's validator often rejects the upload:

```
[ERROR] Upload failed — items.video_embed: no controlling_field with path 'media_type' exists
```

Fix: drop the `visibility` rule from the JSON, and guard the render at template level instead.

```jsonc
// fields.json — items live inside another repeater
{ "type": "group", "name": "tabs",
  "occurrence": { "min": 1, "max": 6 },
  "children": [
    { "type": "group", "name": "items",
      "occurrence": { "min": 1, "max": 6 },
      "children": [
        { "type": "choice", "name": "media_type", "label": "Media type",
          "choices": [["image","Image"],["video","Video"]], "default": "image" },
        { "type": "image", "name": "media_image", "label": "Image" },
        // NO visibility rule on media_video — handled in template
        { "type": "text", "name": "media_video", "label": "Video ID" }
      ]}
  ]
}
```

```hubl
{# module.html — template-level guard works reliably #}
{% for tab in module.tabs %}
  {% for item in tab.items %}
    {% if item.media_type == "image" %}
      <img src="{{ item.media_image.src }}" alt="{{ item.media_image.alt }}">
    {% elif item.media_type == "video" %}
      <iframe src="https://youtube.com/embed/{{ item.media_video }}" loading="lazy"></iframe>
    {% endif %}
  {% endfor %}
{% endfor %}
```

---

## 4. `responsive_breakpoints` — only `mobile` is valid

In `theme.json`:

```jsonc
// RIGHT
"responsive_breakpoints": [
  { "name": "mobile", "mediaQuery": "(max-width: 767px)" }
]

// WRONG — tablet/desktop entries are rejected
"responsive_breakpoints": [
  { "name": "mobile",  "mediaQuery": "(max-width: 767px)" },
  { "name": "tablet",  "mediaQuery": "(max-width: 991px)" },
  { "name": "desktop", "mediaQuery": "(min-width: 992px)" }
]
```

You may also omit the array entirely and rely on CSS `@media` queries (`@media (max-width: 767px) { … }`).

---

## 5. `meta.json` `categories` enum

```jsonc
// RIGHT — only these values are valid:
"categories": ["BODY_CONTENT"]
// TEXT, COMMERCE, DESIGN, FUNCTIONALITY, FORMS_AND_BUTTONS, BODY_CONTENT, MEDIA, BLOG, SOCIAL

// WRONG — these are rejected:
"categories": ["BANNER_AND_CTA"]   // does not exist; use TEXT or FORMS_AND_BUTTONS
"categories": ["HERO"]              // does not exist; use BODY_CONTENT
"categories": ["CTA"]               // does not exist; use FORMS_AND_BUTTONS
```

---

## 6. Sticky-header pattern — hide on scroll-down, show on scroll-up

The canonical OneIMS sticky-header pattern (Aagard, Minitab, MyGovBill all use it). Track scroll direction, toggle a class:

```js
// main.js
let _lastY = window.scrollY;
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
.aci-header { position: fixed; inset: 0 0 auto 0; transition: transform .3s ease; }
.aci-header.scrolled-down { transform: translateY(-100%); }
```

---

## 7. Mobile menu — logo always remains a working link

When the mobile drawer is open, the logo at the top must still link to homepage. Don't disable, hide, or swap the link in the open state:

```html
<a href="{{ request.site.absolute_url|default('/') }}" class="aci-logo" aria-label="Home">
  <img src="{{ module.logo.src }}" alt="{{ module.logo.alt }}">
</a>
```

```css
.aci-mobile-drawer.is-open .aci-logo {
  /* keep z-index high, position absolute if needed — but never pointer-events: none */
  position: relative;
  z-index: 10;
}
```

---

## 8. Featured-card section under search — flatten the layout

Blog listings often use an asymmetric featured layout (e.g. `grid-template-columns: 2fr 1fr` with one big card + smaller side cards). When the user is searching and only some featured cards match, the survivors look stranded.

Fix: when search is active, flatten the featured section to match the main grid:

```html
<form id="blog-search" action="/hs-search-results" method="get"
      onsubmit="document.body.classList.add('is-searching')">
  <!-- … -->
</form>

<section class="aci-blog-featured">
  <!-- featured cards -->
</section>
```

```css
body.is-searching .aci-blog-featured {
  grid-template-columns: repeat(3, 1fr);   /* match main grid */
  gap: var(--aci-grid-gap-x);
}
body.is-searching .aci-blog-featured__big {
  grid-column: auto;                       /* drop the 2fr span */
}
```

---

## 9. Blog dates — date only, no time

```hubl
{# Strip the time component, use a long format consistently #}
{{ content.publish_date|format_date('long') }}

{# Or: explicit format #}
{{ content.publish_date|formatdate('%B %d, %Y') }}
```

Style the same across featured and grid cards (typically right-aligned, muted color via `--aci-secondary-text` or similar token).

---

## 10. Pagination orphans

Avoid layouts that leave a single orphaned card on the last page (e.g. 1 featured + 9 grid on page 1, then 10 on page 2 in a 3-col grid → 1 orphan on the bottom row).

Two clean fixes:
1. **Adjust per-page count** so every page has a row-aligned count (multiples of column count).
2. **Hide the featured slot on page 2+** so every paginated page renders a uniform N-column grid.

Expose `posts_per_page` and `hide_featured_after_first_page` via theme settings so editors can tune without code changes.

---

## 11. Video as blog featured image

HubSpot blog doesn't accept video files as featured images. The OneIMS workaround:

1. Grab a representative frame from the video (still image).
2. Save as JPG/PNG.
3. Upload as the blog post's `featured_image`.
4. (Optional) Add a play-overlay UI in `blog-post.html` so the still reads as "video".

---

## 12. Favicon with light/dark mode

Single-file approach — SVG with embedded `@media (prefers-color-scheme: dark)`:

```html
<!-- favicon.svg -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 32 32">
  <style>
    .fg { fill: #111111; }
    @media (prefers-color-scheme: dark) { .fg { fill: #FFFFFF; } }
  </style>
  <path class="fg" d="…"/>
</svg>
```

```html
<!-- templates/layouts/base.html <head> — favicon.svg lives at theme root, two levels up -->
<link rel="icon" type="image/svg+xml" href="{{ get_asset_url('../../favicon.svg') }}">
```

No `.ico`, no second light/dark copy. Saves bytes and stays correct under OS theme switches.

---

## 13. Field reference syntax — `module.x` vs `theme.x`

| Scope | Where it lives | HUBL reference |
|-------|----------------|----------------|
| Module instance field | `<module>/fields.json` | `module.my_field` |
| Repeater item | repeater in `fields.json` | `module.items[loop.index0].my_field` |
| Theme setting | theme-root `fields.json` | `theme.group.field` |
| HubSpot system | built-in | `content.x`, `request.x`, `site_settings.x` |

The theme tokens (`theme.x`) should only appear inside `css/theme-overrides.css.html`. Modules read `var(--aci-…)` — not `{{ theme.x }}`.

---

## 14. Grid / container padding that's responsive

Don't hardcode `padding: 24px`. Use viewport-relative or theme-token-driven:

```css
.aci-container { padding-inline: clamp(1.5rem, 3vw, var(--aci-container-px)); }
```

Or wrap via `@media`:

```css
.aci-container { padding-inline: var(--aci-container-px); }
@media (max-width: 1400px) { .aci-container { padding-inline: 1.5rem; } }
@media (max-width: 767px)  { .aci-container { padding-inline: 1rem; } }
```

---

## 15. Avoid `text` fields at theme root

Theme-root `fields.json` rejects `type: "text"`:

```
[ERROR] 'text' fields are not supported in theme fields.json.
```

For theme-level free text (e.g. footer copyright, default eyebrow), use `richtext` or `textarea`. This restriction does NOT apply to *module*-level `fields.json` — `text` is fine there.

---

## 16. Color pickers inside repeaters — per item, not group

Wrong:

```jsonc
// One color picker outside the repeater applies to every row — editor can't differentiate.
{ "type": "color", "name": "title_color", ... },
{ "type": "group", "name": "items", "occurrence": { ... },
  "children": [ /* … */ ] }
```

Right:

```jsonc
// Each row picks its own color.
{ "type": "group", "name": "items", "occurrence": { ... },
  "children": [
    { "type": "color", "name": "title_color", "label": "Title color (per tile)",
      "default": { "color": "#FFFFFF", "opacity": 100 } },
    /* … */
  ] }
```

---

## 17. Editor preview disappearing after `fields.json` edit

After editing a module's `fields.json` (especially repeaters), the module may temporarily vanish from the HubSpot editor preview while the frontend still renders correctly. This is HubSpot's editor cache, not a real bug.

Workaround:
1. Refresh the page (Cmd-R).
2. Wait 30–60 seconds for HubSpot's background re-indexer.
3. If still missing: open the module from "Add module" picker — that forces a re-read.

Don't restructure the module assuming it's broken — wait first.

---

## 18. Blog listing helper trio — pagination with offset

The Meridian blog/listing modules are the reference implementation:

```hubl
{% set posts = blog_recent_posts(blog_id, post_limit, post_limit * (current_page - 1)) %}
{% set total_posts = blog_total_post_count(blog_id) %}
{# page links: blog_page_link(n) #}
```

- `blog_recent_posts(group, limit, offset)` — the third arg is the offset; that's how you paginate.
- `blog_recent_tag_posts(group, tag, limit)` exists for tag filtering.
- Use the `blog` **field type** so editors pick which blog feeds the module.

---

## 19. `visibility` — three resolution forms, use the right one

| Situation | Use |
|---|---|
| Sibling at the same flat level | `controlling_field: "field_name"` |
| Sibling inside a (single-nested) group | `controlling_field: null` + **`controlling_field_path: "group.field_name"`** (dotted) — the KineticGPO section-shell idiom, `MATCHES_REGEX` for multi-value (`"image|video"`) |
| Sibling inside a doubly-nested repeater | Neither resolves reliably — drop `visibility` and guard with `{% if item.x == "y" %}` in the template (§3) |

Note: HubSpot's own generated fields sometimes reference the controlling field by **UUID** rather than name (seen across Meridian). Valid, but never hand-author UUIDs — use names/paths.

---

## 20. Widget data reads need `|default(...)`

`content.widgets.<module>.body.<field>` contains **only the fields an editor actually edited**. Untouched instances are missing keys entirely. Every cross-page read (feeds, listings, related content) must be defensive:

```hubl
{% set eyebrow = widget.body.eyebrow_text|default('') %}
```

---

## 21. HubDB — publish-before-upload and the `orderBy` trap

- A module `fields.json` containing `hubdbrow`/`hubdb_table` fails upload unless the table is **created and published** in the target portal first (`hs hubdb create`, publish, then `hs upload`).
- **Never add `orderBy` to a query against a manually-curated table** — it silently discards the drag-order the client curated (KineticGPO contract subcategories). No `orderBy` = rows come back in table order.

---

## 22. Absolute vs relative module paths in `dnd_module`

Both `path='../modules/Banner CTA'` and `path='/Landin-Page-Templates/modules/Banner CTA'` work. The absolute (theme-rooted) form is safer inside deeply nested `sections/` partials — relative paths break when a partial is included from a different depth (Minitab uses both; absolute is the recommendation).

---

## 23. HubSpot's default form CSS — tame it before it breaks your layout

The HubSpot form embed ships its own stylesheet that silently fights your design. The defaults that bite:

- `.hs-form fieldset { max-width: 500px }` — your form never fills its column, no matter the container width.
- `.hs-input { width: 95% }` — inputs stop 5% short, so a two-column field row looks misaligned.
- Checkboxes and radios are laid out with **floats**, so a custom grid silently collapses them into a stack and the label wraps under the box.

Reset it at the form scope — do NOT fight it field by field:

```css
.hs-form fieldset { max-width: none; width: 100%; }
.hs-form .hs-input { width: 100%; }
.hs-form .hs-form-field { margin-bottom: 1rem; }
/* layout with grid instead of HubSpot's floats */
.hs-form fieldset.form-columns-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; }
/* pin the checkbox/radio input so the grid doesn't stretch it */
.hs-form input[type="checkbox"],
.hs-form input[type="radio"] { width: auto !important; float: none !important; margin: 0 .5rem 0 0; }
```

The `!important` on the checkbox width/float is required — HubSpot's inline-weight default wins otherwise.

---

## 24. Field names that collide with dict methods

Two different failure modes, two different name lists — avoid both:

- **Hard reserved (fails `hs upload`):** `class`, `style`, `content`, `tag`, `type`, `id`, `name`, `label`, plus the HubSpot system set. See [upload-validation.md §1](upload-validation.md).
- **Soft collisions (render fine on HubSpot, break the local Jinja2 test harness):** a field named `copy`, `items`, `keys`, `values`, `get`, `pop`, or `update` shadows a Python `dict` method. HubSpot's HubL tolerates it, but the local Jinja2 harness resolves `module.copy` to the bound method, not your field, and the render throws or returns garbage.

Rule: never name a field after a dict method. Reach for `copy_text`, `list_items`, `key_points` instead. This keeps the same source renderable both in the harness and on HubSpot.

---

## 25. `overflow-x: clip` collapses the editor canvas

`html, body { overflow-x: clip }` is a tempting way to kill horizontal scroll. The **live page** looks perfect. But inside the HubSpot page editor, `overflow-x: clip` on `html`/`body` breaks the editor's canvas **height** measurement — the drag-and-drop canvas renders blank or zero-height, and the editor is unusable.

Use `overflow-x: hidden` on `body` only:

```css
body { overflow-x: hidden; }   /* not: html, body { overflow-x: clip } */
```

`hidden` contains the overflow without poisoning the editor's height calc.

---

## 26. `backdrop-filter` on a fixed header breaks the sticky nav

A frosted-glass header (`backdrop-filter: blur()`) on the **fixed/sticky** nav element creates a new containing block. Any `position: fixed` child (mobile menu overlay, dropdown) is now positioned relative to the header, not the viewport, and mega-menus/overlays jump or clip.

Keep the filter off the positioning element: put `backdrop-filter` on an **inner** wrapper inside the fixed header, or use a solid/opaque background on the fixed element itself. Never combine `position: fixed` + `backdrop-filter` on the same node.

---

## 27. `module.css` outranks any skin / override layer

When a theme uses a skin layer (a theme-level stylesheet that restyles modules), remember the cascade: a module's own `module.css` loads after and is more specific than the skin, so the skin's rules lose. Two consequences:

- Keep each module's CSS to **2–3 classes** of genuinely structural styling (layout, not color/spacing tokens). The more a module hardcodes, the less a skin can retheme it.
- Push color, spacing, radius, and type to **tokens** (`var(--…)`) so the skin can retheme by swapping tokens, not by out-specificity-ing `module.css`.
- If you test with a local harness, the harness **must load the skin stylesheet** in the same order HubSpot does, or your skin overrides silently no-op in preview.
