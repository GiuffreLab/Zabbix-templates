# Zabbix Template Documentation: UniFi UDM Pro Max

## Overview

This Zabbix template is designed to monitor **UniFi UDM Pro Max routers** using SNMP. It provides detailed visibility into system performance, interface metrics, firmware details, uptime, and CPU load averages. It includes preconfigured items, triggers, graphs, and a custom dashboard.

While it was built for the **UniFi UDM Pro Max routers** it may work for other variations of the **UDM Router Platform**

---

## Template Name

**`Unifi UDM Pro Max`**

This template belongs to the group:
`Templates/Network devices`

---

## Key Features

### ✅ Core System Monitoring

* **CPU Load Averages**
  Monitors 1, 5, and 15 minute CPU averages via:

  * `.1.3.6.1.4.1.2021.10.1.3.1` (1 min)
  * `.1.3.6.1.4.1.2021.10.1.3.2` (5 min)
  * `.1.3.6.1.4.1.2021.10.1.3.3` (15 min)

* **Uptime Monitoring**

  * Raw uptime in ticks (`sysUpTime.0`)
  * Human-readable format (via JS preprocessing)
  * Includes a trigger: `Device Recently Rebooted` (if uptime < 15 min)

* **Hostname & Firmware Version**

  * Hostname from `sysName.0`
  * Firmware from `sysDescr.0` with regex to extract version

* **Interface Count**

  * SNMP OID: `.1.3.6.1.2.1.2.1.0`
    Reports the total number of interfaces

* **IP Packet Stats**

  * IP packets received (`ipInReceives.0`)
  * IP packets sent (`ipOutRequests.0`)

---

### 🔎 Interface Discovery & Monitoring

This template uses **SNMP walk-based low-level discovery (LLD)** to find active network interfaces with:

* Descriptions matching `eth`, `br`, `ath`, `honeypot`, `Annapurna`, `RealTek` etc.
* Interface type = 6 (Ethernet)
* Operational status = 1 (UP)

For each discovered interface, it monitors:

* Inbound & outbound traffic (bps, calculated from octets)
* Inbound & outbound errors (triggers if error count increases)
* Inbound & outbound discards (with warning trigger)
* MAC address
* Interface speed (bps)

---

### 🚨 Triggers

* **Inbound/Outbound Errors Detected**
  Triggers if error count changes on any interface.
* **Discards Detected**
  Triggers if discard counts change on any interface.
* **Device Recently Rebooted**
  Uptime trigger if below 900 seconds.

---

### 📊 Graphs

#### Static Graphs

* **CPU Load Averages**

  * Line graph with 1m, 5m, and 15m loads

#### Graph Prototypes (per interface)

* **Interface Traffic on {#IFDESCR}**
* **Interface Errors on {#IFDESCR}**

---

### 📋 Dashboard: `UDM Pro Max Monitoring`

Includes widgets for:

* Hostname
* Firmware Version
* Uptime (clean format)
* Interface Count
* CPU load (1m, 5m, 15m)
* Graphs for:

  * Interface Traffic
  * Interface Errors

All widgets are laid out logically for quick troubleshooting and capacity monitoring.

---

## Macros

These macros must be defined on the host:

```text
{$SNMP_COMMUNITY} = public
{$SNMP_PORT}      = 161
```

---

## Tags

These are used for classification and filtering within Zabbix:

```yaml
class: network
target: router
target: ubiquiti
```

---

## Use Cases

This template is ideal for:

* Home lab or SMB environments running UniFi UDM Pro Max routers.
* Anyone needing to track bandwidth, interface errors, CPU load, and device health.
* Dashboards tailored to display real-time device health.

---

## Requirements

* SNMP must be enabled on the UDM Pro.
* Zabbix server must have SNMP access (port 161/UDP).
* SNMP community string must match `{$SNMP_COMMUNITY}`.

---

## Notes

* Interface discovery is filtered to only include relevant, operational interfaces.
* CPU load graphs and uptime calculations use preprocessing for better clarity.
* Graphs use gradient lines and color-coding for readability.

---