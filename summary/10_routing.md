# 🛣️ Routing Protocols

[⬅ Back to Index](index.md)

---

## 📋 ប្រភេទ Routing

| ប្រភេទ | ការពិពណ៌នា | ការប្រើ |
|--------|-------------|---------|
| **Static Route** | Admin កំណត់ដោយដៃ | Small Network |
| **RIP** | Simple Dynamic | Very Small Network |
| **OSPF** | Fast, Scalable | Enterprise (យើងប្រើ) |
| **BGP** | Internet Routing | ISP Only |

---

## 🔵 Static Route (ងាយបំផុត)

**គោលការណ៍**: Admin កំណត់ Route ដោយដៃ។

### Configuration Example:
```cisco
Router(config)# ip route 192.168.20.0 255.255.255.0 10.0.0.2
```

**មានន័យថា**: បើ Packet ចង់ទៅ 192.168.20.0/24, ត្រូវផ្ញើតាម 10.0.0.2។

**គុណសម្បត្តិ**៖ ងាយ, គ្មាន Overhead
**គុណវិបត្តិ**៖ ត្រូវប្តូរដោយដៃពេលមានការផ្លាស់ប្តូរ

---

## 🟢 OSPF (Dynamic Routing)

**OSPF (Open Shortest Path First)** = ស្តង់ដារ Dynamic Routing ដ៏ពេញនិយម។

### លក្ខណៈ:
- ✅ Auto-discovery (រកឃើញ Neighbor ដោយស្វ័យប្រវត្តិ)
- ✅ Fast Convergence (ស្គាល់ការផ្លាស់ប្តូរបានលឿន)
- ✅ Loop-free (គ្មាន Routing Loop)
- ✅ Open Standard (មិនកំណត់ Vendor)

### Configuration Example:
```cisco
Router(config)# router ospf 1
Router(config-router)# network 192.168.10.0 0.0.0.255 area 0
Router(config-router)# network 10.0.0.0 0.0.0.255 area 0
```

---

## 🌐 Default Route

**Default Route** = Route "ចុងក្រោយ" បើគ្មាន Route ណាមួយ match។

```cisco
Router(config)# ip route 0.0.0.0 0.0.0.0 <ISP_Gateway_IP>
```

**ការប្រើ**: បើ Packet មិនស្គាល់គោលដៅ → ផ្ញើទៅ Internet Router។

---

## 📊 Routing Table (ឧទាហរណ៍)

```
Router# show ip route

C   192.168.10.0/24 is directly connected, Vlan10
C   192.168.20.0/24 is directly connected, Vlan20
S   192.168.20.0/24 [1/0] via 10.0.0.2      ← Static Route
O   192.168.30.0/24 [110/2] via 10.0.0.6   ← OSPF
S*  0.0.0.0/0 [1/0] via 203.0.113.1        ← Default Route
```

**កូដ**:
- **C** = Connected (ភ្ជាប់ផ្ទាល់)
- **S** = Static
- **O** = OSPF
- **S\*** = Default Static

---

## 💡 សម្រាប់និស្សិត CCNA

- 🎯 **Static Route** — ត្រូវរៀនមុនគេ
- 🎯 **OSPF Single Area (Area 0)** — Standard
- 🎯 **Show Commands**: `show ip route`, `show ip protocols`
- 🎯 **Verify**: `ping`, `traceroute`

---

[⬅ Prev: VLAN Design](09_vlan_design.md) | [Index](index.md) | [Next: Equipment ➡](11_equipment.md)
