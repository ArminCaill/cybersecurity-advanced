# Les 1: DNS

## What is the swiss cheese model and how can it be applied to cybersecurity?

Het Swiss-cheese-model (James Reason) stelt dat beveiliging uit meerdere lagen bestaat, elk met ‘gaten’ (zwaktes). Een incident gebeurt wanneer de gaten in meerdere lagen tijdelijk op één lijn komen.

Toepassing:

- Defense-in-depth: combineer lagen: policies, awareness, identiteitsbeheer (MFA), netwerksegmentatie, Endpoint Detection en Response/Antivirus, patching, back-ups, monitoring.
- Incidentanalyse: map per laag welke controle faalde (bv. ontbrekende patch + zwakke segmentatie + geen egress-monitoring → Remote Code Execution + laterale beweging + data-exfiltratie).
- Verbeteren: verklein gaten (patch/ hardening), verschuif ze (compensating controls), en voeg lagen toe (MFA, allow-lists, egress-filters).

## What different type of network attacks exist?

- Recon/scans: ping sweep, portscan (TCP SYN/Connect, UDP), banner grabbing.
- Spoofing/MitM: ARP-spoofing, DNS-spoofing, IP-/MAC-spoofing, SSL-stripping, rogue AP / evil twin.
- Protocol-/infra-misbruik: DHCP starvation/rogue DHCP, STP-/VLAN-hopping, BGP-hijacking.
- DoS/DDoS: volumetrisch (UDP/ICMP floods), protocol (SYN flood), applicatie (HTTP GET/POST flood).
- Fragment/legacy tricks: smurf, ping of death, teardrop.
- DNS-specifiek: cache poisoning, amplification, tunneling, rebinding, zone-transfer-misconfig.
- Web/app-laag (netwerkimpact): slowloris, websocket floods.
- Wi-Fi: deauth/disassoc floods, WPS brute-force.

### What is a (D)DoS attack?

- DoS: één bron overspoelt een doelwit (bandbreedte/CPU/app).
- DDoS: veel verspreide bronnen (botnet) tegelijk; moeilijker te blokkeren.

### How can DNS be considered an attack vector as well?

- Amplification/reflectie: kleine query → groot antwoord naar spooffed bron (open resolvers, EDNS0).
- Cache poisoning: valse records in recursorcache.
- Tunneling/exfil: data in TXT/labels (iodine, dnscat2).
- Rebinding: browser omzeilt same-origin naar intern IP via snelle DNS-wissels.
- Misconfig/lekken: zone-transfer open, infosrijke TXT/SRV.

## DNS

### What information can be enumerated from a DNS server

- Recordtypes: A/AAAA (hosts), CNAME (aliassen), MX (mail), NS/SOA (authoritatief/domein-details), TXT (SPF/DKIM/DMARC, soms tooling-sporen), SRV (services), PTR (reverse), CAA (welke CA mag certs uitgeven).
- Structuur & naming-conventies: subdomeinen die interne services verraden (vpn., dev., staging., jira., etc.).

### When is it intended? What is considered a "normal" DNS resolve and how can you perform it using a CLI tool?

- Normale resolutie: je vraagt een specifieke naam en krijgt bijbehorende records terug (geen volledige zone).
- Bedoeld vs. niet bedoeld: losse lookups zijn normaal. Volledige zone-lijsten of interne namen zijn doorgaans niet bedoeld om publiek uit te lezen.

```bash
nslookup www.ugent.be
dig www.ugent.be A           # standaard lookup
dig +short www.ugent.be      # beknopt
dig +short NS ugent.be       # authoritative server
```

### What is, and how can you perform a reverse lookup?

- Betekenis: IP → hostname via PTR record in in-addr.arpa (IPv4) of ip6.arpa (IPv6).

```bash
dig +short -x 157.193.43.50
host 8.8.8.8
```

### What is meant by authoritative nameservers?

- De NS-servers die “de waarheid” voor een zone beheren en beantwoorden zonder te cachen. Ze dienen als bron voor recursors.

```bash
dig NS www.ugent.be +short
dig SOA www.ugent.be
```

### What is a zone transfer attack and why is it called an attack? Is a zone transfer always harmful?

- Legitiem doel: sync tussen primary en secondary nameservers.
- Aanval: als een authoritatieve NS per ongeluk iedereen AXFR/IXFR laat doen, kan een aanvaller de volledige zone downloaden (hostlijst → recon goudmijn).
- Is het altijd schadelijk? Nee. Tussen vertrouwde servers is het normaal; het wordt pas een probleem bij ongeautoriseerde transfers.

```bash
dig AXFR www.ugent.be @ns1.www.ugent.be
# of
dig IXFR=12345 www.ugent.be @ns1.www.ugent.be

```

## tcpdump (or alternatives)

### How can you create a network dump, only using the CLI, on a machine without a GUI?

```bash
# Interfaces tonen
tcpdump -D

# Alles op een interface, “snaplen” onbeperkt (-s 0), schrijven naar pcap:
sudo tcpdump -i eth0 -s 0 -w /tmp/capture.pcap

# Rotatie (100 MB per file, max 5 files)
sudo tcpdump -i eth0 -s 0 -C 100 -W 5 -w /tmp/cap-%Y%m%d-%H%M%S.pcap

```

Alternatieven: tshark (CLI-Wireshark), dumpcap (performant capture-daemon).

### How can you exclude SSH traffic?

Handig bij remote captures zodat je je eigen SSH-sessie niet logt.

```bash
# Alle SSH (poort 22) uitsluiten
sudo tcpdump -i eth0 -s 0 -w /tmp/capture.pcap 'not tcp port 22'

# Nog specifieker: sluit alleen jouw SSH naar de host uit
sudo tcpdump -i eth0 -s 0 -w /tmp/capture.pcap 'not (tcp port 22 and host <jouw_client_ip>)'

```

### How can you only include HTTP traffic?

```bash
# Klassieke HTTP/1.1
sudo tcpdump -i eth0 -s 0 -w /tmp/http.pcap 'tcp port 80'

# Veelgebruikte alternatieve poorten erbij
sudo tcpdump -i eth0 -s 0 -w /tmp/http.pcap '(tcp port 80 or tcp port 8080 or tcp port 8000)'

# Let op: HTTPS is versleuteld (poort 443, TCP of QUIC/UDP) en toont geen inhoud.
# HTTP/3 (QUIC) herkennen/opnemen:
sudo tcpdump -i eth0 -s 0 -w /tmp/quic.pcap 'udp port 443'

```

Tip: Voeg -nn toe om geen DNS/poort-namen te resolven en -v/-vv voor extra details.

## Wireshark

### What can an analyst learn from the following windows inside wireshark

- Conversations:
  - Wat: Tabellen per laag (Ethernet/IP/TCP/UDP/IPv6) met 2-wegs “gesprekken” (src↔dst), #packets, bytes, duur, throughput; voor TCP extra info (retransmissions, RTT bij filters).
  - Waarom nuttig: vind top talkers, ongewone peers, lange/spraakzame sessies, asymmetrie (veel uitgaand t.o.v. ingaand), mogelijke data-exfil of DoS-bronnen.
- Statistics:
  - Summary: totale duur, aantal (gecapturede/gedisplayde) pakketten, gemiddelde pps/throughput, linktype, capture-filter. => Baseline van het verkeer; spot afwijkingen (plots pieken/dips).
  - I/O Graphs: tijdreeksen voor pps/bytes; makkelijk om spikes (DDoS/scan), periodiciteit of burst-patronen te zien.
- Protocol Hierarchy:
  - Wat: boom met percentage per protocol (L2→L7) en bytes/packets per tak.
  - Waarom nuttig: snel zien welke protocollen domineren (bv. veel TLS vs. ongewoon SMB/DNS-TXT), en afwijkingen herkennen (bv. onverwacht QUIC/UDP op servernet).

## Lab

- Figure out a way to sniff traffic origination from the employee using tcpdump on the companyrouter: `sudo tcpdump -i eth2 -n host 172.30.0.123`
- Figure out a way to capture the data in a file. Copy this file from the companyrouter to your host and verify you can analyze this file with wireshark (on your host): `sudo tcpdump -i eth2 -w capture.pcap` en `scp vagrant@192.168.62.253:/home/vagrant/capture.pcap .`
- How can you start tcpdump and filter out this ssh traffic? `sudo tcpdump -i eth2 port 22`
- Find a way to capture only HTTP traffic and only from and to the webserver-machine: `sudo tcpdump -i eth2 host 172.30.0.10 and port 80 -w http-capture.pcap`

Configure this machine correctly in such a way that it has internet access and is able to connect to all other virtual machines of the environment.

`sudo nano /etc/network/interfaces`

```bash
auto eth0
iface eth0 inet static
    address 192.168.62.142
    netmask 255.255.255.0
    gateway 192.168.62.254
```

`sudo systemctl restart networking`

What is the default gateway of each machine? `ip route`
What is the DNS server of each machine? `cat /etc/resolv.conf`
Which machines have a static IP and which use DHCP? `cat /etc/network/interfaces`

Investigate whether the DNS server of the company network is vulnerable to a DNS zone transfer "attack": on kali: `dig @172.30.0.4 cybersec.internal AXFR` of windows `nslookup server 172.30.0.4 set type=AXFR cybersec.internal`
Try to configure the server to allow & prevent this attack: `sudo cat /etc/bind/named.conf`

```bash
zone "cybersec.internal" {
    type master;
    file "/var/bind/cybersec.internal";
    allow-transfer { any; };  # Allow zone transfers from any machine or change 'any' to 'none' or specific IP
};
```

`sudo systemctl restart bind9`
