# 🚀 ArrowPulse — Installation Guide

ArrowPulse is a read-only, AI-powered deliverability intelligence platform for [GreenArrow MTA](https://www.greenarrowemail.com/). It connects to GreenArrow's PostgreSQL database (never writes to it), analyzes deferrals, bounces, ISP behavior, and SMTP reply patterns, and turns them into actionable intelligence.

As of **v2.0.1**, the supported installation method is **Docker Compose** — fully configured via a single `.env` file, no setup wizard, no manual SQL. From a blank server to a running system in about 5 minutes.

---

## 📋 Requirements

- Linux server: 2 vCPU / 4 GB RAM / 30 GB disk minimum (e.g. AWS `t3.medium`, Ubuntu 22.04+)
- Docker Engine 24+ with the Docker Compose plugin
- Network access from this server to your GreenArrow PostgreSQL database (port 5432)
- A valid ArrowPulse license key
- Outbound HTTPS access (license activation, optional AI features)

GreenArrow PostgreSQL versions from **8.3** upward are supported.

---

## ⚡ Quick Start

### 1. Install Docker (if not present)

```bash
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER && newgrp docker
```

### 2. Download and extract ArrowPulse

```bash
curl -sL https://github.com/DevHeaven/arrowpulse_releases/releases/download/v2.0.1/arrowpulse-2.0.1.zip -o arrowpulse.zip
unzip -q arrowpulse.zip
sudo mv arrowpulse-2.0.1 /opt/arrowpulse
cd /opt/arrowpulse
```

### 3. Configure

```bash
cp .env.docker.example .env
nano .env
```

Fill in the **7 required values** — everything else has sensible defaults:

| Variable | Description |
|----------|-------------|
| `GREENARROW_DB_HOST` | GreenArrow PostgreSQL host/IP (not `localhost` — use a routable address) |
| `GREENARROW_DB_USERNAME` | Database user — **read-only recommended** |
| `GREENARROW_DB_PASSWORD` | Database password |
| `ARROWPULSE_DB_PASSWORD` | Any strong password for ArrowPulse's own bundled database |
| `ADMIN_EMAIL` | First admin account email |
| `ADMIN_PASSWORD` | First admin account password |
| `ARROWPULSE_LICENSE_KEY` | Your license key |

Also set `APP_URL` to the address you'll browse to (e.g. `http://your-server-ip:8080`).

### 4. Allow the connection on your GreenArrow server (one-time)

Add this server's IP to `pg_hba.conf` on the GreenArrow PostgreSQL host with a read-only user, then reload PostgreSQL.

### 5. Start

```bash
docker compose up -d --build
docker compose logs -f app     # watch initialization (migrations, admin user, license)
```

### 6. Verify and log in

```bash
curl localhost:8080/health     # "status":"healthy"
docker compose ps              # all services healthy
```

Open `http://your-server-ip:8080` and log in with your `ADMIN_EMAIL` / `ADMIN_PASSWORD`.

> **Back up your encryption key** after first boot:
> `docker compose exec app cat storage/app/private/app_key` — paste it into `.env` as `APP_KEY=...`.
> This key encrypts stored credentials; keep it with your backups.

---

## 🏗️ What You Get

Six containers, health-checked and boot-ordered automatically:

| Service | Purpose |
|---------|---------|
| `app` | Laravel application (PHP-FPM) |
| `nginx` | Web server |
| `horizon` | Queue workers (domain stats, engagement, AI processing) |
| `scheduler` | Scheduled tasks (stats warehouse, license checks) |
| `redis` | Queues, cache, sessions |
| `arrowpulse-db` | ArrowPulse's own PostgreSQL (users, settings, analytics) |

Your GreenArrow PostgreSQL stays **external** — ArrowPulse only reads from it.

On every container start, the idempotent `arrowpulse:init` command runs migrations, seeds the admin user (first boot only), activates the license (once), and syncs your `.env` into the application — so upgrades and restarts never need manual steps.

Settings controlled by environment variables are **locked in the admin UI** with a clear banner — your `.env` is the single source of truth.

### Health endpoint

`GET /health` returns JSON with connectivity status:

- `healthy` (HTTP 200) — everything connected
- `degraded` (HTTP 200) — GreenArrow DB unreachable; ArrowPulse runs and the dashboard shows a clear connection error
- `unhealthy` (HTTP 503) — ArrowPulse's own database is down

---

## 🔄 Upgrading

```bash
# 1. Download and extract the new release
curl -sL https://github.com/DevHeaven/arrowpulse_releases/releases/download/vX.Y.Z/arrowpulse-X.Y.Z.zip -o arrowpulse-new.zip
unzip -q arrowpulse-new.zip

# 2. Carry over your configuration
cp /opt/arrowpulse/.env arrowpulse-X.Y.Z/.env

# 3. Swap directories
sudo mv /opt/arrowpulse /opt/arrowpulse-previous
sudo mv arrowpulse-X.Y.Z /opt/arrowpulse

# 4. Rebuild and restart — migrations run automatically
cd /opt/arrowpulse
docker compose up -d --build

# 5. Verify, then clean up
curl localhost:8080/health
sudo rm -rf /opt/arrowpulse-previous
```

Your data lives in Docker volumes (`pgdata`, `app-storage`, `redis-data`) and survives upgrades and container recreation.

**Rollback:** swap the directories back and `docker compose up -d --build`.

---

## 💾 Backups

```bash
# Database
docker compose exec arrowpulse-db pg_dump -U arrowpulse arrowpulse > backup.sql

# Encryption key (critical — encrypted settings are unreadable without it)
docker compose exec app cat storage/app/private/app_key
```

---

## 🐛 Troubleshooting

**Container won't start** — `docker compose logs app`. Most common: `ARROWPULSE_DB_PASSWORD` not set.

**Can't connect to GreenArrow** — the dashboard will show a clear "GreenArrow database unreachable" banner and `/health` reports `degraded`. Check:
- `GREENARROW_DB_HOST` is not `localhost`/`127.0.0.1` (that resolves *inside* the container)
- `pg_hba.conf` on the GreenArrow server allows this host
- Test from inside the container:
  `docker compose exec app php artisan tinker --execute="DB::connection('greenarrow')->getPdo(); echo 'OK';"`

**License activation fails** — verify the key, and check outbound HTTPS to `https://licensing.arrowpulse.io`. A previously activated system keeps running through a 48-hour grace window if the license server is unreachable.

**Reset everything** (⚠️ deletes all ArrowPulse data):
`docker compose down -v && docker compose up -d --build`

The full deployment guide (`README.DOCKER.md`) is included in the release, covering external PostgreSQL, log shipping, network security, and more. A manual (non-Docker) installation reference remains available as `README.PRODUCTION.md`, but Docker is the recommended and supported path.

---

## 🔒 Production Checklist

- ✅ Only expose port 8080 (or put a TLS-terminating proxy in front and expose 443)
- ✅ Strong `ADMIN_PASSWORD` and `ARROWPULSE_DB_PASSWORD`
- ✅ Read-only PostgreSQL user for the GreenArrow connection
- ✅ `APP_KEY` backed up
- ✅ Regular `pg_dump` backups scheduled

---

## 📞 Support

1. Check `docker compose logs app` for detailed errors
2. Review the troubleshooting sections above and in `README.DOCKER.md`
3. Contact **support@teneo.be** with the error output, your environment details, and steps to reproduce

---

*Last Updated: July 2026 — ArrowPulse v2.0.1*
