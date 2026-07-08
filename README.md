# maydaystats.com

The Quarto source for the maydaystats site: baseball, hockey, and
volleyball analytics, most posts backed by a real data pipeline (any
exception says so explicitly in the post itself).

## Local setup

1. Install Quarto CLI (https://quarto.org/docs/get-started/) - on macOS:
   ```bash
   brew install --cask quarto
   ```
2. Create a Python environment and install the authoring dependencies:
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt
   ```
3. Authenticate to GCP so posts that query BigQuery can run locally:
   ```bash
   gcloud auth application-default login
   gcloud config set project maydaystats
   ```
4. Preview the site with live reload:
   ```bash
   quarto preview
   ```

## Adding a new post

Create a new folder under `posts/<sport>/<date>-<slug>/index.qmd`. Copy
the frontmatter shape from `posts/baseball/2026-07-08-first-pitches/index.qmd`
- `categories:` should include the sport, and `Featured` for flagship
deep-dives, for documentation and filtering purposes.

Note that the `Featured` category tag alone does **not** put a post on
the Featured Research page. `research.qmd`'s listing uses an explicit
`contents:` list of post paths rather than a category filter, because
Quarto's listing `categories: true` only adds a clickable filter widget,
it doesn't restrict the default view to a given tag. To actually feature
a new post, add its path to the `listing: contents:` list in
`research.qmd` as well as tagging it `Featured`.

## Rendering and the freeze workflow

This project uses Quarto's `freeze: auto` (set in `_quarto.yml`). That
means:

- A document's code only executes when you run `quarto render` locally
  (where you have real GCP credentials), and only if Quarto detects its
  own source actually changed since the last time it was frozen. This
  check is Quarto's own change-detection, not a file timestamp, so a
  fresh git checkout doesn't fool it into re-running everything.
- Unchanged documents keep reusing their existing `_freeze/` output.
- The computed output gets cached into `_freeze/`, which gets committed
  to git along with your `.qmd` source.
- The Cloudflare Pages build (see `DEPLOY.md`) reuses frozen output for
  anything that hasn't changed, so the deploy step never needs cloud
  credentials for those posts, and stays fast.

Practical implication: **always run `quarto render` locally and commit
the resulting `_freeze/` changes before pushing a new or updated post.**
If you forget, Cloudflare's build will fail for that post (nothing frozen
to fall back on).

Don't set `freeze: true` here instead. It means "never re-render during
a project render, full stop," it does not check whether the source
changed, so editing an existing post's code silently keeps serving the
old output until you manually delete that post's `_freeze/` subfolder.
That's a real trap: `quarto render` will report success and you'll see
no error, the page will just quietly still show the old numbers. If a
render ever seems to have "no effect" after an edit, check whether
`_freeze/<path-to-post>/index/execute-results/html.json` actually
changed (`git status` after rendering should show it as modified); if it
didn't, something is preventing re-execution and is worth digging into
before assuming the fix didn't work.

If you want to force a specific post to re-execute regardless of
freeze, render it directly rather than the whole project, e.g.
`quarto render posts/hockey/2026-07-08-year-in-review/index.qmd`.
Incremental renders like this always execute their code, freeze only
governs full project renders (`quarto render` with no arguments).

See `DEPLOY.md` for pushing to GitHub and wiring up Cloudflare Pages.
