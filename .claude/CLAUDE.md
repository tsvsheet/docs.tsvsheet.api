# `docs.<project>` — public documentation repository

This is the **public** documentation site for a project, created from [`nicerobot/template.repo-docs`](https://github.com/nicerobot/template.repo-docs). It is published to GitHub Pages (and/or Cloudflare).

## Conventions

- **Everything here is public.** This repo holds public development documentation — contributing guides, design, rules — built as a self-contained [Hugo](https://gohugo.io) site at the repo root. Markdown content goes under [`content/`](../content/).
- **Private content does NOT live here.** Ideas, tasks, internal notes, and specs live in the project's private hub repo (`project.<project>`), never in this repo. There is no `public/`/`private/` split — a docs repo is wholly public.
- **Cross-links stay relative.** Link between pages with the target's source path (e.g. `[Architecture](architecture.md)`); Hugo's embedded link render hook (`markup.goldmark.renderHooks.link`, enabled in [`../hugo.json`](../hugo.json)) resolves them to the right URLs, so the same links also work when browsing the markdown on GitHub.
- The Pages workflow ([`.github/workflows/pages.yml`](../.github/workflows/pages.yml)) is always present and **self-gates on whether Pages can serve the site**: a public repo always builds (self-enabling Pages), a private repo builds only when Pages is already enabled (paid-plan private Pages), and a private repo without Pages skips neutrally. Going public or enabling Pages requires no file change. The Pages source must be **GitHub Actions** (`build_type: workflow`) — the legacy branch source renders the README instead of the Hugo site.
- Preview locally with `make serve`; build with `make build`.
