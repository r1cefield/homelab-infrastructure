# Grafana Setup 

## Overview
Grafana is an open-source visualization and dashboarding tool. In this setup, Grafana runs as a Docker container with persistent data stored on a mounted host directory. This approach gives you full control over where data lives, making it easy to back up, move, or migrate.

**What this guide covers:**

- Creating a clean directory structure for persistent data
- Deploying Grafana with Docker Compose

## Prerequisites

| Requirement | Notes |
|-------------|-------|
| Docker Engine | https://docs.docker.com/engine/install/ |
| Docker Compose v2 | Comes bundled with Docker Desktop; `docker compose` not `docker-compose` |
| A dedicated storage path | e.g. a secondary HDD mounted at `/data` to avoid SSD wear |

> ***Note***: You may use SSD for grafana as well, currently using a WD Black HDD for my setup since that's all I have at the moment

## Directory Structure

```
/your/storage/path/
└── monitoring/
    └── grafana/
        └── provisioning/
            ├── dashboards/
            └── datasources/

~/directory/                      ← Or wherever you keep compose files
├── .env                          ← Credentials (gitignored)
├── .gitignore
└── docker-compose.yml
```

## Step 1 - Create Directories

Create the persistent data directory for Grafana on your storage path.

```bash
mkdir -p /your/storage/path/monitoring/grafana
```

## Step 2 - Create the `.gitignore`

Before writing any credentials, add `.env` to `.gitignore` so it is never accidentally commited to version control.

```bash
echo ".env" >> ~/docker/.gitignore
```

> ***Note***: Do this before Step 3. Order matters - you want the ignore rule in place before the file exists.

## Step 3 - Create `.env`

The `.env` file stores credentials and environment variables separately from the compose file. Docker Compose loads this file automatically when `env_file` is specified

```bash
nano ~/docker/.env
```

```env
# Grafana
# Full variable reference: https://github.com/grafana/grafana/blob/main/conf/defaults.ini

GF_SECURITY_ADMIN_USER=<your_admin_username>
GF_SECURITY_ADMIN_PASSWORD=<your_admin_password>
GF_USERS_ALLOW_SIGN_UP=false
TZ=<your_timezone>
```

| Variable | Purpose | Doc |
|----------|---------|-----|
| `GF_SECURITY_ADMIN_USER` | Sets the admin username on first run | [defaults.ini](https://github.com/grafana/grafana/blob/main/conf/defaults.ini) |
| `GF_SECURITY_ADMIN_PASSWORD` | Sets the admin password on first run | [defaults.ini](https://github.com/grafana/grafana/blob/main/conf/defaults.ini) |
| `GF_USERS_ALLOW_SIGN_UP` | Disables public self-registration | [defaults.ini](https://github.com/grafana/grafana/blob/main/conf/defaults.ini) |
| `TZ` | Sets container timezone | System env var |

>> ***Note:*** All Grafana config options follow the pattern of `GF_<SECTION>_<KEY>`.

## step 4 -Create docker-compose.yml


## Step 5 - Set Permissions

Grafana run as UID `472` inside the container. If the host directory is owned by root, Grafana cannot write to it and will fail to start.

``` bash
sudo chown -R 472:472 /your/storage/path/monitoring/grafana
ls -la /your/storage/path/monitoring/ | grep grafana
# Expected output
drwxr-xr-x  2 472 472 ... grafana
```

**What this does:**
- `chown` — changes directory ownership
- `-R` — applies recursively to all subdirectories
- `472:472` — sets both user and group to Grafana's internal UID/GID

## Step 6 - Deploy

```bash
docker compose up <container name>
```

## Step 7 - Verify

Check the container is running:
```bash
docker compose ps
```

## 


## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| `GF_PATHS_DATA is not writable` | Host directory owned by root, not UID 472 | `sudo chown -R 472:472 /your/storage/path/monitoring/grafana` then redeploy |
| `mkdir: can't create directory '/var/lib/grafana/plugins': Permission denied` | Same as above — permissions issue | Same fix as above |
| Can't log in with `.env` credentials | `grafana.db` already exists from a previous run — env vars only apply on first start | **Option A (keep data):** `docker exec -it grafana grafana-cli admin reset-admin-password <newpass>` / **Option B (fresh start):** `docker compose down && sudo rm -rf /your/storage/path/monitoring/grafana/* && sudo chown -R 472:472 /your/storage/path/monitoring/grafana && docker compose up -d grafana` |
| Container shows `"Mounts": []` | Named volume defined but no `volumes:` block at bottom of compose file | Switch to bind mount syntax: `- /host/path:/var/lib/grafana` |
| Container exits immediately | Check logs with `docker compose logs grafana` | Look for permission errors or config issues in the output |
| Port 3000 not accessible | Port already in use on host | Change host port: `- "3001:3000"` or stop the conflicting service |