# Addressing and naming standards

Set in Stage 1 and kept for every later stage, so adding a site never means renumbering.

## IPv4 scheme

`10.SS.VV.0/24`: SS is the site number and VV the VLAN number, written without leading zeros (Branch 1 staff is `10.1.10.0/24`). The gateway is always `.1`. From Stage 3 that's the VRRP address, with the two cores at `.2` and `.3`.

## Sites

| Site | Code | SS | Joins |
|---|---|---|---|
| New office, Spring Valley | lv01 | 1 | Stage 1; becomes Branch 1 in Stage 4 |
| Headquarters, Summerlin | lvhq | 10 | Stage 3 |
| Branch, Henderson | hnd01 | 2 | Stage 4 |
| Branch, North Las Vegas | nlv01 | 3 | Stage 4 |
| Regional office, Reno | rno01 | 20 | Stage 5 |

## VLANs

| VLAN | Name | Subnet | What's on it | Joins |
|---|---|---|---|---|
| 10 | STAFF | 10.SS.10.0/24 | Staff PCs and laptops | 1 |
| 20 | CARD | 10.SS.20.0/24 | Teller card terminals and the ATM; wired only, never on Wi-Fi | 1 |
| 30 | VOICE | 10.SS.30.0/24 | IP phones | 3 |
| 40 | SECURITY | 10.SS.40.0/24 | Cameras and door controller | 2 |
| 50 | GUEST | 172.16.SS.0/24 | Member Wi-Fi; internet only | 2 |
| 60 | SERVERS | 10.SS.60.0/24 | File server, services VM | 1 |
| 99 | MGMT | 10.SS.99.0/24 | Switch, router and AP management | 1 |
| 998 | NATIVE | none | Native VLAN on trunks; carries nothing | 1 |
| 999 | PARKING | none | Unused ports, shut down | 1 |

## Inside each /24

| Range | Use |
|---|---|
| .1 | Gateway |
| .2 – .9 | Network devices (cores, switches, firewall interfaces) |
| .10 – .49 | Static addresses and DHCP reservations (servers, printers, card terminals) |
| .100 – .199 | DHCP pool |
| everything else | Reserved |

## Other ranges

| Use | Range |
|---|---|
| Loopbacks (router IDs) | 10.255.SS.N/32 |
| Point-to-point links | 10.254.0.0/16, carved into /31s |
| VPN tunnels | 10.253.0.0/16, a /30 per tunnel |
| Internet and ISP links | 198.51.100.0/24, carved into /30s |
| Internet test hosts (core banking, a web server) | 192.0.2.0/24 |
| Wardstone's own public block (Stage 5) | 203.0.113.0/24 |
| IPv6 (Stage 4) | 2001:db8:SS::/48 per site, 2001:db8:SS:VV::/64 per VLAN |
| BGP AS numbers (Stage 5) | Wardstone 65010, ISP A 64496, ISP B 64497, provider WAN 64500 |
| Internal DNS | `<site>.wardstone.internal`, such as `files.lv01.wardstone.internal` |

## Names

- **Devices:** `<site>-<role>-<nn>`, with roles `edge`, `fw`, `core`, `acc`, `sw`, `ap`, `srv` (`lv01-edge-01`, `lvhq-core-02`)
- **Interface descriptions:** `TO <device> <port> | <purpose>`, such as `TO lv01-edge-01 eth1 | uplink`

---

# Stage 1: the takeover

## The old office (inherited, built badly on purpose)

One flat network, the way the previous IT provider left it. Build it, then document it in the discovery report as if you'd never seen it.

| Device | Address | Problems to "discover" |
|---|---|---|
| old-router (VyOS, playing a consumer router) | 192.168.1.1/24 | Default login, telnet on, no logging |
| old-switch (Arista, factory defaults) | no management IP | Everything in VLAN 1 |
| printer | 192.168.1.50 (static) | Duplicate address with a staff laptop |
| staff laptop | 192.168.1.50 (static, hand-set) | The cause of the outage |
| file server | 192.168.1.10 (static) | Shared by IP, not by name |
| card terminal | 192.168.1.60 (static) | On the same network as everything else |
| mystery device | 192.168.1.77 | Telnet open, nobody knows what it is |

## The new office (lv01)

### Subnets

| VLAN | Subnet | Gateway | DHCP pool | Statics and reservations |
|---|---|---|---|---|
| 10 STAFF | 10.1.10.0/24 | 10.1.10.1 | .100 – .199 | printer 10.1.10.10 (reservation) |
| 20 CARD | 10.1.20.0/24 | 10.1.20.1 | .100 – .149 | card-01 10.1.20.10 (reservation) |
| 60 SERVERS | 10.1.60.0/24 | 10.1.60.1 | none | files-01 10.1.60.10 (static) |
| 99 MGMT | 10.1.99.0/24 | 10.1.99.1 | none | lv01-sw-01 10.1.99.2 |

### Links

| Link | Subnet | Side A | Side B |
|---|---|---|---|
| inet-01 to lv01-edge-01 | 198.51.100.0/30 | inet-01 eth1 .1 | lv01-edge-01 eth0 .2 |
| inet-01 to corebank | 192.0.2.0/24 | inet-01 eth2 .1 | corebank .10 |
| inet-01 to GNS3 NAT node | DHCP from the NAT node | inet-01 eth0 | NAT |
| lv01-edge-01 to lv01-sw-01, **Phase 2A** | 802.1Q trunk, native VLAN 998 | lv01-edge-01 eth1 (vif 10, 20, 60, 99) | lv01-sw-01 Et1 |
| lv01-edge-01 to lv01-sw-01, **Phase 2B** | 10.254.1.0/31, routed | lv01-edge-01 eth1 .0 | 3560-CX uplink .1 |

In Phase 2A, the gateways (`.1`) live on the VyOS subinterfaces (router-on-a-stick). In Phase 2B they move to SVIs on the 3560-CX, and the edge reaches `10.1.0.0/16` through a static route to `10.254.1.1`.

### Loopbacks

| Device | Loopback |
|---|---|
| lv01-edge-01 | 10.255.1.1/32 |
| lv01-sw-01 | 10.255.1.2/32 |

### Switch port map: lv01-sw-01

| Port | Mode | VLAN | Connected to |
|---|---|---|---|
| 1 | Trunk (Phase 2A) / routed (Phase 2B) | 10, 20, 60, 99; native 998 | lv01-edge-01 eth1 |
| 2 | Access | 10 | teller-01 |
| 3 | Access | 10 | staff-laptop-01 |
| 4 | Access | 20 | card-01 |
| 5 | Access | 60 | files-01 |
| 6 – 8 | Access, shut down | 999 | unused |

On the real 3560-CX the port names differ (`Gi1/0/1` and so on); keep the same numbering.

### Hosts

| Host | Image | VLAN | Address |
|---|---|---|---|
| teller-01 | Alpine | 10 | DHCP |
| staff-laptop-01 | Alpine | 10 | DHCP |
| card-01 | Alpine | 20 | 10.1.20.10 (reservation) |
| files-01 | Alpine | 60 | 10.1.60.10 (static) |
| corebank | Alpine | outside | 192.0.2.10, listening on TCP 443 |

### Services

| Service | Provided by |
|---|---|
| DHCP | lv01-edge-01 (Phase 2A), the 3560-CX (Phase 2B) |
| DNS | lv01-edge-01 as forwarder; `files.lv01.wardstone.internal` = 10.1.60.10 |
| NTP | lv01-edge-01 syncs to the internet; every other device syncs to it |
| NAT | PAT on lv01-edge-01 eth0 for 10.1.0.0/16 |
