# Network Flow Diagrams
## ដ្យាក្រាមលំហូរបណ្តាញធនាគារ (Visual Representation)

---

## ១. Logical Network Topology (Mermaid)

```mermaid
graph TB
    subgraph "INTERNET"
        ISP[Internet / ISP<br/>203.0.113.0/30]
    end

    subgraph "PERIMETER"
        EDGE[R-EDGE Router<br/>Cisco 2911]
        FW1[Firewall-1<br/>Fortigate 600F<br/>ACTIVE]
        FW2[Firewall-2<br/>Fortigate 600F<br/>STANDBY]
    end

    subgraph "DMZ Zone - 172.16.99.0/24"
        WEB[Web Server<br/>172.16.99.10]
        MAIL[Mail Server<br/>172.16.99.20]
        DNS_EXT[External DNS<br/>172.16.99.30]
    end

    subgraph "CORE LAYER"
        CORE1[CORE-SW1<br/>Cisco Nexus 9336]
        CORE2[CORE-SW2<br/>Cisco Nexus 9336]
    end

    subgraph "DISTRIBUTION LAYER"
        DIST1[DIST-SW1<br/>Server Farm]
        DIST2[DIST-SW2<br/>User LAN]
        DIST3[DIST-SW3<br/>Wireless]
        DIST4[DIST-SW4<br/>WAN Edge]
    end

    subgraph "SERVER FARM - VLAN 20"
        CB[Core Banking<br/>10.10.20.10]
        DB[Database<br/>10.10.20.20]
        APP[App Server<br/>10.10.20.30]
        AD[AD/DNS<br/>10.10.20.40]
    end

    subgraph "USER LAN - VLAN 30"
        ACC1[Access SW]
        PC[Staff PCs<br/>10.10.30.0/24]
        VOIP[IP Phones<br/>VLAN 50]
    end

    subgraph "WIRELESS"
        WLC[Wireless Controller]
        AP_CORP[AP Corporate<br/>VLAN 60]
        AP_GUEST[AP Guest<br/>VLAN 70]
    end

    subgraph "WAN - BRANCHES"
        MPLS[MPLS Cloud]
        SR[Siem Reap<br/>10.20.0.0/16]
        BB[Battambang<br/>10.30.0.0/16]
        KC[Kampong Cham<br/>10.40.0.0/16]
        SH[Sihanoukville<br/>10.50.0.0/16]
        KP[Kampot<br/>10.60.0.0/16]
    end

    ISP --> EDGE
    EDGE --> FW1
    EDGE --> FW2
    FW1 -.HA Sync.- FW2
    FW1 --> WEB
    FW1 --> MAIL
    FW1 --> DNS_EXT
    FW1 --> CORE1
    FW2 --> CORE2
    CORE1 -.VSS.- CORE2
    CORE1 --> DIST1
    CORE1 --> DIST2
    CORE2 --> DIST3
    CORE2 --> DIST4
    DIST1 --> CB
    DIST1 --> DB
    DIST1 --> APP
    DIST1 --> AD
    DIST2 --> ACC1
    ACC1 --> PC
    ACC1 --> VOIP
    DIST3 --> WLC
    WLC --> AP_CORP
    WLC --> AP_GUEST
    DIST4 --> MPLS
    MPLS --> SR
    MPLS --> BB
    MPLS --> KC
    MPLS --> SH
    MPLS --> KP

    style ISP fill:#dae8fc,stroke:#6c8ebf
    style FW1 fill:#f8cecc,stroke:#b85450
    style FW2 fill:#ffe6cc,stroke:#d79b00
    style CORE1 fill:#d5e8d4,stroke:#82b366
    style CORE2 fill:#d5e8d4,stroke:#82b366
    style CB fill:#fff2cc,stroke:#d6b656
    style DB fill:#fff2cc,stroke:#d6b656
```

---

## ២. Packet Flow Diagram — TCP/IP Layers

### ២.១ Data Flow: Staff PC ↔ Core Banking Server

```mermaid
sequenceDiagram
    participant PC as Staff PC<br/>10.10.30.10
    participant SW as Access Switch
    participant CORE as Core Switch<br/>(L3)
    participant SRV as Core Banking<br/>10.10.20.10

    Note over PC,SRV: Layer 4 - Application (HTTPS)
    PC->>SW: HTTPS Request<br/>[HTTP/1.1 GET /login]

    Note over PC,SRV: Layer 3 - Transport (TCP)
    PC->>SW: TCP SYN [Src Port 5000, Dst Port 443]
    SW->>CORE: Forward (Trunk VLAN 30)
    CORE->>SRV: TCP SYN
    SRV->>CORE: TCP SYN-ACK
    CORE->>SW: TCP SYN-ACK
    SW->>PC: TCP SYN-ACK
    PC->>SRV: TCP ACK [Handshake Complete]

    Note over PC,SRV: Layer 2 - Internet (IP Routing)
    Note right of CORE: Src IP: 10.10.30.10<br/>Dst IP: 10.10.20.10<br/>Route: Vlan30 → Vlan20

    Note over PC,SRV: Layer 1 - Network Access (Ethernet)
    Note right of SW: Src MAC: PC_MAC<br/>Dst MAC: Gateway_MAC<br/>Frame: 802.1Q Tag VLAN 30

    Note over PC,SRV: Data Transfer
    PC->>SRV: HTTPS Payload (Encrypted)
    SRV->>PC: HTTPS Response<br/>[TLS 1.3 Encrypted]
```

---

## ៣. VLAN Segmentation Diagram

```
+----------------------------------------------------------+
|                     CORE SWITCH                          |
|                  (Inter-VLAN Routing)                    |
+----------------------------------------------------------+
        |         |         |         |         |
        v         v         v         v         v
   +--------+ +--------+ +--------+ +--------+ +--------+
   | VLAN10 | | VLAN20 | | VLAN30 | | VLAN40 | | VLAN60 |
   |  MGMT  | | SERVER | | STAFF  | |  ATM   | |WIFI-CP |
   |        | |        | |        | |        | |        |
   |10.10.  | |10.10.  | |10.10.  | |10.10.  | |10.10.  |
   |10.0/24 | |20.0/24 | |30.0/24 | |40.0/24 | |60.0/24 |
   +--------+ +--------+ +--------+ +--------+ +--------+
        |         |         |         |         |
     Admin     Core       PCs       ATM     Corporate
     Access    Banking    IP Phones Machines Wi-Fi
              Database
```

---

## ៤. Security Zone Flow

```mermaid
graph LR
    subgraph "UNTRUSTED"
        INT[Internet]
    end

    subgraph "SEMI-TRUSTED"
        DMZ[DMZ<br/>Web/Mail/DNS]
    end

    subgraph "TRUSTED"
        SRV[Server Zone<br/>Core Banking]
        USR[User Zone]
        ATM_Z[ATM Zone]
    end

    subgraph "RESTRICTED"
        GST[Guest Wi-Fi]
    end

    INT -->|Allow: 80,443| DMZ
    INT -->|Deny All| SRV
    INT -->|Deny All| USR
    DMZ -->|Allow: 443 to CB| SRV
    USR -->|Allow: 443,80,53| SRV
    USR -.Deny.- ATM_Z
    ATM_Z -->|Only to CB| SRV
    GST -->|Only Internet| INT
    GST -.Deny.- SRV
    GST -.Deny.- USR

    style INT fill:#f8cecc,stroke:#b85450
    style DMZ fill:#fff2cc,stroke:#d6b656
    style SRV fill:#d5e8d4,stroke:#82b366
    style GST fill:#dae8fc,stroke:#6c8ebf
```

---

## ៥. WAN / Branch Connectivity Flow

```
                    HEAD OFFICE (ភ្នំពេញ)
                    ┌─────────────────────┐
                    │  R-EDGE (2911)      │
                    │  BGP AS 65001       │
                    └──────────┬──────────┘
                               │
                    ┌──────────┴──────────┐
                    │   MPLS Provider     │
                    │   (Metfone/Cellcard)│
                    └──────────┬──────────┘
                               │
        ┌───────────┬──────────┼──────────┬────────────┐
        │           │          │          │            │
   ┌────┴────┐ ┌───┴───┐ ┌────┴────┐ ┌───┴────┐ ┌────┴────┐
   │  R-SR   │ │ R-BB  │ │  R-KC   │ │  R-SH  │ │  R-KP   │
   │ សៀមរាប  │ │បាត់ដំបង│ │កំពង់ចាម │ │ព្រះសីហនុ│ │  កំពត   │
   │10.20/16 │ │10.30/16│ │10.40/16 │ │10.50/16│ │10.60/16 │
   └─────────┘ └───────┘ └─────────┘ └────────┘ └─────────┘

   Primary Link: MPLS (Metfone Business - 100 Mbps)
   Backup Link:  4G/LTE (Automatic Failover < 30s)
   VPN: IPsec Site-to-Site (AES-256, SHA-256)
```

---

## ៦. High Availability Flow

```mermaid
graph TB
    subgraph "Active Path (Primary)"
        A1[ISP-1] --> A2[Edge Router]
        A2 --> A3[Firewall-1 ACTIVE]
        A3 --> A4[Core-SW1 ACTIVE]
        A4 --> A5[Users/Servers]
    end

    subgraph "Standby Path (Failover)"
        B1[ISP-2] -.Backup.-> B2[Edge Router 2]
        B2 -.-> B3[Firewall-2 STANDBY]
        B3 -.-> B4[Core-SW2 STANDBY]
        B4 -.-> B5[Users/Servers]
    end

    A3 -.HA Sync.- B3
    A4 -.VSS/VPC.- B4

    Failure{Primary Fails?}
    Failure -->|Yes| Failover[Auto Failover < 3 sec]
    Failover --> Standby[Standby becomes Active]

    style A3 fill:#d5e8d4,stroke:#82b366
    style A4 fill:#d5e8d4,stroke:#82b366
    style B3 fill:#f8cecc,stroke:#b85450
    style B4 fill:#f8cecc,stroke:#b85450
```

---

## ៧. Data Flow - Internet Banking Transaction

```mermaid
sequenceDiagram
    participant U as Customer<br/>(Internet)
    participant FW as Firewall
    participant WAF as Web App Firewall
    participant WEB as Web Server (DMZ)
    participant APP as App Server
    participant DB as Database
    participant CB as Core Banking

    U->>FW: HTTPS Request (443)
    FW->>WAF: Filter Traffic
    WAF->>WEB: Allow (SQL Injection Check)
    WEB->>APP: API Call (Internal)
    APP->>DB: Query User Account
    DB-->>APP: Account Info
    APP->>CB: Transaction Request<br/>(Encrypted)
    CB->>DB: Update Balance
    CB-->>APP: Transaction Confirmed
    APP-->>WEB: Response
    WEB-->>WAF: JSON Data
    WAF-->>FW: Encrypt Response
    FW-->>U: HTTPS Response<br/>(TLS 1.3)

    Note over U,CB: All Transactions Logged in SIEM
```

---

## ៨. Physical Rack Layout (Head Office Data Center)

```
┌─────────────────────────────────────┐
│         RACK 1 (Network)            │
├─────────────────────────────────────┤
│ U42 │ [CABLE MGMT]                  │
│ U41 │ [PATCH PANEL 48-port]         │
│ U40 │ [PATCH PANEL 48-port]         │
│ U39 │ [EDGE ROUTER - Cisco ASR]     │
│ U38 │ [FIREWALL-1 - Fortigate 600F] │
│ U37 │ [FIREWALL-2 - Fortigate 600F] │
│ U36 │ [CORE-SW1 - Nexus 9336]       │
│ U35 │ [CORE-SW2 - Nexus 9336]       │
│ U34 │ [DIST-SW1 - Catalyst 9500]    │
│ U33 │ [DIST-SW2 - Catalyst 9500]    │
│ U32 │ [DIST-SW3 - Catalyst 9500]    │
│ U31 │ [DIST-SW4 - Catalyst 9500]    │
│ U30 │ [WLC - Wireless Controller]   │
│ ... │ ...                           │
│ U2  │ [UPS - APC Smart-UPS 3000]    │
│ U1  │ [PDU]                         │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│         RACK 2 (Servers)            │
├─────────────────────────────────────┤
│ U42 │ [KVM Console]                 │
│ U40 │ [Core Banking Server]         │
│ U38 │ [Database Server (Primary)]   │
│ U36 │ [Database Server (Backup)]    │
│ U34 │ [Application Server]          │
│ U32 │ [AD / DNS Server]             │
│ U30 │ [Backup Server]               │
│ U28 │ [SIEM Server (Splunk)]        │
│ ... │ ...                           │
│ U2  │ [UPS]                         │
└─────────────────────────────────────┘
```

---

## ៩. TCP/IP Encapsulation Flow

```
Application Layer:  [   HTTP/HTTPS Data   ]
                            ↓
Transport Layer:    [ TCP Header | Data ]
                            ↓
Internet Layer:     [ IP Header | TCP | Data ]
                            ↓
Network Access:     [ Ethernet | IP | TCP | Data | FCS ]
                            ↓
                    ┌─────────────────────┐
                    │  Physical Medium    │
                    │  (Fiber/Copper)     │
                    └─────────────────────┘
```

### ឧទាហរណ៍ Packet ដែលកំពុងឆ្លងកាត់ (Real Data):

```
┌──────────────────────────────────────────────────┐
│ Ethernet Frame                                    │
│ ┌──────────────────────────────────────────────┐ │
│ │ Dst MAC: 00:1A:2B:3C:4D:5E                  │ │
│ │ Src MAC: 00:1F:2E:3D:4C:5B                  │ │
│ │ Type: 0x0800 (IPv4)                          │ │
│ │ VLAN Tag: 802.1Q, ID=30                      │ │
│ │ ┌────────────────────────────────────────┐   │ │
│ │ │ IP Header                              │   │ │
│ │ │ Version: 4  |  IHL: 20  |  TTL: 64    │   │ │
│ │ │ Src IP: 10.10.30.10                    │   │ │
│ │ │ Dst IP: 10.10.20.10                    │   │ │
│ │ │ Protocol: 6 (TCP)                      │   │ │
│ │ │ ┌──────────────────────────────────┐   │   │ │
│ │ │ │ TCP Header                       │   │   │ │
│ │ │ │ Src Port: 54321                  │   │   │ │
│ │ │ │ Dst Port: 443 (HTTPS)            │   │   │ │
│ │ │ │ Flags: [SYN]                     │   │   │ │
│ │ │ │ ┌────────────────────────────┐   │   │   │ │
│ │ │ │ │  Application Data (TLS)    │   │   │   │ │
│ │ │ │ │  Encrypted Payload         │   │   │   │ │
│ │ │ │ └────────────────────────────┘   │   │   │ │
│ │ │ └──────────────────────────────────┘   │   │ │
│ │ └────────────────────────────────────────┘   │ │
│ │ FCS (Frame Check Sequence)                    │ │
│ └──────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────┘
```

---

## ១០. Implementation Timeline (Gantt-style)

```mermaid
gantt
    title Bank Network Implementation Timeline
    dateFormat  YYYY-MM-DD

    section Phase 1: Planning
    Requirements Gathering    :done,    p1a, 2026-08-01, 7d
    Network Design            :done,    p1b, after p1a, 10d
    Vendor Selection          :done,    p1c, after p1b, 5d

    section Phase 2: Hardware
    Equipment Procurement     :active,  p2a, 2026-08-20, 14d
    Physical Installation     :         p2b, after p2a, 10d
    Cabling                   :         p2c, after p2b, 7d

    section Phase 3: Configuration
    Core & Distribution       :         p3a, 2026-09-15, 5d
    Access Layer              :         p3b, after p3a, 7d
    Firewall & Security       :         p3c, after p3b, 5d
    WAN & VPN                 :         p3d, after p3c, 7d

    section Phase 4: Testing
    Connectivity Test         :         p4a, 2026-10-15, 3d
    Performance Test          :         p4b, after p4a, 3d
    Security Audit            :         p4c, after p4b, 5d

    section Phase 5: Go-Live
    Pilot Deployment          :         p5a, 2026-11-01, 7d
    Full Rollout              :         p5b, after p5a, 14d
    Training & Handover       :         p5c, after p5b, 7d
```

---

## ១១. របៀបបើកឯកសារ

### Draw.io / Visio File (`Network_Topology.drawio`):
1. ចូល https://app.diagrams.net (ឥតគិតថ្លៃ)
2. File → Open From → Device → ជ្រើសរើស `Network_Topology.drawio`
3. **Export ជា Visio**: File → Export as → VSDX
4. **Export ជា PNG/PDF**: File → Export as → PNG/PDF

### Mermaid Diagrams (ក្នុងឯកសារនេះ):
- បើកជា Markdown Viewer (VS Code, GitHub, Typora)
- ដ្យាក្រាមនឹង Render ដោយស្វ័យប្រវត្តិ
- Online: https://mermaid.live

### Packet Tracer Lab:
- អនុវត្តតាមឯកសារ `PacketTracer_Lab_Guide.md`
- Save ជា `Bank_Enterprise_Network.pkt`

---

**រៀបចំដោយ**: ក្រុមវិស្វករប្រព័ន្ធ
**កាលបរិច្ឆេទ**: ថ្ងៃទី ១៩ សីហា ២០២៦
