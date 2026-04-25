# SiteMango

This repository is configured for automatic deployment to **GitHub Pages**.

## Deploy flow

- Push to the `main` branch to trigger deployment.
- Or run the workflow manually from **Actions → Deploy static site → Run workflow**.

The workflow lives at:

- `.github/workflows/deploy.yml`

## Local preview

Open `index.html` directly in a browser, or run a simple local server:

```bash
python3 -m http.server 8000
```

Then visit `http://localhost:8000`.
