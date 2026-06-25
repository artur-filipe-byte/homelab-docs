# Jellyfin + DuckDNS - O Meu Netflix Caseiro

**Atualizado:** Junho 2026

---

![Topologia de Rede](imagens/topologia-rede.svg)
*Diagrama da infraestrutura do homelab (o Jellyfin está no PC Windows)*

---

## O que este projeto demonstra

| Competência | Como se aplica |
|------------|----------------|
| **Redes** | Port forwarding, NAT, firewall rules, DNS dinâmico (DuckDNS) |
| **Windows** | Configuração de serviço, Task Scheduler, firewall do Windows |
| **DNS** | Atualização dinâmica de IP público com DuckDNS |
| **Segurança** | Exposição controlada de serviços, limitação de acesso por firewall |
| **Troubleshooting** | Diagnóstico de conectividade externa, problemas de DNS e firewall |
| **Documentação** | Guia completo para replicar o setup de media server remoto |

---

## Para que serve isto

Tenho o **Jellyfin** instalado no meu PC para servir filmes e series em casa e para acesso remoto quando estou fora. O **DuckDNS** da um nome fixo ao meu IP caseiro, que muda de vez em quando.

Basicamente:
- Abres `[TEU-DOMINIO].duckdns.org` no browser ou na app do Jellyfin
- Aparece o login
- Metes as credenciais que te dei
- Ves os filmes que tenho na biblioteca

E nao, nao pago Netflix.

---

## O Setup: Jellyfin no Windows

### Instalacao

Instalei o Jellyfin pelo site oficial (`jellyfin.org`) - versao Windows. E um instalador normal, next-next-finish. O Jellyfin corre como servico do Windows, o que significa que arranca sozinho quando ligo o PC.

Fica a ouvir na porta **8096**.

### Firewall

Quando instalas o Jellyfin, ele pergunta se queres criar uma regra de firewall. Eu disse que sim, mas apenas para a rede **Privada** (a de casa). Isto porque nao quero o Jellyfin exposto a internet sem o DuckDNS a fazer de intermediario.

### Bibliotecas

Adicionei as pastas onde tenho os filmes e series. O Jellyfin trata de ir buscar capas, descricoes e metadata automaticamente. Meti o idioma em Portugues (PT).

---

## O Setup: DuckDNS (Para Acesso Remoto)

O DuckDNS e fixe porque e gratis e faz exatamente o que preciso: um dominio que aponta sempre para minha casa, mesmo quando o IP muda.

### 1. Registar no DuckDNS

Vai a `duckdns.org` e faz login com a tua conta Google/GitHub. Escolhes um nome de subdominio (ex: `jellyfin-artur`) e ficas com `jellyfin-artur.duckdns.org`.

### 2. Fazer o IP atualizar automaticamente

O DuckDNS precisa de saber qual e o teu IP de cada vez que ele muda. Como o meu PC nao esta sempre ligado, tenho um script que corre de X em X minutos para atualizar o DuckDNS.

**O que fiz (se quiseres fazer igual):**

Criei um script PowerShell no Windows:

```powershell
$token = "o-teu-token-do-duckdns"
$domain = "o-teu-dominio"
$url = "https://www.duckdns.org/update?domains=$domain&token=$token&ip="
Invoke-WebRequest -Uri $url -UseBasicParsing | Out-Null
```

Depois no Windows, abri o **Task Scheduler** (Agendador de Tarefas) e criei uma tarefa para correr este script a cada **5 minutos**. O DuckDNS aguenta.

### 3. Port Forwarding no Router

Aqui entrei no router e fiz forward da porta **8096** (do Jellyfin) para o IP do meu PC.

No router:
- **NAT / Port Forwarding**
- Criar regra: Porta externa 8096 -> IP do PC:8096
- Protocolo TCP

E tambem criei uma regra de firewall a permitir esse trafego (se o router nao tiver feito automaticamente).

### 4. Testar

Para testar, desliguei-me do WiFi do telemovel (para nao estar na rede de casa) e abri `http://[TEU-DOMINIO].duckdns.org:8096` no browser. Apareceu o ecra de login do Jellyfin. Funcionou.

---

## Como Aceder Remotamente

So tenho de saber:
1. Instalar a app **Jellyfin** (ou usar o browser)
2. Servidor: `http://[TEU-DOMINIO].duckdns.org:8096`
3. User e password que criei

Funciona no telemovel, tablet, TV (Android TV), etc.

---

## Licoes Que Aprendi

1. **O IP de casa muda.** Da a necessidade do DuckDNS. Sem ele, estava sempre a dar um IP novo.
2. **Firewall tem de estar afinada.** Abrir portas no router sem cuidado e perigoso. So abri a 8096 e apenas para o Jellyfin.
3. **O PC tem de estar ligado.** Se desligo o PC, o Jellyfin vai abaixo. Solucao possivel: meter o Jellyfin num servidor Linux ligado 24/7.
4. **DuckDNS e mesmo gratis.** Nao paguei nada e funciona ha meses sem falhar.

---

## Problemas Que Ja Tive

**"Nao consigo aceder de fora"**
- Verificar se o PC esta ligado
- Verificar se o DuckDNS atualizou (ir a duckdns.org e ver o IP)
- Verificar se o port forward esta ativo no router

**"Esta muito lento"**
A culpa e da net de casa (upload limitado). Para ver em casa sem problemas, uso o IP local.

---

## O Que Mudava / Ainda Quero Fazer

- [ ] Passar o Jellyfin para o servidor Linux para nao precisar do Windows ligado 24/7
- [ ] Meter HTTPS (com Let's Encrypt)
- [ ] Limitar a banda para acesso remoto nao lixar a net de casa

---

*Projeto pessoal. Aprendi sobre NAT, DNS dinamico e porque e que ter um servidor em casa da mais trabalho do que parece.*
