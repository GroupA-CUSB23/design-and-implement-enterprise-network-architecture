# Cisco Packet Tracer Lab Guide
## ការសាងសង់បណ្តាញធនាគារនៅ Packet Tracer (ជំហានៗ)

---

## ១. តម្រូវការមុនចាប់ផ្តើម

- **Cisco Packet Tracer**: Version 8.2 ឬថ្មីជាង
- **RAM**: យ៉ាងតិច 4GB
- **ចំណេះដឹងមូលដ្ឋាន**: Cisco IOS Commands, VLAN, Routing

---

## ២. ឧបករណ៍ដែលត្រូវប្រើក្នុង Packet Tracer

| ប្រភេទ | ម៉ូឌែលក្នុង Packet Tracer | ចំនួន |
|--------|---------------------------|-------|
| Router | **Cisco 2911** (ISR 2900) | 6 (HO + 5 សាខា) |
| Layer 3 Switch | **Cisco 3650-24PS** | 2 (Core) |
| Layer 2 Switch | **Cisco 2960-24TT** | 6 |
| Server | **Server-PT** | 4 (Web, DNS, DHCP, Core Banking) |
| PC | **PC-PT** | 10+ |
| Access Point | **AP-PT** | 4 |
| Wireless Device | **Laptop-PT** / **Smartphone-PT** | 4 |
| Firewall | **5506-X** (ASA) | 1 |

---

## ៣. Physical Topology Diagram (ASCII)

```
                    [ Cloud - Internet ]
                            |
                    [ Router: R-EDGE ]
                        (2911)
                            |
                    [ Firewall: ASA5506 ]
                            |
                +-----------+-----------+
                |                       |
        [ Core-SW1: 3650 ]======[ Core-SW2: 3650 ]
                |                       |
      +---------+                       +----------+
      |                                            |
[ Server Farm ]   [ User LAN ]   [ Wi-Fi ]   [ WAN Router ]
   (VLAN 20)       (VLAN 30)     (VLAN 60/70)   (2911)
                                                      |
                                              [ Frame Relay Cloud ]
                                                      |
                            +---------+---------+---------+---------+
                            |         |         |         |         |
                        [R-SR]    [R-BB]    [R-KC]    [R-SH]    [R-KP]
                       Siem Reap Battambang KpgCham  Sihanouk   Kampot
```

---

## ៤. ជំហានទី ១៖ បង្កើត Project និងទាញឧបករណ៍

### 4.1 បើក Packet Tracer និងបង្កើត File ថ្មី

1. File → New (Ctrl+N)
2. រក្សាទុកជា `Bank_Enterprise_Network.pkt`

### 4.2 ទាញឧបករណ៍ចូល Workspace

**នៅផ្នែក Head Office (ខាងឆ្វេង):**
- ទាញ **1x Cloud (Internet)** ដាក់ខាងលើ
- ទាញ **1x Router 2911** — ដាក់ឈ្មោះ `R-EDGE`
- ទាញ **1x ASA 5506-X** — ដាក់ឈ្មោះ `FW-HO`
- ទាញ **2x Switch 3650-24PS** — ដាក់ឈ្មោះ `CORE-SW1`, `CORE-SW2`
- ទាញ **2x Switch 2960-24TT** — ដាក់ឈ្មោះ `ACC-SW-SERVER`, `ACC-SW-USER`
- ទាញ **4x Server-PT** — ដាក់ឈ្មោះ:
  - `SRV-CORE-BANKING` (10.10.20.10)
  - `SRV-DB` (10.10.20.20)
  - `SRV-DNS` (10.10.20.30)
  - `SRV-WEB-DMZ` (172.16.99.10)
- ទាញ **3x PC-PT** — ជាបុគ្គលិកធនាគារ
- ទាញ **1x AP-PT** និង **1x Laptop-PT**

**នៅផ្នែកសាខានីមួយៗ (ខាងស្តាំ):**
- **1x Router 2911** ក្នុងសាខានីមួយៗ (ដាក់ឈ្មោះ `R-SR`, `R-BB`, `R-KC`, `R-SH`, `R-KP`)
- **1x Switch 2960**
- **2x PC**

---

## ៥. ជំហានទី ២៖ តភ្ជាប់ខ្សែ (Cabling)

### ប្រភេទខ្សែសំខាន់ៗ

| ការតភ្ជាប់ | ប្រភេទខ្សែ |
|-----------|-----------|
| Router ↔ Router (Serial) | **Serial DCE** |
| Router ↔ Switch | **Copper Straight-Through** |
| Switch ↔ Switch | **Copper Cross-Over** |
| Switch ↔ PC/Server | **Copper Straight-Through** |
| AP ↔ Switch | **Copper Straight-Through** |
| Cloud ↔ Router | **Coaxial** ឬ **Fiber** |

### ការតភ្ជាប់ជាក់ស្តែង

```
Cloud       Gi0/0 <──> Gi0/0    R-EDGE
R-EDGE      Gi0/1 <──> Gi1      FW-HO
FW-HO       Gi2   <──> Gi1/0/1  CORE-SW1
CORE-SW1    Gi1/0/24 <──> Gi1/0/24 CORE-SW2  (EtherChannel/Trunk)
CORE-SW1    Gi1/0/2  <──> Gi0/1  ACC-SW-SERVER
CORE-SW2    Gi1/0/2  <──> Gi0/1  ACC-SW-USER
ACC-SW-SERVER  Fa0/1 <──> Fa0    SRV-CORE-BANKING
ACC-SW-USER    Fa0/1 <──> Fa0    PC1
```

---

## ៦. ជំហានទី ៣៖ Configuration របស់ Core Switch 1

Click លើ `CORE-SW1` → CLI Tab → ចម្លងបញ្ជាខាងក្រោម៖

```cisco
enable
configure terminal

hostname CORE-SW1

! Enable password
enable secret Cisco@123
service password-encryption

! Create VLANs
vlan 10
 name MGMT
vlan 20
 name SERVERS
vlan 30
 name STAFF
vlan 40
 name ATM
vlan 50
 name VOIP
vlan 60
 name WIFI_CORP
vlan 70
 name WIFI_GUEST
exit

! Configure SVI (Switched Virtual Interface)
interface Vlan10
 description Management
 ip address 10.10.10.1 255.255.255.0
 no shutdown

interface Vlan20
 description Server Farm
 ip address 10.10.20.1 255.255.255.0
 no shutdown

interface Vlan30
 description Staff
 ip address 10.10.30.1 255.255.255.0
 no shutdown

interface Vlan40
 description ATM
 ip address 10.10.40.1 255.255.255.0
 no shutdown

interface Vlan60
 description WiFi Corporate
 ip address 10.10.60.1 255.255.255.0
 no shutdown

interface Vlan70
 description WiFi Guest
 ip address 10.10.70.1 255.255.255.0
 no shutdown

! Enable IP Routing
ip routing

! Trunk Port to CORE-SW2
interface GigabitEthernet1/0/24
 description To-CORE-SW2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50,60,70

! Trunk to Access Switches
interface range GigabitEthernet1/0/2 - 4
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50,60,70

! Port to Firewall
interface GigabitEthernet1/0/1
 description To-Firewall
 no switchport
 ip address 172.16.1.2 255.255.255.252
 no shutdown

! Default Route to Firewall
ip route 0.0.0.0 0.0.0.0 172.16.1.1

! Save Configuration
end
write memory
```

---

## ៧. ជំហានទី ៤៖ Configuration Access Switch (User)

Click លើ `ACC-SW-USER` → CLI Tab:

```cisco
enable
configure terminal

hostname ACC-SW-USER
enable secret Cisco@123

! Create VLANs
vlan 30
 name STAFF
vlan 50
 name VOIP
exit

! Trunk to Core Switch
interface GigabitEthernet0/1
 description To-CORE-SW1
 switchport mode trunk
 switchport trunk allowed vlan 30,50

! Access Ports for Staff PCs (VLAN 30)
interface range FastEthernet0/1 - 10
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast

! Access + Voice VLAN (for IP Phones)
interface range FastEthernet0/11 - 15
 switchport mode access
 switchport access vlan 30
 switchport voice vlan 50
 spanning-tree portfast

end
write memory
```

---

## ៨. ជំហានទី ៥៖ DHCP Server Configuration

Click លើ `CORE-SW1` → CLI:

```cisco
configure terminal

! DHCP Pool for Staff (VLAN 30)
ip dhcp pool STAFF_POOL
 network 10.10.30.0 255.255.255.0
 default-router 10.10.30.1
 dns-server 10.10.20.30
 domain-name bank.com.kh
 lease 7

! DHCP Pool for WiFi Guest (VLAN 70)
ip dhcp pool GUEST_POOL
 network 10.10.70.0 255.255.255.0
 default-router 10.10.70.1
 dns-server 8.8.8.8
 lease 0 4

! Exclude static IPs
ip dhcp excluded-address 10.10.30.1 10.10.30.10
ip dhcp excluded-address 10.10.70.1 10.10.70.5

end
write memory
```

---

## ៩. ជំហានទី ៦៖ Router Configuration (R-EDGE)

Click លើ `R-EDGE` → CLI:

```cisco
enable
configure terminal

hostname R-EDGE
enable secret Cisco@123

! Interface to Internet
interface GigabitEthernet0/0
 description To-Internet
 ip address 203.0.113.2 255.255.255.252
 no shutdown

! Interface to Firewall
interface GigabitEthernet0/1
 description To-Firewall
 ip address 172.16.0.1 255.255.255.252
 no shutdown

! Default route to ISP
ip route 0.0.0.0 0.0.0.0 203.0.113.1

! Route back to internal network
ip route 10.10.0.0 255.255.0.0 172.16.0.2
ip route 10.20.0.0 255.255.0.0 172.16.0.2
ip route 10.30.0.0 255.255.0.0 172.16.0.2

! NAT for Internet Access
access-list 1 permit 10.0.0.0 0.255.255.255
ip nat inside source list 1 interface GigabitEthernet0/0 overload

interface GigabitEthernet0/0
 ip nat outside
interface GigabitEthernet0/1
 ip nat inside

! Enable SSH
ip domain-name bank.com.kh
crypto key generate rsa
! (When prompted, choose 2048)
username admin privilege 15 secret Admin@123
line vty 0 4
 login local
 transport input ssh
end
write memory
```

---

## ១០. ជំហានទី ៧៖ Branch Router Configuration (R-SR - សៀមរាប)

```cisco
enable
configure terminal

hostname R-SR
enable secret Cisco@123

! LAN Interface
interface GigabitEthernet0/0
 description Siem Reap LAN
 ip address 10.20.10.1 255.255.255.0
 no shutdown

! WAN Interface to HO
interface Serial0/0/0
 description To-HO
 ip address 172.16.1.6 255.255.255.252
 clock rate 64000
 no shutdown

! Static Route to HO Network
ip route 10.10.0.0 255.255.0.0 172.16.1.5
ip route 0.0.0.0 0.0.0.0 172.16.1.5

! DHCP for Branch
ip dhcp pool BRANCH_SR
 network 10.20.10.0 255.255.255.0
 default-router 10.20.10.1
 dns-server 10.10.20.30

ip dhcp excluded-address 10.20.10.1 10.20.10.10

end
write memory
```

**បន្ថែម Branch ដទៃទៀត** ដោយផ្លាស់ IP និង Hostname៖

| Router | LAN Subnet | WAN IP |
|--------|-----------|--------|
| R-SR (សៀមរាប) | 10.20.10.0/24 | 172.16.1.6/30 |
| R-BB (បាត់ដំបង) | 10.30.10.0/24 | 172.16.1.10/30 |
| R-KC (កំពង់ចាម) | 10.40.10.0/24 | 172.16.1.14/30 |
| R-SH (ព្រះសីហនុ) | 10.50.10.0/24 | 172.16.1.18/30 |
| R-KP (កំពត) | 10.60.10.0/24 | 172.16.1.22/30 |

---

## ១១. ជំហានទី ៨៖ OSPF Routing (Dynamic Routing)

**នៅ R-EDGE:**

```cisco
configure terminal
router ospf 1
 router-id 1.1.1.1
 network 172.16.0.0 0.0.0.3 area 0
 network 172.16.1.0 0.0.0.255 area 0
end
write memory
```

**នៅ Branch Router (ឧ. R-SR):**

```cisco
configure terminal
router ospf 1
 router-id 2.2.2.2
 network 10.20.10.0 0.0.0.255 area 0
 network 172.16.1.4 0.0.0.3 area 0
end
write memory
```

---

## ១២. ជំហានទី ៩៖ Server Configuration

### DNS Server (SRV-DNS - 10.10.20.30)

1. Click លើ Server → Config → FastEthernet0
   - IP: `10.10.20.30`
   - Subnet: `255.255.255.0`
   - Gateway: `10.10.20.1`
2. Services → DNS → **On**
   - Add Record: `bank.com.kh` → `10.10.20.10`
   - Add Record: `www.bank.com.kh` → `172.16.99.10`

### Web Server (SRV-WEB-DMZ - 172.16.99.10)

1. Config → FastEthernet0
   - IP: `172.16.99.10`
   - Subnet: `255.255.255.0`
   - Gateway: `172.16.99.1`
2. Services → HTTP → **On**
3. កែច្នៃ `index.html`៖
   ```html
   <html>
   <body style="background:#003366; color:white; text-align:center;">
   <h1>Welcome to Bank of Cambodia - Internet Banking</h1>
   <p>Secure Login Portal</p>
   </body>
   </html>
   ```

### Core Banking Server (SRV-CORE-BANKING - 10.10.20.10)

1. Config → FastEthernet0
   - IP: `10.10.20.10`
   - Subnet: `255.255.255.0`
   - Gateway: `10.10.20.1`
2. Services → HTTP → **On**

---

## ១៣. ជំហានទី ១០៖ PC Configuration

**Staff PC (VLAN 30):**

Click PC → Desktop → IP Configuration:
- **DHCP** → Request → ត្រូវទទួលបាន IP ក្នុងជួរ `10.10.30.11-254`

**Verification:**
- Desktop → Command Prompt:
  ```
  ipconfig /all
  ping 10.10.20.10  (Core Banking)
  ping 10.10.20.30  (DNS)
  ping 10.20.10.11  (Branch PC)
  ```

---

## ១៤. ជំហានទី ១១៖ Wireless AP Configuration

1. Click **AP-PT** → Config → Port 1
   - SSID: `BANK_CORP`
   - Authentication: **WPA2-PSK**
   - Password: `BankSecure@2026`
2. តភ្ជាប់ AP ជាមួយ Switch (VLAN 60)
3. Click **Laptop** → Config → Wireless
   - SSID: `BANK_CORP`
   - Password: `BankSecure@2026`
   - IP Configuration: DHCP

---

## ១៥. ជំហានទី ១២៖ Security - ACL Configuration

**ការពារ Server Zone ពី User Zone:**

នៅ `CORE-SW1`:

```cisco
configure terminal

! ACL: អនុញ្ញាតតែ HTTP/HTTPS ពី Staff ទៅ Core Banking
ip access-list extended STAFF_TO_SERVER
 permit tcp 10.10.30.0 0.0.0.255 host 10.10.20.10 eq 443
 permit tcp 10.10.30.0 0.0.0.255 host 10.10.20.10 eq 80
 permit udp 10.10.30.0 0.0.0.255 host 10.10.20.30 eq 53
 permit icmp 10.10.30.0 0.0.0.255 10.10.20.0 0.0.0.255 echo-reply
 deny ip any any log

! Apply on VLAN 30
interface Vlan30
 ip access-group STAFF_TO_SERVER in

end
write memory
```

**ការពារ Guest Wi-Fi ពី Internal:**

```cisco
ip access-list extended GUEST_ISOLATE
 deny ip 10.10.70.0 0.0.0.255 10.10.0.0 0.0.255.255
 permit ip 10.10.70.0 0.0.0.255 any

interface Vlan70
 ip access-group GUEST_ISOLATE in
```

---

## ១៦. ជំហានទី ១៣៖ Site-to-Site VPN (IPsec)

**R-EDGE (HO):**

```cisco
configure terminal

! Phase 1
crypto isakmp policy 10
 encryption aes 256
 hash sha
 authentication pre-share
 group 5
 lifetime 3600

crypto isakmp key BankVPN@2026 address 172.16.1.6

! Phase 2
crypto ipsec transform-set BANK-TS esp-aes 256 esp-sha-hmac

! Crypto ACL
ip access-list extended VPN-ACL
 permit ip 10.10.0.0 0.0.255.255 10.20.0.0 0.0.255.255

! Crypto Map
crypto map BANK-MAP 10 ipsec-isakmp
 set peer 172.16.1.6
 set transform-set BANK-TS
 match address VPN-ACL

interface GigabitEthernet0/1
 crypto map BANK-MAP

end
write memory
```

**R-SR (Branch):**

```cisco
configure terminal
crypto isakmp policy 10
 encryption aes 256
 hash sha
 authentication pre-share
 group 5

crypto isakmp key BankVPN@2026 address 172.16.1.5
crypto ipsec transform-set BANK-TS esp-aes 256 esp-sha-hmac

ip access-list extended VPN-ACL
 permit ip 10.20.0.0 0.0.255.255 10.10.0.0 0.0.255.255

crypto map BANK-MAP 10 ipsec-isakmp
 set peer 172.16.1.5
 set transform-set BANK-TS
 match address VPN-ACL

interface Serial0/0/0
 crypto map BANK-MAP

end
write memory
```

---

## ១៧. ជំហានទី ១៤៖ ការសាកល្បង (Verification & Testing)

### Test 1: Connectivity Tests

នៅ Staff PC (Desktop → Command Prompt):

```
ping 10.10.30.1        → Gateway (must reply)
ping 10.10.20.10       → Core Banking Server
ping 10.10.20.30       → DNS Server
ping 10.20.10.11       → Siem Reap Branch PC
ping www.bank.com.kh   → Web Server (via DNS)
tracert 10.20.10.11    → បង្ហាញ Route
```

### Test 2: Web Browser

Staff PC → Desktop → Web Browser:
- URL: `http://www.bank.com.kh` → ត្រូវបង្ហាញ Welcome Page
- URL: `http://10.10.20.10` → Core Banking

### Test 3: ACL Test

- Staff PC ping Core Banking (10.10.20.10) → **Success**
- Staff PC ping Database (10.10.20.20) → **Denied**
- Guest Laptop ping Staff PC → **Denied**

### Test 4: Show Commands

នៅ CORE-SW1:
```cisco
show vlan brief
show ip interface brief
show ip route
show running-config
show interfaces trunk
show mac address-table
```

នៅ Router:
```cisco
show ip route ospf
show crypto isakmp sa
show crypto ipsec sa
show ip nat translations
```

---

## ១៨. ជំហានទី ១៥៖ Simulation Mode (Packet Flow Visualization)

### របៀបមើលដំណើរការ Packet:

1. ចុច **Simulation Mode** (កំពូលខាងស្តាំ) ឬ Shift+S
2. Edit Filters → តែ **ICMP, HTTP, DNS**
3. នៅ Staff PC → Desktop → Command Prompt → `ping 10.10.20.10`
4. ចុច **Auto Capture / Play**
5. ឃើញ Packet ធ្វើដំណើរជា Animation៖
   - PC → Access Switch → Core Switch → Access Switch → Server → Reply Back

### អ្វីដែលត្រូវសង្កេតឃើញ (TCP/IP Layers):

| Layer | ការសង្កេត |
|-------|-----------|
| **Application** | HTTP GET, DNS Query |
| **Transport** | TCP 3-way Handshake (SYN, SYN-ACK, ACK) |
| **Internet** | IP Header (Source/Destination IP) |
| **Network Access** | Ethernet Frame (Source/Destination MAC) |

Click លើ Packet → PDU Details → មើល Header នៃស្រទាប់នីមួយៗ

---

## ១៩. Configuration Backup

រៀងរាល់ Device (Router/Switch):

```cisco
copy running-config startup-config
copy running-config tftp
! Enter TFTP server IP: 10.10.20.30
! Enter filename: R-EDGE-config.txt
```

---

## ២០. រៀបចំ Documentation

រៀបចំតារាងទាំងនេះជា Excel/PDF ភ្ជាប់ជាមួយ Project៖

### IP Address Assignment Sheet

| Device | Interface | IP Address | Subnet | VLAN |
|--------|-----------|-----------|--------|------|
| R-EDGE | Gi0/0 | 203.0.113.2 | /30 | - |
| R-EDGE | Gi0/1 | 172.16.0.1 | /30 | - |
| CORE-SW1 | Vlan10 | 10.10.10.1 | /24 | 10 |
| CORE-SW1 | Vlan20 | 10.10.20.1 | /24 | 20 |
| CORE-SW1 | Vlan30 | 10.10.30.1 | /24 | 30 |
| SRV-CORE-BANKING | Fa0 | 10.10.20.10 | /24 | 20 |
| SRV-DNS | Fa0 | 10.10.20.30 | /24 | 20 |
| R-SR | Gi0/0 | 10.20.10.1 | /24 | - |
| R-SR | Se0/0/0 | 172.16.1.6 | /30 | - |

---

## ២១. Troubleshooting Tips

| បញ្ហា | ការដោះស្រាយ |
|-------|-------------|
| Ping មិនដើរ | Check `show ip interface brief`, `show ip route` |
| VLAN មិនធ្វើការ | Check trunk config, `show vlan brief` |
| DHCP មិនផ្តល់ IP | Check `ip helper-address` នៅ SVI |
| OSPF Neighbor មិនឡើង | Check network statement និង area ID |
| VPN មិន Up | Check `show crypto isakmp sa` និង Pre-shared Key |
| Serial Link Down | Check clock rate នៅ DCE side |

---

## ២២. ការនាំចេញ (Export)

### Export ជា PDF (Documentation):

1. File → Print → Save as PDF
2. Include: Logical Topology, Physical Topology, Device List

### Export Configuration:

1. Click Device → Config → **Export** button
2. Save ជា `.txt` file

---

## ២៣. Deliverables

កំណត់ File សម្រាប់ Submit៖

1. ✅ `Bank_Enterprise_Network.pkt` — Packet Tracer File
2. ✅ `Network_Topology.drawio` — Visio Diagram
3. ✅ `Enterprise_Network_Design_TCPIP.md` — Design Document
4. ✅ `PacketTracer_Lab_Guide.md` — This Lab Guide
5. ✅ `IP_Address_Sheet.xlsx` — IP Documentation
6. ✅ `Device_Configs/` — Folder of all router/switch configs

---

## ២៤. គន្លឹះសំខាន់ៗ

- **រក្សាទុកឯកសារជានិច្ច**: Ctrl+S រៀងរាល់ ៥ នាទី
- **ប្រើ Notes**: បន្ថែម Sticky Note នៅក្នុង Workspace ដើម្បីពន្យល់
- **Color Coding**: ប្រើពណ៌ខុសគ្នាសម្រាប់ VLAN ខុសគ្នា
- **Backup Config**: Copy Running-Config ជានិច្ច
- **Test បន្តិចម្តងៗ**: Configure ១ Device → Test → បន្តទៅ Device បន្ទាប់

---

**រៀបចំដោយ**: ក្រុមវិស្វករប្រព័ន្ធ
**កាលបរិច្ឆេទ**: ថ្ងៃទី ១៩ សីហា ២០២៦
**Packet Tracer Version**: 8.2+
