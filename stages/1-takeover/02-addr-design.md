# Stage 1: Address design

The plan for rebuilding the office in place, following the [addressing standards](../../docs/standards/addressing.md):

- The chain of desk switches goes away. Every desk gets cabled back to a patch panel in the MDF, and one proper switch (lv01-sw-01) replaces MDF-SWITCH-01 and the three desk switches (a star topology).
- The flat 192.168.1.0/24 is split into proper VLANs and subnets, with a gateway for each one (router-on-a-stick). The gateways started on the edge router (WFCU-EDGE-01) and moved to the OPNsense firewall in [Stage 2](../2-locking-it-down/01-firewall-install.md).

## Single points of failure, after the rebuild

The edge router, the firewall and the one MDF switch are still single points of failure. At 12 staff, that's an accepted risk rather than something to buy around. A second of each would double the cost for an office this size, which is usually not ideal. To accommodate this accepted risk, you could prepare the following ahead of time:

- Configs backed up via documentation, so a replacement can be up in under an hour
- A UPS in the MDF in the case of power issues
- A next-day replacement plan with the vendor (or a cold spare)

Real redundancy (two cores, dual uplinks) comes in [Stage 3](../3-campus/), when the business is big enough to justify it.

## Subnets

| VLAN | Subnet | Gateway (on the firewall) | DHCP pool | Statics |
|---|---|---|---|---|
| 10 STAFF | 10.1.10.0/24 | 10.1.10.1 | .100 – .199 | lv01-printer-01 .99 |
| 20 CARD | 10.1.20.0/29 | 10.1.20.1 | none | lv01-atm-01 .2 |
| 50 GUEST | 10.1.50.0/24 | 10.1.50.1 | .10 – .254 | none |
| 60 SERVERS | 10.1.60.0/24 | 10.1.60.1 | none | lv01-fs-01 .2 |
| 99 MGMT | 10.1.99.0/24 | 10.1.99.1 | .100 – .199 | lv01-sw-01 .2, ADMIN-PC .10 |
| Router–firewall link | 10.1.254.0/30 | none | none | WFCU-EDGE-01 .1, OPNsense .2 |

CARD is a /29, since the office will only ever have one ATM. That leaves room for a few more hosts, so IT can still plug in for maintenance or troubleshooting.

The AP gets its address from the MGMT pool. The admin PC is static at .10, so the firewall rules can single it out. The AP carries the staff and guest SSIDs on a trunk, mapped to VLANs 10 and 50.

## Specialty/Network Devices

| Device | Role | Replaces |
|---|---|---|
| lv01-sw-01 | One switch in the MDF, with every desk cabled back to it | MDF-SWITCH-01 and the three desk switches |
| WFCU-EDGE-01 | Edge router: the ISP link and NAT | Stays |
| OPNsense | Firewall: gateway for every VLAN, DHCP, rules between VLANs | new, in Stage 2 |
| ADMIN-PC | Management host on the MGMT VLAN | new |
