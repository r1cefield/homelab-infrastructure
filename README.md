# My Homelab Setup

This repository contains the documentation and configuration of my production-grade homelab infrastructure. It is designed with a focus on enterprise-level networking, security, and automation best practices.


# Repository  Structure
- **Apps** - List of all apps and services I currently use
- **Media Server** - Jellyfin, *arr stack, and more
- **Server Monitoring** - Graphs and Visualizations for Proxmox and more
- **Storage** - Current storage and backup solution
- **Service Management**: Self-hosted applications with reverse proxy

## Hardware
### Servers and NAS
**Main Rig (Proxmox)**

This machine runs my Proxmox Server and serves as the central node for media storage, automation services (including the Arr stack), system monitoring, and other infrastructure components.

- AMD EPYC 7532
- 96GB DDR4 3200MHZ ECC Server RAM
- NVIDIA 3060 12GB 
- NVIDIA 1050 4GB
- Intel 670p 512GB (Boot Drive)
- SK Hynix Gold S31 1TB (Spare)
- 2x Samsung 860 EVO 500GB (Dedicated VM's)

**ASUS PN52**

This bare-metal Linux server acts as the primary gateway for both external and internal HTTP/HTTPS traffic. It utilizes NGINX as a reverse proxy to manage access to self-hosted applications and services, and runs Uptime Kuma for real-time availability monitoring and alerting.
- AMD 5600H
- 8GB DDR4 3200MHZ SODIMM RAM
- Silicon Power SATA SSD 512GB

**Synology DS920+**

This NAS is primarily used for file storage and for backing up both the Proxmox Server and the ASUS PN52 system.

- Intel Celeron J4125 
- 4GB DDR4
- x2 Seagate Barracuda 4TB
- x1 Seagate Barracuda 3TB
- x1 Western Digital Blue 3TB

## Network

- [Cisco Catalyst 2960L-24TS-LL Switch](https://www.cisco.com/c/en/us/support/switches/catalyst-2960-l-series-switches/series.html)

- [NETGEAR Orbi Pro WiFi 6 Mini Mesh](https://www.netgear.com/au/business/wifi/mesh/sxk30/)

## Network Diagram (Work in Progress)
<img src="Homelab.png">