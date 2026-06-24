# Homelab Server - Nextcloud + Samba : My DIY NAS

**Updated:** June 2026

---

## What it does

I have an old laptop running as a home server. It runs **Nextcloud** inside Docker and **Samba** for network file sharing. Basically it is my personal Google Drive, no subscription fees and all my data stays at home.

My mother uses it too. She has Nextcloud on her phone and photos sync automatically when she is home. She has no idea what a server is. She just knows "the photos show up on the computer".

---

## Hardware

This matters because I do not need fancy hardware. I use what I already had:

| Component | What it is |
|------------|---------|
| **Machine** | Old laptop (2012, still works) |
| **RAM** | 8 GB. Plenty for what it does |
| **Disk** | 687 GB SSD |
| **Network** | Static IP configured on the router |

The best part? The laptop was collecting dust. I gave it a second life.

---

## What is running

- **Nextcloud** - browser access at `http://[SERVER-IP]:8080`
- **Samba** - mapped as a network drive in Windows Explorer
- **Ollama** - local LLMs for testing (not part of the main project, just there)

If I change something in Nextcloud through the web (a folder, a file), it shows up on Windows through the mapped drive right away. And vice versa. This was the hardest part to get right and I will explain below.

---

## Setup (how I did it)

### 1. Base system

Installed Ubuntu 22.04 and then Docker:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install docker.io docker-compose-v2 -y
sudo usermod -aG docker $USER
```

### 2. File folders

I created folders that make sense for my daily use. Old-school organization by content type:

```bash
mkdir -p /home/$USER/{Documents,Pictures,Music,"photos archive",shared}
```

### 3. Nextcloud with Docker Compose

I created a `docker-compose.yml`. It has three containers:

- **MariaDB** - the database where Nextcloud stores everything (users, metadata, etc.)
- **Nextcloud app** - the service itself, on port 8080
- **Redis** - cache so Nextcloud does not get slow

The trick here is that I mounted my local folders inside the Nextcloud container using `bind mounts`. This way Nextcloud sees the same folders I see on Windows:

```yaml
volumes:
  - /home/$USER/Pictures:/data/Pictures
  - /home/$USER/Documents:/data/Documents
  - /home/$USER/Music:/data/Music
  - "/home/$USER/photos archive:/data/photos archive"
```

To start:

```bash
cd /path/to/nextcloud
docker compose up -d
```

### 4. Samba (the annoying permissions part)

In Samba, I created two shares:

- `[homes]` - access to my personal folder (authenticated with my user)
- `[shared]` - an extra folder for things I want to share without the cloud

The problem is that Samba uses my system user and Nextcloud uses the `www-data` user. If one creates a file, the other cannot write to it. This drove me crazy for an afternoon until I figured out the solution:

```bash
# Add both users to each other's groups
sudo usermod -aG www-data $USER
sudo usermod -aG $USER www-data   # yes, this exists

# Fix ownership and permissions
sudo chown -R $USER:www-data /home/$USER/{Documents,Pictures,Music,"photos archive",shared}
sudo find /home/$USER/{Documents,Pictures,Music,"photos archive",shared} -type d -exec chmod 775 {} \;
sudo find /home/$USER/{Documents,Pictures,Music,"photos archive",shared} -type f -exec chmod 664 {} \;

# SGID makes any new file inherit the www-data group automatically
sudo chmod g+s /home/$USER/{Documents,Pictures,Music,"photos archive",shared}
```

### 5. Mapping the drive in Windows

In Windows Explorer, I right-clicked "This PC" -> "Map network drive". Picked a drive letter and pointed it to `\\[SERVER-IP]\$USER`. Checked "Reconnect at sign-in" and entered the password.

Now when I open that drive I have all my folders. Documents, Pictures, Music, everything as if they were local folders.

---

## Lessons learned from this project

1. **Shared permissions between Samba and Docker.** Nextcloud (www-data) and Samba (my user) are different Linux users. They must belong to the same group and folders must have the SGID bit set to work properly.
2. **Static IP is mandatory.** I spent a day figuring out why the mapped drive kept asking for a password. The server IP had changed because of DHCP.
3. **An old laptop works perfectly fine.** Do not buy an expensive NAS to start. Use what you already have.
4. **Docker makes things easy.** When Nextcloud broke after an update, it was just `docker compose pull && docker compose up -d` and it was fixed.

---

## Maintenance (what I do occasionally)

```bash
# Update Nextcloud
cd /path/to/nextcloud
docker compose pull && docker compose up -d
docker image prune -a

# Check everything is running
docker ps --format 'table {{.Names}}\t{{.Status}}'
```

I do not have automated backups configured yet. It is on my to-do list.

---

## Troubleshooting (problems I have had)

**"The mapped drive asks for a password and does not accept it"**
The server IP changed. Check with `ip a` on the server and remap the drive.

**"Nextcloud says Internal Server Error"**
Run `docker compose logs app | tail -30` to see the error. Usually fixed with `docker exec -it nextcloud-app-1 php occ upgrade`.

---

## To-Do

- [ ] Automated backup to external disk
- [ ] Reverse proxy for HTTPS
- [ ] Monitoring (Uptime Kuma)
- [ ] Expand storage when the disk fills up

---

*Personal project. Learned a lot about Docker, Linux permissions and Samba doing this.*
