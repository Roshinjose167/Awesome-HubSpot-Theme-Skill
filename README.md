<p align="center">
  <img src="banner.svg" alt="Awesome HubSpot Theme Skill — works in Claude Code, Codex, Antigravity, Cursor, GitHub Copilot, OpenCode" width="100%">
</p>

<h1 align="center">Awesome HubSpot Theme Skill</h1>

<p align="center">
  <em>One skill, six agents.</em> Turn a one-line brief like
  <code>create hubspot theme for a pharmacy company</code>
  into a full, production-ready HubSpot CMS theme — fields.json, modules, centralized CSS/JS, the works.
</p>

<p align="center">
  <strong>Author:</strong> Roshin Jose ·
  <a href="mailto:roshinjose1@gmail.com">roshinjose1@gmail.com</a> ·
  <a href="tel:+919645228131">+91 96452 28131</a><br>
  <strong>Version:</strong> 1.1.0  ·
  <strong>Works in:</strong> Claude Code · OpenAI Codex · Google Antigravity · Cursor · GitHub Copilot · OpenCode
  <em>(and any <a href="https://agents.md">agents.md</a>-compatible agent — Cline, Aider, etc.)</em>
</p>

---

A single AI skill that turns a one-line brief into a full HubSpot CMS theme — `theme.json`, a complete `fields.json` global settings panel, centralized `main.css` / `main.js`, and a catalog of ~50 reusable modules. Cross-tool: works in **Claude Code**, **Google Antigravity**, and **OpenAI Codex**.

## What this skill does

You type a brief like:

```
create hubspot theme for a pharmacy company
```

and the agent produces a production-ready theme with:

- Complete **Theme Settings** panel exposing every brand token an editor would touch (colors, typography, header, buttons, forms, tables, spacing, footer, blog, layout & structure)
- A `:root` CSS-variable layer driven entirely by theme tokens — no hardcoded values
- ~50 modules across hero, content, navigation, interactive, forms, data display, media, feedback
- Buttons that reuse a single global Primary/Secondary style — never redefined per-module
- Mobile-first stacking rules baked in
- Validation checklist before the agent reports "done"

The full instruction set is in [`SKILL.md`](SKILL.md). Reference docs:

- [`reference/theme-fields-spec.md`](reference/theme-fields-spec.md) — every `fields.json` group with the exact JSON shape
- [`reference/modules-spec.md`](reference/modules-spec.md) — every module, every field
- [`reference/architecture-rules.md`](reference/architecture-rules.md) — CSS/JS centralization, token discipline, button reuse, no-inline-style policy

## Supported tools

| Tool | File you install | Where it goes | Self-contained? |
|------|------------------|---------------|-----------------|
| **Claude Code** | `SKILL.md` (this folder) | symlinked into `~/.claude/skills/hubspot-theme-builder` | ✓ (loads `reference/*.md` on demand) |
| **OpenAI Codex** | [`prompts/codex-system-prompt.md`](prompts/codex-system-prompt.md) | paste as system prompt | ✓ |
| **Google Antigravity** | [`prompts/antigravity-system-prompt.md`](prompts/antigravity-system-prompt.md) | paste as system prompt | ✓ |
| **Cursor** | [`prompts/cursor-rules.mdc`](prompts/cursor-rules.mdc) | copy to `<project>/.cursor/rules/hubspot-theme-builder.mdc` | ✓ |
| **GitHub Copilot** | [`prompts/copilot-instructions.md`](prompts/copilot-instructions.md) | copy to `<project>/.github/copilot-instructions.md` | ✓ |
| **OpenCode** (and any [agents.md](https://agents.md)-compliant agent: Cline, Aider, etc.) | [`prompts/opencode-AGENTS.md`](prompts/opencode-AGENTS.md) | copy to `<project>/AGENTS.md` | ✓ |

All paste-in files inline the full spec, so the receiving agent needs no other files. The Claude Code skill stays modular (loads `reference/*.md` on demand) — when you change `SKILL.md` or `reference/*.md`, regenerate the paste-in copies (see "Updating the skill" below).

## How to use — per tool

### Claude Code

Symlinked once at install time (see "Install" below). After install, run Claude Code in any working directory and type:

```
create hubspot theme for a pharmacy company
```

The trigger phrase in `SKILL.md`'s frontmatter matches and the agent loads the full instruction set automatically. To invoke explicitly: `/hubspot-theme-builder create theme for a pharmacy company`.

### OpenAI Codex

1. Open Codex (codex.com or the Codex CLI).
2. Paste the entire contents of [`prompts/codex-system-prompt.md`](prompts/codex-system-prompt.md) as the system prompt.
3. Send your brief: `create hubspot theme for a pharmacy company`.

### Google Antigravity

1. Open Antigravity, open the agent's system prompt configuration.
2. Paste [`prompts/antigravity-system-prompt.md`](prompts/antigravity-system-prompt.md).
3. Start a new conversation with your brief.

### Cursor

In your HubSpot theme project root:

```bash
mkdir -p .cursor/rules
cp /Users/apple/Documents/OneIMS-Marketplace/aihubspotthemeskill/prompts/cursor-rules.mdc \
   .cursor/rules/hubspot-theme-builder.mdc
```

Cursor picks it up automatically. The `globs` in the file's frontmatter activate the rule when you're editing `theme.json`, `fields.json`, anything inside `*.module/`, or `*.hubl` files. To invoke unconditionally, type `@hubspot-theme-builder` in Cursor chat. Send your brief.

### GitHub Copilot (Copilot Chat in VS Code, JetBrains, Visual Studio, github.com)

In your HubSpot theme repo root:

```bash
mkdir -p .github
cp /Users/apple/Documents/OneIMS-Marketplace/aihubspotthemeskill/prompts/copilot-instructions.md \
   .github/copilot-instructions.md
git add .github/copilot-instructions.md && git commit -m "Add Copilot instructions for HubSpot theme work"
```

Copilot Chat reads `.github/copilot-instructions.md` automatically for every chat in this repo. No further config. Send your brief in Copilot Chat.

### OpenCode / Cline / Aider / any [agents.md](https://agents.md)-compatible agent

In your HubSpot theme project root:

```bash
cp /Users/apple/Documents/OneIMS-Marketplace/aihubspotthemeskill/prompts/opencode-AGENTS.md \
   ./AGENTS.md
```

Run OpenCode (or your agent of choice). It reads `AGENTS.md` automatically. Send your brief.

## Install (one-time, Claude Code only)

```bash
ln -s "/Users/apple/Documents/OneIMS-Marketplace/aihubspotthemeskill" ~/.claude/skills/hubspot-theme-builder
```

Restart Claude Code. Verify with `/skills` — `hubspot-theme-builder` should appear.

The Codex / Antigravity / Cursor / Copilot / OpenCode files don't need a global install — they live alongside the project that uses them, copied in as shown above.

## What inputs the skill collects

On first turn the agent asks (in one short message, then proceeds):

1. **Theme name** — folder/label, lowercase-hyphen.
2. **Industry** — confirms the brief.
3. **Brand colors** — Primary / Secondary / Tertiary. Defaults to `#E74C3C / #0D899E / #201600` if unspecified.
4. **Primary font** — defaults to `Plus Jakarta Sans, sans-serif`.
5. **Constraints** — anything to avoid touching (e.g. existing modules, page templates).

If the user replies "use defaults" or skips, the agent proceeds with the defaults documented in `SKILL.md`.

## Hard rules the agent follows

- **Non-destructive.** Never renames or deletes existing theme files.
- **Theme tokens are the only source of truth.** No hardcoded hex/font/px in modules.
- **CSS/JS centralized** in `main.css` / `main.js`. Modules don't ship `<style>` or `<script>` tags.
- **Buttons reuse global Primary/Secondary.** Modules expose `style: primary | secondary` choice, never their own button styling.
- **Mobile**: media-above-content in any two-column layout.
- **Accessibility baseline**: alt fields, focus states, ≥4.5:1 contrast at default values.

Full rule list is in `SKILL.md` under "Hard rules".

## File map

```
aihubspotthemeskill/
├── SKILL.md                         ← main instruction set (Claude reads this)
├── README.md                        ← this file
├── reference/
│   ├── theme-fields-spec.md         ← exact fields.json JSON for every group
│   ├── modules-spec.md              ← per-module fields + render notes
│   └── architecture-rules.md        ← CSS/JS centralization, tokens, button reuse
└── prompts/
    ├── antigravity-system-prompt.md ← paste into Antigravity (self-contained)
    ├── codex-system-prompt.md       ← paste into Codex (self-contained)
    ├── cursor-rules.mdc             ← copy to <project>/.cursor/rules/
    ├── copilot-instructions.md      ← copy to <project>/.github/copilot-instructions.md
    └── opencode-AGENTS.md           ← copy to <project>/AGENTS.md (works for OpenCode/Cline/Aider)
```

## Updating the skill

Edit `SKILL.md` or any `reference/*.md` and the next conversation picks up the change — no restart needed for Claude Code skill content (only for skill registration).

When you change the spec in `SKILL.md` / reference docs, also update the five inlined copies in `prompts/` — they're snapshots, not symlinks. Each one shares the same body content; only the header wrapper differs (paste-target, file-path comment, etc.). Quick way to keep them in sync:

```bash
cd /Users/apple/Documents/OneIMS-Marketplace/aihubspotthemeskill/prompts

# 1. Edit the canonical body in antigravity-system-prompt.md.
# 2. For each other file, replace the body (everything after the wrapper header)
#    with the new body. The wrappers stay tool-specific.
```

## Versioning

| Version | Date       | Notes                                                                                                            |
|---------|------------|------------------------------------------------------------------------------------------------------------------|
| 1.0.0   | 2026-04-26 | Initial release. Layout & Structure group, ~50 modules, paste-in prompts for Claude / Antigravity / Codex.       |
| 1.1.0   | 2026-04-26 | Added Cursor (`.cursor/rules/*.mdc`), GitHub Copilot (`.github/copilot-instructions.md`), and OpenCode/agents.md. |

## Author

**Roshin Jose**
Email: [roshinjose1@gmail.com](mailto:roshinjose1@gmail.com)
Phone: [+91 96452 28131](tel:+919645228131)

For bug reports, feature requests, or licensing questions, reach out via email.
