# Wardstone Network

Introducing Wardstone Federal Credit Union, a fictional credit union based in Las Vegas, Nevada. This is how I took the business from a single 12-person office to two large offices and three branches, with redundant connectivity across two ISPs.

Every stage starts with a business event. In Stage 1, I take over an existing network and rebuild it. Stages 2 and 3 are new builds from empty buildings: a two-tier campus, then a three-tier headquarters, each designed from the requirements up and built step by step, with the configs and a test for every step. Stage 4 takes the network out to the internet with BGP.

I built this in GNS3 using VyOS, Arista EOS, FRR, and Alpine Linux, alongside real Cisco hardware, including a Catalyst 3560-CX switch and a Catalyst 9130 access point.

> *Final architecture diagram will go here*

## Stages

| Stage | Size | What happens | Status |
|---|---|---|---|
| [1 The Takeover](stages/1-takeover/) | 12 staff, one office | Take over an undocumented network from the previous IT, rebuild it in place with a star topology and VLANs, then lock it down | Discovery and rebuild done, lock down next |
| [2 The Campus](stages/2-campus/) | About 120 staff, three buildings | A two-tier network built from scratch: MLAG core, LACP uplinks, 802.1X, then a one-weekend move | Planning |
| [3 The Enterprise](stages/3-enterprise/) | About 380 staff: a five-building headquarters, the campus and 3 branches | A three-tier network built from scratch, with OSPF and IPsec tunnels to every site | Planning |
| [4 Going External with BGP](stages/4-bgp/) | Every site, a regional office, 2 ISPs | An ISP outage, a route leak, traffic on the wrong ISP | Planning |

## Best write-ups

- [Stage 1: Discovery](stages/1-takeover/discover-and-identify.md): what the previous IT left behind, and what needed fixing
- [Stage 1: The rebuild](stages/1-takeover/rebuild.md): from a daisy chain to a star topology with VLANs, including the NAT rule that broke core banking

More get added as stages finish.

## How it's documented

- **Builds:** every build step has its configs and a test that proves it works (in each stage's folder)
- **Problems:** every problem has a write-up with symptom, cause, and a fix with proof (`stages/<n>/problems/`)
- **Change records:** one per major change, with a rollback plan (`docs/changes/`)
- **Design decisions:** why one option was chosen over another (`docs/decisions/`)
- **Standards:** addressing, naming, physical design, and the baseline config every device gets (`docs/standards/`)
- **Physical design:** floor plans and closet (MDF/IDF) layouts in each stage's `diagrams/` folder

## Rebuilding a stage in GNS3

Each stage folder will hold an exported GNS3 project and the final config of every device.

This will allow anyone to import the project into GNS3, start the nodes, paste each config and test the network for themselves.

Passwords and keys in exported configs will be replaced with placeholders.
