# Stage 1: The Takeover

Wardstone Federal Credit Union (WFCU) has 12 staff in its Spring Valley office, with new hires on the way. After an outage nobody could explain, WFCU fired its IT provider, who left no documentation behind.

WFCU needs help right away. The job: find out what's actually on the network, rebuild it into a proper small-office network, and lock it down. All of it happens in the office they're already in, without closing the doors.

(The move to a new building comes in Stage 2.)

- How I built the old office in the lab: [lab-setup.md](lab-setup.md)
- What I found, and what needs fixing: [discover-and-identify.md](discover-and-identify.md)
- The office's new addressing plan: [addr-design.md](addr-design.md)
- How I rebuilt the network: [rebuild.md](rebuild.md)

## What I walked into

- **A linear network full of single points of failure:** three desk switches daisy-chained off the MDF switch, so one dead switch takes down every desk behind it
- **One flat network:** every device, including the ATM and guests on the Wi-Fi, shares one VLAN and one subnet
- **No documentation:** no diagram, no inventory, no labels, and no passwords handed over
- A duplicate IP and devices still on their factory defaults

## Phases

| Phase | What I need to do | Write-up | Status |
|---|---|---|---|
| 1. Discovery | Map the inherited network as if I'd never seen it, and identify what needs fixing | [discover-and-identify.md](discover-and-identify.md) | Done |
| 2. Rebuild | Replace the switch chain with a star topology, split the network into VLANs, and re-address every device | [rebuild.md](rebuild.md) | Done |
| 3. Lock down | Isolate the VLANs from each other, harden the access ports, and set up the real Wi-Fi | | Next |

## Issues and where they're fixed

| Issue | Fix | Status |
|---|---|---|
| A daisy chain of desk switches | Star topology, one 48-port switch in the MDF | Fixed in the rebuild |
| One flat network | VLANs, subnets and router-on-a-stick | VLANs built; isolating them is the lock down |
| A duplicate IP, statics inside the DHCP range | New addressing plan, statics outside the DHCP pool | Fixed in the rebuild |
| No device security | Local logins, SSH only, unused ports shut down, PortFast and BPDU guard | Partly done; the rest is the lock down |
| A consumer router, no firewall | Replace it with a real firewall | Planned as a separate project |
| No documentation | This repo: inventory, diagrams, port map, addressing plan | Fixed |
| Found during the rebuild: core banking unreachable | The NAT rule still matched the old subnet | Fixed in the rebuild |

## Verification

**After the rebuild:** every device can reach every other one, across all VLANs. Routing works, but nothing is isolated yet.

**Target after the lock down** (✓ allowed, ✗ blocked):

| From \ To | STAFF | CARD | GUEST | SERVERS | MGMT | Core banking | Internet |
|---|---|---|---|---|---|---|---|
| STAFF | ✓ | ✗ | ✗ | ✓ | ✗ | ✓ | ✓ |
| CARD | ✗ | ✓ | ✗ | ✗ | ✗ | ✓ | ✗ |
| GUEST | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✓ |
| MGMT | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

## Progress

**Discovery**

- [x] Office "as found" built in GNS3
- [x] Every device inventoried, including the hidden desk switches
- [x] Issues identified for the rebuild

**Rebuild**

- [x] Desk-switch chain replaced with one MDF switch (star topology)
- [x] VLAN and address plan
- [x] Switch baseline: hostname, banner, local login, SSH, management SVI on VLAN 99
- [x] VLANs, trunk and router-on-a-stick built
- [x] Devices re-addressed, with a DHCP scope per VLAN
- [x] NAT fixed for the new subnets
- [ ] Change record for the re-addressing, with a rollback plan

**Lock down**

- [ ] Rules between the VLANs, matching the target matrix above
- [ ] Port security, DHCP snooping, DAI and storm control on the access ports
- [ ] The real 3560-CX takes over its slice of the office, with the same tests passing
- [ ] Real Wi-Fi on the 9130: staff and guest SSIDs

**Wrap-up**

- [ ] Reachability matrix run after the lock down
- [ ] Configs, diagrams and the GNS3 project exported
