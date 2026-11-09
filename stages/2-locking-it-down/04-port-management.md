# Lock down, part 4: Port security

[← Lock down overview](README.md)

Now that the rest of the network is locked down, it's time to focus on the first line of defense on the LAN: the access layer. This part is very important because it stops users, employees, and non-employees alike from harming the network, whether they mean to or not.

For this part, I got hands-on and consoled into my real Cisco switch (Catalyst 3560-CX) to demonstrate these features.

<img width="645" height="597" alt="Screenshot 2026-11-08 085737" src="https://github.com/user-attachments/assets/5dec3e6d-d2d4-47f4-ae97-bcf7e591b992" />

> **Side note:** The IPs and MACs in this part are purely for the demonstration, and don't tie back to the Wardstone network. But if I were to bring these features into Wardstone's access layer, this is how I'd do it.

This part covers three features:

1. DHCP snooping
2. Dynamic ARP Inspection (DAI)
3. Port security

Spanning tree protection with BPDU guard is also very common, but it's not covered here: it's already configured on every active access port from the rebuild.

## 1. DHCP snooping

**Why DHCP Snooping is important:** An adversary can send server-side DHCP messages to trick devices on the network into treating them as the DHCP server, default gateway, DNS server, and so on. That opens the door to DHCP poisoning, DHCP exhaustion, and man-in-the-middle attacks.

**Fix:** DHCP snooping, which can be enabled via a two-step process:

1. Enable it globally.
2. Enable it per VLAN.

<img width="781" height="292" alt="Screenshot 2026-11-08 100237" src="https://github.com/user-attachments/assets/3a8837e5-ebb6-4a1e-a858-f58f74c2e4e6" />

Small note, the VLAN part of the command is a bit misleading. It only turns DHCP snooping on or off for specific VLANs. It doesn't give each VLAN its own settings, since those are configured per interface.

Next, I trusted the upstream-facing port, the one facing the DHCP server.

<img width="482" height="72" alt="Screenshot 2026-11-08 100637" src="https://github.com/user-attachments/assets/7f3104df-c1d7-4217-96b8-331e2c555522" />

**Why:** Every port is untrusted by default. Trusting a port says "this port is supposed to receive server-side DHCP messages, so don't check them". Every other port can only send client-side messages, so a rogue DHCP server on a desk port gets ignored.

### Rate limiting

**Issue:** There are very few, if any, situations where an end device needs to send a huge number of DHCP requests in a short time. That kind of activity usually means a DHCP exhaustion attack, where an adversary drains all the addresses the DHCP server can hand out.

**Fix:** A rate limit on the port, such as 10 DHCP packets per second. Past that, the port disables itself.

<img width="597" height="182" alt="Screenshot 2026-11-08 101748" src="https://github.com/user-attachments/assets/611fa246-89e4-4f65-905e-062c27a61e6f" />

That raised a question: is there a way to turn the port back on automatically, so that engineers or admins dont have to manually go in and re-enable the interface?

Yes, via errdisable recovery. To enable this, i used the following command:

<img width="823" height="727" alt="Screenshot 2026-11-08 104721" src="https://github.com/user-attachments/assets/4be237ea-8e44-4a09-bb09-d72e3456b37d" />

**Why:** After 5 minutes (the default errdisable recovery time), the disabled port comes back online on its own, so engineers and admins don't have to go in and re-enable ports by hand.

Although it wouldn't be hard for an engineer or admin to go in and manually enable the interface, setting a strong base, and then mitigating the need for manual interaction is a very important.

### Option 82

**What is Option 82?:** Option 82 is a field that DHCP relay agents and snooping-enabled switches inject into client-side DHCP packets. The switch adds it by default, and it causes problems when other snooping-enabled devices receive those packets on untrusted ports.

**Fix:** One global command that stops the switch from adding option 82.

<img width="587" height="61" alt="Screenshot 2026-11-08 105633" src="https://github.com/user-attachments/assets/cf056429-0232-46f5-b442-981fb7784063" />
<img width="734" height="505" alt="Screenshot 2026-11-08 110537" src="https://github.com/user-attachments/assets/0d97a141-470d-4b9a-b1e6-da33ebf702ad" />

## 2. Dynamic ARP Inspection (DAI)

**What is ARP Inspection and Why is it Important?:** ARP has no way to verify who's telling the truth. An adversary can send ARP messages claiming to be another device, like the default gateway ("I have IP 192.168.1.1", for example), so traffic meant for the gateway goes to them instead.

**Fix:** DAI. It's similar to DHCP snooping, and actually works together with it. Like DHCP snooping, it has trusted and untrusted ports.

When a device completes a DHCP handshake (DORA) through a snooping-enabled switch, its IP and MAC are saved in the DHCP snooping binding table. DAI checks every ARP message (requests and replies) against that table. If the source MAC and IP don't match an entry, DAI drops the message.

In the gateway example, the adversary's MAC was never given that IP by DHCP, so there's no matching entry, and DAI drops the message.

To enable DAI, I turned it on for the VLAN and trusted the upstream-facing port, just like DHCP snooping.

<img width="457" height="76" alt="Screenshot 2026-11-08 121229" src="https://github.com/user-attachments/assets/f4900558-bbcf-4765-be95-0186fcd22e0b" />

**Note:** DAI gives a bit more leeway than DHCP snooping on trusted ports. A port facing downstream can be trusted if it connects to other network equipment, or if the switch downstream also runs DAI. Anything with a host on it should stay untrusted.

### Static devices: ARP ACLs

**Issue:** A static device never completes a DHCP handshake, so it never gets an entry in the binding table, and DAI would block it.

**Fix:** An ARP ACL that matches the device's IP to its MAC, set manually by the engineer. Here's the process I took to create one and apply it for an example static device:

<img width="774" height="359" alt="Screenshot 2026-11-08 123154" src="https://github.com/user-attachments/assets/87b314fe-212a-4590-8651-8dd09229466e" />
<img width="579" height="300" alt="Screenshot 2026-11-08 123311" src="https://github.com/user-attachments/assets/b26443b5-51ac-408a-be87-0ecb078a2bb7" />

**Why this Matters:** DAI now has two references to check: the DHCP snooping binding table and the ARP ACL. If neither has a match, the traffic is dropped.

In addition, everything can be checked and validated with the right show commands:

<img width="764" height="826" alt="Screenshot 2026-11-08 123637" src="https://github.com/user-attachments/assets/46d7aafa-08da-4027-80eb-a16925d4bfcf" />
<img width="557" height="97" alt="Screenshot 2026-11-08 123846" src="https://github.com/user-attachments/assets/0f71a58d-76af-4f85-b8ba-bf1fcf960fc0" />

## 3. Port security

**Why Port Security is Important:** Some ports are meant for one device, or a select handful, like the ports for the ATM or the servers in the Wardstone network. But anything plugged into them gets on the network.

**Fix:** Port security. It's slightly different from DHCP snooping and DAI, as those two are enabled per VLAN, while port security is entirely per interface. If an outside device tries to connect, the port can shut down or block the traffic.

<img width="499" height="230" alt="Screenshot 2026-11-08 125658" src="https://github.com/user-attachments/assets/a89295c4-c310-44ff-b9ba-5dde8ccbaccf" />

Port security comes with a handful of default settings. Once enabled, the violation mode is shutdown, and the maximum is 1 MAC address. I already had my spare MacBook plugged in when I ran the command, so it grabbed that MAC and filled the one spot.

<img width="538" height="307" alt="Screenshot 2026-11-08 130209" src="https://github.com/user-attachments/assets/51e73a8b-017f-4f79-8288-e145a7d1c527" />
<img width="1372" height="884" alt="image" src="https://github.com/user-attachments/assets/be3f79bb-c348-4473-82cb-f9b3f8dfdda2" />

Although this looks correct on the surface, there are a few big issues that we will address in the following sections

### Dynamic MACs don't stick

**Issue:** By default, port security learns MACs dynamically. It grabs the first MAC address that sends traffic through the interface. But if the port goes down, whether it's shut down, turned off, or the device is unplugged, the learned MAC is erased, and port security grabs the next MAC that shows up.

A simple unplug and replug let me plug in my other laptop with zero repercussions:

<img width="917" height="827" alt="Screenshot 2026-11-08 141225" src="https://github.com/user-attachments/assets/9feb1cd9-f4d9-4b95-9540-8a40787c1b10" />

In a sense, this hole in the system defeats the purpose of the port security command, which is to make sure only one specific device, or a handful of devices, can use one specific port.

**Fix:** Sticky MAC. The port only accepts traffic from the MAC addresses it has "stuck" to (assigned, in other words). A device was already plugged in when I configured it, so that device's MAC became the sticky MAC.

<img width="686" height="582" alt="Screenshot 2026-11-08 142302" src="https://github.com/user-attachments/assets/ac7910ce-8ac0-44df-8ea9-ac827de2453c" />

**Why:** Sticky MACs are saved into the running config, so they stay even when the port goes down. Saving the config keeps them through a reboot too, which should be a standard procedure after every config change anyways. 

<img width="825" height="291" alt="Screenshot 2026-11-08 151041" src="https://github.com/user-attachments/assets/1d6b7183-7996-4247-9a0a-8011d3e93d3e" />

### Testing

I connected the MacBook from earlier, and port security did its thing: it shut the port down, per the violation mode. The interface's port security status shows the violation count going up and the status as shutdown.

<img width="918" height="436" alt="Screenshot 2026-11-08 151342" src="https://github.com/user-attachments/assets/c7328d61-f495-4eea-9485-fa3cf382a6c2" />
<img width="540" height="305" alt="Screenshot 2026-11-08 152021" src="https://github.com/user-attachments/assets/6e12a4d4-738b-461c-82e7-87721f0159e0" />

Just like with DHCP snooping and ARP inspection, errdisable recovery can bring the port back online automatically after a shutdown. Enabling this feature is similiar across the board.

<img width="665" height="504" alt="Screenshot 2026-11-08 153156" src="https://github.com/user-attachments/assets/b78e88ba-5208-4554-957f-d2ef23053a6e" />

This shows how powerful the setting is:

Even when the port comes back online, the sticky MAC stays, so the wrong device keeps getting shut down forever until the correct device gets reconnected or the wrong device is not connected

Simple yet extremely effective, and can be customized across all ports for various reasons

<img width="914" height="357" alt="Screenshot 2026-11-08 153706" src="https://github.com/user-attachments/assets/df619ee2-3c87-42ce-ad5e-9fab3b0867ed" />

## Wrap-up

All three features could go much deeper than what we did in this situation, but what's here is a solid baseline that could be replicated across Wardstone and many other networks.

For example, for port security, you can change the violation mode so that it just blocks traffic and logs the activity, compared to shutting down the port

For ARP Inspection, you can add additional verification checks, such as the source MAC address in the Ethernet frame. This is a separate field from the ARP payload, which holds the sender MAC and sender IP, which is sourced from the DHCP Snooping Binding Table.

Nonetheless, these configurations are a massive addition to the overall security posture of a network. 
