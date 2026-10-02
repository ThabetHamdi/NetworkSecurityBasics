# Device Hardening (Router / Switch)

## Scenario
Baseline template applied to every Cisco device before it goes into production.

## Baseline
```
hostname HQ-R1
ip domain-name corp.local
service timestamps debug datetime msec localtime show-timezone
service timestamps log datetime msec localtime show-timezone
service password-encryption
service tcp-keepalives-in
service tcp-keepalives-out
no service pad
no service config
no ip source-route
no ip bootp server
no ip http server
ip http secure-server           ! only if HTTPS mgmt needed
ip http authentication local
no ip finger
no ip domain-lookup
no cdp run                      ! or per-interface on trusted links only
no lldp run                     ! same
no service udp-small-servers
no service tcp-small-servers

! Credentials
security passwords min-length 12
enable algorithm-type scrypt secret <CHANGE-ME>
username admin privilege 15 algorithm-type scrypt secret <CHANGE-ME>
login block-for 120 attempts 3 within 60
login on-failure log
login on-success log

! SSH only
crypto key generate rsa modulus 4096 label SSH-KEY
ip ssh rsa keypair-name SSH-KEY
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
ip ssh server algorithm encryption aes256-ctr aes256-gcm
ip ssh server algorithm mac hmac-sha2-256 hmac-sha2-512
ip ssh server algorithm kex diffie-hellman-group14-sha1 ecdh-sha2-nistp256

ip access-list standard MGMT-ONLY
 permit 10.10.99.0 0.0.0.255
 deny any log
line vty 0 15
 transport input ssh
 transport output none
 access-class MGMT-ONLY in
 exec-timeout 5 0
 login local                   ! or: login authentication TACACS-LIST
 logging synchronous
line con 0
 exec-timeout 5 0
 login local
line aux 0
 no exec
 transport input none

banner login ^
AUTHORIZED ACCESS ONLY. Activity is monitored and logged. Disconnect now if not authorized.
^

! Control-plane / privilege
privilege exec level 5 show running-config
archive
 log config
  logging enable
  hidekeys
 path flash:archive-
 write-memory
 maximum 14
service config-lock                ! if supported
! Disable unused ports (switch)
! interface range ... shutdown / switchport access vlan 999
! Interface-level
interface GigabitEthernet0/0
 no ip redirects
 no ip unreachables
 no ip proxy-arp
 no ip directed-broadcast
 ip verify unicast source reachable-via rx
```
## Verification
```
show ip ssh
show users
show running-config | section line
show login failures
show archive config differences
```
