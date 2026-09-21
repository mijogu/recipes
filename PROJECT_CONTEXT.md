# Recipe Collection Project — Context Summary

## Goal
Move away from printed/handwritten recipe cards toward a git-based recipe
collection published via GitHub Pages, so recipes can be edited without
reprinting. Recipes should have interactive checkboxes for steps as you cook.

## Path so far

1. Started with a **printable PDF recipe card** (fold-in-half design) as a
   physical recipe box replacement.
2. Decided printing was too inflexible → pivoted to a **GitHub repo + GitHub
   Pages** site instead, so recipes update without reprinting.
3. Explored two implementation paths and built a working example of each:
   - **Option A — Cooklang/CookCLI**: adopt the existing plain-text recipe
     ecosystem (`.cook` files, git-native, static site generator built in).
   - **Option B — Custom HTML**: single-file HTML/JS site, JSON-like recipe
     data, checkboxes backed by `localStorage`. Keeps full control of the
     custom design.
4. Currently **evaluating Option A** — installed CookCLI locally and
   validated the recipe file against the real parser (see below). Plan is to
   run `cook server` next to see the rendered recipe page before deciding
   between A and B.

## Design system (established in the PDF, carried into both options)

- **Ingredient categories** (color-coded, grocery-aisle style):
  - Produce — green `#7A9D4E`
  - Pantry & Canned — tan `#B08B5B`
  - Spices & Seasoning — rust `#C1652F`
  - Dairy & Alt-Dairy — blue `#5C7FA6`
- **Accent color** `#3F6C51` for headers/rules (used across the PDF and HTML
  versions).
- Ingredient list order = order of use (not alphabetical).
- **"Prep Ahead" tasks are grouped by recipe component** (e.g. "For the
  Base," "For the Puree," "For Serving") rather than a flat checklist — this
  is the one custom structural idea that doesn't have a native equivalent in
  Cooklang, so it's implemented there as named sections (`== Prep Ahead ==`,
  `== Cooking Day ==`).

## Files produced (test recipe: Lemon Chickpea Soup, from rainbowplantlife.com)

| File | Purpose |
|---|---|
| `lemon_chickpea_soup.pdf` | Printable fold-in-half card (superseded by the digital plan, kept as the design reference) |
| `recipe_card_template.py` | Python/reportlab script that generated the PDF — data-driven, reusable per recipe |
| `lemon-chickpea-soup.html` | Working Option B prototype — checkboxes, progress bar, `localStorage` persistence, same color/grouping system |
| `lemon-chickpea-soup.cook` | Option A — same recipe in Cooklang syntax, using `==` sections for Prep Ahead / Cooking Day |
| `aisle.conf` | Cooklang's native ingredient-category config (produce/pantry/spice/dairy) — the closest built-in equivalent to the color-coded aisle grouping |

## Validation already done (real CookCLI v0.30.0, downloaded and run directly)

- `cook recipe "lemon-chickpea-soup.cook"` — parsed correctly; both `==`
  sections rendered as intended ("Prep Ahead" / "Cooking Day"), ingredients
  aggregated correctly across sections.
- `cook doctor` caught two real issues, now fixed:
  1. `aisle.conf` listed **"extra virgin olive oil"** but the recipe calls it
     **"olive oil"** — names must match exactly between files.
  2. `aisle.conf` used `#` for comments, which isn't valid there — Cooklang
     config files use `--` for comments, not `.cook` files' rules.
- `cook build web` succeeded — generated a real static site (`index.html` +
  rendered recipe page) confirming the GitHub Pages publish pipeline works
  end-to-end with this recipe file.

## Immediate next step

Run `cook server` locally (pointed at the folder containing
`lemon-chickpea-soup.cook` and `config/aisle.conf`) and view the rendered
recipe at `localhost:9080` to see the actual styled output before deciding
whether Cooklang's native rendering is good enough, or whether to go with
the custom HTML build instead.

## Open decision

**Option A (Cooklang) vs. Option B (custom HTML)** — not yet finalized.
Cooklang gets shopping lists, scaling, timers, and mobile apps for free but
doesn't natively support the "Prep Ahead grouped by component" idea (worked
around via sections). Custom HTML preserves the exact design already built
but means owning all future maintenance.
