# ជំហានទី ២៖ ប្លង់ Logical & Physical Topology
## ការពន្យល់ពី IP Addressing (Layer 3), VLANs (Layer 2) និងការជ្រើសរើសឧបករណ៍បណ្តាញ

---

## មាតិកា

1. [Logical Topology (ប្លង់សណ្ឋានឡូជីខល)](#១-logical-topology)
2. [Physical Topology (ប្លង់សណ្ឋានរូបវន្ត)](#២-physical-topology)
3. [IP Addressing Scheme - Layer 3](#៣-ip-addressing-scheme-layer-3)
4. [VLAN Design - Layer 2](#៤-vlan-design-layer-2)
5. [ការជ្រើសរើសឧបករណ៍បណ្តាញ](#៥-ការជ្រើសរើសឧបករណ៍បណ្តាញ)
6. [ការវិភាគ និងហេតុផលនៃការជ្រើសរើស](#៦-ការវិភាគនិងហេតុផល)

---

## ១. Logical Topology

**Logical Topology** បង្ហាញពីរបៀបដែលទិន្នន័យធ្វើដំណើរ (Data Flow) និងទំនាក់ទំនងតាមឡូជីខលរវាងឧបករណ៍ បណ្តាញ។ វាមិនផ្តោតលើទីតាំងរូបវន្តទេ។

### ១.១ Logical Topology Diagram - Head Office

```
                    ╔═══════════════════════╗
                    ║      INTERNET         ║
                    ║   (Public Cloud)      ║
                    ║   203.0.113.0/30      ║
                    ╚═══════════╤═══════════╝
                                │
                    ┌───────────┴───────────┐
                    │   BGP AS 65001        │
                    │   Edge Router         │
                    │   (R-EDGE)            │
                    │   Public IP:          │
                    │   203.0.113.2/30      │
                    └───────────┬───────────┘
                                │
                    ┌───────────┴───────────┐
                    │   Firewall Cluster    │
                    │   HA Pair (Active/    │
                    │   Standby)            │
                    │   Inside: 172.16.1.1  │
                    └─────┬─────────────┬───┘
                          │             │
                ┌─────────┘             └──────────┐
                │                                  │
        ╔═══════╧═══════╗                  ╔══════╧═══════╗
        ║   DMZ Zone    ║                  ║ Trust Zone   ║
        ║172.16.99.0/24 ║                  ║              ║
        ║               ║                  ║              ║
        ║ Web / Mail /  ║                  ║              ║
        ║ DNS           ║                  ║              ║
        ╚═══════════════╝                  ╚══════╤═══════╝
                                                  │
                                    ┌─────────────┴─────────────┐
                                    │   Core Layer (L3)         │
                                    │   HSRP Virtual IP         │
                                    │   Inter-VLAN Routing      │
                                    └─────────────┬─────────────┘
                                                  │
        ┌──────────────┬──────────────┬───────────┼───────────┬──────────────┐
        │              │              │           │           │              │
   ╔════╧═══╗    ╔════╧═══╗    ╔════╧═══╗   ╔════╧═══╗  ╔════╧═══╗    ╔════╧═════╗
   │VLAN 10 │    │VLAN 20 │    │VLAN 30 │   │VLAN 40 │  │VLAN 50 │    │VLAN 60/70│
   │ MGMT   │    │SERVERS │    │ STAFF  │   │  ATM   │  │ VOIP   │    │ WIRELESS │
   │        │    │        │    │        │   │        │  │        │    │          │
   │10.10.  │    │10.10.  │    │10.10.  │   │10.10.  │  │10.10.  │    │10.10.60/ │
   │10.0/24 │    │20.0/24 │    │30.0/24 │   │40.0/24 │  │50.0/24 │    │70.0/24   │
   ╚════════╝    ╚════════╝    ╚════════╝   ╚════════╝  ╚════════╝    ╚══════════╝
```

### ១.២ Logical Topology - WAN Connectivity

```
                    ╔═══════════════════════╗
                    ║   HEAD OFFICE (HO)    ║
                    ║   10.10.0.0/16        ║
                    ║   AS 65001            ║
                    ╚═══════════╤═══════════╝
                                │
                        MPLS + IPsec VPN
                                │
                    ╔═══════════╧═══════════╗
                    ║  ISP MPLS Cloud       ║
                    ║  (Metfone Business)   ║
                    ╚═══════════╤═══════════╝
                                │
        ┌──────────┬────────────┼────────────┬──────────┐
        │          │            │            │          │
   ╔════╧═══╗ ╔════╧═══╗  ╔════╧═══╗   ╔════╧═══╗ ╔════╧═══╗
   │Siem    │ │Battam- │  │Kampong │   │Sihanouk│ │Kampot  │
   │Reap    │ │bang    │  │Cham    │   │ville   │ │        │
   │10.20.  │ │10.30.  │  │10.40.  │   │10.50.  │ │10.60.  │
   │0.0/16  │ │0.0/16  │  │0.0/16  │   │0.0/16  │ │0.0/16  │
   ╚════════╝ ╚════════╝  ╚════════╝   ╚════════╝ ╚════════╝

   Routing Protocol: OSPF Area 0 (Internal), BGP (External)
   Encryption: IPsec (AES-256, SHA-256, DH Group 14)
   Backup Link: 4G/LTE with automatic failover < 30 seconds
```

### ១.៣ Logical Data Flow

| ប្រតិបត្តិការ | Source | Destination | Protocol | Port |
|-------------|--------|-------------|----------|------|
| Staff → Core Banking | 10.10.30.0/24 | 10.10.20.10 | HTTPS/TCP | 443 |
| ATM → Core Banking | 10.10.40.0/24 | 10.10.20.10 | ISO 8583/TCP | 5000 |
| Branch → HO Server | 10.20.0.0/16 | 10.10.20.0/24 | HTTPS/TCP | 443 |
| Customer → Internet Banking | Internet | 172.16.99.10 | HTTPS/TCP | 443 |
| Staff → DNS | 10.10.30.0/24 | 10.10.20.40 | DNS/UDP | 53 |
| Admin → Devices | 10.10.10.0/24 | All Devices | SSH/TCP | 22 |

---

## ២. Physical Topology

**Physical Topology** បង្ហាញពីទីតាំងជាក់ស្តែងនៃឧបករណ៍ ខ្សែភ្ជាប់ និង Rack Layout។

### ២.១ Physical Topology - Head Office Data Center

```
┌─────────────────────────────────────────────────────────────────┐
│              HEAD OFFICE - DATA CENTER (ភ្នំពេញ)                 │
│                                                                 │
│  ┌────────────┐          ┌────────────┐                        │
│  │   ISP-1    │          │   ISP-2    │  (Redundant WAN)       │
│  │  Metfone   │          │  Cellcard  │                        │
│  └──────┬─────┘          └─────┬──────┘                        │
│         │  Fiber Optic         │  Fiber Optic                  │
│         │  (Single-mode)       │  (Single-mode)                │
│         ▼                      ▼                               │
│  ┌────────────────────────────────────┐                        │
│  │  Rack 1 - Network Equipment        │                        │
│  │  ┌──────────────────────────┐      │                        │
│  │  │ Edge Router (Cisco ASR)  │─────Cat6a Copper             │
│  │  ├──────────────────────────┤      │                        │
│  │  │ Firewall-1 (Fortigate)   │──── HA ──┐                   │
│  │  ├──────────────────────────┤          │                    │
│  │  │ Firewall-2 (Fortigate)   │◀─────────┘                   │
│  │  ├──────────────────────────┤                              │
│  │  │ Core-SW1 (Nexus 9336)    │──── VPC ──┐                  │
│  │  ├──────────────────────────┤            │                  │
│  │  │ Core-SW2 (Nexus 9336)    │◀───────────┘                 │
│  │  ├──────────────────────────┤                              │
│  │  │ Distribution SW1 (9500)  │                              │
│  │  ├──────────────────────────┤                              │
│  │  │ Distribution SW2 (9500)  │                              │
│  │  ├──────────────────────────┤                              │
│  │  │ Distribution SW3 (9500)  │                              │
│  │  ├──────────────────────────┤                              │
│  │  │ Distribution SW4 (9500)  │                              │
│  │  ├──────────────────────────┤                              │
│  │  │ Wireless Controller      │                              │
│  │  ├──────────────────────────┤                              │
│  │  │ UPS (APC 3000VA)         │                              │
│  │  └──────────────────────────┘                              │
│  └────────────────────────────────────┘                        │
│                    │                                            │
│         Fiber Optic (10Gbps LC-LC)                             │
│                    ▼                                            │
│  ┌────────────────────────────────────┐                        │
│  │  Rack 2 - Server Farm              │                        │
│  │  ┌──────────────────────────┐      │                        │
│  │  │ Core Banking (T24)       │──Cat6a──┐                    │
│  │  ├──────────────────────────┤          │                    │
│  │  │ Database Primary (Oracle)│          │                    │
│  │  ├──────────────────────────┤    Access │                   │
│  │  │ Database Backup          │    Switch │                   │
│  │  ├──────────────────────────┤   (2960)  │                   │
│  │  │ Application Server       │◀─────────┘                    │
│  │  ├──────────────────────────┤                              │
│  │  │ AD/DNS/DHCP Server       │                              │
│  │  ├──────────────────────────┤                              │
│  │  │ Backup Server            │                              │
│  │  ├──────────────────────────┤                              │
│  │  │ SIEM (Splunk)            │                              │
│  │  └──────────────────────────┘                              │
│  └────────────────────────────────────┘                        │
│                                                                 │
│  ┌────────────────────────────────────┐                        │
│  │  Rack 3 - DMZ Servers              │                        │
│  │  Web Server, Mail, External DNS    │                        │
│  └────────────────────────────────────┘                        │
│                                                                 │
│         │                                                       │
│    ┌────┴────────────────────────────────┐                     │
│    │  Distribution to Floors             │                     │
│    │  (Fiber to each floor IDF)          │                     │
│    └─────────────────────────────────────┘                     │
└─────────────────────────────────────────────────────────────────┘
                    │
              ┌─────┴─────┐
              │           │
          Floor 1       Floor 2
       IDF Closet    IDF Closet
       (2960 SW)     (2960 SW)
              │           │
         Cat6a         Cat6a
         to PCs        to PCs
```

### ២.២ Physical Topology - Branch Office

```
┌───────────────────────────────────────────────────┐
│         BRANCH OFFICE (ឧ. សៀមរាប)                  │
│                                                   │
│   ISP Modem                                       │
│      │                                            │
│      │ Fiber Optic                                │
│      ▼                                            │
│  ┌───────────┐                                    │
│  │  Branch   │                                    │
│  │  Router   │ Cisco ISR 4331                     │
│  │  (R-SR)   │                                    │
│  └─────┬─────┘                                    │
│        │                                          │
│        │ Cat6a                                    │
│        ▼                                          │
│  ┌───────────┐         ┌──────────┐               │
│  │ Firewall  │────────>│  ATM     │               │
│  │ (Fortigate│         │  Switch  │──> 3 ATMs     │
│  │  100F)    │         └──────────┘               │
│  └─────┬─────┘                                    │
│        │                                          │
│        ▼                                          │
│  ┌───────────┐                                    │
│  │  Access   │ Cisco 2960-24TT                    │
│  │  Switch   │                                    │
│  └─────┬─────┘                                    │
│        │                                          │
│    ┌───┴───┐──── Cat6a ──> Staff PCs (25 units)   │
│    │       │                                      │
│    │       └── AP (Corporate + Guest Wi-Fi)       │
│    │                                              │
│    └── IP Phones (25 units)                       │
│                                                   │
└───────────────────────────────────────────────────┘
```

### ២.៣ ប្រភេទខ្សែ (Cable Types)

| ប្រភេទខ្សែ | ការប្រើប្រាស់ | ចម្ងាយអតិបរមា | ល្បឿន |
|-----------|--------------|---------------|--------|
| **Single-mode Fiber (SMF)** | ISP → Router, Branch WAN | 10-40 km | 1-100 Gbps |
| **Multi-mode Fiber (MMF OM4)** | Between Racks, Core-Dist | 400m | 10-40 Gbps |
| **Cat6a UTP** | Access → PCs, Servers | 100m | 10 Gbps |
| **Cat6 UTP** | End-user connectivity | 55m at 10G | 1-10 Gbps |
| **DAC (Direct Attach)** | Server → ToR Switch | 7m | 10-40 Gbps |
| **Console Cable (RJ45-Serial)** | Management | 3m | - |

---

## ៣. IP Addressing Scheme (Layer 3)

### ៣.១ ការរៀបចំមូលដ្ឋាន

យើងប្រើ **RFC 1918 Private Address Space** ដើម្បី៖
- **សុវត្ថិភាព**: មិនអាចចូលបានផ្ទាល់ពី Internet
- **ភាពសន្សំសំចៃ**: មិនត្រូវការទិញ Public IP ច្រើន
- **ភាពបត់បែន**: អាចប្តូរបានតាមតម្រូវការ

### ៣.២ IP Address Allocation Plan

**Master Plan៖ 10.0.0.0/8** (16 លាន IPs)

```
10.0.0.0/8
├── 10.10.0.0/16    →  Head Office (HO) - ភ្នំពេញ
├── 10.20.0.0/16    →  Branch 1 - សៀមរាប
├── 10.30.0.0/16    →  Branch 2 - បាត់ដំបង
├── 10.40.0.0/16    →  Branch 3 - កំពង់ចាម
├── 10.50.0.0/16    →  Branch 4 - ព្រះសីហនុ
└── 10.60.0.0/16    →  Branch 5 - កំពត

172.16.0.0/16       →  WAN Point-to-Point Links
172.16.99.0/24      →  DMZ Zone
203.0.113.0/30      →  Public IP (ISP Assigned)
```

### ៣.៣ Subnetting សម្រាប់ Head Office (10.10.0.0/16)

| VLAN | Subnet | ជួរ Usable IP | Broadcast | Gateway | Users |
|------|--------|--------------|-----------|---------|-------|
| 10 (MGMT) | 10.10.10.0/24 | 10.10.10.1-254 | 10.10.10.255 | 10.10.10.1 | 50 |
| 20 (SERVERS) | 10.10.20.0/24 | 10.10.20.1-254 | 10.10.20.255 | 10.10.20.1 | 30 |
| 30 (STAFF) | 10.10.30.0/23 | 10.10.30.1 - 10.10.31.254 | 10.10.31.255 | 10.10.30.1 | 510 |
| 40 (ATM) | 10.10.40.0/24 | 10.10.40.1-254 | 10.10.40.255 | 10.10.40.1 | 20 |
| 50 (VOIP) | 10.10.50.0/24 | 10.10.50.1-254 | 10.10.50.255 | 10.10.50.1 | 300 |
| 60 (WIFI_CORP) | 10.10.60.0/24 | 10.10.60.1-254 | 10.10.60.255 | 10.10.60.1 | 100 |
| 70 (WIFI_GUEST) | 10.10.70.0/24 | 10.10.70.1-254 | 10.10.70.255 | 10.10.70.1 | 100 |
| 99 (DMZ) | 172.16.99.0/24 | 172.16.99.1-254 | 172.16.99.255 | 172.16.99.1 | 20 |

### ៣.៤ Subnetting សម្រាប់សាខា (ឧ. សៀមរាប 10.20.0.0/16)

| VLAN | Subnet | ការប្រើប្រាស់ | Gateway |
|------|--------|--------------|---------|
| 10 | 10.20.10.0/24 | Staff LAN | 10.20.10.1 |
| 20 | 10.20.20.0/24 | Local Servers | 10.20.20.1 |
| 40 | 10.20.40.0/24 | ATM (3 machines) | 10.20.40.1 |
| 50 | 10.20.50.0/24 | VoIP | 10.20.50.1 |
| 60 | 10.20.60.0/24 | Wi-Fi Corporate | 10.20.60.1 |
| 70 | 10.20.70.0/24 | Wi-Fi Guest | 10.20.70.1 |

### ៣.៥ WAN Point-to-Point IP (172.16.1.0/24)

ប្រើ **/30** សម្រាប់ P2P link (សន្សំ IP - 4 IPs per subnet, 2 usable)

| Link | Subnet | HO Side | Branch Side |
|------|--------|---------|-------------|
| HO ↔ Siem Reap | 172.16.1.0/30 | 172.16.1.1 | 172.16.1.2 |
| HO ↔ Battambang | 172.16.1.4/30 | 172.16.1.5 | 172.16.1.6 |
| HO ↔ Kampong Cham | 172.16.1.8/30 | 172.16.1.9 | 172.16.1.10 |
| HO ↔ Sihanoukville | 172.16.1.12/30 | 172.16.1.13 | 172.16.1.14 |
| HO ↔ Kampot | 172.16.1.16/30 | 172.16.1.17 | 172.16.1.18 |

### ៣.៦ Layer 3 Routing Protocols

| Protocol | ការប្រើប្រាស់ | ហេតុផល |
|----------|--------------|--------|
| **OSPF** | Internal Routing (HO + Branches) | ល្បឿនលឿន, Auto-convergence, Scalable |
| **BGP** | External Routing (ISP Peering) | Standard សម្រាប់ Internet |
| **Static Route** | DMZ, Special routes | គ្រប់គ្រងបានច្បាស់ |
| **HSRP** | Gateway Redundancy | Failover < 3 seconds |

### ៣.៧ ហេតុផលនៃការជ្រើសរើស IP Scheme

1. **Hierarchical Design**: ងាយស្រួល Summarize Route
2. **Room to Grow**: /16 សម្រាប់ទីតាំងនីមួយៗ គ្រប់គ្រាន់រហូតដល់ 65,000 hosts
3. **VLSM (Variable Length Subnet Masking)**: ប្រើ /30 សម្រាប់ WAN link, /24 សម្រាប់ LAN
4. **Easy Troubleshooting**: 10.20.x.x = សៀមរាប, 10.30.x.x = បាត់ដំបង
5. **Compliance**: អនុលោមតាម RFC 1918

---

## ៤. VLAN Design (Layer 2)

### ៤.១ តើ VLAN ជាអ្វី?

**VLAN (Virtual LAN)** គឺជាការបំបែក Broadcast Domain នៅលើ Switch តែមួយ ជាបណ្តាញឡូជីខលដាច់ដោយឡែក។

### ៤.២ អត្ថប្រយោជន៍នៃ VLAN

| អត្ថប្រយោជន៍ | ការពិពណ៌នា |
|-----------|-------------|
| **សុវត្ថិភាព** | បំបែក Traffic (ឧ. ATM មិនអាចមើលឃើញ Guest Wi-Fi) |
| **Performance** | កាត់បន្ថយ Broadcast Domain |
| **Flexibility** | ផ្លាស់ទី User ដោយមិនចាំបាច់ប្តូរ Cable |
| **Cost Saving** | ប្រើ Switch រូបវន្តតែមួយសម្រាប់ VLAN ច្រើន |
| **Compliance** | ការពារ PCI-DSS Requirement (ATM Segregation) |

### ៤.៣ VLAN Table សម្រាប់ Head Office

| VLAN ID | ឈ្មោះ | គោលបំណង | Access Control |
|---------|-------|--------|----------------|
| 1 | DEFAULT | មិនប្រើ (ធ្វើ Disable) | Blocked |
| 10 | MGMT | Network Device Management | អ្នកគ្រប់គ្រងតែប៉ុណ្ណោះ |
| 20 | SERVERS | Core Banking, DB, App Server | ACL ចាប់សម្រិត |
| 30 | STAFF | បុគ្គលិក PC | អាចទៅ SERVER តាម Port ជាក់លាក់ |
| 40 | ATM | ATM Machines | ទៅតែ Core Banking |
| 50 | VOIP | IP Phone | QoS Priority |
| 60 | WIFI_CORP | Wi-Fi បុគ្គលិក | ដូច Staff |
| 70 | WIFI_GUEST | Wi-Fi អតិថិជន | Internet Only |
| 99 | NATIVE | Trunk Native VLAN | មិនផ្ទុក User Data |
| 999 | BLACKHOLE | Ports មិនប្រើ | Shutdown |

### ៤.៤ VLAN Trunking Configuration

**Trunk Port (រវាង Switch):**
- Encapsulation: **802.1Q (dot1q)**
- Native VLAN: **99** (មិនមែន VLAN 1 - សុវត្ថិភាព)
- Allowed VLAN: **10, 20, 30, 40, 50, 60, 70**

**Access Port (ទៅ End-Device):**
- Mode: **Access**
- VLAN: Assign ១ VLAN ជាក់លាក់
- Feature: **Portfast, BPDU Guard**

### ៤.៥ Inter-VLAN Routing

នៅ **Layer 3 Switch (Core)** យើងបង្កើត **SVI (Switched Virtual Interface)** សម្រាប់ VLAN នីមួយៗ៖

```cisco
interface Vlan10
 ip address 10.10.10.1 255.255.255.0
interface Vlan20
 ip address 10.10.20.1 255.255.255.0
interface Vlan30
 ip address 10.10.30.1 255.255.255.0
```

**Inter-VLAN Routing Flow:**
```
PC (VLAN 30) → Access SW → Trunk → Core SW (L3)
              → SVI Vlan30 → Route → SVI Vlan20
              → Trunk → Access SW → Server (VLAN 20)
```

### ៤.៦ VLAN Security Best Practices

1. **Disable unused ports** និងដាក់ក្នុង VLAN 999 (Blackhole)
2. **VLAN 1 មិនប្រើ** - ប្តូរ Native VLAN ទៅ 99
3. **DHCP Snooping** - ការពារ Rogue DHCP Server
4. **Dynamic ARP Inspection** - ការពារ ARP Poisoning
5. **Port Security** - Limit MAC Address ក្នុង Port
6. **Storm Control** - ការពារ Broadcast Storm
7. **Private VLAN** សម្រាប់ Server Farm - Isolate Server ពីគ្នា

---

## ៥. ការជ្រើសរើសឧបករណ៍បណ្តាញ

### ៥.១ Core Layer Equipment

#### 🔷 Core Switch: Cisco Nexus 9336C-FX2

| លក្ខណៈបច្ចេកទេស | តម្លៃ |
|---------------|-------|
| **Ports** | 36x 100G QSFP28 |
| **Throughput** | 7.2 Tbps |
| **Latency** | < 1 microsecond |
| **Features** | VXLAN, EVPN, VPC, ACI-ready |
| **Redundancy** | Dual Power, Dual Fan |
| **តម្លៃប៉ាន់ស្មាន** | ~$25,000 USD |

**ហេតុផលជ្រើសរើស:**
- High Throughput សម្រាប់ Bank Transaction
- VPC (Virtual Port Channel) សម្រាប់ HA
- Programmable (Ansible, Python API)

### ៥.២ Distribution Layer Equipment

#### 🔷 Cisco Catalyst 9500-24Y4C

| លក្ខណៈបច្ចេកទេស | តម្លៃ |
|---------------|-------|
| **Ports** | 24x 25G SFP28 + 4x 100G QSFP28 |
| **Throughput** | 2 Tbps |
| **Layer 3 Features** | OSPF, BGP, EIGRP, MPLS |
| **StackWise Virtual** | HA between chassis |
| **តម្លៃប៉ាន់ស្មាន** | ~$15,000 USD |

**ហេតុផលជ្រើសរើស:**
- L3 Routing Capabilities
- StackWise Virtual for redundancy
- Cisco DNA Center support

### ៥.៣ Access Layer Equipment

#### 🔷 Cisco Catalyst 9200-48P

| លក្ខណៈបច្ចេកទេស | តម្លៃ |
|---------------|-------|
| **Ports** | 48x 1G RJ45 (PoE+) + 4x 10G SFP+ |
| **PoE Budget** | 740W (សម្រាប់ IP Phone/AP) |
| **StackWise** | Up to 8 units |
| **Features** | VLAN, QoS, Port Security |
| **តម្លៃប៉ាន់ស្មាន** | ~$5,000 USD |

**ហេតុផលជ្រើសរើស:**
- PoE+ សម្រាប់ IP Phone និង AP
- ងាយស្រួល Stack (កាត់បន្ថយ Management)
- Cost-effective

### ៥.៤ Firewall

#### 🔷 Fortinet FortiGate 600F

| លក្ខណៈបច្ចេកទេស | តម្លៃ |
|---------------|-------|
| **Firewall Throughput** | 66 Gbps |
| **IPS Throughput** | 10 Gbps |
| **VPN Throughput** | 25 Gbps (IPsec) |
| **Concurrent Sessions** | 8 Million |
| **HA Support** | Active/Passive, Active/Active |
| **Features** | NGFW, IPS, App Control, SD-WAN |
| **តម្លៃប៉ាន់ស្មាន** | ~$30,000 USD (per unit) |

**ហេតុផលជ្រើសរើស:**
- Next-Gen Firewall (NGFW) - IPS integrated
- FortiGuard Threat Intelligence
- SD-WAN capability សម្រាប់ Branch
- ជាមួយ FortiAnalyzer សម្រាប់ Log Management

### ៥.៥ Router

#### 🔷 Edge Router: Cisco ASR 1001-HX

| លក្ខណៈបច្ចេកទេស | តម្លៃ |
|---------------|-------|
| **Throughput** | 60 Gbps |
| **Interfaces** | 8x 10G + 4x 1G |
| **Features** | BGP, MPLS, IPsec, QoS |
| **តម្លៃប៉ាន់ស្មាន** | ~$20,000 USD |

#### 🔷 Branch Router: Cisco ISR 4331

| លក្ខណៈបច្ចេកទេស | តម្លៃ |
|---------------|-------|
| **Throughput** | 100 Mbps - 300 Mbps |
| **Interfaces** | 3x 1G + Modular Slots |
| **Features** | SD-WAN, DMVPN, MPLS |
| **តម្លៃប៉ាន់ស្មាន** | ~$3,000 USD |

### ៥.៦ Wireless Equipment

#### 🔷 Wireless Controller: Cisco 9800-40

| លក្ខណៈបច្ចេកទេស | តម្លៃ |
|---------------|-------|
| **AP Support** | Up to 2,000 APs |
| **Client Capacity** | 32,000 |
| **Throughput** | 40 Gbps |
| **តម្លៃប៉ាន់ស្មាន** | ~$15,000 USD |

#### 🔷 Access Point: Cisco Catalyst 9130AXI

| លក្ខណៈបច្ចេកទេស | តម្លៃ |
|---------------|-------|
| **Standard** | Wi-Fi 6 (802.11ax) |
| **Speed** | Up to 5.38 Gbps |
| **Features** | OFDMA, MU-MIMO, BLE |
| **តម្លៃប៉ាន់ស្មាន** | ~$1,500 USD (per AP) |

### ៥.៧ សរុប Bill of Materials (BoM)

| ឧបករណ៍ | ម៉ូឌែល | ចំនួន | តម្លៃឯកតា | សរុប |
|---------|--------|-------|-----------|------|
| Core Switch | Cisco Nexus 9336C-FX2 | 2 | $25,000 | $50,000 |
| Distribution SW | Cisco Catalyst 9500 | 4 | $15,000 | $60,000 |
| Access Switch | Cisco Catalyst 9200 | 20 | $5,000 | $100,000 |
| Edge Router | Cisco ASR 1001-HX | 2 | $20,000 | $40,000 |
| Branch Router | Cisco ISR 4331 | 5 | $3,000 | $15,000 |
| Firewall | Fortigate 600F | 2 | $30,000 | $60,000 |
| Wireless Controller | Cisco 9800-40 | 2 | $15,000 | $30,000 |
| Access Point | Cisco Catalyst 9130AXI | 30 | $1,500 | $45,000 |
| UPS | APC Smart-UPS 3000 | 3 | $2,500 | $7,500 |
| Rack (42U) | APC Netshelter | 3 | $2,000 | $6,000 |
| Structured Cabling | Cat6a/Fiber | Lot | - | $50,000 |
| **សរុបរួម** | | | | **~$463,500** |

---

## ៦. ការវិភាគ និងហេតុផល

### ៦.១ ហេតុអ្វីជ្រើសរើស Three-Tier Architecture?

| ស្រទាប់ | មុខងារ | ហេតុផល |
|--------|--------|--------|
| **Core** | Backbone High-Speed | Throughput ខ្ពស់, Latency ទាប |
| **Distribution** | Policy, Aggregation | Inter-VLAN Routing, ACL, QoS |
| **Access** | End-user Connectivity | Cost-effective, PoE for AP/Phone |

### ៦.២ ហេតុអ្វីជ្រើសរើស Cisco?

1. **Market Leader** - ជឿជាក់បានក្នុងវិស័យធនាគារ
2. **Support** - ជាមួយ Smart Net ២៤/៧
3. **Documentation** - មានឯកសារច្រើន
4. **ECOSYSTEM** - ជាមួយ Cisco DNA, ISE, Prime Infrastructure
5. **Trained Engineers** - នៅកម្ពុជាមានវិស្វករ CCNP/CCIE ច្រើន

### ៦.៣ ហេតុអ្វីជ្រើសរើស Fortigate សម្រាប់ Firewall?

1. **Performance** - Throughput ខ្ពស់ជាង Cisco ASA ក្នុងតម្លៃដូចគ្នា
2. **FortiGuard** - Threat Intelligence ធ្វើបច្ចុប្បន្នភាពរាល់នាទី
3. **SD-WAN** - Built-in មិនត្រូវការ Software បន្ថែម
4. **Cost** - Total Cost of Ownership ទាបជាង

### ៦.៤ Redundancy Design

| ស្រទាប់ | Redundancy Method |
|--------|-------------------|
| **ISP** | Dual ISP (Metfone + Cellcard) |
| **Router** | HSRP with 2 Routers |
| **Firewall** | Active/Standby HA Pair |
| **Core Switch** | VPC (Virtual Port Channel) |
| **Distribution** | StackWise Virtual |
| **Access** | Stacking (Cisco StackWise) |
| **Server** | Dual NIC Teaming |
| **Power** | Dual UPS + Generator |
| **Cooling** | N+1 Precision AC |

### ៦.៥ Performance Metrics

| Metric | Target | Method |
|--------|--------|--------|
| **Uptime** | 99.99% (~52 minutes downtime/year) | Full HA Design |
| **Latency (LAN)** | < 1ms | High-speed switching |
| **Latency (HO ↔ Branch)** | < 50ms | MPLS Provider SLA |
| **Bandwidth (Core)** | 100 Gbps | Nexus 9336C |
| **Bandwidth (Distribution)** | 25-40 Gbps | Catalyst 9500 |
| **Bandwidth (Access)** | 1 Gbps per user | Catalyst 9200 |
| **Wi-Fi Speed** | Up to 1 Gbps | Wi-Fi 6 (802.11ax) |
| **Failover Time** | < 3 seconds | HSRP, HA |

---

## ៧. សេចក្តីសន្និដ្ឋានជំហានទី ២

ជំហានទី ២ បានបង្ហាញយ៉ាងច្បាស់អំពី៖

✅ **Logical Topology** — បង្ហាញលំហូរទិន្នន័យ និងទំនាក់ទំនងរវាង VLAN
✅ **Physical Topology** — Rack Layout, Cable Type, ទីតាំងឧបករណ៍ជាក់ស្តែង
✅ **IP Addressing (L3)** — Hierarchical Scheme សម្រាប់ HO + សាខា ៥
✅ **VLAN Design (L2)** — 8 VLANs បំបែកតាមមុខងារ ធានាសុវត្ថិភាព
✅ **Equipment Selection** — Cisco + Fortinet ដោយផ្តោតលើ HA និង Performance

### ជំហានបន្ទាប់៖
- **ជំហានទី ៣**: ការកំណត់រចនាសម្ព័ន្ធ (Configuration) និង Deployment
- **ជំហានទី ៤**: ការសាកល្បង (Testing & Validation)
- **ជំហានទី ៥**: ការប្រគល់ និងបណ្តុះបណ្តាល (Handover & Training)

---

**រៀបចំដោយ**: ក្រុមវិស្វករប្រព័ន្ធ
**កាលបរិច្ឆេទ**: ថ្ងៃទី ១៩ សីហា ២០២៦
**Version**: 1.0
**ឯកសារយោង**: [Enterprise_Network_Design_TCPIP.md](Enterprise_Network_Design_TCPIP.md)
