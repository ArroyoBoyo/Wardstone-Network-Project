# Stage 3, part 2: Core build

[← Stage 3 overview](README.md)

This section involves the actual building out of the core infrastructure.

## The main building

Seeing as the main building holds the MDF and other critical site devices, I decided to start there.

First, I built out the main infrastructure: the edge router, the distribution layer and the access layer. No configurations were made just yet, but I wanted to get a lay of the land first.

<img width="738" height="414" alt="Screenshot 2026-11-10 223202" src="https://github.com/user-attachments/assets/5a1699c2-c9b6-4732-b92d-2a2fe6ab8f04" />

As you can see, there are 2 cables between the distribution switches. Those cables will later be configured as an LACP bundle. This will not only allow for better bandwidth between the switches, but also provide a redundant link in case one of the connections fails.

Each access switch also has two uplinks, one to each distribution switch. These will be bundled with LACP across both switches, so if a distribution switch or an uplink goes down, the access switch stays connected through the other one, and both links carry traffic while everything's healthy.

Yes, in the case a device or link goes down, the network would potentially suffer in terms of performance, but it would still remain active, and would give the networking/admin team enough time to assess the issue.

## Connecting the hosts

Next, I connected the staff and other devices to their respective access switches.

Traditionally, host devices only have 1 Ethernet port, which means if the switch a device is connected to goes down, the device goes down with it. To mitigate this, I split up the staff and other host devices across the access switches. This way, even if one access switch goes down, part of the staff computers and business equipment will remain up and operational.

This setup would also allow for easy isolation, because if one half of the building connected to switch A is down, but the ones connected to switch B are up, we know where to start looking

I also placed the other host devices served by the main building's access switches, such as the APs and printers.

<img width="717" height="562" alt="Screenshot 2026-11-10 234619" src="https://github.com/user-attachments/assets/a1863fbc-84ec-47dc-80a6-862f6defa1a0" />

**Small note:** for the sake of clutter, the PCs present are a small representation of the entire office, as are the printers and APs. For example, in an office this size, there would usually be another 3–5 printers spread throughout the office. And depending on how long the office is, there might be another 2–3 APs to make sure the entire office is covered, spread across channels 1, 6 and 11 on 2.4 GHz to avoid overlapping channels.

Nonetheless, all these concepts and build configurations would stay true, even if we decided to populate the entire board with all 50 PCs, phones, etc.

You may also have noticed that there's only 1 firewall and 1 router, which contradicts our redundancy objective for this section. This will be addressed later in the project.

## Segmenting the network

Next, I visually segmented the network, making it easier to understand which devices go in which VLAN, and how the devices are set up and spread across the access switches.

<img width="1178" height="1134" alt="01" src="https://github.com/user-attachments/assets/30ebd020-1ac4-498a-a4f6-4aec2a178667" />

Now obviously, this diagram is arguably "too pretty", as a real production MDF closet and network layout would likely be much more chaotic. However, I made it as pretty and organized as possible for the sake of the project presentation.

## The lending and call center buildings

Now that the MDF closet and the main building's physical infrastructure are set up, I started on the lending and call center buildings. This helps get the full lay of the land, and shows which devices will truly be where in the new campus.

This completes the foundation for the physical structure of the campus.

<img width="2275" height="1814" alt="FUll building lay out" src="https://github.com/user-attachments/assets/96997ebf-c845-41e4-accc-4cacc2d40a52" />

As you can see, redundancy is plastered all over the current setup:

- **Servers** are dual-homed, with an active and a standby link.
- **Access switches in the IDF closets** link to each distribution switch through 2 separate fiber connections, in separate conduits.
- **Hosts** are split among the access switches where possible. If a single switch goes offline, half of the office is still on site and able to operate while the network team investigates the issue.

## A voice VLAN for the call center

In addition to the design portion, I also decided to flip the script on the call center and configure a dedicated voice VLAN (30) for the IP phones.

Smaller offices often leave phones on the same VLAN as the staff PCs. But in a dedicated call center, it's best practice to separate voice traffic, for quality and security reasons.

The main plus, in my opinion, is that it makes QoS much easier to configure, so voice traffic can be prioritized for stability and call quality, which is essential for operations and member relations.

The configuration of this, as well as the rest of the network, will be touched on in the next section.
