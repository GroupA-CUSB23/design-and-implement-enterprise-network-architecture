# សង្ខេបគម្រោង
## ស្ថាបត្យកម្មបណ្តាញធនាគារកម្ពុជា ដោយប្រើគំរូ TCP/IP

---

## ១. សេណារីយ៉ូ (Scenario)

ក្រុមវិស្វករប្រព័ន្ធត្រូវបានជួលដោយ **ធនាគារពាណិជ្ជកម្មនៅកម្ពុជា** ដើម្បីរចនា និងអនុវត្តស្ថាបត្យកម្មបណ្តាញសហគ្រាស។

### 🏢 ទីតាំង
| ទីតាំង | បុគ្គលិក | Devices |
|--------|----------|---------|
| **Head Office** (ភ្នំពេញ) | 300 នាក់ | ~700 |
| **សាខាធំ** (សៀមរាប, បាត់ដំបង) | 50 នាក់ | ~135 |
| **សាខាតូច** (កំពង់ចាម, ព្រះសីហនុ, កំពត) | 25 នាក់ | ~70 |

### 💼 សេវាកម្ម
- **Core Banking**: Temenos T24
- **ATM & POS Networks**
- **Internet Banking + Mobile Banking** (24/7)
- **Compliance**: NBC + PCI-DSS v4.0

---

## ២. រចនាសម្ព័ន្ធ (Architecture)

### 🏗️ Three-Tier Hierarchical Design + TCP/IP 4 Layers

```
[ INTERNET ]
     │
[ Edge Router → Firewall HA ] ── [ DMZ Zone ]
     │
[ CORE Layer ] ─ HA (VPC)
     │
[ DISTRIBUTION Layer ] ─ 4 units
     │
[ ACCESS Layer ] ─ Server, User, Wi-Fi, WAN
     │
[ MPLS Cloud ] → សាខាទាំង ៥
```

### 🖥️ Equipment Selection

| ស្រទាប់ | ម៉ូឌែល | ចំនួន | ចំណុចខ្លាំង |
|---------|--------|-------|-------------|
| **Core** | Cisco Nexus 9336C-FX2 | 2 | 100 Gbps, VPC HA |
| **Distribution** | Cisco Catalyst 9500 | 4 | L3 Routing, StackWise |
| **Access** | Cisco Catalyst 9200-48P | 20 | PoE+, Stacking |
| **Firewall** | Fortigate 600F | 2 | NGFW, HA, SD-WAN |
| **Edge Router** | Cisco ASR 1001-HX | 2 | 60 Gbps, BGP |
| **Branch Router** | Cisco ISR 4331 | 5 | SD-WAN, DMVPN |
| **Wireless** | Cisco 9800 WLC + 9130AXI AP | 32 | Wi-Fi 6 |

### 🔢 IP Addressing (Layer 3)

| ទីតាំង | Subnet |
|--------|--------|
| Head Office | `10.10.0.0/16` |
| សៀមរាប | `10.20.0.0/16` |
| បាត់ដំបង | `10.30.0.0/16` |
| កំពង់ចាម | `10.40.0.0/16` |
| ព្រះសីហនុ | `10.50.0.0/16` |
| កំពត | `10.60.0.0/16` |
| WAN P2P | `172.16.1.0/24` (/30 links) |
| DMZ | `172.16.99.0/24` |

**Routing**: OSPF (Internal) + BGP (Internet) + HSRP (Gateway HA)

### 🏷️ VLAN Design (Layer 2)

| VLAN | ឈ្មោះ | ការប្រើប្រាស់ |
|------|-------|--------------|
| 10 | MGMT | Network Management |
| 20 | SERVERS | Core Banking, DB |
| 30 | STAFF | បុគ្គលិក (/23) |
| 40 | ATM | ATM Machines (PCI-DSS) |
| 50 | VOIP | IP Phones |
| 60 | WIFI_CORP | Wi-Fi បុគ្គលិក |
| 70 | WIFI_GUEST | Wi-Fi អតិថិជន (Isolated) |
| 99 | DMZ | Public Servers |

### 🔒 Security (Defense in Depth)

- **Perimeter**: NGFW (Fortigate) + IPS + Anti-DDoS
- **Network**: VLAN Segmentation + ACL + NAC
- **Endpoint**: EDR (CrowdStrike) + Patch Mgmt
- **Data**: AES-256 Encryption + TLS 1.3 + Backup
- **Access**: Zero Trust + MFA + Least Privilege
- **Monitoring**: SIEM (Splunk) 24/7

### ⚡ Performance Metrics

| Metric | Target |
|--------|--------|
| Uptime SLA | **99.99%** (< 52 min/year) |
| LAN Latency | < 1 ms |
| WAN Latency | < 50 ms |
| Failover | < 30 វិនាទី |
| Core Throughput | 100 Gbps |
| RTO / RPO | 4h / 15min |

### 💰 Investment

| ចំណាយ | តម្លៃ |
|-------|-------|
| Hardware | ~$563,500 |
| Software & Licenses | ~$80,000 |
| Cabling & Installation | ~$100,000 |
| **សរុប** | **~$743,500 USD** |
| ROI | 3-5 ឆ្នាំ |

---

## ✅ សេចក្តីសន្និដ្ឋាន

- **Scalable** — អាចពង្រីករហូតដល់ 10,000 users
- **Secure** — Defense in Depth + Zero Trust + Compliance
- **Reliable** — Full Redundancy (HA គ្រប់ស្រទាប់)
- **Modern** — Cisco + Fortinet + Wi-Fi 6 + SD-WAN Ready

---

**រៀបចំដោយ**: ក្រុមវិស្វករប្រព័ន្ធ | **កាលបរិច្ឆេទ**: ១៩ សីហា ២០២៦
