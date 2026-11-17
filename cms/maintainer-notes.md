# CMS & pipeline maintainer notes

Adapted from the datadonation.eu (D3I) setup, which this site reuses as-is.

## First-time setup for the new repository (TODO)

Repo: `d3i-infra/next-research-app` (public; the org is on the free plan, which only serves Pages from public repos). Site: https://d3i-infra.github.io/next-research-app/

Done:
- `admin/config.yml` and `_config.yml` point at the repo and the Pages URL.
- Pages source is "GitHub Actions".
- Workflow permissions are "Read and write".
- `cms-staging` exists.

The org forbids Actions from creating PRs, so `publish-cms-edits.yml` squash-merges with plain git instead of a PR.

Still to do:
1. **Custom domain** (when there is one): update `url` in `_config.yml`, and `site_url` / `display_url` in `admin/config.yml`.
2. **Editors**: each one creates a fine-grained GitHub PAT, resource owner `d3i-infra`, repository `next-research-app` only, *Contents: Read and write*. They paste it at `/admin/` → **"Sign in with Access Token"**. The org may have to approve fine-grained tokens first.
   - Do **not** use "Sign in with GitHub": that button needs an OAuth helper, which by default lives at Netlify (`api.netlify.com`), so it fails here. Making it work needs our own OAuth proxy (e.g. the `sveltia-cms-auth` Cloudflare Worker, configured with `base_url` under `backend:`).

## How it works

- **Saves** in the CMS commit to `cms-staging`.
- **Publish Changes** (CMS header button, or Actions → "Publish CMS edits") builds `cms-staging`, squash-merges it into `main` with plain git (no PR), resets `cms-staging`, and deploys staging.
- **Hosting**: GitHub Pages only, deployed on every push to `main` (so every Publish goes live). The build gets the Pages base path (`/next-research-app/`) automatically.
- No Netlify and no separate production step. The D3I setup's Netlify production deploy was removed; get it from the D3I repo if a gated production site is needed later.
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
| Lists → Funders | `_data/funders.yml` | Funding Bodies cards + footer "Funded by" (`show_in_footer`) |
| Site settings | `_data/settings.yml` | Header, footer, "Have a study in mind?" banner |

**Not in the CMS, on purpose:**
- Navigation (`_data/navigation.yml`): URLs need matching permalinks.
- Layouts and `main.css`.
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
- **Links typed inside markdown bodies** (e.g. `[home](/)`) do not get the staging base path. They break on a project Pages site (they point to the domain root); they will work once the site has its own domain. Link fields (buttons, cards) are fine.
- **No contact form.** GitHub Pages cannot receive form posts, so Contact shows the details and a "Write to us" mailto button (shown only when the email field holds a real address). To bring a form back, post it to a form service (Formspree, Web3Forms, ...) from `_layouts/contact.html`.
- **Sveltia is pinned** in `admin/index.html`. To upgrade: bump the version, then run `node cms/validate-config.mjs` (CI runs it too). A schema error blocks sign-in for everyone.
- Use `widget: datetime, type: date` for dates (not `widget: date`). Use `accept:` for image types (not `file_types`).

## Useful commands

```sh
bundle exec jekyll serve
npm install --no-save --no-package-lock @cfworker/json-schema yaml && node cms/validate-config.mjs
```
