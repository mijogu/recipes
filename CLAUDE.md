# Recipes

Personal recipe collection written in Cooklang (`.cook` files) and rendered
by CookCLI. Cooklang is the only format: no custom HTML templates. The
static build is meant for GitHub Pages (not deployed yet).

## Layout

- `recipes/<slug>.cook` - one recipe per file, slug = kebab-case title
- `config/aisle.conf` - ingredient -> aisle category
- `.claude/skills/recipe-import/` - imports a recipe from a URL. Use it for
  any import; its steps live there, not here.
- `PROJECT_CONTEXT.md` - why Cooklang, what was tested, what was dropped

## Conventions

- Prep Ahead is grouped by recipe component: one section per component,
  named `== Prep Ahead: For the Base ==`, `== Prep Ahead: For the Sauce ==`,
  etc., then `== Cooking Day ==`. Cooklang has no nested sections, and
  markdown bold shows up literally, so do not use `**For the Base**`.
- Put a blank line between steps. Consecutive lines merge into one step.
- `aisle.conf` categories: `[produce]`, `[pantry & canned]`,
  `[spices & seasoning]`, `[dairy & alt-dairy]`. Comments use `--`, not `#`.
  Ingredient names must match the `.cook` files exactly.

## Commands

- `cook doctor` - validates recipes and checks every ingredient is in
  `aisle.conf`. Run before every commit.
- `cook recipe recipes/<slug>.cook` - CLI view; shows how steps are split
- `cook server` - local site at http://localhost:9080 with the full feature
  set (shopping list, pantry, edit, scale). It writes `.shopping-list` and
  `.shopping-checked`; both are gitignored.
- `cook build web --base-url /recipes/` - static site into `_site/`

The static build has no shopping list, pantry, editing, scale control, or
aisle grouping. Do not promise those on the hosted site.

## Git workflow

- Never push to `main`. Use a `claude/<topic>` branch and open a PR.
- Commits are signed through 1Password. If a commit fails with
  "1Password: failed to fill whole buffer", unlock 1Password and retry.
  Do not disable signing.

## Known gotchas

- `cook import` can get HTTP 403 from sites with bot protection (e.g.
  rainbowplantlife.com). Fall back to fetching the page directly; do not
  try to evade the block.
- `cook import` without `--skip-conversion` needs the user's own LLM API
  key. The skill uses `--skip-conversion`, which needs none.
