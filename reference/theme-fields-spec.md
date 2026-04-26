# Theme Settings — `fields.json` spec

This is the exact shape the agent must produce at the theme root. Every group below is required. Field names are `snake_case`. Default colors and fonts can be replaced with the brand values gathered in the input phase.

## Order in the panel

1. Layout & Structure
2. Global Colors
3. Global Fonts
4. Website Header
5. Typography
6. Primary Button
7. Secondary Button
8. Forms
9. Tables
10. Spacing
11. Footer
12. Blog

```json
[
  {
    "type": "group", "name": "layout_structure", "label": "Layout & Structure", "expanded": true,
    "children": [
      { "type": "group", "name": "container", "label": "Container / Wrapper", "children": [
        { "type": "number", "name": "max_width", "label": "Max width (px)", "default": 1300, "min": 320 },
        { "type": "number", "name": "horizontal_padding", "label": "Horizontal padding (px)", "default": 24, "min": 0 }
      ]},
      { "type": "group", "name": "section", "label": "Section", "children": [
        { "type": "number", "name": "vertical_spacing", "label": "Vertical spacing (px)", "default": 70, "min": 0 },
        { "type": "color",  "name": "default_bg",       "label": "Default background", "default": { "color": "#FFFFFF", "opacity": 100 } }
      ]},
      { "type": "group", "name": "grid", "label": "Grid / Row / Column", "children": [
        { "type": "number", "name": "column_gap", "label": "Column gap (px)", "default": 24, "min": 0 },
        { "type": "number", "name": "row_gap",    "label": "Row gap (px)",    "default": 24, "min": 0 },
        { "type": "number", "name": "columns",    "label": "Default columns (12-grid)", "default": 12, "min": 1, "max": 12 }
      ]},
      { "type": "group", "name": "flex", "label": "Flexbox Layout", "children": [
        { "type": "number", "name": "gap", "label": "Default gap (px)", "default": 16, "min": 0 },
        { "type": "choice", "name": "align", "label": "Default alignment",
          "choices": [["start","Start"],["center","Center"],["end","End"],["between","Space between"]],
          "default": "start" }
      ]},
      { "type": "group", "name": "sidebar", "label": "Sidebar", "children": [
        { "type": "number", "name": "width", "label": "Width (px)", "default": 320, "min": 200 },
        { "type": "number", "name": "gap",   "label": "Gap to main (px)", "default": 32, "min": 0 },
        { "type": "choice", "name": "position", "label": "Position",
          "choices": [["left","Left"],["right","Right"]], "default": "right" }
      ]},
      { "type": "group", "name": "fullwidth", "label": "Full-width Section", "children": [
        { "type": "boolean", "name": "edge_bleed", "label": "Edge bleed by default", "default": false }
      ]},
      { "type": "group", "name": "split_5050", "label": "Split Layout (50/50)", "children": [
        { "type": "number", "name": "gap", "label": "Gap (px)", "default": 48, "min": 0 },
        { "type": "number", "name": "stack_at", "label": "Stack at (px breakpoint)", "default": 768, "min": 320 }
      ]}
    ]
  },

  {
    "type": "group", "name": "global_colors", "label": "Global Colors", "expanded": true,
    "children": [
      { "type": "color", "name": "primary_color",   "label": "Primary color",   "default": { "color": "#E74C3C", "opacity": 100 } },
      { "type": "color", "name": "secondary_color", "label": "Secondary color", "default": { "color": "#0D899E", "opacity": 100 } },
      { "type": "color", "name": "tertiary_color",  "label": "Tertiary color",  "default": { "color": "#201600", "opacity": 100 } },
      { "type": "color", "name": "success", "label": "Success", "default": { "color": "#16A34A", "opacity": 100 } },
      { "type": "color", "name": "warning", "label": "Warning", "default": { "color": "#F59E0B", "opacity": 100 } },
      { "type": "color", "name": "danger",  "label": "Danger",  "default": { "color": "#DC2626", "opacity": 100 } },
      { "type": "color", "name": "info",    "label": "Info",    "default": { "color": "#2563EB", "opacity": 100 } }
    ]
  },

  {
    "type": "group", "name": "global_fonts", "label": "Global Fonts",
    "children": [
      { "type": "font", "name": "primary_font", "label": "Primary font", "load_external_fonts": true,
        "default": { "font": "Plus Jakarta Sans", "font_set": "GOOGLE", "fallback": "sans-serif",
                     "variant": "regular", "size": 16, "size_unit": "px", "color": "#201600", "styles": {} } },
      { "type": "font", "name": "secondary_font", "label": "Secondary font (optional — headings default to Primary)", "load_external_fonts": true,
        "default": { "font": "Plus Jakarta Sans", "font_set": "GOOGLE", "fallback": "sans-serif",
                     "variant": "regular", "size": 16, "size_unit": "px", "color": "#201600", "styles": {} } }
    ]
  },

  {
    "type": "group", "name": "website_header", "label": "Website Header",
    "children": [
      { "type": "color",   "name": "header_scrolled_bg", "label": "Background (.aci-header.scrolled)", "default": { "color": "#FFFFFF", "opacity": 100 } },
      { "type": "color",   "name": "nav_link_color",     "label": "Menu link color (.nav-link)",        "default": { "color": "#201600", "opacity": 100 } },
      { "type": "color",   "name": "child_menu_bg",      "label": "Child menu background",              "default": { "color": "#FFFFFF", "opacity": 100 } },
      { "type": "number",  "name": "logo_max_height",    "label": "Logo max height (px)",                "default": 48, "min": 16 },
      { "type": "boolean", "name": "sticky",             "label": "Sticky header",                       "default": true },
      { "type": "boolean", "name": "scroll_shrink",      "label": "Shrink on scroll",                    "default": true }
    ]
  },

  {
    "type": "group", "name": "typography", "label": "Typography",
    "children": [
      { "type": "font",   "name": "body_font", "label": "Body text", "load_external_fonts": true,
        "default": { "font": "Plus Jakarta Sans", "font_set": "GOOGLE", "fallback": "sans-serif", "size": 16, "size_unit": "px", "color": "#201600" } },
      { "type": "font",   "name": "h1_font", "label": "Heading 1", "load_external_fonts": true,
        "default": { "font": "Plus Jakarta Sans", "font_set": "GOOGLE", "fallback": "sans-serif", "size": 56, "size_unit": "px", "color": "#201600", "variant": "700" } },
      { "type": "font",   "name": "h2_font", "label": "Heading 2", "load_external_fonts": true,
        "default": { "font": "Plus Jakarta Sans", "font_set": "GOOGLE", "fallback": "sans-serif", "size": 44, "size_unit": "px", "color": "#201600", "variant": "700" } },
      { "type": "font",   "name": "h3_font", "label": "Heading 3", "load_external_fonts": true,
        "default": { "font": "Plus Jakarta Sans", "font_set": "GOOGLE", "fallback": "sans-serif", "size": 36, "size_unit": "px", "color": "#201600", "variant": "700" } },
      { "type": "font",   "name": "h4_font", "label": "Heading 4", "load_external_fonts": true,
        "default": { "font": "Plus Jakarta Sans", "font_set": "GOOGLE", "fallback": "sans-serif", "size": 28, "size_unit": "px", "color": "#201600", "variant": "600" } },
      { "type": "font",   "name": "h5_font", "label": "Heading 5", "load_external_fonts": true,
        "default": { "font": "Plus Jakarta Sans", "font_set": "GOOGLE", "fallback": "sans-serif", "size": 22, "size_unit": "px", "color": "#201600", "variant": "600" } },
      { "type": "font",   "name": "h6_font", "label": "Heading 6", "load_external_fonts": true,
        "default": { "font": "Plus Jakarta Sans", "font_set": "GOOGLE", "fallback": "sans-serif", "size": 18, "size_unit": "px", "color": "#201600", "variant": "600" } },
      { "type": "number", "name": "heading_line_height", "label": "Headings line height", "default": 1.2, "step": 0.05 },
      { "type": "number", "name": "body_line_height",    "label": "Body line height",     "default": 1.6, "step": 0.05 },
      { "type": "color",  "name": "link_color",          "label": "Link color",            "default": { "color": "#E74C3C", "opacity": 100 } },
      { "type": "color",  "name": "link_hover_color",    "label": "Link hover color",      "default": { "color": "#0D899E", "opacity": 100 } }
    ]
  },

  {
    "type": "group", "name": "primary_button", "label": "Primary Button",
    "children": [
      { "type": "color",  "name": "bg",                 "label": "Background",          "default": { "color": "#E74C3C", "opacity": 100 } },
      { "type": "color",  "name": "text",               "label": "Text",                "default": { "color": "#FFFFFF", "opacity": 100 } },
      { "type": "number", "name": "border_width",       "label": "Border width (px)",   "default": 0, "min": 0 },
      { "type": "color",  "name": "border_color",       "label": "Border color",        "default": { "color": "#E74C3C", "opacity": 100 } },
      { "type": "number", "name": "corner_radius",      "label": "Corner radius (px)",  "default": 8, "min": 0 },
      { "type": "number", "name": "horizontal_padding", "label": "Horizontal padding",  "default": 28, "min": 0 },
      { "type": "number", "name": "vertical_padding",   "label": "Vertical padding",    "default": 14, "min": 0 },
      { "type": "color",  "name": "bg_hover",           "label": "Background (hover)",  "default": { "color": "#0D899E", "opacity": 100 } },
      { "type": "color",  "name": "text_hover",         "label": "Text (hover)",        "default": { "color": "#FFFFFF", "opacity": 100 } }
    ]
  },

  {
    "type": "group", "name": "secondary_button", "label": "Secondary Button",
    "children": [
      { "type": "color",  "name": "bg",                 "label": "Background",          "default": { "color": "#FFFFFF", "opacity": 100 } },
      { "type": "color",  "name": "text",               "label": "Text",                "default": { "color": "#201600", "opacity": 100 } },
      { "type": "number", "name": "border_width",       "label": "Border width (px)",   "default": 1, "min": 0 },
      { "type": "color",  "name": "border_color",       "label": "Border color",        "default": { "color": "#201600", "opacity": 100 } },
      { "type": "number", "name": "corner_radius",      "label": "Corner radius (px)",  "default": 8, "min": 0 },
      { "type": "number", "name": "horizontal_padding", "label": "Horizontal padding",  "default": 28, "min": 0 },
      { "type": "number", "name": "vertical_padding",   "label": "Vertical padding",    "default": 14, "min": 0 },
      { "type": "color",  "name": "bg_hover",           "label": "Background (hover)",  "default": { "color": "#201600", "opacity": 100 } },
      { "type": "color",  "name": "text_hover",         "label": "Text (hover)",        "default": { "color": "#FFFFFF", "opacity": 100 } }
    ]
  },

  {
    "type": "group", "name": "forms", "label": "Forms",
    "children": [
      { "type": "color",  "name": "header_bg",         "label": "Header background",         "default": { "color": "#0D899E", "opacity": 100 } },
      { "type": "color",  "name": "header_text",       "label": "Header text",               "default": { "color": "#FFFFFF", "opacity": 100 } },
      { "type": "color",  "name": "form_bg",           "label": "Form background",           "default": { "color": "#FFFFFF", "opacity": 100 } },
      { "type": "color",  "name": "form_border",       "label": "Form border",               "default": { "color": "#E5E7EB", "opacity": 100 } },
      { "type": "number", "name": "form_radius",       "label": "Form corner radius (px)",   "default": 12, "min": 0 },
      { "type": "number", "name": "form_h_padding",    "label": "Form horizontal padding",   "default": 32, "min": 0 },
      { "type": "number", "name": "form_v_padding",    "label": "Form vertical padding",     "default": 32, "min": 0 },
      { "type": "color",  "name": "label_color",       "label": "Label color",               "default": { "color": "#201600", "opacity": 100 } },
      { "type": "color",  "name": "help_text_color",   "label": "Help text color",           "default": { "color": "#6B7280", "opacity": 100 } },
      { "type": "color",  "name": "field_border",      "label": "Field border",              "default": { "color": "#D1D5DB", "opacity": 100 } },
      { "type": "color",  "name": "field_focus_border","label": "Field focus border",        "default": { "color": "#E74C3C", "opacity": 100 } },
      { "type": "color",  "name": "button_text",       "label": "Button text",               "default": { "color": "#FFFFFF", "opacity": 100 } },
      { "type": "color",  "name": "button_bg",         "label": "Button background",         "default": { "color": "#E74C3C", "opacity": 100 } },
      { "type": "color",  "name": "button_text_hover", "label": "Button text (hover)",       "default": { "color": "#FFFFFF", "opacity": 100 } },
      { "type": "color",  "name": "button_bg_hover",   "label": "Button background (hover)", "default": { "color": "#0D899E", "opacity": 100 } }
    ]
  },

  {
    "type": "group", "name": "tables", "label": "Tables (HTML Table)",
    "children": [
      { "type": "color", "name": "header_bg",   "label": "Header background", "default": { "color": "#201600", "opacity": 100 } },
      { "type": "color", "name": "header_text", "label": "Header text",       "default": { "color": "#FFFFFF", "opacity": 100 } },
      { "type": "color", "name": "body_bg",     "label": "Body background",   "default": { "color": "#FFFFFF", "opacity": 100 } },
      { "type": "color", "name": "body_text",   "label": "Body text",         "default": { "color": "#201600", "opacity": 100 } },
      { "type": "color", "name": "footer_bg",   "label": "Footer background", "default": { "color": "#F9FAFB", "opacity": 100 } },
      { "type": "color", "name": "footer_text", "label": "Footer text",       "default": { "color": "#201600", "opacity": 100 } },
      { "type": "color", "name": "border_color","label": "Border color",      "default": { "color": "#E5E7EB", "opacity": 100 } }
    ]
  },

  {
    "type": "group", "name": "spacing", "label": "Spacing",
    "children": [
      { "type": "number", "name": "vertical_spacing",   "label": "Vertical spacing (px)",         "default": 70,   "min": 0 },
      { "type": "number", "name": "max_content_width", "label": "Maximum content width (px)",     "default": 1300, "min": 320 }
    ]
  },

  {
    "type": "group", "name": "footer", "label": "Footer",
    "children": [
      { "type": "color", "name": "bg",              "label": "Background",          "default": { "color": "#201600", "opacity": 100 } },
      { "type": "color", "name": "text",            "label": "Text",                "default": { "color": "#FFFFFF", "opacity": 100 } },
      { "type": "color", "name": "divider",         "label": "Divider",             "default": { "color": "#3F2F1E", "opacity": 100 } },
      { "type": "color", "name": "bottom_bar_bg",   "label": "Bottom bar bg",       "default": { "color": "#15100B", "opacity": 100 } },
      { "type": "color", "name": "bottom_bar_text", "label": "Bottom bar font",     "default": { "color": "#FFFFFF", "opacity": 100 } }
    ]
  },

  {
    "type": "group", "name": "blog", "label": "Blog",
    "children": [
      { "type": "color",  "name": "header_bg",          "label": "Blog header background", "default": { "color": "#201600", "opacity": 100 } },
      { "type": "color",  "name": "header_text",        "label": "Blog header text",       "default": { "color": "#FFFFFF", "opacity": 100 } },
      { "type": "choice", "name": "layout_no_sidebar",  "label": "Without-sidebar layout",
        "choices": [["grid_2x2","Grid 2 x 2"],["grid_3x3","Grid 3 x 3"],["grid_4x4","Grid 4 x 4"]],
        "default": "grid_3x3" },
      { "type": "color",  "name": "with_sidebar_bg",    "label": "With-sidebar background", "default": { "color": "#F9FAFB", "opacity": 100 } }
    ]
  }
]
```

## CSS variable map (must be exposed in `theme-overrides.css.html`)

```css
:root {
  /* Layout & Structure */
  --aci-container-max:        {{ theme.layout_structure.container.max_width }}px;
  --aci-container-px:         {{ theme.layout_structure.container.horizontal_padding }}px;
  --aci-section-py:           {{ theme.layout_structure.section.vertical_spacing }}px;
  --aci-section-bg:           {{ theme.layout_structure.section.default_bg.color }};
  --aci-grid-gap-x:           {{ theme.layout_structure.grid.column_gap }}px;
  --aci-grid-gap-y:           {{ theme.layout_structure.grid.row_gap }}px;
  --aci-flex-gap:             {{ theme.layout_structure.flex.gap }}px;
  --aci-sidebar-w:            {{ theme.layout_structure.sidebar.width }}px;
  --aci-sidebar-gap:          {{ theme.layout_structure.sidebar.gap }}px;
  --aci-split-gap:            {{ theme.layout_structure.split_5050.gap }}px;
  --aci-split-stack:          {{ theme.layout_structure.split_5050.stack_at }}px;

  /* Global Colors */
  --aci-primary:              {{ theme.global_colors.primary_color.color }};
  --aci-secondary:            {{ theme.global_colors.secondary_color.color }};
  --aci-tertiary:             {{ theme.global_colors.tertiary_color.color }};
  --aci-success:              {{ theme.global_colors.success.color }};
  --aci-warning:              {{ theme.global_colors.warning.color }};
  --aci-danger:               {{ theme.global_colors.danger.color }};
  --aci-info:                 {{ theme.global_colors.info.color }};

  /* Fonts */
  --aci-font-primary:         {{ theme.global_fonts.primary_font.font|default("Plus Jakarta Sans") }}, {{ theme.global_fonts.primary_font.fallback|default("sans-serif") }};
  --aci-font-secondary:       {{ theme.global_fonts.secondary_font.font|default(theme.global_fonts.primary_font.font|default("Plus Jakarta Sans")) }}, {{ theme.global_fonts.secondary_font.fallback|default("sans-serif") }};

  /* Header */
  --aci-header-scrolled-bg:   {{ theme.website_header.header_scrolled_bg.color }};
  --aci-nav-link:             {{ theme.website_header.nav_link_color.color }};
  --aci-child-menu-bg:        {{ theme.website_header.child_menu_bg.color }};
  --aci-logo-max:             {{ theme.website_header.logo_max_height }}px;

  /* Typography */
  --aci-body-color:           {{ theme.typography.body_font.color }};
  --aci-body-size:            {{ theme.typography.body_font.size }}{{ theme.typography.body_font.size_unit|default("px") }};
  --aci-body-line:            {{ theme.typography.body_line_height }};
  --aci-heading-line:         {{ theme.typography.heading_line_height }};
  --aci-h1-size:              {{ theme.typography.h1_font.size }}{{ theme.typography.h1_font.size_unit|default("px") }};
  --aci-h1-color:             {{ theme.typography.h1_font.color }};
  --aci-h2-size:              {{ theme.typography.h2_font.size }}{{ theme.typography.h2_font.size_unit|default("px") }};
  --aci-h2-color:             {{ theme.typography.h2_font.color }};
  --aci-h3-size:              {{ theme.typography.h3_font.size }}{{ theme.typography.h3_font.size_unit|default("px") }};
  --aci-h3-color:             {{ theme.typography.h3_font.color }};
  --aci-h4-size:              {{ theme.typography.h4_font.size }}{{ theme.typography.h4_font.size_unit|default("px") }};
  --aci-h4-color:             {{ theme.typography.h4_font.color }};
  --aci-h5-size:              {{ theme.typography.h5_font.size }}{{ theme.typography.h5_font.size_unit|default("px") }};
  --aci-h5-color:             {{ theme.typography.h5_font.color }};
  --aci-h6-size:              {{ theme.typography.h6_font.size }}{{ theme.typography.h6_font.size_unit|default("px") }};
  --aci-h6-color:             {{ theme.typography.h6_font.color }};
  --aci-link:                 {{ theme.typography.link_color.color }};
  --aci-link-hover:           {{ theme.typography.link_hover_color.color }};

  /* Buttons (primary) */
  --aci-btn-primary-bg:       {{ theme.primary_button.bg.color }};
  --aci-btn-primary-text:     {{ theme.primary_button.text.color }};
  --aci-btn-primary-bw:       {{ theme.primary_button.border_width }}px;
  --aci-btn-primary-border:   {{ theme.primary_button.border_color.color }};
  --aci-btn-primary-radius:   {{ theme.primary_button.corner_radius }}px;
  --aci-btn-primary-px:       {{ theme.primary_button.horizontal_padding }}px;
  --aci-btn-primary-py:       {{ theme.primary_button.vertical_padding }}px;
  --aci-btn-primary-bg-h:     {{ theme.primary_button.bg_hover.color }};
  --aci-btn-primary-text-h:   {{ theme.primary_button.text_hover.color }};

  /* Buttons (secondary) — same shape */
  --aci-btn-secondary-bg:     {{ theme.secondary_button.bg.color }};
  --aci-btn-secondary-text:   {{ theme.secondary_button.text.color }};
  --aci-btn-secondary-bw:     {{ theme.secondary_button.border_width }}px;
  --aci-btn-secondary-border: {{ theme.secondary_button.border_color.color }};
  --aci-btn-secondary-radius: {{ theme.secondary_button.corner_radius }}px;
  --aci-btn-secondary-px:     {{ theme.secondary_button.horizontal_padding }}px;
  --aci-btn-secondary-py:     {{ theme.secondary_button.vertical_padding }}px;
  --aci-btn-secondary-bg-h:   {{ theme.secondary_button.bg_hover.color }};
  --aci-btn-secondary-text-h: {{ theme.secondary_button.text_hover.color }};

  /* Forms */
  --aci-form-header-bg:       {{ theme.forms.header_bg.color }};
  --aci-form-header-text:     {{ theme.forms.header_text.color }};
  --aci-form-bg:              {{ theme.forms.form_bg.color }};
  --aci-form-border:          {{ theme.forms.form_border.color }};
  --aci-form-radius:          {{ theme.forms.form_radius }}px;
  --aci-form-px:              {{ theme.forms.form_h_padding }}px;
  --aci-form-py:              {{ theme.forms.form_v_padding }}px;
  --aci-form-label:           {{ theme.forms.label_color.color }};
  --aci-form-help:            {{ theme.forms.help_text_color.color }};
  --aci-form-field-border:    {{ theme.forms.field_border.color }};
  --aci-form-field-focus:     {{ theme.forms.field_focus_border.color }};
  --aci-form-btn-text:        {{ theme.forms.button_text.color }};
  --aci-form-btn-bg:          {{ theme.forms.button_bg.color }};
  --aci-form-btn-text-h:      {{ theme.forms.button_text_hover.color }};
  --aci-form-btn-bg-h:        {{ theme.forms.button_bg_hover.color }};

  /* Tables */
  --aci-tbl-header-bg:        {{ theme.tables.header_bg.color }};
  --aci-tbl-header-text:      {{ theme.tables.header_text.color }};
  --aci-tbl-body-bg:          {{ theme.tables.body_bg.color }};
  --aci-tbl-body-text:        {{ theme.tables.body_text.color }};
  --aci-tbl-footer-bg:        {{ theme.tables.footer_bg.color }};
  --aci-tbl-footer-text:      {{ theme.tables.footer_text.color }};
  --aci-tbl-border:           {{ theme.tables.border_color.color }};

  /* Spacing */
  --aci-vspace:               {{ theme.spacing.vertical_spacing }}px;
  --aci-max-w:                {{ theme.spacing.max_content_width }}px;

  /* Footer */
  --aci-footer-bg:            {{ theme.footer.bg.color }};
  --aci-footer-text:          {{ theme.footer.text.color }};
  --aci-footer-divider:       {{ theme.footer.divider.color }};
  --aci-footer-bottom-bg:     {{ theme.footer.bottom_bar_bg.color }};
  --aci-footer-bottom-text:   {{ theme.footer.bottom_bar_text.color }};

  /* Blog */
  --aci-blog-header-bg:       {{ theme.blog.header_bg.color }};
  --aci-blog-header-text:     {{ theme.blog.header_text.color }};
  --aci-blog-with-sidebar-bg: {{ theme.blog.with_sidebar_bg.color }};
}
```
