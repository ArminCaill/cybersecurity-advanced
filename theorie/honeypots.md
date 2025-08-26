# Les 5: honeypots

## What is a honeypot

Een honeypot is een bewust nep systeem of dienst —vaak kwetsbaar of aantrekkelijk ingericht— dat aanvallers moet lokken zodat je hun gedrag kunt observeren, vroegtijdig detecteren en dreigingsinformatie verzamelen, zonder echte assets te riskeren. Hij staat strikt geïsoleerd en zwaar gelogd; normaal hoort er geen legitiem verkeer op binnen te komen.

## What type of honeypots exist?

Er bestaan grofweg low-, medium- en high-interaction honeypots:

- low emuleren slechts protocollen/poorten (veilig en licht)
- high draaien echte diensten/OS in een strak geïsoleerde sandbox (rijkere intel maar meer risico/beheer)
- medium zit ertussenin.

Qua functie zie je:

- netwerk-/service-honeypots (bv. SSH/HTTP/ICS)
- web/app-honeypots (vallen voor SQLi/XSS)
- client-honeypots die bewust malafide sites/docs bezoeken
- honeytokens (valse credentials/URLs/API-keys) die alerten zodra ze gebruikt worden.

Qua gebruik onderscheid je production honeypots binnen je omgeving voor vroege detectie/afleiding, en research honeypots aan de internetkant om campagnes, malware en TTP’s te bestuderen.

Wat ze proberen te bereiken: detectie en early-warning, inzamelen van IOC’s/TTP’s voor threat intel, misleiding en vertraging van aanvallers (deception), en het kanaliseren van ongewenst verkeer naar een gecontroleerde speelweide.
Trade-off: low-interaction is veiliger maar oppervlakkig; high-interaction levert diepe inzichten tegen de prijs van strengere isolatie, continu toezicht en meer operationele zorg.

## How do honeypots differ from honey-/canarytokens?

- Een honeypot is een vals, interactief systeem of dienst (bijv. nep-SSH/webserver) dat aanvallers lokt zodat je hun gedrag kunt observeren en analyseren.
- Honey-/canarytokens zijn daarentegen lok-artefacten (bijv. valse API-sleutel, URL, document of DNS-naam) die geen dienst draaien maar een stil alarm af laten gaan zodra ze worden geopend of gebruikt.
- Kortom: honeypot = interactief lokaas met rijke telemetry; honeytoken = lichtgewicht trigger die onmiddellijk misbruik detecteert.

## Docker

### Is Docker virtualisation (type 1 or type 2), emulation, simulation?

Docker is OS-level containerisatie: processen delen de host-kernel en zijn geïsoleerd met namespaces/cgroups.
Het is dus geen type-1 of type-2 hypervisor, geen emulatie en geen simulatie (behalve bij multi-arch met qemu tijdens builds). Op macOS/Windows draait Docker binnen een lichte VM om een Linux-kernel te leveren, maar de containers zelf blijven OS-level.

### What are some security implications when using Docker? Is it considered to be more secure compared to virtual machines? Why (not)?

Docker draait containers die de host-kernel delen, dus een kernel-bug of container-breakout kan de hele host raken. Risico’s zitten ook in te ruime privileges (root in de container, toegang tot docker.sock ≈ root), supply-chain (malafide/vuile images, gelekte secrets) en standaard capabilities/netwerk die te open staan.
Vergeleken met VM’s bieden containers zwakkere isolatie (geen eigen kernel, kleinere maar scherpere scheidingslaag), terwijl VM’s via een hypervisor sterkere tenant-isolatie geven.

Kortom: Docker is niet per se veiliger dan VM’s; het is lichter en snel, maar vraagt strak hardenen (rootless, capabilities droppen, read-only FS, seccomp + AppArmor/SELinux, user namespaces, geen docker.sock mounten, image-scans/signing) en frequente kernel-patching. Voor onbetrouwbare/multi-tenant workloads blijft een VM vaak de veiligere keuze.

### Would you use Docker to deploy a honeypot in production? Why (not)?

Meestal niet: een honeypot trekt actief aanvallers aan en Docker-containers delen de host-kernel, waardoor een breakout de hele host kan raken; in productie kies je liever een dedicated (virtuele) machine voor sterkere isolatie en gecontroleerde egress. Als je tóch Docker inzet, doe dat binnen een aparte VM, rootless en zonder --privileged, met minimale capabilities, read-only filesystem en strikte netwerk-/logisolatie.

## Lab

https://reintech.io/blog/installing-docker-on-almalinux-9
https://www.youtube.com/watch?v=kvQLeaj2DGI
https://github.com/vfreex/docker-cowrie/blob/master/README.md

docker run -p 2222:2222 cowrie/cowrie:latest
ssh -p 2222 comp (wachtwoord vagrant werkt niet)

What are some (at least 2) advantages of running services (for example cowrie but it could be sql server as well) using docker?
Docker maakt services consistent en portable (alle dependencies in één image), waardoor je ze snel kunt uitrollen of terugrollen op elke host met Docker. Containers zijn lichtgewicht en geïsoleerd, dus je kunt makkelijk meerdere versies naast elkaar draaien zonder package-conflicten en ze schoon verwijderen. Bovendien kun je met Compose een hele stack declaratief starten, en via resource-limits en capabilities het gedrag en de impact van een service strak begrenzen.

What could be a disadvantage?
Een nadeel is dat containers de host-kernel delen, waardoor een fout of misconfiguratie (bv. --privileged of toegang tot docker.sock) een grotere impact kan hebben dan verwacht; bovendien brengt Docker extra complexiteit mee rond netwerk/volumes en image-patching, wat debugging en security lastiger maakt.

Explain what is meant with "Docker uses a client-server architecture."
Docker bestaat uit een client (de docker-CLI) die via een API (Unix-socket of TCP) praat met de server/daemon (dockerd): de client stuurt opdrachten (build, run, pull), de daemon voert ze uit en beheert images/containers — lokaal of remote op een andere host. Deze scheiding maakt centrale aansturing en automatisering mogelijk.

As which user is the docker daemon running by default?
Standaard draait de Docker-daemon (dockerd) als root. De socket (/var/run/docker.sock) is root:docker, waardoor leden van de docker-groep in de praktijk root-machtigingen krijgen via de daemon; alleen in rootless mode (niet standaard) draait hij als niet-root.

What could be an advantage of running a honeypot inside a virtual machine compared to running it inside a container?
Een VM biedt sterkere isolatie (eigen kernel + hypervisor-scheiding), waardoor een compromittering van de honeypot veel minder snel de host of andere workloads raakt; bovendien kun je met snapshots/rollbacks en aparte netwerksegmentatie de impact beperken en forensics eenvoudiger doen.

What type of honeypot is "honeyup"?
Honeyup is een webapplicatie-honeypot die een kwetsbare uploadfunctie nabootst om uploadpogingen en payloads te vangen—typisch low- tot medium-interaction (focus op het upload-endpoint, geen volledig echte applicatie/OS).

What is the idea behind "opencanary"?
OpenCanary is een lichtgewicht, open-source honeypot: het draait nepservices (zoals SSH/FTP/HTTP/SMB) en stuurt meteen een alert zodra iemand ermee communiceert, zodat je met minimale setup vroegtijdig indringers detecteert.

Is a HTTP(S) honeypot a good idea? Why or why not?
Ja, maar met beleid: een HTTP(S)-honeypot kan vroege detectie en dreigings-intel opleveren, maar trekt extreem veel “ruis” en aanvallen aan, en als hij wordt misbruikt kan hij als pivot dienen. Kies daarom alleen als je hem strak isoleert (liefst in een aparte VM/DMZ), geen echte data aanbiedt en egress blokkeert; anders leveren WAF-/serverlogs en honeytokens vaak minder risico en onderhoud op voor vergelijkbare signalen.
