# Mount Thor Docs

Customer documentation for Mount Thor, served at `docs.mountthor.com`.

This is a [Mintlify](https://mintlify.com) site. MDX pages + `docs.json`
config; deployed automatically by Mintlify's GitHub App on every push to
`main`. Universal-password gated for the alpha customer cohort.

The customer-facing API reference is auto-generated from
[`https://api.mountthor.com/openapi.json`](https://api.mountthor.com/openapi.json),
which Mintlify re-fetches on every build.

## Local development

```bash
npm i -g mint
mint dev          # http://localhost:3000
mint validate     # frontmatter + structure
mint broken-links # link integrity
```

## See also

- `AGENTS.md` — guidance for AI editors / contributors.
- `RUNBOOK.md` — operator runbook (Mintlify dashboard, custom domain,
  password rotation).
- `.watermark` — upstream commit SHA the docs were derived from.

## Source of truth

Customer-developer content here is derived from
[`Mount-Thor/mount-thor`](https://github.com/Mount-Thor/mount-thor) under
`governance/docs/developers/README.md`. Don't promise a customer-facing
shape here that the monorepo hasn't shipped or scheduled.
