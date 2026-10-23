# Stage 1: The Takeover

Wardstone Federal Credit Union (WFCU) has 10 staff in a cramped office it is beginning to outgrow as new hires are underway. After an outage nobody could explain, WFCU dropped its old IT provider, who left no documentation, and it has just signed a lease on a new office in Spring Valley 

The job: find out what the old network actually is, build the new office properly, and move everyone over one weekend so the doors open on Monday with everything working.

Addressing for this stage is in [docs/standards/addressing.md](../../docs/standards/addressing.md#stage-1-the-takeover).

> Diagrams will go here: (the old office), (the new office), (floor plan (maybe)) and (MDF rack (maybe))

## Phases

| Phase | What I needed to do | What I’ll deliver |
|---|---|---|
| 1. Discovery | Map the inherited network as if I’d never seen it | Created discovery report (inventory and as-is diagram) |
| 2. Design and build | Build the new office in GNS3 (2A), then on the real switch (2B) | Created an address plan, to-be diagram, configs, problem write-ups |
| 3. Cutover | Move every device to the new network in one planned window | Set up a simple runbook, before-and-after tests, after-action report |

## Problems

| # | Problem | Concepts | Write-up |
|---|---|---|---|
| 1.1 | Nobody knows what's on the network | Discovery: ping sweep, ARP and MAC tables, service scan | [ ] |
| 1.2 | The outage nobody could explain | Duplicate IP, ARP, DHCP reservations | [ ] |
| 1.3 | One flat network for everything | VLANs, trunks, inter-VLAN routing | [ ] |
| 1.4 | One public address for the whole office | Default route, NAT, DNS | [ ] |
| 1.5 | An ACL that blocks core banking | ACL order, implicit deny | [ ] |
| 1.6 | Devices nobody can trust | SSH, NTP, logging, hardening | [ ] |
| 1.7 | Moving everyone in one weekend | Cutover planning, rollback | [ ] |

## Verification

Reachability matrix, run before the move (old network) and after (new network):

| From \ To | STAFF | CARD | SERVERS | MGMT | corebank 443 | Internet |
|---|---|---|---|---|---|---|
| STAFF | | | | | | |
| CARD | | | | | | |
| SERVERS | | | | | | |
