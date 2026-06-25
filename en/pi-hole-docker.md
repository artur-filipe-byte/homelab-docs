# Pi-hole - My Network-Wide Ad Blocker

**Updated:** June 2026

---

![Network Topology](../pt/imagens/topologia-rede.svg)
*Pi-hole runs on the Qosmio server inside Docker*

---

## What it does

**Pi-hole** is a DNS server that blocks ads and trackers **at the network level**. This means I don't need to install anything on my phone, PC, or my mother's TV — everything that goes through the router goes through Pi-hole first, and ads are filtered before reaching any device.

In simple terms:
- You open a website
- Before loading, the browser asks "what is the IP of this site?"
- Pi-hole answers — but if it's an ad domain, **it doesn't answer** — the ad simply doesn't appear
- Result: less traffic, faster pages, fewer trackers

---

## The Setup

Pi-hole runs inside **Docker** on the Qosmio server, alongside Nextcloud and other services.

### 1. Create the container

```bash
docker run -d \
  --name pihole \
  -p 53:53/tcp -p 53:53/udp \
  -p 8053:80/tcp \
  -e TZ="Europe/Lisbon" \
  -e WEBPASSWORD=*** \
  -v pihole_etc:/etc/pihole \
  -v pihole_dnsmasq:/etc/dnsmasq.d \
  --restart unless-stopped \
  pihole/pihole:latest
```

- **Port 53** — DNS (TCP and UDP). This is how the router queries Pi-hole.
- **Port 8053** — Web admin interface. Access at `http://[IP-SERVIDOR]:8053/admin`
- **WEBPASSWORD** — Admin panel password
- **Volumes** — Persist data across container restarts

### 2. Configure the router to use Pi-hole as DNS

In **pfSense**, I went to:

```
Services > DHCP Server > LAN
```

And changed the **DNS Server** to the server IP where Pi-hole runs:

```
DNS Servers: [IP-SERVIDOR]
```

This makes every device that gets an IP via DHCP automatically use Pi-hole as their DNS server.

### 3. Test

To verify Pi-hole is working:

```bash
# On the server
docker logs pihole | tail -5

# From any device on the network
nslookup google.com
# Should show Pi-hole's IP as the DNS server
```

Then open the admin panel at `http://[IP-SERVIDOR]:8053/admin` and check that queries are coming in.

---

## Daily Impact

The Pi-hole dashboard shows:
- **Total queries** — how many DNS requests passed through
- **Blocked queries** — percentage of blocked traffic
- **Top domains** — most requested sites on the network
- **Top clients** — which devices make the most requests

In my setup, Pi-hole blocks about **20-30%** of all DNS traffic. These are ads and trackers that never reach the devices at home.

---

## What I Learned

- **DNS seems simple until it stops working** — understanding how DNS works on the network was one of the most useful things I learned
- **A single container for DNS** — Pi-hole uses almost zero resources (CPU/RAM), but the daily impact is huge
- **Whitelisting matters** — over-blocking can break websites. The key is monitoring and adjusting

---

*Personal project. Learned about DNS, DHCP, Docker, and why even a small container can make a big difference.*
