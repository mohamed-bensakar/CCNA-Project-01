# Verification Guide

These are the results the configuration is designed to produce. Run each test in Packet Tracer to confirm it. The first ping may time out while ARP resolves.

PC addresses come from DHCP, so run `ipconfig` on the destination PC first.

## Connectivity Tests

| # | From | To | Expected | Proves |
|:-:|---|---|---|---|
| 1 | PC0 | 192.168.1.1 | Success | Zone 0 gateway |
| 2 | PC0 | a PC in PC9–PC12 | Success | OSPF between areas 0 and 1 |
| 3 | PC0 | a PC in PC22–PC24 | Success | OSPF between areas 0 and 2 |
| 4 | PC18 (Marketing) | a PC in PC22–PC24 (Admins) | Success | Inter-VLAN routing |
| 5 | PC18 (Marketing) | a PC in PC25–PC27 (Finance) | **Blocked** | ACL `MARKETING-IN` |
| 6 | PC25 (Finance) | a PC in PC18–PC21 (Marketing) | **Blocked** | ACL `FINANCE-IN` |
| 7 | PC0 | 200.1.1.2 | Success | Default route and NAT |

## Show Commands

| Feature | Device | Command | Look for |
|---|---|---|---|
| OSPF neighbors | R-ZONE0 | `show ip ospf neighbor` | 2.2.2.2, 3.3.3.3, 4.4.4.4 in FULL state |
| Routes | R-ZONE2 | `show ip route` | `O IA` routes and an `O*E2` default |
| NAT | R-ZONE0 | `show ip nat translations` | Entries using 200.1.1.1 (after test 7) |
| HSRP | R-ZONE1, R2-ZONE1 | `show standby brief` | One Active, one Standby |
| ACL hits | R-ZONE2 | `show access-lists` | Match counters on `deny` (after tests 5 and 6) |
| EtherChannel | Switch1, Switch2 | `show etherchannel summary` | `Po1(SU)` with both ports `(P)` |
| Trunks | Switch0 | `show interfaces trunk` | VLANs 10, 20, 30 allowed |
| Spanning tree | Switch0, 5, 6 | `show spanning-tree` | One blocked port in the triangle per VLAN |

## HSRP Failover Test

1. From PC9, run `ping -t 192.168.1.1`.
2. Use `show standby brief` to find the Active router.
3. Shut down its LAN interface (`Gi0/1` on R-ZONE1, `Gi0/0` on R2-ZONE1).
4. Replies should resume after a short timeout, with the other router now Active.
5. Restore the interface with `no shutdown`.
