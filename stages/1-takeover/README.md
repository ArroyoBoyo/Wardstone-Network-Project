# Stage 1: The Takeover

Wardstone Federal Credit Union (WFCU) has 10 staff in a cramped office it's beginning to outgrow, with new hires underway. After an outage nobody could explain, WFCU dropped its old IT provider, who left no documentation, and it has just signed a lease on a new office in Spring Valley.

The job: find out what the old network actually is, build the new office properly, and move everyone over one weekend so the doors open on Monday with everything working.

- How I built the old office in the lab: [discovery/lab-setup.md](discovery/lab-setup.md)
- The new office's addressing plan: [design.md](design.md)

## Phases

| Phase | What I need to do | What I'll deliver |
|---|---|---|
| 1. Discovery | Map the inherited network as if I'd never seen it | Discovery report: inventory and as-is diagram |
| 2. Design and build | Build the new office in GNS3 (2A), then on the real switch (2B) | Address plan, to-be diagram, configs, problem write-ups |
| 3. Cutover | Move every device to the new network in one planned window | Runbook, before-and-after tests, after-action report |

## Problems

| # | Problem | Concepts | Write-up |
|---|---|---|---|
| 1.1 | Nobody knows what's on the network | Discovery: ping sweep, ARP and MAC tables | [ ] |
| 1.2 | The outage nobody could explain | Duplicate IP, ARP, DHCP reservations | [ ] |
| 1.3 | One flat network for everything | VLANs, trunks, inter-VLAN routing | [ ] |
| 1.4 | One public address for the whole office | Default route, NAT, DNS | [ ] |
| 1.5 | An ACL that blocks core banking | ACL order, implicit deny | [ ] |
| 1.6 | Devices nobody can trust | SSH, NTP, logging, hardening | [ ] |
| 1.7 | Moving everyone in one weekend | Cutover planning, rollback | [ ] |

## Verification

Reachability matrix, run before the move (old network) and after (new network).

Old network: every device can reach every other device, including the ATM.

| From \ To | STAFF | CARD | SERVERS | MGMT | corebank 443 | Internet |
|---|---|---|---|---|---|---|
| STAFF | | | | | | |
| CARD | | | | | | |
| SERVERS | | | | | | |

## Progress

- [ ] Old office built in GNS3 and its discovery report written (`discovery/`)
- [ ] Floor plan of the new office, MDF rack setup, and new network design
- [ ] Address and VLAN plan for the new office written before any config
- [ ] Phase 2A built and verified in GNS3
- [ ] Real 3560-CX reset, software updated, baseline config applied
- [ ] Phase 2B cutover, with the same tests passing on the real switch
- [ ] Cutover runbook, the move, and the after-action report (`cutover/`)
- [ ] Problems 1.1 to 1.7 written up; configs, diagrams and the GNS3 project exported
