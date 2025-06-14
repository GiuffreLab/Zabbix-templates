# Zabbix Template Documentation: UniFi Switches

## Overview

This Zabbix template is tailored to monitor **UniFi Switches** via SNMP. It provides comprehensive visibility into CPU and memory usage, device metadata, interface-level statistics, and network throughput. It is suitable for both home lab and enterprise-grade deployments of UniFi-managed switches.

---

## Template Name

**`Unifi Switches`**

Assigned to group:
`Templates/Network devices`

---

## Key Features

### 🧠 System Monitoring

* **CPU Load Averages**

  * 1-minute: `.1.3.6.1.4.1.2021.10.1.3.1`
  * 5-minute: `.1.3.6.1.4.1.2021.10.1.3.2`
  * 15-minute: `.1.3.6.1.4.1.2021.10.1.3.3`
  * All tracked as floating point values

* **Memory Monitoring**

  * Total RAM: `.1.3.6.1.4.1.2021.4.5.0` (in kB)
  * Available RAM: `.1.3.6.1.4.1.2021.4.6.0` (in kB)

* **Uptime (Clean Format)**

  * Pulled from `sysUpTime.0`
  * Transformed into human-readable string via JavaScript preprocessing

* **Device Metadata (Linked to Inventory Fields)**

  * `sysName.0` → Hostname
  * `sysDescr.0` → Firmware / system description
  * `sysLocation.0` → Location
  * `sysContact.0` → Contact

---

## 🔍 Interface Discovery & Monitoring

This template performs **SNMP-based LLD** (low-level discovery) of active Ethernet interfaces:

* Filters:

  * `IFTYPE` = 6 (Ethernet)
  * `IFOPERSTATUS` = 1 (UP)

### For Each Discovered Interface:

* Inbound/Outbound traffic (`bps`, with change/sec and 8x multiplier)
* Inbound/Outbound discards (`packets`)
* Inbound/Outbound errors (`errors`)
* Aliases (`{#IFALIAS}`) are included for human-friendly labeling

---

### 📈 Graph Prototypes

* **Interface Traffic on {#IFDESCR} ({#IFALIAS})**

  * Green: Inbound
  * Blue: Outbound

* **Interface Errors on {#IFDESCR} ({#IFALIAS})**

  * Green: Inbound Errors
  * Blue: Outbound Errors
  * Orange: Inbound Discards
  * Red: Outbound Discards

---

## Macros

Define the following macros at the host level for proper SNMP access:

```text
{$SNMP_COMMUNITY} = public
{$SNMP_PORT}      = 161
```

---

## Tags

```yaml
class: network
target: switch
target: ubiquiti
```

These tags help categorize the template and its associated metrics.

---

## Use Cases

This template is ideal for:

* Monitoring performance and health of UniFi aggregation and access switches.
* Tracking port utilization and identifying link issues (errors/discards).
* Enriching Zabbix inventory with real-time metadata from UniFi hardware.
* Visualizing bandwidth and performance trends per port using native graph prototypes.

---

## Requirements

* SNMP enabled on UniFi switch (via controller or local config).
* Zabbix server must be able to reach the switch on UDP port 161.
* SNMP community string must be configured to match `{$SNMP_COMMUNITY}`.

---

## Notes

* This template does not define triggers by default—custom triggers can be added based on `ifInErrors`, `ifOutErrors`, or memory thresholds.
* Designed to work generically across UniFi switch models while still supporting interface aliasing for readability.
* LLD uses `IFALIAS` (OID: `.1.3.6.1.2.1.31.1.1.1.18`) to provide port names or labels from the controller config.

---
