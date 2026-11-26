# Planned Improvements

These changes are **not yet applied** to the `.pkt` file or to `configs/`.

## 1. Remove DHCP address overlaps

Some pools include their own server's static address, which can cause a conflict.

| Server | Current | Planned change |
|---|---|---|
| DHCP-Z0 | Pool .2–.255 includes server .100 | Start the pool at .101, maximum 150 users |
| DHCP-Z1 | Pool .10–.109 includes server .100 | Start the pool at .101 |
| DHCP-VLAN10 | Pool .2–.51 includes server .50 | Move the server to 192.168.3.100 |

DHCP-Z0 also has a second pool (`serverPool`) on the same network with no default gateway. It should be changed so it cannot lease addresses in 192.168.1.0/24.

## 2. Match the EtherChannel trunk settings

The member ports allow only VLAN 20, but `Port-channel1` has no allowed list. On Switch1 and Switch2:

```cisco
interface Port-channel1
 switchport trunk allowed vlan 20
```

## 3. Make HSRP predictable

Neither router has a set priority, preemption or uplink tracking.

```cisco
! R-ZONE1
interface GigabitEthernet0/1
 standby 10 priority 105
 standby 10 preempt
 standby 10 track Serial0/0/0

! R2-ZONE1
interface GigabitEthernet0/0
 standby 10 preempt
```

## 4. Secure device access

No `enable secret` or remote login is configured yet. Use your own strong passwords.

```cisco
enable secret <password>
username admin secret <password>
ip domain-name lab.local
crypto key generate rsa
line console 0
 login local
line vty 0 4
 login local
 transport input ssh
```

Also planned: port security on access ports and shutting down unused ports.

## 5. Tidy naming

- Give each switch a unique hostname (all are currently `Switch`).
- Use one VLAN numbering scheme: VLAN 20 is ADMINS in Zone 0 but FINANCE in Zone 2.
- Remove the unused VLAN 30 on Switch2.
