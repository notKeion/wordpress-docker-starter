# WordPress + Caddy (Auto-HTTPS) — Ultra-Light Starter

Single `docker-compose.yml` that runs **WordPress + MariaDB** behind **Caddy** with **automatic HTTPS** via Let’s Encrypt.  
Tuned to stay around **≤512 MB RAM**. Fork it and ship simple brochure/blog sites fast.

---

## What’s Inside

- 🔐 **Auto-HTTPS** (Caddy + Let’s Encrypt) — zero config beyond domain + email
- 🐳 **Single compose file** — no extra proxy files
- 💾 **Bind mounts**: `./wp` (site files) and `./dbdata` (database)
- ⚙️ **Memory-trimmed** MariaDB & WP caps for small droplets

---

## Repo Layout

- `docker-compose.yml` — Caddy + WordPress + MariaDB
- `.env.example` — copy to `.env` and set secrets
- `wp/` — WordPress files (commit with `.gitkeep`)
- `dbdata/` — MariaDB data (commit with `.gitkeep`)

---

## Quick Start (Local Development)
1. Copy env and set values:
    ```bash
    cp .env.example .env
    ```
2. For production, set:
    ```bash
    DOMAIN=yourdomain.com
    LETSENCRYPT_EMAIL=you@yourdomain.com
    ```
3. Bring it up:
    ```bash
    docker compose up -d
    ```
4. Open Site:
	- **Production**: https://yourdomain.com (cert auto-provisioned)
	- **Local**: https://localhost (browser may warn for local cert)
5. Stop / Remove:
    ```bash
    docker compose down
    # Remove all data (careful!):
    # docker compose down -v && rm -rf wp/* dbdata/*
    ```
---

## Deploy to DigitalOcean (Fast Path)
1. SSH as root
    ```bash
    ssh root@YOUR_DROPLET_IP
    ```
2. Install Docker Quickly
    ```bash
    curl -fsSL https://get.docker.com | sh
    ```
3. Clone your fork and prep
    ```bash
    mkdir -p /opt && cd /opt
    git clone https://github.com/notKeion/wordpress-docker-starter.git wordpress-docker-starter
    cd wordpress-docker-starter

    cp .env.example .env
    # Set a real domain + email and strong DB passwords
    nano .env

    mkdir -p wp dbdata && touch wp/.gitkeep dbdata/.gitkeep
    ```
    Generate a secure password (Optionally, but recommended):
    ```bash
    # Strong hex password (64 chars ≈ 256-bit), great for .env values
    openssl rand -hex 32
    ```
4. Open ports and start:
    ```bash
    # If you use UFW:
    ufw allow 80; ufw allow 443; ufw --force enable

    docker compose up -d
    ```
5. Updating Later:
    ```bash
    cd /opt/wordpress-caddy-starter
    git pull --ff-only
    docker compose pull
    docker compose up -d --remove-orphans
    ```

## Memory Tuning (if you hit OOM)
- In docker-compose.yml:
    - Raise wordpress.mem_limit to 320m if WP admin/plugin installs restart PHP.
	- Raise db --innodb_buffer_pool_size to 96M (or 128M) for larger datasets.
- Avoid heavy plugins; keep active plugins lean.


## Common Commands
```bash
docker compose ps                 # show services
docker compose logs -f wordpress  # tail WP logs
docker compose logs -f db         # tail DB logs
docker compose exec wordpress bash  # shell into WP container
```

## Database Backups
```bash
docker compose exec db sh -c \
'mysqldump -u"$MARIADB_USER" -p"$MARIADB_PASSWORD" "$MARIADB_DATABASE"' \
> db_backup.sql
```
**Files**: archive wp/ (uploads, themes, plugins).

