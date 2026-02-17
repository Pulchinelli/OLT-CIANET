# Zabbix 7 Template — Cianet GPON OLT G8PS X by SNMP

[![Zabbix 7.0](https://img.shields.io/badge/Zabbix-7.0-red?logo=zabbix)](https://www.zabbix.com/)
[![SNMP v2c](https://img.shields.io/badge/SNMP-v2c-blue)](https://en.wikipedia.org/wiki/Simple_Network_Management_Protocol)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

SNMP monitoring template for **Cianet G8PS X / G8PS X V2** GPON OLTs on **Zabbix 7.x**.

Built from a real `snmpwalk` on a production OLT — Enterprise OID `1.3.6.1.4.1.17409` (Cianet Indústria e Comércio S/A, Brazilian manufacturer).

> **Tested and validated in production** with 26 active ONUs, 5 PON ports in use, 3 uplinks, firmware V1.1.0_201201 with Zabbix 7 only (not tested in other versions).

---

## 📋 Features

### Fixed Items (System)

| Metric | OID | Description |
|---|---|---|
| sysDescr | `1.3.6.1.2.1.1.1.0` | System description (OLT model) |
| sysName | `1.3.6.1.2.1.1.5.0` | OLT hostname |
| sysLocation | `1.3.6.1.2.1.1.6.0` | Configured location |
| sysContact | `1.3.6.1.2.1.1.4.0` | Contact information |
| sysUpTime | `1.3.6.1.2.1.1.3.0` | Uptime in centiseconds |
| sysObjectID | `1.3.6.1.2.1.1.2.0` | Vendor object ID |
| Cianet Hostname | `...17409.2.3.1.2.1.1.2.1` | OLT configured hostname |
| Model | `...17409.2.3.1.2.1.1.3.1` | Hardware model |
| Serial Number | `...17409.2.3.1.1.13.0` | Serial number |
| Firmware | `...17409.2.3.1.3.1.1.8.1.0` | Firmware version |
| Management IP | `...17409.2.3.1.1.2.0` | Inband management IP |
| MAC Address | `...17409.2.3.1.1.6.0` | Inband MAC address |
| ICMP ping | Simple Check | ICMP availability |
| ICMP loss | Simple Check | Packet loss (%) |
| ICMP response time | Simple Check | Round-trip time (s) |
| SNMP availability | Internal | SNMP agent status |

### Discovery Rules (LLD)

| # | Discovery Rule | Discovers | Items per entity |
|---|---|---|---|
| 1 | **Uplink interfaces** | ge0/0/X, xge0/0/X | Status, in/out traffic, speed |
| 2 | **PON ports** | pon0/0/1 to pon0/0/8 | Authorized ONUs, max ONUs, total/allocated/free BW, status, SFP temperature, TX power, bias current, in/out traffic |
| 3 | **ONU interfaces** | pon0/0/X:Y | Status, in/out traffic |
| 4 | **Power supplies** | power card 1, 2 | PSU status |
| 5 | **Fans** | fan card 1, 2, 3 | Fan status |

### Triggers

| Trigger | Severity | Condition |
|---|---|---|
| OLT rebooted | ⚠️ Warning | Uptime < 10 minutes |
| OLT unreachable (ICMP) | 🔴 High | 3 consecutive failed pings |
| High ICMP loss | ⚠️ Warning | Loss > `{$ICMP_LOSS_WARN}`% |
| High ICMP RTT | ⚠️ Warning | RTT > `{$ICMP_RESPONSE_TIME_WARN}`s |
| SNMP unavailable | ⚠️ Warning | No SNMP data for `{$SNMP.TIMEOUT}` |
| Firmware changed | ℹ️ Info | Firmware value changed |
| Uplink DOWN | 🟠 Average | Uplink state changed to DOWN |
| PON DOWN (with ONUs) | 🔴 High | PON port down with authorized ONUs |
| SFP temperature high | ⚠️ Warning | Temp > `{$PON.SFP.TEMP.MAX}`°C |
| ONU offline | 🟠 Average | ONU changed to DOWN |
| PSU failure | 🔴 High | Status ≠ Normal |
| Fan failure | 🟠 Average | Status ≠ Normal |

### Value Maps

- `ifOperStatus` — up / down / testing / unknown / dormant / notPresent / lowerLayerDown
- `zabbix.host.available` — not available / available / unknown
- `Service state` — Down / Up
- `Cianet PSU Status` — Normal / Failure / Absent
- `Cianet Fan Status` — Normal / Failure / Absent

---

## 📁 Repository Structure

```
├── CIANET_GPON_OLT_G8PSX_SNMP.yaml   # Zabbix 7 template
├── README.md                           # Portuguese README
├── README.en.md                        # This file (English)
└── LICENSE                             # MIT License
```

---

## ⚙️ Requirements

- **Zabbix Server/Proxy** 7.x
- **Cianet G8PS X OLT** (or G8PS X V2) with:
  - SNMP v2c enabled
  - Read community configured
- **UDP/161** connectivity between Zabbix and the OLT

---

## 🚀 Installation

### 1. Import the Template

1. In Zabbix frontend: **Data collection → Templates → Import**
2. Select `CIANET_GPON_OLT_G8PSX_SNMP.yaml`
3. Under Rules, check:
   - ✅ Templates: Create new + Update existing
   - ✅ Items, Triggers, Discovery rules, Value maps
4. Click **Import**

### 2. Create the Host

1. **Data collection → Hosts → Create host**
2. Fill in:
   - **Host name:** `OLT-CIANET` (or your desired name)
   - **Groups:** `OLTs Cianet` (or your group)
   - **Templates:** `Cianet OLT G8PS X by SNMP`
3. In the **Interfaces** tab, add an SNMP interface:
   - **IP:** OLT management IP (e.g., `192.168.1.100`)
   - **Port:** `161`
   - **SNMP version:** `SNMPv2`
   - **Community:** `{$SNMP_COMMUNITY}`
4. (Optional) Override macros in the **Macros** tab for your environment

---

## 🔧 Macros

| Macro | Default | Description |
|---|---|---|
| `{$SNMP_COMMUNITY}` | `@read-write-community` | OLT SNMP community |
| `{$SNMP.TIMEOUT}` | `5m` | SNMP unavailability timeout |
| `{$ICMP_LOSS_WARN}` | `20` | ICMP loss threshold (%) |
| `{$ICMP_RESPONSE_TIME_WARN}` | `0.15` | ICMP latency threshold (s) |
| `{$PON.SFP.TEMP.MAX}` | `65` | Maximum SFP temperature (°C) |

> 💡 **Tip:** Override `{$SNMP_COMMUNITY}` at host level for OLTs with different communities.

---

## 📊 Metrics in Production

On a typical G8PS X V2 OLT with 5 active PONs and ~25 ONUs:

| Component | Count | Metrics generated |
|---|---|---|
| Fixed items | 16 | 16 |
| Uplinks (6x) | 4 items/uplink | ~24 |
| PON ports (8x) | 11 items/PON | ~88 |
| ONUs (~25x) | 3 items/ONU | ~75 |
| PSUs (2x) | 1 item/PSU | 2 |
| Fans (3x) | 1 item/fan | 3 |
| **Total** | | **~208 metrics** |

---

## 📡 Cianet Enterprise OIDs (Reference)

```
1.3.6.1.4.1.17409                          # Cianet Enterprise OID
├── 2.3.1.1.*                               # System info (IP, MAC, serial, VLAN)
├── 2.3.1.2.*                               # Device info (hostname, model, uptime)
├── 2.3.1.3.*                               # Card info (firmware, serial)
├── 2.3.1.4.*                               # Power supply table
├── 2.3.1.5.*                               # Fan table
├── 2.3.2.1.*                               # Uplink port table (ge/xge)
├── 2.3.2.2.*                               # LAG table
├── 2.3.3.1.*                               # PON port table (ONUs, BW, status)
├── 2.3.3.5.*                               # PON optical (temp, TX, bias)
├── 2.3.7.*                                 # VLAN configuration
└── 2.3.9.*                                 # ONU detail/performance
```

---

## ⚠️ Limitations

- This template was developed and tested specifically for **Cianet G8PS X V2**. Other Cianet OLTs (e.g., G16PS, G24PS) may use different OIDs.
- For environments with hundreds of ONUs, consider increasing LLD intervals or disabling ONU-level discovery.
- The YAML file is indentation-sensitive — edit carefully.
- Host Resources MIB (`1.3.6.1.2.1.25`) is **not supported** by this equipment.

---

## 🤝 Contributing

Issues and Pull Requests are welcome! Suggestions:

- OID adjustments for other Cianet firmware versions or models
- Dashboard or graph additions
- Trigger and threshold optimization
- Translations to other languages

---

## 📜 License

This project is licensed under the [MIT License](LICENSE).

---

## 🙏 Credits

- Developed by **Rafael S. Pulchinelli** — Network Analyst (Sao Paulo, Brazil)
- Based on real production data (complete snmpwalk on Enterprise MIB 17409)
