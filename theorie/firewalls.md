# Les 2: Firewalls

## Firewalls in termen van het OSI-model

Een (stateful) firewall werkt vooral op laag 3/4 (IP, TCP/UDP): hij beslist op basis van bron/dest IP, poorten en connectiestatus welke pakketten door mogen.
Moderne “next-gen” firewalls inspecteren bovendien laag 7 (applicatie/HTTP, DNS, etc.) om regels te maken op basis van applicaties of URL-paden.
Er bestaan ook transparante/bridge firewalls die op laag 2 inline zitten zonder te routeren. Concreet: hoe hoger in het OSI-model, hoe context-rijker de beslissing, maar ook hoe zwaarder de inspectie.

## Host-based vs. Network-based firewall

Host-based firewall (HBF) draait op de eindhost (Windows Firewall, ufw/nftables/ipfw).

- Voordelen: werkt ook buiten het bedrijfsnetwerk; fijnmazige per-proces/per-user regels; microsegmentatie tot op de host; goede telemetrie dicht bij de workload.
- Nadelen: beheer-overhead (policy distributie); kan uitgeschakeld of gemanipuleerd worden bij compromittering; (lichte) performance-impact op de host.

Network-based firewall (NBF) staat centraal (fysiek of virtueel) tussen zones/VLANs of naar internet.

- Voordelen: één centrale policy; goed zicht op in- en uitgaand verkeer; vaak hardware-acceleratie; eenvoudiger compliance en logging.
- Nadelen: blind voor end-to-end versleuteling zonder TLS-inspectie; single point of failure/throughput-bottleneck; beschermt geen off-net of thuiswerkend device.

Best practice: combineer beide. NBF voor macrosegmentatie tussen zones; HBF voor microsegmentatie en “last line of defense”.

## Netwerksegmentatie en (firewall) zones

Segmentatie is het logisch scheiden van het netwerk in subnets/VLANs met eigen beveiligingsregels.
Een zone groepeert segmenten met vergelijkbaar vertrouwensniveau (bijv. *WAN/extern*, *DMZ*, *intern*, *beheer*, *gasten*, *OT/ICS*).
Tussen zones definieer je default-deny en laat je alléén noodzakelijke flows toe (bijv. web → app → db). Dit beperkt laterale beweging en de impact van incidenten.

## DMZ: wat en 1-firewall vs 2-firewall ontwerp

Een DMZ is een (semi)-vertrouwde zone waar publiek bereikbare diensten staan (web, mail-gateway, VPN concentrator). Doel: externe toegang isoleren van het interne LAN.

- Met 1 firewall (“three-legged”): één appliance met drie interfaces: WAN + DMZ + LAN.
  - Voordeel: eenvoud, kosten.
  - Nadeel: één enkel apparaat; fouten of misconfig hebben groter bereik.
- Met 2 firewalls (“back-to-back”): een edge firewall (WAN + DMZ) en een inside firewall (DMZ + LAN), liefst verschillende vendors (defense-in-depth).
  - Voordelen: extra laag, gescheiden beheer/telemetrie, kleinere blast-radius.
  - Nadelen: meer complexiteit, routing, hogere kosten.

## Nmap: poortstatussen en hoe Nmap dat afleidt

Nmap labelt per poort o.a. open, closed of filtered (er zijn ook combinaties zoals *open|filtered*).

- open: er luistert een service.  
- closed: geen service luistert, maar host is bereikbaar.  
- filtered: een firewall blokkeert of maskeert het antwoord.  

Voorbeelden:

```bash
# TCP SYN-scan op top 1000 poorten met service-detectie
sudo nmap -sS -sV 192.0.2.10

# UDP-scan op een paar bekende poorten
sudo nmap -sU -p 53,123,161 192.0.2.10

# Combineer TCP en UDP, plus OS- en script-detectie (intensiever)
sudo nmap -sS -sU -A 192.0.2.10
```

### Banner Grabbing

Banner grabbing is het opvragen van service-metadata (versie/implementatie) die servers vaak sturen bij connectie.

Waarom nuttig?

- Kwetsbaarheidsinschatting (versies herkennen).  
- Asset discovery in dynamische omgevingen (VMs, containers, serverless endpoints).  
- Troubleshooting van misconfiguraties.

```bash
# Service- en versie-detectie (probeert banners te lezen)
sudo nmap -sV -p 22,80,443 example.com

# Specifieke banners (NSE script)
sudo nmap -sV --script=banner -p 21,25,80,110,143 example.com
```

## systemd: waar staan configuraties?

Systemd zoekt unit-files in meerdere paden (later genoemde paden overriden eerdere):

- Distributie/vendor: /usr/lib/systemd/system of /lib/systemd/system
- Beheerder (persistent): /etc/systemd/system
- Runtime (door tools gegenereerd): /run/systemd/system
- User-units: ~/.config/systemd/user (en soms ~/.local/share/systemd/user)
- Drop-ins: directories als /etc/systemd/system/unit.d/*.conf voor overrides

```bash
# Toon de effectieve unit-file (incl. drop-ins) zoals systemd 'm ziet
systemctl cat <unit>.service

# Toon alle properties/statusvelden van de unit (raw key=value)
systemctl show <unit>.service

# Maak/bewerkt een drop-in override; opent $EDITOR en herlaadt nadien
sudo systemctl edit <unit>.service

# Lijst van actieve units (wat draait er effectief)
systemctl list-units [--type=service]

# Lijst van unit-files op schijf + enable/disable state
systemctl list-unit-files [--type=service]

# Herlaad unit-files na wijzigingen (geen services herstarten)
sudo systemctl daemon-reload
```

### Eigen systemd-service maken

Maak een unit-file in /etc/systemd/system/hello.service

```bash
[Unit]
Description=Hello demo service
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=appuser
Group=appuser
WorkingDirectory=/opt/hello
ExecStart=/usr/bin/python3 /opt/hello/app.py
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now hello.service
systemctl status hello.service
journalctl -u hello.service -e
```

### systemd-timers (cron-vervanger mét units)

Een timer (.timer) start een bijbehorende service (.service) op een schema.

- OnCalendar= voor tijdschema’s (“Mon..Fri 09:00”, “daily”, “--01 03:00:00”).
- OnBootSec=/OnUnitActiveSec= voor relatieve intervallen.

## Proxies & Load Balancing

### Wat is een (forward) proxy?

Een forward proxy zit aan de client-kant: clients praten eerst met de proxy, die namens hen het internet op gaat.
Doelen: policy/filtratie, caching, anonymisatie, egress-controle en auth.
Voorbeelden: Squid, Blue Coat/Symantec Proxy, Zscaler (cloud).

### Wat is een reverse proxy?

Een reverse proxy zit voor de servers en ontvangt inkomend verkeer van het internet.
Functies: TLS-terminatie, WAF, caching, compressie, path-based/host-based routing, rate-limiting en auth.
Voorbeelden: Nginx, HAProxy, Envoy, Traefik, Apache httpd (mod_proxy), Caddy.

### Relatie met load balancers

Veel reverse proxies kunnen load balancen: ze verdelen verkeer over meerdere backends (L4: TCP/UDP; L7: HTTP-headers/URL).
Traditionele load balancers (hardware of cloud) doen vaak hetzelfde, plus health checks, sticky sessions, en geavanceerde algoritmen (least-conn, EWMA).

Software die reverse proxy én load balancer kan zijn: Nginx, HAProxy, Envoy, Traefik, Apache httpd.

## Lab

Wat is een 'attack vector'? Een attack vector is de *route of methode* waarlangs een aanvaller toegang probeert te krijgen tot een doelwit om een schadelijke actie uit te voeren.
Het is dus “hoe ze binnenkomen” —via welk kanaal/medium— niet per se *welk* specifiek lek of *welke* malware wordt gebruik.

sudo dnf install firewalld
sudo systemctl enable --now firewalld
sudo firewall-cmd --add-service http --permanent
sudo firewall-cmd --add-service https --permanent
sudo firewall-cmd --remove-service ssh --permanent
sudo firewall-cmd --add-rich-rule='rule family=ipv4 source address=192.168.62.254/32 service name=ssh log prefix="SSH Logs" level="notice" accept'  --permanent
sudo firewall-cmd --add-rich-rule='rule protocol value=icmp reject' --permanent (niet handig voor testen met nmap!)
sudo firewall-cmd --list-all
sudo firewall-cmd --list-rich-rules
sudo firewall-cmd --reload

<https://wiki.alpinelinux.org/wiki/Uncomplicated_Firewall>

sudo apk add ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from 172.30.255.254 to any port 22 proto tcp
sudo ufw enable
sudo ufw status numbered

rc-service ServiceName status
rc-status
rc-service ServiceName start/stop/restart
