# AAA with TACACS+ / RADIUS

## Scenario
Central authentication, per-command authorization and accounting for all network staff, with local fallback if servers are unreachable.

## Configuration
```
aaa new-model
tacacs server ISE1
 address ipv4 10.10.99.50
 key <CHANGE-ME>
 timeout 3
tacacs server ISE2
 address ipv4 10.10.99.51
 key <CHANGE-ME>
aaa group server tacacs+ TAC-GRP
 server name ISE1
 server name ISE2
 ip vrf forwarding MGMT           ! if using mgmt VRF
 ip tacacs source-interface Loopback0

aaa authentication login default group TAC-GRP local
aaa authentication enable default group TAC-GRP enable
aaa authorization exec default group TAC-GRP local if-authenticated
aaa authorization commands 15 default group TAC-GRP local
aaa accounting exec default start-stop group TAC-GRP
aaa accounting commands 15 default start-stop group TAC-GRP
aaa authentication login CONSOLE-LOCAL local
line con 0
 login authentication CONSOLE-LOCAL

! RADIUS (802.1X / VPN)
radius server NPS1
 address ipv4 10.10.20.60 auth-port 1812 acct-port 1813
 key <CHANGE-ME>
aaa group server radius RAD-GRP
 server name NPS1
```
## Verification
```
test aaa group tacacs+ admin <PASS> legacy
show aaa servers
show tacacs
show aaa sessions
```
Always keep a local break-glass admin and test **in a second session** before logging out.
