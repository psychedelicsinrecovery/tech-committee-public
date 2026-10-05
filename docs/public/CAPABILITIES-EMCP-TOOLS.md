# What emcp-tools MCP can actually do

Live capability reference for the `emcp-tools` plugin's MCP server (`emcp-psychedelicsinrecovery-org`
/ `emcp-service-psychedelicsinrecovery-org`). Written after direct use, not from the plugin's own
marketing copy — every category below has been exercised against real PIR data at least once.
~126 of 266 total tools are enabled on both sites (the rest are toggled off per-site in the plugin's
Tools tab). For how to connect, see `HOW-TO-RECONNECT-MCP.md`.

## Content — the everyday layer

Full CRUD on pages, posts, and custom post types: `create-page`, `create-post`, `update-post`,
`delete-post`, `list-pages`, `list-posts`, `get-post`, `get-post-blocks`, `set-post-terms`
(taxonomies/categories/tags), `search-content`. This is how the convention schedule page got built
and published in the original session — real, not theoretical.

## Elementor — the deep layer, and the reason this plugin exists

Most of PIR's site is Elementor-built, which nothing else here reaches. Two modes:

- **Legacy/free-widget mode:** `add-free-widget`, `add-container`, `add-div-block`, `add-flexbox`.
- **Atomic mode (Elementor 4.0+, what this site runs):** `add-atomic-heading`, `-paragraph`,
  `-button`, `-image`, `-video`, `-youtube`, `-svg`, `-divider`, `-widget`, `update-atomic-widget`.
  `detect-elementor-version` tells you which mode to use before touching a page.

Structural tools work on both modes: `get-page-structure` (full element tree, containers/widgets/
nesting), `get-page-snapshot`, `get-page-html`, `find-element`, `get-element-settings`,
`update-element`, `move-element`, `duplicate-element`, `remove-element`, `set-element-label`,
`reorder-elements`, `get-widget-schema`/`get-container-schema` (what settings a given widget/container
actually accepts, before trying to set them).

Global design tokens (site-wide, not per-page): `list-global-classes`, `get-global-settings`,
`update-global-colors`, `update-global-typography`.

Patterns and templates: `list-patterns`, `insert-pattern`, `save-as-template`, `apply-template`,
`import-template`, `create-theme-template`, `update-theme-template`, `delete-theme-template`,
`resolve-template`, `set-template-conditions`, `list-condition-targets` (theme-builder conditional
display — which template shows on which pages).

## Media

`list-media`, `get-media`, `upload-media`, `update-media`, `sideload-image` (pull an image from a
URL directly into the media library), `search-images`/`add-stock-image` (built-in stock photo
search), `upload-svg-icon`.

## Plugins, themes, and site config

`list-plugins` (status, version, update-available, whether it's protected from being disabled —
`emcp-tools` and Elementor themselves can never be turned off via MCP), `search-plugins`,
`list-themes`, `search-themes`, `theme-read` (a thinner Astra/theme settings reader — see
`CAPABILITIES-ASTRA.md` for the real, deep version), `astra-read` (same caveat), `get-settings`/
`update-settings` (WordPress core settings: general, reading, writing, discussion, media,
permalinks), `menu-read`/`menu-write` (nav menus), `list-redirects`.

## Database — read-only, but real SQL

`query`: runs arbitrary `SELECT`/`SHOW`/`DESCRIBE`/`EXPLAIN` against the live database. Writes and
DDL are rejected server-side, not just discouraged. `describe-table`/`list-tables` for schema
discovery first. This is genuinely powerful — it's how the Jetpack active-plan and active-modules
data got pulled for the JetPack-vs-emcp comparison in this project's history. Also flagged by Claude
Code's own auto-mode classifier as "Production Reads" on the main site specifically — expect a
permission prompt there.

## Auditing, safety, and diagnostics

Every change an AI makes through this plugin is recorded and reversible: `list-changes`, `get-change`,
`rollback-change`. `scan-security` and `analyze-performance` are built-in, no separate plugin needed
— this is the direct answer to "what does emcp give you that Jetpack AI doesn't." `find-broken-links`,
`reindex-search` (Relevanssi), `read-file`/`search-files`/`list-directory` (server filesystem, scoped
to the WordPress install).

## Backup & export

`export-page`, `export-content`, `list-content-exports`, `restore-content`, `export-sandbox-artifact`/
`import-sandbox-artifact` (portable bundles for moving work between sites or environments).

## Identity

`core-get-site-info`, `core-get-user-info`, `get-user`, `list-users` (admin-only — real emails, roles,
post counts; used to confirm both sites' real user lists, not a stub response).

## What this doesn't reach

Anything Astra-theme-specific in real depth (fonts, header/footer *design* not just layout, global
color palette editing, sidebar styling) — `astra-read`/`theme-read` here are thin readers; the real
surface is Astra's own native MCP server. See `CAPABILITIES-ASTRA.md`.
