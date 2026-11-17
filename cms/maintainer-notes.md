# CMS & pipeline maintainer notes

Adapted from the datadonation.eu (D3I) setup, which this site reuses as-is.

## First-time setup for the new repository (TODO)

1. **`admin/config.yml`**: set `backend.repo` (`OWNER/REPO`) and `site_url` / `display_url` (staging URL).
2. **`_config.yml`**: set `url` to the production domain. Also the `url:` in `.github/workflows/deploy-production.yml`.
3. **Branches**: create `cms-staging` from `main` (`git push origin main:cms-staging`).
4. **Settings → Pages**: Source = "GitHub Actions" (staging deploys from `deploy-staging.yml`).
5. **Settings → Environments**: create `production` with a required reviewer.
6. **Secrets**: `NETLIFY_AUTH_TOKEN`, `NETLIFY_SITE_ID` (repository-level).
7. **Settings → Actions → General**: Workflow permissions "Read and write", and allow Actions to create pull requests (the publish workflow opens and merges a PR).
8. **Netlify**: enable *Forms → Form detection*, and add a notification (email) for the `contact` form.
9. **Editors**: each one creates a fine-grained GitHub PAT for this repo (Contents: read/write) and pastes it at `/admin/` → "Sign in with Access Token".

## How it works

- **Saves** in the CMS commit to `cms-staging`.
- **Publish Changes** (CMS header button, or Actions → "Publish CMS edits") builds `cms-staging`, squash-merges it into `main`, resets `cms-staging`, and deploys staging.
- **Staging**: GitHub Pages, on every push to `main`. The build gets the Pages base path (`/REPO/`) automatically.
- **Production**: Netlify, only via a manual run of "Deploy to Netlify (production)" plus reviewer approval.
- `reset-cms-staging.yml` force-resets `cms-staging` to `main` on every push to `main`.
- **Caveat**: if you ever merge `cms-staging` → `main` by hand, check that the squash message has no `[skip ci]` line, or the push workflows won't run.

The D3I daily cron rebuild and `production` snapshot branch were dropped: this site has no future-dated content. Bring them back from the D3I repo if news/events with scheduled dates are added.

## Content model

| CMS entry | File | Rendered by |
|-----------|------|-------------|
| Pages → Home | `index.md` | `_layouts/home.html` |
| Pages → Why our app? | `_pages/why-our-app.md` | `_layouts/why.html` |
| Pages → About | `_pages/about.md` | `_layouts/about.html` |
| Pages → Funding Bodies | `_pages/funding-bodies.md` | `_layouts/funding.html` (cards from Funders) |
| Pages → Team / Collaborators | `_pages/team.md` | `_layouts/team.html` (members and organisations are lists in the page itself) |
| Pages → Contact Us | `_pages/contact.md` | `_layouts/contact.html` |
| Pages → Contact: thank-you page | `_pages/contact-thanks.md` | `_layouts/page.html` |
| Lists → Funders | `_data/funders.yml` | Funding Bodies cards + footer "Funded by" (`show_in_footer`) |
| Site settings | `_data/settings.yml` | Header, footer, "Have a study in mind?" banner |

**Not in the CMS, on purpose:**
- Navigation (`_data/navigation.yml`): URLs need matching permalinks.
- Layouts and `main.css`.
- The contact form's fields: their names are the Netlify submission keys.
- 404 page.
- Hidden per-page keys (`layout`, `permalink`, `nav_key`, `parent`): kept as `widget: hidden` so saves preserve them.

**Under construction** (in the wireframes' nav, but no page yet): News & Events, Privacy policy. See the comment in `_data/navigation.yml`.

### Adding a page

1. Create `_pages/<name>.md` with `layout`, `permalink`, `nav_key` and the content fields.
2. Add a layout, or reuse one.
3. Add a file entry under `pages` in `admin/config.yml`, with the hidden keys.
4. Add it to `_data/navigation.yml` if it needs a menu item.

## Rules that keep the CMS safe (learned on D3I)

- **Empty optional fields are saved as `''`**, which Liquid treats as true. Always guard with `{% if x and x != "" %}`, never a bare `{% if x %}`. Liquid 4 has no `blank` literal.
- **No Liquid in editable text.** The markdown editor mangles `{% %}` tags. Anything dynamic belongs in a layout or include, driven by fields.
- **First CMS save normalizes a file** (key order, quotes). Expect one cosmetic diff per file.
- **Links typed inside markdown bodies** (e.g. `[home](/)`) do not get the staging base path. They work on production; on a project Pages staging site they point to the domain root. Link fields (buttons, cards) are fine.
- **The contact form only works on Netlify (production).** GitHub Pages answers the POST with 405.
- **Sveltia is pinned** in `admin/index.html`. To upgrade: bump the version, then run `node cms/validate-config.mjs` (CI runs it too). A schema error blocks sign-in for everyone.
- Use `widget: datetime, type: date` for dates (not `widget: date`). Use `accept:` for image types (not `file_types`).

## Useful commands

```sh
bundle exec jekyll serve
npm install --no-save --no-package-lock @cfworker/json-schema yaml && node cms/validate-config.mjs
```
