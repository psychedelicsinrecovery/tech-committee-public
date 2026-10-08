# 🧩 PIR® Discord: apps, bots & integrations

*The architecture of PIR®'s Discord server: who does what, what each helper can see, what it keeps, and how
to hand it on. Written so a future volunteer, or their AI assistant, can pick it up and keep it running.*
*Generated from TechCom's private Discord docs; private details removed.*

## 🏗️ Three layers

| Layer | Who | Why |
|---|---|---|
| 🧱 **Infrastructure:** structure and safety | Discord's own **Onboarding**, **AutoMod** and **Server Guide**, plus **C.A.R.L.** (Carl-bot) for reaction roles | Repetitive Discord chores with no AI: roles, welcomes, spam filters |
| 🌌 **Intelligence:** members' questions | **A.L.E.X.** (our own app on the Alexandrina engine) | Cited answers and helpdesk tickets, with deterministic safety rules before any AI |
| ☄️ **Operations:** upkeep | **C.A.T.A.L.Y.S.T.** (ops app) driven by GitHub Actions, written by Christopher's agents (Alfred, Cosmos, Littlebird) | Channel descriptions, announcements, inventories. No AI runs inside Discord for this; nothing reads members' messages. |

## 🤖 Apps & bots

### 🌌 A.L.E.X. (our own)
- **Does:** `/ask` cited answers; helpdesks `/techcom` `/litcom` `/github` `/pr` `/mod`; the ask-alex chat panel; ask-alex conversation mode.
- **Sees:** only what members type into its commands and forms, plus messages in **#ask-alex and its threads** in conversation mode (the one channel with the Message Content intent).
- **Stores:** nothing about members. Tickets are GitHub Issues/Discussions tagged with a one-way hash, never a Discord ID.
- **Runs on:** Vercel (commands, buttons, web desk) and a Google Cloud e2-micro (conversation worker).
- **Code:** `alex-desk` (private) → `alex-desk-public`. Engine: `drasticstatic/alexandrina`.

### ☄️ C.A.T.A.L.Y.S.T. (PIR®'s ops app; created as "C.A.T.A.L.Y.S.T.", to be renamed)
- **Why a separate app:** A.L.E.X. is public-facing and locked down. Upkeep needs different powers (Manage Channels; access to private channels for descriptions and pins), and keeping those in a separate app means no member can ever reach them through A.L.E.X.
- **Design:** no slash commands, **no Message Content intent**, no AI inside Discord. GitHub Actions post and edit through it; every post is signed by the agent that wrote it.
- **Later, maybe:** an `/alfred` command for TechCom only, if a conversational ops helper ever earns its keep.

### 🧚 C.A.R.L. (Carl-bot) · planned
- **Use for:** reaction roles and role menus, welcome DMs, logging **moderation actions** (bans, timeouts, role changes).
- ⚠️ **Privacy:** don't enable message-edit/delete logging in private or recovery channels, because that copies members' words onto Carl-bot's servers. If enabled anywhere, the privacy policy must say so first.
- **Never for:** 🔏 private groups (anyone can click a reaction; see *Private groups* below).

### 🛡️ D.Y.N.O. (Dyno) · optional
- **Use only if** Discord's free **AutoMod** (keyword, spam, mention-spam and suspicious-link filters) proves not enough.
- **Don't double up:** one automod at a time, or members get duplicate warnings. Different command prefixes from Carl-bot (`?` vs `!`) if both are installed.

### 🗓️ S.E.S.H. (Sesh) · installed
- Meeting and event scheduling with time-zone conversion and reminders. Event details live on Sesh's servers; keep Zoom passcodes out of event descriptions.

### 🧵 H.Y.P.H.A. (Thread-Watcher) · installed
- Keeps chosen threads from auto-archiving. Needs Manage Threads in the channels it watches.

### 🎨 V.I.S.I.O.N. (Midjourney) · installed
- AI image generation. ⚠️ Prompts and images are **public** on midjourney.com unless the account uses Stealth mode. Never put names or recovery details in a prompt.

### 🚫 Not using
- **ProBot:** its onboarding strengths (image welcome cards, leveling) are paywalled or need boosts, and Discord's native Onboarding covers it.
- **MEE6:** basic features moved behind paywalls.

## 🧭 Native Discord features we rely on
- **Onboarding:** questions → roles and channels (see `onboarding-plan.md`).
- **AutoMod:** free spam, keyword and link filters. Our first line before any third-party bot.
- **Server Guide / Welcome screen:** new-member to-dos and resource pages.
- **Community:** announcement channels, rules channel, member screening.

## 🔐 Permission philosophy
1. **Least privilege:** each app gets only what its job needs, only in the channels it works in.
2. **Temporary powers:** Manage Channels is granted for a task, then removed.
3. **No message reading** except A.L.E.X. in #ask-alex (disclosed in the privacy policy).
4. **Private groups stay human:** no bot adds anyone to 🔏 BIPOC, LGBTQ+, men's or women's groups without a trusted servant's OK.
5. **People can always tell who's talking:** every bot has a persona, and every agent post says which agent wrote it.

## 🔏 Private groups: how members join
Onboarding can't safely auto-assign private-group roles (anyone could pick them). The plan is a **request desk**:
1. A member taps **Request to join** on a pinned panel (or types `/join`) and picks a group.
2. A private review thread opens, visible only to that group's chair or trusted servant and the admins. No GitHub issue, no record outside Discord.
3. The reviewer taps **Welcome** (the bot adds the role and posts a greeting in the group) or **Not right now** (a kind private reply).

## 😶 Reaction roles: what members can self-assign (proposal)
| Self-assign ✅ | Never self-assign 🚫 |
|---|---|
| 🔔 Announcement pings, 🗓️ meeting reminders, 📻 Integration Radio drops | 🔏 BIPOC / LGBTQ+ / Men's / Women's (request desk) |
| 🌍 Time-zone roles (helps S.E.S.H. and meeting planning) | 👮 Moderator, 🏛️ B.O.D., committee roles (granted by admins) |
| 🍄 Medicine-channel visibility, if the Medicines category stops being a default | 👷 person-in-service (Onboarding Q1, or an admin) |

## ⚙️ Automations (GitHub Actions)
| Workflow | Repo | Effect |
|---|---|---|
| `discord-inventory` | tech-committee | Read-only snapshot → `discord/snapshots/` |
| `set-channel-topics` | tech-committee | Applies `channel-topics.yaml` (a person's edit always wins) |
| `post-message` | tech-committee | Posts an announcement file |
| `post-message` (panel) | alex-desk | Posts A.L.E.X.'s interactive panels |
| `register-commands`, `deploy`, `reindex` | alex-desk | A.L.E.X.'s commands, Vercel deploy, knowledge index |

## 🤝 Handing this on
If you're picking this up, point your AI assistant (Claude, Cosmos, …) at this file, `SERVER-MAP.md`,
`onboarding-plan.md` and `alex-desk/SUSTAINABILITY.md`. Ask it to run `discord-inventory` first and compare the
snapshot with these docs before changing anything.
