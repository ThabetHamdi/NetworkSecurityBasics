# CoPP, uRPF & Zone-Based Firewall

## Control-Plane Policing
```
ip access-list extended COPP-ROUTING
 permit ospf any any
 permit tcp any any eq bgp
 permit tcp any eq bgp any
 permit eigrp any any
 permit udp any any eq 3784        ! BFD
ip access-list extended COPP-MGMT
 permit tcp 10.10.99.0 0.0.0.255 any eq 22
 permit udp host 10.10.99.60 any eq snmp
ip access-list extended COPP-ICMP
 permit icmp any any
class-map match-any CM-ROUTING
 match access-group name COPP-ROUTING
class-map match-any CM-MGMT
 match access-group name COPP-MGMT
class-map match-any CM-ICMP
 match access-group name COPP-ICMP
policy-map COPP
 class CM-ROUTING
  police 256000 conform-action transmit exceed-action transmit
 class CM-MGMT
  police 128000 conform-action transmit exceed-action drop
 class CM-ICMP
  police 32000 conform-action transmit exceed-action drop
 class class-default
  police 64000 conform-action transmit exceed-action drop
control-plane
 service-policy input COPP
```
## uRPF
```
interface GigabitEthernet0/0
 ip verify unicast source reachable-via rx          ! strict (single-homed edge)
interface GigabitEthernet0/2
 ip verify unicast source reachable-via any         ! loose (multi-homed)
```
## Zone-Based Firewall (inside → outside)
```
zone security INSIDE
zone security OUTSIDE
zone security DMZ
class-map type inspect match-any CM-INSIDE-OUT
 match protocol tcp
 match protocol udp
 match protocol icmp
class-map type inspect match-any CM-OUT-DMZ
 match access-group name OUT-TO-DMZ
ip access-list extended OUT-TO-DMZ
 permit tcp any host 192.0.2.10 eq 443
policy-map type inspect PM-IN-OUT
 class type inspect CM-INSIDE-OUT
  inspect
 class class-default
  drop log
policy-map type inspect PM-OUT-DMZ
 class type inspect CM-OUT-DMZ
  inspect
 class class-default
  drop log
zone-pair security ZP-IN-OUT source INSIDE destination OUTSIDE
 service-policy type inspect PM-IN-OUT
zone-pair security ZP-OUT-DMZ source OUTSIDE destination DMZ
 service-policy type inspect PM-OUT-DMZ
interface Gi0/1
 zone-member security INSIDE
interface Gi0/0
 zone-member security OUTSIDE
interface Gi0/2
 zone-member security DMZ
```
Traffic to/from the router itself uses the *self* zone; add `zone-pair ... source OUTSIDE destination self` with IKE/ESP/BGP allowed.
## Verification
```
show policy-map control-plane
show ip interface Gi0/0 | include verify
show zone security
show policy-map type inspect zone-pair sessions
```
