# Wardstone Network

Introducing Wardstone Federal Credit Union, a fictional credit union based in Las Vegas, Nevada. This is how I took the business from a single 12-person office to two large offices and three branches, with redundant connectivity across two ISPs.

Every stage starts with a business event. The growth causes real problems, and each problem is reproduced in the lab, fixed with a core networking practice, and proven with before-and-after evidence.

Built in GNS3 (VyOS, Arista EOS, FRR, Alpine Linux) with real Cisco gear: a Catalyst 3560-CX switch and a Catalyst 9130 access point.

* Final architecture diagram *
## Stages

| Stage | Size | What goes wrong | Status |
|---|---|---|---|
| [1 The Takeover](stages/1-takeover/) | 12 staff, moving offices | An undocumented network, a duplicate IP behind an outage, a flat network, a weekend move | In progress |
| [2 Locking It Down](stages/2-locking-it-down/) | About 20 staff, Wi-Fi | Rogue DHCP, a desk switch loop, ARP spoofing, a laptop in the ATM port | Planned |
| [3 Headquarters](stages/3-headquarters/) | 60 staff | Single points of failure, a stolen STP root, a shared Wi-Fi password | Planned |
| [4 Multiple Locations](stages/4-multiple-locations/) | HQ + 3 branches | Static routes that don't scale, OSPF stuck over a tunnel, choppy calls | Planned |
| [5 Going External with BGP](stages/5-going-external-bgp/) | 2 offices, 2 ISPs | An ISP outage, a route leak, traffic on the wrong ISP | Planned |

## Best write-ups

Links to the three strongest problem write-ups go here as stages finish.

## How it's documented

- **Problems:** every problem has a write-up with symptom, cause, fix and proof (`stages/<n>/problems/`)
- **Postmortems:** the biggest incidents in each stage (`docs/postmortems/`)
- **Change records:** one per fix, with a rollback plan (`docs/changes/`)
- **Design decisions:** why one option was chosen over another (`docs/decisions/`)
- **Standards:** addressing, naming, physical design and the baseline config every device gets (`docs/standards/`)
- **Physical design:** floor plans, closet (MDF/IDF) layouts, rack elevations and bills of materials in each stage's `diagrams/` folder

## Rebuilding a stage

Each stage folder holds an exported GNS3 project (`gns3/`) and the final config of every device (`configs/`). Import the project into GNS3 2.2, start the nodes, and paste each config.

Passwords and keys in exported configs are replaced with placeholders.
