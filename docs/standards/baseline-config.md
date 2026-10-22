# Baseline config

Every device gets this before it carries any traffic. Check it off in the device's config file header.

- [ ] Hostname following the naming standard
- [ ] Login banner: authorized use only
- [ ] SSH v2 with named local accounts; telnet off
- [ ] Enable secret set (Cisco and Arista)
- [ ] Management only on VLAN 99 (or the loopback for routers)
- [ ] NTP from lv01-edge-01 (from HQ from Stage 3)
- [ ] Logging buffered with timestamps; syslog to the services VM from Stage 3
- [ ] Unused ports in VLAN 999 and shut down
- [ ] Native VLAN 998 on every trunk
- [ ] A description on every port in use
- [ ] Config saved, and exported to Git with passwords replaced by placeholders

## Cisco IOS (3560-CX)

```
hostname lv01-sw-01
banner login ^Authorized use only. Activity is logged.^
service password-encryption
enable secret <PLACEHOLDER>
username <name> privilege 15 secret <PLACEHOLDER>
ip domain name lv01.wardstone.internal
crypto key generate rsa modulus 2048
ip ssh version 2
line vty 0 15
 transport input ssh
 login local
line con 0
 login local
ntp server 10.1.99.1
service timestamps log datetime msec
logging buffered 64000
interface range Gi1/0/6 - 8
 switchport mode access
 switchport access vlan 999
 shutdown
```

## Arista EOS

```
hostname lv01-sw-01
banner login
Authorized use only. Activity is logged.
EOF
enable password <PLACEHOLDER>
username <name> privilege 15 secret <PLACEHOLDER>
management ssh
   idle-timeout 30
management telnet
   shutdown
ntp server 10.1.99.1
logging buffered 64000
interface Ethernet6-8
   switchport access vlan 999
   shutdown
```

Run `zerotouch disable` once on every new Arista node before configuring it.

## VyOS

```
set system host-name lv01-edge-01
set system login banner pre-login 'Authorized use only. Activity is logged.'
set system login user <name> authentication plaintext-password <PLACEHOLDER>
set service ssh port 22
set service ntp server pool.ntp.org
set system syslog local facility all level info
```
