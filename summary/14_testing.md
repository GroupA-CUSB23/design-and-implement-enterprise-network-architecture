# ✅ Testing & Validation

[⬅ Back to Index](index.md)

---

## 🧪 Testing Categories

| Type | Purpose | Tool |
|------|---------|------|
| **Connectivity** | ត្រួតពិនិត្យការតភ្ជាប់ | ping, tracert |
| **Performance** | វាស់ Speed & Latency | iPerf, Speedtest |
| **Security** | រកឃើញភាពអាចវាយប្រហារ | nmap |
| **Application** | សាកល្បង Web/Email | Browser, Mail client |

---

## 🔌 Connectivity Testing

### Ping Test (បំផុតងាយ):

```bash
# ពី PC នៅ Staff (VLAN 10) ទៅ Server (VLAN 20)
C:\> ping 192.168.20.10

Reply from 192.168.20.10: bytes=32 time=2ms TTL=63
Reply from 192.168.20.10: bytes=32 time=1ms TTL=63
```

**លទ្ធផលរំពឹងទុក**: Reply ត្រឡប់មកវិញ = ✅ Success

---

### Traceroute (មើលផ្លូវ Packet):

```bash
C:\> tracert 8.8.8.8

1  192.168.10.1    (Gateway)
2  10.0.0.1        (Router)
3  203.0.113.1     (ISP)
4  ...
8  8.8.8.8         (Google DNS)
```

**មានប្រយោជន៍**: មើលឃើញ Packet ធ្វើដំណើរតាមផ្លូវណា។

---

### Show Commands (Cisco):

```cisco
Router# show ip route          ! មើល Routing Table
Router# show ip interface brief ! មើលស្ថានភាព Interface
Router# show running-config    ! មើល Configuration
Switch# show vlan brief        ! មើល VLAN List
Switch# show mac address-table ! មើល MAC Table
```

---

## ⚡ Performance Testing

### iPerf3 (វាស់ Bandwidth):

**នៅ Server**:
```bash
iperf3 -s
```

**នៅ Client**:
```bash
iperf3 -c 192.168.20.10
```

លទ្ធផល: បង្ហាញ Bandwidth ជាក់ស្តែង (Mbps)។

---

## 🔒 Security Testing

### Nmap (Port Scanner):

```bash
# ស្កេន Port នៅ Server
nmap 192.168.20.10

PORT     STATE   SERVICE
80/tcp   open    http
443/tcp  open    https
22/tcp   open    ssh
```

**ការវិភាគ**: ត្រូវ Close Port ដែលមិនប្រើ។

---

## 🌐 Application Testing

- ✅ **Web Browser** — សាកល្បង `http://www.google.com`
- ✅ **Email** — ផ្ញើ Test Email
- ✅ **File Sharing** — Access `\\server\share`
- ✅ **VPN** — ភ្ជាប់ពី Branch ទៅ HO

---

## 📋 Test Report Template

```
Test ID: TC-001
Test Name: Staff PC to Server Ping
Date: 2026-XX-XX
Tester: [ឈ្មោះ]

Steps:
1. Login to Staff PC
2. Open Command Prompt
3. Run: ping 192.168.20.10

Expected: 100% ping success, latency < 5ms
Actual:   ✅ PASS - Average 2ms

Notes: All packets received.
```

---

## ✅ Checklist ពេលបញ្ចប់

- [ ] គ្រប់ PC អាច Ping Gateway បាន
- [ ] គ្រប់ VLAN ដាច់ដោយឡែក (មិន Ping គ្នា)
- [ ] Inter-VLAN Routing ដំណើរការ (តាម ACL)
- [ ] Internet Access ដំណើរការ
- [ ] VPN ភ្ជាប់សាខាបានពេញលេញ
- [ ] Firewall Block Traffic ដែលមិនចង់បាន
- [ ] Server Services (Web, Email) ដំណើរការ
- [ ] Backup System តេស្តរួច

---

[⬅ Prev: Implementation](13_implementation.md) | [Index](index.md) | [Next: Conclusion ➡](15_conclusion.md)
