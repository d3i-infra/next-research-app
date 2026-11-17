# The Next Research App: website

Jekyll site with a Sveltia CMS admin at `/admin/`. No theme: plain HTML layouts
and one stylesheet.

## Develop locally

```sh
bundle config set --local path vendor/bundle
bundle install
bundle exec jekyll serve      # http://localhost:4000
```

The CMS can edit your working copy directly: open `http://localhost:4000/admin/`
in a Chromium browser and choose "Work with Local Repository".

## Where things are

| What | Where |
|------|-------|
| Page content (all of it is front matter) | `index.md`, `_pages/*.md` |
| Lists shared across pages | `_data/funders.yml` |
| Site name, logo, footer, "Have a study in mind?" banner | `_data/settings.yml` |
| Navigation (developer-owned) | `_data/navigation.yml` |
| Page structure | `_layouts/` (one per page type), `_includes/` |
| **All styling** | `assets/css/main.css` |
| CMS config | `admin/config.yml` (validated in CI by `cms/validate-config.mjs`) |
| Wireframes the pages were built from | `docs/wireframes/` |

Empty images render as a labelled grey placeholder, so the site works before
any images are uploaded.

See `cms/maintainer-notes.md` for the CMS, branches and deploys.
