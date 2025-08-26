# Les 4: IDS en IPS

## What is the difference between an IDS and an IPS?

- Een IDS (Intrusion Detection System) luistert mee naar netwerk- of host-verkeer en alarmeert bij verdacht gedrag, maar grijpt zelf niet in
- Een IPS (Intrusion Prevention System) zit inline in de datastroom en kan verdacht verkeer actief blokkeren/onderbreken.
- Kort: IDS = detecteren & melden (vaak out-of-band), IPS = detecteren & voorkómen (inline, met risico op verstoringen bij false positives).

## What are some fundamental differences between a firewall and an IDS/IPS, conceptually?

- Een firewall is vooral een beleids-handhaver: hij beslist (meestal stateful, op L3/L4 en soms L7) welk verkeer überhaupt mag passeren op basis van IP/poort/identiteit/zone en doet segmentatie/NAT.
- Een IDS/IPS is vooral een detectiemechanisme: het analyseert (dieper op L7) ook toegestaan verkeer op kwaadaardige patronen/anomalieën; IDS alarmeert, IPS kan inline blokkeren.
- Kort: firewall stopt ongeautoriseerd verkeer volgens policy; IDS/IPS detecteert of stopt malicious gedrag binnen ogenschijnlijk geautoriseerd verkeer (met meer kans op false positives bij IPS).

## physical IDS/IPS device

Where would you place it in the network if it would be separate from your firewall?
Zet een los IDS/IPS op een chokepoint: typisch achter de firewall aan de internet-edge of tussen kritieke zones (bv. LAN-DMZ).
How would you set this up?
Als IDS hang je het out-of-band via een SPAN/mirror-poort of TAP zodat het passief meekijkt (aan switch achter firewall); als IPS plaats je het inline (bridge/L2) zodat het verkeer kan blokkeren (tussen firewall en switch).
What is the impact on the network by adding this device?
IDS voegt geen latency toe maar kan verkeer missen bij slechte mirroring; IPS voegt extra latency/throughput-eisen toe, kan een bottleneck of fout-positieve drops veroorzaken en vraagt zorg voor symmetrische routing/MTU en capaciteit.

## What is security onion?

Security Onion is een open-source Linux-platform om snel een SOC/monitoringstack op te zetten voor intrusion detection, logbeheer en threat hunting. Het bundelt o.a. Suricata/Zeek (netwerksensoren), osquery/Wazuh (endpoint-telemetrie) en dashboards/zoektools in één beheerbare omgeving, zodat je via SPAN/TAP en agenten netwerk- en hostgegevens centraal kunt verzamelen, analyseren en alarmeren.

## Lab

https://cloudspinx.com/install-suricata-on-rocky-linux-8almalinux-8/

What is the difference between the fast.log and the eve.json files?

- fast.log = simpel, regel-per-alert tekstlog. Snel te lezen/greppen, minimale info (tijd, src/dst, poorten, SID, message). Lichtgewicht en handig voor snelle triage.
- eve.json = rijk, JSON-geformatteerde eventstream. Bevat niet alleen alerts maar ook flows, DNS/HTTP/TLS, fileinfo, stats, extra metadata (sensor, rule, payload-refs, tags), en is gemaakt voor SIEM/ELK en correlatie.
- Paden (typisch):
  - /var/log/suricata/fast.log
  - /var/log/suricata/eve.json (aan/uit en detail via suricata.yaml)

sudo nano /var/lib/suricata/rules/suricata.rules

```bash
alert icmp any any -> $HOME_NET any (msg:"ICMP ping"; sid:1; rev:1;)
```
