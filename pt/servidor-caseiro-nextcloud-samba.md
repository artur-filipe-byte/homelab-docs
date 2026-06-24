# Servidor Caseiro - Nextcloud + Samba : O Meu NAS

**Atualizado:** Junho 2026

---

## Para que serve isto

Tenho um portatil velho a servir de servidor ca em casa. Tem o **Nextcloud** a correr dentro de Docker e o **Samba** para partilhar ficheiros na rede. Basicamente e o meu Google Drive pessoal, sem pagar subscricao e com os dados todos em casa.

A minha mae tambem usa. Tem o Nextcloud no telemovel e as fotos fazem sync automatico quando esta em casa. Ela nem sabe o que e um servidor. So sabe que as fotos aparecem no computador.

---

## O Hardware

Isto e importante porque nao preciso de um servidor de jeito. Uso o que ja tinha:

| Componente | O que e |
|------------|---------|
| **Maquina** | Portatil velho (2012, ainda funciona) |
| **RAM** | 8 GB. Chega perfeitamente para o que faz |
| **Disco** | SSD 687 GB |
| **Rede** | IP fixo configurado no router |

O melhor disto tudo? O portatil estava encostado a ganhar po. Dei-lhe uma segunda vida.

---

## O que esta a correr

- **Nextcloud** - acesso por browser em `http://[IP-SERVIDOR]:8080`
- **Samba** - para mapear uma drive no Windows Explorer
- **Ollama** - tenho la LLMs locais para testar (nao faz parte do projeto principal, mas esta la)

Quando mudo alguma coisa no Nextcloud pela web (uma pasta, um ficheiro), aparece logo no Windows atraves da drive mapeada. E vice-versa. Isto foi o mais dificil de conseguir e vou explicar a seguir.

---

## O Setup (como fiz)

### 1. Sistema base

Instalei Ubuntu 22.04 e depois o Docker:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install docker.io docker-compose-v2 -y
sudo usermod -aG docker $USER
```

### 2. Pastas para os ficheiros

Criei as pastas que fazem sentido para o meu dia-a-dia. Organizacao a antiga, por tipo de conteudo:

```bash
mkdir -p /home/$USER/{Documentos,Imagens,Música,"fotos arquivos",shared}
```

### 3. Nextcloud com Docker Compose

Fiz um ficheiro `docker-compose.yml`. Tem tres contentores:

- **MariaDB** - a base de dados onde o Nextcloud guarda tudo (utilizadores, metadata, etc.)
- **Nextcloud app** - o servico em si, na porta 8080
- **Redis** - cache para o Nextcloud nao ficar lento

O truque aqui e que montei as pastas dentro do contentor do Nextcloud com `bind mounts`. Assim o Nextcloud ve as mesmas pastas que eu vejo no Windows:

```yaml
volumes:
  - /home/$USER/Imagens:/data/Imagens
  - /home/$USER/Documentos:/data/Documentos
  - /home/$USER/Música:/data/Música
  - "/home/$USER/fotos arquivos:/data/fotos arquivos"
```

Para ligar:

```bash
cd /caminho/para/nextcloud
docker compose up -d
```

### 4. Samba (a parte chata das permissoes)

No Samba, criei duas shares:

- `[homes]` - da acesso a minha pasta pessoal (autenticacao com o meu user)
- `[shared]` - uma pasta extra para coisas que quero partilhar sem ser pela cloud

O problema e que o Samba usa o meu user e o Nextcloud usa o user `www-data`. Se um cria um ficheiro, o outro nao consegue escrever nele. Isto deu-me cabo da cabeca durante uma tarde ate perceber a solucao:

```bash
# Meter os dois utilizadores nos grupos um do outro
sudo usermod -aG www-data $USER
sudo usermod -aG $USER www-data   # sim, isto existe

# Corrigir ownership e permissoes
sudo chown -R $USER:www-data /home/$USER/{Documentos,Imagens,Música,"fotos arquivos",shared}
sudo find /home/$USER/{Documentos,Imagens,Música,"fotos arquivos",shared} -type d -exec chmod 775 {} \;
sudo find /home/$USER/{Documentos,Imagens,Música,"fotos arquivos",shared} -type f -exec chmod 664 {} \;

# SGID faz com que qualquer ficheiro novo herde o grupo www-data
sudo chmod g+s /home/$USER/{Documentos,Imagens,Música,"fotos arquivos",shared}
```

### 5. Mapear a drive no Windows

No Explorador do Windows, cliquei direito em "This PC" -> "Map network drive". Escolhi uma letra e apontei para `\\[IP-SERVIDOR]\$USER`. Meti "Reconnect at sign-in" e inseri a password.

Agora quando abro essa drive tenho la as pastas todas. Documentos, Imagens, Música, tudo como se fossem pastas locais.

---

## Licoes que aprendi com este projeto

1. **Permissoes partilhadas entre Samba e Docker.** O Nextcloud (www-data) e o Samba (o meu user) sao users diferentes no Linux. Tem de pertencer ao mesmo grupo e as pastas tem de ter o SGID bit para funcionar.
2. **IP fixo e obrigatorio.** Passei um dia a perceber porque e que a drive aparecia a pedir password. O IP do servidor tinha mudado com o DHCP.
3. **Um portatil velho serve perfeitamente.** Nao compres um NAS caro para comecar. Usa o que tens.
4. **Docker facilita muito.** Quando o Nextcloud avariou depois de uma atualizacao, foi so `docker compose pull && docker compose up -d` e ficou resolvido.

---

## Manutencao (o que faco de vez em quando)

```bash
# Atualizar o Nextcloud
cd /caminho/para/nextcloud
docker compose pull && docker compose up -d
docker image prune -a

# Ver se esta tudo a funcionar
docker ps --format 'table {{.Names}}\t{{.Status}}'
```

Ainda nao tenho backups automaticos configurados. Esta na lista de coisas para fazer.

---

## Troubleshooting (problemas que ja tive)

**"A drive pede password e nao aceita"**
O IP do servidor mudou. Verificar com `ip a` no servidor e re-mapear a drive.

**"Nextcloud diz Internal Server Error"**
Correr `docker compose logs app | tail -30` para ver o erro. Normalmente resolve com `docker exec -it nextcloud-app-1 php occ upgrade`.

---

## O que falta fazer (To-Do)

- [ ] Backup automatico para disco externo
- [ ] Proxy reverso para ter HTTPS
- [ ] Monitorizacao (Uptime Kuma)
- [ ] Expandir storage quando o disco encher

---

*Projeto pessoal. Aprendi imenso sobre Docker, permissoes Linux e Samba a fazer isto.*
