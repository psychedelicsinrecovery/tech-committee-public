# What Astra's native MCP can actually do

Live capability reference for Astra Pro's own MCP server (`astra-native-main` /
`astra-native-service`) — a separate integration from `emcp-tools`, covering theme-level design
settings in real depth. `emcp-tools` bundles a thin `astra-read`/`theme-read` pair that only reads;
this is the full read/write surface, ~80 tools per site. For how to connect, see
`HOW-TO-RECONNECT-MCP.md`.

## Typography

`get-font-body`, `get-font-h1` through `get-font-h6`, `get-font-heading` (the shared heading default),
`update-*` counterparts for each. `list-font-family` (what's available to pick from),
`get-font-google-local`/`update-font-google-local`/`flush-font-local` (serve Google Fonts locally
instead of from Google's CDN — a real privacy/performance lever), `get-font-preload-local`/
`update-font-preload-local`.

## Header & footer builder

`get-header-builder`/`update-header-builder` and the footer equivalents control *layout* (which
widgets sit in which zone — logo, nav, buttons, socials); `-design` variants control the visual
styling of those zones. `list-header-builder-setting` for discovering what's configurable before
setting it. `update-header-component` for a single component within the header without touching the
rest. This is the direct answer to a header/footer inconsistency between the main and service sites —
these tools can read one site's configuration and replicate it on the other.

## Global design tokens

`get-global-buttons`/`update-global-buttons` (site-wide button styling), `get-global-palette` (site
color palette — **read-only from MCP**: its update counterpart is excluded from this session's tool
list because its schema doesn't validate cleanly against the API; changing the palette currently
needs the Astra UI directly), `get-color-background`/`update-color-background`,
`update-theme-color` (a more targeted single-color update than the full palette).

## Layout

`get-container-layout`/`update-container-layout`, `list-container-setting`,
`get-paragraph-margin`/`update-paragraph-margin`, `get-link-underline`/`update-link-underline`,
`get-scroll-to-top`/`update-scroll-to-top`, `get-transparent-header`/`update-transparent-header`.

## Sidebar

`get-sidebar`/`update-sidebar` (on/off), plus `-layout`, `-width`, `-style`, `-sticky` variants —
full control over whether a sidebar shows, where, how wide, and whether it scrolls with the page.

## Per-content-type defaults

`get-single-page`/`update-single-page` and `get-single-post`/`update-single-post` (default layout
for pages vs. posts specifically), `get-blog-archive`/`update-blog-archive` (the blog index page),
`get-breadcrumb`/`update-breadcrumb`, `get-post-meta`/`update-post-meta` (per-post Astra-specific
metadata, distinct from `emcp-tools`' generic post fields).

## Site identity

`get-site-title-logo`/`update-site-title-logo` — site title, tagline, logo (including retina/mobile
variants), and their visibility per device (desktop/tablet/mobile independently).

## Performance

`get-performance`/`update-performance` — local vs. CDN font loading, font preloading. Verified live:
both currently disabled on the main site (`load_google_fonts_locally: false`,
`preload_local_fonts: false`) — a real, actionable performance lever sitting unused.

## What this doesn't reach

Content itself — no page/post creation, no Elementor widget manipulation. This is theme-level design
only. For content and Elementor building, see `CAPABILITIES-EMCP-TOOLS.md`.
