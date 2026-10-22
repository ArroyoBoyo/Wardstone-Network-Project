# Physical design standards

## Closets

- **MDF:** one per site. ISP equipment, edge router, firewall, distro switches, servers, main UPS, etc.
- **IDF:** added when any desk would be more than 100 m of cable from the MDF, or on another floor (for later project stages). Holds access switches, uplinked to the MDF by fiber (two uplinks from Stage 3, one to each MDF switch).

## Cabling

| Item | Standard |
|---|---|
| Desk cabling | Cat6, horizontal run 90 m or less (100 m including patch cords) |
| Closet to closet | Fiber (if longer than 100 m) |
| Drops per workstation | 2-4 |
| Printers, card terminals, cameras | 1 each |
| APs | 1 ceiling drop each |

## Labels

Every wall jack matches its patch panel port

EX: jack `A-10` is patch panel A, port 10, in that site's MDF. Jacks served by an IDF add the closet, like `IDF2-B-05', etc.

## Physical security

- Closets locked, access limited and logged, a camera on the MDF door from Stage 2.
- Card-terminal cabling never runs through public space (PCI DSS Requirement 9).

## Port map format Example

| Switch port | Patch panel port | Wall jack | VLAN | Device | Notes |
|---|---|---|---|---|---|
| Gi1/0/2 | A-02 | A-02 | 10 | teller-01 | |

## Lab note

Designs use real-world switch sizes (a 48-port PoE switch in the Stage 1 MDF). However, the lab also implements a slice of physical hardware configuration via the 8-port 3560-CX. Virtual switches stand in for the rest.
