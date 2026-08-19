# 🎨 SVG Flow Diagrams

Visual flow diagrams for the Bank Enterprise Network Design project.

---

## 📋 Diagram List

| # | Diagram | Description | Used in |
|---|---------|-------------|---------|
| 01 | [TCP/IP Model](01_tcpip_model.svg) | 4-Layer Model with protocols | [03_tcpip_model.md](../../03_tcpip_model.md) |
| 02 | [Three-Tier Architecture](02_three_tier_architecture.svg) | Core / Dist / Access layers | [04_architecture.md](../../04_architecture.md) |
| 03 | [Logical Topology](03_logical_topology.svg) | Complete data flow topology | [05_logical_topology.md](../../05_logical_topology.md) |
| 04 | [Physical Topology](04_physical_topology.svg) | Data Center rack layout | [06_physical_topology.md](../../06_physical_topology.md) |
| 05 | [WAN Topology](05_wan_topology.svg) | HO + 5 Branches connectivity | [07_wan_topology.md](../../07_wan_topology.md) |
| 06 | [VLAN Design](06_vlan_design.svg) | 8 VLANs + Inter-VLAN routing | [09_vlan_design.md](../../09_vlan_design.md) |
| 07 | [IP Addressing](07_ip_addressing.svg) | Hierarchical IP scheme | [08_ip_addressing.md](../../08_ip_addressing.md) |
| 08 | [Defense in Depth](08_security_defense_in_depth.svg) | Multi-layer security | [12_security.md](../../12_security.md) |
| 09 | [HA Failover](09_ha_failover.svg) | Redundancy + Failover flow | [12_security.md](../../12_security.md) |
| 10 | [Transaction Flow](10_data_flow_transaction.svg) | Internet Banking end-to-end | [05_logical_topology.md](../../05_logical_topology.md) |
| 11 | [Implementation Timeline](11_implementation_timeline.svg) | 3-month Gantt chart | [13_implementation.md](../../13_implementation.md) |
| 12 | [TCP/IP Encapsulation](12_tcpip_encapsulation.svg) | Packet wrapping by layer | [03_tcpip_model.md](../../03_tcpip_model.md) |

---

## 🎨 Usage

### View in Browser
Open any `.svg` file directly in a web browser (Chrome, Firefox, Edge).

### Embed in Markdown
```markdown
![Description](assets/svg/01_tcpip_model.svg)
```

### Embed in HTML
```html
<img src="01_tcpip_model.svg" alt="TCP/IP Model" width="800"/>
```

### Convert to PNG
Use online tools or:
```bash
# Using Inkscape
inkscape input.svg --export-type=png --export-filename=output.png

# Using ImageMagick
magick convert input.svg output.png
```

### Edit
Open with:
- **Inkscape** (free, cross-platform)
- **Adobe Illustrator**
- **Figma** (import as SVG)
- **VS Code** (with SVG preview extension)

---

## 🎨 Design System

All diagrams use consistent colors:

| Color | Usage |
|-------|-------|
| 🔵 Blue (#2874A6) | Network Core / Distribution |
| 🟢 Green (#27AE60) | Access / Success / Active |
| 🔴 Red (#C0392B) | Security / Firewall / Critical |
| 🟠 Orange (#E67E22) | Warning / Standby / WAN |
| 🟣 Purple (#8E44AD) | Wireless / Cloud / MPLS |
| 🟡 Yellow (#F1C40F) | DMZ / Alert |
| ⚫ Dark Gray (#2C3E50) | Rack / Infrastructure |

---

**Format**: SVG (Scalable Vector Graphics) — sharp at any zoom level
**Total Size**: ~35 KB (all 12 diagrams combined)
**Editable**: XML-based, human-readable
