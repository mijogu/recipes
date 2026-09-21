# Recipes

A personal recipe collection written in [Cooklang](https://cooklang.org)
plain-text files. Recipes are edited in git, so a change never means
reprinting a card.

The plan is to publish the static site on GitHub Pages at
`https://mijogu.github.io/recipes/`. It is not deployed yet.

## Preview locally

Install [CookCLI](https://cooklang.org/cli/) (`brew install cookcli`), then
from the repo root:

```bash
cook server
```

Open http://localhost:9080. The local server has everything: shopping list,
pantry, editing, and a scale control. The hosted static site has only the
recipe pages, search, and Cook mode.

## Add a recipe

**With Claude Code:** give it a recipe URL. The `recipe-import` skill
(`.claude/skills/recipe-import/SKILL.md`) writes `recipes/<slug>.cook`,
updates `config/aisle.conf`, runs `cook doctor`, and opens a pull request.

**By hand:**

1. Create `recipes/<slug>.cook`. Group prep work by component with sections
   like `== Prep Ahead: For the Base ==`, then `== Cooking Day ==`, and put a
   blank line between steps. Give each ingredient a quantity once
   (`@carrots{4}`); later mentions have none (`@carrots{}`).
2. Add any new ingredient to `config/aisle.conf` under its aisle category.
   A name variant goes on the existing line as an alias
   (`olive oil | extra virgin olive oil`), and comments use `--`.
3. Run `cook doctor`.
4. Open a pull request. Do not push to `main`.

## Layout

| Path | What it is |
|---|---|
| `recipes/` | One `.cook` file per recipe |
| `config/aisle.conf` | Ingredient -> aisle category |
| `.claude/skills/recipe-import/` | The URL-import skill |
| `CLAUDE.md` | Conventions and commands for Claude Code |
| `PROJECT_CONTEXT.md` | Why Cooklang, what was tested, what was dropped |

## Build the static site

```bash
cook build web --base-url /recipes/
```

Output goes to `_site/`, which is gitignored.
