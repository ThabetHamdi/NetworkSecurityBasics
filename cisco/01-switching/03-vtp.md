# VTP

## Scenario
Centralize VLAN creation on one switch in a 6-switch campus. **Best practice: prefer `vtp mode off`/transparent.** If used, use VTPv3 with password.

## VTPv3
```
! Server (CORE1)
vtp domain HQ
vtp version 3
vtp mode server
vtp password <CHANGE-ME> hidden
! After creating VLANs:
vtp primary vlan force

! Clients
vtp domain HQ
vtp version 3
vtp mode client
vtp password <CHANGE-ME> hidden

! Disable VTP completely (recommended on access/edge)
vtp mode off
```
## Verification
```
show vtp status
show vtp password
show vlan brief
```
## Pitfalls
- A new switch with a **higher revision number** can wipe the domain VLANs (VTPv1/2). Reset revision: change mode to transparent then back, or change domain name.
- VTP is exchanged over trunk ports only.
