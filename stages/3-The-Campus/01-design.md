# Stage 3, part 1: Design

[← Stage 3 overview](README.md)

Before we get into building the new architecture, let's lay out the plan for how the network will be structured.

## Addressing

Much like the first phase rebuild, this site will have its own addressing. All IPs will start with 10., following the same structure: the site number (10 in this case), the VLAN, and the final IP.

This setup is a bit different, as there will be a different STAFF VLAN for each building. For example, a staff member in the main building would have an IP like 10.10.10.50, one in the Operations building 10.10.11.50, and one in the Call Center 10.10.12.50.

Below is the structure I will follow for the VLAN portion of our network:

**Main building**

| VLAN | Name | Addressing | Subnet |
|---|---|---|---|
| 10 | STAFF | DHCP, .50 – .254 | /24 |
| 20 | ATMs | Static | /28 |
| 50 | GUEST | DHCP, .100 – .254 | /24 |
| 60 | SERVERS | Static | /24 |
| 99 | MANAGEMENT | DHCP, .100 – .199 | /24 |

**Operations building**

| VLAN | Name | Addressing | Subnet |
|---|---|---|---|
| 11 | STAFF | DHCP, .50 – .254 | /24 |
| 20 | ATMs | Same as main | |
| 50 | GUEST | Same as main | |
| 99 | MANAGEMENT | Same as main | |

**Call Center building**

| VLAN | Name | Addressing | Subnet |
|---|---|---|---|
| 12 | STAFF | DHCP, .50 – .254 | /24 |
| 50 | GUEST | Same as main | |
| 99 | MANAGEMENT | Same as main | |

This addressing structure, specifically STAFF, accounts for employees' computers, printers and scanners, with plenty of room for growth. And of course, the ATMs, SERVERS and other segments all have more than enough addresses for their section.

At this scale, there would likely be a robust security system with dozens of cameras spread throughout each building. I decided not to include that in this portion of the project. However, the process would be the same, as it would receive its own dedicated VLAN, subnet, etc.

## Physical structure

Seeing as there are 3 buildings, the physical infrastructure will be broken up as follows:

**Main: the core MDF room**

- ISP handoff (likely DMARC for fiber connection at a site this size)
- Site edge router + firewall pair
- The site distribution switches
- Its own two 48-port access switches (with space for a third if needed)
- Patch panel labeled for the respective wall drops
- Fiber patch panel for the IDF closets
- Site servers
- PDU
- UPS

**Operations: IDF**

- Two access switches (room for a third if ever needed)
- Patch panel labeled for the respective wall drops
- Fiber patch panel leading to the MDF closet
- PDU
- UPS

**Call Center: IDF**

- Two access switches (room for a third if ever needed)
- Patch panel labeled for the respective wall drops
- Fiber patch panel leading to the MDF closet
- PDU
- UPS

A few things to touch on:

**1. Redundancy.** Since redundancy is a main focus of this section, each access switch has two uplinks, one to each distribution switch, and each IDF connects to the MDF through two fiber runs in separate conduits.

**2. The access switch choice.** Although the patch panels, PDU and UPS are standard, the main focus here is the access switch choice.

For the main building, I decided to go with only two switches for now, since a third switch isn't exactly needed at the moment. Standard practice accounts for at least 20–30% spare space after full setup, to account for future growth.

To meet this goal without spending unnecessary money, every desk gets two drops, but only one is patched into a switch. The second stays unpatched at the panel until it's needed, which keeps the switch count down without re-cabling later. This also keeps wall ports from being open and active for no reason.

Yes, as the company grows, we'd maybe eventually need an extra switch, but in the current state it's not needed and would just collect dust in the MDF closet. The main thing is making sure the patch panel is correctly labeled and associated with the properly labeled port. And if we're cutting it close on ports, then we'd order the third switch.
