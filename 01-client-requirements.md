# Client Requirements Analysis

**Project ID:** CMPG325-2026-112 &nbsp;|&nbsp; **Client ID:** CLI-112
**Client:** Dikgosi History & Archives Research Centre (Mafikeng) &nbsp;|&nbsp; **Industry:** Research

## 1. Client Background

The Dikgosi History & Archives Research Centre is a research organisation that catalogues, digitises,
and provides access to historical records. Based on the assigned brief, the network must support
research/administrative staff and a public-facing reading room, while hosting an internal web
service used for cataloguing/archive access.

## 2. Stakeholders Identified

| Group | Description | Network Need |
|---|---|---|
| Administration | Finance, HR, front-office staff | General LAN + internet access |
| Archives & Research Staff | Historians, archivists, digitisation staff | LAN access + access to internal web portal (VLAN 40) |
| Reading Room (Public) | Walk-in researchers/patrons using public terminals | Isolated, internet-only access; **no** access to staff/server VLANs |
| Off-site Administrator (CR9) | Network administrator working remotely | Secure (SSH) remote management of routers/switches only |

> **Assumption:** the brief does not specify exact headcounts. Two representative hosts per
> department are modelled in Packet Tracer; each VLAN's `/24` subnet leaves headroom to scale to
> the real staff complement without re-addressing.

## 3. Functional Requirements

1. Full IP connectivity between all authorised departments, using the assigned block **10.42.0.0/16**.
2. An internal HTTP/web server (assigned technical challenge) hosting an internal portal,
   reachable by Administration and Archives/Research staff. Not exposed to the public internet,
   since the challenge is explicitly *internal* web service hosting.
3. Logical segmentation (VLANs) so that public Reading Room traffic cannot reach staff or server
   subnets.
4. Secure remote management access for **one off-site administrator** (CR9): SSH only, restricted
   by source address, with Telnet disabled on all devices.
5. DHCP for end-user subnets (Admin, Archives, Reading Room); static addressing for
   infrastructure and the server.
6. A working, testable Cisco Packet Tracer implementation demonstrating the above.

## 4. Non-Functional Requirements

- **Availability:** No network downtime is tolerated during month-end processing (Design
  Constraint, §8 of brief), addressed through link redundancy on the internal distribution-access
  path (see `02-network-design-decisions.md`).
- **Security:** Management access restricted and encrypted (SSH, ACLs); public and staff traffic
  kept in separate broadcast domains.
- **Scalability:** `/16` addressing block leaves substantial room for future departments/services.
- **Maintainability:** Clear VLAN-to-department mapping, consistent addressing scheme, documented
  design decisions (this repository).

## 5. Out of Scope / Assumptions

- Full WAN/ISP-side redundancy (dual ISP links, HSRP) is **not** implemented; the brief's downtime
  constraint is addressed at the internal LAN layer, where cabling/port failures are statistically
  the most common cause of local outages. Noted as a recommendation for future work.
- The internal web server is not NAT-exposed to the internet; only the off-site administrator's
  management traffic (CR9) traverses the WAN link.
- Exact departmental headcounts, floor plan, and physical cabling lengths are not specified in the
  brief and have been assumed for the purposes of this simulation.

## 6. Traceability to Brief

| Brief Item | Where Addressed |
|---|---|
| §6 Client Requirements | This document |
| §7 Network Design Requirements | `02-network-design-decisions.md`, diagrams |
| §8 Design Constraint (no downtime) | `02-network-design-decisions.md` §3 |
| §9 Assigned Challenge (HTTP/Web Server) | `02-network-design-decisions.md` §4 |
| §10 CR9 (remote management) | `02-network-design-decisions.md` §5 |
| Addressing block 10.42.0.0/16 | `03-ip-addressing-plan.md` |
