# Mount Thor Docs — Operator Runbook

This is the operator-facing runbook for `Mount-Thor/docs` (the Mintlify-hosted
customer documentation site at `docs.mountthor.com`). Customer-facing content
goes in MDX pages, not here.

## Architecture at a glance

- **Source repo:** `Mount-Thor/docs` (private). MDX pages + `docs.json` config.
- **Build/deploy:** GitHub Actions validates the docs repo. Production
  deployment is triggered from the Mintlify dashboard for `docs.mountthor.com`
  unless the Mintlify API trigger is wired into CI.
- **Custom domain:** `docs.mountthor.com`. DNS managed in Mount Thor's
  AWS Route53 zone for `mountthor.com` (subdomain CNAME → `cname.mintlify.builders`).
- **TLS:** Mintlify-issued Let's Encrypt cert. Auto-renewed by Mintlify;
  not managed in our infra.
- **Auth gate:** Mintlify Pro universal password (set in dashboard). Custom
  OAuth via WorkOS is deferred to Mintlify Enterprise.
- **API reference sources:** `docs.json` points at
  `api-reference/mount-thor-local.openapi.json`, a checked-in combined OpenAPI
  artifact. Refresh it from the live admin API at
  `https://api.mountthor.com/openapi.json` plus the checked-in compute API at
  `api-reference/customer-compute-crds.v1.openapi.json`.

## Manual one-time setup

These steps live outside git and only need to happen once.

### 1. Mintlify GitHub App

1. Sign in to `https://dashboard.mintlify.com` as the workspace owner.
2. Navigate to **Settings** → **GitHub App** and install on
   `Mount-Thor/docs` only (not org-wide).
3. The app needs `contents: read`, `metadata: read`, and webhook access for
   PR builds. No write permissions required.

### 2. Custom domain

1. In the Mintlify dashboard, **Settings** → **Custom Domain**, add
   `docs.mountthor.com`.
2. Mintlify shows two TXT verification values for
   `_acme-challenge.docs.mountthor.com` and
   `_cf-custom-hostname.docs.mountthor.com`.
3. Copy those values into the OpenTofu stack
   `infra/aws/live/accounts/mthr-prod/customer-public-dns/`
   (set `mintlify_acme_challenge_value` and
   `mintlify_cf_custom_hostname_value` tfvars), open a PR, get apply
   approval on the `aws-prod-apply` environment.
4. Back in the Mintlify dashboard, click **Verify**. Once both TXT records
   resolve, Mintlify provisions a Let's Encrypt cert (a few minutes).

The `docs.mountthor.com` CNAME → `cname.mintlify.builders` and the placeholder
TXT records are pre-wired in the OpenTofu stack; only the captured TXT values
need to land.

### 3. Universal password

1. **Settings** → **Authentication** → enable Universal Password.
2. Set the password. Store it in the team password manager
   (1Password / Bitwarden / etc.).
3. Communicate the password to the alpha customer cohort via the secure
   channel agreed with Mount Thor — never in Slack DM, email, or unencrypted
   chat.
4. Rotate quarterly or whenever a customer leaves the cohort. Update the
   shared password in the team manager and re-distribute via the secure
   channel.

## Day-to-day workflow

### Editing pages

```bash
git checkout -b chris/<short-topic>
# edit MDX + docs.json
mint dev          # local preview at http://localhost:3000
mint validate     # frontmatter + structure
mint broken-links # link integrity
git commit -m "<scope>: <change>"
git push -u origin <branch>
gh pr create --base main
```

PR preview behavior depends on the Mintlify dashboard Git connection. If a PR
preview is not posted, use the local `mint dev` preview and the CI validation
checks before merging.

### Merging

Squash merge after reviewer approval. If the Mintlify dashboard does not show a
fresh deployment for the merge commit, click the dashboard deploy button for
`docs.mountthor.com`.

### When the OpenAPI spec changes upstream

The admin `/openapi.json` URL is not fetched directly by Mintlify. Refresh the
checked-in combined artifact at `api-reference/mount-thor-local.openapi.json`
from the live admin OpenAPI and the checked compute OpenAPI, then merge the
updated artifact with the docs change.

The compute OpenAPI document is checked into this repo. When the monorepo
changes `governance/generated/customer-compute-crds.v1.openapi.json`, copy the
new artifact to `api-reference/customer-compute-crds.v1.openapi.json`, remove
non-workflow resources such as core `ConfigMap` and `MacIncident`, and bump
`.watermark` in the same PR.

### When to update `.watermark`

Whenever you migrate or refresh content from the Mount Thor monorepo
(typically `governance/docs/developers/README.md` or sibling specs), bump
`.watermark` at the repo root with the source SHA. Reviewers can see at a
glance how stale a page might be relative to the source.

## Failure modes

### Mintlify build fails

- **Bad MDX syntax:** Mintlify's build log points at the file and line.
  Fix in a PR; the build retries automatically.
- **Missing image / link:** `mint broken-links` should have caught this
  pre-merge. If it didn't, file-level fixes only.
- **OpenAPI drift check failure:** confirm the live admin endpoint is healthy
  (`curl -i https://api.mountthor.com/openapi.json`), regenerate
  `api-reference/mount-thor-local.openapi.json`, and rerun validation.

### Custom domain stops resolving

- **Nameserver flap at GoDaddy:** verify `dig NS mountthor.com @8.8.8.8`
  still returns the AWS nameservers. If GoDaddy's UI was edited and
  reverted nameservers to GoDaddy defaults, restore them per
  `governance/changes/2026-05-07-customer-public-dns-migration.yaml` and
  the OpenTofu zone NS output.
- **Cert renewal failed:** check Mintlify dashboard alerts. The Let's
  Encrypt validation TXT records (`_acme-challenge.docs.mountthor.com`)
  must resolve; verify with `dig TXT _acme-challenge.docs.mountthor.com`.

### The site went public unexpectedly

- Check Mintlify dashboard **Authentication** — ensure Universal Password
  is still enabled.
- Check for accidental `"public": true` flags on navigation groups in
  `docs.json`.
- Review recent commits — `docs.json` is the only thing that gates
  per-group visibility.

## What lives where

| Concern | Location |
|---|---|
| Page content | `*.mdx` files at the repo root + topic subdirs |
| Site config (theme, nav, OpenAPI sources) | `docs.json` |
| Brand assets | `logo/`, `favicon.svg` |
| CI validation | `.github/workflows/validate.yml` |
| Source-derivation tracking | `.watermark` |
| Mintlify dashboard config (GitHub App install, custom domain, password) | `https://dashboard.mintlify.com` (not in git) |
| Public DNS records | OpenTofu in `Mount-Thor/mount-thor` at `infra/aws/live/accounts/mthr-prod/customer-public-dns/` |
| WorkOS docs OAuth secret (placeholder) | AWS Secrets Manager `mthr-workos-docs` (out-of-band populate when Enterprise) |
