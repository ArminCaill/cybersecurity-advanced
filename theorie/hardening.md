# Les 6: Hardening

## What is meant by the term hardening?

Hardening is het doelbewust verkleinen van het aanvalsoppervlak: onnodige diensten uit, strenge configuraties, least-privilege, patches, logging/monitoring en versleuteling op orde.

Waarom zijn defaults vaak niet oké?
Standaardinstellingen zijn gemaakt voor gemak en compatibiliteit, niet voor jouw risicoprofiel—ze laten vaak te veel features/poorten open, geven te brede rechten, gebruiken zwakkere policies/ciphers of voorbeeldaccounts, en lekken onnodige informatie.

Explain short, no details, what the following things are and why they exist:

- CIS Benchmark: consensus-gebaseerde hardeningrichtlijnen van het Center for Internet Security om systemen veilig te configureren en compliance te ondersteunen. Ze bestaan om een uniforme, bewezen baseline te geven per platform.
- OpenSCAP: open-source implementatie van NIST’s SCAP-standaarden (OVAL/XCCDF) om systemen automatisch te scannen op configuratie-compliance en kwetsbaarheden. Het bestaat om security-audits en rapportage te standaardiseren en te automatiseren.
- Lynis: open-source audit/hardening-tool voor Unix/Linux die je systeem doorlicht en concrete verbeterpunten geeft. Het bestaat om snel zwakke instellingen te vinden en te versterken.
- Microsoft Security Compliance Toolkit: Microsoft’s set met security-baselines, templates en tools (o.a. Policy Analyzer, LGPO) om Windows/Edge-beveiligingsinstellingen te testen en uit te rollen. Het bestaat om Microsoft-aanbevolen baselines praktisch toepasbaar te maken
- Docker Bench: script dat je Docker-host en -containers vergelijkt met de CIS Docker-benchmark om misconfiguraties te vinden. Het bestaat om productie-Docker omgevingen snel en automatisch te toetsen.
- Maester: PowerShell-gebaseerd framework voor geautomatiseerde security-tests van Microsoft 365/Entra-tenants (incl. integratie met DevOps) om configuraties continu te valideren. Het bestaat om posture-gaps vroeg te detecteren en te remediëren
- dev-sec.io: community-framework met InSpec-baselines en hardening-rollen (bijv. Linux/SSH/Nginx) om best-practices als code toe te passen. Het bestaat om hardening reproduceerbaar en automatiseerbaar te maken

## What is meant by threat hunting

Threat hunting is het proactief en gestuurd zoeken in je omgeving (logs, endpoint- en netwerktelemetrie) naar verdachte signalen die aan traditionele detectie ontsnappen. Het doel is vroegtijdig aanvallen te vinden, hun scope te bepalen, en je detectieregels en defenses te verbeteren.

## What is meant by threat modeling

Threat modeling is het gestructureerd in kaart brengen van wat je wilt beschermen (assets), hoe het systeem werkt (dataflows/trust boundaries), welke dreigingen en zwaktes er bestaan en welke maatregelen prioriteit hebben. Het doel is al vroeg tijdens het ontwerp en verandering gerichte risico-reductie te plannen in plaats van achteraf te reageren.

## Why do some people call Ansible a tool suitable for hardening but also for hunting?

Omdat Ansible configuratie als code uitrolt en afdwingt (idempotent), is het ideaal voor hardening (CIS-baselines, rechten, services, patches) én omdat het agentless op hele fleets snel feiten kan verzamelen en checks kan uitvoeren, kun je het ook voor hunting inzetten (IoC-scans, YARA/artefactverzameling, suspicious processen/users/poorten controleren en direct remediëren).

## Lab

sudo dnf install ansible
ansible --version

sudo useradd -m ansible
sudo passwd ansible (ansible)
su - ansible

ssh-copy-id vagrant@172.30.0.4/10/15/123

nano ansible/inventory.yml

ansible -i inventory.yml -m "ping" internal_lan
ansible-playbook -i inventory.yml create_user.yml
