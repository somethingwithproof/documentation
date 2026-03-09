# Automation

Cacti's Automation feature performs network discovery and automatic device/graph
creation. It periodically scans configured subnets, identifies responsive hosts,
and creates Cacti devices and graphs without manual intervention.

## Automation sub-topics

- [Automation Networks](Automation-Networks.md) — configure subnets for
  periodic scanning, discovery threads, and schedule types

## Overview

Automation relies on SNMP and ICMP to identify devices. When a new host is
found, Cacti matches it against configured **Rules** and creates the device
entry and associated graphs automatically.

Key components:

- **Networks** — define the IP ranges to scan and the scan schedule
- **Device Rules** — determine what Cacti device template to assign to a
  discovered host based on its SNMP sysDescr or sysObjectID
- **Graph Rules** — determine which graphs to create for each new device

Access automation from `Console > Automation`.

---
Copyright (c) 2004-2026 The Cacti Group
