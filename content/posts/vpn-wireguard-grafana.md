---
title: "Weekend Project #2 — WireGuard VPN + Grafana Monitoring"
date: 2025-09-27T12:00:00+00:00
draft: false
tags: ["wireguard", "vpn", "grafana", "prometheus", "learning", "devops"]
categories: ["Weekend Projects"]
author: "Bart Fine"
showToc: true
TocOpen: true
hidemeta: false
comments: true
description: "How I set up a personal VPN with WireGuard on a VPS and added Grafana monitoring to track performance."
summary: "Spun up a VPS, built a WireGuard VPN I can connect to from my phone and laptop, and stood up Grafana + Prometheus for monitoring."
---
## Why this project
I’m learning networking/infrastructure by building small, useful things. In this one I wanted a secure VPN I control plus a basic monitoring stack to see what my server is doing.

## Mental model (what I built)
- **VPS**: a tiny always-on Linux machine with a public IP (my “apartment in a skyscraper”).
- **WireGuard**: creates an encrypted tunnel between my devices and the VPS.
- **NAT**: the VPS rewrites packets so the internet sees only the VPS IP; replies are routed back to the right device.
- **Prometheus + exporters**: collect metrics (CPU, memory, ping).
- **Grafana**: shows those metrics on a dashboard (and can alert).

## Steps I took (high level)
1. **Provision** a DigitalOcean droplet (Ubuntu LTS, 1 CPU / 1 GB).
2. **Secure baseline**: upgrade packages; enable UFW; keep SSH open.
3. **Install WireGuard** and generate keys.
4. **Configure server**:
   - VPN subnet: `10.6.0.0/24` (server is `10.6.0.1`).
   - Enable IP forwarding + NAT so clients can reach the internet.
5. **Add clients**:
   - **Phone** (`10.6.0.2`) imported via QR code.
   - **Laptop** (`10.6.0.3`) via a `.conf` file.
6. **Test**:
   - `wg` showed handshakes + traffic counters.
   - `https://icanhazip.com` showed the VPS public IP from both devices.
7. **Monitoring stack** (Docker Compose):
   - `prom/prometheus`, `node-exporter` (host metrics), `prom/blackbox-exporter` (pings), `grafana/grafana-oss`.
   - Grafana panels: CPU %, memory (GB), uptime, ping latency/success.

**Server config looked like** (/etc/wireguard/wg0.conf):
   - [Interface] 'Address = 10.6.0.1/24', 'ListenPort = 51820', 'PrivateKey = <server-private-key>'. 
   - PostUp = iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE; iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT. 
   - PostDown = iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE; iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT'. 

**Client config pattern looked like**
   - [Interface], `PrivateKey = <client-private>`, 'Address = 10.6.0.X/32' 
'DNS = 1.1.1.1'. 

   - [Peer], `PublicKey = <server-public>`, `Endpoint = <VPS-IP>:51820`, `AllowedIPs = 0.0.0.0/0`, 
`PersistentKeepalive = 25`. 

## Key commands (representative)
```bash
# On the VPS
apt update && apt -y upgrade
apt -y install wireguard qrencode ufw chrony curl

# Firewall
ufw allow OpenSSH
ufw enable
ufw allow 51820/udp
ufw allow 3000/tcp   # Grafana UI

# WireGuard keys
umask 077
wg genkey | tee /etc/wireguard/server_private.key | wg pubkey | tee /etc/wireguard/server_public.key

# Enable forwarding
echo 'net.ipv4.ip_forward=1' | tee /etc/sysctl.d/99-wg-forward.conf
sysctl --system

# Bring up WireGuard
wg-quick up wg0
systemctl enable wg-quick@wg0
wg
'''

## Gotchas I fixed:
1. **WireGuard PostUp error**: my iptables commands wrapped onto new lines → WireGuard parser choked. Fix: keep PostUp/PostDown each on one line separated by semicolons 
2. **Windows saved laptop.conf as .txt**: enable “File name extensions” in Explorer, rename to laptop.conf.
3. **Grafana SMTP**: my cloud provider blocks outbound SMTP by default, so email tests timed out. Solution would be: request unblocking or use a relay on port 2525 (SendGrid/Mailgun).

## What I can do now:
- Securley tunnel my phone and laptop through my own VPS
- See CPU, memory, uptime and ping in Grafana
- Extend with more exporters or alerting later
