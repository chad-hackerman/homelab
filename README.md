# 🏠 Cybersecurity Homelab

![Proxmox](https://img.shields.io/badge/Proxmox-8.1-orange) ![pfSense](https://img.shields.io/badge/pfSense-2.7-blue) ![Security Onion](https://img.shields.io/badge/Security%20Onion-2.4-green) ![Status](https://img.shields.io/badge/Infrastructure-Active-brightgreen) ![Docs](https://img.shields.io/badge/Documentation-Complete-blue)

Complete security testing environment for hands-on practice with enterprise security tools and attack simulation.

---

## 📋 Overview

**Purpose:** An isolated, enterprise-grade security lab for:
- Penetration testing practice
- Security tool evaluation
- Attack simulation and defense
- Certification exam preparation (OSCP, CEH, Security+)

---

## 🖥️ Hardware

| Component | Spec |
|-----------|------|
| Servers | 2x Dell PowerEdge R720 (Proxmox cluster) |
| RAM | 128GB total (64GB per node) |
| Storage | 4TB RAID 10 |
| Networking | Dedicated gigabit switch |

---

## 🌐 Network Architecture

```mermaid
graph TB
    Internet((Internet)) --> WAN

    subgraph pfSense ["🔥 pfSense Firewall"]
        WAN[WAN Interface]
        LAN[LAN Interface]
        VLAN10[VLAN 10 - Management]
        VLAN20[VLAN 20 - Attack Network]
        VLAN30[VLAN 30 - Target Network]
    end

    WAN --> LAN
    LAN --> VLAN10
    LAN --> VLAN20
    LAN --> VLAN30

    subgraph MGMT ["🔒 Management Network (10.0.10.0/24)"]
        PROXMOX1[Proxmox Node 1<br/>10.0.10.11]
        PROXMOX2[Proxmox Node 2<br/>10.0.10.12]
        ELK[ELK Stack<br/>10.0.10.20]
        SO[Security Onion<br/>10.0.10.30]
    end

    subgraph ATTACK ["⚔️ Attack Network (10.0.20.0/24)"]
        KALI[Kali Linux<br/>10.0.20.10]
    end

    subgraph TARGET ["🎯 Target Network (10.0.30.0/24)"]
        WIN[Windows Server 2022<br/>AD DC - 10.0.30.10]
        UBUNTU[Ubuntu Server<br/>10.0.30.20]
        VULN[Vulnerable VMs<br/>10.0.30.50-99]
    end

    VLAN10 --> MGMT
    VLAN20 --> ATTACK
    VLAN30 --> TARGET

    SO -.->|Mirror Port - Traffic Analysis| TARGET
    ELK -.->|Log Ingestion| TARGET
    ELK -.->|Log Ingestion| MGMT
```

---

## 🔒 Security Zones

| Zone | Subnet | Purpose | Access |
|------|--------|---------|--------|
| Management | 10.0.10.0/24 | Admin, monitoring, SIEM | Restricted |
| Attack | 10.0.20.0/24 | Kali Linux, offensive tools | Isolated |
| Target | 10.0.30.0/24 | Vulnerable systems, AD lab | Attack network only |

---

## 🖥️ Virtual Machines

| VM | OS | IP | Role |
|----|----|----|------|
| pfSense | FreeBSD | 10.0.10.1 | Firewall / router |
| Security Onion | Ubuntu | 10.0.10.30 | NSM / IDS |
| ELK Stack | Ubuntu | 10.0.10.20 | SIEM / log analysis |
| Kali Linux | Debian | 10.0.20.10 | Penetration testing |
| Windows Server 2022 | Windows | 10.0.30.10 | Active Directory DC |
| Ubuntu Server | Ubuntu | 10.0.30.20 | Web services |

---

## 📁 Repository Structure

```
homelab/
├── README.md
├── docs/
│   ├── setup-guide.md          # Full lab setup walkthrough
│   ├── network-design.md       # Network design decisions
│   ├── vm-provisioning.md      # VM creation and configuration
│   └── learning-outcomes.md    # Skills and certifications
├── configs/
│   ├── pfsense/                # Firewall rules and VLAN config
│   ├── elk/                    # Logstash pipelines, index templates
│   ├── security-onion/         # SO sensor and alert tuning
│   └── active-directory/       # AD hardening and GPO configs
└── diagrams/
    └── network-topology.md     # Mermaid network diagrams
```

---

## 📚 Documentation

- [Full Setup Guide](./docs/setup-guide.md)
- [Network Design](./docs/network-design.md)
- [VM Provisioning](./docs/vm-provisioning.md)
- [Learning Outcomes](./docs/learning-outcomes.md)

---

## ⚙️ Configs

- [pfSense Firewall Rules](./configs/pfsense/firewall-rules.md)
- [ELK Stack Pipeline](./configs/elk/logstash-pipeline.conf)
- [Security Onion Tuning](./configs/security-onion/sensor-config.md)
- [Active Directory Hardening](./configs/active-directory/hardening.md)

---

## 🎓 Learning Outcomes

- Enterprise firewall configuration and rules
- SIEM deployment and log analysis
- Network segmentation and VLANs
- IDS/IPS configuration and tuning
- Active Directory security hardening

---

*Part of [Chad Hackerman's Portfolio](https://github.com/chad-hackerman/chad-hackerman-portfolio)*
