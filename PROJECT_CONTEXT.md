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

## Ingredient tagging (2026-09-20)

Each ingredient gets a quantity exactly once, where it is measured or first
used (usually Prep Ahead). On its first mention in each later step it is
tagged with no quantity (`@carrots{}`), so Cooking Day steps show which
ingredients they use. Partial amounts ("one can of") stay as plain text.
This replaces an earlier approach where Cooking Day mentioned
prep-measured ingredients as untagged prose.

Tested with CookCLI 0.36.0:

- Quantity-less tags leave shopping-list totals unchanged, including for
  split-use ingredients (2 cans of chickpeas, one used in each of two steps).
- Repeating a quantity adds it up: `@carrots{4}` twice gave 8. Repeating
  partial amounts gave "2 can, 2 cans", because can and cans do not merge.
- `@&name` (the Cooklang reference modifier) is not supported here. It
  creates a separate ingredient named "&name", which also lands on the
  shopping list.
- The spec is silent on duplicate ingredients, so this is CookCLI
  behavior. Other Cooklang apps were not tested.

Trade-off: the Cooking Day ingredient list is now a long list of names
without quantities that repeats names from the Prep Ahead lists.

## Metadata and aisle aliases (2026-09-21)

Checked against Cooklang's conventions page and adjusted two things.

- **Source metadata:** the conventions list `author`, `source` (a URL or
  text), and `source.name`. The soup had site, author, and URL in one
  `source` string, so it now uses three separate keys. Tested with
  CookCLI: `author` plus a URL `source` plus a flat `source.name` all parse
  and show up. A nested `source:` block showed only the name in the CLI, so
  the flat form is used.
- **Aisle aliases:** the earlier rule "names must match exactly" came from
  a `cook doctor` complaint and was stricter than Cooklang requires.
  `aisle.conf` supports aliases with `|` (`olive oil | extra virgin olive
  oil`). Tested: `cook doctor` passes for the alias, and the shopping list
  shows the first name.

## Hosting

GitHub Pages serving the static build at `https://mijogu.github.io/recipes/`.
Pages is free for public repos. Private-repo Pages likely needs a paid
GitHub plan (unchecked).

`.github/workflows/site.yml` does the work:

- Every pull request installs CookCLI, runs the recipe checks, and builds
  the site. Pushes to `main` (and manual runs) then publish it. The
  `github-pages` environment only accepts deployments from `main`, so a PR
  can never publish.
- CookCLI is pinned to one release and verified against its SHA-256 (from
  the release API) before use. The CookCLI docs give no CI guidance and show
  a manual push to a `gh-pages` branch; this uses GitHub's official Pages
  actions instead, so there is no deploy branch.
- `cook doctor` exits 0 even when it finds problems, so it is not a usable
  gate. The workflow runs `cook doctor validate --strict` (exits 1 on bad
  frontmatter) and fails on the "not found in aisle" message from
  `cook doctor aisle`.
- The site base path comes from `actions/configure-pages`, so a custom
  domain later needs no change here.
- The published site includes each recipe's raw `.cook` file next to its
  page.

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
