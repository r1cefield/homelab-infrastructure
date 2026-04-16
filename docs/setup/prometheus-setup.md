# Prometheus - Docker Setup Guide

## Overview

Prometheus is an open-source monitoring and alerting toolkit that collects metrics from configured targets by scraping HTTP endpoints at set intervals.

In this setup, Prometheus runs as a Docker container with persistent data stored on a bind-mounted host directory.

**What this guide covers**
- Creating the Prometheus config file
- Deploying Prometheus with Docker Compose
- Adding scrape targets across VLANs

## Prerequisites

| Requirement | Notes |
|-------------|-------|
| Docker + Docker Compose v2 | https://docs.docker.com/engine/install/ |
| Grafana already deployed | Prometheus feeds data into Grafana |
| Storage path available | e.g. `/data/monitoring/` |
| Targets reachable | Firewall rules and routes in place for cross-VLAN targets |

## Directory Structure
```
/your/storage/path/monitoring/
└── prometheus/
    └── prometheus.yml    ← Scrape configuration

~/docker/
└── docker-compose.yml
```

## Step 1 - Create Directories

``` bash
mkdir -p /your/storage/path/monitoring/prometheus
```

## Step 2 - Create prometheus.yml

```bash
sudo nano /your/storage/path/monitoring/prometheus/prometheus.yml
```
Inside of prometheus.yml
```yaml
global:
  scrape_interval: 15s      # How often to scrape targets
  evaluation_interval: 15s  # How often to evaluate alert rules

scrape_configs:
  - job_name: "prometheus"  # Prometheus scraping itself
    static_configs:
      - targets: ["localhost:9090"]
```
**Field breakdown**
| Field | Purpose | Doc |
|-------|---------|-----|
| `scrape_interval` | How often Prometheus pulls metrics from targets | [global config](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#global) |
| `evaluation_interval` | How often alert rules are evaluated | [global config](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#global) |
| `job_name` | Label assigned to metrics from this target | [scrape_config](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#scrape_config) |
| `targets` | List of `host:port` endpoints to scrape | [scrape_config](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#scrape_config) |

> **Note:** The config file is owned by UID `65534` (nobody) after deployment.
> Always edit with `sudo nano` or temporarily change ownership to edit,
> then restore with `sudo chown -R 65534:65534 /your/storage/path/monitoring/prometheus`

## Step 3 - Add to docker-compose.yml

Add this to your docker-compose file.

```yaml
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    restart: unless-stopped
    cpus: "1.0"
    mem_limit: 512m
    ports:
      - "9090:9090"
    volumes:
      - /your/storage/path/monitoring/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - /your/storage/path/monitoring/prometheus:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.enable-lifecycle'
```
**Field breakdown:**

| Field | Purpose | Doc |
|-------|---------|-----|
| `image` | Official Prometheus image | [installation](https://prometheus.io/docs/prometheus/latest/installation/#using-docker) |
| `restart` | Auto-restart on crash or reboot | [restart](https://docs.docker.com/reference/compose-file/services/#restart) |
| `cpus` / `mem_limit` | Resource limits | [mem_limit](https://docs.docker.com/reference/compose-file/services/#mem_limit) |
| `volumes` | Bind mounts config and data directory | [volumes](https://docs.docker.com/reference/compose-file/services/#volumes) |
| `--config.file` | Points Prometheus to the config file | [installation](https://prometheus.io/docs/prometheus/latest/installation/#using-docker) |
| `--storage.tsdb.path` | Where to store metrics on disk | [storage](https://prometheus.io/docs/prometheus/latest/storage/) |
| `--web.enable-lifecycle` | Enables config reload without restart | [management API](https://prometheus.io/docs/prometheus/latest/management_api/) |

> **Note:** The config file bind mount must be listed BEFORE the data directory mount, otherwise Prometheus cannot find `prometheus.yml`.

## Step 4 - Set Permissions

Prometheus runs as UID `65534` (nobody) inside the container.

```bash
sudo chown -R 65534:65534 /your/storage/path/monitoring/prometheus
```

Verify:

```bash
ls -la /your/storage/path/monitoring/ | grep prometheus
```

Expected output:

```
drwxr-xr-x  2 65534 65534 ... prometheus
```
## Step 5 - Deploy

``` bash 
cd ~/docker
docker compose up -d prometheus
```

## Step 6 - Verify
```bash
docker compose ps prometheus
docker compose logs prometheus --tail 20 
```

> **Note:** `--tail 20` limits output to the last 20 lines, useful for checking startup errors without scrolling through the full log.

## Step 7 - Access Prometheus UI

Open your browser and go to:

`http://<your-vm-ip>:9090` 

> **Note:** Replace `<your-vm-ip>` with the actual IP of your monitoring VM

After accessing to Prometheus. Go to **Status → Targets** — `prometheus` job should show **UP**.


## Step 8 — Add Scrape Targets

Each new device to monitor requires:
1. Node Exporter installed and running on the target
2. A new `job_name` entry in `prometheus.yml`
3. A config reload

### Add a Target
``` bash
sudo nano /your/storage/path/monitoring/prometheus/prometheus.yml
```

Inside `prometheus.yml` add a new job under `scrape_configs`:

```yaml
# Same VLAN — straightforward
scrape_configs:
  - job_name: "<hostname>"
    static_configs:
      - targets: ["<same-vlan-ip>:9100"]

# Different VLAN — requires static route + firewall rule
  - job_name: "<hostname>"
    static_configs:
      - targets: ["<different-vlan-ip>:9100"]
```

### Reload config without restarting

```bash
curl -X POST http://localhost:9090/-/reload
# Then check http://<your-vm-ip>:9090/targets
```
### Cross-VLAN Targets

If your Prometheus host and target are on different VLANs, packets will
never reach the target without two additional steps.

**Why it happens:**
Prometheus host (VLAN A) → tries to reach target (VLAN B) → sends packet
to default gateway → default gateway has no route to VLAN B subnet →
packet dropped — target unreachable

**Fix 1. Add a static route on the Prometheus host.**
```bash
# Temporary — lost on reboot
sudo ip route add <target-subnet> via <router-ip>

# Verify
ip route show | grep <target-subnet>
```

Make it permanent via netplan:

```bash
# Find your netplan file first
ls /etc/netplan/

# Then edit whichever file is listed
sudo nano /etc/netplan/<your-file>.yaml
```
**Note:** The netplan filename varies depending on how your OS was installed. Use ls /etc/netplan/ to find the correct filename before editing.

```yaml
network:
  version: 2
  ethernets:
    ens18:
      dhcp4: true
      routes:
        - to: <target-subnet>      # e.g. 10.0.0.0/24
          via: <router-ip>         # e.g. 192.168.100.188 (pfSense WAN IP)
```
**Note:** ens18 is the network interface name on this setup. Replace it with your actual interface name. Run ip link show to find yours.
```bash
sudo netplan apply

# Verify route persisted
ip route show | grep <target-subnet>
```
> **Note:** The `via` IP must be your router's IP on the **same subnet**
> as the Prometheus host — not the router's IP on the target subnet.

**Fix 2 - Allow port 9100 through your firewall**

This step depends on your firewall/router. The goal is to allow TCP traffic from your prometheus host to the target on port 9100.

*If using pfSense:*

- Go to **Firewall → Rules → \<target VLAN interface\>**
- Add a `Pass` rule:
  - Source: `<prometheus-host-ip>`
  - Destination: `<target-ip>`
  - Destination Port: `9100`
- Place this rule **above** any existing block rules
- Save and Apply Changes

*If using UFW on the target:*
```bash
sudo ufw allow from <prometheus-host-ip> to any port 9100
sudo ufw status | grep 9100
```

*If using iptables:*
```bash
sudo iptables -A INPUT -p tcp -s <prometheus-host-ip> --dport 9100 -j ACCEPT
```

***Note:*** Always restrict port 9100 to only the Prometheus host IP, never open it to `0.0.0.0/0`.

**Verify end to end:**
```bash
curl http://<target-ip>:9100/metrics | head -5
```

## Current Scrape Targets

| Job | Target | Notes |
|-----|--------|-------|
| prometheus | `localhost:9090` | Prometheus scraping itself |
| node-exporter | `<same-vlan-ip>:9100` | Monitoring VM |
| proxmox | `<same-vlan-ip>:9100` | Proxmox host |
| media-vm | `<different-vlan-ip>:9100` | Requires static route + firewall rule |
| pn52 | `<same-vlan-ip>:9100` | Services VM |

> **Note:** Add a new row each time you add a scrape target.
> Mark cross-VLAN targets in the Notes column for reference.


## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| `permission denied` on `/prometheus/queries.active` | Host directory owned by root, not UID 65534 | `sudo chown -R 65534:65534 /your/storage/path/monitoring/prometheus` |
| `prometheus.yml is a directory` | Config path was created as a directory instead of a file | `sudo rm -rf /path/prometheus.yml` then recreate as a file |
| Target shows `UNKNOWN` | First scrape hasn't happened yet | Wait 15 seconds and refresh |
| Target shows `DOWN - network is unreachable` | No route to target subnet | Add static route via router IP |
| Target shows `DOWN - context deadline exceeded` | Firewall blocking port 9100 | Add UFW rule or firewall rule on router |
| Can't edit `prometheus.yml` | File owned by UID 65534 | Use `sudo nano` or temporarily chown to your user |

