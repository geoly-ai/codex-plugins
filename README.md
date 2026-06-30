# GEOly — Codex plugin marketplace

Self-hosted [OpenAI Codex](https://developers.openai.com/codex) plugin marketplace for **GEOly**, the GEO (Generative Engine Optimization) platform that tracks how brands are mentioned and cited across AI engines (ChatGPT, Gemini, Perplexity, Grok, Google AI).

This repo packages the **GEOly MCP server** + the **`geoly-mcp` skill** into one installable Codex plugin. It is a *self-hosted* marketplace — it is **not** OpenAI's official Plugin Directory (self-serve publishing to that directory is "coming soon" per OpenAI; until then this repo is the distribution channel).

> Intended home: a public repo `geoly-ai/codex-plugins`. The plugin connects to the hosted GEOly MCP endpoint, so users need a GEOly account; OAuth runs at install time.

---

## Install (for Codex users)

```bash
# 1) Register this marketplace (public repo → owner/repo shorthand works without credentials)
codex plugin marketplace add geoly-ai/codex-plugins

# 2) (optional) confirm it registered
codex plugin marketplace list

# 3) Install the plugin (the install verb is `add`, not `install`).
#    @geoly is the marketplace slug, used to disambiguate same-named plugins.
codex plugin add geoly-mcp@geoly

# 4) On install (policy.authentication = ON_INSTALL) Codex opens the GEOly OAuth
#    browser flow — sign in / authorize, and the geoly MCP tools become available.
codex plugin list
```

Inside a Codex session you can use the slash-command equivalents:

```
/plugin marketplace add geoly-ai/codex-plugins
/plugin install geoly-mcp@geoly
```

### Pin a version

```bash
codex plugin marketplace add geoly-ai/codex-plugins --ref v0.1.0   # branch or tag
# session slash form:  /plugin marketplace add geoly-ai/codex-plugins#v0.1.0
```

### Upgrade / remove / troubleshoot

```bash
codex plugin marketplace upgrade geoly   # pull newer commits from this repo
codex plugin remove geoly-mcp            # uninstall the plugin
codex plugin marketplace remove geoly    # unregister this marketplace

# Underlying OAuth primitives (the install flow usually drives these for you):
codex mcp login geoly                    # manually run the OAuth login
codex mcp logout geoly                   # clear stored OAuth credentials
codex mcp list                           # list configured MCP servers
```

---

## What you get

`geoly-mcp` exposes up to ~61 MCP tools (the exact set depends on your plan, org membership, and write profile), covering:

- **Your brand (authenticated):** `get_brand_overview`, `query_analytics`, `get_prompt_list` / `get_prompt_detail` / `get_prompt_citations` / `get_prompt_mention_rates`, `get_citation_overview`, `get_competitor_overview`, `get_platform_matrix`, `get_topic_analytics`, `get_audit_list`, …
- **Public / industry intelligence (Grow+ plan):** `search_public_entities`, `get_public_category`, `get_category_whitespace`, `get_category_brand_momentum`, `get_public_brand_perception`, `compare_public_brands`, `get_topic_competition_difficulty`, …
- **Write actions (require an interactive `standard`/`admin` profile):** `create_prompt`, `create_topic`, `create_competitor`, `trigger_prompt` (consumes credits).

The bundled skill (`SKILL.md` + `references/`) teaches Codex to pick the right tool, follow the org/brand discovery flow, and quote the correct KPI caliber (citation rate, mention rate, AIGVR, Share of Model).

---

## Requirements & notes

- **GEOly account required.** After OAuth you need an active subscription; **public/industry tools require a Grow+ plan**. Free/basic users can authenticate but some tools return `402` or are hidden.
- **Single-org context recommended.** A user-level token spanning ≥2 orgs enters `multi-org` mode, which is read-only and disables the public/industry tool set. Pass a single org if you need those tools.
- **Hosted, no local server.** The plugin points at `https://app.geoly.ai/api/mcp` (streamable HTTP + OAuth); nothing runs on your machine.

---

## Maintainer notes (GEOly team)

This is a standalone repo. Clone it as a **sibling** of the `geoly-ai/geoly-app` repo:

```bash
git clone https://github.com/geoly-ai/codex-plugins.git ../geoly-codex-plugins
```

The skill content under `plugins/geoly-mcp/skills/geoly-mcp/` is a **copy** of the canonical source `geoly-mcp/` in the `geoly-ai/geoly-app` repo. Do not hand-edit it here — edit the source and re-sync:

```bash
# from the geoly-app repo root (defaults to ../geoly-codex-plugins)
node scripts/build-codex-plugin.mjs
# or pass an explicit path / set CODEX_PLUGIN_REPO
node scripts/build-codex-plugin.mjs /path/to/geoly-codex-plugins
```

That script mirrors `geoly-app/geoly-mcp/{SKILL.md,CHANGELOG.md,references/**}` into this plugin and reports any drift. Then `git commit && git push` here.

Bump `plugins/geoly-mcp/.codex-plugin/plugin.json` `version` (semver) on each release and tag the repo so users can `--ref` pin.

**Before announcing, run the on-device checklist** in `geoly-app/docs/mcp/CODEX_PLUGIN_DISTRIBUTION.md` (OAuth auto-trigger on install, `oauth_resource` value, `dependencies.tools` recognition, public-tool gating).

### Layout

```
codex-plugins/
├── .agents/plugins/marketplace.json     # marketplace catalog (fixed path)
└── plugins/geoly-mcp/
    ├── .codex-plugin/plugin.json        # plugin manifest (fixed subdir; interface = plugin metadata)
    ├── .mcp.json                        # remote MCP server (OAuth)
    ├── assets/                          # icons / logos
    └── skills/geoly-mcp/                # ← synced copy of geoly-app/geoly-mcp/
        ├── SKILL.md
        ├── CHANGELOG.md
        ├── agents/openai.yaml           # skill-level adapter: metadata + policy + MCP dependency
        └── references/
```
