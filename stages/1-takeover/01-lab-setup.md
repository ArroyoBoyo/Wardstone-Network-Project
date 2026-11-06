# Stage 1: Building the "before" network

For the starting stage, I created a simple, bare bones network that's meant to mimic what the old IT company had left Wardstone with when we took over. Nothing fancy, nothing documented, just enough to get everyone online. This is the network I do discovery on, stabilize, and then rebuild in place.

<img width="1076" height="840" alt="First setup" src="https://github.com/user-attachments/assets/d9efc880-d4d0-43be-8691-4bbd750b8897" />

## The layout

```
Internet (GNS3 NAT)
   |
PE-01 (ISP router) ---- Corebank
   |  198.51.100.0/30
WFCU-EDGE-01 (our edge router)
   |  192.168.1.0/24, VLAN 1, everything
MDF-SWITCH-01 ---- AP-01, printer1, ATM, file-server
   |
front-desk-sw ---- PC-09 to PC-12
   |
lending-sw ---- PC-05 to PC-08
   |
back-office-sw ---- PC-01 to PC-04
```

Every time the office needed more ports, the old IT just added another cheap switch onto the end of the last one. The back office's traffic crosses three desk switches and the MDF switch just to reach the file server or the internet.

Corebank hangs off PE-01 on its own network. Core banking is the system that holds every member's accounts, balances and transactions. Small credit unions almost never host it themselves, they rent it from a vendor who runs it in their own data center, and the tellers reach it over the internet. So in the lab, PE-01 plays the whole internet, and the corebank node behind it plays the vendor's data center (which i will dive into in another project).

## The provider router (PE-01)

The internet connects into the provider edge router. PE is what ISPs call their router at the customer's site.

| Interface | Connects to | Address |
|---|---|---|
| eth0 | Internet (GNS3 NAT) | 192.168.122.2/24 |
| eth1 | Our edge router | 198.51.100.1/30 |
| eth2 | Corebank | 192.0.2.1/24 |

The only configs I did on the provider router were setting the IPs, bringing the interfaces up, and a default route out to the internet, since most of the configuration would ofc be on our own network's routers.

```
conf t
int eth0
 ip add 192.168.122.2/24
int eth1
 ip add 198.51.100.1/30
int eth2
 ip add 192.0.2.1/24
ip route 0.0.0.0/0 192.168.122.1
end
write memory
```

<img width="232" height="391" alt="PE-01 config" src="https://github.com/user-attachments/assets/e1f8a877-9f00-41e9-8d59-d473a46ae67c" />

One thing worth pointing out in PE-01's routing table: there's no route to 192.168.1.0/24, our office network. That's correct, and it's how the real internet works. The ISP never sees our private addresses, because our edge router's NAT rewrites everything to 198.51.100.2, which PE-01 already knows as a connected route.

<img width="554" height="184" alt="PE-01 routing table" src="https://github.com/user-attachments/assets/0fbb9915-323a-44c1-9b85-d5121497c4f0" />

## Our edge router (WFCU-EDGE-01)

The business edge router of course needs more love.

| Interface | Connects to | Address |
|---|---|---|
| eth0 | Uplink to the ISP | 198.51.100.2/30 |
| eth1 | Downstream to the internal network | 192.168.1.1/24 |

<img width="603" height="213" alt="WFCU-EDGE-01 interfaces" src="https://github.com/user-attachments/assets/eaacd822-7fa2-4d03-a316-34234a393139" />

### The handoff to the ISP

The link between PE-01 and our edge is a point to point connection, and it's the cable where the ISP's network ends and ours begins. It's a /30 because a point to point link only ever has two devices on it, so it gets the smallest subnet alongside the smallest room for external device interference.

| Address | Who has it |
|---|---|
| 198.51.100.0 | Network address |
| 198.51.100.1 | PE-01, our gateway to the internet |
| 198.51.100.2 | Our edge router, the office's one public IP |
| 198.51.100.3 | Broadcast |

In the real world, the ISP assigns both of these, and they'd be real public IPs. 198.51.100.0/24 is a range reserved for documentation, so I decided to use that.

### Default route

After the IPs, I set the default route to the ISP router as WFCU's route to the outer internet:

```
set protocols static route 0.0.0.0/0 next-hop 198.51.100.1
```

(WFCU-EDGE-01 is running VyOS for labbing purposes, hence the change in syntax) 
### NAT

Next on the edge router, I configured the NAT policy. This is essential for how the devices on the network communicate with the internet, as it translates the unroutable, private IP addresses into a usable, public IP address. It's how the whole office shares its one public IP.

```
set nat source rule 100 outbound-interface name eth0
set nat source rule 100 source address 192.168.1.0/24
set nat source rule 100 translation address masquerade
```

- `outbound-interface name eth0` sets which interface the translation happens on. Anything leaving out of eth0, our publicly linked interface, gets its source address rewritten.
- `source address 192.168.1.0/24` allows any device in the internal network to use it.
- `translation address masquerade` tells VyOS what to translate to. Masquerade means "use eth0's own address," so every inside device shows up as 198.51.100.2. This is PAT, or NAT overload.

The Cisco equivalent would be `ip nat inside source list 1 interface g0/0 overload`.

<img width="385" height="228" alt="image" src="https://github.com/user-attachments/assets/b90a9101-3683-44da-b87d-d7ea330eb690" />

### DHCP

Now that the devices have a way to interact with the internet, the network needs DHCP, so devices that join it know how to get their IP and find their way out.

Traditionally, these services would run on a dedicated server, or multiple servers, but as this is a small office starting from square 1, DHCP is done locally on the router.

```
set service dhcp-server shared-network-name WFCU subnet 192.168.1.0/24 subnet-id 1
edit service dhcp-server shared-network-name WFCU subnet 192.168.1.0/24
set option default-router 192.168.1.1
set range 0 start 192.168.1.11
set range 0 stop 192.168.1.254
top
commit
save
```

<img width="341" height="212" alt="image" src="https://github.com/user-attachments/assets/aaee2f9b-b44b-428a-a223-a69d8bedc2e3" />

## The switches

### MDF-SWITCH-01

No configs made to the switch, as the old IT just bought it and used it to connect more than one device to the internet. Everything sits on its factory defaults, all in VLAN 1.

### The desk switches

When the office ran out of ports, the old IT daisy-chained three cheap unmanaged switches off the MDF switch, one after another:

| Switch | Location | PCs |
|---|---|---|
| front-desk-sw | Teller line and front desk | PC-09 to PC-12 |
| lending-sw | Lending area | PC-05 to PC-08 |
| back-office-sw | Back office | PC-01 to PC-04 |

Unmanaged switches have no IP address and no config, so nothing on the network "knows" they're there. In the lab, they're GNS3's built-in Ethernet switches.

This chain is one of the biggest weaknesses in the office due to the following:

- **Single points of failure:** if lending-sw loses power, the back office goes down with it, even though nothing is wrong back there.
- **Bottlenecks:** eight PCs' traffic shares the single cable between front-desk-sw and MDF-SWITCH-01.
- **Hard to troubleshoot:** with no management, the only way to know what's behind each switch is to trace cables by hand.

<img width="974" height="340" alt="image" src="https://github.com/user-attachments/assets/37b892ae-c917-43c0-a74b-e56c01d778c0" />

## The devices

The office has 12 PCs for 12 staff. The old IT hand-set static IPs on the important devices, and left the rest of the PCs on DHCP:

| Device | Address | Gateway |
|---|---|---|
| Corebank | 192.0.2.10/24 (static) | 192.0.2.1 (PE-01) |
| File server | 192.168.1.10/24 (static) | 192.168.1.1 |
| Printer (printer1) | 192.168.1.50/24 (static) | 192.168.1.1 |
| ATM | 192.168.1.60/24 (static) | 192.168.1.1 |
| PC-01 | 192.168.1.50/24 (static, hand-set) | 192.168.1.1 |
| PC-02 to PC-12 | DHCP (.11 – .254) | 192.168.1.1 |
| AP-01 | Consumer access point, staff and guests on the same Wi-Fi | |

Yes, PC-01 and the printer share the same address. That's on purpose: it's the duplicate IP behind the outage that got the old IT company fired, and it's what discovery should uncover.

AP-01 is a placeholder, since GNS3 can't simulate Wi-Fi. It stands for the consumer access point the old IT plugged into the MDF switch, putting guests on the same network as the ATM. The real Wi-Fi comes in Stage 2 with the Catalyst 9130.

## What I'll be looking at during discovery

The following are issues that will be focused on for this portion of the demenstration:

- **A linear network full of single points of failure:** three desk switches chained one after another, so one dead switch takes down every desk behind it
- **One flat network:** every device, including the ATM and guests on the Wi-Fi, shares one VLAN and one subnet, so anything can reach anything
- **No documentation:** no diagram, no inventory, no labels, no passwords handed over, and inconsistent naming
- The static IPs (.50 and .60) sit inside the DHCP range, and PC-01 and the printer already share .50
- The switch is on factory defaults, with no admin password
- DHCP doesn't hand out a DNS server
