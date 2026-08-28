# Network Design Decisions

This document explains and justifies the design choices shown in `diagrams/physical-topology.svg`
and `diagrams/logical-topology.svg`.

## 1. Topology Choice: Collapsed Core with Redundant Access Trunk

A three-device design was chosen rather than a single router+switch pair:

- **RTR-EDGE** (edge router): the only device facing the WAN. Handles NAT/PAT for outbound
  internet access and terminates the secure remote-management path (CR9).
- **CORE-L3SW** (multilayer switch): performs **inter-VLAN routing** via SVIs instead of a
  router-on-a-stick sub-interface design. This keeps all local (VLAN-to-VLAN) traffic off the
  WAN-facing router entirely and scales better if more VLANs are added later.
- **ACC-READSW** (access switch): a physically and logically separate access switch dedicated to
  the public Reading Room, so public patron traffic never shares an access device with
  staff/archive traffic.

**Why not a single flat switch?** Segregating the public reading room onto its own access switch
makes the "no access to staff/server VLANs" requirement easier to enforce and easier to explain
during the video demonstration; the isolation is visible at the physical layer, not just in VLAN
config.

## 2. VLAN Segmentation

| VLAN | Name | Rationale |
|---|---|---|
| 10 | Administration | Separate broadcast domain for admin/finance traffic |
| 20 | Archives & Research | Staff who need access to the internal web portal |
| 30 | Reading Room (Public) | Isolated: public/transient users, no internal access |
| 40 | Servers | Hosts the internal HTTP server; reachable only from VLANs 10/20 |
| 99 | Management | Switch/router management interfaces, SSH-only |

Inter-VLAN access control (which VLANs can reach VLAN 40, for example) is implemented with
router/switch ACLs at configuration time (Milestone 2); the VLAN boundaries themselves are the
first layer of that control.

## 3. Addressing the "No Downtime During Month-End Processing" Constraint (§8)

**Decision:** CORE-L3SW and ACC-READSW are connected via an **EtherChannel** (`Po1`, bundling two
physical FastEthernet/GigabitEthernet links) carrying the 802.1Q trunk, instead of a single link.

- If one physical link or port fails, the channel continues forwarding on the remaining link(s);
  no spanning-tree re-convergence delay, no dropped sessions for reading-room users.
- This was chosen over a dual-router/HSRP design because it directly targets the most likely
  failure point (a single access-layer cable/port) at much lower configuration complexity, which
  fits the "Intermediate" difficulty classification of this project.
- **Scope note:** WAN-side redundancy (a second ISP/router) was considered but is out of scope;
  see `01-client-requirements.md` §5.

## 4. Assigned Networking Challenge: HTTP/Web Server (Internal Hosting)

- `WEB-SRV` sits in the dedicated Servers VLAN (40), with a static IP (10.42.40.10).
- It will run the Packet Tracer HTTP service, hosting an internal catalogue/portal page.
- Access is restricted to VLAN 10 (Administration) and VLAN 20 (Archives & Research); the
  Reading Room (VLAN 30) and the WAN are **not** permitted to reach it, since the brief specifies
  *internal* web service hosting, not a public-facing site.
- At Milestone 2 this will be enforced with an extended ACL on CORE-L3SW's Vlan40 SVI, and
  verified with `ping`/browser tests from each VLAN plus a documented negative test (Reading Room
  → denied).

## 5. Client Change Request CR9: Secure Remote Management Access

- The off-site administrator connects over the simulated WAN (Internet cloud) directly to
  RTR-EDGE.
- **SSH (not Telnet)** will be enabled on RTR-EDGE, CORE-L3SW, and ACC-READSW, with `enable secret`
  and a locally-authenticated user for VTY lines.
- An access list on RTR-EDGE's VTY lines/WAN interface will permit management traffic **only**
  from the administrator's known address (192.0.2.2), denying all other inbound WAN traffic.
- This follows the brief's instruction to "follow the specific wording of this project rather than
  introducing additional scope": no site-to-site VPN or additional remote users were added beyond
  the single off-site administrator specified in CR9.

## 6. Addressing Plan Summary

See `03-ip-addressing-plan.md` for the full table. In brief: the assigned `10.42.0.0/16` block is
subnetted into five `/24`s (one per VLAN) plus a `/30` for the WAN link, keeping the scheme simple
to explain on video while leaving room to grow.

## 7. DHCP vs. Static Addressing

| Segment | Method | Reason |
|---|---|---|
| VLAN 10, 20, 30 (end-user PCs) | DHCP (pool per VLAN) | Standard practice for user devices; demonstrates DHCP configuration skill |
| VLAN 40 (WEB-SRV) | Static | Servers must have a predictable address |
| VLAN 99 (switch/router mgmt) | Static | Infrastructure devices are always statically addressed |
| WAN link | Static | Point-to-point link between RTR-EDGE and the simulated ISP/admin |

Reading Room DHCP leases are set shorter (4 hours) than staff leases, reflecting the transient
nature of public patron sessions.
