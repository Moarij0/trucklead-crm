# 🚀 TruckLead CRM — Deployment Guide

A production-ready, white-label SaaS. The client needs **no technical knowledge** —
you deploy it once and hand over the URL + admin login.

Everything runs with **one command** using Docker: PostgreSQL database, FastAPI
backend, the React frontend, and automatic HTTPS — all wired together.

---

## 1. What you need

- A Linux server (VPS) with a public IP. Any of these work great:
  **DigitalOcean, Hetzner, Contabo, AWS (Lightsail/EC2), Railway.**
  Recommended minimum: **1 vCPU / 2 GB RAM / 25 GB disk**.
- (For HTTPS) A domain or subdomain, e.g. `crm.yourclient.com`, pointed at the server IP.

That's it. Docker installs everything else.

---

## 2. One-time server setup

SSH into the server and install Docker:

```bash
curl -fsSL https://get.docker.com | sh
```

Copy the project to the server (via `git clone`, `scp`, or an upload), then:

```bash
cd TruckLeadCRM
cp .env.example .env
nano .env        # set passwords, admin login, and SITE_ADDRESS (see below)
```

**Minimum things to change in `.env`:**

| Variable | Set it to |
|----------|-----------|
| `POSTGRES_PASSWORD` | a strong random password |
| `SECRET_KEY` | run `openssl rand -hex 32` and paste the result |
| `ADMIN_EMAIL` / `ADMIN_PASSWORD` | the client's first admin login |
| `SITE_ADDRESS` | `:80` for IP-only, or `crm.yourclient.com` for HTTPS |

---

## 3. Deploy (the one command)

```bash
docker compose up -d --build
```

- Builds and starts everything.
- Creates the database schema automatically (**automatic migrations** — no manual steps).
- Creates the admin account from `.env`.

Open the app:

- **With a domain:** `https://crm.yourclient.com` (HTTPS is automatic — see below)
- **IP only:** `http://YOUR_SERVER_IP`

Log in with the `ADMIN_EMAIL` / `ADMIN_PASSWORD` you set.

---

## 4. HTTPS (automatic, free)

1. In your DNS, add an **A record**: `crm.yourclient.com → YOUR_SERVER_IP`.
2. In `.env` set `SITE_ADDRESS=crm.yourclient.com` (no `http://`, no port).
3. Redeploy:

   ```bash
   docker compose up -d --build
   ```

Caddy automatically obtains and renews a Let's Encrypt certificate. Make sure
ports **80 and 443** are open in the server firewall.

---

## 5. White-label branding (no code)

After first login, go to **Settings → Branding** (admin only) and set:

- Company / app name, tagline
- Logo + favicon (upload)
- Accent color (live preview) & default theme
- Contact email, phone, website, support URL

Changes apply instantly across the whole app. This is how you resell the CRM
to different clients under their own brand — **nothing is hardcoded.**

Add the client's team under **Settings → Team**.

---

## 6. Importing the client's leads

**Leads → Import** opens a wizard:
1. Upload a **CSV or Excel (.xlsx)** file.
2. Columns are auto-detected — adjust the mapping if needed.
3. Choose duplicate handling (skip / merge / create).
4. Preview, then import (handles thousands of rows).

Export anytime via **Leads → Export → CSV / Excel** (respects active filters).

---

## 7. Backups

Manual backup:

```bash
./scripts/backup.sh
```

Automatic daily backup (2 AM) — add to `crontab -e`:

```bash
0 2 * * * cd /opt/TruckLeadCRM && ./scripts/backup.sh >> /var/log/truckcrm-backup.log 2>&1
```

Backups are gzip-compressed in `./backups/` (last 14 kept). Restore one with:

```bash
./scripts/restore.sh ./backups/truckcrm-YYYYMMDD-HHMMSS.sql.gz
```

> Store copies off-server (e.g. `scp` to your machine or an S3 bucket) for safety.

---

## 8. Updating the app

```bash
git pull            # or upload the new files
docker compose up -d --build
```

The database and its data are preserved (stored in the `db_data` Docker volume).

---

## 9. Provider quick notes

- **DigitalOcean / Hetzner / Contabo / Lightsail:** create an Ubuntu droplet/instance,
  open ports 22/80/443, then follow steps 2–4. Simplest path.
- **AWS EC2:** same, but open 80/443 in the Security Group.
- **Railway:** deploy `backend/` and `frontend/` as services and add a Railway
  PostgreSQL plugin; set the same env vars. (Railway provides HTTPS domains, so
  you can skip Caddy and set `CORS_ORIGINS` to the frontend URL.) For most clients,
  a single small VPS with the Docker Compose above is cheaper and simpler.

---

## 10. Handover checklist

- [ ] `.env` secrets all changed from defaults
- [ ] `SITE_ADDRESS` = client domain, HTTPS working (padlock in browser)
- [ ] `SEED_ON_STARTUP=false` (no demo leads in production)
- [ ] Branding set in Settings (logo, name, colors, contact)
- [ ] Client's leads imported
- [ ] Admin + team accounts created; you gave the client their login
- [ ] Daily backup cron installed
- [ ] Firewall allows only 22, 80, 443

---

## 11. Troubleshooting

| Symptom | Fix |
|---------|-----|
| Site not loading | `docker compose ps` then `docker compose logs web backend` |
| HTTPS not issued | DNS A-record must point to the server; ports 80/443 open; `SITE_ADDRESS` is the bare domain |
| "Database not reachable" in logs | `docker compose logs db`; ensure the `db` container is healthy |
| Forgot admin password | set a new `ADMIN_PASSWORD` for a *new* email in `.env` and redeploy, or reset via **Settings → Team** |
| Reset everything | `docker compose down -v` (⚠️ deletes all data) then `docker compose up -d --build` |

Health check endpoint: `https://your-domain/api/health` → `{"status":"ok"}`.
