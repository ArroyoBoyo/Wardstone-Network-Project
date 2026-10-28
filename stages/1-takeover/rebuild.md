# Stage 1: The Rebuild

When building a new network, I like to start at Layer 2 and build outwards. The MDF switch is the center of the office's network, so it makes sense to start there.

Here is the final network topology for the redesign at the current office. The following document covers the steps I took and configurations made to make this come to life.

<img width="1420" height="1085" alt="network redesign 1 0" src="https://github.com/user-attachments/assets/272e10b5-c41c-4aac-a555-853b31aa4fbd" />

## 1. Switch housekeeping

Before anything goes into production, every device gets some basic housekeeping. On the new switch:

- Disabled Zero Touch Provisioning (Arista)
- Set the hostname to `lv01-sw-01`, following the naming standard from the [addressing standards](../../docs/standards/addressing.md) (site - role - ID)
- Added a login banner to warn off unauthorized users
- Created a local admin user and set an enable password for privileged exec mode
- Required that local login for both console and remote access

On Cisco, login is set per line (console, vty, and so on). On Arista, one command applies local login to the console and remote connections at once:

```
aaa authentication login default local
```

### Management access (SSH)

On Arista, SSH is on by default and telnet is off by default. However, for SSH to actually work, the switch still needs:

- a local user, with login required (done above)
- an IP address, which I put on an SVI in VLAN 99, the management VLAN
- a default route to the gateway

```
interface Vlan99
   ip address 10.1.99.2/24
   no shutdown
ip route 0.0.0.0/0 10.1.99.1
```

Quick refresh: in 10.1.99.2, the second octet is the site (1) and the third is the VLAN (99). 10.1.99.1 is the gateway for that subnet, as planned in the [addressing plan](addr-design.md).

<img width="401" height="52" alt="Screenshot 2026-10-26 000545" src="https://github.com/user-attachments/assets/2421d74a-0fa6-4711-b9d8-1cfb190a18e9" />

For comparison, here's how you would configure this on a Cisco switch. Same concepts, just different syntax:

```
hostname lv01-sw-01
ip domain-name wfcu.local
crypto key generate rsa modulus 2048
ip ssh version 2
username admin privilege 15 secret <password>
enable secret <password>
line vty 0 15
 login local
 transport input ssh
interface vlan 99
 ip address 10.1.99.2 255.255.255.0
 no shutdown
ip default-gateway 10.1.99.1
```

And here is proof that SSH is enabled and configured properly 

<img width="656" height="178" alt="ssh" src="https://github.com/user-attachments/assets/057d2840-7ec7-4510-a7b5-abcb9ed7814a" />
<img width="569" height="107" alt="image" src="https://github.com/user-attachments/assets/87bb8f41-8d0c-4087-8b3b-fbc23ef8380e" />

### VLANs

Next, I created the VLANs from the planning phase:

| VLAN | Name | Use |
|---|---|---|
| 10 | STAFF | Workstations and the printer |
| 20 | CARD | The ATM (a /29, since only a few card devices will ever be on it) |
| 50 | GUEST | Guest Wi-Fi |
| 60 | SERVERS | The file server |
| 99 | MGMT | The switch, AP and admin PC |
| 998 | NATIVE | Native VLAN on trunks; carries nothing |
| 999 | UNUSED | Every port not in use |

Every port starts in VLAN 999 and is shut down, and only moves out when a device is plugged in. That way, someone plugging a random device into the switch gets nothing.

<img width="568" height="321" alt="Screenshot 2026-10-25 233837" src="https://github.com/user-attachments/assets/3b7c0060-0f99-4440-8a26-e6f706f6edc0" />

I also turned on logging, so there's a record to go back to when troubleshooting, and saved the config with `write memory` after every block of changes.

## 2. Connecting the devices

With the housekeeping done, it was time to plug in devices from around the office. The following is the current port map and its VLAN/trunk configurations

| Port | Device | VLAN |
|---|---|---|
| Et1 | Trunk to WFCU-EDGE-01 | 10, 20, 50, 60, 99 (native 998) |
| Et2 – Et13 | PC-01 to PC-12 | 10 STAFF |
| Et14 | lv01-printer-01 | 10 STAFF |
| Et15 | lv01-atm-01 | 20 CARD |
| Et16 | lv01-fs-01 | 60 SERVERS |
| Et17 | lv01-ap-01 (trunk) | 10, 50, 99 (native 99) |
| Et18 | ADMIN-PC | 99 MGMT |
| Et19 – Et48 | Unused | 999, shut down |

Every port with an end device on it got a description, PortFast and BPDU guard:

- **PortFast** skips STP's listening and learning wait, so the port starts forwarding as soon as a device plugs in.
- **BPDU guard** shuts the port down if it ever receives a BPDU, so a switch plugged into a desk port can't cause a loop.

<img width="919" height="254" alt="portfast + bpdugaurd" src="https://github.com/user-attachments/assets/932f0144-efef-4cee-93e8-a9244474b2aa" />

## 3. Routing between VLANs (router-on-a-stick method)

Now, it was time to configure the layer 3 portion of the network via our edge router

The switch is ready, but the router still has no interfaces for the new VLANs yet, so nothing on the internal network can reach a gateway. What I did to fix this:

**Switch side:** I made Et1, the port to the router, a trunk allowing VLANs 10, 20, 50, 60 and 99, with native VLAN 998. Arista only supports 802.1Q, so there's no encapsulation command like on older Cisco switches.

**Router side:** VyOS doesn't need a native VLAN set the way a Cisco router does (`encapsulation dot1Q 998 native`). Tagged traffic goes to the matching subinterface (`vif`), and untagged traffic lands on the physical interface itself. So I removed the old flat network's address (192.168.1.1) from eth1. Now untagged traffic on native VLAN 998 lands on an interface with no address and goes nowhere, and the old network is gone for good.

Then I created a `vif` for each VLAN, with its gateway at .1. The concept is the same as subinterfaces on Cisco:

```
set interfaces ethernet eth1 vif 10 address '10.1.10.1/24'
set interfaces ethernet eth1 vif 20 address '10.1.20.1/29'
set interfaces ethernet eth1 vif 50 address '10.1.50.1/24'
set interfaces ethernet eth1 vif 60 address '10.1.60.1/24'
set interfaces ethernet eth1 vif 99 address '10.1.99.1/24'
```

<img width="535" height="361" alt="vyos subinterface creation 10" src="https://github.com/user-attachments/assets/b71b958f-501d-482a-8c29-ff5edcf77651" />

Now, each device has a respective default gateway for its VLAN, which enables inter-VLAN routing within the network. 

### DHCP

Each VLAN that uses DHCP has its own scope, with the default gateway, DNS server, and range of IP addresss:

| VLAN | Range | Gateway | DNS |
|---|---|---|---|
| 10 STAFF | .100 – .199 | 10.1.10.1 | 1.1.1.1 |
| 50 GUEST | .10 – .254 (nothing on the guest network is static) | 10.1.50.1 | 1.1.1.1 |
| 99 MGMT | .100 – .199 | 10.1.99.1 | 1.1.1.1 |

VyOS picks the scope based on the interface the request comes in on. A request from VLAN 10 arrives on `eth1.10`, whose address sits in 10.1.10.0/24, so it gets a STAFF address. After this, every PC got its address.

<img width="847" height="508" alt="dhcp and dns setup " src="https://github.com/user-attachments/assets/e71e2c69-4155-4764-b5e4-fcf323c8cb66" />
<img width="534" height="108" alt="DHCP request" src="https://github.com/user-attachments/assets/f0cb8c5d-6679-4652-bc76-da4bc8852c36" />
<img width="391" height="112" alt="ping test after dhcp config" src="https://github.com/user-attachments/assets/b3a3fe6f-2426-4928-bc30-63b9d1e00a5c" />

### Static addresses

The rest of the devices are set statically (IP, subnet mask, gateway and DNS by hand). A device I need to reach by IP shouldn't have DHCP hand it a new address one day, leaving IT confused about why they can't remote in anymore.

| Device | Address |
|---|---|
| lv01-printer-01 | 10.1.10.99/24 (outside the DHCP pool) |
| lv01-atm-01 | 10.1.20.2/29 |
| lv01-fs-01 | 10.1.60.2/24 |
| ADMIN-PC | Static, in 10.1.99.0/24 |

<img width="376" height="204" alt="image" src="https://github.com/user-attachments/assets/05aaa6f0-29a5-40b9-af40-aae1d8dd539e" />
<img width="340" height="208" alt="image" src="https://github.com/user-attachments/assets/a087deb5-a2f3-4da3-9021-3948e3f9d0be" />

The AP stays on DHCP in the MGMT VLAN, which is standard for access points.

Finally, the ping sweep below from PC-01 shows that attempts were successful when trying to reach other devices on the network, across subnets, VLANs, etc.  

<img width="421" height="605" alt="successful ping sweep post config" src="https://github.com/user-attachments/assets/c83ba1a6-850f-4ec2-93a1-0cb4f7bf27df" />

However, it is good news that the network routing is working, but that also means nothing is isolated, and all devices can still reach each other freely. A guest can reach the file server, and staff can reach the ATM, because the router routes between every VLAN by default. This is a MAJOR issue, and the next phase of the project fixes it. 

## 4. Troubleshooting: core banking stopped working

With the internal network working, it was time to test connectivity to the external network. Before the rebuild, every device could reach the core banking system, which is essential for business operations. Now, the ping failed.

<img width="422" height="78" alt="failed corebank pin" src="https://github.com/user-attachments/assets/c2cee518-a735-4ea5-a1ec-7baf16f11d5c" />

With LAN traffic working just fine, I moved up from layer 2 on the switch to layer 3 on the router. My first two suspects were routing and NAT.

1. **Routing table:** the default route and the connected route to the ISP were both correct, so routing checked out.
2. **NAT:** this was it. The source NAT rule only translated 192.168.1.0/24, which doesn't exist anymore.

<img width="556" height="558" alt="nat and route showing" src="https://github.com/user-attachments/assets/d16ac1ff-cf5a-4f87-8c2b-65c25ed0798d" />

To fix this, I replaced the rule's allowed source with 10.1.0.0/16, which matches every subnet at the lv01 office. The rule still uses `masquerade`, which is VyOS's syntax for PAT, which allows every device to share the router's one public address, with a different source port for each connection.

```
set nat source rule 100 source address 10.1.0.0/16
```
<img width="493" height="256" alt="new design nat policy" src="https://github.com/user-attachments/assets/cc8259fc-5d7c-47ab-a921-aa4432d08a92" />

After that, I ping-tested again, and every device could successfully reach core banking again.

<img width="418" height="167" alt="post nat ping to corebank" src="https://github.com/user-attachments/assets/ba124c80-9292-4e1a-b98e-db8042c98926" />

## 5. Correction: moving the AP to management

I originally put the AP in the GUEST VLAN. After some thought, I decided it's better to put APs in the management VLAN so guests can't reach the AP's login page, and map guests to VLAN 50 through the guest SSID instead.

To fix it, I:

1. Released the AP's GUEST DHCP address
2. Made its switch port a trunk, with native VLAN 99 for the AP's own management address and allowed VLANs 10, 50 and 99, since it carries both the STAFF and GUEST SSIDs
3. Created the MGMT vif on the router, with a DHCP scope for 10.1.99.0/24 (.1 gateway, 1.1.1.1 DNS, range .100 – .199)

The actual AP config comes later, on the real hardware (Catalyst 9130). That's where we would configure the virtual interfaces, SSIDs, and the VLAN mapping for the GUEST VLAN and subnet

## Before the lock down

- [ ] Write a change record for the re-addressing, with a rollback plan

## Next: the lock down

The office is rebuilt, but every device can still reach each other on the network, which is NOT ideal. This will be addressed in the next phase. Core opbjectives include:

- GUEST gets the internet only, and cannot access the internal business network
- CARD reaches core banking and nothing else
- MGMT is reachable only from the admin PC
- Port security, DHCP snooping, DAI and storm control on the access ports
