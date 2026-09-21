# Project Context and Decisions

## Goal

Move away from printed and handwritten recipe cards to a git-based recipe
collection published on GitHub Pages, so recipes can be edited without
reprinting.

## Decision: Cooklang only (2026-09-20)

Chose Cooklang + CookCLI over a custom single-file HTML site. There is no
custom template to maintain, and CookCLI provides search, scaling, timers,
shopping lists, and a static site generator.

Verified with CookCLI 0.36.0:

- `cook doctor` passes: recipes valid, every ingredient present in
  `aisle.conf`.
- `cook build web --base-url /recipes/` produces a self-contained static
  site (about 736 KB for one recipe) with correct subpath links.
- The "Prep Ahead grouped by component" idea works with one Cooklang
  section per component (`== Prep Ahead: For the Base ==`). Both the steps
  and the ingredient list group under those names. Markdown bold headings
  do not work: they render literally as fake numbered steps.
- A blank line between steps is required. Without it, Cooklang merges a
  whole section into one paragraph.

Trade-offs accepted:

- The static site has no shopping list, pantry, editing, scale control, or
  aisle grouping. The aisle grouping only shows up in the local server's
  shopping list.
- Ingredient lists are alphabetical within a section, not in order of use.
- The HTML prototype's step checkboxes and progress bar have no confirmed
  equivalent. Cook mode steps through the sections instead.

## Hosting

GitHub Pages serving the static build. Decided; not yet deployed.
Needs a GitHub Actions workflow that installs CookCLI, builds with
`--base-url /recipes/`, and publishes `_site/`. Pages is free for public
repos. Private-repo Pages likely needs a paid GitHub plan (unchecked).

Alternatives if the shopping list matters:

- Run `cook server --host` on an always-on machine (put it behind Tailscale
  or basic auth; the server documents no login of its own). Edits made in
  its web UI change files on that machine, not in git.
- The Cook mobile app is free and has shopping lists. It reads recipes from
  iCloud Drive or a synced folder, not from GitHub. Cook Cloud sync
  (EUR 4.99/month) is only needed for automatic sync and is not needed here.

## Importing recipes

Claude does the import through the `recipe-import` skill. The skill tries
`cook import --skip-conversion <url>` for extraction (free, no API key, runs
locally) and falls back to a direct page fetch. Some sites return HTTP 403
to the CLI; that is not worked around.

`cook import` without `--skip-conversion` calls an LLM with the user's own
API key. It is not used here. No shared or crowdsourced recipe database was
found in the docs; the source was not audited.

## Preserved from earlier design work

The PDF card and the custom HTML prototype are retired and not in this repo.
What carries over is the aisle categories, which are `aisle.conf`
sections. The colors below belonged to the PDF and HTML versions and only
matter if the site is ever themed:

| Category | Color |
|---|---|
| Produce | `#7A9D4E` |
| Pantry & Canned | `#B08B5B` |
| Spices & Seasoning | `#C1652F` |
| Dairy & Alt-Dairy | `#5C7FA6` |
| Accent (headers, rules) | `#3F6C51` |

The original notes are in git history (commit "Add original project
context notes").
