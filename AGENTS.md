# AGENTS.md — Brave Hardy Shopify theme

Instructions for AI coding agents (and humans) changing this theme. Read the whole
file before touching anything. `README.md` explains the Shopify concepts; this file is
the exact, tested procedure.

## The one rule

**Shopify is the source of truth, not this repo.** Merchants edit the live theme in
the Shopify admin all the time (Theme Editor, "Edit default theme content", and
Shopify itself sometimes rewrites Liquid files). So before any change, sync the
repo **from** the live theme. When uploading, push **only the files you changed**
to an **unpublished copy** of the live theme. Never push to the live theme, and never
push the whole folder.

## Store facts

- Store: `108fdd-3` (set in `shopify.theme.toml`, so `--store` is not needed from the repo root).
- Public site: https://www.bravehardy.com
- Theme naming convention: `Andrew Dev 2.0.N - <short description>`. Increment `N`
  from the highest existing name. Theme names can be **50 characters at most**.
- The theme library holds at most 20 themes. Tell the user when it is getting full;
  do not delete themes yourself.
- Apps that inject into the theme: Stoq (restock alerts / pre-orders), Judge.me (reviews).
  Their markup comes from app embeds and blocks, not from files in this repo.

## Who owns which files

| Files | Owner | Rule |
|---|---|---|
| `layout/`, `sections/*.liquid`, `snippets/`, `blocks/`, `assets/`, `config/settings_schema.json` | Developer (code) | Edit here, push with `--only`. |
| `templates/**/*.json`, `sections/*-group*.json`, `config/settings_data.json` | Merchant (Theme Editor) | Refresh from live only. Don't push by default (`.shopifyignore` lists them). If a change genuinely needs template wording or settings changed, edit the freshly synced file and push it with `--only` to the new staging copy only, then tell the user exactly what changed. Never push `config/settings_data.json`. |
| `locales/*.json` | **Both** | Merchants change storefront wording via "Edit default theme content", which writes here. This folder is **not** in `.shopifyignore`, so pushing a stale copy silently reverts their wording. Always sync first, and push a locale file only if you changed it. |

## Shopify CLI gotchas

- Run `shopify` commands from the repo root. Other directories may not have the
  nvm `PATH`, so if `shopify` is "not found", `cd` to the repo first.
- **Auto-upgrade is off** (`shopify config autoupgrade off`). With it on, the CLI
  reinstalls itself with npm after a command finishes. If that command is killed or
  piped into `head`, the install breaks halfway and the `shopify` binary disappears.
  Fix with `npm install -g @shopify/cli@latest`. Upgrade on purpose with `shopify upgrade`.
- Never pipe a `shopify` command into `head`, and don't kill one mid-run. Use `--json`
  or `--no-color` with `tail` instead. Give pushes and pulls a long timeout (several minutes).
- `shopify theme pull` into the repo applies `.shopifyignore`, so it does **not**
  refresh templates or section-group JSON. Without `-n` it also deletes local files that
  aren't on the remote. That's why the sync below pulls into a scratch folder.
- `shopify theme push` without `--nodelete` deletes remote files that aren't local.
  Without `--only` it uploads everything, including stale locales. Always use both.
- `--only` **overrides** `.shopifyignore`. Naming a template JSON or
  `settings_data.json` in `--only` uploads it. That's how to deliberately push a template
  change to staging, and also how to clobber merchant content by accident.
- `shopify theme dev --theme <id>` uploads the local theme to that theme when it starts.
  Only point it at a staging copy that was synced first, never at the live theme.
- `shopify theme duplicate` returns straight away, but Shopify keeps copying files in
  the background for a minute or more. A push made during that window reports
  "pushed successfully" and is then **silently overwritten** by the copy. Wait until
  the preview URL returns 200 (it 404s while copying), push, then verify by pulling
  your files back (step 5).

## Procedure for every change

### 1. Find the live theme

```bash
shopify theme list --json --no-color
```

Note the `live` theme's `id` and the highest `Andrew Dev 2.0.N` number.

### 2. Sync the repo from live (always, before editing)

```bash
git status                      # must be clean, or understand why not
rm -rf /tmp/bh-live && mkdir -p /tmp/bh-live
shopify theme pull --theme <live-id> --path /tmp/bh-live --no-color
diff -rq --exclude=.git --exclude=.DS_Store --exclude=.cursor --exclude=.shopify . /tmp/bh-live \
  | grep -v "Only in \.:"
```

- `Files X and Y differ`: live is newer. Copy it over: `cp /tmp/bh-live/X X`.
- `Only in /tmp/bh-live/...`: a file was added on Shopify (often by an app). Copy it in.
- `Only in ./<theme folder>/...`: a file exists locally but not live. Find out why
  before doing anything (someone deleted it in the admin, or it was never pushed).
- The only repo-only files expected are `README.md`, `AGENTS.md`, `CLAUDE.md`,
  `shopify.theme.toml`, `.gitignore`, `.shopifyignore`.

Show the user what changed on live, especially code or locale changes they may not
know about. Then commit the sync on its own, e.g. `Sync from live theme (<live name>)`,
so it isn't mixed up with your changes.

### 3. Make the change

- Edit code files only (see the ownership table). If the change needs template or
  section-group JSON changes (new section on a page, block settings), the user makes
  those in the Theme Editor on the staging theme after step 5. Tell them exactly what to set.
- Match the surrounding Liquid/JS style. The theme is Clean Canvas "Enterprise".
- Keep a list of every file you changed: `git diff --name-only`.

### 4. Validate

```bash
shopify theme check --no-color -o json
```

The theme has existing offenses (10 errors, 40 warnings as of 2026-09-28) in files we
haven't touched. Make sure **your** files add none.

### 5. Upload to a staging copy of live

```bash
shopify theme duplicate --theme <live-id> --name "Andrew Dev 2.0.<N+1> - <description>" --force --json --no-color

# Wait for the copy to finish: repeat until this prints 200 (404 = still copying).
# -L and the cookie jar are required: the preview link redirects and sets a cookie.
rm -f /tmp/cj; curl -sL -c /tmp/cj -b /tmp/cj -o /dev/null -w "%{http_code}\n" \
  "https://108fdd-3.myshopify.com/?preview_theme_id=<new-id>"

shopify theme push --theme <new-id> --nodelete \
  --only path/to/changed-file-1 --only path/to/changed-file-2 --no-color

# Verify: pull the whole staging theme back and diff against the step-2 live snapshot.
# Only your changed files should be listed.
rm -rf /tmp/bh-stage && mkdir -p /tmp/bh-stage
shopify theme pull --theme <new-id> --path /tmp/bh-stage --no-color
diff -rq /tmp/bh-live /tmp/bh-stage
```

Because the copy is a fresh snapshot of live, it has all current merchant settings.
Only your files differ.

### 6. Preview and verify

Give the user the preview and editor links:

- `https://108fdd-3.myshopify.com?preview_theme_id=<new-id>`
- `https://108fdd-3.myshopify.com/admin/themes/<new-id>/editor`

Say which pages and scenarios to check. Verify what you can yourself. The preview URL
works with plain `curl` once the copy has finished (append `?preview_theme_id=<new-id>`
to any storefront path, on the `108fdd-3.myshopify.com` domain, using `curl -sL` with a
cookie jar as above). Check the rendered HTML contains the expected theme name in
`Shopify.theme`, and that there's no `Liquid error`.

- Shopify caches rendered pages for a short while. After re-pushing a file, add a
  cache-buster such as `&cb=$RANDOM` to the URL, or you may be looking at the old render.
- Useful test products: an in-stock single variant
  (`beavercraft-childrens-small-whittling-knife-with-safety-guard`), partly sold-out
  variants (`nalgene-wide-mouth-sustain-water-bottle-1-litre`,
  `osprey-rook-65-mens-backpack`), sold out (`kelty-late-start-4-footprint`), and a
  collection with sold-out items (`collections/beavercraft`). Stock changes, so re-check
  with `/products.json` if one no longer fits.
- For a visual check, headless Chrome can screenshot a staging page (it follows the
  preview redirect itself):
  `"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless=new --hide-scrollbars --window-size=1280,1600 --virtual-time-budget=15000 --screenshot=/tmp/shot.png "<preview url>"`.
  Desktop widths are reliable, but narrow `--window-size` values don't give a true mobile
  layout, so ask the user to check mobile on a real phone.
- Logic that depends on a setting that's off on live (e.g. superscript cents) can be
  unit-tested locally with `python-liquid` in a throwaway venv, stubbing Shopify-only
  filters such as `money`.
- Check Shopify Liquid object properties against the docs before relying on them. An
  unknown property such as `value.exists` is silently `nil`, not an error.

### 7. Publish (only with the user's explicit go-ahead)

Publishing changes the live store, so always ask first. Just before publishing, check
whether live was edited since step 5: re-run the step-2 pull and diff. If it was, create
a fresh duplicate and re-push the same `--only` files. It's cheap and avoids shipping
stale content. Then:

```bash
shopify theme publish --theme <new-id> --no-color
```

The old live theme stays in the library. **Rollback** means publishing it again
(admin → Online Store → Themes → ⋯ → Publish, or `shopify theme publish --theme <old-id>`).

### 8. Commit

Commit the code change to git (separately from the step-2 sync commit) and mention
the new theme name/ID in the message. Push to GitHub (`origin`) when the user asks.

## Storefront conventions worth knowing

- **AI assistants and scrapers read hidden markup.** They read `hidden` elements,
  `display:none`, `data-*` attributes and JSON blobs, and they don't run JS. Don't render
  notices that can't apply to the current product, even if they're hidden. Only output
  them when some variant could need them. Example: `snippets/backorder.liquid` renders
  only when a variant can be sold past zero stock.
- Store-wide shipping and returns facts (R75 courier, transit days, 30-day returns)
  are theme settings under **Theme settings → Shipping & returns (structured data)**,
  output as JSON-LD on every page by `snippets/structured-data-header.liquid`. If the
  shipping or refund policy changes, update those settings too.
- Prices are whole rands; the store never shows cents. The format is set in the Shopify
  admin (Settings → General → Currency display, using `{{amount_no_decimals}}`), not in
  the theme. Always output prices through Shopify's `money` filters (or
  `snippets/price-as-money.liquid`) so they follow that setting. Never hard-code or
  build price strings with decimals in Liquid or JS. JSON-LD prices stay exact
  (e.g. `"1339.00"`); that's correct.
- Stock is real-time and all stock is held at the Cape Town store. Storefront copy
  must never suggest drop-shipping or backorders for items that can't be backordered
  (inventory policy "deny").
