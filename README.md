# maydaystats.com

The Quarto source for the maydaystats site: baseball, hockey, and
volleyball analytics, each post backed by a real data pipeline.

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
deep-dives (this is what makes a post show up on the Featured Research
page).

## Rendering and the freeze workflow

This project uses Quarto's `freeze: true` (set in `_quarto.yml`). That
means:

- Code chunks only execute when you run `quarto render` or `quarto preview`
  locally, where you have real GCP credentials.
- The computed output gets cached into a `_freeze/` folder, which gets
  committed to git along with your `.qmd` source.
- The Cloudflare Pages build (see `DEPLOY.md`) reuses that frozen output
  instead of re-running any Python or querying BigQuery - so the deploy
  step never needs cloud credentials, and stays fast.

Practical implication: **always run `quarto render` locally and commit
the resulting `_freeze/` changes before pushing a new or updated post.**
If you forget, Cloudflare's build will fail (or silently render stale
content) because there's nothing frozen to fall back on for new pages.

See `DEPLOY.md` for pushing to GitHub and wiring up Cloudflare Pages.
