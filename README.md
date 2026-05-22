# Brave Hardy Shopify theme

This repo is the Online Store theme for the **108fdd-3** shop. The store is set in [`shopify.theme.toml`](shopify.theme.toml), so you normally run Shopify CLI commands from **this directory** without passing `--store`.

## Prerequisites

- [Shopify CLI](https://shopify.dev/docs/api/shopify-cli) installed (`shopify version`).
- Logged in (`shopify theme dev` or `shopify theme pull` will prompt you to authenticate if needed).

## Theme IDs

List themes and their numeric IDs:

```bash
shopify theme list
```

Use the ID with `--theme` on push, pull, dev, and publish. In the admin, the ID also appears in the URL after `/themes/` when you open a theme.

| Role | Meaning |
|------|---------|
| `[live]` | What customers see on the storefront |
| `[unpublished]` | A copy in the theme library (safe staging target) |
| `[development]` | Ephemeral theme created by `shopify theme dev` on your machine |

`shopify theme dev` syncs to a **development** theme, not the live theme. A local preview can look different from production even after a pull if you are comparing the wrong theme or URL.

## Recommended workflow (safe publish)

Use a **duplicate of the live theme** so code changes can be tested and published without overwriting merchant settings from your laptop.

### 1. Duplicate live in the admin

**Online Store → Themes** → ⋯ on the live theme → **Duplicate**. Note the new theme’s ID from `shopify theme list` (or the admin URL).

The duplicate is a full snapshot at that moment: Liquid, assets, `settings_data.json`, and template JSON.

### 2. Develop against the duplicate

```bash
shopify theme dev --theme <duplicate-id> --theme-editor-sync
```

Edit code locally; use the preview URL from the terminal. Optional Theme Editor changes on the **duplicate** stay on Shopify.

### 3. Push code only (settings protected)

[`/.shopifyignore`](.shopifyignore) excludes `config/settings_data.json` from push, so CLI uploads do not overwrite Theme Editor settings on the remote theme from your repo.

```bash
shopify theme push --theme <duplicate-id>
```

Do **not** use `--publish` until you have previewed the duplicate. Avoid `shopify theme push --publish` unless you intend to go live in one step.

### 4. Publish when ready

In **Online Store → Themes**, open the duplicate → **Publish**.

Or via CLI (files must already be on that theme):

```bash
shopify theme publish --theme <duplicate-id>
```

Publishing switches which theme is live. It does not reset settings to defaults; customers see that theme’s full state (code plus its `settings_data.json` and template JSON). The previous live theme becomes unpublished and keeps its own settings snapshot in the library.

**Before publish:** if merchants edited the **current** live theme in the admin while you worked on the duplicate, merge those changes (Theme Editor on the duplicate, or a selective pull from live) so you do not ship stale content.

## Local testing (default dev theme)

Without `--theme`, `shopify theme dev` uses or creates a **[development]** theme tied to your CLI session:

```bash
shopify theme dev --theme-editor-sync
```

That is fine for quick experiments. For a preview that matches production settings, prefer the duplicate workflow above.

To validate Liquid and theme structure:

```bash
shopify theme check
```

## Refresh from the store

Download files from Shopify:

```bash
shopify theme pull
```

Add `-l` to pull the **live** theme. Add `-n` to avoid deleting local files that are not on the remote.

To refresh **merchant content** without overwriting theme code, pull only JSON settings:

```bash
shopify theme pull -l -n \
  --only config/settings_data.json \
  --only "templates/*.json" \
  --only "sections/*-group*.json"
```

Homepage layout and header behaviour live in `templates/*.json` and `sections/*-group*.json`, not only `settings_data.json`. [`config/settings_schema.json`](config/settings_schema.json) defines which settings exist (developer-owned); [`config/settings_data.json`](config/settings_data.json) stores the values (merchant-owned).

## Schema vs data

| File | Role |
|------|------|
| `config/settings_schema.json` | Defines global theme settings in the admin (labels, types, defaults). Ship with code. |
| `config/settings_data.json` | Saved values from the Theme Editor. Listed in `.shopifyignore` so push does not clobber remote settings. |

## Reference

- [Theme CLI environments](https://shopify.dev/docs/storefronts/themes/tools/cli/environments) (`shopify.theme.toml`)
- [Theme directory structure](https://shopify.dev/docs/storefronts/themes/tools/cli#directory-structure)
