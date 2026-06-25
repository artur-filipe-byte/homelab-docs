# Pi-hole - O Meu Bloqueador de Anúncios Na Rede Toda

**Atualizado:** Junho 2026

---

![Topologia de Rede](imagens/topologia-rede.svg)
*O Pi-hole está no servidor Qosmio, dentro de Docker*

---

## Para que serve isto

O **Pi-hole** é um servidor DNS que bloqueia anúncios e rastreadores **ao nível da rede**. Isto significa que não preciso de instalar nada no telemóvel, no PC ou na TV da minha mãe — tudo o que passa pelo router passa primeiro pelo Pi-hole e os anúncios são filtrados antes de chegar aos dispositivos.

Basicamente:
- Abres um site qualquer
- Antes de carregar, o browser pergunta "qual é o IP deste site?"
- O Pi-hole responde, mas se for um domínio de anúncio conhecido, **não responde** — o anúncio simplesmente não aparece
- Resultado: menos tráfego, páginas mais rápidas, menos rastreadores

---

## O Setup

O Pi-hole corre dentro de **Docker** no servidor Qosmio, ao lado do Nextcloud e outros serviços.

### 1. Criar o container

```bash
docker run -d \
  --name pihole \
  -p 53:53/tcp -p 53:53/udp \
  -p 8053:80/tcp \
  -e TZ="Europe/Lisbon" \
  -e WEBPASSWORD="[PASSWORD]" \
  -v pihole_etc:/etc/pihole \
  -v pihole_dnsmasq:/etc/dnsmasq.d \
  --restart unless-stopped \
  pihole/pihole:latest
```

Explicação:
- **Porta 53** — DNS (TCP e UDP). É por aqui que o router pergunta ao Pi-hole.
- **Porta 8053** — Interface web do Pi-hole (admin). Abro em `http://[IP-SERVIDOR]:8053/admin`
- **WEBPASSWORD** — Password para entrar no painel de admin
- **Volumes** — Para os dados persistirem quando o container reinicia

### 2. Configurar o router para usar o Pi-hole como DNS

No **pfSense**, fui a:

```
Services > DHCP Server > LAN
```

E mudei o **DNS Server** para o IP do servidor onde o Pi-hole corre:

```
DNS Servers: [IP-SERVIDOR]
```

Isto faz com que todos os dispositivos na rede que pedem IP via DHCP recebam automaticamente o Pi-hole como DNS.

Para dispositivos com IP fixo (como o servidor), mudei manualmente o DNS nas definições de rede.

### 3. Testar

Para testar se o Pi-hole está a funcionar:

```bash
# No servidor
docker logs pihole | tail -5

# De qualquer dispositivo na rede
nslookup google.com
# Devemos ver o IP do Pi-hole como servidor DNS
```

Depois abri o painel em `http://[IP-SERVIDOR]:8053/admin` e verifiquei que as queries estavam a aparecer.

---

## O Que Acontece No Dia a Dia

O painel do Pi-hole mostra:

- **Queries totais** — quantos pedidos DNS passaram por ele
- **Bloqueadas** — quantas foram bloqueadas (% de bloqueio)
- **Top domínios** — quais os sites mais pedidos na rede
- **Top clientes** — quais os dispositivos que mais fazem pedidos

No meu caso, o Pi-hole bloqueia cerca de **20-30%** de todo o tráfego DNS. Isto são anúncios e rastreadores que nunca chegam a carregar nos dispositivos de casa.

---

## O Impacto Real

| Antes do Pi-hole | Depois do Pi-hole |
|------------------|-------------------|
| Anúncios no telemóvel da minha mãe | Mãe sem anúncios no telemóvel |
| Sites lentos cheios de rastreadores | Sites carregam visivelmente mais rápido |
| TVs e dispositivos a fazer pedidos para tracking desconhecido | Tráfego bloqueado na origem |

A minha mãe não sabe o que é um Pi-hole, nem precisa de saber. As fotos continuam a fazer sync para o Nextcloud e a internet funciona como antes, só que sem anúncios.

---

## Problemas Que Já Tive

**"Internet deixou de funcionar"** — Aconteceu quando o container do Pi-hole foi abaixo e o router ainda estava a apontar para ele. Solução: configurei o pfSense com um DNS secundário (8.8.8.8) para fallback.

**"Algum site não carrega bem"** — O Pi-hole às vezes bloqueia domínios legítimos. No painel admin, vou a "Query Log", encontro o domínio e faço "Whitelist". Rápido de resolver.

---

## O Que Aprendi

- **DNS parece simples até deixar de funcionar** — perceber como o DNS funciona na rede foi uma das coisas mais úteis que aprendi
- **Um container só para DNS** — o Pi-hole é mínino em recursos (cpu/ram quase zero), mas o impacto no dia a dia é enorme
- **Whitelist é importante** — bloquear demais pode partir sites. O segredo é monitorizar e ajustar

---

## Links Úteis

- Painel do Pi-hole: `http://[IP-SERVIDOR]:8053/admin`
- Lista de bloqueio padrão: incluída no Pi-hole
- Listas extra que adicionei: OISD, NoTracking

---

*Projeto pessoal. Aprendi sobre DNS, DHCP, Docker e porque é que até um container pequeno pode fazer uma grande diferença.*
