# Stage 1: New office design (lv01)

The addressing plan for the new Spring Valley office, following the [addressing standards](../../docs/standards/addressing.md).

## Subnets

| VLAN | Subnet | Gateway | DHCP pool | Statics and reservations |
|---|---|---|---|---|
| 10 STAFF | 10.1.10.0/24 | 10.1.10.1 | .100 – .199 | printer1 .10 (reservation) |
| 20 CARD | 10.1.20.0/24 | 10.1.20.1 | none | ATM .10 (reservation) |
| 60 SERVERS | 10.1.60.0/24 | 10.1.60.1 | none | file-server .10 (static) |
| 99 MGMT | 10.1.99.0/24 | 10.1.99.1 | none | lv01-sw-01 .2 |

## Links

| Link | Subnet | Addresses |
|---|---|---|
| PE-01 to lv01-edge-01 | 198.51.100.4/30 | PE-01 .5, lv01-edge-01 .6 |
| lv01-edge-01 to lv01-sw-01 (Phase 2A) | Trunk: VLANs 10, 20, 60, 99; native 998 | Gateways on the edge router |
| lv01-edge-01 to lv01-sw-01 (Phase 2B) | 10.254.1.0/31, routed | edge .0, switch .1 |

## Devices moving over

| Host | Old address | New VLAN | New address |
|---|---|---|---|
| staff-pc-01 | 192.168.1.50 | 10 STAFF | DHCP |
| printer1 | 192.168.1.50 | 10 STAFF | 10.1.10.10 |
| ATM | 192.168.1.60 | 20 CARD | 10.1.20.10 |
| file-server | 192.168.1.10 | 60 SERVERS | 10.1.60.10 |
