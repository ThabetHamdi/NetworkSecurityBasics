# EIGRP (Named mode)

## Scenario
HQ core + Branch 1 run EIGRP AS 100 with authentication, stub branch, summarization, unequal-cost load balancing and BFD.

## HQ-R1
```
router eigrp CORP
 address-family ipv4 unicast autonomous-system 100
  eigrp router-id 1.1.1.1
  af-interface default
   passive-interface
   hello-interval 5
   hold-time 15
  exit-af-interface
  af-interface GigabitEthernet0/1
   no passive-interface
   authentication mode hmac-sha-256 0 <CHANGE-ME>
   summary-address 10.10.0.0 255.255.0.0
   bfd
  exit-af-interface
  af-interface Tunnel0
   no passive-interface
   authentication mode hmac-sha-256 0 <CHANGE-ME>
  exit-af-interface
  topology base
   variance 2                          ! unequal-cost load balancing
   maximum-paths 4
  exit-af-topology
  network 10.10.0.0 0.0.255.255
  network 172.16.255.0 0.0.0.255
 exit-address-family
```
## Branch (stub)
```
router eigrp CORP
 address-family ipv4 unicast autonomous-system 100
  eigrp stub connected summary
  eigrp router-id 11.11.11.11
  af-interface default
   passive-interface
  exit-af-interface
  af-interface Tunnel0
   no passive-interface
   authentication mode hmac-sha-256 0 <CHANGE-ME>
  exit-af-interface
  network 10.20.0.0 0.0.255.255
  network 172.16.255.0 0.0.0.255
```
## Classic-mode equivalent (older IOS)
```
router eigrp 100
 no auto-summary
 passive-interface default
 no passive-interface Gi0/1
 network 10.0.0.0 0.255.255.255
interface Gi0/1
 ip authentication mode eigrp 100 md5
 ip authentication key-chain eigrp 100 EIGRP-KEYS
 ip summary-address eigrp 100 10.10.0.0 255.255.0.0
```
## Verification
```
show ip eigrp neighbors
show ip eigrp topology
show ip eigrp interfaces detail
show ip route eigrp
show ip protocols
```
Notes: K-values must match; default metric uses bandwidth+delay; `eigrp stub` limits query scope.
