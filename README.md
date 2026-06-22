# ascend-rs-docs

Public, billing-immune deploy for the **tile-rs / ascend-rs** documentation site
(bilingual mdbook: English + 中文).

This repo holds **no source** — only the GitHub Actions workflow
[`.github/workflows/deploy.yml`](.github/workflows/deploy.yml). On each run it:

1. Checks out the **private** source `yijunyu/ascend-rs-priv` (read-only, via
   `PRIVATE_SOURCE_TOKEN`).
2. Builds the bilingual mdbook (`blog/mdbook/`, mdbook 0.5.2, vendored mermaid).
3. Assembles a combined site — `en/` + `zh/` + a root landing `index.html`.
4. Publishes it to **this repo's** GitHub Pages.

### Why a separate public repo?

The private `ascend-rs-priv` repo's GitHub Actions are blocked by a billing
gate, so its own `mdbook.yml` can no longer deploy. Public-repo Actions are free
and unaffected. This is the same "public repo builds from private source"
(cargo-slicer) shape already used by the `codegen-release` CI.

### Required secret

| Secret | Scope |
|--------|-------|
| `PRIVATE_SOURCE_TOKEN` | fine-grained PAT, **Contents: Read** on `yijunyu/ascend-rs-priv` |

```sh
gh secret set PRIVATE_SOURCE_TOKEN --repo yijunyu/ascend-rs-docs
```

(The same token already configured for `yijunyu/tile-rs` works.)

### Triggers

- `workflow_dispatch` — manual.
- `repository_dispatch` (type `rebuild-docs`) — the private repo can trigger a
  rebuild: `POST /repos/yijunyu/ascend-rs-docs/dispatches {"event_type":"rebuild-docs"}`.
- daily `schedule` (06:17 UTC) — safety refresh.

### Default URL

The site deploys to the **default** Pages URL:

> https://yijunyu.github.io/ascend-rs-docs/  (`/en/`, `/zh/`)

It deliberately does **not** write a `CNAME` — the live `ascend-rs.org` stays on
the private repo's Pages until the maintainer performs the cutover below.

### Custom-domain cutover (`ascend-rs.org`) — manual, reversible

DNS for `ascend-rs.org` already points at `yijunyu.github.io`, so this is purely
a Pages-settings move. Only **one** repo may claim the domain at a time.

1. **Remove** the domain from the old repo:
   `ascend-rs-priv` → Settings → Pages → Custom domain → clear `ascend-rs.org`, Save.
2. **Add** it to this repo:
   `ascend-rs-docs` → Settings → Pages → Custom domain → `ascend-rs.org`, Save,
   then wait for the DNS check and enable **Enforce HTTPS**.
3. (Optional) add a `CNAME` step to `deploy.yml` so future deploys keep the
   domain claimed:
   ```yaml
   echo 'ascend-rs.org' > "$SITE/CNAME"
   ```

To revert: clear the domain here and re-add it on `ascend-rs-priv`.
