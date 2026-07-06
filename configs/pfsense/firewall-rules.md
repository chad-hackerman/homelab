# pfSense Firewall Rules

Zone-based firewall rules implementing the lab's security policy.

---

## Rule Philosophy

Rules follow a **default-deny** approach — all traffic is blocked unless explicitly permitted. Rules are evaluated top-to-bottom; first match wins.

---

## Management Network (VLAN10) Rules

Management can reach everything — admins need full access to manage all zones.

| # | Action | Protocol | Source | Destination | Port | Description |
|---|--------|----------|--------|-------------|------|-------------|
| 1 | Allow | Any | 10.0.10.0/24 | Any | Any | Management full access |
| 2 | Block | Any | Any | Any | Any | Default deny |

---

## Attack Network (VLAN20) Rules

Kali can reach the Target Network and internet, but not Management.

| # | Action | Protocol | Source | Destination | Port | Description |
|---|--------|----------|--------|-------------|------|-------------|
| 1 | Block | Any | 10.0.20.0/24 | 10.0.10.0/24 | Any | Block access to Management |
| 2 | Allow | Any | 10.0.20.0/24 | 10.0.30.0/24 | Any | Allow access to Target |
| 3 | Allow | TCP/UDP | 10.0.20.0/24 | Any | 80,443,53 | Internet access |
| 4 | Block | Any | Any | Any | Any | Default deny |

---

## Target Network (VLAN30) Rules

Target VMs can reach the internet for updates only. Cannot reach Management or Attack zones.

| # | Action | Protocol | Source | Destination | Port | Description |
|---|--------|----------|--------|-------------|------|-------------|
| 1 | Block | Any | 10.0.30.0/24 | 10.0.10.0/24 | Any | Block access to Management |
| 2 | Block | Any | 10.0.30.0/24 | 10.0.20.0/24 | Any | Block access to Attack zone |
| 3 | Allow | TCP | 10.0.30.0/24 | Any | 80,443 | Allow web (updates) |
| 4 | Allow | UDP | 10.0.30.0/24 | Any | 53 | Allow DNS |
| 5 | Allow | TCP | 10.0.30.0/24 | 10.0.10.20 | 5044 | Allow log forwarding to ELK |
| 6 | Block | Any | Any | Any | Any | Default deny |

---

## pfSense Configuration Export

Key settings from `config.xml` (sensitive values redacted):

```xml
<pfsense>
  <vlans>
    <vlan>
      <if>em1</if>
      <tag>10</tag>
      <descr>Management</descr>
    </vlan>
    <vlan>
      <if>em1</if>
      <tag>20</tag>
      <descr>Attack</descr>
    </vlan>
    <vlan>
      <if>em1</if>
      <tag>30</tag>
      <descr>Target</descr>
    </vlan>
  </vlans>

  <interfaces>
    <vlan10>
      <ipaddr>10.0.10.1</ipaddr>
      <subnet>24</subnet>
      <descr>Management</descr>
    </vlan10>
    <vlan20>
      <ipaddr>10.0.20.1</ipaddr>
      <subnet>24</subnet>
      <descr>Attack</descr>
    </vlan20>
    <vlan30>
      <ipaddr>10.0.30.1</ipaddr>
      <subnet>24</subnet>
      <descr>Target</descr>
    </vlan30>
  </interfaces>

  <dhcpd>
    <vlan10>
      <range>
        <from>10.0.10.100</from>
        <to>10.0.10.200</to>
      </range>
    </vlan10>
    <vlan20>
      <range>
        <from>10.0.20.100</from>
        <to>10.0.20.200</to>
      </range>
    </vlan20>
    <vlan30>
      <range>
        <from>10.0.30.100</from>
        <to>10.0.30.200</to>
      </range>
    </vlan30>
  </dhcpd>
</pfsense>
```

---

*[← Back to README](https://github.com/chad-hackerman/homelab)*
