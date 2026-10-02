# IP SLA, Object Tracking & EEM

```
ip sla 10
 icmp-echo 8.8.8.8 source-interface Gi0/0
 frequency 10
ip sla schedule 10 life forever start-time now
ip sla 11
 http get http://192.0.2.100/health
 frequency 30
ip sla schedule 11 life forever start-time now
track 10 ip sla 10 reachability
track 11 ip sla 11 state
track 20 list boolean and
 object 10
 object 11

! Delay-based failover
ip sla 12
 udp-jitter 203.0.113.2 16384 codec g711ulaw
track 12 ip sla 12 state          ! goes down if probe threshold is violated (set 'threshold' in the SLA)

! EEM: auto-backup running-config on every config change
event manager applet BACKUP-CFG
 event syslog pattern "%SYS-5-CONFIG_I"
 action 1.0 cli command "enable"
 action 2.0 cli command "copy running-config flash:backup-cfg"
 action 3.0 syslog msg "Config backed up"
```
```
show ip sla statistics
show track
show event manager policy registered
```
