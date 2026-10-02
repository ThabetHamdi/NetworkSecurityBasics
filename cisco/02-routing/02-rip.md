# RIPv2

## Scenario
Small legacy branch (BR2) with 3 routers running RIPv2 toward HQ. (Use only for tiny/legacy networks; prefer OSPF/EIGRP.)

## Configuration
```
key chain RIP-KEYS
 key 1
  key-string <CHANGE-ME>
  cryptographic-algorithm md5

router rip
 version 2
 no auto-summary
 passive-interface default
 no passive-interface GigabitEthernet0/1
 network 10.0.0.0
 distribute-list prefix BR2-OUT out GigabitEthernet0/1
 default-information originate
 timers basic 30 180 180 240

interface GigabitEthernet0/1
 ip rip authentication mode md5
 ip rip authentication key-chain RIP-KEYS
 ip summary-address rip 10.30.0.0 255.255.0.0

ip prefix-list BR2-OUT seq 5 permit 10.30.0.0/16 le 24
```
## Verification
```
show ip protocols
show ip rip database
show ip route rip
debug ip rip          ! lab only
```
## Notes
Max 15 hops, 30 s updates, slow convergence; split-horizon on by default. RIPng for IPv6: `ipv6 router rip NAME` + `ipv6 rip NAME enable`.
