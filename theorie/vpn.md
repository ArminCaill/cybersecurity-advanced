# Les 10: VPN

## Review some downsides of IPsec

- IPsec is krachtig maar complex: policy’s (SPD/SA’s), IKE-profielen en certificaten maken beheer en troubleshooting lastig.
- Het heeft compatibiliteitsproblemen met NAT (AH breekt vaak; ESP vereist NAT-T/UDP 4500) en faalt sneller bij asymmetrische routing of strikte firewalls.
- Er is overhead: extra headers verkleinen effectieve payload, veroorzaken fragmentatie en soms zwarte gaten.
- Crypto kost performance (CPU) en vraagt goede key-/cert-levensduurbeheer en kloksync.
- Interoperabiliteit tussen vendors/versies kan stroef zijn; grootschalige mesh-tunnels en mobiele/roamende clients schalen minder soepel dan TLS/HTTPS-gebaseerde oplossingen.
- Omdat ESP de payload versleutelt, verlies je zichtbaarheid voor NIDS/WAF tenzij je extra sensoren of decryptie voorziet.

## what is a CA is and how does it work?

Een CA (Certification Authority) is een vertrouwde instantie die digitale certificaten uitgeeft en ondertekent, waarmee zij een publieke sleutel aan een identiteit (domeinnaam, organisatie, persoon) koppelt.
Zo werkt het kort: de aanvrager maakt een sleutelpaar en stuurt een CSR (met publieke sleutel en identiteit) naar de CA; de CA verifieert de identiteit (bv. domeincontrole), tekent vervolgens een X.509-certificaat met haar private key, en publiceert eventueel intrekkingsinformatie (CRL/OCSP). Clients (browsers/OS) vertrouwen het certificaat omdat zij de root- of intermediate-CA in hun trust store hebben en bij verbinding de certificaatketen tot aan die vertrouwde root kunnen valideren.

## Review how it is possible for us to browse to https://chamilo.hogent.be and don't receive a warning in our browser? How does this work. You should be able to fully explain this in detail.

Omdat je browser TLS/PKI-vertrouwen kan opbouwen: je gaat naar https://chamilo.hogent.be, de browser doet DNS-resolutie naar een IP en start op poort 443 een TLS-handshake waarbij hij via SNI de hostnaam doorgeeft; de server stuurt een certificaatketen (certificaat voor chamilo.hogent.be + één of meer intermediates) terug, en jouw browser verifieert of (1) de hostnaam in het cert staat (CN/SAN-match), (2) de digitale handtekeningen in de keten geldig zijn tot een root-CA die al in de trust store van je OS/browser zit, (3) het cert niet verlopen of ingetrokken is (eventueel via OCSP/CRL, vaak met OCSP-stapling), en (4) de TLS-versie/ciphers aan beleid voldoen; slaagt dit alles, dan wordt een versleutelde sessie opgezet en verschijnt geen waarschuwing, terwijl je wél een waarschuwing zou krijgen bij bv. naam-mismatch, self-signed/ongekende CA, verlopen of ingetrokken certificaat.

## What is the goal of OpenVPN?

Het doel van OpenVPN is veilige, versleutelde tunnels opzetten over onbetrouwbare netwerken (internet) zodat apparaten of hele netwerken privé met elkaar kunnen communiceren, zowel remote-access (client-naar-site) als site-to-site, met authenticatie (certificaten/keys) en integriteit via TLS, over UDP of TCP.

## How does OpenVPN work? What are the crucial elements to have a working OpenVPN setup?

OpenVPN zet een versleutelde tunnel op over internet door een TLS-besturingskanaal op te bouwen (server-client authenticeren met certificaten/keys) en daarna het dataverkeer via een symmetrisch versleuteld kanaal in een virtuele interface (TUN = L3/IP of TAP = L2/Ethernet) te vervoeren; de server “pusht” routes/DNS, en met IP-forwarding/NAT kan verkeer veilig het interne netwerk in.

Cruciale elementen voor een werkende setup (kort):

- PKI/identiteiten: eigen CA, servercertificaat + clients certificaten/keys, CRL; optioneel tls-crypt/tls-auth om het TLS-kanaal te beschermen.
- Serverconfig: dev tun, proto udp (1194), server 10.8.0.0/24 of topology subnet, juiste cipher/MAC en keepalive; logging aan.
- Netwerk/Firewall: UDP 1194 open; IP-forwarding aan; indien nodig NAT (MASQUERADE) naar het LAN; routes naar interne netten publiceren.
- Clientprofiel (.ovpn): remote <fqdn> 1194 udp, referenties naar CA/server chain en clientkey; optioneel user/password.
- Platformdrivers: werkende TUN/TAP-driver (bv. wintun/tap-windows op Windows, tun module op Linux).
- Beheer/veiligheid: sleutelrotatie, sterke crypto-suites, geen split-tunnel waar niet gewenst, en revocation bij verlies van een clientkey.

## What is PKI?

PKI (Public Key Infrastructure) is het geheel van rollen, processen en systemen (o.a. CA’s, registrars, policies, CRL/OCSP) waarmee publieke-sleutelcertificaten worden uitgegeven, beheerd en gevalideerd, zodat je identiteiten kunt authenticeren, data versleutelen en digitale handtekeningen betrouwbaar kunt controleren.

## What are the fundamental differences between IPsec and Openvpn?

- Laag (OSI): IPsec werkt op netwerkniveau (L3) en beschermt IP-pakketten zelf; OpenVPN bouwt een TLS-tunnel over TCP/UDP (L4/L5).
- Doel: beiden willen veilige VPN-tunnels bieden; IPsec is “native” L3-beveiliging (transport/tunnel) met IKE, OpenVPN is een TLS-gebaseerde gebruikersruimte-tunnel.
- Wanneer IPsec? Standaardprotocollen, site-to-site tussen firewalls/routers, vaak hogere performance en hardware-offload, brede OS-integratie.
- Wanneer OpenVPN? NAT-/firewall-vriendelijk (één poort, UDP/TCP), makkelijker uit te rollen voor remote clients, flexibel (TUN/TAP), minder vendor-interop gedoe.
- Trade-offs: IPsec is krachtig maar complexer (IKE/policies, MTU/NAT-T); OpenVPN is eenvoudiger en robuust door lastige netwerken, maar soms lagere throughput/meer overhead dan goed geconfigureerde IPsec met offload.

## Is wireguard more comparable to OpenVPN or to IPsec or to both?

WireGuard lijkt in laag en principe meer op IPsec (L3-tunnel over UDP, geen TLS), maar is qua gebruik en uitrolsimpliciteit dichter bij OpenVPN (eenvoudige config, goede NAT-traversal); kortom: het zit tussen beide in, IPsec-achtig qua laag/encapsulatie, OpenVPN-achtig qua praktische inzet.

## Lab

[vagrant@companyrouter ~]$ sudo dnf install --assumeyes openvpn easy-rsa

Public Key Infrastructure:

https://github.com/OpenVPN/easy-rsa/blob/master/README.quickstart.md

Op companyrouter:
ww = openvpn
common name = web
CA certificate is at: /home/vagrant/easy-rsa/easyrsa3/pki/ca.crt

Op web:
ww = webrequest
Private-Key and Public-Certificate-Request files created.
Your files are:

- req: /home/vagrant/easy-rsa/easyrsa3/pki/reqs/web.req
- key: /home/vagrant/easy-rsa/easyrsa3/pki/private/web.key

scp vagrant@172.30.0.10:/home/vagrant/easy-rsa/easyrsa3/pki/reqs/web.req vagrant@172.30.255.254:/home/vagrant

Certificate created at: /home/vagrant/easy-rsa/easyrsa3/pki/issued/web.crt

sudo openssl verify -CAfile /home/vagrant/easy-rsa/easyrsa3/pki/ca.crt /home/vagrant/easy-rsa/easyrsa3/pki/issued/web.crt

ww = clientrequest
Private-Key and Public-Certificate-Request files created.
Your files are:

- req: /home/vagrant/easy-rsa/easyrsa3/pki/reqs/client.req
- key: /home/vagrant/easy-rsa/easyrsa3/pki/private/client.key

Certificate created at: /home/vagrant/easy-rsa/easyrsa3/pki/issued/client.crt

DH parameters of size 2048 created at: /home/vagrant/easy-rsa/easyrsa3/pki/dh.pem

Configuring the server??
