# Jellyfin + DuckDNS - My DIY Netflix

**Updated:** June 2026

---

![Network Topology](../pt/imagens/topologia-rede.svg)
*Homelab infrastructure diagram (Jellyfin runs on the Windows PC)*

---

## What this project demonstrates

| Skill | Application |
|-------|-------------|
| **Networking** | Port forwarding, NAT, firewall rules, dynamic DNS (DuckDNS) |
| **Windows** | Service configuration, Task Scheduler, Windows Firewall |
| **DNS** | Dynamic public IP updates with DuckDNS |
| **Security** | Controlled service exposure, firewall access limitation |
| **Troubleshooting** | External connectivity diagnostics, DNS and firewall issues |
| **Documentation** | Complete guide to replicate a remote media server setup |

---

## What it does

I have **Jellyfin** installed on my Windows PC to serve movies and TV shows at home and for my friends to watch when they are away. **DuckDNS** gives a fixed name to my home IP, which changes from time to time.

Basically:
- Open `[YOUR-DOMAIN].duckdns.org` in a browser or the Jellyfin app
- The login screen appears
- Enter the credentials I gave you
- Watch the movies in my library

And no, I do not pay for Netflix.

---

## Setup: Jellyfin on Windows

### Installation

I installed Jellyfin from the official site (`jellyfin.org`) - Windows version. It is a standard installer, next-next-finish. Jellyfin runs as a Windows service, which means it starts automatically when I turn on the PC.

It listens on port **8096**.

### Firewall

When you install Jellyfin, it asks if you want to create a firewall rule. I said yes, but only for the **Private** network (home network). This is because I do not want Jellyfin exposed to the internet without DuckDNS acting as intermediary.

### Libraries

I added the folders where I keep movies and TV shows. Jellyfin fetches covers, descriptions and metadata automatically. I set the language to Portuguese.

---

## Setup: DuckDNS (For My Friends to Access Remotely)

DuckDNS is great because it is free and does exactly what I need: a domain that always points to my home, even when the IP changes.

### 1. Register on DuckDNS

Go to `duckdns.org` and log in with your Google/GitHub account. Pick a subdomain name (e.g. `jellyfin-yourname`) and you get `jellyfin-yourname.duckdns.org`.

### 2. Auto-update the IP

DuckDNS needs to know your current IP whenever it changes. Since my PC is not always on, I have a script that runs every few minutes to update DuckDNS.

**What I did (if you want to do the same):**

I created a PowerShell script on Windows:

```powershell
$token = "your-duckdns-token"
$domain = "your-domain"
$url = "https://www.duckdns.org/update?domains=$domain&token=$token&ip="
Invoke-WebRequest -Uri $url -UseBasicParsing | Out-Null
```

Then in Windows, I opened **Task Scheduler** and created a task to run this script every **5 minutes**. DuckDNS can handle it.

### 3. Port Forwarding on the Router

I logged into my router and forwarded port **8096** (Jellyfin) to my PC's IP.

On the router:
- **NAT / Port Forwarding**
- Create rule: External port 8096 -> PC IP:8096
- Protocol TCP

I also created a firewall rule to allow that traffic (if the router did not do it automatically).

### 4. Testing

To test, I turned off WiFi on my phone (so I was not on the home network) and opened `http://[YOUR-DOMAIN].duckdns.org:8096` in the browser. The Jellyfin login screen appeared. It worked.

---

## How My Friends Access It

I just tell them:
1. Install the **Jellyfin** app (or use a browser)
2. Server: `http://[YOUR-DOMAIN].duckdns.org:8096`
3. Username and password I created for them

It works on phones, tablets, TVs (Android TV), etc.

---

## Lessons Learned

1. **Home IPs change.** That is why DuckDNS is necessary. Without it, I would be giving my friends a new IP address all the time.
2. **Firewall must be properly configured.** Opening ports on the router without care is dangerous. I only opened port 8096 and only for Jellyfin.
3. **The PC must stay on.** If I turn off the PC, Jellyfin goes down. Possible solution: move Jellyfin to a Linux server running 24/7.
4. **DuckDNS is truly free.** I paid nothing and it has been working for months without issues.

---

## Problems I Have Had

**"Cannot access from outside"**
- Check if the PC is turned on
- Check if DuckDNS has updated (go to duckdns.org and see the IP)
- Check if the port forward is active on the router

**"It is very slow"**
My home internet upload speed is the bottleneck. For watching at home without issues, I use the local IP instead.

---

## What I Would Change / Still Want To Do

- [ ] Move Jellyfin to a Linux server so I do not need Windows on 24/7
- [ ] Set up HTTPS (with Let's Encrypt)
- [ ] Limit bandwidth friends can use so it does not kill my home internet

---

*Personal project. Learned about NAT, dynamic DNS and why running a home server is more work than it seems.*
