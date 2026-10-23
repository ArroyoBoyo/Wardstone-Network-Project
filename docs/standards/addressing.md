# Addressing and naming standards

How Wardstone numbers and names everything. Each stage's actual addresses live in that stage's folder.

## Address scheme

`10.SS.VV.0/24`: SS is the site number, and VV is the VLAN number. The gateway is always .1.

| Site | Code | SS |
|---|---|---|
| Spring Valley | lv01 | 1 |

More sites get added here as they open.

## VLANs

| VLAN | Name | What's on it |
|---|---|---|
| 10 | STAFF | Staff PCs, laptops and printers |
| 20 | CARD | Card terminals and the ATM; wired only, never on Wi-Fi |
| 60 | SERVERS | File server and other servers |
| 99 | MGMT | Switch, router and AP management |
| 998 | NATIVE | Native VLAN on trunks; carries nothing |
| 999 | UNUSED | Unused ports, shut down |

More VLANs (voice, cameras, guest Wi-Fi) get added when their stage arrives.

## Inside each /24

| Range | Use |
|---|---|
| .1 | Gateway |
| .2 – .9 | Network devices |
| .10 – .49 | Statics and DHCP reservations |
| .100 – .199 | DHCP pool |

Statics and the DHCP pool never overlap. That overlap is what caused the old office's outage.

## Naming

- **Devices:** `<site>-<role>-<nn>`, such as `lv01-edge-01` or `lv01-sw-01`
