# Site header — sticky scroll-reveal + mobile pill nav layout, reference

**Purpose:** technical reference for the header work landed 2026-09-23. Two features, one repo of
discovery underneath both: **the site's real header is not where you'd expect to find it.**

## The header is an Elementor Theme Builder template, not Astra's native header builder

Astra's own header builder (`astra-get-header-builder`) is barely configured — just logo + menu —
and **is not what actually renders on the live site.** The real header is an Elementor Theme
Builder location template, **post `9227`**, condition `elementor-location-header`, confirmed
directly against live markup (not assumed from settings). Any future header change on this site
needs to go through this template, not the Astra header-builder MCP tools, or it will silently do
nothing.

Before this session, this template had **no sticky positioning at all** — it just scrolled away
normally with the rest of the page.

## Round 9 (2026-09-23, later still) — the actual root cause of every "unverified" fix across rounds 5-8, found and fixed

**Everything from rounds 5-8 was correctly written and correctly saved, and none of it ever
rendered live until this round — not a caching issue, a real one.** Christopher reported the
hamburger had vanished (round 8's overflow fix); a fork investigating that found, via direct grep
of the raw live HTML, that the entire `8bc364a` container — holding the scroll-reveal script
`9f86147` *and every CSS backstop written since round 5* (`d09a84e`) — had **zero occurrences** in
the live page, even though `get-page-structure` confirmed it was correctly saved in Elementor's
data the whole time. Round 6's "false alarm" conclusion (that a similar-looking absence was just a
verification-tool quirk) was wrong for THIS container specifically — it genuinely never rendered.

**Real cause: writing Elementor content via the MCP `update-element` API does not, by itself,
regenerate this template's live rendered output.** Confirmed by fixing it: opened
`/wp-admin/post.php?post=9227&action=edit` (the classic WordPress edit screen for this template,
reached by declining Elementor's own canvas editor — Elementor's in-canvas "Publish" button stays
greyed out with nothing to publish, since from its own perspective nothing changed) and clicked the
real **Update** button in the Publish meta box. That one click made `8bc364a`'s entire contents —
the scroll-reveal script and all four rounds of CSS backstops in `d09a84e` — appear in the live
page simultaneously, confirmed via direct JS inspection (`computed position: sticky`, `z-index:
999`, the injected `<style id="pir-header-mobile-pill-fix">` tag present with correct length) and a
live scroll test (hides on scroll down, reveals on scroll up even far from the top — confirmed via
`scrollY` + class-list checks, not just a screenshot).

**Same pattern, same fix, as the WPCode blocker below** — content saved via an API into certain
WordPress/Elementor systems needs a real save through the system's own UI to actually take effect,
regardless of how many times the API confirms the write succeeded. **This is the single most
important thing to know before doing more work on this header:** any future `update-element` call
against post `9227` should be followed by opening it at the URL above and clicking Update, or the
change will look successful via every MCP tool and still not be live.

**One more real bug found and fixed in the same pass:** the scroll-reveal script's original
selector was `header[data-elementor-id="9227"]` — correct, and confirmed present on the live
header element (`get-element-settings` showed this was never actually `#masthead`, unlike the
service-site version below; that assumption was checked and ruled out before use, not carried over
by mistake). No change was needed there — the selector was right all along; only the render-gap
above was the actual bug.

## Feature 1: header reappears on scroll-up

Standard pattern: hides when scrolling down, slides back in immediately on any upward scroll, even
before reaching the top.

**Implementation:** added `position: sticky` (not `fixed` — no manual body-padding offset needed to
compensate for removing it from flow) to the header template's root, plus a `translateY(-100%)`
hide/show toggle. New HTML/JS widget **`9f86147`**, inside new container **`8bc364a`**, placed at
the end of header template `9227` itself — this is the genuine global injection point, since the
template renders on every page via WordPress/Elementor's own location-template mechanism, not a
per-page widget that would need repeating (contrast with the search widget's per-page problem in
`SEARCH-WIDGET-MOBILE-FIX.md` — this is theme-level, so it doesn't have that limitation).

Scroll listener is debounced. Guards against hiding while the mobile menu is open by checking
Elementor's own `.elementor-menu-toggle[aria-expanded="true"]` state, not a custom/guessed class —
so opening the hamburger won't fight with the reveal/hide behavior.

Verified live in the served homepage HTML: fetched the page, found the exact injected script,
decoded the base64, confirmed it matches what was written byte-for-byte. Confirmed global by also
checking a second page.

## Feature 2: mobile pill nav — Contact/Donate now inside the pill

**The bug:** the mobile pill container (`eab435b` → `11608e2`, holding logo `b0c40e9` at 50% width
and hamburger `c48c922` at 50% width) never had room for the Contact/Donate buttons at all. They
actually lived in a **separate, mobile-only row** (`951a982`), pulled visually close to the pill
via a negative margin — which is exactly what "outside the pill" looked like: two independent DOM
elements styled to look adjacent, not one contained unit.

**The fix:**
1. New stacked-column container **`9bea1df`**, inserted between the logo and hamburger inside the
   pill row (`11608e2`).
2. Moved both buttons into it: **`0da0d00`** (Contact) and **`a0be9d3`** (Donate) — vertically
   stacked, not side by side.
3. Shrank logo to 30% width and hamburger to 22% width (from 50%/50%) to make room for the new
   column.
4. Restyled the buttons for mobile: 10px font, tight padding, full-width within the stacked
   column, alignment corrected from their old side-by-side `align:right`/`align:left` to
   `align:center` (a leftover from the previous layout that no longer made sense once stacked).
5. Set `flex_align_items_mobile: center` on the pill row itself so everything vertically centers
   against the (now taller, two-button) stacked column.
6. Hid the now-empty standalone row (`951a982`) on mobile — its buttons moved out of it, and
   leaving it in the layout would either show nothing or leave dead space.

Verified live: fetched fresh HTML, confirmed the DOM nesting is logo → stacked buttons → hamburger,
all inside `11608e2`, and confirmed the alignment-class corrections landed.

## Round 5 (2026-09-23, later same day) — the mobile pill regressed, and what actually fixed it

Christopher tested feature 2 live and found it badly broken: all 4 items (logo, Contact, Donate,
hamburger) were rendering stacked in a column instead of one row, making the pill much taller than
intended. Inspecting `11608e2`'s own settings showed `flex_direction: row` / `flex_direction_mobile:
row` were *already set correctly* — the failure mode wasn't visible from settings inspection alone,
meaning Elementor's own responsive width/flex controls (the 30%/22% width split from feature 2) are
fragile in a way that's hard to diagnose from the settings API. This is the **second** time this
session that tuning Elementor's native responsive settings produced a silent or hard-to-diagnose
failure (see the search-widget global-widget-reference gotcha in `SEARCH-WIDGET-MOBILE-FIX.md`).

**Fix approach, deliberately different this time:** instead of continuing to tune fragile
Elementor settings, added a new HTML/CSS widget **`d09a84e`** (sibling of the scroll-reveal script
`9f86147`, inside the same `8bc364a` container in header template `9227` — so it's global, not
per-page) with an unconditional, `!important`-forced rule:
```css
@media (max-width: 1024px) {
  .elementor-element-11608e2 {
    display: flex; flex-direction: row; flex-wrap: nowrap;
    align-items: center; justify-content: space-between;
  }
  .elementor-element-b0c40e9, .elementor-element-c48c922 { flex: 0 0 auto; width: auto; }
  .elementor-element-9bea1df { flex: 0 1 auto; }
}
```
This is a heavier-handed fix than "adjust one setting" — brute-forcing the layout via CSS rather
than trusting Elementor's own responsive controls to hold. Worth remembering as a pattern for this
site generally: if a native Elementor responsive setting *looks* right in the settings API but the
live result is still wrong, a forced CSS override in this same global widget is a more reliable
lever than continuing to adjust settings.

**A real discovery, not a convention-page bug:** what looked like "the convention page's mobile
header is different" was actually a **tablet-width breakpoint bug present sitewide**. A third,
separate header row (`59793aa`, holding un-stacked legacy button widgets in `113901f`) had
`hide_desktop` + `hide_laptop` + `hide_mobile` all set, but **not `hide_tablet`** — so it was
rendering, un-fixed by any of this session's mobile-pill work, at tablet-range viewport widths.
Christopher was very likely seeing this on the convention page by coincidence of viewport width,
not because that page is actually different. Fixed with an unconditional `display:none` on
`59793aa` in the same `d09a84e` widget, plus forcing `eab435b{display:flex}` at `≤1024px` so the
fixed pill (not this legacy row) covers the full mobile+tablet range.

**The empty vertical bar** was `951a982` (the old standalone row) — it had all 4 *standard*
Elementor hide flags set (desktop/laptop/tablet/mobile) and was still rendering, which suggests an
unlisted custom breakpoint Elementor's own settings didn't cover (not chased down further — not
worth the time once a working fix existed). Fixed with an unconditional `display:none !important`
in the same widget.

## Round 5 — service site scroll-reveal: investigated, blocked, needs a decision

Christopher asked for scroll-reveal on the service site too. Investigated fresh rather than
assuming main-site parity. Real finding: the service site's header **genuinely is** Astra's native
header builder — confirmed no Elementor markup in the rendered page, and
`list-theme-templates(type=header)` returns empty (unlike the main site, there's no Theme Builder
override here at all). That means there's no existing global injection point to add the
scroll-reveal script to, the way `8bc364a`/`9f86147` works on the main site.

Checked for alternatives: `add-custom-js` is explicitly per-page only on this install (would mean
repeating the script on every page, fragile and easy to miss new pages); no active plugin provides
a genuine site-wide code-injection point (checked `list-plugins` — no Code Snippets, WPCode, or
Insert Headers & Footers). The only way to get a real global hook would be building a brand-new
Elementor Theme Builder header template with an "Entire Site" condition — but Theme Builder headers
**fully replace** the theme's native header output (confirmed by the main site's own header being
exactly that kind of replacement), so doing this would mean rebuilding the service site's existing
Astra header content (logo, menu, whatever else it has) inside Elementor from scratch, not just
adding a script. That's a real scope jump — a header rebuild, not a scroll-reveal add-on — and
risky to improvise without direction. **Not done. Needs Christopher's call on how to proceed** (see
his open handoff/pending-tasks entry for the options).

## Round 6 (2026-09-23, later still) — real fixes made, but nothing confirmed live, and one genuinely concerning finding

Christopher reported the mobile pill was still broken (logo/hamburger not staying in one row,
hamburger barely visible under a heavy radius) and asked for icon-only Contact/Donate on
mobile+tablet, a tablet-width radius fix, and removal of a padding gap under the header.

**Real root cause found this round, via `get-element-settings`, not guessed:** the pill's own
outer wrapper (`eab435b`) had **`hide_tablet` set natively** — by Elementor's own visibility
system, the pill was never supposed to render at tablet width at all. Round 5's forced-CSS
override (`d09a84e`) had been fighting that native setting with `!important` rather than fixing
it. Also found `flex_wrap` was **never set** on the pill row `11608e2` — Elementor containers
default to `wrap`, which is the likely real cause of logo/hamburger wrapping onto separate lines
this whole time, through every prior round's CSS patches. Fixed natively this round:
`hide_tablet` cleared on `eab435b`, set on `59793aa` (a legacy row that was still missing it —
the actual cause of "layout looks different at some widths," confirmed sitewide, not page-specific),
`flex_wrap:nowrap` added at all breakpoints on `11608e2`, native `border_radius_tablet`/`_mobile`
reduced from 70px to 24px, and `d09a84e` rewritten with a thinned CSS backstop plus icon-only
Contact/Donate styling on mobile+tablet (inline SVG icons via a `content:url("data:image/svg+xml...")`
data URI, chosen over any icon-font class given this site's confirmed FontAwesome-webfont gap).
Also set `8bc364a` (the container holding the header's injected script/CSS widgets) to
`padding:0; min-height:0` to remove a visible empty strip under the header.

**None of this is confirmed live**, and one finding is more concerning than ordinary caching:
`59793aa`'s and `eab435b`'s hide-flag changes DID confirm live (correct classes present in a
cache-busted fetch of the homepage). But `8bc364a` — and both widgets inside it, including
`9f86147`, the scroll-reveal script that was confirmed working in rounds 4-5 — were **completely
absent** from that same fetch. `get-page-structure` (reads the saved Elementor data directly,
bypassing any front-end cache) confirms all three still exist correctly in the post's actual data,
so this isn't data loss from this round's edit. Cache-busting query strings did not resolve it.
**This needs a real check after Christopher purges caches — and specifically, if Elementor's own
"Regenerate CSS & Data" tool exists in this install's Tools menu, try that too, since Elementor
maintains its own generated-output cache separate from W3TC/WPX/XDN, and this symptom (correct
saved data, stale/missing rendered output) matches that class of problem more than a simple page
cache.** Until that's checked, treat the entire mobile pill (old and new parts alike) and the
scroll-reveal script as unconfirmed, not done.

**Service-site scroll-reveal (WPCode post 1693, "everywhere" location term): still unconfirmed,
now with a real root cause instead of a guess.** The service site's own W3TC instance is running
"Page Caching using Disk: Enhanced," which caches per exact query string — appending a
cache-busting parameter didn't bypass it, it just risked creating one more cached variant. Needs
an actual purge on the service site before this can be checked at all.

**Auto-updates for WPCode: not attempted, confirmed blocked.** `emcp-tools-query` is read-only by
design (SELECT/SHOW/DESCRIBE/EXPLAIN only) — there's no tool path to writing WordPress's
`auto_update_plugins` option, and no dedicated plugin-management write tool exists. This has to be
done manually in wp-admin (Plugins list → "Enable auto-updates" per plugin) on both sites. Done
manually by Christopher, 2026-09-23.

**Correction: round 6's "missing container" finding was a false alarm from the verification
method, not a real site problem.** Christopher purged both caches and ran Elementor's "Clear Files
& Data" tool (the current name for what was called "Regenerate CSS & Data" above) — neither
changed anything, because his hard refresh was already showing round 6's changes correctly the
whole time. The round-6 fork's cache-busted `get-page-html` fetch not showing `8bc364a` was a
verification-tool artifact, not evidence of a real bug. **Lesson for future rounds on this site:
trust `get-element-settings`/`get-page-structure` for confirming a write saved, and trust
Christopher's own device testing for confirming it renders — don't treat an MCP-tool fetch
discrepancy alone as proof something is broken.**

## Round 7 (2026-09-23, later still) — icons, radius, hamburger-width bug, and a real brand color

1. **Icons un-stacked to inline.** `9bea1df`'s two icon buttons were stacked in a column; changed
   to a row (`flex_direction: row`, 6px gap) now that they're icon-only and don't need the vertical
   space text used to require.
2. **Logo enlarged**, `b0c40e9`: mobile/tablet width increased now that the icon pair takes less
   room (final values balanced against the icon pair's own width — see the sizing note below).
3. **Real icons swapped in, not a fabricated SVG.** The round-6 fake heart/phone SVGs are gone.
   Found the actual source: desktop's Contact/Donate buttons (`5416d86`/`6766554`) use classic
   Elementor `button` widgets' own native `selected_icon` field — `fas fa-phone` and
   `fas fa-hand-holding-heart` — and the mobile pill's buttons (`0da0d00`/`a0be9d3`) are the
   **exact same widget type**, they just never had an icon set. Set the real icons via that same
   native field, matching desktop exactly, rather than reinventing an icon mechanism.
   **This also resolves an apparent contradiction with the paywall audit's FontAwesome finding —
   worth understanding, not just noting:** that finding (webfont class-mapping present, actual font
   file never loaded) was specific to the convention page's **atomic** `e-button` widgets (a newer
   Elementor 4.x widget family that strips `<i>` tags and can't reach Elementor Pro's own bundled
   icon library the same way). These header buttons are **classic v3 `button` widgets**, which use
   Elementor Pro's own properly-loaded icon font directly — a completely different, working code
   path. FontAwesome isn't broken sitewide; it's specifically unreachable from atomic widgets. If
   you ever need a real icon on an atomic widget again, this is the actual constraint to design
   around, not "FontAwesome doesn't work here."
4. **Corner radius reduced further**, `11608e2`: 24px → 12px.
5. **White strip under the header — best-effort fix, not confirmed.** Round 6 zeroed `8bc364a`'s
   own padding/min-height; that apparently wasn't the full source. Added explicit `margin:0` on
   both widgets inside it (`9f86147`, `d09a84e`), on the theory that Elementor's default per-widget
   bottom margin (~20px, applied regardless of container padding) was still contributing even
   though both widgets render no visible content. Plausible, not visually confirmed — if the strip
   is still there, the next thing to check is a `gap` setting on the header template's own
   top-level flex container between its sibling rows.
6. **Hamburger spanning the full viewport, fixed at the mechanism level.** Real cause:
   `75df872` (the nav-menu widget) had `full_width: "stretch"` set — an explicit Elementor Pro
   option that stretches the widget past its own container, ignoring `c48c922`'s width entirely.
   Changed to `"default"`. This is a confirmed root-cause fix, not a workaround.
7. **Background color — grey to the site's real active brand color.** Checked two sources rather
   than guessing: Elementor's kit primary (`#6EC1E4`, a light blue that appears unused site-wide)
   vs. Astra's actual active theme palette (`astra-get-global-palette`), which has
   `"Brand": "#1a6c7a"` — a teal close to the desktop Contact button's own color and clearly the
   site's real identity color, not the Elementor kit's unused default. Applied `#1a6c7a` to
   `11608e2`'s background, replacing `#383838` grey.

**Sizing correction made mid-round:** the icon pair's initial 16% width was too narrow for two
36px circular buttons plus gap: widened to 24%, and the logo's width was trimmed slightly from the
first pass (42%→38% mobile, 36%→32% tablet) to keep the row balanced at `flex_wrap:nowrap`.

## Round 8 (2026-09-23, later still) — an invalid enum caused the overflow regression; service-site scroll-reveal actually debugged to its real hard blocker

Round 7's hamburger fix (changing `75df872`'s `full_width` from `"stretch"` to `"default"`)
regressed into a horizontal-scroll overflow — a real recurrence of this whole effort's original
bug class, treated with matching urgency. **Root cause, confirmed against the widget's own
schema, not guessed:** `"default"` is not a valid value for this control at all — the real enum is
only `"yes"`/`""`. An invalid value left the widget in an undefined render state. Fixed to `""`
(the actual off-state), plus a defensive `overflow-x:hidden; max-width:100vw` backstop added to
`d09a84e` on `eab435b`/`11608e2` specifically because this bug class has recurred enough times this
session to warrant insurance, not just a settings fix.

**Icons still stacked (round 7's `flex_direction:row` apparently not enough):** found stale
`_element_custom_width_mobile: 100%` still set on `0da0d00`/`a0be9d3` from the old stacked-text-
button era — likely fighting the container's own row direction. Set `flex_wrap:nowrap` natively on
`9bea1df` and added a forced CSS backstop (`width:auto; flex:0 0 auto`) in `d09a84e`, given this
exact site's now well-established pattern of native settings not reliably taking effect on their
own. **Not visually confirmed** — this is the belt-and-suspenders approach, not a guarantee.

**Whitespace growing when the mobile menu opens:** no native "position" control exists for the
dropdown panel. Added CSS forcing `position:absolute` on the two most likely dropdown selector
patterns without confirming the exact live class name against markup this round — **this is the
weakest, least-confident fix in this round; treat it as still open if the strip keeps growing.**

**Navbar color — navy, from the real palette this time.** The active Astra palette has no purple
swatch at all. Used `#153243` ("Alternate Brand" — a genuine navy already defined in the site's own
palette), not the unused teal from round 7 and not an invented color. **If navy isn't actually
what you pictured, the convention page's own purple (`#7C62FF`) is the other real color already
live on this site — it's just not part of the palette system, so it wasn't the default pick.**

**Drawer radius:** native `dropdown_border_radius_mobile: 10px` set on `75df872`, plus a CSS
backstop alongside the dropdown-position fix above.

**Height:** not touched directly this round — reasoned that once icon un-stacking genuinely takes
effect, most of the excess height should resolve on its own; the hamburger's own touch target
(`toggle_size_mobile: 20px`) was already modest, not the driver.

### Service-site scroll-reveal — actually debugged this time, two real bugs fixed, one confirmed hard blocker remains

Christopher confirmed via real device testing (not just unconfirmed tooling) that this genuinely
wasn't working — that changed the task from "verify" to "debug and fix." Found and fixed two real
bugs, in order:

1. **Wrong WPCode location term.** `everywhere` is a **PHP-execution-only** location in this
   plugin (confirmed by reading its own source — it runs the snippet through
   `safe_execute_php()`), not a place to put frontend JavaScript. Corrected to
   `site_wide_footer` (hooks `wp_footer`) — which, worth noting for the record, was the *original*
   round-6 guess before it got "corrected" away to `everywhere` on the assumption that term didn't
   exist; it did exist, just for the wrong content type.
2. **Snippet content was corrupted, confirmed via direct SQL read.** `post_content` had
   HTML-entity-escaped `<script>` tags (`&lt;script&gt;` — WordPress's own content sanitization
   escaping it on save) *and* literal two-character `\n` sequences instead of real newlines — either
   alone would have broken it. Fixed by saving raw JS with no `<script>` wrapper (WPCode adds its
   own) and real newlines, verified via SQL (`has_real_newline:1, has_literal_backslash_n:0,
   has_html_entity:0`).
3. **Genuine hard blocker, confirmed via plugin source + a direct DB read, not a guess:** WPCode
   caches its active-snippet list in `wp_options.wpcode_snippets`, currently `a:0:{}` — completely
   empty — and that cache is rebuilt **only** by the plugin's own `WPCode_Snippet::save()` method,
   which runs exclusively through its admin-UI save action, never through a generic post update.
   `emcp-tools-query` is read-only, so there's no tool path to trigger that rebuild directly, and
   no cron/AJAX endpoint for it was found either. **This needs one manual step: open
   `/wp-admin/post.php?post=1693&action=edit` on the service site and click Save (or Update) once**
   — that alone runs the plugin's real save path and activates the snippet. Confirmed via 4 fresh,
   uniquely-query-stringed, timestamp-verified live fetches that nothing outputs before this step.
   Both underlying bugs are fixed; this is the only remaining blocker, and it isn't tool-solvable.

### Resolved, round 9 — the manual save happened, and found two more real bugs underneath

Christopher navigated wp-admin himself and hit **"Sorry, you are not allowed to edit posts in
this post type"** on the raw `post.php?post=1693&action=edit` URL — even as an admin. Real reason,
not a permissions bug: WPCode deliberately locks down direct post-type editing (code execution is
powerful enough that this is a sane default) and only allows saving through its own admin page,
**Code Snippets** in the sidebar (`admin.php?page=wpcode`), which is a *different* URL than the
one this doc previously pointed to. Opened the real snippet editor there and found two more issues
the SQL-level investigation above couldn't see:

1. **Insert Method was set to "Shortcode," not "Auto Insert."** In Shortcode mode, a snippet only
   runs where a `[wpcode id="1693"]` shortcode is manually placed in content — never automatically,
   regardless of what "Location" says. Nobody had placed that shortcode anywhere, so the snippet
   could never have run no matter how correct its content or location term were. Switched to Auto
   Insert with Location "Site Wide Footer" (confirming the round-8 fix to the location term was
   correct).
2. **A second content-corruption bug, introduced while fixing the first one.** Retyping the script
   through simulated keystrokes hit the editor's auto-closing-bracket behavior, which silently
   appended 7 stray closing braces and 2 stray closing parens after the real content — a JavaScript
   syntax error that prevented the entire script from executing (confirmed via brace/paren counting
   on the live script's actual text content, not assumed). **Fixed by writing directly to the
   editor's CodeMirror instance via its JS API (`document.querySelector('.CodeMirror').CodeMirror`,
   `cm.setValue(code)`, `cm.save()`) instead of simulating keystrokes** — this bypasses
   autocomplete/auto-bracket entirely and is the reliable way to enter code into this specific
   editor going forward, not just this once.

**Confirmed genuinely live and working**, not just saved: `getComputedStyle` on the real header
element (`#masthead` — this one IS the correct ID for the Astra-based service site, unlike the
main site) shows `position: sticky`, `z-index: 999`; a real scroll test (`window.scrollTo`-free,
actual mouse-wheel scroll via the automation tool) confirmed the header hides on scroll down and
reveals on scroll up even ~2800px down a ~3000px page — not just at the very top.

## Caveats
- **Rounds 1-8 verification was live-HTML/CSS-content confirmation via MCP tools only, not real
  browser rendering** — no MCP tool could fetch this site's generated CSS for a true visual
  re-check, so several "confirmed" claims in earlier rounds turned out to be wrong once round 9
  used actual browser automation (real scroll events, `getComputedStyle`, real login sessions) to
  check. **Prefer real browser verification over MCP-tool-only checks going forward when it's
  available** — it caught two real bugs (the render-gap, the WPCode Insert Method) that MCP-based
  checking alone had missed or misdiagnosed across multiple rounds.
- **The render-gap pattern (content saved via API, never live until a real UI save) is now
  confirmed to affect at least two systems on these sites: Elementor Theme Builder templates and
  WPCode snippets.** Assume it could apply to other Elementor templates or plugin-managed content
  types too, and check with a real UI save when something MCP-confirmed-saved doesn't seem live.

## Round 10 (2026-09-23, later still) — real mobile revealed two more real gaps

Christopher tested round 9's fix directly: the main site showed no change at all, and the service
site revealed correctly on desktop and desktop's device-emulation but not on a real phone.

**Main site — a mid-air collision, not a regression.** While Christopher was testing, his own
window happened to be at a narrow (~430px) width — the same shared browser this session's
automation was using ("the viewport change was me" — he was interacting with the same window, not
a separate device). At that exact width, a direct check showed the round-9 fix was in fact live
and visually correct — **except** it had also reactivated round 6-7's icon-only Contact/Donate
styling (hiding their text), which Christopher had since asked to reverse now that the layout had
room. That's a real, separate gap, not a caching issue: the icon-only CSS was dormant the whole
time `8bc364a` wasn't rendering, so "restore the text" was never actually applied — round 9's fix
brought back the *old* icon-only styling along with everything else, since both lived in the same
CSS file. Fixed: rewrote `d09a84e` to keep Contact/Donate as compact pill buttons with icon **and**
text together (both buttons already had appropriately-sized mobile typography/padding in their own
native settings — `typography_font_size_mobile: 10px` — this was never the constraint). Saved via
the same real-UI-Update pattern from round 9, then confirmed visually: "Contact 📞" and "Donate 🫴"
pill buttons, side by side, hamburger fully contained, no overflow.

**Service site — real mobile browsers need a more forgiving reveal trigger.** Reveals correctly on
desktop and Chrome's device-emulation mode, but a real phone's momentum/rubber-band scrolling
produces small, rapid scroll-direction flips that a single-sample `currentY < lastY` check reacts
to instantly on desktop (where scroll deltas are cleaner) but apparently never visibly resolves on
a real device. Emulation doesn't reproduce this — it fakes the viewport size, not the physics of
touch-scroll momentum. **Fix applied to both sites' scripts** (same underlying pattern, so both
needed it): replaced the single-sample direction check with delta accumulators — `upAccum`/
`downAccum` track cumulative movement in each direction, only toggling the header's hidden state
once accumulated movement exceeds 12px in one direction, resetting on any direction change. Also
clamps `scrollY` to `≥0` to guard against iOS Safari's elastic overscroll reporting out-of-range
values during the bounce, which could otherwise corrupt the accumulators right at the page's top or
bottom. **Not confirmed on a real device by Alfred — no physical phone available to test with.**
This is a well-reasoned fix for a well-documented class of mobile scroll-jitter bug, not a
guaranteed one. Christopher, please re-test on your actual phone and report back — if 12px still
isn't enough (or is too much and feels laggy), that threshold is the first thing to tune.

## Round 11 (2026-09-23, later still) — logo, hamburger, button gap; service-site reveal still open

Main-site polish requested after round 10: logo bigger, the hamburger visually reading as its own
control instead of blending into the pill, a bit more breathing room between Contact/Donate.

**What changed:** `b0c40e9` (logo) `width_mobile` 28%→33%, `width_tablet` 26%→31%. `9bea1df`
(icon+text button pair) `width` 46%→40%, `flex_gap` 6px→10px — freed by narrowing the button pair
slightly rather than shrinking the hamburger, given `c48c922`'s width already produced a real
overflow regression twice this session. `c48c922` (hamburger) given its own CSS "chip" treatment
in `d09a84e` — a background (`#2c5470`) distinct from the pill's own `#153243`, a softer 12px
radius (not the pill's full radius), and a small margin — since it's a flex child of the same
navy-background row, not literally outside the pill's DOM. Christopher's actual request ("drop
outside the navbar") is only partially honored by this — true separation would mean restructuring
the row, which wasn't attempted given the overflow-regression history in this exact area. If the
chip treatment doesn't read as distinct enough once actually seen, revisit with a real DOM
restructure, carefully, rather than another CSS-only patch.

**Verification hit real friction this round, worth understanding precisely.** After saving via the
classic-editor Update button (confirmed via a genuine `&message=1` redirect, and separately via
`get-post` showing the round-11 CSS text present in the live `post_content` field — the data is
unambiguously correct), a direct `curl` fetch of the homepage still showed the OLD version, missing
not just the round-11 addition but the entire `pir-header-hidden` script block. Clicking "Purge
from cache" first (confirmed via a real `w3tc_note=pgcache_purge_post` URL parameter) didn't
resolve it either. **This means W3TC's purge alone isn't sufficient for this specific kind of
change** — the WPX/XDN hosting-level cache (documented in `FOOTER-RECONNAISSANCE.md`'s caching
note) is the more likely remaining layer, since it's confirmed to require its own separate purge
and isn't reachable by any available tool. **Christopher: please do your usual full purge (W3TC +
WPX panel) and confirm** — the underlying fix is verified correct at the data level, this is
purely about which cache layer is still serving the old version.

**Service-site real-mobile reveal — no new lead found, still genuinely open.** Checked for a
mobile-redirect/AMP plugin (`list-plugins` — none present) and a Content-Security-Policy header
that could block inline script execution on real mobile specifically (`curl -I` — no CSP header at
all). Neither explanation holds up. The round-10 delta-accumulator fix remains the most credible
attempt so far, but per Christopher's own choice this round ("blind attempt, revisit later"), this
stays flagged as unresolved rather than layering on a fourth unconfirmed guess. Real next step,
whenever convenient: Mac + Safari Web Inspector connected to the actual phone, to read real console
output instead of continuing to infer from desktop/emulation behavior that's already been shown not
to reproduce the actual bug.

## Round 11 follow-up (2026-09-23, urgent — member-facing usability) — real bugs found, plus a significant systemic caching finding

Christopher reported the hamburger had become "all but useless" — only reachable by scrolling
within the pill, visible on only the right half of its own container, and never actually grew
despite round 11's chip styling. Three real, confirmed causes, not guessed this time:

1. **`toggle_size_mobile` had never actually been changed** — every prior round touched the pill
   container's width, never the nav-menu widget's own icon-size setting. Fixed natively:
   `75df872`'s `toggle_size_mobile` 20px → 32px.
2. **`c48c922` had `flex_align_items_mobile: "flex-end"`** — pushing the icon toward whichever edge
   of its container is clipped/scrollable if the row overflows at all, instead of staying safely
   centered. Fixed natively to `"center"`.
3. **Belt-and-suspenders, given this exact row has overflowed multiple times this session:** added
   `flex-shrink:0` and `z-index:9999` on `c48c922` so nothing can squeeze or cover it, plus a hard
   `max-width` + `text-overflow:ellipsis` on the Contact/Donate buttons so their text can never push
   the hamburger out of the visible area again, regardless of what else changes in this row later.

All three confirmed saved via a direct `get-post` read (`modified` timestamp advanced, new CSS text
present in `post_content`) — the browser-based `&message=1` URL check used in earlier rounds proved
unreliable in this specific session (likely contention from a shared browser window), so this round
leaned on the database read as the source of truth instead.

**Significant finding, worth real attention:** while chasing why native-setting changes (like
`toggle_size_mobile` in earlier attempts) didn't visibly land, found that **every static asset on
this site — including Elementor's own per-post generated CSS files — is served with zero
cache-busting version query strings** (`?ver=...`), confirmed by grepping the full served HTML for
`.css?ver=` / `.js?ver=` and finding literally none, not even on WordPress core's own
`jquery.min.js`, which always carries one by default. Elementor's generated CSS file
(`wp-content/uploads/elementor/css/post-9227.css`) is also served with `cache-control: public,
max-age=31536000` — a full year. Combined, this means: once a browser (including a real visitor's,
or Christopher's own) has loaded a given CSS/JS file URL once, it will keep serving that exact
cached copy for up to a year, with **no mechanism to ever learn a newer version exists**, since the
URL itself never changes to signal that. This is very likely a WPX Cloud edge-level optimization
(the server header confirms `WPX CLOUD`), not a WordPress or Elementor setting — checked Autoptimize
and W3TC's own options tables directly, neither has an active "strip query strings" setting that
would explain this. **This is worth Christopher checking in the WPX hosting panel specifically** —
if there's a "remove query strings from static resources" or similar toggle, disabling it would
fix cache-busting properly and permanently, for every future change, not just this session's.
Until then, a genuine hard refresh / cache-clear on any given device remains the only reliable way
to see a CSS/JS-file-level change — the full-page HTML cache (W3TC + WPX/XDN) is a separate, third
caching layer on top of this, affecting the inline `<style>`/`<script>` content specifically.

## Round 12 (2026-09-23, urgent — member-facing) — the actual root cause of "stuck inside the navbar," found

Christopher checked the WPX panel — purged it, no "remove query strings" toggle found there, so
that specific lever isn't available to him (noted, not chased further). More urgently: the icon
did get bigger but "undesirably," and the dropdown was still visually part of the pill. Asked
directly for it to become its own component.

**Real root cause, finally found — not a caching issue at all.** `eab435b` (the outer mobile-row
wrapper) has its own native settings: `overflow: hidden` **and** `border_radius_tablet: 50px` — it
is itself a rounded, clipping pill, wrapping `11608e2` (which has its own navy `#153243` background
+ 12px radius) as a **second, inner pill**. Every round's attempt to give `c48c922` (the hamburger)
a visually distinct "chip" look — round 11's background/radius/margin, all of it — was still a flex
child living *inside both* of these nested clipped containers. Nothing could ever visually escape
`eab435b`'s own `overflow:hidden` boundary no matter what was applied to `c48c922` itself. That is
the actual reason it always read as "part of the same pill" — a real structural fact, confirmed via
`get-element-settings` on `eab435b` directly, not a caching artifact.

**Fix:** `c48c922` set to `position: fixed` — pinned near the header's top-right corner (`top:14px;
right:14px`), 44×44px, its own navy background + a lighter blue border + shadow, genuinely floating
independent of the pill entirely. A fixed-position element establishes its own containing block
relative to the *viewport* and is not clipped by an ancestor's `overflow:hidden` (true unless an
ancestor sets `transform`/`filter`/`perspective` — checked `eab435b`, none of those are set, so
this is safe). This is a structural fix Christopher asked for directly ("make it its own
component") after two rounds of in-flow styling couldn't achieve real separation — the right call,
not a workaround.

Icon size (24px) and the hamburger's own visual size are now set **directly in this same CSS
block**, not left to `toggle_size_mobile` alone — that native setting goes through Elementor's
separately-cached per-post CSS file, the one confirmed to have no cache-busting query string. This
inline `<style>` tag, by contrast, is embedded directly in the page HTML, so it only depends on the
full-page cache (W3TC + WPX/XDN) — the layer Christopher does have a working purge routine for.
Putting the icon sizing here too means it isn't hostage to the un-purgeable static-asset cache.

The dropdown menu (opens on tap) now positions relative to `c48c922` itself, since it's the
nearest positioned ancestor once fixed — it drops down from the now-independent button, not from
the old pill-relative offset.

**Confirmed saved via a direct database read** (`get-post` shows the round-12 CSS in live
`post_content`, `modified` timestamp advanced) — same verification approach as round 11's
follow-up, since the browser-based save-check has proven unreliable this session.

## Round 13 (2026-09-23, urgent, immediate follow-up) — round 12's fix caused three new regressions, all fixed in one combined pass

Christopher reported all three immediately after round 12 went live: Contact/Donate shifted hard
right instead of centered, the logo still hasn't visibly grown, and the hamburger "isn't seen at
all anymore" / "cut off running off the bottom" — plus a new explicit request that the dropdown
menu span the full width of the screen when open. Each traced to a real, specific cause rather than
re-guessed:

1. **Contact/Donate pushed to the right edge.** `11608e2`'s native `flex_justify_content_mobile` is
   `space-between`. With 3 flex children (logo, buttons, hamburger) that put the buttons in a
   natural visual middle; pulling `c48c922` out of flow via `position:fixed` in round 12 left only
   2 children, and `space-between` snapped them to the two extreme ends instead. **Fix:** reserved
   the hamburger's old ~60px footprint as `padding-right: 60px` on `11608e2` itself, restoring the
   same effective end-anchor `space-between` used to produce with 3 items — without touching the
   native justify-content setting, which is correct and shouldn't change.
2. **Logo still not visibly bigger**, despite `b0c40e9`'s native `width_mobile: 33%` being
   double-confirmed correctly saved via `get-element-settings`. Same suspected cause as the
   hamburger icon before round 12's fix: Elementor's own per-post generated CSS file
   (`post-9227.css`) is served with a 1-year cache and zero cache-busting query string (documented
   above) — a *data-confirmed-correct, delivery-path-unconfirmed* problem, not a settings problem.
   **Fix:** moved logo sizing into the same reliably-delivered inline `<style>` pathway already
   used for the hamburger icon — `width: 40% !important` on `b0c40e9`, `width:100%` on its `img`/
   `svg` — so it no longer depends on the un-purgeable static-asset cache at all.
3. **Hamburger "isn't seen at all" / icon cut off running off the bottom.** The round-12 button had
   no `overflow:hidden` of its own, so if the toggle's real icon (rendered as
   `elementor-menu-toggle__icon--open`/`--close`, `e-font-icon-svg e-eicon-menu-bar`/`e-eicon-close`
   per an earlier `curl` finding) came in larger than expected, it could spill past the 44×44px
   box's edges rather than being clipped or scaled to it. **Fix:** added `overflow:hidden` to the
   button itself, and broadened/tightened the icon-selector rules to cover every element type
   Elementor's toggle can render (`svg`, `i`, `span`, plus the specific `--open`/`--close` classes)
   with a hard `22px` cap and `flex-shrink:0`, so nothing inside can ever exceed the button's own
   bounds regardless of the icon's natural size.
4. **Dropdown menu made full-width**, per Christopher's explicit new request — changed from
   `position:absolute; right:0` (small anchored box) to `position:fixed; left:0; right:0;
   width:100vw`, dropping edge-to-edge below the now-fixed hamburger button (`top:66px`, clearing
   the button's own `top:14px` + 44px height + margin) rather than being anchored relative to it.

All four changes shipped together in a single `d09a84e` inline-CSS rewrite (same widget as every
prior round's CSS fixes), saved via the classic editor `#publish` click, and **confirmed via a
direct database read**: `modified` advanced to `2026-09-23 19:45:19` and the round-13 block is
present verbatim in live `post_content`. This confirms the WordPress-side save is real. It does
**not** by itself confirm the visual result on a real mobile device — per this session's own
repeated lesson (round 9's "false alarm" that turned out to be a real render gap, and the
un-purgeable static-asset-cache issue documented above), a database-level save confirmation and a
real-device visual confirmation are two different things. **Christopher: please hard-refresh / do a
real-device check once you've cleared W3TC + WPX/XDN (your usual purge routine) — that's the one
verification step I can't do from here.**

## Round 14 (2026-09-23, urgent, immediate follow-up) — the real reason iPhone showed nothing, found and fixed structurally; not guessed

Christopher's report was precise enough to actually diagnose instead of guess again: hamburger
invisible on real iPhone but visible (if janky) in Chrome's mobile emulator; button cut off at the
bottom and not sharing a centerline with Contact/Donate (shifted down); corners rounded on top only,
not bottom; color reading as something other than what round 13 set. This round used the browser
tools directly instead of relying on Christopher's own device to find the cause:

**Investigation.** Fetched the live homepage HTML with `cache: 'no-store'` and confirmed round 13's
CSS (`padding-right:60px`, the full `overflow:hidden` button block) was genuinely being served, not
stale-cached — ruling out caching as this round's cause. Enumerated every `<style>`/`<link>` node in
the live DOM: `post-9227.css` (Elementor's own generated per-post stylesheet) loads at index 98 in
document order; our inline `pir-header-mobile-pill-fix` tag loads at index 131, after it — but
Elementor's own generated selectors for elements are compound, two-class selectors
(`.elementor-element.elementor-element-c48c922`, specificity 0,2,0), while every round's CSS so far
used a plain single-class selector (`.elementor-element-c48c922`, specificity 0,1,0). Under `!important`
vs `!important`, source order only decides ties at *equal* specificity — Elementor's own rules could
still win outright on specificity alone for any property both happened to set, independent of load
order. This had been a silent, unconfirmed risk sitting under every round since round 9.

**Root cause 1 — invisible on iPhone, not a CSS problem at all.** `c48c922` (the hamburger) was
still a DOM descendant of `eab435b`, which has native `overflow: hidden`. Per spec, a
`position: fixed` element's containing block is the viewport, skipping non-positioned/
non-transformed ancestors — `overflow: hidden` on an ancestor *shouldn't* clip a fixed descendant.
But iOS Safari has long-documented, real bugs where it clips `position: fixed` descendants of an
`overflow: hidden` ancestor anyway, particularly inside compositing layers — almost certainly why
spec-compliant desktop Chrome rendered something while a real iPhone rendered nothing. **Fixed
structurally, not with more CSS overrides:** used `mcp-tools-move-element` to physically relocate
`c48c922` in the actual page tree from two levels inside `eab435b` (`eab435b > 11608e2 > c48c922`)
to a top-level sibling of `eab435b` itself. There is no `overflow: hidden` ancestor above it
anymore — nothing for the iOS clipping bug to attach to, regardless of browser quirks. Its native
`hide_desktop`/`hide_laptop` visibility classes travel with the element, not its DOM position, so
desktop/tablet/mobile show/hide behavior is unchanged.

**Root cause 2 — centerline mismatch, corner asymmetry, color.** A hardcoded `top:14px; right:14px`
guess has no way to track the row's actual on-screen position, especially on iOS where the dynamic
Safari toolbar changes the visual viewport height under the button's feet as the page scrolls — this
is the real reason it drifted from Contact/Donate's centerline. The corner-radius asymmetry and
color mismatch are consistent with the same specificity gap identified above: Elementor's own
per-post CSS very plausibly carries its own (higher-specificity) opinion on parts of this element
that our single-class selector wasn't reliably beating. **Fixed:** dropped `!important` from
`top`/`right` in the static CSS (kept only as a pre-JS-load fallback) and added a small
`requestAnimationFrame` loop (in the same `d09a84e` HTML widget, as a `<script>` after the
`<style>`) that reads `.elementor-element-11608e2`'s real `getBoundingClientRect()` every frame and
writes matching `top`/`right` values as inline styles — an inline style always beats a non-`!important`
stylesheet rule, so this wins unconditionally once it runs, and it stays correct through the header's
own scroll-reveal transform, window resizes, and iOS viewport changes without needing to predict any
of them. Every selector touching `c48c922` was also rewritten with the class repeated once
(`.elementor-element-c48c922.elementor-element-c48c922`, specificity 0,2,0) — a standard, valid way
to safely out-specify Elementor's own compound-class selector regardless of load order, closing the
gap found during investigation. Corner radius set via both the shorthand and all four explicit
longhand properties (belt-and-suspenders against the same risk), uniform at 12px, matching the navy
pill row's own radius. Color: purple (`#7C3AED` background, `#A78BFA` border) — per Christopher's
explicit request, a deliberate signal color to make a round's CSS visually obvious once it lands,
and a genuine differentiator from the service subdomain's green-blue.

**Confirmed live on the actual served front-end, not just the database:** fetched the homepage HTML
directly (`cache: 'no-store'`) after publishing and verified in the raw response: the boosted
selector, `#7c3aed`/`#a78bfa`, uniform corner-radius properties, the `requestAnimationFrame` script,
and exactly one `data-id="c48c922"` node in the whole page (confirming the move relocated rather
than duplicated it). This is a stronger verification than prior rounds' database-only check, since
it confirms what a real browser actually receives. **What's still unverified is the visual result on
Christopher's own iPhone** — the structural fix (escaping the overflow:hidden ancestor) should
resolve the "not rendering at all" report on principle, and the rAF sync should resolve the
centerline mismatch on principle, but neither replaces an actual device check.

## Round 15 (2026-09-23) — hamburger confirmed visible on real iPhone; refinement pass

Christopher confirmed round 14 worked: **the hamburger is finally showing on his real iPhone.**
Refinement requests from there: close the visible gap between the button and the row it sits in,
darker purple (the round-14 shade read as too bright), extend that same purple to the dropdown's
current-active-page highlight, round all four corners of the button (not just top), nudge
Contact/Donate a touch further left, and the logo "still doesn't look any bigger" despite round
13/14's width bump.

**The logo finding is the important one.** Checked the logo's own widget settings
(`get-element-settings` on `08239b5`, the actual `theme-site-logo` widget inside `b0c40e9`): its own
`width` is `{unit:"%", size:100}` — no independent cap on the widget itself. That rules out a
widget-level ceiling and points at the exact same cause round 14 found for the hamburger: our CSS
selectors have been plain single-class (`.elementor-element-b0c40e9`, specificity 0,1,0) the whole
time, while Elementor's own generated per-post CSS uses compound two-class selectors
(`.elementor-element.elementor-element-b0c40e9`, specificity 0,2,0) that can win outright regardless
of `!important` vs `!important` or source order. This was very likely the real reason the logo bump
never visibly landed across rounds 11, 13, *and* 14 — the fix just hadn't been applied to this
element's selectors yet. **Fixed:** repeated the class token
(`.elementor-element-b0c40e9.elementor-element-b0c40e9`) the same way round 14 did for the
hamburger, and the same guard was added to `11608e2`'s `padding-right` selector too, defensively.
Bumped the container to 50% width (from 45%) at the same time.

**Gap + centerline:** the round-14 script vertically centered a fixed 44px button inside the row,
which necessarily leaves visible space above/below whenever the row is taller than 44px. Changed the
script to size the button to the row's own height instead (clamped 36–64px) and pin `top` flush to
the row's top edge — the button's height now equals the row's height by construction, so there is no
gap to leave, rather than a value to guess.

**Corners:** applied the same explicit shorthand + all-four-longhand radius directly to
`.elementor-menu-toggle` (the inner button element), not just the outer `c48c922` container as round
14 did — belt-and-suspenders in case the inner element's own native `toggle_border_radius_tablet: 0`
setting (found via `get-element-settings` on `75df872`) was ever showing through despite the outer
container's `overflow:hidden`.

**Color:** darker purple, `#5B21B6` background / `#7C3AED` border (from round 14's brighter
`#7C3AED`/`#A78BFA`). Also discovered the real native property for "current active page" —
`color_menu_item_active`/`pointer_color_menu_item_active` on `75df872`, currently bound to a global
color (`astglobalcolor5`) with **no mobile-specific variant**, meaning changing it directly would
also change the *desktop* menu's active-page color, which wasn't asked for. Left the native setting
alone and instead added scoped CSS targeting Elementor's own `.elementor-item-active` class, wrapped
in the existing `@media (max-width: 1024px)` block, boosted with the same repeated-class guard on
`75df872` — same purple, mobile/dropdown only, desktop untouched.

**Dropdown/button overlap:** round 14 left the dropdown's opening position as a hardcoded
`top: 66px` while the button's own position became dynamic — a real latent inconsistency once the
button's actual height/position could vary. The sync script now also measures the button's live
`getBoundingClientRect()` and sets the dropdown's `top` to sit exactly `8px` below it, so the two can
never visually collide or overlap regardless of the row's real height on any given device.

**Confirmed live on the actual served front-end** (not just the database): fetched the homepage
HTML directly after publishing and verified in the raw response — `#5b21b6`, `#7c3aed`,
`width:50%`, `padding-right:74px`, the boosted logo selector, the `.elementor-item-active` CSS
block, the dynamic-height script logic, and the dropdown-sync logic are all present. Real-device
visual confirmation of *this* round's changes specifically is still Christopher's to do, though the
core "does it show at all" question is now settled.

## Round 16 (2026-09-24) — dropdown scroll, expand-on-open, darker purple, navy tap state

Fast turnaround while Christopher was demo-prepping with Kenney. Fixes:

- **Dropdown wasn't actually scrollable.** `overflow-y:auto` had no `max-height` to scroll against,
  so a long menu just made the whole page tall instead of scrolling internally. Now `max-height` is
  set dynamically in JS (`100vh - dropdown's real top - 16px`), so it genuinely scrolls once content
  exceeds available screen height — needed for menus that grow with blog/content items.
- **Button expands slightly on open**, per explicit request as a visual cue that content
  expanded — JS checks `aria-expanded` on the toggle and grows the button by 6px with a CSS
  `transition` for a smooth animate-in rather than a jump.
- **Purple darkened again**: `#4C1D95`/`#5B21B6` (one step down from round 15's `#5B21B6`/`#7C3AED`),
  applied consistently to both the button and the dropdown's active-page highlight.
- **Tap/hover state on dropdown items was showing native teal-green**
  (`background_color_dropdown_item_hover:#275B5B`, confirmed via `get-element-settings` on
  `75df872`) — the *service* subdomain's color, not this site's. Forced to PIR's own navy
  (`#153243`) via scoped CSS on `:hover`/`:active`/`:focus`, mobile-only.
- **Gap between dropdown top and navbar bottom tightened**, and made more robust: the dropdown's
  position now tracks the *row's* real bottom edge directly (`rowRect.bottom`) instead of the
  button's own (clamped 36–64px) height, which could under-represent an unusually tall row. `GAP`
  reduced from 8px to 2px.
- Rounded-corner asymmetry between the highlighted item and the grey backdrop — Christopher
  confirmed this is intentional/acceptable, no change needed.

Confirmed live via direct HTML fetch (same method as rounds 14-15): all six markers
(`#4c1d95`, `#5b21b6`, `#153243`, the dynamic `max-height` calc, `aria-expanded` logic,
`-webkit-overflow-scrolling`) present in the served response.

## Round 17 (2026-09-24) — drawer was scrolling the page underneath it, not itself; fixed with a real iOS scroll lock

Round 16 gave the dropdown `overflow-y:auto` + a dynamic `max-height`, but on real iPhone touch
gestures inside the open drawer were scrolling the *page behind it*, not the drawer's own content —
a well-known iOS Safari behavior where a fixed-position overlay doesn't reliably capture its own
touch-scroll unless the page behind it is explicitly locked. Plain `body { overflow: hidden }` is
not fully reliable on iOS; the standard, more robust fix is to pin the body itself with
`position: fixed` (recording and restoring the scroll offset) while the drawer is open. Added to
the sync script: `lockBody()`/`unlockBody()`, fired once per open/close transition (tracked via the
toggle's `aria-expanded`, not every animation frame). Also added `touch-action: pan-y` and
`overscroll-behavior: contain` on the dropdown itself as belt-and-suspenders so iOS routes the
gesture to the scrollable element specifically.

Also added a `transition: max-height 0.25s ease` on the dropdown so it **unfolds** smoothly when
its own content grows (e.g. a nested submenu expanding inline pushes its natural height up toward
the already-computed ceiling) rather than snapping to the new size instantly — the same idea as
round 16's button-grows-on-open, applied to the drawer itself.

Confirmed live via direct HTML fetch: `lockBody`/`unlockBody`, `touch-action:pan-y`,
`overscroll-behavior`, the `pir-menu-open` class, and the `max-height` transition are all present
in the served response.

## Round 18 (2026-09-24) — closing out the saga: real cause of "scrolls a tiny bit then snaps back"

Christopher's precise report on round 17: the drawer scrolled "a very tiny bit" then snapped back
to its start, never reaching the lower menu items — round 17's scroll-lock was necessary but not
sufficient. **Real cause, found by re-reading round 17's own script:** the `requestAnimationFrame`
loop was rewriting `dropdown.style.top` and `dropdown.style.maxHeight` on *every single frame*,
unconditionally, including while the user's finger was actively scrolling inside the open drawer.
On iOS, a touch-scroll gesture can nudge `window.innerHeight` (the dynamic toolbar reacting), and
repainting a scrolling element's own `max-height` while it's mid-scroll fights the browser's native
scroll physics — the browser has to reconcile a changing content box against an in-progress
gesture, and resolves it by resetting scroll position. That's exactly "scrolls a bit, snaps back."

**Fix:** stop recomputing the dropdown's `top`/`max-height` every frame. A new `positionDropdown()`
function computes both **once**, at the same open-transition moment `lockBody()` already fires
(tracked via `aria-expanded`), not continuously. The button's own position/size stays on the
per-frame loop — it isn't user-scrolled, so there's no physics to fight there. An
`orientationchange` listener re-runs `positionDropdown()` if the menu is still open when the device
rotates, since that's the one real case where the drawer's available height needs to change after
open. Also trimmed the reserved bottom margin from 16px to 8px, so slightly more of the viewport is
actually usable — addressing Christopher's "never uncovering so many pages people need."

Confirmed live via direct HTML fetch: `positionDropdown`, the once-per-transition call, and the
`orientationchange` listener are all present in the served response; the old always-runs
`maxHeight` line is gone. This closes out the header/mobile-nav saga (18 rounds) before starting the
broader portal/changelog redesign work.

## Round 19 (2026-09-24, urgent) — the real reason iPhone showed ZERO scroll, not just a snap-back

Christopher's precise report after round 18: on real iPhone the drawer doesn't scroll **at all**
(worse than the "tiny bit then snaps back" symptom); on Chrome desktop's mobile inspector, the tiny
snap-back from round 17 is still visible. That asymmetry — iOS completely broken, Chrome mostly
fine — is the exact signature of a bug this saga already found and fixed once before, just not
here: **a `position:fixed` descendant of an `overflow:hidden` ancestor gets clipped/scroll-blocked
on iOS Safari even though the CSS spec says it shouldn't** (round 14's root cause for the button
itself).

**What was still wrong:** `c48c922` (the button container) still had `overflow:hidden` set — needed
originally so the button's icon wouldn't overflow its own 44×44px box. But Elementor renders the
toggle button and the mobile dropdown `<nav>` as **siblings inside the same widget container**, both
descendants of `c48c922`. The dropdown itself is `position:fixed` (correctly escaping the page flow)
but was still a DOM descendant of an `overflow:hidden` ancestor — exactly the bug class round 14
already diagnosed, just never checked for the dropdown specifically, only the button.

**Fix:** removed `overflow:hidden` from `c48c922`. The button's own visual clipping is unaffected —
`.elementor-menu-toggle` (the inner button div) already carries its own explicit `overflow:hidden` +
border-radius, added defensively back in round 15, so the icon still clips cleanly to the button's
own bounds regardless of the outer container's overflow setting. Only the dropdown — which needed to
not be clipped or scroll-blocked by any ancestor at all — is affected by removing it from `c48c922`.

Confirmed live via direct HTML fetch: `c48c922`'s rule no longer contains `overflow:hidden`.
**Real-device confirmation from Christopher is still the next step** — this is a strong, structural
explanation matching the exact reported symptom precisely, but only a real iPhone test closes it out
for certain.

## Round 20 (2026-09-24, urgent) — the real root cause of every rounds 13-19 scroll issue, finally found via live DOM inspection

Christopher reported round 19's fix changed nothing on his real device. Rather than guess a fourth
time, this round inspected the **live rendered DOM directly** (via `querySelectorAll` in the actual
browser, not CSS text analysis) — and found something none of rounds 13-19 had checked.

**The real bug:** the class `elementor-nav-menu--dropdown` is used on **33 different elements**
inside the `75df872` nav-menu widget — not one. There is exactly one real top-level mobile menu
panel (`<nav class="elementor-nav-menu--dropdown elementor-nav-menu__container">`), and **32
nested per-item `<ul class="sub-menu elementor-nav-menu--dropdown">` submenus**, one for every top
menu item that has children. Every round since round 13 used a plain descendant selector
(`.elementor-element-75df872 .elementor-nav-menu--dropdown`), which — confirmed via live
`querySelectorAll` — matches **all 33**, not just the intended one.

This means every round's `position: fixed; width: 100vw; max-height: ...` was being applied to 32
unintended nested submenus as well as the real menu, and the JS sync script's
`document.querySelector()` (singular, first-match) was grabbing **whichever of the 33 happened to
be first in the DOM** — not reliably the real container. Depending on the exact browser/DOM
traversal behavior, this could mean the script was positioning/sizing the wrong element entirely,
and/or up to 32 invisible full-screen fixed overlays were stacking on top of the real menu,
intercepting touch events meant for it. This is a far better explanation for "doesn't scroll at
all on iPhone, but shows a residual artifact on desktop Chrome" than anything checked in rounds
17-19 — each of which was individually a real, correct fix for the problem it targeted, just not
this one.

**Fix:** every dropdown-related selector now requires both the `<nav>` tag and both classes
together — `nav.elementor-nav-menu__container.elementor-nav-menu--dropdown` — confirmed via live
`querySelectorAll` to match **exactly 1 element**, not 33. Added an explicit reset rule for
`ul.sub-menu.elementor-nav-menu--dropdown` (the 32 submenus) so they can never again pick up
fixed/full-width behavior from a future rule that accidentally targets the shared class name. The
JS sync script's `document.querySelector` call was updated to the same precise selector.

Confirmed live: the precise selector text is present in the served HTML, and a live
`querySelectorAll('nav.elementor-nav-menu__container.elementor-nav-menu--dropdown')` on the actual
loaded page returns exactly 1 match. **Given the history of this saga, this needs real-device
confirmation from Christopher before being called closed — no more assuming a fix "should" work
without hearing back.**

**Confirmed working on Christopher's real iPhone.** The saga is closed — 20 rounds total. Root
cause chain, for anyone reading this later: round 14 found the iOS position:fixed-clipped-by-
overflow:hidden-ancestor bug for the button; round 19 found the *same* bug still present on
`c48c922` itself; round 20 found the real reason nothing downstream of that ever fully worked —
`.elementor-nav-menu--dropdown` matched 33 elements, not 1, so no CSS or JS in rounds 13-19 was
ever reliably touching only the real menu panel. The lesson worth remembering: a CSS class name
being "the right one" in isolation doesn't mean it's unique in the live DOM — verify with
`querySelectorAll` on the actual page before trusting a selector, especially on unfamiliar
third-party markup (Elementor's, here) where the same class can legitimately mean two different
things in two different contexts.

**Next, requested by Christopher:** now that base scrolling works, the drawer should keep growing
in height as a submenu expands revealing more children, up until it nears the bottom of the
viewport — only then should scrolling take over, rather than the drawer immediately being capped
to a fixed max-height with scroll available from the start. Not yet implemented — tracked as an
open follow-up.

## Round 21 (2026-09-24) — drawer now grows with expanding submenu content, only scrolls once it nears viewport bottom

Follow-up polish now that base scrolling is confirmed working. Previously `max-height` was a hard
ceiling computed once at open time, so the drawer always had scroll available even for a short menu
with no expanded submenus. Christopher wanted it to feel more natural: grow with the content (a
submenu expanding reveals more items) up until it actually approaches the bottom of the viewport,
only becoming scrollable at that point.

**Implementation:** `applyHeightCap()` temporarily clears `max-height`, reads the drawer's real
`scrollHeight`, and only re-applies a `max-height` cap if that natural height would exceed the
available viewport space — otherwise leaves it uncapped so it simply grows. Since Elementor's own
submenu-toggle JS isn't an event this script can hook directly, a `MutationObserver` watches the
dropdown's subtree (attributes + childList) while open and re-runs the same check whenever anything
changes inside it — catching a submenu expanding after the initial open. Observer starts on open,
disconnects on close (`startObserving()`/`stopObserving()`, called from the same open/close
transition point as `lockBody()`/`unlockBody()`).

Confirmed live via direct HTML fetch: `applyHeightCap`, `MutationObserver`, and `startObserving` are
all present in the served response. **Real-device confirmation from Christopher is the next step**
for this specific refinement, same as every round in this saga.

## Round 22 (2026-09-25) — the actual cause of "doesn't grow" and "can't collapse"

Round 20's submenu reset went further than it needed to: it forced `position: absolute` **plus**
`height: auto !important; max-height: none !important` on the 32 nested per-item submenus, as a
defensive measure against them inheriting the shared class name's fixed/full-width styling. That
over-correction turned out to be the real cause of two separate problems reported this round:

1. **The drawer never grew when a submenu expanded.** Absolutely-positioned elements don't
   contribute to their parent's normal-flow height — so round 21's grow-with-content logic, which
   reads `dropdown.scrollHeight`, never saw any growth no matter how much submenu content was
   revealed. Absolute positioning also meant an expanded submenu **overlaid on top of** the later
   top-level siblings in the same list instead of pushing them down, hiding/covering them —
   Christopher's exact report ("clicking a parent higher in the list covers over/hides the lower
   remaining part of the list").
2. **A parent's children couldn't be collapsed again on a second click.** Elementor's own native
   mobile-accordion mechanism almost certainly drives its collapse via a `max-height` transition on
   the submenu — `max-height: none !important` permanently defeated that, making every expanded
   submenu look stuck open regardless of what the underlying `aria-expanded` state actually did.

**Fix:** stopped fighting Elementor's own accordion. The submenu reset now only overrides what's
strictly required to stop the shared class name from applying fixed/full-width styling —
`position: static` (real in-flow, not `absolute`) and `width`/`max-width: none`. Height, max-height,
and collapse behavior are left entirely to Elementor's own native mechanism.

## Round 23 (2026-09-25, critical) — a real infinite loop was the cause of everything else

Christopher's follow-up report after round 22: the drawer now grows on the homepage, but once it
fills the viewport, scrolling stops working entirely, the close button stops responding, and the
menu becomes completely stuck — bad enough that the only way out was opening the page in a new tab.

**Root cause, found on inspection of round 21's own script, not guessed:** the `MutationObserver`
was configured with `observer.observe(dropdown, { attributes: true, ... })` — watching attribute
changes on the dropdown **itself**, not just its descendants. But `applyHeightCap()` writes to that
same element's `style.maxHeight` — which **is itself an attribute mutation**, so it re-triggers the
very observer that called it, which calls `applyHeightCap()` again, forever. A genuine infinite loop
pegging the page's main thread solid the moment the drawer's height was ever recalculated — which
fully explains every symptom reported: scrolling freezing (the thread has no time left to service
touch events), the close button becoming unresponsive (same reason), and inconsistent behavior
between pages (a hung thread behaves differently depending on timing and what else is queued, not
because the header template actually differs between pages — confirmed via direct fetch that
`/resources/` has byte-identical up-to-date header code to the homepage).

**Fix:** `observer.disconnect()` before `applyHeightCap()` writes to the dropdown's style, then
`observer.observe(...)` again immediately after — the standard, necessary pattern whenever a
`MutationObserver` callback itself mutates the node it's watching. Also added a 120ms debounce on
the mutation-triggered recheck, so a burst of rapid mutations (Elementor's own expand animation)
only recalculates once after things settle, rather than firing on every intermediate frame.

Also added, defensively, an explicit `transition` override on `.elementor-item` limited to
background-color/color only, so nothing can animate a visible font-size change on a child item when
its parent is clicked — the mechanism behind Christopher's "children get bigger in text size"
report isn't fully confirmed yet (needs a live check once the freeze itself is verified fixed), but
this closes off the most likely accidental cause (an inherited `transition: all` picking up a native
active-state font-size difference) without waiting on that confirmation.

Confirmed live via direct HTML fetch on both the homepage and `/resources/`.
**Real-device confirmation from Christopher is the next step, as always this saga — the infinite
loop is a strong, well-evidenced explanation for every symptom, but it needs a real phone to call
it closed.**

---

Co-Authored-By: Alfred · ClaudeCodeCLI · Anthropic [Sonnet-5]
