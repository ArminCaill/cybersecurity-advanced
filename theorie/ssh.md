# Les 3: SSH

## Essentiële elementen

Inloggen met SSH-sleutels is veiliger en makkelijker: je bewijst bezit van je private key zonder een wachtwoord te versturen, waardoor brute force en phishing grotendeels wegvallen.

De authorized_keys is het bestand op de server waarin per Linux-gebruiker staat welke publieke SSH-sleutels toegang mogen geven tot die account. Standaard staat het in ~/.ssh/authorized_keys.

Het known_hosts-bestand staat op de client en bevat de host keys (publieke serversleutels) van SSH-servers waarmee je eerder hebt verbonden. Bij elke nieuwe verbinding vergelijkt je SSH-client de server die zich meldt met de sleutel in known_hosts.

### The difference between a passphrase on a key vs user/password

Een passphrase op een sleutel beschermt je private key op schijf (at rest). Je gebruikt de passphrase lokaal om de sleutel te ontgrendelen; die passphrase gaat nooit naar de server. Na ontgrendeling (vaak via een ssh-agent) log je in door een digitale handtekening te zetten met je sleutelpaar.

Een gebruikersnaam/wachtwoord is het inloggeheim zelf dat de server bij elke login controleert. Ondanks versleutelde SSH-transport kan het wachtwoord worden geraden, hergebruikt, gelekt of gephished.

Kortom: sleutel + passphrase = je geheim verlaat je apparaat niet en is doorgaans sterker en beter beheersbaar (intrekken = publieke sleutel verwijderen); user/password stuurt telkens een herbruikbaar geheim en heeft meer aanvalsoppervlak.

## Jump/Bastion host

Een jump- of bastionhost is een geharde, van buiten bereikbare machine die dient als enige gecontroleerde toegangspoort tot een privénetwerk. Beheerders maken eerst verbinding met de bastion (SSH/RDP) en “springen” vandaar naar interne servers die niet direct aan het internet hangen.

Waarom gebruikt een bedrijf dit?

- Kleiner aanvalsoppervlak: alleen de bastion heeft een open inkomende poort; interne hosts accepteren beheerverkeer alleen vanaf de bastion.
- Sterke toegangscontrole: centraal MFA, IP-allowlists, kortdurende SSH-certificaten en just-in-time toegang afdwingen.
- Audit & compliance: sessielogging/-opname en command auditing op één plek.
- Segmentatie: duidelijke scheiding tussen internet/DMZ en private subnets; firewallregels blijven overzichtelijk.

## Local and remote port forwarding using SSH

- Local port forwarding (-L): je luistert lokaal en stuurt verkeer via de SSH-tunnel naar een doel achter de server.
  - Gebruik: iets op een intern netwerk veilig bereiken vanaf je laptop.
  - Voorbeeld: ssh -N -L localhost:1234:localhost:80 vagrant@172.30.0.10
- Remote port forwarding (-R): de server luistert (op afstand) en stuurt verkeer via de SSH-tunnel terug naar jouw client (of iets achter jouw client).
  - Gebruik: jouw lokale service beschikbaar maken voor iemand aan de server-kant.
  - Voorbeeld: ssh -N -R localhost:1234:localhost:80 vagrant@192.168.62.1

## SOCKS protocol

SOCKS is een generieke proxy-tunnel op sessielaag (L5). Je client praat eerst met een SOCKS-proxy (v4/v5), onderhandelt optioneel authenticatie, en vraagt de proxy om namens jou een verbinding te maken naar een doelhost/poort (CONNECT). Belangrijk: SOCKS begrijpt het applicatieprotocol niet (anders dan een HTTP-proxy); het vervoert gewoon TCP/UDP alsof het door jou zelf was opgezet.

Use cases:

- Bereiken van interne diensten via één ingang: maak lokaal een SOCKS5-proxy over SSH en browse interne web-UIs zonder ze publiek te maken.

## Lab

Plaats je publieke sleutel op de bastion én op de interne hosts (kan rechtstreeks via de jump):

```bash
# public key kopiëren naar bastion op linux
ssh-copy-id vagrant@192.168.62.253

# public key kopiëren naar bastion op windows
Get-Content $env:USERPROFILE\.ssh\id_rsa.pub | ssh vagrant@192.168.62.253 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"

# naar een interne host via de bastion
ssh-copy-id -o ProxyJump=vagrant@192.168.62.253 vagrant@172.30.0.10    # 4, 15, 123
```

```bash
# ===== Bastion / Jump host =====
Host comp
  HostName 192.168.62.253
  User vagrant
  Port 22
  IdentityFile ~/.ssh/id_rsa

Host home
  HostName 192.168.62.42
  User vagrant
  Port 22

# ===== Handige aliassen (korte namen) =====
Host dns
  HostName 172.30.0.4
  User vagrant
  Port 22
  ProxyJump comp

Host web
  HostName 172.30.0.10
  User vagrant
  Port 22
  ProxyJump comp

Host db
  HostName 172.30.0.15
  User vagrant
  Port 22
  ProxyJump comp

Host emp
  HostName 172.30.0.123
  User vagrant
  Port 22
  ProxyJump comp

Host isp
  HostName 192.168.62.254
  User vagrant
  Port 22

Host rem
  HostName 172.10.10.123
  User vagrant
  Port 22
  ProxyJump home
```

```bash
# naar een interne host via de bastion op windows
Get-Content $env:USERPROFILE\.ssh\id_rsa.pub | ssh web "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

vim /etc/ssh/sshd_config

PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no

Op host (local forward): ssh -N -L localhost:1234:localhost:80 web
Op webserver (remote forward): ssh -N -R localhost:1234:localhost:80 vagrant@192.168.62.42
