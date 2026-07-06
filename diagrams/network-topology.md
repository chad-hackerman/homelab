# Network Topology Diagrams

---

## Full Lab Topology

```mermaid
graph TB
    Internet((🌐 Internet))

    Internet --> |WAN| FW

    subgraph FW ["🔥 pfSense 2.7 — 10.0.10.1"]
        FW_LABEL[Firewall / Router\nVLAN Trunk]
    end

    FW --> |VLAN 10| MGMT_NET
    FW --> |VLAN 20| ATTACK_NET
    FW --> |VLAN 30| TARGET_NET

    subgraph MGMT_NET ["🔒 Management Network — 10.0.10.0/24"]
        PVE1[🖥️ Proxmox Node 1\n10.0.10.11]
        PVE2[🖥️ Proxmox Node 2\n10.0.10.12]
        ELK[📊 ELK Stack\n10.0.10.20]
        SO[🔍 Security Onion\n10.0.10.30]
    end

    subgraph ATTACK_NET ["⚔️ Attack Network — 10.0.20.0/24"]
        KALI[🐉 Kali Linux\n10.0.20.10]
    end

    subgraph TARGET_NET ["🎯 Target Network — 10.0.30.0/24"]
        DC[🏢 Windows Server 2022\nAD DC — 10.0.30.10]
        UBUNTU[🐧 Ubuntu Server\n10.0.30.20]
        VULN[💀 Vulnerable VMs\n10.0.30.50-99]
    end

    KALI -->|Attacks| TARGET_NET
    SO -.->|Passive Monitor\nSPAN Port| TARGET_NET
    TARGET_NET -->|Log Forwarding :5044| ELK
    SO -.->|Alerts| ELK
```

---

## Traffic Flow — Attack Simulation

```mermaid
sequenceDiagram
    participant K as Kali Linux
    participant S as Switch/SPAN
    participant T as Target VM
    participant SO as Security Onion
    participant ELK as ELK Stack

    K->>T: 1. Attack traffic (exploit, scan, etc.)
    S-->>SO: 2. SPAN copy of all packets
    T->>ELK: 3. Windows Event Logs (Winlogbeat)
    SO->>ELK: 4. Suricata/Zeek alerts
    Note over ELK: 5. Correlate logs + alerts
    ELK-->>SO: 6. Analyst reviews in Kibana
```

---

## VLAN Segmentation

```mermaid
graph LR
    subgraph switch ["Gigabit Switch"]
        T1[Trunk Port\nAll VLANs]
        A1[Access Port\nVLAN 10]
        A2[Access Port\nVLAN 20]
        A3[Access Port\nVLAN 30]
        SPAN[SPAN/Mirror Port]
    end

    T1 --- PROXMOX[Proxmox Hosts]
    A1 --- ADMIN[Admin Workstation]
    A2 --- NOTHING[Isolated]
    A3 --- NOTHING2[Isolated]
    SPAN --- SO_MON[Security Onion\nMonitor NIC]
```

---

## VM Placement Across Proxmox Cluster

```mermaid
graph TB
    subgraph PVE1 ["Proxmox Node 1 — pve1 (10.0.10.11)"]
        VM100[VM 100: pfSense]
        VM101[VM 101: Security Onion]
        VM103[VM 103: Kali Linux]
    end

    subgraph PVE2 ["Proxmox Node 2 — pve2 (10.0.10.12)"]
        VM102[VM 102: ELK Stack]
        VM104[VM 104: Windows Server 2022]
        VM105[VM 105: Ubuntu Server]
    end

    subgraph STORAGE ["Shared ZFS Storage (tank)"]
        SNAPSHOTS[VM Snapshots]
        TEMPLATES[VM Templates]
    end

    PVE1 <-->|Cluster Sync| PVE2
    PVE1 --> STORAGE
    PVE2 --> STORAGE
```

---

*[← Back to README](../README.md)*
