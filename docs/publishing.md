# Publishing godot-suite

Two moving parts: the **plugin** (git-hosted) and the **marketplace catalog**
(static-hosted). Claude Code's URL-based marketplaces download only the
`marketplace.json` file — not plugin files — so the catalog must reference the
plugin via a git source, not a relative path.

## Layout

- `.claude-plugin/marketplace.json` — catalog for **git installs** (relative
  `source: ./plugins/godot-suite`; works when the whole repo is cloned via
  `claude plugin marketplace add llmforagents/godot-suite`).
- `public/marketplace.json` — catalog for the **Cloudflare-hosted URL install**.
  Its plugin `source` is a `git-subdir` pointing at the GitHub repo, because only
  this JSON is fetched over HTTPS:
  ```json
  { "source": "git-subdir",
    "url": "https://github.com/llmforagents/godot-suite.git",
    "path": "plugins/godot-suite" }
  ```
- `public/index.html` — landing page served at the site root.

## Hosts

- **Plugin:** `github.com/llmforagents/godot-suite` (public).
- **Catalog:** Cloudflare Pages project `godot-suite`, custom domain
  `godot-suite.llm4agents.com` → `https://godot-suite.llm4agents.com/marketplace.json`.

## Deploy / update the catalog

Requires a Cloudflare API token with **Pages: Edit** (and **DNS: Edit** on the
`llm4agents.com` zone for the custom domain, one-time). Never commit the token.

```bash
export CLOUDFLARE_API_TOKEN=<your-token>
export CLOUDFLARE_ACCOUNT_ID=cbbac055674897e53d1853b6f33bba6b
npx wrangler pages deploy public --project-name godot-suite
```

## Release a new plugin version

1. Bump `version` in `plugins/godot-suite/.claude-plugin/plugin.json` and in both
   `marketplace.json` files.
2. Commit and `git push` to GitHub (the `git-subdir` source pulls latest).
3. Re-deploy the catalog only if `public/marketplace.json` changed.
