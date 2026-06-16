# IBKR-Indian-Tax-Buddy-Docs

Documentation website for [IBKR Indian Tax Buddy](https://github.com/abhi4u1947/IBKR-Indian-Tax-Buddy),
built with [Jekyll](https://jekyllrb.com/) and the
[just-the-docs](https://just-the-docs.com/) theme and published with GitHub
Pages.

## Live site

Once deployed, the site is available at:

<https://abhi4u1947.github.io/IBKR-Indian-Tax-Buddy-Docs/>

## How publishing works

Every push to `main` (or the `claude/github-pages-website-dyc8w7` development
branch) triggers the [`Deploy docs site to GitHub Pages`](.github/workflows/pages.yml)
workflow, which builds the Jekyll site and deploys it to GitHub Pages. The
workflow enables Pages automatically on first run, so no manual Settings step
is required.

## Local development

```bash
bundle install
bundle exec jekyll serve
```

Then open <http://localhost:4000/IBKR-Indian-Tax-Buddy-Docs/>.

## Structure

- `index.md` — site home page.
- `docs/` — documentation pages.
- `_config.yml` — Jekyll and theme configuration.
- `.github/workflows/pages.yml` — build and deploy workflow.
