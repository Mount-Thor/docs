# Mount Thor Docs — Agent Instructions

This is the customer-facing documentation site at `docs.mountthor.com`,
built on [Mintlify](https://mintlify.com). Pages are MDX with YAML
frontmatter; site config lives in `docs.json`.

> For Mintlify product knowledge (components, configuration, writing
> standards), install the Mintlify skill:
> `npx skills add https://mintlify.com/docs`

## Repo shape

- `index.mdx`, `quickstart.mdx` — top-level entry points.
- `concepts/` — auth model, alpha-status surface, foundational concepts.
- `compute/` — bare metal and VM resource walkthroughs.
- `troubleshooting/` — failure codes and remediation.
- `api-reference/` — Mintlify auto-generates one page per route from
  `https://api.mountthor.com/openapi.json` here. The single hand-written
  page is `api-reference/introduction.mdx`.
- `logo/`, `favicon.svg`, `images/`, `snippets/` — brand and reusable bits.
- `RUNBOOK.md` — operator runbook (manual Mintlify dashboard steps, custom
  domain setup, password rotation cadence). Not customer-facing.
- `.watermark` — records the upstream `Mount-Thor/mount-thor` commit SHA
  the docs were derived from. Bump whenever you migrate or refresh content
  from the monorepo.

## Source of truth and why

Customer-developer content here is **derived from**, not the source of
truth for, `Mount-Thor/mount-thor`. The canonical specs live in:

- `governance/docs/developers/README.md` — alpha onboarding runbook + customer
  product flow. The MDX pages under `compute/`, `concepts/auth.mdx`, and
  `troubleshooting/common-failures.mdx` are derived from sections of this
  file.
- `governance/docs/api-surface-registry-and-openapi-publication-spec.md` —
  the customer API contract. Drives the `[live]` / `[todo]` markers used
  throughout these docs.
- `https://api.mountthor.com/openapi.json` — the actual published OpenAPI
  document, generated from typed `utoipa` annotations in admin-api.
  Mintlify fetches this URL at build time.

Edits here that imply behavior change must be traceable to a change in the
monorepo first (or land alongside one). Don't promise a customer-facing
shape on this site that the monorepo hasn't shipped or scheduled.

## Conventions

### Voice

- Active voice, second person (`you`). Never `the user`.
- One idea per sentence. Lead with the goal.
- Sentence case for headings (`## Open access`, not `## Open Access`).
- Bold for UI elements (`Click **Settings**`).
- Code formatting for file names, commands, route paths, and code refs.

### `[live]` / `[todo]` markers

Preserve them on routes, fields, and behaviors that have a current and a
target shape. Customers explicitly ask for this — it's how they plan
migration. Markers come from
`governance/docs/api-surface-registry-and-openapi-publication-spec.md`
and from the live admin-api implementation.

### Linking conventions

- Internal links: `/concepts/auth`, `/compute/bare-metal` (no `.mdx`).
- API reference cross-link: `/api-reference/introduction` — Mintlify
  auto-generates the per-route pages alongside it.
- External: full URL, no shorteners.

### Components

Mintlify components used here:

- `<Note>` for context callouts.
- `<Warning>` for "do this carefully or you'll lose data" scenarios.
- `<Tip>` for optional optimizations.
- `<CardGroup>` + `<Card>` for navigation hubs (homepage, "next steps"
  sections).
- `<Steps>` + `<Step>` for ordered procedures (auth lifecycle, etc.).
- `<CodeGroup>` when showing the same operation across multiple shells /
  languages.

Avoid: `<Frame>` decorations, marketing component patterns, anything that
implies consumer-tech aesthetics. The brand reference points are
Palantir / Bloomberg / Stripe — institutional restraint.

## Local development

```bash
npm i -g mint
mint dev          # http://localhost:3000
mint validate     # frontmatter + structure
mint broken-links # link integrity
```

## CI

`.github/workflows/validate.yml` runs `mint validate` and `mint broken-links`
on PRs. Both must pass before merge.

## Don't

- Don't write `noindex`-style instructions or hide content from search.
  Customer-facing content here is intentionally accessible to authenticated
  customers.
- Don't add the `"public": true` flag to navigation groups without an
  explicit reason. The site is universally password-gated until Mintlify
  Enterprise + WorkOS OAuth lands.
- Don't migrate the **Onboarding Operating Rules** section from the
  governance README — that's internal Mount Thor policy, not customer
  content.
- Don't recolor the brand mark or wordmark. The mountain triangle is
  always `#C9A84C` (gold). Wordmark is always Cormorant Garamond 500 at
  0.18em letter-spacing.
- Don't introduce `kbd`-style "press these keys" UI patterns. The
  customer-developer audience reads MDX through `kubectl` and `curl`,
  not through a UI shell.

## When in doubt

Defer to `Mount-Thor/mount-thor`'s `governance/docs/developers/README.md`
for behavior. If a contradiction exists between this site and the
governance README, the governance README wins until this site is updated.
