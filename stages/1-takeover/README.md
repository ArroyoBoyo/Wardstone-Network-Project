# Stage 1: The Takeover

**The business.** Wardstone Federal Credit Union has 12 staff in a cramped office it has outgrown. After an outage nobody could explain, it dropped its old IT provider, who left no documentation, and it has signed a lease on a new office in Spring Valley. The job: find out what the old network actually is, build the new office properly, and move everyone over one weekend so the doors open on Monday with everything working.

**Size:** 12 staff (designed for 15), the old office plus the new one, 1 router and 1 switch per office.

Addressing for this stage is in [docs/standards/addressing.md](../../docs/standards/addressing.md#stage-1-the-takeover).

> Diagrams go here: `diagrams/as-is.svg` (the old office), `diagrams/to-be.svg` (the new office), `diagrams/floor-plan.svg` and `diagrams/mdf-rack.svg`

## Phases

| Phase | What you do | What you hand over |
|---|---|---|
| 1. Discovery | Map the inherited network as if you'd never seen it | Discovery report: inventory, as-is diagram, findings ranked by risk |
| 2. Design and build | Build the new office in GNS3 (2A), then on the real switch (2B) | Address plan, to-be diagram, configs, problem write-ups |
| 3. Cutover | Move every device to the new network in one planned window | Cutover runbook with rollback, before-and-after tests, after-action report |

## Problems

| # | Problem | Concepts | Write-up |
|---|---|---|---|
| 1.1 | Nobody knows what's on the network | Discovery: ping sweep, ARP and MAC tables, LLDP, service scan | [ ] |
| 1.2 | The outage nobody could explain | Duplicate IP, ARP, MAC table, DHCP reservations | [ ] |
| 1.3 | One flat network for everything | VLANs, trunks, inter-VLAN routing | [ ] |
| 1.4 | One public address for the whole office | Default route, PAT, DNS | [ ] |
| 1.5 | An ACL that blocks core banking | ACL order, implicit deny | [ ] |
| 1.6 | Devices nobody can trust | SSH, NTP, logging, hardening | [ ] |
| 1.7 | Moving everyone in one weekend | Cutover planning, rollback, DNS names over IPs | [ ] |

## Build

- [ ] Old office built in GNS3 and its discovery report written (`discovery/`)
- [ ] Floor plan with every drop, MDF rack elevation and bill of materials (`docs/templates/bom.md`)
- [ ] Address and VLAN plan for the new office written before any config
- [ ] Phase 2A built and verified in GNS3
- [ ] 3560-CX reset, software checked, baseline config applied
- [ ] Phase 2B cut over, with the same tests passing on the real switch
- [ ] Cutover runbook written, the move done, after-action report filed (`cutover/`)
- [ ] Problems 1.1 to 1.7 written up; configs, diagrams and the GNS3 project exported

## Verification

Reachability matrix, run before the move (old network) and after (new network):

| From \ To | STAFF | CARD | SERVERS | MGMT | corebank 443 | Internet |
|---|---|---|---|---|---|---|
| STAFF | | | | | | |
| CARD | | | | | | |
| SERVERS | | | | | | |

## What changed since the last stage

First stage: everything is new.
