# Addressing and naming standards

## Address scheme

`10.SS.VV.0/24`: SS is the site number, and VV is the VLAN number. The gateway is always .1.

## Sites

| Site | Code | SS | Joins |
|---|---|---|---|
| New office, Spring Valley | lv01 | 1 | Stage 1; becomes Branch 1 in Stage 4 |
| Headquarters, Summerlin | lvhq | 10 | Stage 3 |
| Branch, Henderson | hnd01 | 2 | Stage 4 |
| Branch, North Las Vegas | nlv01 | 3 | Stage 4 |
| Regional office, Reno | rno01 | 20 | Stage 5 |

## VLANs

| VLAN | Name | Subnet | What's on it |
|---|---|---|---|
| 10 | STAFF | 10.SS.10.0/24 | Staff PCs, laptops and printers |
| 20 | CARD | 10.SS.20.0/24 | Teller card terminals and the ATM; wired only, never on Wi-Fi |
| 30 | VOICE | 10.SS.30.0/24 | IP phones |
| 40 | SECURITY | 10.SS.40.0/24 | Cameras and door controller |
| 50 | GUEST | 172.16.SS.0/24 | Member Wi-Fi; internet only |
| 60 | SERVERS | 10.SS.60.0/24 | File server, services VMs, etc |
| 99 | MGMT | 10.SS.99.0/24 | Switch, router and AP management |
| 998 | NATIVE | none | Native VLAN on trunks; carries nothing |
| 999 | UNUSED | none | Unused ports, shut down |

## Inside each /24

| Range | Use |
|---|---|
| .1 | Gateway |
| .2 – .9 | Network devices (cores/distro, switches, firewall interfaces) |
| .10 – .49 | Statics and DHCP reservations (servers, printers, card terminals) |
| .100 – .199 | DHCP pool |
| everything else | Reserved |

Statics and the DHCP pool never overlap. That overlap is exactly what caused the old office's outage.

## Other ranges

| Use | Range |
|---|---|
| ISP handoffs | 198.51.100.0/24, a /30 per site |
| Internet test hosts (corebank) | 192.0.2.0/24 |
| Point-to-point links inside a site | 10.254.SS.0/24, carved into /31s |

More ranges (loopbacks, VPN tunnels, IPv6, BGP) get added here when their stage arrives.

## Naming

- **Devices:** `<site>-<role>-<nn>`, with roles `edge`, `fw`, `core`, `acc`, `sw`, `ap`, `srv` (lv01-edge-01, lvhq-core-02)
  
---

# Stage 1: the takeover

## The old office (as inherited)

One flat network the way the previous IT provider left it. I rebuilt it in the lab, then documented it in the discovery report as if I'd inherited it. The old office doesn't follow any of the standards above, which is the point.

| Device | Address | What's wrong |
|---|---|---|
| WFCU-EDGE-01 | 192.168.1.1/24 | DHCP pool (.11 – .254) overlaps the static devices, and hands out no DNS server |
| MDF-SWITCH-01 | no management IP | Factory defaults, no admin password, everything in VLAN 1 |
| staff-pc-01 | 192.168.1.50 (static, hand-set) | Duplicate address with the printer: the cause of the outage |
| printer1 | 192.168.1.50 (static) | Duplicate address with the staff PC |
| file-server | 192.168.1.10 (static) | Shared by IP, not by name |
| ATM | 192.168.1.60 (static) | On the same network as everything else |

| Link | Subnet | Addresses |
|---|---|---|
| Internet to PE-01 | 192.168.122.0/24 (GNS3 NAT) | PE-01 .2 |
| PE-01 to WFCU-EDGE-01 | 198.51.100.0/30 | PE-01 .1, WFCU-EDGE-01 .2 |
| PE-01 to corebank | 192.0.2.0/24 | PE-01 .1, corebank .10 |

## The new office (lv01)

### Subnets

| VLAN | Subnet | Gateway | DHCP pool | Statics and reservations |
|---|---|---|---|---|
| 10 STAFF | 10.1.10.0/24 | 10.1.10.1 | .100 – .199 | printer1 10.1.10.10 (reservation) |
| 20 CARD | 10.1.20.0/24 | 10.1.20.1 | none | ATM 10.1.20.10 (reservation) |
| 60 SERVERS | 10.1.60.0/24 | 10.1.60.1 | none | file-server 10.1.60.10 (static) |
| 99 MGMT | 10.1.99.0/24 | 10.1.99.1 | none | lv01-sw-01 10.1.99.2 |

### Links

| Link | Subnet | Addresses |
|---|---|---|
| PE-01 to lv01-edge-01 (the new office's own circuit) | 198.51.100.4/30 | PE-01 .5, lv01-edge-01 .6 |
| lv01-edge-01 to lv01-sw-01, Phase 2A | 802.1Q trunk: VLANs 10, 20, 60, 99; native 998 | Gateways on the edge's subinterfaces |
| lv01-edge-01 to lv01-sw-01, Phase 2B | 10.254.1.0/31, routed | lv01-edge-01 .0, lv01-sw-01 .1 |

### Hosts

The same devices as the old office, moved over during the cutover with new addresses.

| Host | Old address | New VLAN | New address |
|---|---|---|---|
| staff-pc-01 | 192.168.1.50 | 10 STAFF | DHCP |
| printer1 | 192.168.1.50 | 10 STAFF | 10.1.10.10 (reservation) |
| ATM | 192.168.1.60 | 20 CARD | 10.1.20.10 (reservation) |
| file-server | 192.168.1.10 | 60 SERVERS | 10.1.60.10 (static) |
| corebank | 192.0.2.10 | outside | unchanged, listening on TCP 443 |

### Switch port map: lv01-sw-01

Filled in during the Phase 2 design. Laid out for the real 3560-CX-8PC-S: 8 PoE+ access ports and 2 uplinks.

| Port | Mode | VLAN | Connected to |
|---|---|---|---|
| Gi1/0/1 – 1/0/8 | | | |
| Gi1/0/9 – 1/0/10 (uplinks) | | | |

### Services

| Service | Provided by |
|---|---|
| DHCP | lv01-edge-01 (Phase 2A), the 3560-CX (Phase 2B) |
| DNS | lv01-edge-01 as forwarder; `files.lv01.wardstone.internal` = 10.1.60.10 |
| NTP | lv01-edge-01 syncs to the internet; every other device syncs to it |
| NAT | PAT on lv01-edge-01 eth0 for 10.1.0.0/16 |
