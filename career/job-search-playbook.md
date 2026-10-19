# Job Search Playbook — Santhosh R

Companion to `roadmap-35lpa.pdf`. That file is *what to learn*. This one is *how to actually get
hired*. Target: SDE-2 at an Indian product company, Bengaluru, ₹28–35 L.

---

## 1. How these jobs actually get filled

Before picking platforms, understand the channels — they do not convert anywhere near equally.

| Channel | Rough conversion | What it needs from you |
|---|---|---|
| **Referral** | By far the highest. At most product companies referrals are a large share of engineering hires | A network. This is why 22 connections was such an expensive number |
| **Recruiter inbound** | Moderate. They find you | An optimised LinkedIn + a *fresh* Naukri profile |
| **Cold careers-page apply** | Lowest. Single-digit percentages are normal | Nothing — which is exactly why it doesn't work |

**The implication:** most people spend 90% of their effort on the channel that converts worst.
Your effort should be inverted — referral first, inbound second, cold apply as the fallback for
companies where you have no other route in.

---

## 2. Profiles to create

### Tier 1 — this week

| Platform | Why it matters for you |
|---|---|
| **Naukri.com** | Non-negotiable in India. A large share of Indian recruiters search Naukri's resume database *before* LinkedIn. If you are not there, you are invisible to them |
| **GitHub** (you have it — needs work) | Your only public proof of how you actually write code. Currently unpinned repos and a stale README |
| **Instahyre** | Curated, product-company heavy, low noise. Recruiters approach you. Good signal for exactly your target tier |
| **LeetCode** (public profile) | Not useful *yet* — becomes real evidence at 250+ solved. Create it now so the contribution history has age on it |

### Tier 2 — worth an hour each

| Platform | Why |
|---|---|
| **Wellfound** (ex-AngelList) | Startups, often direct founder/hiring-manager contact, no recruiter layer |
| **Cutshort** | Indian, decent product-company coverage, skill-match based |
| **Hirist** | Indian, tech-only. Lower noise than Naukri |

### Tier 3 — research tools, not application channels

| Platform | Use it for |
|---|---|
| **levels.fyi** | Real comp data by company and level. Check before every negotiation |
| **AmbitionBox** | India-specific salary + company reviews |
| **Glassdoor** | Interview question banks — people post actual rounds |
| **Blind / Grapevine** | Candid employee intel on teams, managers, layoffs, comp bands. Grapevine is the India-focused one |

### Skip

Indeed, Monster, Shine, Foundit — overwhelmingly service-company and staffing traffic. You will get
volume, not quality, and the recruiter calls will waste your evenings.

**X/Twitter** — genuinely useful for dev-community visibility and warm intros, but only if you will
actually post. A dormant account does nothing. Decide honestly; LinkedIn already covers this need.

---

## 3. GitHub — fix these five things

This is your highest-leverage profile after LinkedIn, and it is currently underselling you badly.

1. **Profile README.** Create a repo named `Santhu32144` (same as your username) with a `README.md`.
   It renders on your profile page. Keep it short: who you are, what you've shipped, the stack,
   links to Habex and santhu.site.
2. **Pin three repos.** Jourvia is ready. Habex currently lives at `ZenblockLabs/exphabbits` — transfer it to your personal account and rename it `habex` first. FraudShield is on a collaborator’s account, so it cannot be pinned from yours. Unpinned, a visitor lands on a chronological
   jumble and leaves.
3. **Fix the Jourvia README.** Your own notes say it is stale and badly understates the app. A
   recruiter clicking from a polished LinkedIn card into a thin README is worse than no link.
4. **Every pinned repo needs:** a one-line description at the top, screenshots or a GIF, a stack
   list, setup instructions, and a live link where one exists. Most engineers skip screenshots —
   they are the single thing that makes a repo look finished.
5. **Green squares.** The contribution graph is read as a proxy for consistency, fairly or not.
   The Habex scale work in Phase 2 of the roadmap will fill it naturally. Don't fake it with
   commit-bot nonsense — that is visible and embarrassing.

---

## 4. Naukri — the one mechanic nobody tells you

Naukri's recruiter search ranks results heavily by **how recently the profile was updated.** A
profile touched today outranks an identical one touched three weeks ago.

**So: log in and make a trivial edit every 5–7 days.** Change a word, re-upload the same resume,
toggle a preference. It takes thirty seconds and it is the difference between page 1 and page 6 of
a recruiter's search results.

**Setting it up:**
- Upload `santhu.pdf` as the resume. Naukri parses it, so keep the text selectable (yours is, since
  it's Chrome-rendered HTML rather than a scan).
- **Resume headline:** same discipline as LinkedIn —
  `Full Stack Engineer | React · TypeScript · Node.js | 2.8 yrs | Bengaluru`
- **Key skills:** load these deliberately, they are the search index —
  `React, TypeScript, JavaScript, Node.js, Express, REST API, PostgreSQL, MongoDB, Socket.IO,
  Redux, HTML, CSS, Git, Linux, Python, FastAPI, Full Stack Development`
- **Expected CTC:** leave blank or set a range above target. Never anchor yourself low here.
- **Notice period:** set it accurately; it is a hard filter recruiters use.

**Brace for the noise.** Naukri will generate calls from staffing firms about roles you don't want.
That is the cost of being in the database where the good recruiters also search. Screen fast: ask
"which company, what's the band, is this direct or through a vendor?" in the first two minutes.

---

## 5. The referral machine

This is the part that actually matters, and it is 80% of where your effort should go.

### Step 1 — Build the target list

From the roadmap, tiered:

- **Primary:** CRED, Razorpay, Groww, Zepto, Swiggy, PhonePe, Postman, Zeta, Navi, Flipkart
- **Strong fit, less crowded:** Juspay, Setu, Hasura, Meesho, Slice, Jupiter, Rapido, Porter,
  Dream11, Ather, Bounce
- **Calibration:** mid-size product firms you would not accept

### Step 2 — Find your route in, per company

For each target, in priority order:

1. **SIT alumni.** Siddaganga school page → Alumni tab → filter *Where they work*. This is your
   single best asset and most people never open it.
2. **Ex-Delta colleagues** who have already moved to product companies. They know your work
   firsthand, which makes their referral far stronger than an alum's.
3. **2nd-degree connections** — LinkedIn search: company + "Software Engineer" + 2nd degree. A
   shared connection makes a cold message warm.
4. **Nobody?** Then cold-apply *and* message an engineer on the team in parallel.

### Step 3 — The message that works

Not "can you refer me." That asks them to vouch for a stranger.

```
Hi [Name] — fellow SIT grad (EIE '24). I'm a full stack engineer at Delta Electronics —
React/TypeScript frontend and a Node backend I built and shipped onto industrial hardware.

I saw [Company] has an SDE-2 opening on [team]. Before I apply, would you be open to a
10-minute call about what the team's actually working on? Happy to work around your schedule.

Either way, good to connect.
```

**Why this works:** you asked for information, not a favour. If the call goes well, the referral is
usually offered without you asking. If they're busy, you've still made a warm contact for later.

### Step 4 — Warm the network *before* you need it

This is the part people get wrong. A referral request from someone who connected eight months ago
and has been posting about their work lands completely differently from one sent by a stranger the
week a job opens.

Which is exactly why the roadmap says start LinkedIn posting in month 1, not month 8.

---

## 6. Recruiter inbound — set it and forget it

- **LinkedIn:** Open to work → **Recruiters only**. Titles: `Software Engineer`, `Full Stack
  Engineer`, `Backend Engineer`, `SDE 2`. Location: Bengaluru + Remote India. Start date:
  *Immediately* — recruiters filter on this.
- **Naukri:** the weekly-refresh habit above.
- **Instahyre:** complete the profile fully; it gates you out of matches otherwise.

Then stop thinking about it. This channel runs in the background while you focus on referrals.

---

## 7. Research before you apply — every time

| Question | Where |
|---|---|
| What does this level actually pay here? | levels.fyi, AmbitionBox |
| What are the interview rounds? | Glassdoor interview section, LeetCode company tag |
| Is the team/company actually healthy? | Blind, Grapevine |
| Who do I know there? | LinkedIn 2nd degree + SIT alumni filter |

Twenty minutes of this before an application is worth more than ten extra applications. It also
gives you the specific questions that make you sound serious in round one.

---

## 8. Track it

One spreadsheet. Columns:

```
Company | Role | Channel (referral/inbound/cold) | Referrer | Applied date |
Status | Next step | Rounds cleared | Comp discussed | Notes
```

Without this you will lose track of who referred you where, and follow-ups are where a
surprising number of processes actually restart. Follow up once after 7 days, once after 14, then
let it go.

---

## 9. When to actually start applying

**This is the part where sequencing matters more than effort.**

You are at month 0 of the roadmap: DSA not started, capstone not built, no scale metrics. Every
primary target gates on two DSA rounds before a human reads your resume.

**A failed loop costs a 6–12 month cooldown at that company.** So:

| When | What |
|---|---|
| **Now** | Create all Tier 1 + 2 profiles. Fix GitHub. Start connecting and posting. Open referral conversations — asking about teams, not jobs |
| **Month 3** | 2–3 **calibration** interviews at companies you'd decline. Pure practice. This is where you discover you go silent when stuck |
| **Month 6** | Strong-fit tier — Juspay, Setu, Meesho, Porter. Real offers possible |
| **Month 8** | Primary targets, all applications inside a **two-week window** so offers land together |

**The two-week window is the whole negotiation strategy.** One offer has almost no leverage. Three
offers in the same week is what turns ₹28 L into ₹35 L.

**The exception:** if a genuinely great role appears through a referral before month 8, take the
interview. Referral routes often skip or soften the DSA screen, and a warm intro is worth more than
perfect timing. Just don't burn your top three companies on a cold application in month 2.

---

## 10. Weekly cadence

| Day | Job-search action (30 min max — the roadmap study hours are separate) |
|---|---|
| Mon | 15–20 LinkedIn connections, always with a note |
| Tue | 2 substantive comments on target-company engineers' posts |
| Wed | Refresh Naukri; check Instahyre matches |
| Thu | 1 referral conversation opened (the info-request message) |
| Fri | — |
| Sat | Write the week's LinkedIn post |
| Sun | Update the tracker; follow up on anything 7+ days stale |

Thirty minutes a day. The heavy lifting is still the roadmap — this is just keeping the pipeline
warm so it's ready when you are.

---

## The one-line version

Build every profile now, build the network now, build the skills for eight months, then apply
everywhere at once. The profiles and the network compound while you learn; the applications only
get one shot each.
