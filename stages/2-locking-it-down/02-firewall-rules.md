# Lock down, part 2: Firewall rules

[← Lock down overview](README.md)

<img width="1005" height="682" alt="1" src="https://github.com/user-attachments/assets/6bf5eae8-7f20-4189-9e04-d40f1e9faf16" />
<img width="1005" height="685" alt="2" src="https://github.com/user-attachments/assets/11d3764c-9364-46a1-baa9-0ea27b226ebf" />
<img width="1006" height="681" alt="3" src="https://github.com/user-attachments/assets/4bd22e6c-ec20-4786-9243-35e71582c048" />

## 1. Every VLAN could still reach every other one

**Issue:** After [installing the firewall](01-firewall-install.md), the network was back to how it was before. Although this is good, it means the real segmentation starts now.

**Fix:** Implemented rules for each VLAN, following the access matrix provided by the client (Wardstone). See the [reachability matrix](../README.md#verification).

## 2. STAFF

**Issue:** Per the matrix, staff should be able to talk to other staff, the servers, core banking and the internet, and nothing else.

**Fix:** The following or a near-exact copy of the simple rules I put in place to accomplish this goal. 

| # | Action | Protocol | Source | Destination |
|---|---|---|---|---|
| 1 | Pass | ICMP | STAFF net | STAFF address |
| 2 | Pass | any | STAFF net | SERVERS net |
| 3 | Pass | any | STAFF net | INTERNAL (inverted) |

**Why:**

- **Rule 1:** When troubleshooting, the computer can ping and communicate with its default gateway. That's how you test if it can reach its "door to the internet", so to speak.
- **No staff-to-staff rule:** You may ask, why I dont have a staff-to-staff rule

  Because all staff computers are on the same VLAN, their traffic never actually reaches the router, as it is traffic through the switch itself

  Therefore, no layer 3 rule is needed, and this concept was shared across all rulesets
  
- **Rule 2:** A simple allow rule, so staff-sourced connections can reach the server network.
- **Rule 3:** Allows connections to every network other than our internal network, so essentially, the internet. More details on this in the following section.

<img width="1014" height="604" alt="Screenshot 2026-11-03 230239" src="https://github.com/user-attachments/assets/9400320f-f4d1-4944-a092-b56e6fa2062c" />

## 3. The internet, without the rest of the office

**Issue:** Obviously, staff needs the internet, but "any" as a destination would also include every other VLAN.

**Fix:** An alias in the firewall settings called INTERNAL, with 10.1.0.0/16 as its "content" or target network in OPNsense.

Combined with the invert option on the rule's destination, it basically says means "allow connections to anything that isn't part of our internal network, AKA the internet."

**Why:** Open internet access comes with its own security risk, which would normally be mitigated with antivirus, IDS/IPS, and EDR. Although those processes are not demonstrated here, they are standard across business architectures, both big and small, and would be implemented.

## 4. The blanket allow rules

**Issue:** The blanket allow rules from testing were still in place.

**Fix:** I removed them, then ran test pings across the subnets, all with a successful output.

## 5. MGMT

**Issue:** Per the matrix, management is supposed to have access to everything. But leaving it completely open-ended isn't the best idea either.

**Fix:** To make it more targeted, only the admin PC gets full access rights. I changed the admin PC from DHCP to a static address, 10.1.99.10, so its rule stays consistent. Then I added the generic ping allow and internet rules for the rest of the management devices.

**Why:** Devices in the management VLAN can still reach the internet for software and firmware updates, but only the admin PC has full admin rights to literally everything. This setup makes it so if a rogue computer somehow made it onto the management VLAN, it wouldn't get admin access.

<img width="1012" height="544" alt="Screenshot 2026-11-04 000947" src="https://github.com/user-attachments/assets/360df2ac-bb71-4ff5-ac53-c09600801500" />

## Result

The access matrix given by the client is now complete!

**Next:** [Part 3: Management access](03-management-access.md)
