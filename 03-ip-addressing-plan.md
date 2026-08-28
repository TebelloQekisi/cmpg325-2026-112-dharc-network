# IP Addressing Plan

**Assigned block:** `10.42.0.0/16` (65,534 usable host addresses)

The block is subnetted into five `/24` networks (one per VLAN) plus a `/30` for the WAN
point-to-point link. This uses only a small fraction of the assigned `/16`, leaving room for
future departments or services without re-addressing the existing network.

## Subnet Table

| VLAN | Name | Network | Usable Range | Broadcast | Gateway (SVI) |
|---|---|---|---|---|---|
| 10 | Administration | 10.42.10.0/24 | 10.42.10.1 – 10.42.10.254 | 10.42.10.255 | 10.42.10.1 |
| 20 | Archives & Research | 10.42.20.0/24 | 10.42.20.1 – 10.42.20.254 | 10.42.20.255 | 10.42.20.1 |
| 30 | Reading Room (Public) | 10.42.30.0/24 | 10.42.30.1 – 10.42.30.254 | 10.42.30.255 | 10.42.30.1 |
| 40 | Servers | 10.42.40.0/24 | 10.42.40.1 – 10.42.40.254 | 10.42.40.255 | 10.42.40.1 |
| 99 | Management | 10.42.99.0/24 | 10.42.99.1 – 10.42.99.254 | 10.42.99.255 | 10.42.99.1 |
| N/A | WAN (RTR-EDGE ↔ ISP) | 192.0.2.0/30 | 192.0.2.1 – 192.0.2.2 | 192.0.2.3 | n/a (point-to-point) |

> WAN addressing uses `192.0.2.0/24` (TEST-NET-1, RFC 5737) as a documentation stand-in for a real
> ISP-assigned public block, since the assigned 10.42.0.0/16 is a private range and cannot
> represent "the internet" side of the simulation.

## Static / Reserved Assignments

| Device | Interface | VLAN | IP Address |
|---|---|---|---|
| RTR-EDGE | Gi0/0 (WAN) | N/A | 192.0.2.1/30 |
| RTR-EDGE | Gi0/1 (trunk to CORE-L3SW) | N/A | no IP (trunk) |
| CORE-L3SW | Vlan10 (SVI) | 10 | 10.42.10.1/24 |
| CORE-L3SW | Vlan20 (SVI) | 20 | 10.42.20.1/24 |
| CORE-L3SW | Vlan30 (SVI) | 30 | 10.42.30.1/24 |
| CORE-L3SW | Vlan40 (SVI) | 40 | 10.42.40.1/24 |
| CORE-L3SW | Vlan99 (SVI, mgmt) | 99 | 10.42.99.1/24 |
| ACC-READSW | Vlan99 (mgmt) | 99 | 10.42.99.2/24 |
| WEB-SRV | NIC | 40 | 10.42.40.10/24 |
| Remote-Admin-PC | NIC (simulated public) | N/A | 192.0.2.2/30 |

Addresses `.2`–`.9` in each end-user VLAN (10, 20, 30) are reserved for future static devices
(e.g. network printers) and excluded from the DHCP pool.

## DHCP Pools

| Pool Name | VLAN | Range | Default Gateway | DNS | Lease Time |
|---|---|---|---|---|---|
| ADMIN-POOL | 10 | 10.42.10.10 – 10.42.10.200 | 10.42.10.1 | 10.42.40.10* | 24 h |
| ARCHIVE-POOL | 20 | 10.42.20.10 – 10.42.20.200 | 10.42.20.1 | 10.42.40.10* | 24 h |
| READING-POOL | 30 | 10.42.30.10 – 10.42.30.200 | 10.42.30.1 | 8.8.8.8 | 4 h |

\* Placeholder DNS entry pointing at the internal server; will be replaced if a dedicated DNS
service is added in a later milestone.

## VLSM Justification

A uniform `/24` per VLAN was chosen over variable-length subnetting because:

- Every department's realistic headcount (tens of staff) fits comfortably within a `/24` (254
  hosts), so VLSM would not save meaningfully usable address space here.
- Uniform `/24`s are simpler to explain and verify on the video demonstration, and reduce the risk
  of addressing/config errors under project time pressure.
- The `/16` block is large enough that address-space efficiency is not a real constraint for this
  project; scalability (see Non-Functional Requirements) is prioritised over conservation.
- The one exception is the WAN link, which correctly uses a `/30` (only 2 usable hosts needed),
  demonstrating VLSM understanding where it is actually appropriate.
