# Wardstone Network

Introducing Wardstone Federal Credit Union, a fictional credit union based in Las Vegas, Nevada. This is how I took the business from a single 12-person office to two large offices and three branches, with redundant connectivity across two ISPs.

Every stage starts with a business event. The growth causes real problems, and each problem is reproduced in the lab, fixed with a core networking practice, and proven with before-and-after evidence.

I built this in GNS3 using VyOS, Arista EOS, FRR, and Alpine Linux, alongside real Cisco hardware, including a Catalyst 3560-CX switch and a Catalyst 9130 access point.

# Final architecture diagram will go here #
## Stages

| Stage | Size | What goes wrong | Status |
|---|---|---|---|
| [1 The Takeover](stages/1-takeover/) | 12 staff, moving offices | An undocumented network from previous IT, a duplicate IP behind an outage, a flat network, a weekend move | In progress |
| [2 Locking It Down](stages/2-locking-it-down/) | About 20 staff, Wi-Fi | Rogue DHCP, a desk switch loop, ARP spoofing, a laptop in the ATM port | Planning |
| [3 Headquarters](stages/3-headquarters/) | 60 staff | Single points of failure, a stolen STP root, a shared Wi-Fi password | Planning |
| [4 Multiple Locations](stages/4-multiple-locations/) | HQ + 3 branches | Static routes that don't scale, OSPF stuck over a tunnel, choppy calls | Planning |
| [5 Going External with BGP](stages/5-going-external-bgp/) | 2 offices, 2 ISPs | An ISP outage, a route leak, traffic on the wrong ISP | Planning |

## Best write-ups

This will hold links to the strongest problem write-ups per stage, growing as stages finish.

## How it's documented

- **Problems:** every problem has a write-up with symptom, cause, and a fix with proof (`stages/<n>/problems/`)
- **Postmortems:** the biggest incidents in each stage, how we fixed them (`docs/postmortems/`)
- **Change records:** one per fix, with a rollback plan (`docs/changes/`)
- **Design decisions:** why one option was chosen over another (`docs/decisions/`)
- **Standards:** addressing, naming, physical design, and the baseline config every device gets (this comes later via proper management transition) (`docs/standards/`)
- **Physical design:** floor plans, closet (MDF/IDF) layouts, and (maybe) bills of materials in each stage's `diagrams/` folder

## Rebuilding a stage

Each stage folder will hold an exported GNS3 project (`gns3/`) and the final config of every device (`configs/`). 

This will allow anyone to import the project into GNS3, start the nodes, and paste each config and test the network for themselves

Passwords and keys in exported configs will be replaced with placeholders.
