# Home Lab Infrastructure Documentation

A collection of technical guides and documentation for building and maintaining a secure home lab network with enterprise-grade features.

## Overview

This repository contains step-by-step guides for implementing network segmentation, security tooling, and infrastructure services in a home lab environment. All guides have been personally tested and validated through hands-on implementation.

## Guides

### Network Design & Architecture
- **[Designing Your Network](0.designing-your-network.md)** - Comprehensive guide to planning and implementing a layered security home network with VLANs, pfSense, and monitoring tools

### Firewall Setup
- **[Setting Up a pfSense Firewall](1.setting-up-your-firewall.md)** - A guide from creating a bootable USB drive with pfSense to setting up the firewall basics.

### Network Configuration
- **[VLAN Setup in pfSense](2.vlan-setup-guide.md)** - Step-by-step guide to configuring network segmentation using VLANs in pfSense 2.8

### Security & Filtering
- **[pfBlockerNG Configuration](3.pfBlocker-setup.md)** - Setting up IP blocking and DNS-based filtering for ads, trackers, and malicious domains
- **[Suricata Configuration](5.suricata-setup.md)** - Setting up Suricata IDS/IPS in pfSense for real-time intrusion detection and prevention

### System Administration
- **[Samba File Share Setup](5.samba-file-share.md)** - Configuring SMB file sharing on Ubuntu Server for local network access

## Lab Environment

These guides reflect implementations on the following hardware:

**Firewall/Router:**
- HP EliteDesk 800 G2 (i5-6500, 8GB RAM)
- Running pfSense with pfBlockerNG and Suricata

**Network Utility Server:**
- AMD Ryzen 5 3600 6-Core CPU 3.6ghz, 16GB RAM, 1.5TB HDD + 128GB NVMe, EVGA Nvidia Geforce GTX 1050Ti 4GB VRAM
- Ubuntu Server running Prometheus, Grafana, Wolf GPU Streaming, Excalidraw Containers

**File Server:**
- HP EliteDesk 800 G3 Mini (i5-6500, 8GB RAM)
- Ubuntu Server with Samba, MariaDB

**Network Infrastructure:**
- TP-Link SG108E Managed Switch (VLAN support)
- TP-Link Omada AC-1350 Access Point (VLAN-capable)

## Network Architecture

```
ISP Modem
    |
pfSense Firewall (192.168.1.1)
    |
Managed Switch (VLANs 10, 20, 30, 40)
    |
    ├── VLAN 10: Desktop   (192.168.10.0/24)
    ├── VLAN 20: WiFi      (192.168.20.0/24)
    ├── VLAN 30: Servers   (192.168.30.0/24)
    └── VLAN 40: IoT/Guest (192.168.40.0/24)
```

## Tools & Technologies

**Firewall & Security:**
- pfSense (open-source firewall/router)
- pfBlockerNG (IP/DNS filtering)
- Suricata (IDS/IPS)
- Wazuh (SIEM)

**Infrastructure:**
- Ubuntu Server (file server, services)
- Samba (SMB file sharing)
- MariaDB (database)

**Monitoring:**
- Netdata (real-time monitoring)
- Prometheus (metrics collection)
- Grafana (visualization)

**Networking:**
- VLANs (network segmentation)
- Managed switches (VLAN support)
- VLAN-capable access points

## Use Cases

These guides support:
- **Home Lab Security** - Implementing enterprise security practices at home
- **Network Segmentation** - Isolating devices and services for security
- **Self-Hosted Services** - Running local file storage, media servers, etc.
- **Privacy-Focused Networking** - Blocking trackers and ads at the network level
- **Learning Infrastructure** - Hands-on practice with real-world networking and security tools

## Development Approach

All documentation was developed through hands-on experimentation in a real home lab environment. AI assistance (Claude/ChatGPT) was used to accelerate research and troubleshooting, but all configurations were personally tested and validated before documentation.

## Philosophy

The goal is to document practical, working implementations rather than theoretical concepts. Each guide includes:
- Real hardware specifications and costs
- Step-by-step commands that work
- Troubleshooting sections based on actual issues encountered
- Security considerations and best practices
- Notes on what worked, what didn't, and why

## Future Documentation

Planned additions:
- Wazuh SIEM deployment and rule tuning
- WireGuard VPN setup for remote access
- Container orchestration with Docker
- Automated backups with rsync/rclone
- Let's Encrypt SSL certificates for internal services
- Linux disk formatting and drive management

## Contributing

This is a personal documentation repository, but corrections and suggestions are welcome. If you notice errors or have improvements, feel free to open an issue.

## Author

**Greg Diny**

10 years experience making award winning alcohol. 

## License

MIT License - Feel free to use these guides for your own projects.

---

*Last Updated: February 2026*
