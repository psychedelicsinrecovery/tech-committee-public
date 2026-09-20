# PIR Site Archive

A Markdown archive of both PIR WordPress sites, for three purposes: (1) self-serve context for the
agent fleet without needing live WordPress access, (2) a rebuild-from-scratch source if WordPress
hosting is ever lost, (3) a content backup.

**Format decision:** plain Markdown, not `.tsx`/React — this repo isn't a frontend framework
project, so introducing a build toolchain here would add complexity with no rendering benefit. What
mattered was organization: directories mirror each site's actual URL structure, every page carries
`url`/`title`/`crawl_date` frontmatter so it's self-describing on its own, and [`INDEX.md`](https://github.com/psychedelicsinrecovery/wordpress-crawls/INDEX.md) +
`index.json` tie it all together — human-readable and machine-readable respectively.

## Structure

```
site-archive/
  README.md          — this file
  INDEX.md            — human-readable table of contents
  index.json           — machine-readable: {site, url, path, title, crawl_date}[]
  psychedelicsinrecovery-org/          — main site, one .md per page, named by URL slug
  service-psychedelicsinrecovery-org/  — service site, same pattern
```

## Status as of 2026-09-20 (sixth pass — every real page on both sites, verified by direct SQL)

**Every real, published *page* on both sites is now archived — 62 total (34 main site, 28 service
site).** What's left is individual *blog posts* (~96 of them on the main site), not pages — see
below. This pass extended the direct-SQL method from the main site (fifth pass) to the service
site, closing every remaining page-level gap on both.

**Why "every page" can be stated with confidence now, not just estimated:** two tools that look
authoritative for enumeration aren't. `emcp-tools-list-pages` silently omits some real pages (no
error, just missing — confirmed missing "Meetings" on the main site). `firecrawl_map` only sees
what's in a site's XML sitemap, and both sites have real pages excluded from theirs. The fix both
times was going straight to the database: `emcp-tools-query`, `SELECT ID, post_title, post_type,
post_name FROM {prefix}_posts WHERE post_status='publish' AND post_type='page'`. The main site's
real table prefix is `wp_eup8um_`; the service site's is the WordPress default `wp_` — both found
via `emcp-tools-list-tables` before the first query on each site returned zero rows. That query is
exhaustive by construction — nothing sitemap- or crawler-visibility-dependent can hide from it.

Main site: 48 rows returned, 34 archived as real content, 14 deliberately skipped (signup forms,
plugin infrastructure, drafts, one likely-duplicate legacy post — see `INDEX.md` for the exact
list). Service site: 27 rows, all 28 archived files accounted for (including demo placeholders and
a redirect stub, kept for completeness per Christopher's explicit "don't miss a single page," plus
one real page — Committee Emails and Setup — that's `post_type: post` rather than `page`, so it
didn't show up in the page-only query but was already correctly archived from an earlier pass).

Three things that looked like open pages resolved to *not pages* this pass, confirmed rather than
left ambiguous: `/notifications/` is a 301 redirect to `/subscribe-general/`; `/announcements/` is
a blog category (2 posts, will be picked up with the blog-post sweep below, not tracked
separately); `/events/` is a plugin custom-post-type (The Events Calendar) that Christopher
confirmed isn't actually used, so it's out of scope by design, not an oversight.

**Not yet archived:** the ~96 individual blog/series posts on the main site (grouped by theme in
`INDEX.md`), and `/meetings/in-person-meetings-2` (flagged, unverified whether it's published or a
draft). See `INDEX.md` for the full, current breakdown — it supersedes this summary if the two ever
drift.

## How to extend this

Firecrawl MCP tools were used — `firecrawl_map` first (cheap, enumerates URLs without fetching
content) to get each site's real URL inventory, then `firecrawl_scrape` per page
(`onlyMainContent: true`, `formats: ["markdown"]`) since the batch `firecrawl_crawl` endpoint was
rate-limited (429) throughout this session even though single-page `firecrawl_scrape` worked fine.
The account's actual per-minute cap turned out to be ~11 requests/min (found empirically, not
documented) — pace scrape calls accordingly, in batches of ~10 with a pause between.

To add a page: scrape it, write the result as a new `.md` file in the right site's directory (named
by URL slug, frontmatter block first), then add its row to `INDEX.md` and its entry to `index.json`.

To refresh a page that's changed on the live site: same process, overwrite the existing file, update
its `crawl_date`.
