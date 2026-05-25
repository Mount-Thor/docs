# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Mintlify documentation site. Content lives in MDX files at the repo root and inside topic directories; site configuration lives in `docs.json`.

## Commands

- `mint dev` — local preview at `http://localhost:3000` (run from the directory containing `docs.json`). `--port 3333` for an alternate port.
- `mint broken-links` — validate internal links across the site.
- `mint update` — upgrade the local CLI when the preview falls out of sync with production.
- `npm i -g mint@4.2.577` — install the pinned CLI used by CI (requires Node ≥ 19).

There is no repo-local production deploy command. GitHub Actions validates the docs; production deployment is triggered from the Mintlify dashboard unless the Mintlify API trigger is wired into CI.

## Architecture

- **`docs.json`** is the source of truth for site structure. Adding a new MDX page is a two-step change: create the file *and* register it under `navigation.tabs[].pages` in `docs.json`. Pages omitted from `docs.json` are not reachable from the nav.
- **Tabs and groups**: top-level navigation currently uses one `Documentation` tab with L1 pages plus an `API Reference` group. Page paths in `docs.json` are routes (no `.mdx` extension, no leading slash).
- **MDX frontmatter** drives page metadata (`title`, `description`, `icon`). The `icon` value is a Font Awesome name.
- **API reference** pages are generated from `api-reference/mount-thor-local.openapi.json` — refresh the checked-in OpenAPI artifact when endpoint shapes change rather than hand-editing generated pages.
- **Reusable content** lives in `snippets/` and is imported into MDX pages; `images/` and `logo/` hold static assets referenced by absolute path (e.g. `/images/foo.png`).
- **`.mintignore`** excludes `drafts/` and `*.draft.mdx` from the build in addition to Mintlify's built-in ignores (`.git`, `.github`, `.claude`, `node_modules`, `README.md`, `LICENSE.md`, `CHANGELOG.md`, `CONTRIBUTING.md`).

## Writing conventions

From `AGENTS.md` and `CONTRIBUTING.md`:

- Active voice, second person ("you"), one idea per sentence.
- Sentence case for headings.
- Bold for UI elements (`**Settings**`); code formatting for filenames, commands, paths, and code references.
- Lead instructions with the user's goal.

## Mintlify product knowledge

For component reference (e.g. `<Steps>`, `<AccordionGroup>`, `<Frame>`, `<Info>`), configuration schema, and writing standards, install the Mintlify skill once per environment:

```bash
npx skills add https://mintlify.com/docs
```
