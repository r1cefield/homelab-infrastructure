# homelab-infrastructure


# 🏗️ Overview
This repository documents my production-grade homelab infrastructure which implements enterprise networking, security, and automation practices. 
The setup demostrastes my practices in:

- **Network Engineering**: VLAN segmentation with pfSense routing
- **Virtualization**: Proxmox hypervisor with VM management
- **Container Orchestration**: Docker with custom networking
- **Security**: Zero-trust networking with Tailscale WireGuard VPN
- **Automation**: Infrastructure as Code and CI/CD principles
- **Service Management**: Self-hosted applications with reverse proxy

## 🛠️ Technology Stack

### Core Infrastructure
- **Hypervisor**: Proxmox VE on PN52 Ubuntu Server
- **Network OS**: pfSense (virtualized router/firewall)
- **Container Runtime**: Docker with custom bridge networks
- **Remote Access**: Tailscale subnet routing with WireGuard

### Network Architecture
- **Management Network**: VLAN 100 (192.168.100.0/24)
- **Storage Network**: VLAN 200 (10.0.0.0/24)  
- **Docker Network**: Custom bridge (172.20.0.0/24)
- **Physical Switch**: Cisco 2960-L with VLAN configuration

### Self-Hosted Services
- **Reverse Proxy**: Nginx Proxy Manager with automated SSL
- **Container Management**: Portainer for Docker orchestration
- **Version Control**: GitLab CE (this instance!)
- **Network Storage**: Synology NAS with automated backups

## 🚀 Key Achievements

- ✅ **Secure Remote Access**: Tailscale subnet routing from anywhere
- ✅ **Zero-Downtime Services**: Docker health checks and restart policies
- ✅ **Automated SSL**: Wildcard certificates with automatic renewal
- ✅ **Network Segmentation**: Proper VLAN isolation with inter-VLAN routing
- ✅ **Infrastructure Documentation**: Living documentation with architecture diagrams
- ✅ **Backup Strategy**: Automated backups across VLANs

## 📊 Network Topology

```
Internet
    ↓
[Main Router] ←→ [Cisco Switch]
    ↓                ↓
VLAN 100         VLAN 200
    ↓                ↓
[Proxmox] ←→ [Synology NAS]
    ↓
[pfSense VM] ←→ [Docker Services]
    ↓
[Tailscale Gateway]
```

## 📋 Skills Demonstrated

### Networking & Security
- VLAN configuration and inter-VLAN routing
- Firewall rule design and implementation
- VPN implementation with subnet advertisement
- SSL/TLS certificate management
- Network troubleshooting and optimization

### Systems & Virtualization
- Linux server administration (Ubuntu)
- Hypervisor management (Proxmox)
- Virtual machine deployment and configuration
- Container orchestration with Docker
- Service discovery and reverse proxy setup

### DevOps & Automation
- Infrastructure documentation practices
- Version control with self-hosted GitLab
- Container deployment strategies
- Backup automation and disaster recovery
- Monitoring and health check implementation

## 🔍 Repository Structure

- `docs/` - Comprehensive infrastructure documentation
- `configs/` - Sanitized configuration files and templates
- `scripts/` - Automation scripts and deployment tools
- `diagrams/` - Network topology and architecture diagrams

## 📈 Current Status

- ✅ Core infrastructure operational
- ✅ Remote access via Tailscale configured
- ✅ Docker services deployed and accessible
- 🔄 Documentation in progress
- 📋 Monitoring and automation planned

## 🛠️ Quick Access

- **GitLab**: https://gitlab.homelab.local
- **Portainer**: https://portainer.homelab.local  
- **Nginx Proxy Manager**: https://nginx.homelab.local

---

> **Note**: This homelab demonstrates enterprise-grade practices adapted for a home environment. All configurations follow security best practices and are documented for educational and professional development purposes.

**Last Updated**: July 2025
```