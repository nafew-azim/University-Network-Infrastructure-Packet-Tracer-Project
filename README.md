# University Network Infrastructure — Packet Tracer Project 🚦

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](#license--author) [![Packet Tracer](https://img.shields.io/badge/Tool-Cisco%20Packet%20Tracer-orange.svg)](#how-to-open--test) ![Status](https://img.shields.io/badge/Status-Completed-green.svg)

> **Elegant campus-scale network** simulation for learning and demonstration: VLANs, DHCP, DNS, HTTP, WAN serial links, dynamic routing (OSPF/EIGRP), inter-VLAN routing, and an extensible exports/automation workflow.

---

## Table of Contents

* [Project Overview](#project-overview)
* [Quick Demo](#quick-demo)
* [Repository Structure](#repository-structure)
* [Features & Services](#features--services)
* [Logical Topology (visuals & files)](#logical-topology-visuals--files)
* [Addressing & VLAN Plan (template)](#addressing--vlan-plan-template)
* [Automated Validation Scripts (suggested)](#automated-validation-scripts-suggested)
* [How to Open & Test (step-by-step)](#how-to-open--test-step-by-step)
* [Troubleshooting & Notes](#troubleshooting--notes)
* [Contributing & PR Guide](#contributing--pr-guide)
* [Changelog](#changelog)
* [License & Author](#license--author)

---

## Project Overview

This repository contains a Cisco Packet Tracer topology that models a **university network** spanning four campuses (schools). The objective is to provide a realistic lab environment where you can:

* design hierarchical IP addressing and VLAN separation
* configure DHCP, DNS, and HTTP services
* implement inter-campus connectivity using serial WAN links
* practice dynamic routing (OSPF/EIGRP) and inter-VLAN routing
* export device configs for documentation and automation

**Intended audience:** undergraduate networking students, instructors, and lab demonstrators.

---

## Quick Demo

*Open the Packet Tracer file and try this quick flow:*

1. Set a PC in Campus A to **DHCP** → confirm it receives an address.
2. From the same PC, open the web browser and navigate to `http://webserver.university.local`.
3. Ping a PC in Campus B to verify routing.

> Short video/gif demo placeholder: `docs/demo.gif` (add an exported recording for the README).

---

## Repository Structure

```
/ (repo root)
├─ 438-project.pkt            # Packet Tracer topology file (main)
├─ README.md                 # This file (rich & detailed)
├─ docs/
│  ├─ topology.png           # exported topology diagram (add manually)
│  ├─ banner.png             # top banner used by README
│  ├─ demo.gif               # short demo recording (optional)
│  └─ screenshots/           # any supporting screenshots
└─ exports/
   └─ configs/               # exported device CLI configs (one file per device)
```

**Note:** Add exported configs to `exports/configs/` (Packet Tracer → File → Export → Config). Naming convention: `hostname.running-config.txt`.

---

## Features & Services

* **Hierarchical addressing** with per-campus subnets and optional departmental VLANs.
* **DHCP**: centralized or per-campus server with excluded-address ranges.
* **DNS**: internal name resolution via DNS Server or static `ip host` entries.
* **HTTP**: example web server hosting a small site (Packet Tracer Server HTTP service).
* **Dynamic routing**: OSPF (recommended) or EIGRP for inter-campus route exchange.
* **WAN links**: Serial connections between campus routers to simulate real-world backbone.
* **Inter-VLAN routing**: Router-on-a-stick or L3 switch implementation.
* **Optional ACLs and NAT** for access control and edge scenarios.

---

## Logical Topology (visuals & files)

* Please add an exported `docs/topology.png` screenshot from Packet Tracer for a visual reference.

**Topology summary:**

* Edge/Core Router connects to an external simulated Internet node.
* Four campus routers (R-A, R-B, R-C, R-D) each serve a campus LAN with one or more VLANs.
* Central services (DNS/DHCP/HTTP) are hosted in a DMZ or central services VLAN.
* Serial links connect campus routers to the core/edge network.

---

## Addressing & VLAN Plan (template)

Use or replace the following plan with values from your topology. When you export configs, I can auto-fill this table.

| Campus          | VLAN ID | Network    | CIDR | Gateway    | DHCP Range                |
| --------------- | ------: | ---------- | ---: | ---------- | ------------------------- |
| Engineering (A) |      10 | 10.10.10.0 |  /24 | 10.10.10.1 | 10.10.10.100–10.10.10.200 |
| Science (B)     |      20 | 10.10.20.0 |  /24 | 10.10.20.1 | 10.10.20.100–10.10.20.200 |
| Arts (C)        |      30 | 10.10.30.0 |  /24 | 10.10.30.1 | 10.10.30.100–10.10.30.200 |
| Business (D)    |      40 | 10.10.40.0 |  /24 | 10.10.40.1 | 10.10.40.100–10.10.40.200 |
| Services (DMZ)  |      50 | 10.10.50.0 |  /24 | 10.10.50.1 | N/A (static servers)      |

**Static IPs**: Reserve .1–.50 for infrastructure and servers; DHCP can begin at .100.

---

## Sample Configuration Snippets (copy & adapt)

> These snippets are intentionally human-friendly and ready to paste into Packet Tracer device CLIs.

### 1) Router — LAN interface and basic static route

```bash
hostname R-A
interface GigabitEthernet0/0
 description LAN to Campus A
 ip address 10.10.10.1 255.255.255.0
 no shutdown
!
interface Serial0/0/0
 description WAN to Core
 ip address 192.168.100.1 255.255.255.252
 no shutdown
!
ip route 0.0.0.0 0.0.0.0 192.168.100.2
```

### 2) DHCP pool on a router (or configure via Server GUI)

```bash
ip dhcp excluded-address 10.10.10.1 10.10.10.50
ip dhcp pool CAMPUS_A
 network 10.10.10.0 255.255.255.0
 default-router 10.10.10.1
 dns-server 10.10.50.10
 domain-name university.local
```

### 3) OSPF sample

```bash
router ospf 1
 router-id 1.1.1.1
 network 10.10.10.0 0.0.0.255 area 0
 network 10.10.20.0 0.0.0.255 area 0
 network 192.168.100.0 0.0.0.3 area 0
```

### 4) Inter-VLAN (Router-on-a-stick) example

```bash
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 10.10.10.1 255.255.255.0
!
interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 10.10.20.1 255.255.255.0
```

### 5) DNS static hosts

```bash
ip host webserver.university.local 10.10.50.10
ip host fileserver.university.local 10.10.50.11
```

---

## Automated Validation Scripts (suggested)

Add simple scripts to `scripts/` to sanity-check the topology by using exported configs or manual steps. Example ideas:

* `validate_dhcp.sh` — ping/check DHCP on sample IPs (requires network access or emulated environment)
* `parse_configs.py` — extract IPs, hostnames, and generate addressing tables from exported `running-config` files

**Example: parse\_configs.py (concept)**

```python
# iterate exported configs -> find `interface`, `ip address`, `hostname`, `ip dhcp pool`
# output CSV with device, interface, ip, mask
```

If you upload exported `exports/configs/*.txt` I can implement `parse_configs.py` and generate a filled Addressing table.

---

## How to Open & Test (step-by-step)

1. Install **Cisco Packet Tracer** (v8.x recommended). Use the latest supported version for compatibility.
2. Open `438-project.pkt`.
3. Visual inspection: expand the workspace and locate core routers, campus routers, and server nodes. Export a topology screenshot to `docs/topology.png`.
4. For each Router:

   * Click → CLI → `enable` → `show running-config` / `show ip interface brief`.
5. Test DHCP:

   * On a PC: Desktop → IP Configuration → select DHCP → verify assigned IP.
6. Test DNS:

   * From a PC: `ping webserver.university.local` or open browser and use FQDN.
7. Test routing:

   * From a PC in Campus A: `ping` a PC in Campus B.
   * On routers: `show ip route`, `show ip ospf neighbor`, `show ip eigrp neighbors`.
8. Test HTTP:

   * On a PC: open browser → `http://10.10.50.10` (or the DNS name).

---

## Lab Exercises & Tasks (suggested)

1. **DHCP Recovery**: simulate DHCP server failure — configure a backup DHCP or switch to local DHCP on router.
2. **Access Control**: create ACLs on campus edge to restrict traffic to the web server except HTTP.
3. **OSPF Tuning**: adjust area design (include a stub area) and measure convergence time after a link flap.
4. **VLAN Expansion**: add a new department VLAN to Campus B and update DHCP and routing.
5. **Monitoring**: configure SNMP community strings and practice polling using Packet Tracer’s SNMP tools.

---

## Validation Checklist (detailed)

* [ ] Topology diagram present in `docs/topology.png`.
* [ ] Device configs exported to `exports/configs/`.
* [ ] DHCP pools configured and clients receive leases.
* [ ] DNS resolves FQDN -> IP.
* [ ] OSPF/EIGRP neighbors established for all inter-router links.
* [ ] Inter-campus ping tests successful.
* [ ] HTTP server accessible from every campus.
* [ ] No overlapping subnets.
* [ ] Management VLAN and switch IPs configured.

---

## Troubleshooting & Notes

* Packet Tracer stores configs in a proprietary format; automated extraction from `.pkt` may not yield all CLI lines — export running configs for precise documentation.
* If `ip helper-address` is missing on SVIs, DHCP requests will not reach centralized DHCP server.
* For OSPF adjacency problems: check `whiteboard` and interface MTU, area assignment, and passive-interface settings.
* When testing, prefer `show ip route` and `show ip arp` to debug learning issues.

---

## Contributing & PR Guide

* Fork the repo → make changes → create PR against `main`.
* Include in PR description: summary, test steps, and screenshots (if relevant).
* Add exported device configs to `exports/configs/` (one file per device) and name them `hostname.running-config.txt`.

**Issue templates (suggestion):**

* `bug_report.md` — describe problem, Packet Tracer version, steps to reproduce.
* `feature_request.md` — describe desired feature and motivation.

---

## Changelog

* **v1.0** — Initial Packet Tracer topology and README (this repo)
* **v1.1** — README improved with templates and sample configs (you can update this line when you add exports)

---

## License

This project is intended for educational use. If you want to allow broad reuse, add an `LICENSE` file (MIT recommended).

---
