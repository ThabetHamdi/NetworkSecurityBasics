# Linux Network Hardening (sysctl, SSH, brute-force protection, auditing)

## /etc/sysctl.d/99-hardening.conf
```
# Forwarding off unless router
net.ipv4.ip_forward = 0
net.ipv6.conf.all.forwarding = 0
# Spoofing / redirects / source-routing
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.secure_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_ra = 0
net.ipv4.conf.all.log_martians = 1
# ICMP
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.icmp_ignore_bogus_error_responses = 1
# SYN flood
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 4096
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_rfc1337 = 1
net.ipv4.tcp_timestamps = 0
# Kernel
kernel.kptr_restrict = 2
kernel.dmesg_restrict = 1
kernel.unprivileged_bpf_disabled = 1
kernel.yama.ptrace_scope = 1
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
net.core.bpf_jit_harden = 2
```
`sysctl --system`

## SSH - /etc/ssh/sshd_config.d/99-hardening.conf
```
Protocol 2
Port 22
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AuthenticationMethods publickey
KbdInteractiveAuthentication no
MaxAuthTries 3
LoginGraceTime 30
ClientAliveInterval 300
ClientAliveCountMax 2
AllowUsers netadmin
AllowTcpForwarding no
X11Forwarding no
PermitEmptyPasswords no
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
LogLevel VERBOSE
Banner /etc/issue.net
```
```bash
ssh-keygen -t ed25519 -a 100 -C "netadmin@corp"
ssh-copy-id -i ~/.ssh/id_ed25519.pub netadmin@host
sshd -t && systemctl reload sshd
sshd -T | grep -Ei 'ciphers|kex|permitroot'
ssh-audit host                      # audit server crypto
```
## fail2ban / CrowdSec
```bash
# /etc/fail2ban/jail.local
[sshd]
enabled = true
maxretry = 3
findtime = 10m
bantime = 1h
backend = systemd
fail2ban-client status sshd
```
## auditd & integrity
```bash
auditctl -w /etc/ssh/sshd_config -p wa -k sshd_cfg
auditctl -w /etc/passwd -p wa -k identity
auditctl -a always,exit -F arch=b64 -S execve -k exec
ausearch -k sshd_cfg -i ; aureport --failed
aide --init ; aide --check          # file integrity
```
## Services / users
```bash
ss -tulpn                           # remove every unneeded listener
systemctl list-unit-files --state=enabled
systemctl disable --now avahi-daemon cups rpcbind
lynis audit system                  # hardening score
chage -M 90 -m 1 -W 14 user
passwd -l olduser
last -a ; lastb ; faillog -a
find / -perm -4000 -type f 2>/dev/null     # SUID audit
```
