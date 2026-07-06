# Lab Setup Guide

Complete walkthrough for building this cybersecurity homelab from scratch.

---

## Prerequisites

- 2x Dell PowerEdge R720 (or equivalent servers with VT-x/VT-d support)
- Gigabit switch with VLAN support (802.1Q)
- Proxmox VE 8.1 ISO
- Sufficient storage (4TB+ recommended)

---

## Phase 1 — Proxmox Cluster Setup

### 1.1 Install Proxmox on Both Nodes

Download Proxmox VE 8.1 from [proxmox.com](https://www.proxmox.com/en/downloads).

Flash to USB and boot each server. During install:
- Node 1: hostname `pve1`, IP `10.0.10.11/24`, gateway `10.0.10.1`
- Node 2: hostname `pve2`, IP `10.0.10.12/24`, gateway `10.0.10.1`

After install, update both nodes:

```bash
apt update && apt upgrade -y
```

### 1.2 Configure Cluster

On Node 1:
```bash
pvecm create homelab-cluster
```

On Node 2:
```bash
pvecm add 10.0.10.11
```

Verify cluster status:
```bash
pvecm status
```

### 1.3 Configure Storage

Set up shared storage or local storage on each node. For RAID 10 with local ZFS:

```bash
# On each node — create ZFS pool across 4 drives
zpool create -f tank mirror /dev/sda /dev/sdb mirror /dev/sdc /dev/sdd

# Add as storage in Proxmox
pvesm add zfspool tank --pool tank --content images,rootdir
```

---

## Phase 2 — Network Setup (pfSense)

### 2.1 Configure Switch VLANs

On your managed switch, create three VLANs:
- VLAN 10 — Management
- VLAN 20 — Attack Network
- VLAN 30 — Target Network

Set the port connected to your Proxmox hosts as a trunk port carrying all VLANs.

### 2.2 Create pfSense VM

In Proxmox:
- **OS:** FreeBSD (pfSense ISO)
- **CPU:** 2 cores
- **RAM:** 4GB
- **NICs:** 2 (WAN + LAN trunk)

### 2.3 Initial pfSense Configuration

On first boot, assign interfaces via the console:
```
WAN → em0 (your external NIC)
LAN → em1 (trunk to Proxmox)
```

Access the web UI at `https://10.0.10.1` and complete the setup wizard.

### 2.4 Create VLAN Interfaces

In pfSense: **Interfaces → VLANs → Add**

| VLAN | Parent | Tag | Description |
|------|--------|-----|-------------|
| VLAN10 | em1 | 10 | Management |
| VLAN20 | em1 | 20 | Attack |
| VLAN30 | em1 | 30 | Target |

Assign each VLAN as an interface and set IP addresses:
- VLAN10: `10.0.10.1/24`
- VLAN20: `10.0.20.1/24`
- VLAN30: `10.0.30.1/24`

---

## Phase 3 — Security Onion

### 3.1 Create Security Onion VM

- **OS:** Ubuntu 22.04 (Security Onion ISO)
- **CPU:** 4 cores
- **RAM:** 16GB
- **Disk:** 500GB
- **NICs:** 2 (management on VLAN10, monitor interface on VLAN30 mirror)

### 3.2 Install Security Onion

Boot the ISO and follow the guided installer. Select **Standalone** node type for a single-node deployment.

```bash
# After OS install, run SO setup
sudo so-setup
```

Choose:
- Installation type: **Standalone**
- Management NIC: `eth0` (VLAN10)
- Monitor NIC: `eth1` (mirror port)
- Set admin credentials

### 3.3 Configure Mirror Port

On your switch, configure a SPAN/mirror port to copy all traffic from the Target VLAN to the Security Onion monitor interface. This allows Security Onion to passively monitor all target network traffic without being in-line.

---

## Phase 4 — ELK Stack

### 4.1 Create ELK VM

- **OS:** Ubuntu 22.04
- **CPU:** 4 cores
- **RAM:** 16GB
- **Disk:** 200GB
- **NIC:** VLAN10 (`10.0.10.20`)

### 4.2 Install Elasticsearch

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt update && sudo apt install elasticsearch kibana logstash -y
```

### 4.3 Configure Elasticsearch

Edit `/etc/elasticsearch/elasticsearch.yml`:

```yaml
cluster.name: homelab
node.name: elk-node-1
network.host: 10.0.10.20
http.port: 9200
discovery.type: single-node
```

```bash
sudo systemctl enable --now elasticsearch kibana logstash
```

Access Kibana at `http://10.0.10.20:5601`

---

## Phase 5 — Kali Linux

### 5.1 Create Kali VM

- **OS:** Kali Linux
- **CPU:** 4 cores
- **RAM:** 8GB
- **Disk:** 100GB
- **NIC:** VLAN20 (`10.0.20.10`)

### 5.2 Post-Install Setup

```bash
sudo apt update && sudo apt full-upgrade -y

# Install additional tools
sudo apt install -y bloodhound neo4j impacket-scripts crackmapexec
```

---

## Phase 6 — Active Directory Lab

### 6.1 Create Windows Server VM

- **OS:** Windows Server 2022
- **CPU:** 4 cores
- **RAM:** 8GB
- **Disk:** 100GB
- **NIC:** VLAN30 (`10.0.30.10`)

### 6.2 Promote to Domain Controller

In PowerShell (run as Administrator):

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

Import-Module ADDSDeployment
Install-ADDSForest `
    -DomainName "homelab.local" `
    -DomainNetbiosName "HOMELAB" `
    -ForestMode "WinThreshold" `
    -DomainMode "WinThreshold" `
    -InstallDns:$true `
    -Force:$true
```

### 6.3 Create Lab Users and Groups

```powershell
# Create OUs
New-ADOrganizationalUnit -Name "Lab Users" -Path "DC=homelab,DC=local"
New-ADOrganizationalUnit -Name "Lab Computers" -Path "DC=homelab,DC=local"

# Create sample users
$users = @("jsmith","bjones","alee","mwilson")
foreach ($user in $users) {
    New-ADUser -Name $user `
        -SamAccountName $user `
        -UserPrincipalName "$user@homelab.local" `
        -Path "OU=Lab Users,DC=homelab,DC=local" `
        -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) `
        -Enabled $true
}
```

---

## Verification Checklist

After completing all phases, verify:

- [ ] Proxmox cluster shows both nodes healthy
- [ ] pfSense routing between VLANs works as expected
- [ ] Kali can reach Target Network (VLAN30)
- [ ] Attack Network cannot reach Management Network (VLAN10)
- [ ] Security Onion seeing traffic from mirror port
- [ ] ELK receiving logs from target systems
- [ ] Windows AD domain functional, users can authenticate
- [ ] Kibana dashboard accessible from Management Network

---

*[← Back to README](../README.md)*
