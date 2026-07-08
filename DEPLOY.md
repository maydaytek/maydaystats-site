# Deploying the site: GitHub + Cloudflare Pages

## 1. Render locally first

Before pushing anything, make sure the site actually renders on your
machine (this is also what populates `_freeze/`, which the deploy build
depends on):

```bash
quarto render
```

Confirm `_site/` was generated and looks right (spot check with
`quarto preview`), then commit everything, including `_freeze/`:

```bash
git init
git add .
git commit -m "Initial site: baseball pipeline validation post"
```

## 2. Push to GitHub

Create a new repository on GitHub (e.g. `maydaystats-site`), then:

```bash
git remote add origin https://github.com/<your-username>/maydaystats-site.git
git branch -M main
git push -u origin main
```

## 3. Connect Cloudflare Pages

In the Cloudflare dashboard: Workers & Pages -> Create -> Pages ->
Connect to Git -> select `maydaystats-site`.

Build settings:

- **Framework preset:** None
- **Build command:**
  ```bash
  curl -LO https://github.com/quarto-dev/quarto-cli/releases/download/v1.6.39/quarto-1.6.39-linux-amd64.tar.gz && mkdir -p /tmp/quarto-install && tar -xzf quarto-1.6.39-linux-amd64.tar.gz -C /tmp/quarto-install && export PATH="/tmp/quarto-install/quarto-1.6.39/bin:$PATH" && quarto render
  ```
- **Build output directory:** `_site`

That build command downloads Quarto's portable (no-install-needed)
distribution and extracts it to `/tmp/quarto-install` - deliberately
*outside* the repo/project directory. Extracting it into the project
directory itself (e.g. with a bare `tar -xzf ... .`) pollutes the
project tree with Quarto's own bundled example templates, and
`quarto render` will try to render those as if they were your content
and fail with an "Invalid Shortcode" error. Keeping the Quarto install
out of the project directory avoids that entirely.

Because `freeze: auto` is set, this render reuses the `_freeze/`
output committed in step 1 for any post whose source hasn't changed,
instead of re-executing its Python or querying BigQuery - so this build
needs no GCP credentials for those posts. Note:
Cloudflare Pages auto-detects `requirements.txt` at the repo root and
will `pip install` it regardless, even though frozen renders don't
need it - this costs a bit of build time but doesn't affect
correctness. If you add a new post without rendering locally first,
the render step will fail (nothing frozen to fall back on), which is
your signal to go back and run `quarto render` locally before pushing.

## 4. Point the domain at it

In Cloudflare Pages -> your project -> Custom domains -> add
`maydaystats.com`. Since the domain's DNS is already on Cloudflare,
this is usually a one-click "activate."

## 5. Every update after this

```bash
# write/edit a post
quarto render          # regenerates _site/ and updates _freeze/
git add .
git commit -m "New post: ..."
git push
```

Cloudflare Pages rebuilds automatically on every push to `main`.
