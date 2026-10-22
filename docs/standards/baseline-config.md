# Baseline config

Every device gets this before it carries any traffic. This is all done manually at first, as automation is planned for future stages/projects

- [ ] Hostname w/ naming standard
- [ ] Banner login
- [ ] Local login setup
- [ ] SSH v2 configured for named local accounts only; telnet off
- [ ] Enable secret for privileged CLI (Cisco and Arista)
- [ ] Management only on VLAN 99 (or the loopback for routers)
- [ ] NTP from lv01-edge-01 (from HQ from Stage 3)
- [ ] Logging buffered with timestamps and syslog (to the services VM from Stage 3)
- [ ] Unused ports in VLAN 999 and shut down
- [ ] Native VLAN 998 on every trunk
      
## Cisco IOS (3560-CX)

```

```

## Arista EOS

```

```

Run `zerotouch disable` once on every new Arista node before configuring it.

## VyOS

```

```
