# Homelab Docs — Portfolio de Infraestrutura IT

[![PT](https://img.shields.io/badge/lang-PT--PT-green)](./pt/)
[![EN](https://img.shields.io/badge/lang-EN-blue)](./en/)
[![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)](#)
[![pfSense](https://img.shields.io/badge/pfSense-212121?logo=pfsense&logoColor=white)](#)
[![Linux](https://img.shields.io/badge/Linux-FCC624?logo=linux&logoColor=black)](#)
[![Nextcloud](https://img.shields.io/badge/Nextcloud-0082C9?logo=nextcloud&logoColor=white)](#)

---

## Quem sou

**Artur Filipe** — Profissional em transicao para IT e ciberseguranca, certificado CompTIA A+ e Security+. 13 anos de experiencia em atendimento ao cliente, gestao de incidentes e resolucao de problemas em tempo real.

Este repositorio documenta projetos praticos que construi no meu homelab para demonstrar competencias em **redes, Docker, Linux, Windows, DNS, seguranca e automacao**.

Disponivel para: **IT Support, SOC Analyst Junior, Cybersecurity Junior, Sysadmin Junior**

| | Links |
|---|---|
| :globe_with_meridians: Website | [arturfilipe.work](https://arturfilipe.work) |
| :incoming_envelope: Email | artur.a.filipe@protonmail.com |
| :octocat: GitHub | [github.com/artur-filipe-byte](https://github.com/artur-filipe-byte) |

---

## Projetos em Destaque

| Projeto | Stack | Competencias | Docs |
|---------|-------|-------------|------|
| **Nextcloud + Samba** (NAS) | Docker, Linux, Nextcloud, Samba, Cloudflare | Docker, Linux admin, permissões, backup, documentacao | [PT](pt/servidor-caseiro-nextcloud-samba.md) · [EN](en/homelab-server-nextcloud-samba.md) |
| **Jellyfin + DuckDNS** (Media Server) | Jellyfin, DuckDNS, Port Forwarding, Windows, pfSense | DNS, NAT, port forwarding, remote access, troubleshooting | [PT](pt/jellyfin-duckdns-windows.md) · [EN](en/jellyfin-duckdns-windows.md) |
| **Pi-hole** (Ad Blocker) | Pi-hole, Docker, pfSense, DHCP, DNS | DNS, DHCP, Docker, administracao de rede, seguranca | [PT](pt/pi-hole-docker.md) · [EN](en/pi-hole-docker.md) |

---

## Topologia de Rede

![Diagrama de Rede](pt/imagens/topologia-rede.svg)

*Infraestrutura completa do homelab: WAN → pfSense → Suricata IDS/IPS → Rede Interna (Qosmio Server + Windows PC)*

---

## O que este repositorio demonstra

| Competencia | Onde se ve |
|------------|-----------|
| **Redes** | pfSense, Suricata, DNS, DHCP, firewalls, VLANs |
| **Docker** | Nextcloud, Pi-hole, Portainer, Elastic Stack em containers |
| **Linux** | Ubuntu/Debian, Bash, SSH, Docker Compose, permissoes |
| **Windows** | Jellyfin, Task Scheduler, servicos Windows |
| **DNS** | Pi-hole, DuckDNS, nslookup, configuracao DHCP |
| **Seguranca** | Firewall rules, Suricata IDS/IPS, DMARC/SPF, Cloudflare |
| **Documentacao** | Guias PT/EN bilíngues, estruturados e reproduziveis |
| **Resolucao de Problemas** | Secoes de troubleshooting em cada projeto |

---

## Estrutura do Repositorio

```text
homelab-docs/
├── README.md              ← Este ficheiro (landing page do portfolio)
├── pt/                    ← Documentacao em Portugues (PT-PT)
│   ├── imagens/           ← Diagramas e screenshots
│   ├── servidor-caseiro-nextcloud-samba.md
│   ├── jellyfin-duckdns-windows.md
│   └── pi-hole-docker.md
└── en/                    ← Documentation in English
    ├── homelab-server-nextcloud-samba.md
    ├── jellyfin-duckdns-windows.md
    └── pi-hole-docker.md
```

---

## Sobre

> Publiquei esta documentacao como parte do meu portfolio de transicao de carreira da hotelaria para IT e ciberseguranca. Cada projeto foi feito, desfeito, debugado e documentado — porque sei que e esse o processo real do trabalho em IT.

---

**Artur Filipe** — [arturfilipe.work](https://arturfilipe.work) · [GitHub](https://github.com/artur-filipe-byte)
