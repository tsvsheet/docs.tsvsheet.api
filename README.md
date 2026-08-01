[![actions](https://github.com/nicerobot/template.repo-docs/actions/workflows/actions.yml/badge.svg)](https://github.com/nicerobot/template.repo-docs/actions/workflows/actions.yml) [![docs](https://github.com/nicerobot/template.repo-docs/actions/workflows/docs.yml/badge.svg)](https://github.com/nicerobot/template.repo-docs/actions/workflows/docs.yml) [![pages](https://github.com/nicerobot/template.repo-docs/actions/workflows/pages.yml/badge.svg)](https://github.com/nicerobot/template.repo-docs/actions/workflows/pages.yml)

# template.repo-docs

Canonical template for a project's **public docs** repository (`<org>/docs.<project>`). A project's developer and user documentation — contributing, design, rules — lives here as a public [Hugo](https://gohugo.io) site published via GitHub Pages. Private content (ideas, tasks, specs) lives in the project's hub repo (`project.<project>`), never here.

## Layout

| Path | Purpose |
| --- | --- |
| [`content/`](content/) | Markdown documentation — the Hugo site content (user-facing: what it does, how to use it). |
| [`dev/`](dev/) | Contributor documentation — unpublished by Hugo, read on GitHub (how it works inside, how to build and test). Public, so never rationale or unshipped intent. |
| [`layouts/`](layouts/) | Hugo templates (managed fleet style — distributed from this template). |
| [`assets/`](assets/) | Hugo assets (managed fleet style — distributed from this template). |
| [`hugo.json`](hugo.json) | Hugo configuration. |
| [`.github/workflows/pages.yml`](.github/workflows/pages.yml) | The GitHub Pages build workflow (managed fleet style). Always present; it deploys whenever Pages can serve the site and skips neutrally otherwise. |
| [`Makefile`](Makefile) | Local preview and build. Run `make` for help. |

## Public, with a self-gating workflow

Everything in this repo is **public** — it exists to be published, so there is no `public/`/`private/` split. Anything private (design notes, tasks, ideas, specs) belongs in the project's private hub repo (`project.<project>`).

A docs repo ships ready to publish: the Pages workflow is always present and **gates on whether Pages can actually serve the site** — a public repo always builds (Pages self-enables on the first run), a private repo builds when Pages is already enabled (paid plans serve private-repo Pages), and a private repo without Pages skips neutrally. There is no file to rename and no change needed to start deploying.

## Going public

1. Make the repository public.
2. Enable Pages: **Settings → Pages → Source: GitHub Actions** (the workflow's `configure-pages` step does this itself on the first run if needed).
3. Push (or re-run the **pages** workflow). It publishes the whole site.

## Creating a new docs repo from this template

```bash
gh repo create <org>/docs.<project> --public --template nicerobot/template.repo-docs
```
