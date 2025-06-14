# Zabbix Template Documentation: UniFi Access Points

## Overview

This Zabbix template provides comprehensive SNMP-based monitoring for **Ubiquiti UniFi Access Points**, covering system health, interface status, wireless radio metrics, and per-SSID usage. It includes low-level discovery for SSIDs and virtual radios, alerts on high CPU or channel utilization, and visualizations for traffic, users, and performance trends.

---

## Template Name

**`Unifi Access Points`**

Assigned group:
`Templates/Network devices`

---

## Key Features

### 🧠 System Monitoring

* **CPU Metrics**

  * CPU usage via `.1.3.6.1.2.1.25.3.3.1.2.196608` (%)
  * Load averages for 1, 5, and 15 minutes

* **Memory / System Info**

  * System contact, location, and hostname linked to inventory
  * Uptime (`sysUpTime.0`) converted to human-readable format
  * Firmware version and hardware model pulled via UniFi OIDs

* **Device Identity**

  * MAC address, IP address, and system model are all tracked and tied to inventory fields

---

## 📶 Wireless Interface Monitoring

* **Radio Channel Utilization**

  * 2.4GHz (BGN): `unifiRadioCuTotal.1`

  * 5GHz (AC): `unifiRadioCuTotal.2`

  * 6GHz (AX): `unifiRadioCuTotal.4` (may report 100% if idle)

  > Triggers alert if utilization hits 80% (warning) or 90% (average)

* **Channel Numbers**

  * 2.4GHz: `unifiVapChannel.1`
  * 5GHz: `unifiVapChannel.5`
  * 6GHz: `unifiVapChannel.7`

---

## 📡 Interface Traffic & Errors

* **LAN Port Monitoring**

  * Inbound/outbound bytes (`Bps`, preprocessed per second)
  * Inbound/outbound errors
  * Interface speed (Mbit/s)

* **SSID / Virtual Interface Discovery**

  * SSIDs discovered using:

    ```text
    discovery[{#UNIFIVAPESSID},.1.3.6.1.4.1.41112.1.6.1.2.1.6, {#UNIVAPRADIO},.1.3.6.1.4.1.41112.1.6.1.2.1.9]
    ```
  * Filters exclude test or hidden SSIDs (e.g., `element-*`, `yorkscan`)

### For Each SSID and Radio:

* Number of connected users
* Traffic in/out (`Bps`)
* Errors in/out (`errors/sec`)
* Channel by radio band

---

## 📊 Graphs

### Static Graphs

* **Channel Utilization**
  Compares 2.4GHz vs 5GHz usage
* **CPU Load & Utilization**

  * Includes both absolute load (1/5/15min) and percentage
* **Network Traffic**

  * LAN traffic in/out shown as stacked graph

### Graph Prototypes (Per SSID & Radio)

* **Connected Users per SSID**
  Stacked view for user counts across radios

---

## Triggers

* **CPU Load**

  * > 50% (average priority)
  * > 75% (high priority)
* **Channel Utilization**

  * > 80% on 2.4G or 5G (warning)
  * > 90% (average)

---

## Macros

These must be set on the host:

```text
{$SNMP_COMMUNITY} = publicy
{$SNMP_PORT}      = 161
```

---

## Tags

```yaml
class: network
target: access point
target: ubiquiti
```

Used for template organization and filtering.

---

## Use Cases

* Monitor **WiFi congestion** and utilization by band.
* Track **SSID usage**, user load, and signal health.
* Alert on **CPU overuse**, radio oversubscription, or traffic errors.
* Visualize **multi-band usage** for planning upgrades or tuning channel plans.
* Combine with switch/UDM templates for a full UniFi infrastructure view.

---

## Requirements

* SNMP must be enabled and accessible on the AP.
* UniFi controller should expose required OIDs (usually via SNMP v2).
* Zabbix must be able to query UDP port 161.

---

## Notes

* SSID LLD items like "Users" and "Traffic" are generated per SSID/radio combo.
* Some items (e.g., `System Time`) are present but disabled.
* Filtering prevents test SSIDs from cluttering discovery output.

---