# Les 11: CA

## Assume your browser gives and error when browsing to a HTTPS website, telling you that the certificate is not valid anymore (for example expired). If you however decide to accept the risk and continue, is your traffic still encrypted or not? Explain

Ja, je verkeer blijft technisch gezien versleuteld (er wordt nog steeds een TLS-sessie opgezet), maar zonder geldige certificaatvalidatie weet je niet met wie je versleutelt. Je verliest dus authenticiteit/identiteitsgarantie en bent kwetsbaar voor man-in-the-middle: een aanvaller kan een ander (ongeldig/expired/self-signed) certificaat aanbieden, waarna je keurig versleuteld praat maar mogelijk met de aanvaller in plaats van met de echte site.

## What is X.509?

X.509 is de standaard voor digitale certificaten en intrekkingslijsten binnen PKI: het definieert het formaat en de velden (o.a. subject, issuer, publieke sleutel, geldigheid, handtekening en extensies zoals SAN/KeyUsage) waarmee clients de identiteit en sleutels van servers/gebruikers kunnen valideren (bv. in TLS, S/MIME, code-signing).

## What is meant by CSR in this context?

Een CSR (Certificate Signing Request) is het bestand dat je naar een CA stuurt om een certificaat aan te vragen: het bevat je publieke sleutel en identiteitsgegevens (bijv. CN/SAN), is digitaal ondertekend met je private key als bewijs van sleutelbezit, en wordt door de CA gebruikt om na verificatie een X.509-certificaat uit te geven; de CSR bevat nooit je private key.

## What is SAN, Subject Alternative Name?

SAN (Subject Alternative Name) is een X.509-certificaatextensie die extra identiteiten bevat zoals DNS-namen, IP-adressen, e-mailadressen of URI’s—waarvoor het certificaat geldig is; moderne browsers valideren hostnamen primair tegen de SAN (een CN alleen is niet meer voldoende).

## What are Certificate chains and cross-certifications?

Certificate chains zijn de reeks certificaten van een server/leaf via één of meer intermediate CA’s naar een root-CA in je trust store; elke schakel is door de volgende ondertekend, zodat je browser de identiteit kan vertrouwen via een geldige keten.

Cross-certifications ontstaan wanneer een CA het publieke sleutelcertificaat van een andere CA mede ondertekent, zodat er alternatieve geldige ketens naar verschillende vertrouwde roots bestaan. Handig voor interoperabiliteit, migraties/rollovers en redundantie tussen PKI’s.

## How does a CA certificate renewal work using a cross-certification

Bij een CA-vernieuwing met cross-certificatie maakt de CA een nieuwe (root of intermediate) CA-key en certificaat en laat dat cross-signen door een bestaand, al vertrouwd root-certificaat. Servers gaan vervolgens de nieuwe keten meegeven die via de cross-signed intermediate nog steeds terugleidt naar de oude vertrouwde root, zodat alle clients blijven valideren terwijl de nieuwe root/intermediate wereldwijd wordt uitgerold. Zodra de nieuwe root overal in trust stores staat, kan de cross-sign worden verwijderd en wordt alleen de nieuwe, “eigen” keten gebruikt. Kortom: cross-certificatie creëert een overgangsbrug die continuïteit garandeert tijdens CA-rotatie of -verval.

## What is the difference between SSL and TLS? What is currently the standard? Which version?

SSL is de oude, achterhaalde voorloper van TLS; TLS is het moderne, veiliger protocol voor versleutelde verbindingen (SSL 2.0/3.0 en TLS 1.0/1.1 zijn uitgefaseerd). De gangbare standaard is TLS 1.3 (met brede support; TLS 1.2 blijft als fallback).

## What is Let's Encrypt?

Let’s Encrypt is een gratis, geautomatiseerde en open certificaatautoriteit (van ISRG) die domein-gevalideerde TLS-certificaten uitgeeft via het ACME-protocol; ze zijn 90 dagen geldig en bedoeld voor automatische uitrol en vernieuwing, en worden breed door browsers vertrouwd.

Isn't it a bad thing that people - including hackers - can create webserver certificates that can be signed for free by a trusted CA that is installed in all recent devices? Explain why/not.

Niet per se: een TLS-certificaat bewijst alleen domeincontrole en levert versleuteling + integriteit, geen “goedkeuring” van de inhoud. Gratis DV-certs (zoals Let’s Encrypt) maken het wél makkelijker om álle sites te versleutelen, zodat afluisteren en manipulatie veel lastiger wordt. Ja, ook aanvallers kunnen voor hun eigen (phishing)-domeinen een DV-cert krijgen, maar dat verandert het risico nauwelijks, ze konden eerder ook al certificaten krijgen; het echte probleem is misleiding met look-alike domeinen, niet het bestaan van TLS. Misbruik wordt bovendien beperkt via Certificate Transparency, blokkades/blacklists en intrekking. Kortom: gratis, breed beschikbare DV-certs verhogen de basisveiligheid van het web, zonder een keurmerk te zijn.

## Lab

https://www.ibm.com/docs/en/informix-servers/15.0.0?topic=openssl-setting-up-ca

ww = ca1passwd

https://www.linuxjournal.com/content/how-secure-your-website-openssl-and-ssl-certificates
