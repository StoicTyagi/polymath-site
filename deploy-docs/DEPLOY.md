# DEPLOY.md — Polymath Site

Static HTML → GitHub → Vercel → custom domain on GoDaddy.

GitHub repo: **https://github.com/StoicTyagi/polymath-site** (already created and pushed)
Domain: **`ananttyagi.in`** (see *Domain spelling note* below — this is the double-t version, registered with GoDaddy on May 7, 2026)

---

## Quick path (TL;DR)

1. **Vercel:** go to https://vercel.com/new → "Import Git Repository" → pick `StoicTyagi/polymath-site` → leave all build settings as default → **Deploy**.
2. **Vercel Domains:** in the project → Settings → Domains → add `ananttyagi.in` and `www.ananttyagi.in` (Vercel will tell you exactly which DNS records to set).
3. **GoDaddy DNS:** in your `ananttyagi.in` DNS panel — add an `A` record `@ → 76.76.21.21` and a `CNAME` `www → cname.vercel-dns.com`. Wait 5–60 min for SSL.

Done. Site is live at `https://ananttyagi.in`.

---

## Detailed steps

### Step 1 — Vercel project setup (web UI)

CLI is not currently available on this machine (npm global install hit a permissions wall — see *Optional: Vercel CLI* at the bottom). The web UI is the path of least resistance and only takes 2 minutes.

1. Open **https://vercel.com/new**. Sign in with GitHub if you haven't (use the `StoicTyagi` account).
2. Vercel will list your GitHub repos. If `polymath-site` doesn't show up, click **"Adjust GitHub App Permissions"** and grant Vercel access to it.
3. Click **Import** next to `polymath-site`.
4. On the configuration screen:
   - **Project Name:** `polymath-site` (or rename — doesn't affect domain)
   - **Framework Preset:** **Other** (it's plain static HTML)
   - **Root Directory:** leave as default (the repo root *is* the site root)
   - **Build & Output Settings:** leave all defaults — no build command, no output directory override. The repo's `vercel.json` handles `cleanUrls`, caching, etc.
5. Click **Deploy**.

Within ~30 seconds you get a live `*.vercel.app` URL. Test it:
- `/` → home page
- `/neural-net` → neural-net page (the `cleanUrls: true` in `vercel.json` makes this work without `/index.html`)
- `/img/school-of-athens.jpg` → image loads

### Step 2 — Add the custom domain in Vercel

1. In the Vercel project → **Settings** → **Domains**.
2. In the input box, type `ananttyagi.in` and click **Add**.
3. Vercel will show a card asking you which version is canonical. Choose **`ananttyagi.in`** as primary (the bare apex). It will then auto-suggest adding `www.ananttyagi.in` as a redirect — accept that.
4. Vercel will now display the DNS records you need to set on GoDaddy. They will match the table in the next step. Leave this tab open — you'll come back to verify.

### Step 3 — GoDaddy DNS configuration

Log into GoDaddy → My Products → find `ananttyagi.in` → **DNS** (or "Manage DNS").

Before adding new records, **delete or edit** any existing default GoDaddy records that conflict:
- The default parking `A` record on `@` (often points to a GoDaddy IP) — delete it.
- The default `CNAME` on `www` (often `@`) — delete or edit it.
- Don't touch `NS` records, the `SOA` record, or any MX records you've intentionally configured (e.g., for email).

Then add:

| Type  | Name | Value                  | TTL              |
|-------|------|------------------------|------------------|
| A     | `@`  | `76.76.21.21`          | 600 (10 min) or default 1 hour |
| CNAME | `www` | `cname.vercel-dns.com` | 600 or default   |

That's it. The apex (`ananttyagi.in`) gets the A record. The `www` subdomain gets a CNAME pointing at Vercel's DNS hostname.

> **Why an A record on apex instead of CNAME?** DNS doesn't allow `CNAME` on the root of a domain (the bare apex). `76.76.21.21` is Vercel's static anycast IP for apex domains, documented at https://vercel.com/docs/projects/domains/working-with-domains.

### Step 4 — Verify and wait for SSL

Back in Vercel → Settings → Domains. Both `ananttyagi.in` and `www.ananttyagi.in` should change from "Invalid Configuration" to a green "Valid Configuration" within 5–60 minutes (DNS propagation).

Once DNS verifies, Vercel auto-provisions a Let's Encrypt SSL certificate. This usually takes another 1–5 minutes. You'll know it's done when:
- Visiting `https://ananttyagi.in` shows the site, padlock in browser
- Vercel shows both domains as "Valid"

If 24 hours pass and SSL still hasn't issued, see *Troubleshooting*.

---

## Subdomain options — which URL to give people

You have three reasonable choices. Pick one and tell Vercel it's the canonical:

1. **Bare apex: `https://ananttyagi.in`** — recommended.
   - Cleanest, shortest, prints best on a business card.
   - `www.ananttyagi.in` set as redirect → apex.
2. **`https://www.ananttyagi.in`** — old-school, slightly more robust to certain bad ISPs/DNS resolvers, but unfashionable in 2026.
3. **`https://polymath.ananttyagi.in`** (or `me.`, `i.`, etc.) — keeps the apex free for something else later (e.g., a future startup landing or email-only mail server). Set this up as a `CNAME polymath → cname.vercel-dns.com`. Worth considering if you ever want `ananttyagi.in` to host a different site.

**Recommendation:** bare apex. It matches the "cozy corner of the internet" brief — it should feel like *the* home, not a subdirectory of one.

---

## DNS records — full reference

For the recommended bare-apex setup:

| Type  | Name      | Value                    | Purpose                                |
|-------|-----------|--------------------------|----------------------------------------|
| A     | `@`       | `76.76.21.21`            | Apex `ananttyagi.in` → Vercel          |
| CNAME | `www`     | `cname.vercel-dns.com`   | `www.ananttyagi.in` → Vercel (redirect)|

If you later want a custom subdomain like `polymath.ananttyagi.in`:

| Type  | Name        | Value                    |
|-------|-------------|--------------------------|
| CNAME | `polymath`  | `cname.vercel-dns.com`   |

---

## Troubleshooting

**"Invalid Configuration" persists in Vercel after >2 hours**
- Check DNS has actually propagated: `dig ananttyagi.in +short` should return `76.76.21.21`. `dig www.ananttyagi.in +short` should resolve via `cname.vercel-dns.com`.
- If you see GoDaddy's parking IP still, you didn't delete the old `A` record — go back and delete it.
- TTL on the old record was high? You may need to wait it out. India ISPs are usually quick (under 1 hr) but worst case is 48 hr.

**SSL certificate not issuing after 24 hours**
- In Vercel domain settings, click **"Refresh"** next to the domain.
- If still failing, click **"Renew Certificate"** — sometimes Let's Encrypt rate-limits temporarily.
- Last resort: remove the domain from Vercel and re-add it. This re-triggers the validation flow.

**Site loads on `*.vercel.app` but not on `ananttyagi.in`**
- DNS hasn't propagated yet, or a record is wrong. Run `dig ananttyagi.in` from a clean DNS resolver: `dig @1.1.1.1 ananttyagi.in +short`.
- Cleared local DNS cache? On macOS: `sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder`.

**Images / CSS not loading on the live site**
- Hard reload: `Cmd+Shift+R`. Vercel's defaults plus the `vercel.json` cache headers should handle this, but a stale CDN cache can hide for a few minutes after deploy.

**`/neural-net` returns 404**
- This means `cleanUrls: true` isn't being honored. Confirm `vercel.json` is at the *repo root* (it is — already committed). Trigger a redeploy from the Vercel dashboard.

**Pushing a new commit doesn't trigger a deploy**
- Check Vercel project → Settings → Git. Confirm "Production Branch" is `main` and the repo is connected. Disconnect/reconnect if needed.

**Want to roll back?**
- Vercel project → Deployments → find the previous green deploy → menu → **"Promote to Production"**. Rollback in 5 seconds.

---

## Domain spelling note — important

The original brief said `anantyagi.in` (single `t`). When I checked WHOIS:

- **`anantyagi.in`** — single-t — **available, not registered**.
- **`ananttyagi.in`** — double-t — **registered with GoDaddy on May 7, 2026**, registrant in Uttar Pradesh, IN. This matches your IG/LinkedIn handle `ananttyagi`.

So the double-t version is the one you actually own, and it's also the one consistent with your public handles. **Good outcome — proceed with `ananttyagi.in`.** All instructions above use this spelling.

If you ever want the single-t for any reason, it's still available to register — but I wouldn't bother. Two domains for one identity is just one more thing to renew. Keep it simple.

---

## Hard rules (already in `.gitignore`)

The following are excluded from the GitHub repo and will NOT ship:

- `.herenow/state.json` — local here.now claim token. Stays out of the public repo.
- `deploy-docs/SHIP-CHECKLIST.md` — PM-only file, never deploys.
- `node_modules/`, `*.log`, `.env*`, `.vercel/` — defensive.

`deploy-docs/DEPLOY.md` (this file) **is** in the repo, because it's useful and not sensitive. If you'd rather it stay private, add `deploy-docs/` to `.gitignore` and `git rm -r --cached deploy-docs/`.

---

## Optional: Vercel CLI

The CLI is convenient for one-shot deploys (`vercel deploy --prod`) and pulling environment variables. It's **not required** — the web flow above does everything.

To install:

```bash
# This machine hit an EACCES on /usr/local/lib/node_modules — try one of these:
sudo npm install -g vercel
# or, better, configure npm to use a user prefix and skip sudo entirely:
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.zshrc
source ~/.zshrc
npm install -g vercel
```

Then from the repo root:

```bash
vercel link              # link this folder to the Vercel project
vercel deploy --prod     # one-shot prod deploy (skips Git triggers)
vercel domains add ananttyagi.in   # alternative to the web UI
```

Most of the time you won't need any of this — `git push origin main` already auto-deploys via the Vercel-GitHub integration.

---

## What's next

1. Do Step 1 (web UI import) — you'll have a live `*.vercel.app` URL in 30 seconds.
2. Do Step 2 (add domain) and Step 3 (GoDaddy DNS) — 5 min of clicking.
3. Wait for SSL — 5–60 min usually.
4. Tell people to go to `ananttyagi.in`.

Future updates: edit files, `git commit`, `git push`. Vercel auto-deploys on every push to `main`. Preview deploys for PRs/branches come for free.
