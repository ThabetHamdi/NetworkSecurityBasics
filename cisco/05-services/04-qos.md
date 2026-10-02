# QoS (Voice / Business / Best-effort)

## Scenario
Protect VoIP (EF), signaling (CS3) and business apps on a 20 Mbps WAN link with LLQ + CBWFQ and shaping.

```
class-map match-any VOICE
 match dscp ef
class-map match-any SIGNALING
 match dscp cs3
class-map match-any BUSINESS
 match dscp af31 af21
class-map match-any SCAVENGER
 match dscp cs1

! Classification at the edge (trust boundary)
ip access-list extended VOIP-RTP
 permit udp any any range 16384 32767
class-map match-all MARK-VOICE
 match access-group name VOIP-RTP
policy-map MARK-IN
 class MARK-VOICE
  set dscp ef
 class class-default
  set dscp default
interface Vlan30
 service-policy input MARK-IN

! WAN egress
policy-map WAN-CHILD
 class VOICE
  priority percent 30
 class SIGNALING
  bandwidth percent 5
 class BUSINESS
  bandwidth percent 35
  random-detect dscp-based
 class SCAVENGER
  bandwidth percent 1
 class class-default
  fair-queue
policy-map WAN-SHAPER
 class class-default
  shape average 20000000
  service-policy WAN-CHILD
interface GigabitEthernet0/0
 service-policy output WAN-SHAPER

! Switch: trust phone DSCP only
interface Gi1/0/1
 mls qos trust device cisco-phone     ! or: auto qos voip cisco-phone
```
```
show policy-map interface Gi0/0
show mls qos interface Gi1/0/1
```
