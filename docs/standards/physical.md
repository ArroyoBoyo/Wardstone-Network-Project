# Physical design standards

## Closets

- **MDF:** one per site. ISP equipment, edge router, firewall, distro switches, servers, main UPS, etc.
- **IDF:** added when any desk would be more than 90 m of cable from the MDF, or on another floor (for later project stages). Holds access switches, uplinked to the MDF by fiber (two uplinks from Stage 3, one to each MDF switch).

## Cabling

| Item | Standard |
|---|---|
| Desk cabling | Cat6, horizontal run 90 m or less (100 m including patch cords) |
| Closet to closet | Fiber |
| Drops per workstation | 2 per desk, 4 per teller station (PC, receipt printer, card terminal and spare) |
| Printers, card terminals, cameras | 1 each |
| APs | 1 ceiling drop each |
| Switchport budget | Sized with 20–30% spare for future growth |

## Labels

Every wall jack matches its patch panel port

EX: jack `A-10` is patch panel A, port 10, in that site's MDF. Jacks served by an IDF add the closet, like `IDF2-B-05`, etc.

## Physical security

- Closets locked, access limited and logged, a camera on the MDF door from Stage 2.
- Card-terminal cabling never runs through public space (PCI DSS Requirement 9).

## Port map format example

| Switch port | Patch panel port | Wall jack | VLAN | Device | Notes |
|---|---|---|---|---|---|
| Gi1/0/2 | A-02 | A-02 | 10 | teller-01 | |

## Lab note

Designs use real-world sizes, such as a 48-port PoE switch in the Stage 1 MDF. The lab builds a slice of that on the real 8-port 3560-CX, and virtual switches stand in for the rest.
