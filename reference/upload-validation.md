# `hs upload` validation — catalog of errors and exact fixes

Every error in this file was hit on a real OneIMS theme upload. Each entry has: the exact error string `hs upload` (or `hs watch`) prints, the root cause, and the exact fix. Run this checklist mentally before reporting any theme as done.

---

## 1. Reserved field `name` values

**Error:**
```
[ERROR] Upload of file modules/<x>.module/fields.json failed — field name cannot be 'label'
```

**Root cause:** HubSpot reserves these strings as field `name` *values*:
`label`, `name`, `body`, `type`, `id`, `css`, `styles`, `child_css`, `overrideable`.

Additionally, the Aagard production theme documented rejections/conflicts on: `class`, `style`, `content`, `tag`. Treat the union of both lists as forbidden — the two client repos each hit a different half of it.

The human-readable `"label":` *key* in a field definition is fine — only the `"name":` *value* is reserved.

**Fix:** rename the field's `name`. Conventions proven in production:
- Descriptive: `heading_text`, `body_content`, `kind`, `tile_label`
- Aagard's canonical renames: `label→text`, `body→body_copy`/`body_richtext`, `tag→card_tag`, `content→body_richtext`, `type→card_type`
- Prefix: `aci_label`, `aci_body`, `aci_title`

Update every reference in:
- The module's `module.html` (`{{ module.label }}` → `{{ module.heading_text }}`)
- The module's `module.js` if it reads fields
- Every `{% module … field_values={ 'label': … } %}` call in templates/sections → change the key.

**Don't rename:** HubSpot system references like `content.name`, `{% menu %}` node names, or the bare `{{ name }}` instance id. Those aren't your custom fields.

---

## 2. `text` type rejected at theme root

**Error:**
```
[ERROR] Upload of file fields.json failed — 'text' fields are not supported in theme fields.json.
```

**Root cause:** theme-root `fields.json` only accepts a subset of field types. `text` isn't one of them.

**Fix:** replace `"type": "text"` with `"type": "richtext"` (rich editor) or `"type": "textarea"` (multi-line plain). This restriction does NOT apply to module `fields.json` — `text` works there.

---

## 3. Icon field `default` missing keys

**Error:**
```
[ERROR] Upload of file modules/<x>.module/fields.json failed — Icon field at path social_icon has an invalid default value (parameter: 'type')
```

(The missing key in the error varies — could be `name`, `unicode`, `type`, `icon_set`.)

**Root cause:** an `icon` field's `default` object must include all four keys — AND (discovered the hard way on KineticGPO) **HubSpot validates icon names against FontAwesome 5.0.10 regardless of what `icon_set` you declare.** Declaring `"icon_set": "fontawesome-6.4.2"` does not make FA6-only icon names pass — they're rejected anyway.

**Fix — in order of preference (KineticGPO → Meridian → legacy):**

1. **Best: skip the `icon` field entirely.** Ship a `choice` of inline stroke-SVG glyph keys and render the SVG in `module.html`. Version-independent, zero external dependency, zero validation surface. This is what KineticGPO converged on after a failed upload (`callout`, `offer-cards`, `document-rows` modules).
2. **Safe: `"default": {}`** (empty object — NOT `null`, NOT a partial) with `"icon_set": "fontawesome-5.14.0"`, as shipped across Meridian IT:
   ```jsonc
   { "name": "feature_icon", "type": "icon",
     "icon_set": "fontawesome-5.14.0", "display_width": null, "default": {} }
   ```
3. **If you must ship a populated default**, use an FA5-valid icon name with all four keys:
   ```jsonc
   { "type": "icon", "name": "feature_icon", "label": "Icon",
     "default": { "name": "rocket", "unicode": "f135", "type": "SOLID", "icon_set": "fontawesome-5.14.0" } }
   ```
   - `name` — icon slug (FontAwesome name without `fa-`) that exists in **FA 5**
   - `unicode` — the codepoint (fontawesome.com/icons)
   - `type` — one of `SOLID`, `REGULAR`, `BRANDS`

Never ship a partial `default` — omit keys and upload fails.

---

## 4. `responsive_breakpoints` invalid entry

**Error:**
```
[ERROR] Upload of file theme.json failed — invalid responsive_breakpoint name 'tablet'
```

**Root cause:** only the entry name `mobile` is valid.

**Fix — the canonical full shape (verbatim from the Meridian IT + Minitab production themes):**
```jsonc
"responsive_breakpoints": [
  {
    "name": "mobile",
    "previewWidth": { "value": 477, "units": "px" },
    "mediaQuery": "@media (max-width: 767px)",
    "messages": { }
  }
]
```
`previewWidth` is an object (`value` + `units`), and `messages: {}` must be present even when empty. Alternatively omit the array entirely and use CSS `@media` only — the Ondex/ACI theme ships no `responsive_breakpoints` key at all, which is the simplest way to sidestep the trap.

---

## 5. `meta.json` `categories` invalid enum

**Error:**
```
[ERROR] Upload of file modules/<x>.module/meta.json failed — invalid category 'BANNER_AND_CTA'
```

**Root cause:** `categories` only accepts:
`TEXT`, `COMMERCE`, `DESIGN`, `FUNCTIONALITY`, `FORMS_AND_BUTTONS`, `BODY_CONTENT`, `MEDIA`, `BLOG`, `SOCIAL`.

**Fix — map common mistakes:**
| Wanted to use         | Use instead         |
|-----------------------|---------------------|
| `BANNER_AND_CTA`      | `TEXT` or `FORMS_AND_BUTTONS` |
| `HERO`                | `BODY_CONTENT`      |
| `CTA`                 | `FORMS_AND_BUTTONS` |
| `NAVIGATION`          | `FUNCTIONALITY`     |
| `FORMS`               | `FORMS_AND_BUTTONS` |
| `IMAGE`               | `MEDIA`             |

---

## 6. Nested-repeater `controlling_field` cannot resolve

**Error:**
```
[ERROR] Upload of file modules/<x>.module/fields.json failed — items.video_embed: no controlling_field with path 'media_type' exists
```

**Root cause:** a child field's `visibility.controlling_field` can't reliably resolve a sibling inside a doubly-nested repeater (repeater inside a repeater).

**Fix:** drop the `visibility` rule and use a template-level `{% if %}` in `module.html`. Full example in [hubl-gotchas.md §3](hubl-gotchas.md).

---

## 7. File upload — empty stream

**Error:**
```
[ERROR] Upload of file images/<x>.jpg to <portal>/images/<x>.jpg failed — File source stream cannot be empty
```

**Root cause:** the local file is 0 bytes or unreadable.

**Fix:** verify with `ls -lh <file>` and `file <file>`. Re-export from source or restore from backup. Do not `git commit` 0-byte placeholders.

---

## 8. Section drop fails in editor — "There was an error inserting your section"

**Symptom:** dragging a section in the HubSpot editor errors out with no further detail.

**Root cause:** typically a validation error in the section's `.html` or in one of the modules it includes (missing `meta.json`, malformed HUBL, invalid field reference).

**Fix:**
1. Open browser DevTools → Console — HubSpot logs the underlying error there.
2. Re-validate every `meta.json` against the categories enum.
3. Re-validate every field name against the reserved-name list.
4. Refresh the editor and try again.

---

## 9. `related_blog_posts(...)` undefined

**Symptom:** template renders blank or HUBL warns of undefined function.

**Root cause:** `related_blog_posts` is not a HUBL function.

**Fix:** use `blog_recent_posts(group, limit)`. See [hubl-gotchas.md §1](hubl-gotchas.md).

---

## 10. Image field `default` shape

**Error:**
```
[ERROR] Upload of file modules/<x>.module/fields.json failed — Image field at path background_image has an invalid default value
```

**Fix — exact shape for image defaults:**
```jsonc
{ "type": "image", "name": "background_image", "label": "Background",
  "responsive": true,
  "resizable": true,
  "default": {
    "src": "",
    "alt": "",
    "width": 1600,
    "height": 900
  }
}
```

To ship an empty image picker, omit `default` entirely rather than supplying `null`.

---

## 11. Color field default shape

```jsonc
{ "type": "color", "name": "primary_color", "label": "Primary",
  "default": { "color": "#E74C3C", "opacity": 100 } }
```

`opacity` is required when supplying a default. Range 0–100.

---

## 12. Font field default shape

```jsonc
{ "type": "font", "name": "body_font", "label": "Body font",
  "load_external_fonts": true,
  "default": {
    "font": "Inter",
    "font_set": "GOOGLE",
    "fallback": "sans-serif",
    "variant": "regular",
    "size": 16,
    "size_unit": "px",
    "color": "#111111",
    "styles": {}
  }
}
```

- `font_set`: `GOOGLE`, `DEFAULT`, or `CUSTOM`
- `load_external_fonts: true` is needed for Google fonts to actually load on the rendered page.
- `styles: {}` is required (can be empty) — HubSpot rejects a default without the key.

---

## 13. Choice field — values must match `controlling_value_regex`

When a field's visibility depends on a `choice` parent, the `controlling_value_regex` must literally match the choice value.

```jsonc
// choice field
{ "type": "choice", "name": "icon_source",
  "choices": [["library","Icon library"],["custom_svg","Custom SVG"]],
  "default": "library" }

// dependent field
{ "type": "file", "name": "custom_svg_file",
  "visibility": { "controlling_field": "icon_source", "controlling_value_regex": "custom_svg", "operator": "EQUAL" } }
```

For multiple acceptable values, use `MATCHES_REGEX`:
```jsonc
"visibility": { "controlling_field": "background_type",
                "controlling_value_regex": "(image|video)",
                "operator": "MATCHES_REGEX" }
```

---

## 14. `theme.json` minimum upload-safe shape

```jsonc
{
  "name": "<theme-slug>",
  "label": "<Human Label>",
  "preview_path": "./templates/home.html",
  "screenshot_path": "./images/theme-thumb.png",
  "enable_domain_stylesheets": false,
  "is_available_for_new_content": true,
  "version": "1.0.0",
  "author": "OneIMS",
  "author_email": "hubspot@oneims.com",
  "responsive_breakpoints": [
    { "name": "mobile", "mediaQuery": "(max-width: 767px)" }
  ]
}
```

`enable_domain_styles` (without the `heets`) is a TYPO — HubSpot silently ignores it. Always use the full key.

---

## 15. `hubdbrow` / `hubdb_table` fields — table must exist AND be published

**Error:** upload of the module `fields.json` fails when it contains a `hubdbrow` (or `hubdb_row`) field whose table isn't in the portal.

**Root cause:** HubSpot validates HubDB references at upload time. The table must be **created and published** in the target portal *before* `hs upload` runs. Hit on KineticGPO's contract subcategories.

**Fix:** either create + publish the table first (`hs hubdb create`, then publish), or — the KineticGPO refactor — replace the HubDB reference with plain fields (`subcategory_name` text + `parent_category` choice), keeping the HubDB table as optional reference data.

---

## 16. `"type": "blog_post"` does not exist

There is **no native blog-post picker field type**. `"type": "blog_post"` rejects on upload. For "related posts" / "featured post" pickers, ship a manual repeater (`link` + `title_text` + `logo_image` + `tag_text` + `description`) — this is what KineticGPO's `related-contracts` module does. (The `blog` field type exists, but it picks a *blog*, not a *post*.)

---

## 17. STYLE tab may only contain style field types

**Error:** upload fails when a group with `"tab": "STYLE"` contains content fields.

**Root cause:** the STYLE tab whitelist is: `color`, `boolean`, `number`, `choice`, `font`, `spacing`, `border`, `gradient`, `backgroundimage`, `alignment`. `text`, `richtext`, `image`, `link`, etc. must live on the CONTENT tab (the default). Aagard hit this on `Two Column Repeater` and `Pillar Cards` — fixed by moving the `bg_settings` group to `"tab": "CONTENT"`.

---

## 18. `dnd_section` accepts a narrow parameter set

Inside templates/sections, `{% dnd_section %}` only accepts: `background_color`, `background_image`, `background_linear_gradient`, `margin`, `padding`, `vertical_alignment`, `full_width`, `max_width`. Passing arbitrary attributes fails or is silently dropped. Style anything else via a module or class.

---

## 19. CSS formatters destroy HubL in `theme-overrides.css`

**Symptom:** the entire Theme Settings panel goes inert — no setting changes anything on the page.

**Root cause:** running Prettier / a CSS formatter on a HubL-bearing stylesheet rewrites `{% set ... %}` into invalid CSS like `{ % set container_width=... % }`. The Ondex/ACI production theme shipped with **61 statements mangled this way** — the whole token bridge was dead and nobody noticed because `main.css` had hardcoded fallbacks.

**Fix:** add every HubL-bearing `.css` file to `.prettierignore` / formatter excludes. Sanity-check before upload:
```bash
grep -n '{ %\|% }' css/*   # any hit = a formatter has mangled HubL (covers .css AND .css.html)
```

---

## 20. Emit tokens from the field `name`, never the field `id`

`fields.json` fields may carry both `"id": "spc_max_width"` and `"name": "max_width"`. HubL resolves **names only**: `theme.spacing.max_width` works, `theme.spacing.spc_max_width` silently resolves to nothing. Ondex shipped this bug — combined with #19 it killed the panel twice over.

---

## 21. Token-path drift — every `theme.x.y` must exist in `fields.json`

Three separate production themes shipped `theme-overrides` referencing fields that don't exist:
- Meridian: reads `theme.global_colors.tirtiary_color` (field is `tertiary_color`), `theme.forms.form_border_color` (field is `form_bor_color`), `theme.header.header_style_type` (field absent)
- Minitab: reads a whole `theme.blog.*` group that isn't in `fields.json`

HubSpot does NOT error on these — they silently emit nothing. **Before reporting done, diff every `theme.<group>.<field>` path in the token emitter against `fields.json` names.** Same class of check: every `var(--x)` consumed in any CSS file must have a `:root` declaration (Ondex consumes `--aci-dark` and `--aci-teal-deep`, declares neither).

---

## 22. Zero mandatory fields

`required: true` on module fields blocks editors and causes hard-to-diagnose editor errors. KineticGPO removed it theme-wide after validation pain: **ship 0 mandatory fields**, give every field a usable default, and guard renders with `{% if %}` / empty states instead.

---

## 23. Repo hygiene — files that break or pollute uploads

- **Never leave `* copy.json` / `theme copy.json` / `fields copy.json`** in the theme folder (Aagard shipped both) — they upload as real files.
- **No duplicate module dirs** — Meridian has a literal `modules/modules/` orphan that collides on upload.
- **No backup templates at theme root** (`webinar-post____backup.html`, `stacked-cards-preview.html`).
- **Omit `module_id` in new `meta.json`** — it's a portal-linked ID; let HubSpot assign it on first upload. Copying one from another module/portal causes collisions.

---

## 24. System page `templateType` naming

Membership reset pages use `membership_reset_request_page` / `membership_reset_page` template types (KineticGPO corrected these after upload errors). File naming varies by theme (`membership-reset.html` vs `membership-reset-password.html`) — both work; the `templateType` annotation is what's validated.

---

## 25. Missing required tags `standard_header_includes` / `standard_footer_includes`

**Error (Design Manager validation, hit in production on TMA Systems):**
```
Error   0:-1  This template is missing the required tag {{ standard_header_includes }}
Error   0:-1  Template doesn't include the tag standard_footer_includes. Needed for HubSpot analytics
Warning 6:-1  Missing template <Theme>/templates/layouts/base.html
```

**Root cause:** every HubSpot page template must resolve both tags. When they're inherited via `{% extends %}`, the layout must exist **in the portal** — the two Errors are usually symptoms of the Warning: the extends target isn't there (lone template pasted into Design Manager, or only `templates/` uploaded), so HubSpot validates the template layout-less and the inherited tags "disappear".

**Fix:**
1. The base layout carries `{{ standard_header_includes }}` last in `<head>` and `{{ standard_footer_includes }}` before `</body>` (the skill's `base.html` does).
2. Upload the **whole theme root** in one `hs upload <theme-root> <theme-root>` so layout + partials land with the templates. `{% extends %}` / `{% global_partial %}` resolve against portal files, not local ones.
3. Any template with no layout (standalone landing pages) must inline both tags itself.

---

## 26. Membership pages require dedicated form tags — NOT `{% module path="@hubspot/…" %}`

**Error (hit on the B2B SaaS marketplace theme upload):**
```
Error validating template.
- line 0: The template does not contain the module member_login
- line 0: The template does not contain the module member_register
- line 0: The template does not contain the module password_reset
- line 0: The template does not contain the module password_reset_request
```

**Root cause:** the four membership templates rendered their form with `{% module "member_login" path="@hubspot/membership_login" %}`. HubSpot does **not** accept a generic module here — each membership template must contain a specific **dedicated HubL tag**. `@hubspot/membership_login` is a real module path but it does **not** satisfy the validator.

**Fix — use the dedicated tags (confirmed against approved marketplace themes):**

| Template (`templateType`) | Required tag |
|---|---|
| `membership-login.html` (`membership_login_page`) | `{% member_login "login" … %}` |
| `membership-register.html` (`membership_register_page`) | `{% member_register "register" … %}` |
| `membership-reset-password.html` (`membership_reset_page`) | `{% password_reset "reset" … %}` |
| `membership-reset-password-request.html` (`membership_reset_request_page`) | `{% password_reset_request "reset_request" … %}` |

Params: `member_login` → `email_label, password_label, remember_me_label, reset_password_text, submit_button_text`; `member_register` / `password_reset` → `password_label, password_confirm_label, submit_button_text` (register also takes `email_label`); `password_reset_request` → `email_label, submit_button_text`. Wrap each tag in your own markup (headings, auth panel) — only the tag itself is validated.

**The OTHER system pages ARE plain modules** (matches approved themes — do NOT convert these to tags):
`password-prompt.html` → `@hubspot/password_prompt` · `subscription-preferences.html` → `@hubspot/email_subscriptions` · `subscriptions-confirmation.html` → `@hubspot/email_subscriptions_confirmation` · `backup-unsubscribe.html` → `@hubspot/email_simple_subscription` · `search-results.html` → `@hubspot/search_results`, all via `{% module "…" path="@hubspot/…" %}`.

---

## 27. Template Marketplace review requirements (beyond `hs upload`)

These do **not** fail `hs upload` — they fail the **Marketplace review** (the pass after a clean upload). All four were required edits on the B2B SaaS submission:

1. **Listing name — forbidden words.** The marketplace listing name must **not** contain `HubSpot`, `Hub`, `Inbound`, or `Theme`. (The `theme.json` `label` is a separate field, but keep it clean too.) Name by industry/brand: `"Apex — B2B SaaS & Technology"`, not `"B2B SaaS Theme"`.
2. **`theme.json` needs `documentation_url` AND `example_url`** — both publicly accessible (a docs page + a live demo URL, e.g. `https://<portal>.hs-sites.com/…`). Missing either is a required edit.
3. **`language_switcher` is required** for multi-language support. Put `{% language_switcher "id" %}` in the **header and footer partials** (`templates/partials/header.html` / `footer.html`) — **NOT inside a custom module**. Inside a module it renders in the module-preview context (no page translations to resolve) and **hangs / severely slows the Design Manager editor**. It's invisible on single-language sites, so placement is about where it *would* appear.
4. **No large blocks of commented-out code.** Reviewers flag any large multi-line comment block — disabled code *and* verbose design-rationale banners. Strip 3-line-or-longer comment blocks from `css/main.css`, `js/main.js`, every `templates/*.html`, and every `modules/*/module.*`. Keep the required `<!-- templateType … -->` metadata header and short one-line section labels.

---

## Pre-upload checklist

Run this before any `hs upload`:

- [ ] No field has `name` matching `label`, `name`, `body`, `type`, `id`, `css`, `styles`, `child_css`, `overrideable` (nor `class`, `style`, `content`, `tag`).
- [ ] No theme-root field uses `type: "text"`.
- [ ] Icon fields: preferably replaced by an inline-SVG `choice`; otherwise `icon_set: fontawesome-5.14.0` with `default: {}` or a complete FA5-valid 4-key default.
- [ ] Every font field default has `font`, `font_set`, `fallback`, `styles: {}` at minimum.
- [ ] Every color field default has `color` AND `opacity`.
- [ ] Every `meta.json` `categories` value is from the enum, UPPERCASE (lowercase is the legacy risky form).
- [ ] `responsive_breakpoints` is either omitted or contains only `mobile` (with `previewWidth` object + `messages: {}`).
- [ ] No nested-repeater field uses `visibility.controlling_field` for a sibling inside the same nested level.
- [ ] No `hubdbrow`/`hubdb_table` field references a table that isn't created AND published in the target portal.
- [ ] No `"type": "blog_post"` field anywhere.
- [ ] No content field (text/richtext/image/link) inside a `"tab": "STYLE"` group.
- [ ] `grep -n '{ %' css/*` returns nothing (no formatter-mangled HubL — covers `.css` and `.css.html`).
- [ ] Every `theme.<group>.<field>` path in the token emitter exists in `fields.json` (by `name`, not `id`).
- [ ] Every `var(--x)` consumed anywhere is declared in a `:root` block.
- [ ] No `required: true` on any module field — defaults + empty-state guards instead.
- [ ] No `* copy.json`, backup templates, or duplicate module dirs in the upload folder; no `module_id` in new `meta.json`.
- [ ] No HUBL call to `related_blog_posts`.
- [ ] Blog search forms submit (GET) to `/hs-search-results` with `name="term"` and hidden `type` inputs.
- [ ] `theme.json` includes `name`, `label`, `screenshot_path`, `enable_domain_stylesheets` (not `enable_domain_styles`).
- [ ] Every template resolves `{{ standard_header_includes }}` + `{{ standard_footer_includes }}` (via a layout that uploads WITH it, or inline); deploy instructions upload the whole theme root, never a lone template.
- [ ] Membership templates render with the dedicated tags `{% member_login %}` / `{% member_register %}` / `{% password_reset %}` / `{% password_reset_request %}` — NEVER `{% module path="@hubspot/membership_login" %}` (§26).
- [ ] **(Marketplace)** `theme.json` has `documentation_url` + `example_url`; a `{% language_switcher %}` sits in the header + footer **partials** (never in a module); the intended listing name has no `HubSpot`/`Hub`/`Inbound`/`Theme`; no 3-line-or-longer commented-out code blocks anywhere (§27).

If the upload still fails after this checklist passes, the error string almost always points at the exact file + path. Read it carefully — HubSpot's messages are precise.
