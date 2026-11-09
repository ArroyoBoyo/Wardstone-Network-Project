# Stage 3: The Campus

Over the next few years, WFCU continued to grow well past what the Spring Valley office could hold. 

Due to scaling and space issues, they acquired a small campus in Summerlin. This includes three empty buildings around one parking lot

Power and general infrastructure are all already in place, but no cabling, no closets, and no network setup. This is where I come in

| Building | Who's there | Staff |
|---|---|---|
| Main | Member services, the executive offices, and the server room (the MDF) | About 50 |
| Operations | Lending and the back office | About 35 |
| Call center | The call center | About 35 |

Of course, in real life, a business doesn’t suddenly add 100+ employees. This stage is meant to simulate how I’d build a more enterprise-level network, compared to the single-office network in the previous stages.

This stage also focuses more on design and high-level infrastructure. Every major architecture decision comes with the options I considered, and why I chose the one I did. Since redundancy is the main goal, you will see decisions made mainly around that idea.

By the end, losing any one core switch, uplink, firewall or fiber run won’t take a building offline, which in an enterprise environment could be VERY costly. Each one is tested by breaking it on purpose, with the failover times recorded.

## Why a two-tier network

A single office switch like Stage 1's stops working at this size, for a few reasons:

| Reason | Stage 1 office | The campus |
|---|---|---|
| **Size** | 16 devices on one switch | 120 staff with foresight for 250, so several hundred ports across three buildings |
| **Reach** | Every desk within 90 m of one closet | Copper only reaches 90 m, so each building needs its own closet (IDF), linked back to the main building by fiber |
| **Scaling** | Adding a device means using another port | Adding a building means adding access switches and two fiber uplinks |
| **Cost of downtime** | 12 staff, so one switch is an accepted risk at that size | 120 staff + millions in AUM, so an hour offline could cost the business more than what a second switch would've cost |

Two-tier splits the network into two layers:

- **Distribution (or "collapsed core"):** Two switches in the main building's MDF. They connect every building, route between the VLANs, and act as each VLAN's gateway.
- **Access:** The switches in each building's closet, where the PCs, printers, and APs plug in (Essentially, any host devices)

**Why not three-tier?** A separate distribution layer only pays off when there are too many buildings for the core to connect directly. At three buildings and only 120 users, it would add switches and cost without solving a real problem.

Although a three-tier would've still worked, thinking from a business perspective, a two-tier makes more sense in this case

## Redundancy: the main focus

No single failure should take a building offline. Every layer gets a backup, and every backup gets tested. 

Here are the 5 main focuses for this section:

| If this fails | This takes over |
|---|---|
| A fiber run between buildings | The second run, through a different conduit |
| One uplink | The other uplink in the LACP bundle |
| A core switch | The other core in the pair |
| The core holding the gateway | The other core takes over the gateway address via VRRP |
| The active firewall | The second firewall, without dropping open connections |

## Requirements

There are endless configurations that could be made in this section for various reasons, but I decided to focus on 3 main points: 

- **Uptime:** No single core, uplink, firewall, or fiber failure takes a building offline. Losing a core or a firewall should only cost seconds to a few minutes of downtime.
- **Segmentation:** Same idea from previous sections regarding department segmentation and cross-communication guardrails 
- **Growth:** Switch ports, PoE, and addresses sized for 120 staff with room to grow
