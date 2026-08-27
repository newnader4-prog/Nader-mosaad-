# Dr Nader Mosaad — Personal Site + Admin CMS

Your original link-in-bio design, unchanged, now backed by a real content
management system: secure login, an admin dashboard, and a database so you
never have to touch HTML/CSS/JS again.

## Why plain Node.js instead of Next.js + Supabase

The build environment this was created in has no internet access, so
Supabase/Next.js packages couldn't be installed or tested. Per your own
fallback instruction ("if the platform has a preferred native backend/
database system, use it instead"), this is built with **zero npm
dependencies** — only Node's built-in modules:

- `crypto.scrypt` for password hashing (no `bcrypt` needed)
- Signed, `httpOnly` cookies for sessions (no `express-session` needed)
- A JSON file (`data/db.json`) as the database, behind a small `db.js`
  layer so every read/write goes through one place

This means **no `npm install`, no build step, no external services** —
just `node server.js` — and it runs identically on your laptop, a VPS, or
any host that runs Node. If you later want real Postgres/Supabase, only
`db.js` needs to change; nothing else touches the data layer directly.

## Running it

```bash
node server.js
```

Then open **http://localhost:3000**. On first run, visit
**http://localhost:3000/admin/login** — since no admin account exists
yet, you'll be shown a one-time "Create your admin account" form instead
of a login form. After that, it's a normal login page.

To use a different port: `PORT=8080 node server.js`.

## What's inside

```
public/
  index.html          the public homepage (your original design, now data-driven)
  admin/
    login.html         admin login (+ first-run account setup)
    dashboard.html      the CMS dashboard
uploads/               uploaded photos land here
data/
  db.json               the "database" (profile, links, sections, settings, users)
  secret.key             auto-generated, used to sign session cookies — keep private
server.js               all routes: static files + the API
db.js                   the only file that reads/writes db.json
auth.js                 password hashing + session tokens
```

## Using the dashboard

- **Profile** — name, title, description, university, year, college, in
  both English and Arabic.
- **Social Links** — add unlimited links, drag to reorder, toggle
  enabled/disabled, edit URL/icon/style at any time.
- **Sections** — add unlimited custom sections (About, Education,
  Certificates, Services, Portfolio…) with bilingual title + description,
  without ever editing code.
- **Media** — upload/replace your profile photo (PNG/JPEG/WebP, up to 5MB).
- **Appearance** — accent colors, default theme, button style.
- **Account** — change your password.

Every save writes straight to `data/db.json`, and the public homepage
reads from it live — so changes appear immediately on refresh.

## Security notes

- Passwords are hashed with `scrypt` + a random salt per user — never
  stored in plain text.
- Sessions are signed cookies (`httpOnly`, `SameSite=Strict`) — the admin
  dashboard is fully inaccessible without a valid session.
- Login is rate-limited (10 attempts / 15 minutes per IP).
- All admin API routes check the session on every request; there is no
  client-side-only protection.
- **Before deploying publicly**, put this behind HTTPS (e.g. a reverse
  proxy like Caddy, Nginx, or your host's built-in TLS) and uncomment the
  `Secure` cookie flag noted in `server.js`.
- Back up `data/db.json` and `data/secret.key` — losing `secret.key`
  invalidates all sessions (harmless, just re-login); losing `db.json`
  loses your content.

## Deploying

Any host that runs Node 18+ works: a small VPS, Render, Railway, Fly.io,
a Raspberry Pi, etc. There's no build step — just copy the folder up and
run `node server.js` (or `npm start`), then point your domain at it.

## Extending later

The architecture leaves room for everything on your future list (blog,
analytics, QR code, SEO settings, link-click stats, custom domain) without
a rebuild: add a new section type in `db.js`, a couple of routes in
`server.js`, and a panel in the dashboard.
