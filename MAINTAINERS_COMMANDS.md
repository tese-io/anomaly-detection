## 0) First-time setup (once per machine)

```bash
python -m venv .venv
# mac/linux
. .venv/bin/activate
# windows (powershell)
. .venv/Scripts/Activate.ps1

pip install --upgrade pip
pip install mkdocs mkdocs-material
```

---

## 1) Make changes to docs

* Edit or add Markdown files under `docs/`
* Update navigation in `mkdocs.yml` if you add/rename files

**Common nav edit (example):**

```yaml
nav:
  - Home: index.md
  - Architecture: architecture.md
  - Flows:
      - "Text Q&A Flow": flows/text-qa.md
      - "Document Analysis Flow": flows/document-analysis.md
  - "Scoring & Output": scoring.md
  - Implementation: implementation.md
  - Roadmap: roadmap.md
```

---

## 2) Local preview (live reload)

```bash
mkdocs serve
# Open the printed URL (usually http://127.0.0.1:8000)
```

> Tip: If you want strict checking (fail on nav/link errors):

```bash
mkdocs build --strict
```

---

## 3) Commit & push changes (source branch = main)

```bash
git add .
git commit -m "<message>"
git push origin main
```

---

## 4) Publish the site (manual deploy)

```bash
# Build and push to gh-pages (the Pages branch)
mkdocs gh-deploy --force
```

* After this, site is at: `https://tese-io.github.io/anomaly-detection/`

---

## 5) Typical workflows

### A) Add a new page

```bash
# 1) Create the file
echo "# My New Page" > docs/new-page.md

# 2) Add to nav in mkdocs.yml (example)
# nav:
#   - My New Page: new-page.md

# 3) Preview, commit, deploy
mkdocs build --strict
git add .
git commit -m "Add new page to docs + nav"
git push origin main
mkdocs gh-deploy --force
```

### B) Add a new subsection page (nested folder)

```bash
mkdir -p docs/flows
echo "# API Flow" > docs/flows/api-flow.md

# Add under "Flows" in mkdocs.yml:
# - Flows:
#     - API Flow: flows/api-flow.md

mkdocs build --strict
git add .
git commit -m "Add flows/api-flow page + nav"
git push origin main
mkdocs gh-deploy --force
```

### C) Rename or move a page

```bash
git mv docs/architecture.md docs/system-architecture.md

# Update mkdocs.yml:
# - Architecture: system-architecture.md

mkdocs build --strict
git add .
git commit -m "Rename architecture page + update nav"
git push origin main
mkdocs gh-deploy --force
```

### D) Remove a page

```bash
git rm docs/roadmap.md

# Remove it from nav in mkdocs.yml
mkdocs build --strict
git add .
git commit -m "Remove roadmap page"
git push origin main
mkdocs gh-deploy --force
```

---

## 6) Clean up template leftovers (only if needed)

```bash
# Remove template’s sample files if they ever reappear
git rm -r docs_sample 2>/dev/null || true
git rm mkdocs-sample.yml 2>/dev/null || true
git commit -m "Remove template sample docs/config"
git push origin main
mkdocs gh-deploy --force
```

---

## 7) Optional: Auto-deploy on every push (GitHub Actions)

If you prefer not to run `mkdocs gh-deploy` locally, add this workflow:

**`.github/workflows/gh-pages.yml`**

```yaml
name: Deploy MkDocs
on:
  push:
    branches: [ main ]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.x'
      - run: pip install mkdocs mkdocs-material
      - run: mkdocs build --strict
      - uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./site
```

Then set **Settings → Pages → Source → GitHub Actions** (one time).

---

## 8) Roll back a bad deploy

```bash
# Revert the offending commit(s) on main, then redeploy
git log --oneline
git revert <bad_commit_sha>
git push origin main
mkdocs gh-deploy --force
```

---

## 9) Refresh CDN cache (client side)

* Hard refresh the browser: **Ctrl/Cmd+Shift+R**
* Wait ~1–2 minutes if GitHub Pages cache lags

---

## 10) Quick sanity checks

```bash
# Validate config & links
mkdocs build --strict

# List what will be published
ls -1 site

# Confirm branches
git branch -a
#  * main         (source)
#    remotes/origin/gh-pages  (published)
```

---

## 11) Common fixes

* **Nav item 404** → path mismatch in `mkdocs.yml` → fix path; run `mkdocs build --strict`.
* **Theme not applied** → `pip install mkdocs-material` and ensure:

  ```yaml
  theme:
    name: material
  ```
* **Pages not updating** → Re-deploy `mkdocs gh-deploy --force`; hard refresh; check Settings → Pages points to `gh-pages` (or to **GitHub Actions** if using the workflow).
* **YAML parse errors** → use spaces (no tabs), quote strings with `:` or `&`, and keep 2-space indents.
