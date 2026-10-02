# VPN on Linux: WireGuard & strongSwan (IPsec/IKEv2)

## WireGuard site-to-site / road-warrior
```bash
umask 077
wg genkey | tee server.key | wg pubkey > server.pub
wg genpsk > peer1.psk
```
`/etc/wireguard/wg0.conf` (server)
```ini
[Interface]
Address = 172.16.50.1/24
ListenPort = 51820
PrivateKey = <SERVER_PRIVATE_KEY>
PostUp   = nft add rule inet filter forward iifname wg0 accept
PostDown = nft flush chain inet filter forward

[Peer]
PublicKey = <PEER1_PUBLIC_KEY>
PresharedKey = <PSK>
AllowedIPs = 172.16.50.2/32, 10.20.0.0/16
```
Client:
```ini
[Interface]
Address = 172.16.50.2/32
PrivateKey = <CLIENT_PRIVATE_KEY>
DNS = 10.10.20.53
[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
PresharedKey = <PSK>
Endpoint = 198.51.100.1:51820
AllowedIPs = 10.10.0.0/16
PersistentKeepalive = 25
```
```bash
systemctl enable --now wg-quick@wg0
wg show ; wg showconf wg0
```
## strongSwan IKEv2 (interoperates with Cisco)
`/etc/swanctl/conf.d/branch.conf`
```
connections {
  hq-branch {
    version = 2
    local_addrs = 203.0.113.2
    remote_addrs = 198.51.100.1
    proposals = aes256-sha256-ecp256          # = AES256/SHA256/DH19
    dpd_delay = 10s
    local  { auth = psk; id = 203.0.113.2 }
    remote { auth = psk; id = 198.51.100.1 }
    children {
      lan {
        local_ts  = 10.20.0.0/16
        remote_ts = 10.10.0.0/16
        esp_proposals = aes256gcm16-ecp256
        start_action = start
        dpd_action = restart
      }
    }
  }
}
secrets { ike-hq { id = 198.51.100.1 ; secret = "<CHANGE-ME>" } }
```
```bash
swanctl --load-all
swanctl --list-sas
swanctl --initiate --child lan
ip xfrm state ; ip xfrm policy
```
Allow UDP 500/4500 + ESP in nftables (see `03-firewall-nftables.md`).
