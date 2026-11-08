# Lock down, part 3: Management access

[← Lock down overview](README.md)

This part isn't too extensive, but it's very important for base-level security and access management.

SSH is a very powerful tool for admins, but without proper access control, anyone has the chance to access equipment remotely, which isn't ideal for a company of any size. An adversary would still need to know the user, password, IP and so on, but mitigating the risk of unauthorized users getting in should be standard.

The devices this part focuses on:

- The switch
- The edge router
- The firewall

## 1. The switch

**Issue:** Any device that can reach the switch can get to its SSH login. Telnet is already off, so SSH is the protocol to focus on.

**Fix:** An ACL that only allows the admin PC, applied to SSH

To accomplish this, I implemented the following commands:

```
ip access-list standard MGMT
permit host 10.1.99.10
management ssh
ip access-group MGMT in
```

Although the syntax is a bit different from what it would be on a Cisco device, as you can see below, the idea is very similar, and it even has nearly identical syntax for commands.

```
access-list 1 permit host 10.1.99.10
line vty 0 15
access-class 1 in
```

<img width="404" height="323" alt="Screenshot 2026-11-07 152723" src="https://github.com/user-attachments/assets/1763a336-3474-4d47-a57f-feedd90a77a4" />

**Why:** This creates a rule that only accepts SSH from the admin PC. It seems extremely basic, but it does the job, and it's arguably one of the best ways to limit SSH, as it's simple and it runs natively on the device instead of on an external program or firewall.

That's all that's needed, since ACLs have an implicit deny at the end unless stated otherwise.

The ACL is applied in the SSH config mode (`management ssh`), so it only applies when someone tries to reach the switch over SSH. It doesn't affect the console login, which has its own set of rules.

Now, anytime someone tries to SSH into the switch from any address other than 10.1.99.10, access is blocked.

## 2. The edge router

**Issue:** I was unfamiliar with how VyOS handles SSH, and found an important point: SSH is enabled on all interfaces by default. That even includes the WAN port, which faces the open internet.

**Fix:** Make SSH listen only on the router's internal-facing address:

```
set service ssh listen-address 10.1.254.1
```

<img width="570" height="538" alt="Screenshot 2026-11-07 224104" src="https://github.com/user-attachments/assets/a175cb85-1d59-4aba-9429-78a4ba2e793a" />

**Why:** The router now only answers SSH sent to 10.1.254.1, its address on the link to the firewall, and never on the internet-facing side. The firewall rules already only allow the admin PC to reach 10.1.254.1.

## 3. The firewall

**Issue:** The firewall's rules were already in place, so there were only a few settings left

SSH is disabled by default on OPNSense, so I made sure to enable it and enable the settings below

**Fix:** Under **System → Settings → Administration**, I enabled SSH and set it to listen only on the MGMT (VLAN 99) interface.

<img width="1009" height="709" alt="Screenshot 2026-11-08 003626" src="https://github.com/user-attachments/assets/4caa87be-c5e9-4d25-80ec-0132266855f4" />

**Why:** SSH is only reachable from the management VLAN, and the MGMT firewall rules only give the admin PC access to the firewall itself.

Although these changes were minor in complexity, they greatly decreased the threat area surrounding attempts to access critical network devices via SSH!
