# wingflight-docs

Source for [doc.wingflight.org](https://doc.wingflight.org), built with
[MkDocs Material](https://squidfunk.github.io/mkdocs-material/).

## Local preview

```
pip install -r requirements.txt
mkdocs serve
```

Then open http://127.0.0.1:8000/.

## Structure

All content lives under `docs/` as plain Markdown. The site navigation is
defined in `mkdocs.yml` -- add new pages there as well as under `docs/` for
them to appear in the nav.

## Deployment

Pushing to `main` builds the site and publishes it to the `gh-pages` branch
via `.github/workflows/deploy.yml`, served at
[doc.wingflight.org](https://doc.wingflight.org).
