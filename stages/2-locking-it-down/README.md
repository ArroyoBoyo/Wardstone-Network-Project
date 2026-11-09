# Stage 2: The Lock Down

[← Stage 1 overview](../README.md)

After the [rebuild](../rebuild.md), the office had a proper star topology and its own VLAN for each kind of device. But it wasn't secure yet. Every VLAN could still reach every other one, there was no firewall between the office and the internet, and any device could plug into any port or reach the network devices' logins.

The lock down puts standard security practices in place. It turned out to be a big job, so each part has its own page.

## The parts

| Part | What it covers | Status |
|---|---|---|
| [1. Installing the firewall](01-firewall-install.md) | Adding OPNsense behind the edge router, moving the gateways and DHCP onto it, and avoiding double NAT | Done |
| [2. Firewall rules](02-firewall-rules.md) | Rules for each VLAN, following the client's reachability matrix | Done |
| [3. Management access](03-management-access.md) | Only the MGMT VLAN can reach the router, firewall and switch logins | Done |
| [4. Port security](04-port-security.md) | Sticky MAC, DHCP snooping, Dynamic ARP Inspection and storm control on the real 3560-CX | Done |

## Summary

**1. Installing the firewall**

| Issue | Fix |
|---|---|
| No firewall between the internet and the office | An OPNsense firewall placed behind the edge router |
| OPNsense does NAT by default, so chaining it behind the edge router would create double NAT | Source NAT disabled on the firewall, leaving the edge router as the only NAT |
| The VLAN gateways and DHCP lived on the edge router | Moved to the firewall, then removed from the router. The two connect over their own /30 link (10.1.254.0/30), with a static route on the router back to the office |
| Firewall rules deny everything by default (only DHCP was allowed), so the VLANs couldn't even ping their gateways | A temporary allow-all rule on each VLAN to confirm connectivity, replaced with real rules in part 2 |
| The firewall couldn't reach core banking or the internet: it had no IPv4 default route | An IPv4 upstream gateway pointing at the edge router (10.1.254.1) |

**2. Firewall rules**

| Issue | Fix |
|---|---|
| Every VLAN could still reach every other one | Rules for each VLAN, following the client's access matrix |
| VLANs need the internet, but "any" would include the rest of the office | An `INTERNAL` alias (10.1.0.0/16), with the destination inverted |
| The blanket allow rules from testing were still in place | Removed, then tested with pings across the subnets |
| MGMT is supposed to reach everything, but leaving it open-ended isn't the best idea | Only the admin PC (static at 10.1.99.10) gets full access. Other management devices get ping and the internet |

**3. Management access**

| Issue | Fix |
|---|---|
| Any device that can reach the switch can get to its SSH login | An ACL on the switch's SSH, allowing only the admin PC |
| VyOS listens for SSH on every interface, including the one facing the internet | SSH only listens on the router's internal address (10.1.254.1) |
| SSH is off on the firewall by default | Enabled, listening only on the MGMT interface |

**4. Port security**

| Issue | Fix |
|---|---|
| A rogue DHCP server can pose as the DHCP server, gateway or DNS server | DHCP snooping, with only the upstream port trusted |
| A device flooding DHCP requests can drain every address (DHCP exhaustion) | A DHCP rate limit per port, with errdisable recovery to bring the port back automatically |
| A device can send fake ARP messages claiming to be the gateway | Dynamic ARP Inspection, checking ARP against the DHCP snooping table, plus ARP ACLs for static devices |
| Ports meant for one device accept anything plugged into them | Port security with sticky MAC, so only the original device can use the port |

Already done in the Stage 1 rebuild: unused ports are in VLAN 999 and shut down, and access ports have PortFast and BPDU guard.

