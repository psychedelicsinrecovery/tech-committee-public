# 🗂️ PIR Site Archive — Index

**62 pages archived** (34 main site, 28 service site) — every page WordPress currently reports as
real and published on both sites, confirmed by direct database query, not by crawling or a nav
menu. See `README.md` for scope, purpose, and how to extend this.

## How this was verified complete

Two tools that look authoritative aren't: `emcp-tools-list-pages` silently omits some real pages
(no error, just missing — confirmed missing "Meetings"), and `firecrawl_map` only sees pages listed
in the site's XML sitemap. Both passes instead queried the database directly
(`emcp-tools-query`, `SELECT ... FROM {prefix}_posts WHERE post_status='publish' AND
post_type='page'`) — the real table prefix is `wp_eup8um_` on the main site and the default `wp_`
on the service site, found via `emcp-tools-list-tables`. That query is exhaustive by construction:
every row it returns is a real published page, full stop. Main site: 48 rows total (34 real-content
pages archived here, 14 intentionally skipped — see below). Service site: 27 rows (28 archived,
including one linked-but-distinct page discovered this pass; see below for the one-off count note).

**Deliberately not archived** — real rows in the SQL results, excluded on purpose:
- **Main site (14):** 5 Private Meeting Subscribe forms (Zoom-registration pages), 2 Subscribe
  General pages (the Brevo signup form), Search, Submitpost, Thank you!, 2 ThemeNcode PDF Viewer
  plugin-infrastructure pages, an In-Person-2 Elementor draft, and one legacy article
  (`12-steps From AA to Psychedelics in Recovery`, post_id 9977 — near-duplicate of the already
  -archived `12-steps.md`, needs a side-by-side check before deciding whether to archive
  separately).
- **Service site:** none skipped — every real row (including WordPress/theme demo placeholders and
  a redirect stub) is archived below for full completeness, per Christopher's explicit
  "don't miss a single page."

**Not pages at all, resolved rather than left as open questions:**
- 🔀 **`/notifications/`** is a 301 redirect (confirmed via `wp_eup8um_redirection_items`) to
  `/subscribe-general/`, the Brevo email signup form — not a standalone page.
- 📣 **`/announcements/`** is a blog category, not a page — confirmed by Christopher. It currently
  has 2 posts; they'll be picked up naturally when the ~96 remaining blog/series posts are
  archived (see below), not tracked separately here.
- ⏰ **`/events/`** is the `tribe_events` custom post type (The Events Calendar plugin), not a page
  — confirmed by Christopher as not actually used, so intentionally out of scope for this crawl.
- 🛡️ **Main-site "Privacy Policy" (post_id 3)** looked like a possible duplicate in the raw SQL
  output but isn't — its real permalink is `/safety-and-ethics/privacy-policy/`, the same page
  already archived under `safety-and-ethics/`. WordPress's flat `post_name` field just doesn't show
  the parent-page path.

---

## 🌐 psychedelicsinrecovery.org (main site) — 34 pages

### Core / identity

| | Page | File |
|---|---|---|
| 🏠 | [Home](https://www.psychedelicsinrecovery.org/) | `psychedelicsinrecovery-org/index.md` |
| 🪬 | [Vision](https://www.psychedelicsinrecovery.org/vision/) | `psychedelicsinrecovery-org/vision.md` |
| ⏳ | [History](https://www.psychedelicsinrecovery.org/history/) | `psychedelicsinrecovery-org/history.md` |
| ℹ️ | [About](https://www.psychedelicsinrecovery.org/about/) | `psychedelicsinrecovery-org/about.md` |
| 🧱 | [12 Steps](https://www.psychedelicsinrecovery.org/12-Steps/) | `psychedelicsinrecovery-org/12-steps.md` |
| 🩺 | [Safety and Ethics](https://www.psychedelicsinrecovery.org/safety-and-ethics/) | `psychedelicsinrecovery-org/safety-and-ethics.md` |
| ⛑️ | [Need for Safe Spaces](https://www.psychedelicsinrecovery.org/safety-and-ethics/need-for-safe-spaces/) | `psychedelicsinrecovery-org/safety-and-ethics/need-for-safe-spaces.md` |
| 🛡️ | [Privacy Policy](https://www.psychedelicsinrecovery.org/safety-and-ethics/privacy-policy/) | `psychedelicsinrecovery-org/safety-and-ethics/privacy-policy.md` |
| 🫂 | [Inclusivity and Diversity](https://www.psychedelicsinrecovery.org/inclusivity-and-diversity/) | `psychedelicsinrecovery-org/inclusivity-and-diversity.md` |
| 🌵 | [Our Lineages](https://www.psychedelicsinrecovery.org/our-lineages/) | `psychedelicsinrecovery-org/our-lineages.md` |
| 🗿 | [Indigenous Lineages](https://www.psychedelicsinrecovery.org/indigenous-lineages/) | `psychedelicsinrecovery-org/indigenous-lineages.md` |
| 🧩 | [Curious About Integrating Psychedelics?](https://www.psychedelicsinrecovery.org/integrating-psychedelics/) | `psychedelicsinrecovery-org/integrating-psychedelics.md` |
| ❓ | [FAQ](https://www.psychedelicsinrecovery.org/faq/) | `psychedelicsinrecovery-org/faq.md` |

### Meetings & getting involved

| | Page | File |
|---|---|---|
| 🗓️ | [Meetings *ALL*](https://www.psychedelicsinrecovery.org/meetings/) | `psychedelicsinrecovery-org/meetings.md` |
| 💻 | [Online Meetings](https://www.psychedelicsinrecovery.org/meetings/online-meetings/) | `psychedelicsinrecovery-org/meetings/online-meetings.md` |
| 👥 | [In-Person Meetings](https://www.psychedelicsinrecovery.org/meetings/in-person-meetings/) | `psychedelicsinrecovery-org/meetings/in-person-meetings.md` |
| 🔧 | [Service](https://www.psychedelicsinrecovery.org/service/) | `psychedelicsinrecovery-org/service.md` |
| 🆘 | [Crisis Resources](https://www.psychedelicsinrecovery.org/crisis-resources/) | `psychedelicsinrecovery-org/crisis-resources.md` |
| 📞 | [WhatsApp](https://www.psychedelicsinrecovery.org/whatsapp/) | `psychedelicsinrecovery-org/whatsapp.md` |
| 📡 | [Contact](https://www.psychedelicsinrecovery.org/contact/) | `psychedelicsinrecovery-org/contact.md` |
| 🩸 | [Donate (7th Tradition)](https://www.psychedelicsinrecovery.org/7th-tradition/) | `psychedelicsinrecovery-org/7th-tradition.md` |

### Resources, media & publications

| | Page | File |
|---|---|---|
| 🧰 | [Resources](https://www.psychedelicsinrecovery.org/resources/) | `psychedelicsinrecovery-org/resources.md` |
| 📎 | [Member Materials](https://www.psychedelicsinrecovery.org/member-materials/) | `psychedelicsinrecovery-org/member-materials.md` |
| 🙏🏽 | [Common Prayers](https://www.psychedelicsinrecovery.org/common-prayers/) | `psychedelicsinrecovery-org/common-prayers.md` |
| 📚 | [Library](https://www.psychedelicsinrecovery.org/library/) | `psychedelicsinrecovery-org/library.md` |
| 📖 | [Book](https://www.psychedelicsinrecovery.org/book/) | `psychedelicsinrecovery-org/book.md` |
| 🗞️ | [De Vine Newsletter](https://www.psychedelicsinrecovery.org/newsletter/) | `psychedelicsinrecovery-org/newsletter.md` |
| 🌐 | [Public Relations](https://www.psychedelicsinrecovery.org/public-relations/) | `psychedelicsinrecovery-org/public-relations.md` |
| 📃 | [Blog (listing page)](https://www.psychedelicsinrecovery.org/blog/) | `psychedelicsinrecovery-org/blog.md` |
| 🧭 | [Navigating New Challenges](https://www.psychedelicsinrecovery.org/navigating-new-challenges/) | `psychedelicsinrecovery-org/navigating-new-challenges.md` |

### Convention & announcements (individual posts)

| | Page | File |
|---|---|---|
| 🎪 | [2026 PIR® Convention](https://www.psychedelicsinrecovery.org/convention-2026/) | `psychedelicsinrecovery-org/convention-2026.md` |
| 📋 | [Convention Program update](https://www.psychedelicsinrecovery.org/convention-program/) | `psychedelicsinrecovery-org/convention-program.md` |
| 🎂 | [Happy Birthday Albert Hofmann!](https://www.psychedelicsinrecovery.org/happy-birthday-albert-hofmann/) | `psychedelicsinrecovery-org/happy-birthday-albert-hofmann.md` |
| 🚀 | [Website Re-Launch](https://www.psychedelicsinrecovery.org/website-re-launch/) | `psychedelicsinrecovery-org/website-re-launch.md` |

---

## 🛠️ service.psychedelicsinrecovery.org (service site) — 28 pages

### Core

| | Page | File |
|---|---|---|
| 🔧 | [Service Home](https://service.psychedelicsinrecovery.org/) | `service-psychedelicsinrecovery-org/index.md` |
| 📡 | [Contact](https://service.psychedelicsinrecovery.org/contact/) | `service-psychedelicsinrecovery-org/contact.md` |
| 🛡️ | [Privacy Policy](https://service.psychedelicsinrecovery.org/privacy-policy/) | `service-psychedelicsinrecovery-org/privacy-policy.md` |
| 🗓️ | [Calendar](https://service.psychedelicsinrecovery.org/calendar-service/) | `service-psychedelicsinrecovery-org/calendar-service.md` |

### Governance

| | Page | File |
|---|---|---|
| 🏛 | [Board of Directors](https://service.psychedelicsinrecovery.org/board-of-directors/) | `service-psychedelicsinrecovery-org/board-of-directors.md` |
| 🗳️ | [501 Board](https://service.psychedelicsinrecovery.org/board/) | `service-psychedelicsinrecovery-org/board.md` |
| 💰 | [Finance](https://service.psychedelicsinrecovery.org/finance/) | `service-psychedelicsinrecovery-org/finance.md` |

### Committees

| | Page | File |
|---|---|---|
| 🧑‍🤝‍🧑 | [Committees (index)](https://service.psychedelicsinrecovery.org/committees/) | `service-psychedelicsinrecovery-org/committees.md` |
| 🧱 | [12 Step Committee](https://service.psychedelicsinrecovery.org/12-step-committee/) | `service-psychedelicsinrecovery-org/12-step-committee.md` |
| 📖 | [Book Committee](https://service.psychedelicsinrecovery.org/book-committee/) | `service-psychedelicsinrecovery-org/book-committee.md` |
| 🚧 | [Convention Committee](https://service.psychedelicsinrecovery.org/convention-committee/) | `service-psychedelicsinrecovery-org/convention-committee.md` |
| 👷🏽 | [GSR Committee](https://service.psychedelicsinrecovery.org/gsr-committee/) | `service-psychedelicsinrecovery-org/gsr-committee.md` |
| ⚖️ | [GSR](https://service.psychedelicsinrecovery.org/gsr/) | `service-psychedelicsinrecovery-org/gsr.md` |
| 📜 | [Literature Committee](https://service.psychedelicsinrecovery.org/literature-committee/) | `service-psychedelicsinrecovery-org/literature-committee.md` |
| 📗 | [Literature](https://service.psychedelicsinrecovery.org/literature/) | `service-psychedelicsinrecovery-org/literature.md` |
| 🤝🏽 | [PR Committee](https://service.psychedelicsinrecovery.org/pr-committee/) | `service-psychedelicsinrecovery-org/pr-committee.md` |
| 👨🏽‍💻 | [Tech Committee](https://service.psychedelicsinrecovery.org/tech-committee/) | `service-psychedelicsinrecovery-org/tech-committee.md` |
| ⚙️ | [Tech](https://service.psychedelicsinrecovery.org/tech/) | `service-psychedelicsinrecovery-org/tech.md` |
| ⚙️ | [Intergroup](https://service.psychedelicsinrecovery.org/intergroup/) | `service-psychedelicsinrecovery-org/intergroup.md` |
| 📚 | [ForaPIR](https://service.psychedelicsinrecovery.org/forapir/) | `service-psychedelicsinrecovery-org/forapir.md` |
| 🛠️ | [Service Structure Working Group](https://service.psychedelicsinrecovery.org/service-structure-working-group/) | `service-psychedelicsinrecovery-org/service-structure-working-group.md` |
| 👏🏽 | [Former Committees](https://service.psychedelicsinrecovery.org/former-committees/) | `service-psychedelicsinrecovery-org/former-committees.md` |
| ✉️ | [Committee Emails and Setup](https://service.psychedelicsinrecovery.org/committee-email/) | `service-psychedelicsinrecovery-org/committee-email.md` |

### Placeholders, drafts & redirects (archived for completeness, not real PIR copy)

| | Page | File |
|---|---|---|
| 🧪 | [About (demo placeholder)](https://service.psychedelicsinrecovery.org/about-2/) | `service-psychedelicsinrecovery-org/about-2.md` |
| 🧪 | [Services (demo placeholder)](https://service.psychedelicsinrecovery.org/services/) | `service-psychedelicsinrecovery-org/services.md` |
| 🧪 | [Sample Page (WP default)](https://service.psychedelicsinrecovery.org/sample-page/) | `service-psychedelicsinrecovery-org/sample-page.md` |
| 🧪 | [Demo Page - Timeline](https://service.psychedelicsinrecovery.org/demo-page/) | `service-psychedelicsinrecovery-org/demo-page.md` |
| 🔀 | [service-calendar (redirect stub)](https://service.psychedelicsinrecovery.org/service-calendar/) | `service-psychedelicsinrecovery-org/service-calendar-redirect.md` |

---

## 📝 Not yet archived — individual blog & series posts

The main site's `/blog/` listing page (archived above) points at roughly **96 individual posts**,
not yet pulled in individually. They cluster cleanly by theme for whoever picks this up next:

- **🌀 "10 Different Models of Psychedelic Integration" series** — 10 numbered parts plus its
  series-intro post.
- **🦸 "Hero's Journey" series** — ~13 stage posts mapping the psychedelic recovery experience to
  Campbell's hero's-journey structure.
- **📜 AA/Bill Wilson psychedelic history** — ~15 posts on Bill Wilson's LSD experiments, the
  Oxford Group, early AA meetings, Ibogaine, MK-Ultra-adjacent history.
- **💬 Personal stories / essays** — ~15 first-person posts.
- **📚 Book reviews and miscellaneous essays** — the remainder (~30-40 posts).
- **📣 The 2 posts currently under `/announcements/`** will be picked up as part of this same sweep
  — that category isn't tracked as its own line item (see above).

Also not yet archived: `/meetings/in-person-meetings-2` on the main site (a real content page, not
a blog post — confirm via the same SQL-verification method before archiving, since it wasn't in
this pass's page list and may be an Elementor draft rather than published).

Re-run the same SQL query used in this pass (`SELECT ID, post_title, post_type, post_name FROM
wp_eup8um_posts WHERE post_status='publish' AND post_type='post' ORDER BY ID`) to get the
authoritative full post list rather than re-attempting a `firecrawl_map` crawl.
