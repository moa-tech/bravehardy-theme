# Brave Hardy Shopify theme

This repo is the Online Store theme for the **108fdd-3** shop. The store is set in [`shopify.theme.toml`](shopify.theme.toml), so you normally run Shopify CLI commands from **this directory** without passing `--store`.

## Prerequisites

- [Shopify CLI](https://shopify.dev/docs/api/shopify-cli) installed (`shopify version`).
- Logged in (`shopify theme dev` or `shopify theme pull` will prompt you to authenticate if needed).

## Local testing

Start the dev server: it uploads a **development theme**, serves a local preview URL, and syncs file changes (hot reload for CSS and sections).

```bash
shopify theme dev --theme-editor-sync
```

Run that from the repository root (the folder that contains `shopify.theme.toml`).

Use the preview URL printed in the terminal. Add `--open` to open it in your browser, or `--theme-editor-sync` if you want the theme editor to stay in sync while you work.

To validate Liquid and theme structure:

```bash
shopify theme check
```

## Publish changes to Shopify

**Upload** your local files to a theme in the store library (overwrites the remote copy of the theme you target):

```bash
shopify theme push
```

If you need a specific remote theme, use `--theme <id-or-name>`. To push **and** set that theme as the live storefront in one step:

```bash
shopify theme push --publish
```

To **only** switch which remote theme is live (after the files are already on Shopify):

```bash
shopify theme publish --theme <theme_id>
```

## Refresh from the store

To download the latest theme files from Shopify (for example after edits in the admin):

```bash
shopify theme pull
```

Add `-l` to pull the **live** theme.

## Reference

- [Theme CLI environments](https://shopify.dev/docs/storefronts/themes/tools/cli/environments) (`shopify.theme.toml`)
