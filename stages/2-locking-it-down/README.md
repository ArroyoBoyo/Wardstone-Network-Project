# Stage 2: The Lock Down

[← Stage 1 overview](../README.md)

After the [rebuild](../rebuild.md), the office had a proper star topology and its own VLAN for each kind of device. But it wasn't secure yet. Every VLAN could still reach every other one, there was no firewall between the office and the internet, and any device could plug into any port or reach the network devices' logins.

The lock down puts standard security practices in place. It turned out to be a big job, so each part has its own page.

## The parts

| Part | What it covers | Status |
|---|---|---|
| [1. Installing the firewall](01-firewall-install.md) | Adding OPNsense behind the edge router, moving the gateways and DHCP onto it, and avoiding double NAT | Done |
| [2. Firewall rules](02-firewall-rules.md) | Rules for each VLAN, following the client's reachability matrix | Done |
| [3. Management access](03-management-access.md) | Only the MGMT VLAN can reach the router, firewall and switch logins | In progress |
| [4. Port security](04-port-security.md) | Sticky MAC, DHCP snooping, Dynamic ARP Inspection and storm control on the real 3560-CX | Planned |

## Summary

**1. Installing the firewall**

| Issue | Fix |
|---|---|
| No firewall between the internet and the office | An OPNsense firewall placed behind the edge router |
| OPNsense does NAT by default, so chaining it behind the edge router would create double NAT | Source NAT disabled on the firewall, leaving the edge router as the only NAT |
| The VLAN gateways and DHCP lived on the edge router | Moved to the firewall, then removed from the router. The two connect over their own /30 link (10.1.254.0/30), with a static route on the router back to the office |
| Firewall rules deny everything by default (only DHCP was allowed), so the VLANs couldn't even ping their gateways | A temporary allow-all rule on each VLAN to confirm connectivity, replaced with real rules in part 2 |
| The firewall couldn't reach core banking or the internet: it had no IPv4 default route | An IPv4 upstream gateway pointing at the edge router (10.1.254.1) |

**2. Firewall rules.** Each VLAN got its own set of rules, matching the client's access matrix. An `INTERNAL` alias (10.1.0.0/16), used with an inverted destination, means "the internet only", which keeps each VLAN out of the rest of the office. Within MGMT, only the admin PC, now static at 10.1.99.10, gets full access.

**3. Management access.** Locking down who can reach the network devices' logins, so only the MGMT VLAN gets to the router, the firewall and the switch.

**4. Port security.** Protecting the access ports on the real Catalyst 3560-CX:
- ports meant for one device only accept that device
- the switch stops rogue DHCP servers, DHCP exhaustion, ARP spoofing and broadcast storms
