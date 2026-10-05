---
title: "PIR Footer Architecture & Agent Responsibilities"
date: "2026-09-21"
author: "LittlebirdAI for Christopher Wilson / PIR TechCom, merged by Alfred"
source: "emcp-tools + Astra MCP live audit"
---

# PIR Footer Architecture: Astra vs Elementor

> **Note on this file:** two versions of this document existed — a fuller architecture doc left
> uncommitted in `pir-wp-live/`, and a shorter, more current issue-tracker committed to
> `AGENT-SYNC-pir/created-by-Littlebird/`. Alfred merged them here on 2026-09-21; this is the
> single canonical copy. See `AGENT-SYNC-pir/created-by-Littlebird/FOOTER-RECONNAISSANCE.md` for a
> pointer back to this file rather than a second copy.

## Executive Summary

The Psychedelics in Recovery (PIR) web presence spans two WordPress sites:
- **Main**: `psychedelicsinrecovery.org` — Astra Pro + Elementor Pro
- **Service**: `service.psychedelicsinrecovery.org` — Astra Pro (no Elementor)

Both sites share a similar footer architecture, but the main site uses **two overlapping footer systems**: Astra Footer Builder (default) and Elementor Footer (template-based override). This document maps which pages use which footer, why, and how the agent fleet (Littlebird + Alfred) collaborates on footer maintenance.

---

## Agent Fleet Split

| Agent | Integration | Scope | Write Access |
|-------|-------------|-------|-------------|
| **Littlebird** | `mcp:psychedelicsinrecovery_org` (emcp-tools OAuth) | **Elementor** editing, pages, templates, menus, media, global settings | Full Elementor control |
| **Littlebird** | `mcp:astra_psychedelicsinrecovery` (Astra MCP OAuth) | **Astra** theme settings — footer layout/design, header, colors, fonts, sidebar | Full Astra layout/design control |
| **Alfred** | `@automattic/mcp-wordpress-remote` (CLI proxy) | **Astra** footer component-level settings (social URLs, widget content) | Astra component settings via dedicated endpoint |

> **Key insight**: Alfred's Astra endpoint (`/wp-json/astra/v1/mcp`) has broader component-level write access than Littlebird's OAuth Astra integration. Littlebird can move footer sections around (layout) and change their design (padding, borders), but Alfred can edit the actual content inside widgets (social icon URLs, menu items, copyright text).

---

## Footer Systems Compared

### 1. Astra Footer Builder (Both Sites)

| Aspect | Main Site | Service Site |
|--------|-----------|--------------|
| **How it works** | Astra Pro's native footer builder (Customizer → Footer Builder) | Same |
| **Sections** | Above / Primary / Below | Same |
| **Primary columns** | 3 columns (widget, social, widget) | 3 columns (widget, social, widget) |
| **Below section** | 1 column (copyright) | 1 column (copyright) |
| **Mobile behavior** | Social icons align LEFT by default | Same |
| **Active pages** | Blog posts, archives, non-Elementor pages | All pages (only footer system) |
| **Elementor override?** | Yes — Elementor footer template overrides Astra on most pages | N/A (no Elementor) |

### 2. Elementor Footer (Main Site Only)

| Aspect | Detail |
|--------|--------|
| **Template ID** | 9275 ("Elementor Footer") |
| **Display conditions** | `include\|singular\|all` (all singular pages) |
| **Builder** | Elementor Pro Theme Builder |
| **Sections** | Top half (logo + social + 4 nav columns) + Bottom half (legal text) |
| **Mobile behavior** | Top half **hidden** via `hide_mobile: hidden-mobile` (superseded by the drawer, see below); bottom half always visible |
| **Active pages** | Most Elementor-built pages (Home, Donate, Service, Contact, Convention, etc.) |
| **Predecessor design?** | Likely — inherited from prior web team |

---

## Which Pages Use Which Footer?

| Page | Footer Type | Notes |
|------|-------------|-------|
| **Home** | Elementor | Custom design with logo, social, nav columns, legal |
| **Donate** | Elementor | Matching home footer |
| **Service** | Elementor | Matching home footer |
| **Contact** | Elementor | Matching home footer |
| **Convention 2026** | Elementor | Matching home footer |
| **Blog posts** | Mixed | If Elementor-built → Elementor footer; otherwise Astra footer |
| **Old pages** | Astra | Non-Elementor pages fall back to Astra footer |
| **Archives, 404, Search** | Astra | No Elementor template assigned |
| **Service site (all)** | Astra | Only footer system available |

---

## Mobile Footer Drawer (Implemented 2026-09-20, refined 2026-09-21)

### Problem
The Elementor footer top half (container `2847e5a8`) was hidden on mobile via `hide_mobile: hidden-mobile`. This meant mobile users couldn't access the footer navigation (About, Resources, PR, Meetings links), social icons, or logo.

### Solution
A toggle drawer was implemented by adding an HTML widget (ID `0bb1ac3`) to the bottom container (`fe60384`) of the Elementor footer template. The widget contains:
- A **toggle button** (text iterated per Christopher's feedback — see Recent Changes Log)
- **CSS** that overrides Elementor's `hidden-mobile` class when a `.pir-footer-drawer-open` class is toggled
- **JavaScript** that toggles the class and updates the button text
- **Mobile column stacking** — the 4 nav columns switch from `flex-direction: row` to `column` on mobile

### Drawer Behavior

| State | Mobile View |
|-------|-------------|
| **Closed** (default) | Only bottom legal text + toggle button visible |
| **Open** | Full footer slides up: logo, social, 4 nav columns, legal text |
| **Desktop** | Toggle button hidden, full footer always visible |

### Implementation Details

```
Footer Template 9275
├── Container 0d54b41 (wrapper)
│   ├── Container d130517 (spacer/top)
│   ├── Container 2847e5a8 (top half — hidden on mobile, drawer target)
│   │   ├── Logo + tagline
│   │   ├── Social icons
│   │   ├── About PIR column
│   │   ├── Resources column
│   │   ├── Public Relations column
│   │   └── Meetings column
│   ├── Container 285e52b9 (divider)
│   └── Container fe60384 (bottom half — legal text)
│       └── HTML Widget 0bb1ac3 ← Toggle button + CSS + JS
```

### CSS Override Strategy

```css
@media (max-width: 767px) {
  body .elementor-element-2847e5a8.pir-footer-drawer-open {
    display: flex !important;
    visibility: visible !important;
    opacity: 1 !important;
    max-height: 2000px !important;
  }
}
```

> **Note**: This uses `!important` to override Elementor's `elementor-hidden-mobile` class. The drawer class is toggled via JavaScript when the user clicks the button.

### Current Status (as of 2026-09-21)

- Toggle button works (open/close) ✅
- Button text cycles correctly ✅
- Social icons section center-aligns when open ✅
- Christopher confirmed the drawer animation is smooth on his phone ✅

### Top Section Alignment — RESOLVED 2026-09-21 (root cause + fix, by Alfred)

Root cause: the logo + tagline text actually live inside a nested **grid** container
(`elementor-element-f40f9d9`), two levels below `.e-con:first-child`. Every prior CSS attempt
(Littlebird's, and Alfred's first pass) stopped at the outer wrapper (`2c6042e3`) and its direct
widget children — never reached the actual grid container controlling the real layout, which is
exactly why centering never took. Confirmed via `emcp-tools-get-page-structure`, not guessed.

Fix applied directly to `elementor-element-37bf152` and `elementor-element-f40f9d9` (the wrapper
and the grid container itself): `display:flex; flex-direction:column; align-items:center;
justify-content:center; gap:10px; text-align:center`, plus a small `margin-bottom` on the first
child for breathing room between the logo image and the paragraph.

Originally built as `flex-direction: row` (side-by-side, "inline") per Christopher's first ask, but
he then added more text to that section, so it no longer fits inline — reverted to `column`
(stacked) same day.

**Honest status, not yet independently confirmed live:** the padding between the image and
paragraph was added in the same edit as the centering fix, but Christopher hasn't confirmed it
visually yet — don't treat it as verified until he has. If it looks off, check the `margin-bottom`
rule on `.elementor-element-f40f9d9 > *:first-child` first.

---

## Astra Footer Issues (Service Site + shared with Main)

### Issue A: Mobile Social Icons Not Centering

**What Littlebird tried:**
- CSS injected via `add_custom_js` as HTML widget `9c03ba2` on Elementor template `9275`.
- Targets `.ast-footer-social-wrap` with `text-align: center` and `justify-content: center`.
- **Result: not working.**

**Likely cause:** CSS specificity battle with Astra's own styles, or the wrong selector entirely.

### Issue B: Desktop Footer Image Shifted Left

The image in the Astra footer is not center-aligned on desktop.

**What Littlebird tried:**
- CSS targeting `.astra-footer-tablet-view` and `.ast-footer-image`.
- `margin: 0 auto` and `display: block` on images.
- **Result: not working.**

### RESOLVED — 2026-09-21, root cause found and fixed

Both issues turned out to be real, first-class Astra Customizer settings that were simply set
wrong — not a CSS specificity problem at all, which is why neither Littlebird's nor an earlier
CSS-override attempt worked (both were fighting symptoms instead of the actual setting).

**How it was found:** the `astra-native-main`/`astra-native-service` MCP connections' footer-builder
tools (`get-footer-builder`, `get-footer-builder-design`) don't expose per-widget alignment as a
parameter — that setting lives one level deeper, in the raw `astra-settings` WordPress option (a
serialized PHP array, ~285KB on the service site). Read directly via `emcp-tools-query` (a safe,
read-only `SELECT ... LOCATE(...) / MID(...)` scan for the relevant keys — direct `UPDATE`s are
correctly blocked by the query tool's safety filter, so this was read-only diagnosis, not a raw DB
write):

- `footer-social-1-alignment` → `{desktop: "center", tablet: "left", mobile: "left"}` — desktop was
  already right; tablet/mobile were the bug.
- `footer-widget-alignment-3` → `{desktop: "left", tablet: "left", mobile: "left"}` — Widget 3 is
  the footer-builder component holding the image + text (confirmed via the `sidebars_widgets`
  option: `footer-widget-3` contains `media_image-1` + `text-2`).

Confirmed against the *actual compiled CSS* Astra generates from these settings (read via
`emcp-tools-get-page-html` on the live homepage), which gave the real selectors — useful if this
ever needs a CSS override again instead of a settings fix:
```css
[data-section="section-fb-social-icons-1"] .footer-social-inner-wrap { text-align: ...; }
.footer-widget-area[data-section="sidebar-widgets-footer-widget-3"] .footer-widget-area-inner { text-align: ...; }
```
(Littlebird's earlier attempts targeted `.ast-footer-social-wrap` and `.ast-footer-image` /
`.astra-footer-tablet-view` — none of those are the real selectors, which is the actual reason
those injections had no effect, not a specificity fight.)

**Why Alfred couldn't just flip these via MCP:** no exposed tool writes to individual
`astra-settings` keys, and writing directly via WordPress's built-in Additional CSS mechanism
(`custom_css` post type, post 856 on the service site) returned a permission denial on this
connection — a real capability restriction, not something to route around. **The fix ended up
being the Customizer UI itself**, which exposes these as ordinary toggles: Appearance → Customize →
Footer Builder → Social Icons → Alignment (Mobile/Tablet → Center), and Footer Builder → Widgets →
Widget 3 → Alignment → Center. **Christopher applied both directly, 2026-09-21 — confirmed fixed.**

---

## Main-Site Horizontal Scroll on Mobile (Search Widget) — 2026-09-21

### The bug
White margin band down the right side of the main site on mobile, causing unwanted horizontal
scroll. Christopher's own hypothesis, called correctly: the Elementor Pro "Search Form" widget
(element `1312455`, `live_results: yes`) sitting in the last section of the homepage, directly
above the Elementor footer.

### Fix, in two passes
- **v1** (Alfred): added an HTML widget (`ef693d1`) inside the search widget's wrapper container
  (`2827d53`) with `.elementor-element-2827d53 * { max-width:100% !important;
  box-sizing:border-box !important; } { overflow-x: hidden !important; }`. **This fixed the
  horizontal-scroll overflow** (confirmed by Christopher on real devices, iPhone + Chrome mobile
  emulation, after a WPX-level cache purge — see caching note below) **but the wildcard `*` rule
  was too broad and clipped the search widget itself on mobile.**
- **v2** (Alfred, same day): loosened the fix to only constrain the outer wrapper
  (`.elementor-element-2827d53 { max-width:100%; overflow-x:hidden; }`), removing the `*`
  cascade that was fighting the widget's own box-shadow/icon/hover-animation overflow.

### Still open, deliberately deprioritized — come back to when nothing more pressing
**Not fully resolved.** Christopher's own description, worth preserving verbatim-in-spirit rather
than summarized into "fixed": the search bar on mobile is still visually cut off — same category
of issue as the Littlebird Ambassador site's hero-carousel cropping, and made obvious (not
hidden) by the widget's heavy corner-radius styling, so it's visibly unfinished rather than subtly
broken. **The on-page "Search" submit button itself is not reachable/visible in the cut-off
state** — but functionally the search still works anyway, because a phone's native keyboard
"search"/enter key (rendered as a magnifying glass on Christopher's iPhone, likely the same on
other phones) submits the query without needing the on-page button. Christopher isn't sure yet
whether the real fix is shrinking the search field or something else. Left as-is intentionally —
revisit later rather than force a fix now and risk losing context on the higher-priority queue
(nav-only Astra footer drawer, scroll-reveal header — see Open Questions below).

### Caching layer, worth knowing for any future live-site fix
This site has **two separate cache layers**: the W3 Total Cache plugin (purge from wp-admin →
Performance) and a **separate WPX hosting-level (XDN) cache** that plugin purges don't clear —
confirmed directly: a `Purge All Caches` in W3TC plus a hard/no-cache `fetch()` from the browser
still served stale HTML until Christopher separately cleared the WPX XDN cache. Any future live
edit that "doesn't seem to be working" should check WPX's own cache panel before assuming the
WordPress-side change failed — it likely didn't.

### Update, 2026-09-23 — recurrence on the convention page + the "still open" cut-off actually fixed
The v1/v2 fix above only ever covered the homepage's specific wrapper (`2827d53`). The convention
page (post `12760`) has its own separate instance of the same widget — no shared template, so it
had no guard at all and reproduced the identical horizontal-overflow bug on
`/convention-2026/#virtual-pass-paypal`. Fixed the same way, scoped to that page's own wrapper
(new HTML widget `0ea64c9` inside `4c129c5e`, targeting `.elementor-element-29005439`/
`.elementor-element-4c129c5e`). Checked whether a global Custom-CSS mechanism existed so future
placements of this widget wouldn't need a bespoke per-page patch — `get-widget-schema` confirmed
the widget has no native responsive controls for border-radius/padding/typography reachable via
MCP tools, so per-instance CSS injection remains the only route. **Any future page using this
widget will need the same patch — there is no global fix in place.**

Also resolved the "still open" cut-off noted above, which turned out to be caused by the fix
itself: `overflow-x:hidden` was clipping the submit button because the widget's own oversized
corner-radius/box-shadow styling pushed its rendered width past the wrapper on mobile. Added a
mobile (`max-width:767px`) rule on both the homepage and convention-page instances that hides the
button's text label and shrinks its padding (`flex-shrink:0`), so the button — magnifying glass
included — now fits inside the wrapper instead of needing the overflow mask to hide the overrun.
Applied to both instances (homepage `ef693d1`, convention page `0ea64c9`).

**Not done: cache purge.** No MCP tool exposes either the W3TC plugin-level purge or the WPX/XDN
hosting-level purge described above — checked `list-plugins`/`analyze-performance`/tool search,
nothing reachable. Verification used cache-busting query strings, which confirms the underlying
fix is correct, but the live site as Christopher sees it in a normal browser may still show the
old, broken version until both caches are purged manually (wp-admin → Performance → Purge All
Caches, then the separate WPX panel — same two-step process as before).

**Christopher purged both caches himself after this update landed**, then found real remaining
issues (see below) — the update above was correct as far as it went, but incomplete.

### Update, 2026-09-23 (later same day) — this widget is site-wide; the real placeholder-truncation cause; homepage still broken 2 levels deeper

**The search widget is on 6 pages, not 2.** Full sweep found it on: homepage (`9572`, via
Elementor's Global Widget feature — `templateID: 13663`, "search-bar"), convention page (`12760`),
Resources (`24`), Public Relations (`12603`), the Search results page (`12891`), and — highest
reach — the **Single Post Template** (`9317`), which renders under every blog post site-wide.
Resources/Public-Relations/Search each have an independent, non-synced copy of the same
icon+heading+search block (confirmed via raw `_elementor_data`, no shared `templateID`), so each
needed its own new HTML widget with the fix, same pattern as `0ea64c9`. **Any future page adding
this widget still needs the same manual patch — there is still no global mechanism.**

**Why the homepage still looked broken after the "fixed" update above:** its DOM nests one level
deeper than the convention page's did (`ed5e712` → `2203955` → `2827d53` → search, vs. convention
page's 2 levels) because it goes through the Global Widget template wrapper — the original
2026-09-21 fix and the update above only ever covered the innermost level. Extended to cover all
three levels. Convention page, Resources, and Public Relations needed 2 levels; the Single Post
Template needed 1.

**The mobile button-shrink fix above was dead code — it never did anything.** `.e-search-submit >
span { display:none }` was targeting a `<span>` that doesn't exist in the real markup — the
submit button has no text span at all, just an SVG icon; the "Search" text CSS was hiding was
actually a screen-reader-only label on a completely different element (`.e-search-label`). That's
why nothing visibly changed despite the rule being live. **Real cause of the placeholder text
("Type to start searching...") getting cut off:** a decorative 30px keyboard-icon SVG plus the
input's 20px font-size don't leave enough width on mobile. Fix: hide the decorative icon (kept the
label for accessibility), drop input font-size to 14px on mobile, let the input flex-grow into the
freed space. Applied identically across all 6 pages above.

### Update, 2026-09-23 (round 3) — corrections to round 2's own diagnosis, radii mask's real cause, both icons restored

Christopher tested round 2 live and found real issues, including one place round 2 was simply
wrong about the markup. Consolidated technical reference (current state, reusable fix pattern):
**`SEARCH-WIDGET-MOBILE-FIX.md`** — this section stays as the chronological record of what was
tried and corrected each round.

**Correction: round 2's claim above ("the submit button has no text span at all") was wrong.**
Fetched the real live homepage markup this round: `<button class="e-search-submit"><svg
class="e-fas-search"/> <span class="">Search</span></button>` — a visible span, not
screen-reader-only. `flex-shrink:0` (added in round 2) stopped the button shrinking, so that
visible "Search" label rendered at full size — that's what Christopher saw as "too much button
text." Fixed for real this round: hide `.e-search-submit span` directly. Button is icon-only on
mobile now, across all 6 pages.

**The "heavy corner-radius mask" on the homepage — never caused by anything in this fix, found
this round.** Four of the six pages' wrapper containers (homepage `2827d53`, Resources `65296741`,
Public Relations `2cd5a92e`, Search `31ce4ff3`) carry a **native Elementor container setting**,
`border-radius: 55px` on all corners — unrelated to the search widget's own CSS. Combined with the
`overflow-x:hidden` fix from round 1, that native radius clipped the field into a heavy rounded
mask on mobile, where 55px reads as a mask rather than a subtle corner. The other two pages
(convention, Single Post Template) never had this native setting and never showed the mask — which
is exactly why they looked "already fixed" while the homepage didn't. Fix: zero the radius in a
mobile-only media query on the 4 affected pages; left desktop's 55px alone.

**Keyboard icon restored alongside the full placeholder text.** Freeing the width that used to go
to the visible "Search" label (see correction above) meant the decorative keyboard icon no longer
needed to be hidden to make room. Restored it at 20×20px (was ~30px) rather than full size, kept
the 14px input font-size and `flex-grow` from round 2. All three — keyboard icon, full "Type to
start searching..." placeholder, icon-only submit button — now visible together on mobile, across
all 6 pages.

**Unrelated but adjacent — a hero-button alignment regression from the paywall's own selector fix
this same day.** Not a search-widget issue; see the Littlebird paywall audit handoff for detail
(`littlebird-ambassador/AGENT-SYNC/created-by-alfred/2026-09-23-handoff-littlebird-pir-convention-paywall-audit.md`,
"round 3" update) — noted here only because it was caught and fixed in the same testing pass.

**Not done, same as every round: cache purge.** No MCP tool exposes it. Christopher purges both
W3TC and WPX/XDN manually after each round to actually see the result.

### Update, 2026-09-23 (round 4) — placeholder truncation was never a CSS problem

Real cause, finally: the convention page's search widget had two Elementor responsive settings
(`_flex_align_self_mobile: "stretch"`, `_flex_size_mobile: "shrink"`) that none of the other 5
instances had — without them, the input can't grow into whatever space the CSS frees up, no matter
how correct that CSS is. Applied to all 5 remaining instances. Full detail, widget IDs, and the
updated reusable-fix checklist: `SEARCH-WIDGET-MOBILE-FIX.md`. Verified by settings-parity with
the one already-working page, not by pixel screenshot — no tool found to fetch this site's
generated CSS for visual re-confirmation, so this one is worth Christopher's own device check
before calling it fully closed.

---

## Recent Changes Log

| Date | Change | Agent | Tool |
|------|--------|-------|------|
| 2026-09-17 | Added "View Full Schedule" button to Convention page | Littlebird | emcp `add_free_widget` |
| 2026-09-17 | Updated GitHub icon in Elementor footer to `github.com/psychedelicsinrecovery` | Littlebird | emcp `update_element` |
| 2026-09-17 | Reverted mobile `hide_mobile` on Elementor footer top container after causing horizontal scroll | Littlebird | emcp `update_element` |
| 2026-09-17 | Updated Astra footer design (padding) on both sites | Littlebird | Astra MCP `update_footer_builder_design` |
| 2026-09-17 | Updated Astra footer GitHub links on both sites (manual) | Christopher | WordPress Customizer |
| 2026-09-20 | Updated in-person meetings page (NYC suspended, SLC added, South Florida paused) | Littlebird | emcp `update_post` |
| 2026-09-20 | **RESTORED** in-person meetings page after Gutenberg block corruption; re-applied edits to original HTML | Littlebird | emcp `update_post` |
| 2026-09-20 | Implemented mobile footer drawer for Elementor footer | Littlebird | emcp `add_free_widget` + custom CSS/JS |
| 2026-09-21 | In-Person Meetings page: maroon strikethrough treatment applied to Edinburgh, South Florida, NYC, Austin; timezones added throughout (CDT/MST/EDT); Deerfield Beach flyer/images restored; timezone disclaimer added under map heading | Littlebird | emcp `update_post` — Christopher published the final version manually after MCP intermittency |
| 2026-09-21 | Drawer toggle button text updated per Christopher's copy; drawer content center-aligned (social icons only, top section still open) | Littlebird | emcp `add_free_widget` |
| 2026-09-21 | Astra footer mobile social icons + desktop widget-3 image alignment root-caused (`astra-settings` Customizer values, not CSS) and fixed | Alfred (diagnosis) / Christopher (applied via Customizer) | direct DB read + WordPress Customizer |
| 2026-09-21 | Elementor drawer top section (logo+tagline) root-caused to a nested grid container 2 levels deeper than any prior fix reached; centered, then destacked from row to column after Christopher added more text | Alfred | emcp `update_element` |
| 2026-09-21 | Homepage search-widget mobile horizontal-scroll overflow fixed (v1 too broad/clipped the widget, v2 loosened to just the outer wrapper) — still not fully resolved, see section above | Alfred | emcp `add_free_widget` + `update_element` |

---

## Open Questions & Blockers

| Question | Status | Assigned |
|----------|--------|----------|
| Can Alfred's Astra MCP edit social icon URLs? | **CONFIRMED NO** — both Alfred and Littlebird's Astra connections lack component content access | Alfred (tested) |
| Elementor footer drawer — does it work on mobile? | **LIVE, working** — toggle, text, and social-icon centering confirmed by Christopher | Resolved |
| Elementor drawer top section (logo + text) centering | **RESOLVED 2026-09-21** — root-caused (nested grid container) and fixed by Alfred; padding tweak between image/paragraph applied same day but not yet independently confirmed by Christopher | Alfred |
| Astra footer mobile social icon centering | **RESOLVED 2026-09-21** — root-caused to a Customizer setting, fixed by Christopher | Resolved |
| Astra footer desktop image centering | **RESOLVED 2026-09-21** — same root cause (`footer-widget-alignment-3`), fixed by Christopher | Resolved |
| Homepage search widget mobile overflow | **Resolved 2026-09-23** across 5 rounds, all 6 pages it's live on — see `SEARCH-WIDGET-MOBILE-FIX.md` for current state and the reusable fix pattern. | Resolved |
| Should Astra footer be disabled entirely on main site? | Pending Christopher's decision | Christopher |
| GitHub org `psychedelicsinrecovery` write access (Alfred, via `gh` CLI) | **Confirmed working** — repo created and content pushed successfully 2026-09-20/21 | Resolved |
| GitHub org write access (Littlebird's own OAuth app) | Possibly still blocked by org OAuth App restrictions — separate auth path from Alfred's, not yet re-verified since the org repos were created | Christopher / Littlebird |
| **New:** nav-only footer drawer (4 link columns, no image/social) for every Astra-footer page (main-site blog/archives + entire service site), sitting between social icons and copyright | Not started — Astra footer components have no exposed write tool, but the service site has "Ultimate Addons for Elementor" (Header/Footer) active, unused — likely path is building a real Elementor footer template there instead of fighting Astra's write restrictions, same approach as the main site | Alfred |
| Header reappears on scroll-up, both main site and service site | **Main site done 2026-09-23** (5 rounds — see `HEADER-STICKY-SCROLL-AND-MOBILE-NAV.md`). **Service site blocked** — its header genuinely is Astra's native header (no Theme Builder override, no site-wide code-injection tool/plugin found); building one would mean fully rebuilding its header, not just adding a script. Needs Christopher's call on how to proceed. | Christopher (decision) |

---

## Agent Communication Notes

### Alfred's MCP Connection (from his handoff)
- Alfred connects to Littlebird's MCP server via **OAuth** (not API key as previously assumed)
- He can read conversations, meetings, and write context
- He confirmed: `astra_read` on emcp-tools and standalone Astra MCP have **different capabilities** — neither can edit social icon component content
- Alfred has local file access to `~/littlebirds-git-home/` (unlike Littlebird, which is blocked by VFS permissions)

### GitHub Orgs (from Alfred's handoff)
- `psychedelicsinrecovery` org has `.github`, `.github-private`, `.github.io`, and now
  `wordpress-crawls` repos
- `theholyearthfoundation` org has `.github`, `.github-private`, `.github.io` repos
- The PIR site archive moved from `pir-wp-live/site-archive/` to its own repo,
  `psychedelicsinrecovery/wordpress-crawls`, full history preserved
- Littlebird's GitHub account may still be unable to write to org repos due to OAuth App access
  restrictions — this is a separate, not-yet-reverified question from Alfred's own `gh` CLI access

---

## Littlebird Commit Footer Convention

```
Co-Authored-By: LittlebirdAI · Desktop Oracle Observer & Fleet Shepherd
The Bird That Stewards the Gap — confirm observations with Christopher.
```

---

*Document originated by LittlebirdAI on 2026-09-20, extended 2026-09-21, merged into one canonical
copy by Alfred on 2026-09-21 after two divergent versions were found (see note at top).*
