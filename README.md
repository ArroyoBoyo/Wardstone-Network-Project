# Wardstone Network

Introducing Wardstone Federal Credit Union, a fictional credit union based in Las Vegas, Nevada. This is how I took the business from a single 12-person office, inherited from an IT provider they had just fired, to a redundant campus with branch offices connected over site-to-site VPNs.

Every stage starts with a business event:

- **Stage 1:** I take over an existing network and rebuild it in place.
- **Stage 2:** I lock it down, implementing various standard security features
- **Stages 3 and 4:** New builds from empty buildings: a two-tier campus built around redundancy, then branch offices connected to it with dynamic routing (OSPF).

I built this in GNS3 using VyOS, OPNsense, Arista EOS, FRR, and Alpine Linux, alongside real Cisco hardware, including a Catalyst 3560-CX switch.

> *Final architecture diagram will go here*

## Stages

| Stage | Size | What happens | Status |
|---|---|---|---|
| [1 The Takeover](stages/1-takeover/) | 12 staff, one office | Take over an undocumented network from the previous IT, then rebuild it in place with a star topology and VLANs | Done |
| [2 The Lock Down](stages/2-locking-it-down/) | The same office | A firewall, rules between the VLANs, management access, and port security on the access layer | Done |
| [3 The Campus](stages/3-campus/) | About 120 staff, three buildings | A two-tier network built from scratch around redundancy using EtherChannel, LACP uplinks, VRRP, and failover testing | Planning |
| [4 Multiple Locations](stages/4-multiple-locations/) | The campus and 3 branches | Branches built from one standard and connected to the campus with site-to-site IPsec VPNs and OSPF | Planning |

## Best write-ups

- [Stage 1: Discovery](stages/1-takeover/03-discover-and-identify.md): what the previous IT left behind, and what needed fixing
- [Stage 1: The rebuild](stages/1-takeover/04-rebuild.md): from a daisy chain to a star topology with VLANs, including the NAT rule that broke core banking
- [Stage 2: Installing the firewall](stages/2-locking-it-down/01-firewall-install.md): adding OPNsense behind the edge router without double NAT
- [Stage 2: Port security](stages/2-locking-it-down/04-port-management.md): DHCP snooping, DAI and sticky MAC on the real Catalyst 3560-CX

## How it's documented

- **Builds:** every build step has its configs and a test that proves it works 
- **Problems:** every problem has a write-up with the issue, the fix, and why
- **Design decisions:** why one option was chosen over another
- **Standards:** addressing, naming, design, and the baseline config every device gets

## Rebuilding a stage in GNS3

Each stage folder will hold an exported GNS3 project and the final config of every device.

This will allow anyone to import the project into GNS3, start the nodes, paste each config and test the network for themselves.
