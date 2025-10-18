# -Pi-hole-on-Pi5

🧱 Pi-hole-on-Pi5

From ads everywhere → to a clean, private web.

Turn your Raspberry Pi 5 into a network-wide ad blocker and DNS privacy hub using Pi-hole + Unbound + Tailscale.
Everything — from YouTube to mobile apps — gets filtered and accelerated right at the network layer.

🧠 What This Does

This setup makes your Raspberry Pi 5 the brain of your home network.
It intercepts all DNS requests and decides which ones to allow or block — stopping ads, telemetry, and trackers before they even load.

🧩 Components:

Pi-hole — DNS-level ad blocker

Unbound — your own recursive DNS resolver

Tailscale — secure remote access

Router DNS override — routes all home traffic through Pi-hole

🧾 Table of Contents

System Setup

Install Pi-hole

Configure Pi-hole

Add Unbound

Router DNS Config

Tailscale (Optional)

Update Blocklists

Final Result

Add-ons

Tech Stack

⚙️ System Setup

Prerequisites

Raspberry Pi 5 running Ubuntu Server 22.04+

Static IP → 192.168.1.10

SSH access

Router admin access

1️⃣ Install Pi-hole
curl -sSL https://install.pi-hole.net | bash


During setup:

Interface → eth0 (or add tailscale0 if using Tailscale)

Upstream DNS → any (we’ll replace later)

Blocklist → StevenBlack unified list

Privacy → choose preferred level

Verify:

pihole status


Dashboard → http://192.168.1.10/admin

2️⃣ Configure Pi-hole settings
sudo nano /etc/pihole/setupVars.conf


Paste:

PIHOLE_INTERFACE=eth0 tailscale0
IPV4_ADDRESS=192.168.1.10/24
PIHOLE_DNS_1=127.0.0.1#5335
PIHOLE_DNS_2=1.1.1.1
INSTALL_WEB=true
QUERY_LOGGING=true
BLOCKING_ENABLED=true

3️⃣ Add Unbound (private recursive DNS)
sudo apt update
sudo apt install unbound -y


Config:

sudo nano /etc/unbound/unbound.conf.d/pi-hole.conf

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


Fetch root hints:

sudo wget -O /var/lib/unbound/root.hints https://www.internic.net/domain/named.root
sudo systemctl restart unbound


Test:

dig @127.0.0.1 -p 5335 google.com

4️⃣ Router DNS configuration

In your router’s DNS settings:

Field	Value
IPv4 DNS Server 1	192.168.1.10
IPv4 DNS Server 2	1.1.1.1
IPv6 DNS	(leave empty)

💡 This ensures all LAN devices use Pi-hole → Unbound automatically.

5️⃣ Add Tailscale (optional but 🔥)
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --ssh


Now you can access Pi-hole remotely:

http://100.x.x.x/admin

6️⃣ Update blocklists
pihole -g

📊 Optional Add-ons

🔍 Prometheus + Grafana → for real-time DNS analytics

🔁 Cron job → automatic blocklist updates

🧠 DoH/DoT on Unbound → encrypted upstream DNS

✅ Final Result

🚫 Ads + trackers blocked network-wide

🔒 DNS resolved locally via Unbound

🌍 Remote control with Tailscale

⚡ Faster browsing with caching

🧰 Tech Stack
Component	Version / Info
Raspberry Pi 5	Ubuntu Server 22.04
Pi-hole	v6.1.4
Unbound	v1.17+
Tailscale	Latest
Router	DNS override enabled
📸 Screenshots

(Add yours here)

Pi-hole Admin Dashboard

Query Analytics

Router DNS Settings

💡 Inspiration

Built to take control of my own internet — faster, quieter, and private.
No browser extensions, no per-device setup — just one Pi to rule them all. 🧠
