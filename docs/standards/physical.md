# Physical design standards

## Closets

- **MDF (main distribution frame):** one per site. ISP demarc, edge router, firewall, core switches, servers, main UPS.
- **IDF (intermediate distribution frame):** added when any desk would be more than 90 m of cable from the MDF, or on another floor. Holds access switches; uplinked to the MDF by fiber (two uplinks from Stage 3, one to each core).
- Closet codes: `MDF`, `IDF2` (floor 2), `IDF2B` (second closet on floor 2).

## Cabling

| Item | Standard |
|---|---|
| Desk cabling | Cat6, horizontal run 90 m or less (100 m including patch cords) |
| Closet to closet | Fiber |
| Drops per workstation | 2 |
| Printers, card terminals, cameras | 1 each |
| APs | 1 ceiling drop each |

## Labels

Every wall jack matches its patch panel port: jack `A-17` is patch panel A, port 17, in that site's MDF. Jacks served by an IDF add the closet: `IDF2-B-05`.

## Capacity

Switch ports and PoE budget are sized with 20–30% spare. Record usage in the stage's capacity table:

| Switch | Ports total | Ports used | Spare % | PoE budget (W) | PoE used (W) | Spare % |
|---|---|---|---|---|---|---|
| lv01-sw-01 | | | | | | |

## Power and environment

- A UPS in every closet. Runtime target: Stage 1 MDF 15 minutes; HQ server room 30 minutes.
- Server rooms: dedicated circuits, cooling, badge-controlled access.

## Physical security

- Closets locked, access limited and logged, a camera on the MDF door from Stage 2.
- Card-terminal cabling never runs through public space (PCI DSS Requirement 9).

## Port map format

| Switch port | Patch panel port | Wall jack | VLAN | Device | Notes |
|---|---|---|---|---|---|
| Gi1/0/2 | A-02 | A-02 | 10 | teller-01 | |

## Lab note

Designs use real-world switch sizes (a 48-port PoE switch in the Stage 1 MDF). The lab implements a slice: the 8-port 3560-CX and virtual switches stand in for the rest. Document both.
