# LinkedIn Profile Build — Santhosh R

Copy-paste ready. Built from `skill/resume/santhu.pdf` and the 9-slide framework in this folder
(`1.jpg` – `9.jpg`), adapted for someone who **already has real production experience**.

> **The framework's bias, and how yours differs.** The carousel is written for a self-taught dev
> with no job — that's why it says "freelance, open-source and content ARE experience." You have
> 2.8 years at Delta Electronics and a platform shipping on customer hardware. Your problem is the
> opposite one: you have the receipts and your profile doesn't show them. Every section below is
> tuned for *extraction*, not invention.

---

## Where the profile stands right now

Checked against a live screenshot (Sep 2026):

| | Item | State |
|---|------|-------|
| ✅ | Banner | Uploaded |
| ✅ | Profile photo | Uploaded — clear face, neutral background, reads professional |
| ✅ | Location | Bengaluru, Karnataka, India |
| 🔴 | **Headline** | **"Student at Siddaganga Institute Of Technology"** — you graduated Jun 2024 and have been at Delta since Jan 2024 |
| 🔴 | **Experience** | **Empty.** This is *why* the headline says Student — LinkedIn falls back to education when there are no positions |
| 🔴 | **Connections** | **22.** Below ~500 you are effectively invisible to recruiter search and your posts reach nobody |
| ⬜ | About | Not added |
| ⬜ | Skills | Not added |
| ⬜ | Projects | Not added |
| ⬜ | Featured | Not added |
| ⬜ | Custom URL | Still the default hashed one |
| ⬜ | Verification badge | Prompted on your profile, not claimed |
| ⬜ | Open to work | Not configured |

## Do this in order

The order below is not arbitrary. **Experience comes first because adding it fixes the headline by
itself** — the moment a current position exists, LinkedIn stops calling you a student. Everything
after that depends on Experience existing, because skills attach to positions and projects
associate with them.

| # | Task | Time | Why it's in this position |
|---|------|------|---------------------------|
| 1 | **Experience — 3 Delta positions** | 25 min | Kills "Student at SIT" automatically; everything else attaches to it |
| 2 | **Headline** — override the auto one | 10 min | The only line visible in recruiter search results |
| 3 | **Connections → 500+** | start today, ~3 weeks | Slowest item on the list. Starts working the day you start |
| 4 | Education — confirm it's correct | 5 min | Currently your only entry; it needs the degree and CGPA |
| 5 | About | 20 min | The click-decider once they land |
| 6 | Skills — audit, pin 3, link to experience | 15 min | Primary input to LinkedIn's recruiter search filter |
| 7 | Projects | 20 min | Where profile views become interviews |
| 8 | Featured | 10 min | Proof, above the fold |
| 9 | Custom URL + verification badge | 10 min | Free credibility; permanent |
| 10 | Open to work → recruiters only | 5 min | Puts you in the Recruiter search pool without telling Delta |
| 11 | Recommendations — send 3 requests | 15 min | Depends on other people, so start it early |
| 12 | Activity | ongoing | The background check they run before reaching out |

---

## 0. Two settings to fix first

**Your public URL.** It is currently:

```
linkedin.com/in/santhosh-r-64259b3b0
```

That trailing hash is LinkedIn's default and it looks unclaimed. Change it under
*Edit public profile & URL* to one of:

```
linkedin.com/in/santhoshr-dev
linkedin.com/in/santhosh-r-fullstack
linkedin.com/in/santhoshrb
```

Then update it everywhere — your resume header currently prints the hashed version.

**"Open to work" — recruiters only.** You're employed. Turn on *Open to work* but set visibility to
**Recruiters only**, not the green photo frame. It still surfaces you in LinkedIn Recruiter searches
without announcing anything to your current team. Set:

- Job titles: `Software Engineer`, `Full Stack Engineer`, `Backend Engineer`, `SDE 2`
- Locations: `Bengaluru`, `Remote (India)`
- Start date: *Immediately* (this is a filter recruiters actually use)

---

## 1. Banner + Headline

> **Rule:** the banner says what you build + for whom + where to see it. Your headline is not
> "Software Engineer at Delta."
>
> **Formula:** `[Role] ([stack]) | Building [what] | [proof or audience]`
>
> **Goal:** specific attracts, vague repels.

### Your banner

`banner.png` in this folder is rendered at LinkedIn's exact 1584×396, with the bottom-left kept
clear so your profile photo doesn't cover the text. Edit `banner.html` and re-render if you want
to change the wording (command at the bottom of this file).

### Your headline — pick one

LinkedIn's limit is 220 characters. All three are under it.

**Option A — recommended. Leads with stack, proves with shipped work.**
```
Full Stack Engineer (React · TypeScript · Node.js) | Shipped the UI, API and Linux deployment for Delta's Gen-3 Power Distribution Units | Building Habex — habex.santhu.site
```

**Option B — keyword-dense, best for recruiter search matching.**
```
Full Stack Engineer | React · TypeScript · Node.js · PostgreSQL · REST APIs | Real-time systems & embedded Linux at Delta Electronics | Open to SDE-2 roles | santhu.site
```

**Option C — narrative, best if you want to stand out to humans over filters.**
```
I build full-stack systems that ship on real hardware — React operator UIs, Node APIs, and the ARM Linux underneath | Full Stack Engineer @ Delta Electronics | habex.santhu.site
```

**Why not your current framing.** "Senior Software Engineer at Delta Electronics India" tells a
recruiter your employer and nothing about what you can do. Worse, at 2.8 years "Senior" reads as
title inflation to a product-company recruiter. Option A defuses that by leading with the stack and
letting the shipped work carry the weight — the title still appears (correctly) in your Experience
section, where it's verifiable and in context.

---

## 2. About

> **Rule:** the goal is not to introduce yourself, it's to sell the click. Hook on the first line.
> One project, one stack, one ask.

The carousel's template is `I help [who] [stop painful thing] and [get desirable result]` — that's
creator voice, written for someone building an audience. You're an employed engineer talking to
recruiters and hiring managers, so the hook changes: lead with **what you build and the proof**,
not who you help.

**Paste this (LinkedIn allows 2,600 characters; this is ~1,900):**

```
I build software that runs on machines people depend on.

For the past two years at Delta Electronics I've been the sole developer on the Gen-3 Power
Distribution Unit platform — three applications shipping on customer hardware: a React touch-panel
UI that operators use on the device itself, a browser-based admin console, and the TypeScript/Node
backend underneath both.

Owning it end to end meant making the calls myself:

• JWT auth with short-lived access tokens and httpOnly refresh cookies, plus role-based permissions
  that re-resolve on every refresh — so when an admin revokes access, live sessions lose it
  immediately instead of at token expiry.
• A real-time energy sampler that runs independently of the UI, samples power every 5 seconds into
  a rolling one-hour window and mirrors to disk — so data collection never pauses on navigation and
  survives a backend restart.
• A self-extracting Bash installer that ships the ARM64 binary and both web builds, provisions
  systemd units, an nginx reverse proxy and udev USB automount — with versioned OTA updates and
  rollback backups.

Outside work I ship my own things. Habex (habex.santhu.site) is a live expense, habit and investment
tracker on Supabase Postgres — 12 versioned migrations and three edge functions keeping privileged
operations off the client. Jourvia is a React Native trip planner with real-time location sharing
between companions and an offline-first cache over Zustand.

Stack: TypeScript · React · Node/Express · Python/FastAPI · PostgreSQL · Socket.IO · Linux
Going deeper right now on: system design, Redis, and queue-backed architectures.

Open to Full Stack and Backend engineering roles in Bengaluru.
Joint project — code on my collaborator's account: github.com/Vrindakishoree/Fraudshield · Site: santhu.site
```

**What makes this work:** the first line is a hook, not a job title. Every claim is specific enough
that it could only be written by the person who built it — "permissions re-resolve on every refresh"
is not something you can fake. And it closes with one clear ask and two links.

---

## 3. Skills

> **Rule:** it's a primary data source for LinkedIn's recruiter search filters. Delete vague skills.
> Pin the 3 that match your next job. Link each to a project.

**Pin these 3 at the top** (LinkedIn shows only these on the profile card):

```
1. React
2. TypeScript
3. Node.js
```

**Then add, in this order** — these are the terms recruiters actually filter on:

```
JavaScript · REST APIs · PostgreSQL · Express.js · System Design · Redis · Docker
Socket.IO · WebSockets · Python · FastAPI · SQL · Git · Linux · React Native
Redux · Tailwind CSS · Jest · REST API Design · Authentication · Microservices
```

**Delete these from your profile** (they're on your resume and they lower the bar you appear to
clear):

```
✗ Tkinter          — hobby-grade, and it anchors you as a scripter
✗ Selenium         — reads as QA/automation, not the roles you want
✗ MySQL            — redundant once PostgreSQL is listed
✗ Microsoft Office / MS Word / any generic tool
✗ Communication / Teamwork / Leadership / Problem Solving
✗ C, C++           — keep ONLY if you want embedded roles in your pipeline
```

**Then link skills to evidence.** On each of React, TypeScript and Node.js, use *Add to skill* and
attach the Delta experience entry and the Habex project. LinkedIn shows "3 experiences" under the
skill — that's what turns a self-reported list into a claim with receipts.

**Endorsements:** ask five colleagues at Delta to endorse exactly React, TypeScript and Node.js.
Don't chase endorsements on everything — concentration beats spread.

---

## 4. Experience

> **Rule:** the receipts that prove you can do the job. Every bullet: `Built [what] with [stack] →
> [measurable result]`. Put recruiter keywords in the titles. Clarity beats cleverness.

### First: merge your three Delta roles into ONE company entry

This is the single highest-value structural fix on your profile. Right now your three roles read as
three separate jobs. Added as **positions under one Delta Electronics entry**, LinkedIn renders them
as a stacked timeline showing *Intern → Graduate Engineer Trainee → Senior Software Engineer* over
2 years 8 months.

That single change tells a recruiter, in one glance: **this company promoted him twice in under
three years.** Nothing you write in a bullet is as persuasive as that shape.

To do it: add the position to the *existing* Delta Electronics company entry rather than creating a
new experience block. LinkedIn groups them automatically when the company matches.

### Position 1 — current

```
Title:    Senior Software Engineer — Full Stack
Company:  Delta Electronics India
Dates:    Jun 2025 – Present
Location: Bengaluru, Karnataka, India · On-site
```
```
Sole developer of the PDU Gen-3 platform — an embedded React touch-panel UI, a browser-based
admin configurator, and a Node.js/Express microservice — shipping on customer-facing Power
Distribution Units.

• Built the backend from scratch in TypeScript: 17 route modules covering measurement, alarms,
  calibration, event and data logs, configuration, user management and system control, documented
  with Swagger/OpenAPI.
• Implemented JWT authentication (short-lived access tokens + httpOnly refresh cookies) with
  role-based access control where permissions re-resolve on every refresh, so an admin's revocation
  takes effect immediately on live sessions.
• Built a real-time energy sampler decoupled from the UI — samples every 5s into a rolling one-hour
  window and mirrors to disk, so collection never pauses on navigation and survives a restart.
• Replaced per-page config fetching with a boot-time bootstrap and API self-test, surfacing a live
  "backend disconnected" indicator by reusing the existing Socket.IO signal — no extra heartbeat
  endpoint needed.
• Automated device deployment with a self-extracting Bash installer that ships the packaged ARM64
  Linux binary and both web builds, provisions systemd units and an nginx reverse proxy, and
  configures udev-driven USB automount — with versioned OTA updates and rollback backups.
• Delivered a Vite + React + Tailwind admin console with permission-driven navigation.

Stack: TypeScript · React · Node.js · Express · Socket.IO · SQLite · Linux · systemd · nginx
```

### Position 2

```
Title:    Graduate Engineer Trainee — Frontend
Company:  Delta Electronics India
Dates:    Aug 2024 – May 2025
Location: Bengaluru, Karnataka, India · On-site
```
```
• Built the frontend of a customer-facing PDU touch panel in React and TypeScript, overhauling a
  legacy layout into a responsive interface.
• Architected application state with Redux and built a reusable component library from Figma
  designs, working within hardware constraints: fixed resolution, touch-only input, and a custom
  on-screen keyboard.

Stack: React · TypeScript · Redux · Figma
```

### Position 3

```
Title:    Software Engineer Intern
Company:  Delta Electronics India
Dates:    Jan 2024 – Jul 2024
Location: Bengaluru, Karnataka, India · On-site
```
```
• Worked with CAN, Modbus and I2C protocols in industrial hardware.
• Gained hands-on exposure to Linux CLI tooling and GitLab; reworked PDU UI/UX mockups in Figma.
```

### The gap to close before you apply

Re-read those bullets. **Not one number appears in them** — because there are none in your resume
either. Go find them and add them in both places:

- How many PDU units are in the field?
- How many endpoints across the 17 route modules?
- p95 response time on the device?
- How many roles and permissions in the RBAC matrix?
- Panel resolution, memory ceiling, installer runtime?
- Habex: users, largest table row count, requests/day?

Where a count is genuinely small, use a **hard constraint** instead. *"Held p95 under 120 ms on a
512 MB ARM board"* beats any user number, because it proves you were measuring.

---

## 5. Projects

> **Rule:** treat it like a product page, not a file list. Title = what it does + for whom, never
> "Project 1". One line of stack, one line of result. Live demo + GitHub on every single one.
> Sell the outcome you shipped, not the tutorial you followed.

### Project 1 — lead with this one

```
Name:        Habex — expense, habit & investment tracker (live, self-hosted)
Dates:       Jan 2025 – Present  ·  ☑ Currently working on this
Link:        https://habex.santhu.site
Contributors: —
```
```
Live personal-finance app on my own domain: recurring expense rules with monthly views, habit
tracking with a 21-day challenge, shared investment groups, and an admin dashboard.

Backed by Supabase Postgres with 12 versioned SQL migrations and three edge functions (PIN login,
admin user management, activity audit) that keep privileged operations off the client.

React · TypeScript · Supabase (Postgres) · shadcn/ui · Tailwind
Live: habex.santhu.site · Code: github.com/ZenblockLabs/exphabbits
```

### Project 2

```
Name:  Jourvia — travel companion app for groups planning trips together
Dates: 2025
```
```
Cross-platform trip planner built on React Native CLI (no Expo): trip CRUD, Google Maps route
distance, weather, friends via share IDs, and trip sharing with owner/editor permission tiers.

Real-time live-location sharing between trip companions over Firestore, plus an offline-first
caching layer over Zustand so trips stay readable and editable without a connection. Authorization
moved out of client JavaScript and into a committed Firestore security-rules baseline (pending review before deploy); utilities, offline sync and UI
flows covered by a Jest suite.

React Native 0.85 (New Architecture) · TypeScript · Firebase · Zustand · Jest
```

### Project 3

```
Name:  FraudShield — accounts-payable fraud detection API
Dates: 2025
```
```
REST service that scores supplier invoices for fraud risk, with reviewer and admin workflows and
auto-generated OpenAPI docs.

Configurable rules engine — amount thresholds, per-vendor invoice velocity over a rolling window,
and repeated round-amount detection — feeding a scoring service, paired with a gradient-boosted
model trained on a labelled fraud dataset.

Python · FastAPI · SQLAlchemy · Pydantic · scikit-learn
Code: github.com/Santhu32144
```

### Leave off

**Time Tracker** (Tkinter + Selenium). It reads as scripting rather than engineering, and on a
profile with three strong projects it's the one that sets your floor. Same reasoning as removing it
from the resume.

### Before you paste any of this

Every project needs a **live link or a GitHub link, and a README with screenshots.** The carousel is
right that this is the checkout counter — a recruiter who clicks through to an empty repo with no
README leaves with a worse impression than if you'd never listed it. Habex is in good shape. Check
Jourvia's README before you link it; your own notes flag it as stale and understating the app.

---

## 6. Featured

> **Rule:** your trophy cabinet. 2–3 tangible wins. Custom readable thumbnails, not auto-generated
> grey boxes. Shipped work over certificates. Swap monthly.

Add exactly these three, in this order:

| Slot | Type | What | Thumbnail to upload |
|------|------|------|---------------------|
| 1 | Link | `santhu.site` — "Santo — Full-Stack Developer Portfolio" | `og-santhu.png` (blue) |
| 2 | Link | `habex.santhu.site` — "Habex — live expense, habit & investment tracker" | `og-habex.png` (emerald) |
| 3 | Link | `github.com/Santhu32144` — "GitHub — Jourvia, Habex and side projects" | `og-github.png` (violet) |

All three cards are in this folder, 1200×630, built as a matched set — same layout and type
scale, three different accent colours so they read as designed rather than pasted. Upload each via
the **pencil icon on the Thumbnail box** in LinkedIn's *Add a link* dialog. Sources are
`og-*.html`; re-render with the Chrome command at the bottom of this file, swapping the window size
to `1200,630`.

Then **pin your repos on GitHub** so slot 3 lands somewhere curated: Jourvia (Habex first has to be transferred from ZenblockLabs/exphabbits; FraudShield sits on a collaborator's account),
and your `skill/` curricula if you make that public.

**Replace slot 3 with a write-up once you have one.** A post like *"How I cut Habex's p95 from
900 ms to 120 ms with one Redis key"* — which is exactly what Phase 2 of your 35 LPA roadmap
produces — outperforms a GitHub link, because it shows judgment rather than just output.

**Do not feature:** the CodeChef SQL or HackerRank certificates. Same reasoning as the resume — they
lower the bar you appear to clear.

---

## 7. Recommendations

> **Rule:** the 5-star reviews that close the interview. "Great to work with" is useless fluff —
> guide the writer by naming the exact problem you solved for them. Two detailed wins beat ten
> vague ones.

Ask **three people**, and start today — this is the slowest item on the list because it depends on
someone else.

1. **Your reporting manager at Delta** — the PDU Gen-3 platform.
2. **A senior engineer or the hardware/firmware lead** you worked alongside — the CAN/Modbus and
   deployment work.
3. **Your Hack-o-Smart teammate or a college project guide** — how you work under time pressure.

**Send this, filled in** (LinkedIn's default request message is blank and most people freeze when
asked cold — giving them the specifics is doing them a favour, not being pushy):

```
Hi [Name],

I'm tidying up my LinkedIn and I'd really value a short recommendation from you if you have
ten minutes.

To make it easy, here's what I'd suggest mentioning — feel free to ignore or reword any of it:

• I was the sole developer on the PDU Gen-3 platform — the touch-panel UI, the admin console
  and the Node backend — and shipped it onto customer units.
• The piece I'm proudest of is the deployment automation: the self-extracting installer with
  systemd, nginx and OTA rollback, which took device setup from a manual process to one command.
• If it fits, a line on how I handle owning something end to end without much supervision.

Anything specific you remember is worth more than a general compliment. Thanks either way —
happy to write you one too.

Santhosh
```

**Offer to write theirs first.** Reciprocity does most of the work here, and a recommendation you
*give* also appears on your profile under the "Given" tab.

---

## 7.5 Connections — the 22 problem

Not in the carousel, because it assumes you already have a network. You don't, and it quietly
breaks everything else on this page.

**Why 22 connections is the most expensive number on your profile:**

- LinkedIn Recruiter ranks results partly by network proximity. At 22 connections you are outside
  almost everyone's 2nd and 3rd degree, so you simply don't appear in searches you'd otherwise match.
- A profile with 22 connections reads as abandoned. Recruiters treat it as "probably won't reply."
- Your posts reach roughly nobody, which makes the whole Activity strategy pointless until this is fixed.
- Referrals — the single highest-leverage channel in your 35 LPA plan — require a network to ask.

**Target: 500+.** LinkedIn stops displaying an exact count past 500 and just shows "500+", which is
the signal you want. Getting there takes about three weeks at a safe pace.

**Pace:** 15–20 invitations a day, no more. LinkedIn caps invites at roughly 100–200 per week and
restricts accounts that blast past it. There is no rush that's worth a restricted account.

**Who to add, in this order:**

| Priority | Who | How to find them | Why first |
|---|---|---|---|
| 1 | Delta Electronics colleagues | Company page → People | Highest accept rate; they know you |
| 2 | SIT alumni working at product companies | **SIT school page → Alumni tab → filter "Where they work"** | This is your referral pipeline. Filter by Razorpay, CRED, Zepto, Swiggy, Groww, Juspay, PhonePe |
| 3 | Your 2020–24 EIE batchmates | Alumni tab, filtered by years | Same cohort, near-certain accepts |
| 4 | Engineers at your target companies | Search by company + "Software Engineer" + Bengaluru | Warm them up months before you need anything |
| 5 | Bengaluru tech recruiters | Search "Technical Recruiter" + Bengaluru | They accept nearly everyone, and they search their own network first |

**The SIT alumni tool is the highest-leverage thing here.** Go to the Siddaganga Institute of
Technology page, open the Alumni tab, and filter by *Where they work*. Every SIT graduate at
Razorpay or Groww is a warm referral you're one message away from. Connect with them now, while you
want nothing — a referral request eight months into a real connection lands completely differently
from a cold one.

**Always add a note.** Accept rates roughly double. Keep it to two lines:

```
Hi [Name] — fellow SIT grad (EIE, 2024). I'm a full stack engineer at Delta Electronics
working on React/Node, and I'm following what your team is building at [Company].
Would be glad to connect.
```

For Delta colleagues:

```
Hi [Name] — we haven't worked together directly, but I'm on the PDU Gen-3 platform here
at Delta. Connecting up with more of the team on here.
```

**Do not** send the blank default invite. "I'd like to add you to my professional network" is the
request people decline without reading.

**One rule while you do this:** your profile has to be finished *before* you start sending. Every
invite drives a profile view, and right now a visitor sees "Student at Siddaganga Institute of
Technology" with no experience listed. Finish steps 1 and 2 first, then start connecting.

---

## 8. Activity

> **Rule:** the daily background check recruiters run. Share what you shipped and what broke, not
> motivational quotes. Real screenshots, not stock photos. Two thoughtful comments a day beats one
> sporadic post. Delete old "Excited to announce I completed a course" posts.

This is the section that compounds, and it's the one directly tied to the referral strategy in your
35 LPA roadmap. Eight months of visible building is what makes a referral ask feel natural instead
of cold.

**Your cadence — sustainable, not heroic:**

| Frequency | Action |
|-----------|--------|
| Daily | 2 substantive comments on posts by engineers at your target companies |
| Weekly | 1 post — what you shipped or what broke that week |
| Monthly | Swap one Featured item; audit and delete anything stale |

**Ten posts you can already write today**, drawn from work you've actually done:

1. Why I re-resolve permissions on every token refresh instead of trusting the JWT claims
2. The energy sampler that had to survive a backend restart — and why I mirrored to disk
3. Building a React UI for a touch panel with no keyboard, no mouse, and a fixed resolution
4. Shipping a Node app to ARM64 Linux: `pkg`, systemd, nginx and a self-extracting installer
5. OTA updates with rollback on a device you can't SSH into
6. What CAN and Modbus taught me about designing APIs
7. Why I moved Habex's authorization out of client JavaScript and into Postgres RLS
8. Offline-first in React Native: what I got wrong in Jourvia the first time
9. 12 migrations in, here's what I'd change about how I started Habex's schema
10. Reusing an existing Socket.IO signal instead of adding a heartbeat endpoint

Each of those is a post *only you can write*, which is the entire point. Nobody needs another
"10 React tips" carousel.

**Before you start posting, audit backwards.** Delete or hide old "Excited to announce I've
completed [course]" and certificate posts. They're the exact signal the framework warns about, and
they're the first thing a recruiter scrolls past on the way to your real work.

---

## 9. The final pass — roast it

> **Rule:** no redesign, no new projects. Just clearer positioning.

Once everything above is live, screenshot your own profile and run this on it:

```
Roast my LinkedIn profile (screenshot attached). Be brutal about the headline, About,
skills, projects, experience bullets and activity. I'm a full stack engineer with 2.8 years
at Delta Electronics targeting SDE-2 roles at Indian product companies (Razorpay, CRED,
Zepto, Swiggy, Groww, Juspay). Then give me a specific fix for each.
```

Send it to me and I'll do it properly against what we've built here.

---

## Quick checklist

```
☑ Banner uploaded (banner.png)
☑ Profile photo uploaded
☐ Three Delta roles added under ONE company entry   ← do this first
☐ Headline replaced (Option A)
☐ Connections: 22 → 500+ (15–20/day, always with a note)
☐ Education entry completed (B.E. EIE, CGPA 8.94, Dec 2020 – Jun 2024)
☐ Verification badge claimed
☐ Custom URL claimed, resume header updated to match
☐ Open to work → Recruiters only, with titles and Bengaluru set
☐ Experience bullets pasted, stack line added to each
☐ Real numbers found and added to the top 5 bullets
☐ About pasted
☐ Skills audited — 6 deleted, React/TypeScript/Node.js pinned
☐ Skills linked to Delta experience + Habex project
☐ 3 projects added, each with a working link
☐ Jourvia README fixed before linking it
☐ GitHub repos pinned
☐ 3 Featured items added with custom thumbnails
☐ 3 recommendation requests sent
☐ Old certificate/course posts deleted
☐ First post written
```

---

## Re-rendering the banner

Edit `banner.html`, then:

```powershell
& "C:\Program Files\Google\Chrome\Application\chrome.exe" --headless=new --disable-gpu `
  --no-sandbox --hide-scrollbars --window-size=1584,396 `
  --screenshot="D:\Santhosh.R\skill\linkdin\banner.png" `
  "file:///D:/Santhosh.R/skill/linkdin/banner.html"
```

Same headless-Chrome pipeline as `skill/resume/`. LinkedIn banners are 1584 × 396; the layout keeps
the bottom-left clear so your profile photo doesn't cover any text.
