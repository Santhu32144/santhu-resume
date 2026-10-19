# 🚀 Skill-Up Hub — Road to Senior / 50 LPA

> **North star:** reach **~50 LPA at a product company** (Zepto · Swiggy · CRED · Razorpay · Groww, or FAANG) in ~3 years.
> **Starting point:** ~12–18 LPA, React + TypeScript + Redux in production (Delta Electronics, GET), live projects [santhu.site](https://santhu.site) + [habex.santhu.site](https://habex.santhu.site), EIE/hardware background.
> **Source of truth for the "why":** [things.md](things.md) (the full honest gap analysis + 3-year roadmap). **This README is the live, executable version.**

---

## The gap, in one line

You already have the *hard-to-fake* thing — **real production experience + shipped projects**. What's missing is **learnable**: DSA, system design, backend depth, CS fundamentals, and a stronger profile. This folder closes each gap as a **chapter-wise track** you can work through.

```
        WHAT PRODUCT COMPANIES GATE ON
        ┌───────────────┬───────────────┐
        │  DSA           │  System Design │   ← the two interview gates
        ├───────────────┴───────────────┤
        │  Backend depth (Node/PG/Redis) │   ← biggest CV/profile gap
        ├────────────────────────────────┤
        │  CS fundamentals · Adv TS       │   ← asked even to frontend folks
        ├────────────────────────────────┤
        │  Profile: GitHub · LinkedIn · CV │  ← how recruiters find + filter you
        └────────────────────────────────┘
```

---

## Your tracks (each = a folder in `skill/`)

| # | Track | Folder | Status | Closes which gap | Interview leverage |
|---|-------|--------|--------|------------------|--------------------|
| 1 | **System Design** | [`system-design/`](system-design/00-README.md) | ✅ **Built** (27 chapters) | System design | 🔴 High — a hard gate |
| 2 | **DSA Patterns + LeetCode plan** | [`dsa/`](dsa/00-README.md) | ✅ **Built** (23 chapters, TypeScript) | DSA | 🔴 High — the other hard gate |
| 3 | **Backend Depth** (Node · Express · PostgreSQL · Redis · queues · Docker) | [`backend/`](backend/00-README.md) | ✅ **Built** (24 chapters, TypeScript) | Backend depth | 🟠 Medium-high — fixes CV |
| 4 | **Make Habex Full-Stack** (hands-on build) | folded into [`backend/22`](backend/22-capstone-habex-backend.md) | ✅ **Covered** by the backend capstone | Backend + portfolio | 🟠 Turns Habex into your story |
| 5 | **CS Fundamentals** (OS · networking · DB internals) | `cs-fundamentals/` | ⬜ Planned | CS fundamentals | 🟡 Medium |
| 6 | **Advanced TypeScript** (generics · utility types · narrowing) | `typescript/` | ⬜ Planned | TS mastery | 🟡 Medium — a strength to deepen |
| 7 | **Career & Brand** (CV · GitHub · LinkedIn · interview strategy) | `career/` | ⬜ Planned | Profile/brand | 🟢 Compounding |

> Each track uses the **same 8-part chapter template** the system-design track uses: plain-language idea → diagram → analogy → concrete example → tie-back to *your* projects → interview Q&A → trade-offs → TL;DR.

---

## The two interview gates (do these no matter what)

For product-company interviews, two things filter you out before anything else:

1. **DSA** — ~150–200 LeetCode (mostly medium). No shortcut. → [`dsa/`](dsa/00-README.md) ✅ (curriculum built, taught in TypeScript)
2. **System Design** — design Twitter / URL shortener / chat / notifications. → [`system-design/`](system-design/00-README.md) ✅

**Both interview gates now have full curricula, plus the [backend track](backend/00-README.md) is built** — that's the three biggest gaps (DSA, system design, backend) all covered. Remaining tracks (CS fundamentals, advanced TS, career/brand) raise your *level and comp* and polish the profile. Next suggested build: **CS fundamentals** (OS/networking/DB internals — quizzed even on full-stack loops) or **career/brand** (CV metrics, GitHub, LinkedIn).

---

## A sustainable weekly rhythm (you work full-time — ~10–12 hrs/week)

| Day | Focus | Time |
|-----|-------|------|
| Mon | DSA — 1 problem + learn the pattern | 45 min |
| Tue | Current track chapter (e.g. backend) | 1 hr |
| Wed | DSA — 1 problem (same pattern, reinforce) | 45 min |
| Thu | Build on Habex (apply what you learned) | 1 hr |
| Fri | DSA — 1 problem | 45 min |
| Sat | Deep block: 1 system-design chapter + 2 DSA + Habex feature | 3–4 hrs |
| Sun | Review + 1 blog/LinkedIn post + plan next week | 1–1.5 hrs |

**Rule of compounding:** 1 DSA problem/day for a year = ~250 problems. That alone clears the DSA gate.

---

## Milestones (mapped to the [things.md](things.md) roadmap)

### 🎯 Year 1 — Foundation & first jump (target 20–25 LPA)
- [ ] **50 LeetCode** done (arrays, two-pointers, sliding window, trees, BFS/DFS)
- [ ] Finish **system-design** track + do 3 mock designs out loud
- [ ] Ship a **Node + Express + PostgreSQL** backend for **Habex** (auth, DB, REST API)
- [ ] Add **Redis caching** to one Habex endpoint (e.g. dashboard summary)
- [ ] **Deploy backend** (Railway/Render) + basic **Docker**
- [ ] **CV rewrite**: add GitHub + santhu.site, metrics-driven bullets, drop "smart irrigation" filler
- [ ] **LinkedIn** updated; start applying (BrowserStack, Razorpay, Postman, Hasura)

### 🎯 Year 2 — Depth & positioning (target 30–35 LPA)
- [ ] **150 LeetCode** total (add DP, graphs, heaps, intervals)
- [ ] Read *Designing Data-Intensive Applications* (pairs with the system-design track)
- [ ] **Advanced TypeScript** track done
- [ ] One **ambitious project** (SaaS with users, or an OSS tool with stars)
- [ ] First **open-source PRs** merged + 3–5 blog posts

### 🎯 Year 3 — Target & execute (target 45–55 LPA)
- [ ] **200+ LeetCode**, comfortable with mediums under time
- [ ] Fluent in full **system-design + backend** interviews
- [ ] Target Zepto / Swiggy / CRED / Coinbase / Atlassian / Stripe — or an early-stage ESOP play

---

## This week (from [things.md](things.md), made concrete)

1. **Pick the next track to build** (see table above) and start it.
2. **Do LeetCode #1 today** — even one problem. Momentum > intensity.
3. **CV:** add `github.com/<you>` + `santhu.site`; rewrite one bullet to show impact, not description.
4. **Habex:** decide the first backend feature to add (recommend: **user auth + persist expenses to PostgreSQL**).

---

## How to use this hub

- Work **top-to-bottom within a track**; the chapters build on each other.
- Keep the **two gates (DSA + system design) always warm** — a little every week.
- Revise from each chapter's **TL;DR** and the system-design **interview playbook** before interviews.
- Update the milestone checkboxes above as you go — this file *is* your dashboard.

**Next:** tell me which track to build next and I'll generate it in full, same depth as system-design.
