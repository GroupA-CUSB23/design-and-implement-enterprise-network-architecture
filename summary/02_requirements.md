# 📊 ការវិភាគតម្រូវការ (Requirements Analysis)

[⬅ Back to Index](index.md)

---

## 🎯 Business Requirements (តម្រូវការអាជីវកម្ម)

| # | តម្រូវការ | ការពិពណ៌នា |
|---|----------|-------------|
| 1 | ភ្ជាប់ Head Office និងសាខាទាំង ៣ | បុគ្គលិកអាចផ្ញើ File ឬ Email គ្នា |
| 2 | គាំទ្រ ATM | ATM ត្រូវតភ្ជាប់ជាមួយ Server ធនាគារ |
| 3 | Internet Access | បុគ្គលិកអាចប្រើ Web ដើម្បីធ្វើការ |
| 4 | ការពារទិន្នន័យ | មិនអោយ Hacker ចូលបានពី Internet |

---

## ⚙️ Technical Requirements (តម្រូវការបច្ចេកទេស)

| Metric | តម្រូវការ |
|--------|-----------|
| **Bandwidth (Internet)** | យ៉ាងតិច 50 Mbps |
| **Bandwidth (WAN Branch)** | យ៉ាងតិច 10 Mbps |
| **Latency (Ping)** | តិចជាង 100 ms |
| **Uptime** | ធ្វើការបានយ៉ាងតិច 99% |
| **Security** | Firewall + Password protection |

---

## 👥 ចំនួន Devices

| ប្រភេទ | ចំនួន |
|--------|-------|
| PC (Desktop) | 115 |
| ATM | 5 |
| Printer | 10 |
| IP Phone | 30 |
| **Total** | **~160** |

---

## 📋 គោលការណ៍ (Standards)

- **TCP/IP Model** — ជាមូលដ្ឋានសម្រាប់ការទំនាក់ទំនង
- **Private IP** — ប្រើ 192.168.x.x ឬ 10.x.x.x (RFC 1918)
- **VLAN** — បំបែក Traffic សម្រាប់សុវត្ថិភាព
- **Static Route + OSPF** — សម្រាប់ Routing

---

[⬅ Prev: Scenario](01_scenario.md) | [Index](index.md) | [Next: TCP/IP Model ➡](03_tcpip_model.md)
