# Lock down, part 1: Installing the firewall

[← Lock down overview](README.md)

```
<img width="1419" height="1085" alt="firewall install" src="https://github.com/user-attachments/assets/481441c2-e341-4418-a06e-d3ed8ff7bed8" />

```

## 1. No firewall

**Issue:** A router was the only thing separating the office from the internet. There was no wall between data coming in from the internet and the local network, or going back out.

**Fix:** I decided to add OPNsense, a dedicated routing and firewall operating system, and placed it behind the edge router so it lives between the router and the main switch. The other options were putting the firewall in front of the router, or replacing the edge router altogether.

**Why:** A firewall behind the router is a widely accepted standard in corporate and office environments. The edge router acts as a filter for network traffic, blocking out noise before it reaches the firewall. That keeps the firewall from being overwhelmed by the noise of the internet, so it can focus on its inbound and outbound rules.

<img width="574" height="471" alt="Screenshot 2026-11-05 222449" src="https://github.com/user-attachments/assets/a921e243-89fa-4f18-8c79-23ecf92e119b" />

## 2. Double NAT

**Issue:** OPNsense is also a router at its core. Double NAT happens when you chain more than one router that performs NAT. It essentially creates a network inside another network, which breaks VPN connections and causes remote access issues. I've personally had to deal with this issue in the field, and it is very, very troublesome.

**Fix:** NAT stays solely on the edge router. On the firewall, I switched **Firewall → NAT → Source NAT** to disabled and applied it.

**Why:** This removes any risk of double NAT, leaving the edge router as the only NAT operator on the network.

## 3. The password wouldn't save

**Issue:** After logging in with the default credentials, I reset the root password as general housekeeping. After a reboot, the password hadn't saved.

**Fix:** I logged in as `installer` and went through the installer setup, then removed the ISO from the node's CD/DVD settings. After the reboot, the new password stuck. I tested with multiple restarts and got the node to boot as expected.

**Why:** Logging in as root on the ISO saves everything to RAM, which gets wiped on any reboot. The ISO also has to be removed, or the node keeps booting from it and erases the previous configs. Very good note to self that has already come in handy.

## 4. LAN and WAN were swapped

**Issue:** In the assign interfaces page, I originally had the LAN and WAN interfaces swapped. This caused connection and recognition issues between the internet and our LAN network. 

**Fix:** I reassigned them, so the WAN port connects to the edge router and the LAN port links straight to the switch.

**Why:** WAN has to face the router and the internet, and LAN has to face the office.

## 5. The gateways lived on the edge router

**Issue:** Since we kept the edge router, its gateway information had to move to the firewall. Especially since the firewall now manages the egress rules of our network. 

**Fix:**

1. I gave the LAN port a placeholder static IP (192.168.1.1), so I could reach OPNsense's web interface through a web terminal in GNS3.

<img width="1452" height="883" alt="Screenshot 2026-11-05 222659" src="https://github.com/user-attachments/assets/afeb2039-602c-4da2-845e-a059f3168e33" />

2. I created the VLANs on the LAN port. VLAN 10 is shown below, and the same config was copied to VLANs 20, 50, 60 and 99.

<img width="1004" height="821" alt="Screenshot 2026-09-22 225829" src="https://github.com/user-attachments/assets/e02f8462-cbec-401b-843a-a481cfa21337" />

3. Under **Interfaces → Assignments**, I created an interface for each VLAN, similar to subinterfaces on Cisco routers.

<img width="1019" height="825" alt="Screenshot 2026-11-01 122136" src="https://github.com/user-attachments/assets/0b03d137-edd8-405c-bd71-7a4a8e99f2d2" />

4. I enabled each one and gave it a static address as the gateway, just as on the edge router. Every VLAN is a /24, except VLAN 20, which is a /29.

| VLAN | Gateway |
|---|---|
| 10 STAFF | 10.1.10.1/24 |
| 20 CARD | 10.1.20.1/29 |
| 50 GUEST | 10.1.50.1/24 |
| 60 SERVERS | 10.1.60.1/24 |
| 99 MGMT | 10.1.99.1/24 |

**Why:** just like any other routing device, the firewall needs a gateway on each VLAN to route the office's traffic.

## 6. DHCP lived on the edge router

**Issue:** Decided that DHCP should move from the edge router to the firewall.

**Fix:** I enabled the DHCP service on the interfaces, then created the DHCP ranges for each VLAN, with their gateways.

**Why:** Seeing gateway addresses, and eventually inbound and outbound rules will be made on the firewall, it's appropriate to move the DHCP services from the edge router to the firewall

This would also make managing the device assignments and policies for each VLAN much more manageable, and allow for the edge router to focus purely on routing incoming traffic from the internet, and protocols such as OSPF, BGP, etc.coming later)

<img width="1017" height="822" alt="Screenshot 2026-11-01 195316" src="https://github.com/user-attachments/assets/fbb2a773-5625-4566-ac42-78f1113d4a98" />

## 7. The firewall and the router needed their own link

**Issue:** The firewall and the edge router needed a dedicated connection, so traffic can route between the two devices and out to the internet.

**Fix:** I created a separate /30 subnet between the two, much like how our edge router is set up with the PE router:

| Device | Interface | Address |
|---|---|---|
| WFCU-EDGE-01 | eth1 (internal-facing) | 10.1.254.1/30 |
| OPNsense | WAN | 10.1.254.2/30 |

<img width="475" height="280" alt="Screenshot 2026-11-05 224829" src="https://github.com/user-attachments/assets/8b317e55-e1df-40e0-92b7-aa2dfbbaba30" />

**Why:** This link serves a single purpose between only two devices. A /30 mitigates the chance of other devices interfering, and it's general best practice for dedicated point-to-point connections.

## 8. The edge router still had the old configs

**Issue:** with the gateways and DHCP on the firewall, the edge router still had both.

<img width="597" height="480" alt="Screenshot 2026-11-01 204709" src="https://github.com/user-attachments/assets/008c208a-3360-45ab-8eac-38151b1ed01f" />

**Fix:**

1. I removed the `vif`s, since they were already configured on the firewall.

<img width="409" height="172" alt="Screenshot 2026-11-01 204847" src="https://github.com/user-attachments/assets/3973440f-99d4-4b57-bd5f-06fee6072e73" />

2. I tried to wipe the DHCP configs, and the commit failed.

<img width="539" height="378" alt="Screenshot 2026-11-01 205536" src="https://github.com/user-attachments/assets/bb7843f3-bfca-464f-84a0-44ed43f90f9c" />

   I learned that the correct way to completely remove DHCP on VyOS is to remove the whole service, instead of cleaning out each config. This is done via:

```
delete service dhcp-server
```

<img width="590" height="201" alt="Screenshot 2026-11-01 205801" src="https://github.com/user-attachments/assets/6a22d690-083d-411d-9a7c-e11e83075e7d" />

3. I set up the router's side of the link, plus a static route to our LAN through the firewall:

```
set interfaces ethernet eth1 address 10.1.254.1/30
set protocols static route 10.1.0.0/16 next-hop 10.1.254.2
```
<img width="494" height="305" alt="Screenshot 2026-11-01 210151" src="https://github.com/user-attachments/assets/5ed33843-38d7-4443-bc30-67c28868a619" />
<img width="562" height="256" alt="Screenshot 2026-11-01 210706" src="https://github.com/user-attachments/assets/44e10fc3-b005-4cbb-a76b-e9f4c4cdc85e" />

**Why:** The router needs to know that the office's networks are reached through the firewall. And since this office's network scheme all falls within the 10.1 range, 10.1.0.0/16 covers all possible routes

This makes routing to any and all VLANs in the network much, much easier and more manageable. 

## Testing and troubleshooting

### DHCP failed

**Issue:** a DHCP request from PC-01 failed.

<img width="510" height="370" alt="Screenshot 2026-11-01 211313" src="https://github.com/user-attachments/assets/444275d0-0581-4c44-8512-fb7a0e69248a" />

**Fix:** I restarted the connection to the switch, and DHCP worked. I tested it across the STAFF, GUEST and MGMT VLANs.

<img width="512" height="434" alt="Screenshot 2026-11-01 212922" src="https://github.com/user-attachments/assets/a4b9cab8-820c-4b8e-939c-4a66168ebca5" />

### Couldn't ping the gateway

**Issue:** PCs were getting DHCP addresses from the firewall, but couldn't ping their gateway.

**Fix:** The firewall logs made it obvious: the rules weren't set up to accept ping traffic. For the sake of testing, I made a blanket allow rule on each VLAN, passing all traffic. The ping test worked on every VLAN.

<img width="721" height="444" alt="Screenshot 2026-11-01 214600" src="https://github.com/user-attachments/assets/12030e38-8d0d-4993-810a-3d2db506232e" />
<img width="892" height="474" alt="Screenshot 2026-11-01 214934" src="https://github.com/user-attachments/assets/5879ee92-4210-4cba-8f16-0905cc979ad2" />
<img width="682" height="285" alt="Screenshot 2026-11-01 215533" src="https://github.com/user-attachments/assets/72a2fdd2-516c-4426-886f-f60d0584d466" />

**Why:** OPNsense runs on implicit deny, so it denies traffic by default unless a rule says otherwise. DHCP is allowed automatically, which is why that part worked. The blanket rule I implemented was only to confirm connectivity before the real rules.

This rule will later be removed, but I kept it for a bit to ensure connectivity while troubleshooting

### Couldn't reach core banking

**Issue:** pings reached the edge router, but not core banking or the ISP's router.

<img width="866" height="555" alt="Screenshot 2026-11-01 221038" src="https://github.com/user-attachments/assets/6d7222b5-543e-4d73-a64a-5373dfc90324" />

**Fix:** This time, the logs showed no blocks at all, so I looked at routing instead of the rules. Under the gateway configuration, there was only an IPv6 gateway on the WAN port. If I was using and configuring IPv6, this would be fine. But as you can see, there is no mention of IPv6 throughout this project

<img width="1018" height="823" alt="Screenshot 2026-11-01 221451" src="https://github.com/user-attachments/assets/b65d8cef-8700-4765-b86a-0ec60e395f50" />

Therefore, I created an IPv4 gateway pointing at the edge router (10.1.254.1), ticked the upstream gateway option, and disabled the old IPv6 gateway so it wouldn't interfere.

<img width="1018" height="823" alt="Screenshot 2026-11-01 223038" src="https://github.com/user-attachments/assets/a090e2c8-ecfe-49da-8f9a-f491fc95d664" />
<img width="1737" height="1160" alt="Screenshot 2026-11-01 223321" src="https://github.com/user-attachments/assets/065de51a-a5a8-4f0a-8c10-b3daca1413f0" />

**Why:** We only use IPv4, and like any router, the firewall needs a default route to reach the internet. which in this case travels upstream to our edge router.

<img width="1018" height="823" alt="Screenshot 2026-11-01 223038" src="https://github.com/user-attachments/assets/74227604-bdf7-4da8-b465-230da99e717a" />

## Where things stand

After all of that, every VLAN can now reach the LAN and WAN devices again. The network is back to how it was before the firewall went in and is ready for the segmentation the client asked for.

<img width="890" height="632" alt="Screenshot 2026-11-03 232221" src="https://github.com/user-attachments/assets/e9eb67c9-4998-4754-a4d4-7dcd545c4798" />
<img width="1737" height="1160" alt="Screenshot 2026-11-01 223321" src="https://github.com/user-attachments/assets/e2cc8ac0-1c83-4349-a5b9-ad33d900a760" />

**Next:** [Part 2: Firewall rules](02-firewall-rules.md)
