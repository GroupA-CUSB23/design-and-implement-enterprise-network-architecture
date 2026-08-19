# ការរចនា និងអនុវត្តស្ថាបត្យកម្មបណ្តាញសហគ្រាស ដោយប្រើប្រាស់គំរូ TCP/IP

## ករណីសិក្សា៖ ធនាគារពាណិជ្ជកម្មនៅកម្ពុជា

> **ឧបមា៖** ក្រុមរបស់យើងជាក្រុមវិស្វករប្រព័ន្ធ (System Engineers) ត្រូវបានជួលដោយធនាគារមួយនៅកម្ពុជា ដើម្បីរចនា និងអនុវត្តស្ថាបត្យកម្មបណ្តាញសហគ្រាសសម្រាប់ការិយាល័យកណ្តាល (Head Office) និងសាខាចំនួន ៥ នៅតាមខេត្តនានា។

---

## មាតិកា (Table of Contents)

1. [សេចក្តីផ្តើម (Introduction)](#១-សេចក្តីផ្តើម)
2. [ការវិភាគតម្រូវការ (Requirements Analysis)](#២-ការវិភាគតម្រូវការ)
3. [ការសិក្សាគំរូ TCP/IP (TCP/IP Model Study)](#៣-ការសិក្សាគំរូ-tcpip)
4. [ការរចនាស្ថាបត្យកម្មបណ្តាញ (Network Architecture Design)](#៤-ការរចនាស្ថាបត្យកម្មបណ្តាញ)
5. [ការរៀបចំ IP Addressing និង Subnetting](#៥-ការរៀបចំ-ip-addressing-និង-subnetting)
6. [ការរចនាសុវត្ថិភាព (Security Design)](#៦-ការរចនាសុវត្ថិភាព)
7. [ជំហានអនុវត្ត (Implementation Steps)](#៧-ជំហានអនុវត្ត)
8. [ការសាកល្បង និងផ្ទៀងផ្ទាត់ (Testing & Validation)](#៨-ការសាកល្បង-និងផ្ទៀងផ្ទាត់)
9. [ការគ្រប់គ្រង និងតាមដាន (Monitoring & Management)](#៩-ការគ្រប់គ្រង-និងតាមដាន)
10. [សេចក្តីសន្និដ្ឋាន (Conclusion)](#១០-សេចក្តីសន្និដ្ឋាន)

---

## ១. សេចក្តីផ្តើម

ក្នុងយុគសម័យឌីជីថលនេះ ធនាគារត្រូវការប្រព័ន្ធបណ្តាញដែលមាន៖

- **ស្ថេរភាពខ្ពស់ (High Availability)** — មិនអាចផ្តាច់សេវាបានឡើយ
- **សុវត្ថិភាពខ្ពស់ (High Security)** — ការពារទិន្នន័យអតិថិជន និងប្រតិបត្តិការហិរញ្ញវត្ថុ
- **ល្បឿនលឿន (High Performance)** — សម្រាប់ប្រតិបត្តិការ Real-time
- **អាចពង្រីកបាន (Scalability)** — សម្រាប់ការរីកចម្រើននាពេលអនាគត

គំរូ **TCP/IP (Transmission Control Protocol / Internet Protocol)** គឺជាមូលដ្ឋានស្តង់ដារសម្រាប់ការទំនាក់ទំនងបណ្តាញទំនើប។

---

## ២. ការវិភាគតម្រូវការ

### ២.១ តម្រូវការអាជីវកម្ម (Business Requirements)

| លេខរៀង | តម្រូវការ | ការពិពណ៌នា |
|--------|----------|-------------|
| BR-01 | ភ្ជាប់ការិយាល័យកណ្តាល និងសាខាទាំង ៥ | Head Office នៅភ្នំពេញ + សាខានៅសៀមរាប បាត់ដំបង កំពង់ចាម ព្រះសីហនុ និងកំពត |
| BR-02 | គាំទ្រប្រព័ន្ធ Core Banking | ដូចជា Temenos T24, Flexcube |
| BR-03 | ATM និង POS Networks | ភ្ជាប់ជាមួយប្រព័ន្ធកណ្តាល |
| BR-04 | Internet Banking និង Mobile Banking | សេវាកម្ម Online 24/7 |
| BR-05 | ការគោរពតាមបទប្បញ្ញត្តិ | NBC (ធនាគារជាតិកម្ពុជា), PCI-DSS |

### ២.២ តម្រូវការបច្ចេកទេស (Technical Requirements)

- **Bandwidth**: យ៉ាងតិច 100 Mbps ក្នុងសាខានីមួយៗ និង 1 Gbps នៅ Head Office
- **Latency**: តិចជាង 50ms រវាង Head Office និងសាខា
- **Uptime**: 99.99% (Four Nines)
- **Redundancy**: មាន Backup Link គ្រប់សាខា
- **Security**: End-to-end encryption, Firewall, IDS/IPS

### ២.៣ ចំនួនអ្នកប្រើប្រាស់

| ទីតាំង | បុគ្គលិក | ATM | POS | ម៉ាស៊ីនកុំព្យូទ័រ |
|--------|----------|-----|-----|------------------|
| Head Office | 300 | 5 | - | 350 |
| សាខាធំ (សៀមរាប, បាត់ដំបង) | 50 | 3 | 20 | 60 |
| សាខាតូច (កំពង់ចាម, ព្រះសីហនុ, កំពត) | 25 | 2 | 10 | 30 |

---

## ៣. ការសិក្សាគំរូ TCP/IP

គំរូ TCP/IP មាន **៤ ស្រទាប់ (Layers)**៖

### ៣.១ Layer 4 — Application Layer

- **Protocols**: HTTP/HTTPS, FTP, SMTP, DNS, SSH, SNMP
- **ការប្រើប្រាស់ក្នុងធនាគារ**:
  - HTTPS សម្រាប់ Internet Banking
  - SMTP សម្រាប់ការជូនដំណឹងតាមអ៊ីមែល
  - DNS សម្រាប់ដោះស្រាយឈ្មោះ Domain

### ៣.២ Layer 3 — Transport Layer

- **Protocols**: TCP, UDP
- **TCP**: សម្រាប់ប្រតិបត្តិការហិរញ្ញវត្ថុ (ត្រូវការភាពជឿជាក់)
- **UDP**: សម្រាប់ VoIP, Video Conference

### ៣.៣ Layer 2 — Internet Layer

- **Protocols**: IP (IPv4/IPv6), ICMP, ARP, Routing (OSPF, BGP)
- **មុខងារ**: Routing ទិន្នន័យរវាងបណ្តាញផ្សេងៗ

### ៣.៤ Layer 1 — Network Access Layer

- **Protocols**: Ethernet, Wi-Fi, PPP, Frame Relay
- **ឧបករណ៍**: Switch, Access Point, Cable

### តារាងប្រៀបធៀប TCP/IP និង OSI

```
+------------------+---------------------+
|   TCP/IP Model   |     OSI Model       |
+------------------+---------------------+
| Application      | Application         |
|                  | Presentation        |
|                  | Session             |
+------------------+---------------------+
| Transport        | Transport           |
+------------------+---------------------+
| Internet         | Network             |
+------------------+---------------------+
| Network Access   | Data Link           |
|                  | Physical            |
+------------------+---------------------+
```

---

## ៤. ការរចនាស្ថាបត្យកម្មបណ្តាញ

### ៤.១ ស្ថាបត្យកម្មបីស្រទាប់ (Three-Tier Architecture)

យើងប្រើប្រាស់គំរូ **Cisco Hierarchical Network Model**៖

```
        [ Core Layer ]
        (Backbone - High-speed Routing)
              |
        [ Distribution Layer ]
        (Policy, Filtering, Routing)
              |
        [ Access Layer ]
        (End-user connectivity)
```

- **Core Layer**: Cisco Nexus 9000, Router សម្រាប់ Backbone
- **Distribution Layer**: L3 Switches (Cisco Catalyst 9500)
- **Access Layer**: L2 Switches (Cisco Catalyst 9200) និង Wireless AP

### ៤.២ Topology (សណ្ឋានបណ្តាញ)

```
                    [ Internet / ISP ]
                            |
                    +---------------+
                    |   Firewall    |  (Fortigate/Palo Alto)
                    |    (HA Pair)  |
                    +-------+-------+
                            |
                    +---------------+
                    |   DMZ Zone    |  (Web Server, Mail, DNS)
                    +-------+-------+
                            |
                    +---------------+
                    |  Core Switch  |
                    |    (HA Pair)  |
                    +-------+-------+
                            |
       +--------------+-----+-----+---------------+
       |              |           |               |
  [Server Farm]  [User LAN]  [Wi-Fi]         [WAN Link]
   (Core Bank,   (Staff)      (Guest,          |
    Database)                  Corporate)      |
                                     +---------+---------+
                                     |                   |
                                MPLS/VPN            Backup Link
                                     |            (4G/LTE Failover)
                                     |
                            [ សាខាទាំង ៥ ]
```

### ៤.៣ ការបែងចែក Zone

| Zone | ការពិពណ៌នា |
|------|-------------|
| **DMZ** | សម្រាប់ Web Server, Mail Server ដែលអាចមើលឃើញពី Internet |
| **Server Zone** | Core Banking, Database, Application Server |
| **User Zone** | បុគ្គលិកធនាគារ |
| **ATM Zone** | បណ្តាញ ATM ដាច់ដោយឡែក |
| **Management Zone** | សម្រាប់តាមដាន និងគ្រប់គ្រង |
| **Guest Zone** | Wi-Fi សម្រាប់អតិថិជន (ដាច់ដោយឡែក) |

---

## ៥. ការរៀបចំ IP Addressing និង Subnetting

### ៥.១ Address Plan (ប្រើ Private IP តាម RFC 1918)

យើងប្រើ **10.0.0.0/8** សម្រាប់ Internal Network។

| ទីតាំង | Subnet | ចំនួន IP ដែលអាចប្រើ |
|--------|--------|----------------------|
| Head Office | 10.10.0.0/16 | 65,534 |
| សាខាសៀមរាប | 10.20.0.0/16 | 65,534 |
| សាខាបាត់ដំបង | 10.30.0.0/16 | 65,534 |
| សាខាកំពង់ចាម | 10.40.0.0/16 | 65,534 |
| សាខាព្រះសីហនុ | 10.50.0.0/16 | 65,534 |
| សាខាកំពត | 10.60.0.0/16 | 65,534 |

### ៥.២ VLAN Design សម្រាប់ Head Office

| VLAN ID | ឈ្មោះ | Subnet | ការប្រើប្រាស់ |
|---------|-------|--------|----------------|
| 10 | MGMT | 10.10.10.0/24 | Management |
| 20 | SERVERS | 10.10.20.0/24 | Core Banking Servers |
| 30 | STAFF | 10.10.30.0/23 | បុគ្គលិក |
| 40 | ATM | 10.10.40.0/24 | ATM Machines |
| 50 | VOIP | 10.10.50.0/24 | IP Phone |
| 60 | WIFI_CORP | 10.10.60.0/24 | Wi-Fi បុគ្គលិក |
| 70 | WIFI_GUEST | 10.10.70.0/24 | Wi-Fi អតិថិជន |
| 99 | DMZ | 172.16.99.0/24 | DMZ Servers |

### ៥.៣ WAN IP (រវាងសាខា)

ប្រើ **172.16.0.0/24** សម្រាប់ Point-to-Point Links៖

- HO ↔ សៀមរាប: 172.16.1.0/30
- HO ↔ បាត់ដំបង: 172.16.1.4/30
- HO ↔ កំពង់ចាម: 172.16.1.8/30
- HO ↔ ព្រះសីហនុ: 172.16.1.12/30
- HO ↔ កំពត: 172.16.1.16/30

---

## ៦. ការរចនាសុវត្ថិភាព

### ៦.១ Defense in Depth (សុវត្ថិភាពច្រើនស្រទាប់)

```
[ Perimeter ] → [ Network ] → [ Endpoint ] → [ Application ] → [ Data ]
```

### ៦.២ ឧបករណ៍ និងបច្ចេកវិទ្យាសុវត្ថិភាព

| ស្រទាប់ | ដំណោះស្រាយ |
|---------|--------------|
| **Perimeter** | Next-Gen Firewall (Fortigate 600F HA Pair) |
| **DDoS Protection** | Cloudflare / ISP-level protection |
| **IDS/IPS** | Snort / Cisco Firepower |
| **VPN** | IPsec Site-to-Site VPN + SSL VPN សម្រាប់អ្នកប្រើ Remote |
| **Endpoint** | CrowdStrike / Kaspersky Endpoint Security |
| **Email Security** | Proofpoint / Cisco Email Security |
| **SIEM** | Splunk / IBM QRadar សម្រាប់ Log Monitoring |
| **MFA** | Multi-Factor Authentication សម្រាប់អ្នកប្រើទាំងអស់ |

### ៦.៣ គោលការណ៍សុវត្ថិភាព (Security Policies)

1. **Least Privilege**: អ្នកប្រើទទួលបានសិទ្ធិចាំបាច់ប៉ុណ្ណោះ
2. **Network Segmentation**: បំបែក VLAN តាមមុខងារ
3. **Zero Trust**: មិនទុកចិត្តលើអ្វីទាំងអស់ ត្រូវផ្ទៀងផ្ទាត់រាល់ការចូល
4. **Encryption**: TLS 1.3 សម្រាប់ In-transit, AES-256 សម្រាប់ At-rest
5. **Regular Audit**: Vulnerability Scanning ជារៀងរាល់ខែ

---

## ៧. ជំហានអនុវត្ត

### ជំហានទី ១៖ ការរៀបចំ (Planning Phase)

- ✅ ប្រមូលតម្រូវការពីភាគីពាក់ព័ន្ធ
- ✅ រៀបចំ Network Diagram (Logical & Physical)
- ✅ ជ្រើសរើសឧបករណ៍ និង Vendor
- ✅ រៀបចំថវិកា និងកាលវិភាគគម្រោង

### ជំហានទី ២៖ ការតំឡើងផ្នែករឹង (Hardware Installation)

- តំឡើង Rack, UPS, Cooling System នៅ Data Center
- តំឡើង Router, Switch, Firewall, Server
- តភ្ជាប់ Cable (Fiber Optic សម្រាប់ Backbone, Cat6a សម្រាប់ Access)

### ជំហានទី ៣៖ ការកំណត់រចនាសម្ព័ន្ធមូលដ្ឋាន (Base Configuration)

**ឧទាហរណ៍៖ Configuration របស់ Core Switch (Cisco)**

```cisco
! Hostname
hostname CORE-SW-HO-01

! Enable password
enable secret StrongPassword123!

! VLAN Creation
vlan 10
 name MGMT
vlan 20
 name SERVERS
vlan 30
 name STAFF
vlan 40
 name ATM

! SVI Configuration
interface Vlan10
 ip address 10.10.10.1 255.255.255.0
 no shutdown

interface Vlan20
 ip address 10.10.20.1 255.255.255.0
 no shutdown

! Trunk Port to Distribution Switch
interface GigabitEthernet1/0/1
 description To-DIST-SW-01
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40

! Enable SSH
ip domain-name bank.com.kh
crypto key generate rsa modulus 2048
line vty 0 15
 transport input ssh
 login local

username admin privilege 15 secret AdminPass123!
```

### ជំហានទី ៤៖ Routing Configuration

**OSPF សម្រាប់ Internal Routing**៖

```cisco
router ospf 1
 router-id 1.1.1.1
 network 10.10.0.0 0.0.255.255 area 0
 network 172.16.1.0 0.0.0.255 area 0
```

**BGP សម្រាប់ ISP Peering**៖

```cisco
router bgp 65001
 neighbor 203.0.113.1 remote-as 24491
 network 10.10.0.0 mask 255.255.0.0
```

### ជំហានទី ៥៖ VPN Configuration (Site-to-Site)

**IPsec VPN រវាង HO និងសាខាសៀមរាប**៖

```cisco
crypto isakmp policy 10
 encryption aes 256
 hash sha256
 authentication pre-share
 group 14

crypto isakmp key SecureKey123! address 172.16.20.1

crypto ipsec transform-set BANK_TS esp-aes 256 esp-sha256-hmac

crypto map BANK_MAP 10 ipsec-isakmp
 set peer 172.16.20.1
 set transform-set BANK_TS
 match address VPN_ACL

interface GigabitEthernet0/0
 crypto map BANK_MAP

ip access-list extended VPN_ACL
 permit ip 10.10.0.0 0.0.255.255 10.20.0.0 0.0.255.255
```

### ជំហានទី ៦៖ Security Configuration

**ACL ការពារ ATM Zone**៖

```cisco
ip access-list extended ATM_PROTECT
 permit tcp 10.10.40.0 0.0.0.255 host 10.10.20.10 eq 443
 permit udp 10.10.40.0 0.0.0.255 host 10.10.20.20 eq 53
 deny ip any any log

interface Vlan40
 ip access-group ATM_PROTECT in
```

### ជំហានទី ៧៖ Deployment ទៅសាខា

- ដឹកជញ្ជូន និងតំឡើងឧបករណ៍
- Configure Router/Switch តាម Template
- តភ្ជាប់ MPLS/VPN ជាមួយ HO
- សាកល្បង Connectivity

---

## ៨. ការសាកល្បង និងផ្ទៀងផ្ទាត់

### ៨.១ Connectivity Testing

```bash
# Ping test
ping 10.10.20.10

# Traceroute
tracert 10.20.20.10

# Path MTU Discovery
ping 10.20.20.10 -f -l 1472
```

### ៨.២ Performance Testing

- **iPerf3**: វាស់ Bandwidth រវាងសាខា
- **PRTG / Nagios**: តាមដាន Uptime
- **Wireshark**: វិភាគ Packet

### ៨.៣ Security Testing

- **Vulnerability Scan**: Nessus, OpenVAS
- **Penetration Test**: ជួល Third-party ជារៀងរាល់ឆ្នាំ
- **Firewall Rule Audit**: ត្រួតពិនិត្យ Rule ជារៀងរាល់ខែ

### ៨.៤ Disaster Recovery Test

- Simulate ISP Down → ត្រូវ Failover ទៅ Backup Link ក្នុងរយៈពេល < 30 វិនាទី
- Simulate Core Switch Failure → HA Pair ត្រូវទទួលយកតួនាទីភ្លាមៗ

---

## ៩. ការគ្រប់គ្រង និងតាមដាន

### ៩.១ Network Management Tools

| ប្រភេទ | ឧបករណ៍ |
|--------|---------|
| **NMS** | SolarWinds, PRTG, Zabbix |
| **SIEM** | Splunk, IBM QRadar |
| **Log Management** | Graylog, ELK Stack |
| **Configuration Mgmt** | Ansible, Cisco DNA Center |

### ៩.២ Monitoring Metrics

- **Availability**: Uptime %
- **Performance**: Latency, Jitter, Packet Loss
- **Capacity**: Bandwidth Utilization
- **Security**: Failed Login, Suspicious Traffic

### ៩.៣ Backup និង Disaster Recovery

- **Configuration Backup**: ជារៀងរាល់ថ្ងៃ
- **Data Backup**: Full ជារៀងរាល់សប្តាហ៍, Incremental ជារៀងរាល់ថ្ងៃ
- **DR Site**: មាន Data Center បម្រុងទុកនៅខេត្តផ្សេង
- **RTO** (Recovery Time Objective): 4 ម៉ោង
- **RPO** (Recovery Point Objective): 15 នាទី

---

## ១០. សេចក្តីសន្និដ្ឋាន

ការរចនាស្ថាបត្យកម្មបណ្តាញសហគ្រាសសម្រាប់ធនាគារត្រូវការការពិចារណាយ៉ាងលម្អិតលើ៖

1. **គំរូ TCP/IP** ជាមូលដ្ឋានសម្រាប់ការទំនាក់ទំនង
2. **Three-Tier Architecture** ធានាបាននូវ Scalability និង Manageability
3. **Segmentation ដោយ VLAN** បង្កើនសុវត្ថិភាព
4. **Redundancy** នៅគ្រប់ស្រទាប់ធានាបាន High Availability
5. **Defense in Depth** ការពារពីការវាយប្រហារ Cyber Attacks
6. **ការតាមដានជាបន្តបន្ទាប់** ធានាបាននូវប្រតិបត្តិការមានប្រសិទ្ធភាព

### ការណែនាំបន្ថែម

- ធ្វើ **Security Audit** យ៉ាងតិច ១ ដងក្នុងមួយឆ្នាំ
- បណ្តុះបណ្តាលបុគ្គលិកអំពី Cybersecurity Awareness
- ធ្វើបច្ចុប្បន្នភាព Firmware និង Software ជាទៀងទាត់
- រៀបចំ **Incident Response Plan** ឲ្យបានច្បាស់លាស់

---

## ឧបសម្ព័ន្ធ (Appendices)

### ក. បញ្ជីឧបករណ៍ (Bill of Materials)

| ឧបករណ៍ | ម៉ូឌែល | ចំនួន | ទីតាំង |
|---------|--------|-------|--------|
| Firewall | Fortigate 600F | 2 | Head Office |
| Core Switch | Cisco Nexus 9336 | 2 | Head Office |
| Distribution Switch | Cisco Catalyst 9500 | 4 | Head Office |
| Access Switch | Cisco Catalyst 9200 | 20 | Head Office & សាខា |
| Router (Branch) | Cisco ISR 4331 | 5 | សាខានីមួយៗ |
| Wireless AP | Cisco Meraki MR46 | 30 | គ្រប់ទីតាំង |

### ខ. Glossary (វាក្យស័ព្ទ)

- **ACL**: Access Control List
- **BGP**: Border Gateway Protocol
- **DMZ**: Demilitarized Zone
- **HA**: High Availability
- **IPsec**: Internet Protocol Security
- **MPLS**: Multiprotocol Label Switching
- **OSPF**: Open Shortest Path First
- **SIEM**: Security Information and Event Management
- **VLAN**: Virtual Local Area Network
- **VPN**: Virtual Private Network

### គ. ឯកសារយោង (References)

1. Cisco Enterprise Network Design Guide
2. NIST Cybersecurity Framework
3. PCI-DSS v4.0 Requirements
4. NBC Regulations on Bank Technology Risk Management
5. RFC 1918 — Private Address Space
6. RFC 793 — Transmission Control Protocol

---

**រៀបចំដោយ**: ក្រុមវិស្វករប្រព័ន្ធ
**កាលបរិច្ឆេទ**: ថ្ងៃទី ១៩ សីហា ២០២៦
**កំណែ (Version)**: 1.0
