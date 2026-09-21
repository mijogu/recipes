---
name: recipe-import
description: Converts a recipe URL into this repo's Cooklang template — aisle-categorized ingredients and a component-grouped Prep Ahead section. Use when given a recipe URL to import into the site.
---

# Recipe Import

When given a recipe URL (directly, or wrapped in a routine-fire-payload
block), do the following:

1. Fetch the URL and extract the recipe: title, servings, prep/cook time,
   ingredients with quantities, and the method.
2. Write a new `.cook` file at `recipes/<slug>.cook` (slug = kebab-case of
   the title):
   - YAML frontmatter: title, servings, prep time, cook time, source
     (site name + URL), tags
   - `== Prep Ahead ==` section: steps that can be done in advance
     (chopping, measuring, marinating), grouped under bolded component
     sub-headings that reflect how ingredients are actually used in the
     method (e.g. "For the Base," "For the Sauce," "For Serving") — never
     a flat list
   - `== Cooking Day ==` section: the remaining sequential steps
   - Use `@ingredient{quantity%unit}`, `#cookware{}`, `~{quantity%minutes}`
     per Cooklang syntax
3. Update `config/aisle.conf`: add any ingredient not already listed,
   under the matching category (`[produce]`, `[pantry & canned]`,
   `[spices & seasoning]`, `[dairy & alt-dairy]`). Names must match the
   `.cook` file exactly. Comments in this file use `--`, not `#`.
4. Run `cook doctor` and fix anything it flags.
5. Commit the `.cook` file and updated `aisle.conf` to a new branch named
   `claude/import-<slug>`, and open a PR titled "Add recipe: <title>".
   Do not push directly to main.
