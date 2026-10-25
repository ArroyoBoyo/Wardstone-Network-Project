# Stage 1: The Takeover

Wardstone Federal Credit Union (WFCU) has 10 staff in its Spring Valley office, with new hires on the way. After an outage nobody could explain, WFCU fired its IT provider, who left no documentation behind.

WFCU needs help right away. The job: find out what's actually on the network, stop the outages, and turn what's there into a proper small-office network, with a real firewall instead of a consumer router. All of it happens in the office they're already in, without closing the doors.

(The move to a new building comes in a later stage)

- How I built the old office in the lab: [lab-setup.md](lab-setup.md)
- The new office's addressing plan: [addr-design.md](addr-design.md)
- The new office's network plan: [rebuild-design.md](rebuild-design.md)


## What I walked into

- **A linear network full of single points of failure:** three desk switches daisy-chained off the MDF switch, so one dead switch takes down every desk behind it
- **One flat network:** every device, including the ATM and guests on the Wi-Fi, shares one VLAN and one subnet
- **No documentation:** no diagram, no inventory, no labels, and no passwords handed over
- A duplicate IP and devices still on their factory defaults

## Phases

| Phase | What I need to do | What I'll deliver |
|---|---|---|
| 1. Discovery | Map the inherited network as if I'd never seen it | Discovery report: inventory and as-is diagram |
| 2. Stabilize | Fix what's breaking things today, without redesigning anything yet | Duplicate IP fixed, default passwords changed, devices hardened, configs backed up |
| 3. Rebuild | Replace the switch chain, split the network into VLANs, document, and re-address every device after hours | Design, change records with rollback plans, configs, before-and-after tests |
## Problems

| # | Problem | Concepts | Write-up |
|---|---|---|---|
| 1.1 | Nobody knows what's on the network | Discovery sweep and proper documention | [ ] |
| 1.2 | The outage nobody could explain | Duplicate IP, and DHCP reservations | [ ] |
| 1.3 | A daisy chain of desk switches | Single points of failure, star topology, structured cabling | [ ] |
| 1.4 | One flat network for everything | VLANs, subnets, trunks, inter-VLAN routing | [ ] |
| 1.5 | A firewall rule that blocks core banking | Rule order, implicit deny | [ ] |
| 1.6 | Re-addressing a live office | Change windows, rollback, DNS names over IPs | [ ] |

## Verification

Reachability matrix, run before the move (old network) and after (new network).

Old network: every device can reach every other device, including the ATM.

| From \ To | STAFF | CARD | SERVERS | MGMT | corebank | Internet |
|---|---|---|---|---|---|---|
| STAFF | | | | | | |
| CARD | | | | | | |
| SERVERS | | | | | | |

## Progress

**Discovery**

- [x] Office "as found" built in GNS3
- [ ] Discovery sweep: every device inventoried, including the hidden desk switches 
- [ ] Old office issues that need to be addressed immediately

**Stabilize**

- [ ] Duplicate IP traced and fixed, DHCP pool moved off the static addresses
- [ ] Default passwords changed, baseline config applied

**Rebuild in place**

- [ ] New network design: star topology, VLAN and address plan
- [ ] Desk-switch chain replaced with one MDF switch
- [ ] Firewall swapped in for the consumer router
- [ ] VLANs, trunk and firewall rules built
- [ ] Devices re-addressed in after-hours change windows
- [ ] Real 3560-CX takes over its slice of the office, with the same tests passing

**Wrap-up**

- [ ] Reachability matrix run before and after
- [ ] Problems written up; configs, diagrams and the GNS3 project exported for project proof
