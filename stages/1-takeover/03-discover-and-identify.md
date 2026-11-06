# Stage 1.0: Discover and Identify

The first thing to do after taking over is find out what's actually on the network. The old IT left no documentation, so everything here comes from the network itself.

## Discovery

### Inventory

| Device | Count | Address | Where |
|---|---|---|---|
| Workstation PCs | 12 | All DHCP via edge router | Spread across the back office (4), lending offices (4) and front desk (4)|
| Office printer | 1 | 192.168.1.50 (static) | Located in the back off, connected to MDF |
| ATM | 1 | 192.168.1.60 (static) | Located in front commons area, connected to MDF switch |
| File server | 1 | 192.168.1.10 (static) | Located in back office, connected to MDF switch |
| Access point | 1 | DHCP | MDF switch |
| Managed switch | 1 | (IP not set by previous IT) | MDF |
| Unmanaged switches | 3 | N/A | Daisy-chained off the MDF switch |
| Consumer router | 1 | 192.168.1.1 | Connected to MDF, the only device facing the internet |

That's 16 host devices (PC's, servers, ATMs, etc) to move into the new design, plus the network gear being replaced. There are no IP phones or camera currently

## Identify issues to fix in the rebuild

### 1. The daisy chain

The three desk switches are chained one after another off the MDF switch. Every link in the chain is a single point of failure: if the MDF link to the front-desk switch breaks, the whole office goes down. All traffic also shares that one link, so it's vulnerable to bottlenecks in times of high traffic.

Granted, bottlenecks in this size of office are unlikely, but its the principle that we are focusing on, and minimizing the scope of weaknesses in the network.

**Fix:** a star topology

Every desk is cabled back to one managed 48-port PoE switch in the MDF. Although the office has less than half of the devices that the switch has ports, this move is made with expansion in mind. Phones, security cameras, or any other devices the office will add over time

The new switch and firewall are still single points of failure, but at 12 staff that's an accepted risk the office is willing to accept to cut total costs.

### 2. One flat network

Staff PCs, the printer, the ATM, the file server and the Wi-Fi all share one subnet, so any device can reach any other. For a credit union, mixing business and user data is very irresponsible, and must be addressed immediately

**Fix:** split the network into respective VLANs and subnets

| VLAN | Name | What's on it | Subnet prefix | 
|---|---|---|---|
| 10 | STAFF | Workstations and the printer | /24 |
| 20 | CARD | The ATM and card terminals; wired only, reaches core banking and nothing else | /29 |
| 50 | GUEST | Guest Wi-Fi, internet only | /24 |
| 60 | SERVERS | The file server, and any servers added later | /24 |
| 99 | MGMT | Switch, firewall and AP management | /24 |
| 998 | NATIVE | Native VLAN on trunks; carries nothing | N/A |
| 999 | UNUSED | Unused ports, shut down | N/A |

I decided to use a /29 prefix for the ATM machine, as the office will only ever have one in the office.

Excluding the network, broadcast, ATM, and gateway addresses, this allows up to 4 host devices to join the subnet, which allows for IT to join the network for maintenance/troubleshooting if needed.

### 3. Addressing

**Fix:** a new addressing plan with statics and reservations kept outside the DHCP pool. See [addr-design.md](addr-design.md).

This will allow for easy VLAN and location identification, which will be documented and readily accessible to the IT team.

### 4. No device security

Devices still have their default logins, and the switch ports trust anything plugged into them.

**Fix:** local accounts and permitted SSH access only, management on VLAN 99, unused ports shut down, and port security, DHCP snooping, DAI and BPDU guard enabled on the access ports

### 5. No firewall

A consumer router is the only thing between the office and the internet, with no rules for protecting the internal network.

**Fix:** replace it with an Pfsense firewall, which routes between the VLANs and denies everything between them by default unless a rule allows it.

The actual configuration of the firewall will not be prioritized at this part of the project, as I plan to expand on this in another project later on

### 6. No documentation

There are no diagrams, IP lists, or passwords on record. At this size it's recoverable, but it gets worse with every device added and eventually it will become a major headache

**Fix:** this page, plus an IP/VLAN table, network diagrams and port descriptions kept in this repo.
