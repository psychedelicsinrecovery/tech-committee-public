# Elementor Search Form widget — mobile overflow/truncation fix, reference

**Purpose:** a clean technical reference for this specific widget's mobile problems and fixes,
separate from `FOOTER-RECONNAISSANCE.md`'s chronological log of how we got here. Read this if you
need the current state or are adding this widget to a new page. Read `FOOTER-RECONNAISSANCE.md`
(search-widget sections) if you want the history of what went wrong along the way.

## The widget, and where it lives

Elementor Pro's "Search Form" widget (`widget_type: search.default`, `live_results: yes`). It has
no native mobile-responsive controls for border-radius, padding, or typography reachable via the
site's MCP tooling — every fix here is a CSS injection via an HTML widget, not a widget setting.
**There is no global Custom-CSS mechanism for this site.** Adding this widget to a new page means
repeating the fix below on that page; nothing propagates automatically.

As of 2026-09-23, it's live on 6 pages:

| Page | Post ID | Wrapper chain | Fix widget | Native `border-radius:55px`? | Search widget ID |
|---|---|---|---|---|---|
| Homepage | `9572` | `ed5e712` → `2203955` → `2827d53` → search | `ef693d1` | Yes | `1312455` |
| Convention 2026 | `12760` | `29005439` → `4c129c5e` → search | `0ea64c9` | No | `4f2a26d` |
| Resources | `24` | → `65296741` → search | (own widget) | Yes | `335bd7a4` |
| Public Relations | `12603` | → `2cd5a92e` → search | (own widget) | Yes | `3dc17463` |
| Search results | `12891` | → `31ce4ff3` → search | (own widget) | Yes | `be7228c` |
| Single Post Template | `9317` (renders on every blog post) | → `541fe80` → search | (own widget) | No | `6f07d1d` |

Homepage's widget is a synced Elementor Global Widget (`templateID: 13663`, "search-bar") — but
the template itself has no wrapping container, so the overflow/mobile fix still has to be injected
per-page, not once on the template.

## The three problems, and what actually caused each

**1. Horizontal overflow.** The widget's own oversized border-radius + large negative-spread
box-shadow renders wider than its container on mobile. Fix: `max-width:100%; overflow-x:hidden`
on the outer wrapper. Straightforward — the only subtlety is that the wrapper nests a different
number of levels deep depending on the page (1–3), so the rule has to be applied at every level
that page actually has, not just the innermost.

**2. The "heavy corner-radius mask."** This one is *not* caused by anything we wrote. Four of the
six pages' wrapper containers (see table above) carry a native Elementor container setting,
`border-radius: 55px` on all four corners, unrelated to the search widget's own styling. Combined
with the `overflow-x:hidden` fix from problem 1, that 55px radius clips the search field into a
heavily rounded mask, where there's no room for a 55px radius to read as a subtle rounded corner —
it reads as a mask. **Originally fixed mobile-only, assuming desktop's 55px was intentional — that
assumption was wrong.** Christopher confirmed the mask was also visible on desktop for homepage,
Resources, and Public Relations. Fix now applies unconditionally (not scoped to a mobile media
query) on all 3 affected pages. The two pages without this native setting (convention, Single Post
Template) never showed the mask on either breakpoint.

**3. Truncated placeholder text vs. an oversized submit button.** The real live markup is:
```html
<div class="e-search-input-wrapper">
  <svg class="keyboard-icon" .../>  <!-- decorative, ~30px -->
  <input placeholder="Type to start searching..." />
</div>
<button class="e-search-submit">
  <svg class="e-fas-search"/> <span class="">Search</span>  <!-- visible text, NOT screen-reader-only -->
</button>
```
An earlier attempt assumed the submit button had no text span at all — wrong; it has one, and it's
visible, not `.sr-only`. Combined with `flex-shrink:0` on the button (added to stop it collapsing
to nothing), the full "Search" label rendered at full size and ate the width the placeholder text
and the decorative keyboard icon both needed, forcing a choice between showing the icon or showing
the full placeholder string.

**Fix, current state:** hide `.e-search-submit span` directly on mobile — button becomes
icon-only (magnifying glass), freeing the width. That let the keyboard icon come back too, just
slightly smaller (20×20px instead of ~30px) rather than full size, plus a reduced input font-size
(14px) and letting the input `flex-grow` into the reclaimed space. Net result on mobile: keyboard
icon, full "Type to start searching..." placeholder, and an icon-only submit button, all visible,
nothing clipped.

**4. Placeholder text still truncated differently per page, even with the CSS above identical
everywhere.** Not a CSS problem at all — the search widget's own Elementor **responsive settings**
(not the injected CSS) differed between pages. The convention page's widget (`4f2a26d`) uniquely
carried `_flex_align_self_mobile: "stretch"` and `_flex_size_mobile: "shrink"`; none of the other 5
widget instances had these set, so their input field wasn't actually allowed to grow into the
freed space the CSS above provides — it just stayed narrow, cutting the placeholder off earlier.
Fix: applied the identical two settings to all 5 other instances (via `update-element` on the
widget's own settings, not CSS). **Verification caveat:** confirmed via settings-parity with the
one page already known to render correctly (convention page) — no tool was found to fetch this
site's aggregated/minified generated CSS to visually re-confirm pixel-for-pixel, so this is
verified-by-configuration-match, not verified-by-screenshot. Worth a real device check.

**5. The "fix" above never actually applied to the homepage — a Global Widget reference gotcha.**
Live testing after problem 4's fix showed the homepage was still the *worst* result of all 6 pages
("Type to start se...."), not fixed at all. Cause: the homepage's search widget ID, `1312455`, is a
**Global Widget reference** (`widgetType: "global"`) — a pointer to a synced template, not the
widget itself. Writing settings to `1312455` writes to the reference, which does nothing; the real
widget instance lives inside the template it points to: `templateID: 13663`, actual widget ID
**`2b63de67`**. Applied the settings there instead. **Anywhere this site uses a Global Widget, edit
the ID the `templateID` points to, not the reference ID `get-page-structure` shows you on the
page** — this cost real debugging time and is worth checking first next time, not last.

**6. Resources and Public Relations were still worse than convention/blog despite already having
correct widget settings and identical CSS.** Correlated exactly with wrapper nesting depth (3
levels for these two vs. 2 for convention vs. 1 for the Single Post Template) — small per-level
padding accumulating across 3 levels ate enough width to matter, even though nothing was
individually wrong. Fix: added `padding: 0 !important` on every wrapper level (mobile-only) in
these two pages' existing fix widgets (`a3b907b` for Resources, `63e03f2` for Public Relations).

## Reusable mobile CSS shape (adapt selectors per page's actual wrapper IDs)

```css
.elementor-element-<outer-wrapper> { border-radius: 0 !important; } /* unconditional — only if the page has the native 55px radius */
@media (max-width: 767px) {
  .elementor-element-<outer-wrapper> { max-width: 100%; overflow-x: hidden; padding: 0 !important; } /* padding:0 on every nesting level if 3+ levels deep */
  .e-search-submit span { display: none; }
  .e-search-input-wrapper svg.keyboard-icon { width: 20px; height: 20px; }
  .e-search-input-wrapper input { font-size: 14px; flex-grow: 1; }
}
```

## Adding this widget to a new page

1. Find the widget's actual wrapper container ID(s) via `get-page-structure`. **If the widget's
   `widgetType` is `"global"`, that ID is a reference, not the real widget** — follow its
   `templateID` and find the actual widget ID inside that template. Every setting/CSS change below
   goes on the real widget, never the reference.
2. Check whether that wrapper carries a native `border-radius` (via `get-element-settings`) — if
   so, add the zero-radius override, unconditionally (not just mobile — desktop can have this too).
3. Set `_flex_align_self_mobile: "stretch"` and `_flex_size_mobile: "shrink"` on the search
   widget's own settings (`update-element`) — without these, the input won't grow into whatever
   space the CSS frees up, no matter how correct the CSS is.
4. Add an HTML widget inside the innermost wrapper with the CSS shape above, IDs substituted. If
   the wrapper nests 3+ levels deep, add `padding:0 !important` at every level, not just the
   innermost — accumulated padding across levels eats width even when each level looks fine alone.
5. Verify by live fetch. Cache purge is confirmed not needed as of 2026-09-23 (Christopher's
   iPhone/desktop force-refresh both show changes immediately) — earlier rounds' "purge W3TC +
   WPX/XDN first" caveat no longer applies.

---

Co-Authored-By: Alfred · ClaudeCodeCLI · Anthropic [Sonnet-5]
