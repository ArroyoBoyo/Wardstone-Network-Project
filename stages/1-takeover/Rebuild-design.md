# Stage 1: Rebuild design

The plan for rebuilding the office in place, following the [addressing standards](../../docs/standards/addressing.md):

- The chain of desk switches goes away. Every desk gets cabled back to a patch panel in the MDF, and one proper switch replaces MDF-SWITCH-01 and the three desk switches (a star topology).
- The consumer router (WFCU-EDGE-01) is replaced by a firewall.
- The flat 192.168.1.0/24 is split into proper VLANs and subnets.

## Single points of failure, after the rebuild

The firewall and the one MDF switch are still single points of failure. At 12 staff, that's an accepted risk rather than something to buy around: a second switch and firewall would double the cost for an office this size. The risk is covered by:

Real redundancy (two cores, dual uplinks) comes in Stage 3, when the business is big enough to justify it.

## Subnets

| VLAN | Subnet | Gateway (on the firewall) | DHCP pool | Statics and reservations |
|---|---|---|---|---|
| 10 STAFF | 10.1.10.0/24 | 10.1.10.1 | .100 – .199 | printer1 .10, AP-01 .11 (reservations) |
| 20 CARD | 10.1.20.0/24 | 10.1.20.1 | none | ATM .10 (reservation) |
| 60 SERVERS | 10.1.60.0/24 | 10.1.60.1 | none | file-server .10 (static) |
| 99 MGMT | 10.1.99.0/24 | 10.1.99.1 | none | lv01-sw-01 .2, admin PC .10 |

Until the real Wi-Fi arrives in Stage 2, AP-01 stays on the STAFF VLAN with guest access turned off, so guests are no longer on the same network as the ATM.

## Devices

| Device | Role | Replaces |
|---|---|---|
| lv01-fw-01 (OPNsense) | Firewall and gateway: NAT, DHCP, DNS, rules between VLANs | WFCU-EDGE-01 (consumer router) |
| lv01-sw-01 | One switch in the MDF, with every desk cabled back to it | MDF-SWITCH-01 and the three desk switches |
| admin PC (Webterm) | A browser on the MGMT VLAN, for the firewall's web interface | new |

## Devices being re-addressed

| Host | Old address | New VLAN | New address |
|---|---|---|---|
| PC-01 | 192.168.1.50 (static) | 10 STAFF | DHCP |
| PC-02 to PC-12 | DHCP | 10 STAFF | DHCP |
| printer1 | 192.168.1.50 | 10 STAFF | 10.1.10.10 |
| AP-01 | DHCP | 10 STAFF | 10.1.10.11 |
| ATM | 192.168.1.60 | 20 CARD | 10.1.20.10 |
| file-server | 192.168.1.10 | 60 SERVERS | 10.1.60.10 |

## Lab note: the real switch

The design calls for one 24-port PoE switch in the MDF. In GNS3 (Phase A), a single Arista switch plays that role with every device on it.

The real 3560-CX only has 8 access ports, and the virtual PCs can't plug into it anyway, since they live inside GNS3. So in Phase B, the 3560-CX takes a physical slice of the office via the real devices on my desk (a laptop as a test PC, and the 9130 AP).
