---
name: recipe-import
description: Converts a recipe URL into this repo's Cooklang template — aisle-categorized ingredients and a component-grouped Prep Ahead section. Use when given a recipe URL to import into the site.
---

# Recipe Import

When given a recipe URL (directly, or wrapped in a routine-fire-payload
block), do the following:

1. Fetch the URL and extract the recipe: title, servings, prep/cook time,
   ingredients with quantities, and the method. Try
   `cook import --skip-conversion <url>` first — it needs no API key and
   extracts the recipe from the page's structured data as frontmatter
   plus plain ingredient and step text. If it fails (e.g. HTTP 403 from
   bot protection), fetch the page directly instead. If that is blocked
   too, stop and tell the user; do not try to work around the block.
2. Write a new `.cook` file at `recipes/<slug>.cook` (slug = kebab-case of
   the title):
   - YAML frontmatter: title, servings, prep time, cook time, author,
     source (the recipe URL), source.name (the site name), tags. Keep
     these as separate keys; do not combine site, author, and URL into
     one `source` string.
   - Prep Ahead: steps that can be done in advance (chopping, measuring,
     marinating), grouped by recipe component as separate sections named
     `== Prep Ahead: For the Base ==`, `== Prep Ahead: For the Sauce ==`,
     `== Prep Ahead: For Serving ==`, etc., reflecting how ingredients
     are actually used in the method — never a flat list. Cooklang has
     no nested sections and does not render markdown bold (`**...**`
     shows up literally, as a fake numbered step), so use one `==`
     section per component.
   - `== Cooking Day ==` section: the remaining sequential steps
   - Put a blank line between steps. Consecutive lines are merged into
     one step, which renders as a single dense paragraph.
   - Use `@ingredient{quantity%unit}`, `#cookware{}`, `~{quantity%minutes}`
     per Cooklang syntax
   - Give each ingredient a quantity exactly once, where it is measured or
     first used (usually Prep Ahead). Tag it again on its first mention in
     every later step with no quantity, e.g. `@carrots{}`, and keep partial
     amounts as plain text ("one can of @chickpeas{}"). Repeated
     quantities are added up on the shopping list, and unit spellings like
     can/cans are not merged. Do not use `@&name`; it is read as a
     separate ingredient.
   - `--` starts a comment, and everything after it is dropped from the
     rendered recipe. Put tips in the step text or in a `>` note.
3. Update `config/aisle.conf`: add any ingredient not already listed,
   under the matching category (`[produce]`, `[pantry & canned]`,
   `[spices & seasoning]`, `[dairy & alt-dairy]`). Every ingredient name
   in the `.cook` file must appear there, either as the listed name or as
   an alias after a `|`, e.g. `olive oil | extra virgin olive oil`. If a
   recipe uses a variant of an ingredient that is already listed, add an
   alias to that line instead of a new entry; the first name is the one
   shown on the shopping list. Comments in this file use `--`, not `#`.
4. Run `cook doctor` and fix anything it flags. Then run
   `cook recipe recipes/<slug>.cook` and confirm each step is numbered
   separately under its section heading.
5. Commit the `.cook` file and updated `aisle.conf` to a new branch named
   `claude/import-<slug>`, and open a PR titled "Add recipe: <title>".
   Do not push directly to main.
