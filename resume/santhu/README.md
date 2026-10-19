<h1 align="center">Hi, I'm Santhosh 👋</h1>

<p align="center">
  <a href="https://santhu.site">
    <img src="https://readme-typing-svg.demolab.com?font=Fira+Code&weight=500&size=20&duration=3200&pause=1000&color=2F81F7&center=true&vCenter=true&width=640&lines=Full+Stack+Engineer+%40+Delta+Electronics;React+%2B+TypeScript+%C2%B7+Node.js+%C2%B7+Embedded+Linux;From+the+touch+screen+down+to+the+OS" alt="Full Stack Engineer @ Delta Electronics · React + TypeScript · Node.js · Embedded Linux" />
  </a>
</p>

<p align="center">
  <a href="https://santhu.site"><img alt="Portfolio" src="https://img.shields.io/badge/Portfolio-santhu.site-2EA44F?style=for-the-badge&logo=googlechrome&logoColor=white" /></a>
  <a href="https://www.linkedin.com/in/santhosh-r-64259b3b0"><img alt="LinkedIn" src="https://img.shields.io/badge/LinkedIn-Santhosh%20R-0A66C2?style=for-the-badge&logo=data:image/svg%2bxml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCIgZmlsbD0ibm9uZSIgc3Ryb2tlPSJ3aGl0ZSIgc3Ryb2tlLXdpZHRoPSIyIiBzdHJva2UtbGluZWNhcD0icm91bmQiIHN0cm9rZS1saW5lam9pbj0icm91bmQiPjxwYXRoIGQ9Ik0xNiA4YTYgNiAwIDAgMSA2IDZ2N2gtNHYtN2EyIDIgMCAwIDAtMi0yIDIgMiAwIDAgMC0yIDJ2N2gtNHYtN2E2IDYgMCAwIDEgNi02eiIvPjxyZWN0IHdpZHRoPSI0IiBoZWlnaHQ9IjEyIiB4PSIyIiB5PSI5Ii8%2BPGNpcmNsZSBjeD0iNCIgY3k9IjQiIHI9IjIiLz48L3N2Zz4%3D" /></a>
  <a href="mailto:santhosh.dev.6704@gmail.com"><img alt="Email" src="https://img.shields.io/badge/Email-Say%20hello-EA4335?style=for-the-badge&logo=gmail&logoColor=white" /></a>
  <a href="https://habex.santhu.site"><img alt="Habex, live app" src="https://img.shields.io/badge/Live%20app-Habex-6D28D9?style=for-the-badge&logo=vercel&logoColor=white" /></a>
</p>

I'm a **full stack engineer** in Bengaluru. For the past 2.5+ years at **Delta Electronics** I've built the software that runs on industrial power distribution units. That covers the React touch screen an operator taps, the Node.js service behind it, and the embedded Linux it boots on.

- 🏭 **At work:** Senior Software Engineer and primary developer of **PDU Gen-3**, covering the touch-panel UI, the admin console, the backend and the installer
- 📱 **On the side:** **Jourvia**, a React Native app for group trips, and **Habex**, a finance and habit tracker that's live on my domain
- 🌱 **Learning now:** deeper backend (PostgreSQL, Redis, queues, Docker), system design and DSA
- 💬 **Ask me about:** React performance on low-power hardware, JWT auth with RBAC, or getting a web stack running on an embedded board
- 🏍️ **Off the keyboard:** motorcycles

## 🚀 What I've built

### ⚡ PDU Gen-3: software for industrial power distribution units
**Delta Electronics · in production** · `React 18` `Redux Toolkit` `TypeScript` `Node.js` `Express 5` `Socket.IO` `SQLite` `nginx` `Yocto`

Three apps ship together on every unit. Operators use a touch-panel UI, an admin console handles configuration, and a backend runs on the device itself. The code is proprietary, so here is how it fits together:

```mermaid
flowchart LR
    subgraph pdu["Touch-panel computer: ARM64 embedded Linux, built with Yocto"]
        ui["Touch-panel UI<br/>React · Redux Toolkit<br/>WebKit kiosk"]
        admin["Admin console<br/>Vite · React · Tailwind<br/>opened from a browser"]
        nginx["nginx<br/>serves both apps"]
        api["Node.js backend<br/>Express 5 · Socket.IO<br/>JWT + RBAC · systemd"]
        db[("SQLite<br/>config · events · alarms · audit")]
        ui --> nginx
        admin --> nginx
        nginx -->|"/api · REST + WebSocket"| api
        api --> db
    end
    api -->|"device scripts · OTA"| boards["PDU controller boards"]
```

- **Instant revocation:** 15-minute access tokens are held in memory and renewed with an httpOnly refresh cookie. Roles and menu permissions are looked up again on every refresh, so changes reach sessions that are already logged in. 3 roles and 19 permission keys are enforced in both the UI and server middleware.
- **Kiosk resilience:** an energy sampler records power every 5 s into a rolling one-hour window that survives restarts. A boot self-test reports *ready*, *degraded* or *offline*, and the debounced "backend disconnected" banner never blocks the screen.
- **Low-power performance:** memoized rows and paged rendering keep tables of up to 216 branch circuits smooth. The UI also avoids blur effects, which tear on the board's GPU.
- **One-file deploys:** `build.sh` versions the backend and compiles it into an ARM64 binary with `pkg`, then bundles it with both web apps into a self-extracting installer. The installer sets up systemd, validates and reloads nginx, and configures udev USB automount. It backs up the previous build before replacing it. OTA updates cover the panel and its controller boards.
- **Embedded Linux:** customised the board's Yocto image (U-Boot, kernel config, device tree).

### 🧭 Jourvia: a travel companion for group trips
**Android app** · `React Native 0.85 (New Architecture)` `TypeScript` `Zustand` `Firebase` `Kotlin` `Jest` `GitHub Actions` · 🔒 *source is private for now*

Plan a trip with friends, follow the route together, and settle up at the end. About 36k lines of TypeScript and 1.3k of Kotlin.

- **Live trip mode:** GPS auto check-in at every stop (300 m radius, 2-minute dwell) runs in an Android foreground service. Live location sharing and "X reached Y" alerts keep the group in sync.
- **Routes and weather:** paste a Google Maps link and get geocoded stops, real road distance (OSRM), a route map and a forecast for each stop.
- **Group tools:** chat with voice notes, packing lists, expense splitting that settles up in the fewest payments, and a hold-to-trigger SOS that texts your location to emergency contacts.
- **Native Android:** 5 home-screen widgets and a voice recorder written in Kotlin. The UI ships in English and 9 Indian languages through a type-safe i18n engine I wrote.
- **Security and tests:** deny-by-default Firestore rules for owner, editor and member roles. 65 Jest tests run in CI, which also typechecks and lints. Each version tag publishes a signed APK.

### 💸 Habex: expense, habit and investment tracker
**[habex.santhu.site ↗](https://habex.santhu.site)** · [code](https://github.com/Santhu32144/habex) · `React 18` `TypeScript` `Vite` `Supabase` `PostgreSQL` `Tailwind` `shadcn/ui` `Recharts`

A finance and habits app on my own domain. It tracks expenses with budgets and recurring rules, habit streaks with a calendar heatmap and a 21-day challenge, and shared investment groups. It also has an admin dashboard.

- **Database-side security:** row-level security on every table (41 policies). Group roles from `view` up to `admin` are enforced by `SECURITY DEFINER` functions, and receipts go to a private storage bucket.
- **PIN login:** 6-digit PINs are hashed with bcrypt. The database enforces a 15-minute lockout after 5 wrong tries, and an edge function then issues a real Supabase session.
- **Under the hood:** 12 versioned SQL migrations and 3 Deno edge functions (PIN login, admin user list, activity feed). CSV and PDF export works without a PDF library.

### 🌐 santhu.site: my portfolio
**[santhu.site ↗](https://santhu.site)** · [code](https://github.com/Santhu32144/about_me) · `React` `TypeScript` `Vite` `Tailwind` `Framer Motion` `Recharts`

An animated personal site with projects, an experience timeline and a blog of React Native setup guides. A command palette jumps between sections, and there's a Konami-code easter egg if you know the sequence 🎮

## 🧰 Tech stack

<table>
  <tr>
    <td><b>Languages</b></td>
    <td><img src="https://skillicons.dev/icons?i=ts,js,py,kotlin,c,cpp" alt="TypeScript, JavaScript, Python, Kotlin, C, C++" /></td>
  </tr>
  <tr>
    <td><b>Frontend &amp; mobile</b></td>
    <td><img src="https://skillicons.dev/icons?i=react,redux,tailwind,vite,html,css" alt="React, Redux, Tailwind CSS, Vite, HTML, CSS" /><br /><sub>React Native (New Architecture) · Redux Toolkit · Zustand · shadcn/ui · Recharts · Framer Motion</sub></td>
  </tr>
  <tr>
    <td><b>Backend</b></td>
    <td><img src="https://skillicons.dev/icons?i=nodejs,express,fastapi,flask" alt="Node.js, Express, FastAPI, Flask" /><br /><sub>REST API design · Socket.IO · JWT + refresh tokens · RBAC · Winston · SQLAlchemy · Pydantic</sub></td>
  </tr>
  <tr>
    <td><b>Data</b></td>
    <td><img src="https://skillicons.dev/icons?i=postgres,supabase,sqlite,firebase,mysql,mongodb" alt="PostgreSQL, Supabase, SQLite, Firebase, MySQL, MongoDB" /><br /><sub>Row-level security · SQL migrations · Firestore security rules</sub></td>
  </tr>
  <tr>
    <td><b>Embedded &amp; DevOps</b></td>
    <td><img src="https://skillicons.dev/icons?i=linux,ubuntu,bash,nginx,docker,githubactions" alt="Linux, Ubuntu, Bash, nginx, Docker, GitHub Actions" /><br /><sub>Yocto · BitBake · U-Boot · device tree · systemd · udev · Modbus · CAN · I²C</sub></td>
  </tr>
  <tr>
    <td><b>Tools</b></td>
    <td><img src="https://skillicons.dev/icons?i=git,gitlab,vercel,jest,postman,figma" alt="Git, GitLab, Vercel, Jest, Postman, Figma" /></td>
  </tr>
</table>

## 📊 GitHub activity

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://github-readme-stats.vercel.app/api?username=Santhu32144&show_icons=true&hide_border=true&theme=github_dark&custom_title=Santhosh%27s%20GitHub%20Stats" />
    <img height="170" alt="Santhosh's GitHub stats" src="https://github-readme-stats.vercel.app/api?username=Santhu32144&show_icons=true&hide_border=true&custom_title=Santhosh%27s%20GitHub%20Stats" />
  </picture>
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://github-readme-stats.vercel.app/api/top-langs/?username=Santhu32144&layout=compact&hide_border=true&langs_count=6&theme=github_dark" />
    <img height="170" alt="Most used languages" src="https://github-readme-stats.vercel.app/api/top-langs/?username=Santhu32144&layout=compact&hide_border=true&langs_count=6" />
  </picture>
</p>

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://streak-stats.demolab.com/?user=Santhu32144&hide_border=true&theme=github-dark-blue" />
    <img alt="GitHub contribution streak" src="https://streak-stats.demolab.com/?user=Santhu32144&hide_border=true" />
  </picture>
</p>

<p align="center">
  <img src="https://komarev.com/ghpvc/?username=santhu32144&label=Profile%20views&color=0e75b6&style=flat" alt="Profile views" />
</p>
