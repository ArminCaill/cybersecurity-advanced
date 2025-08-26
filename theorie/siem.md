# Les 8: SIEM

## What is a SIEM?

Een SIEM (Security Information and Event Management) is een platform dat logs en beveiligingstelemetrie centraal verzamelt, correleert en alarmeert om dreigingen te detecteren, onderzoeken en rapporteren.
Voorbeelden zijn Microsoft Sentinel, Splunk (Enterprise Security), IBM QRadar, Elastic Security (SIEM), LogRhythm, Exabeam, ArcSight, Sumo Logic en Graylog.

## What is a SOC?

Een SOC (Security Operations Center) is het team en de (vaak 24/7) operatie die met tools zoals SIEM continu het IT-landschap monitort, dreigingen detecteert, incidenten onderzoekt en afhandelt, en via playbooks en threat intel de weerbaarheid van de organisatie verbetert.

## What is meant by compliancy in terms of cybersecurity?

Compliancy in cybersecurity betekent dat je organisatie voldoet aan externe eisen en interne policies (bv. GDPR, ISO 27001, NIS2, PCI DSS) én dit kan aantonen via passende controls, documentatie en audits; het levert een minimale, toetsbare baseline, maar is niet hetzelfde als volledige veiligheid.

## What features does Wazuh offer?

Wazuh biedt agent-gebaseerde endpoint-security met o.a. logverzameling en -analyse, bestandsintegriteitsbewaking (FIM), kwetsbaarheids- en configuratie-assessments, rootkit/malware-detectie, intrusion detection + active response, compliance-checks (bv. CIS/PCI/GDPR), threat-intel correlatie, en centrale SIEM-achtige dashboards en alerts (integreert met Elastic/OpenSearch).

## What is FIM?

FIM (File Integrity Monitoring) is het continu bewaken van kritieke bestanden en configuraties op ongeautoriseerde of onverwachte wijzigingen (wie/wat/wanneer/hoe), zodat je snel compromittering, misbruik of fouten kunt detecteren en forensisch kunt aantonen wat er is veranderd.

## What is Sysmon?

Sysmon (System Monitor) is een Sysinternals/Microsoft-tool voor Windows die als service gedetailleerde systeemtelemetrie (o.a. procescreatie, netwerkverbindingen, image loads, registry-/file-events, hashes) naar het Windows Event Log schrijft, zodat je betere detectie, threat hunting en forensics kunt doen met vaste Event IDs.

## Lab

https://documentation.wazuh.com/current/quickstart.html

Wazuh credentials:

User: admin
Password: BDnQpLEl5OZF+fuM57EJe+U+*ljNuAKa

https://192.168.62.253
