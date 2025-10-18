# 🧱 Pi-hole-on-Pi5
**From ads everywhere → to a clean, private web.**

Turn your **Raspberry Pi 5** into a **network-wide ad blocker and DNS privacy hub** using **Pi-hole + Unbound + Tailscale**.  
Everything — from YouTube to mobile apps — gets filtered and accelerated right at the network layer.

---

## 🌟 Features
- 🚫 Block ads & trackers network-wide  
- 🔒 Run your own recursive DNS resolver (Unbound)  
- 🌍 Secure remote access via Tailscale  
- ⚡ Faster browsing with local DNS caching  
- 🧠 Fully self-hosted — no external dependencies  

---

## 🧠 Overview
This setup turns your Raspberry Pi 5 into the **brain** of your home network.  
It intercepts all DNS requests and decides which ones to allow or block — stopping ads, telemetry, and analytics before they even reach your devices.

🧩 **Components**
| Component | Purpose |
|------------|----------|
| **Pi-hole** | DNS-level ad blocker |
| **Unbound** | Private recursive DNS resolver |
| **Tailscale** | Secure remote access |
| **Router DNS override** | Routes all LAN traffic through Pi-hole |

---

## ⚙️ System Setup

### 🧾 Prerequisites
- Raspberry Pi 5 running **Ubuntu Server 22.04+**
- Static IP → `192.168.1.10`
- SSH access
- Router admin access

---

## 🚀 Installation Steps

### 1️⃣ Install Pi-hole
```bash
curl -sSL https://install.pi-hole.net | bash
```
During setup:

Interface → eth0 (add tailscale0 if using Tailscale)

Upstream DNS → any (we’ll replace later)

Blocklist → StevenBlack unified list

Privacy → as preferred

Verify:

```
pihole status
Access dashboard → http://192.168.1.10/admin
```

2️⃣ Configure Pi-hole Settings
```
sudo nano /etc/pihole/setupVars.conf
```

```
PIHOLE_INTERFACE=eth0 tailscale0
IPV4_ADDRESS=192.168.1.10/24
PIHOLE_DNS_1=127.0.0.1#5335
PIHOLE_DNS_2=1.1.1.1
INSTALL_WEB=true
QUERY_LOGGING=true
BLOCKING_ENABLED=true
```

3️⃣ Add Unbound (private recursive DNS)
```
sudo apt update
sudo apt install unbound -y
```

Create config:

```
sudo nano /etc/unbound/unbound.conf.d/pi-hole.conf
```
```
server:
    interface: 127.0.0.1
    port: 5335
    do-ip4: yes
    do-udp: yes
    do-tcp: yes
    harden-glue: yes
    harden-dnssec-stripped: yes
    use-caps-for-id: yes
    edns-buffer-size: 1232
    prefetch: yes
    qname-minimisation: yes
    cache-min-ttl: 3600
    cache-max-ttl: 86400
    root-hints: "/var/lib/unbound/root.hints"
```

Fetch root hints and restart:

```
sudo wget -O /var/lib/unbound/root.hints https://www.internic.net/domain/named.root
sudo systemctl restart unbound
```

Test resolution:

```
dig @127.0.0.1 -p 5335 google.com
```

4️⃣ Router DNS Configuration
In your router’s DNS settings:

Field	Value
IPv4 DNS Server 1	192.168.1.10 (Your static IP)
IPv4 DNS Server 2	1.1.1.1
IPv6 DNS	(leave empty)

💡 This ensures all LAN devices use Pi-hole → Unbound automatically.
 
5️⃣ Update Blocklists
```
pihole -g
```

✅ Final Result
Outcome	Description
🚫	Ads & trackers blocked network-wide
🔒	Local, private DNS via Unbound
🌍	Remote access via Tailscale
⚡	Faster browsing with cached queries

🧰 Tech Stack
Component	Version / Info
Raspberry Pi 5	Ubuntu Server 22.04
Pi-hole	v6.1.4
Unbound	v1.17+
Tailscale	Latest
Router	DNS override enabled

💡 Inspiration
Built to take control of my own internet — faster, quieter, and private.
No browser extensions, no per-device setup — just one Pi to rule them all. 🧠
