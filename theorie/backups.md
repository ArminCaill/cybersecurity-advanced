# Les 7: Backups

## What different "rules" exist when talking about a backup strategy?

- 3-2-1-regel: houd 3 kopieën van je data, op 2 verschillende media/technologieën, met 1 kopie off-site voor rampen.
- 3-2-1-1-0: als 3-2-1, plus 1 immutabele of offline (air-gapped) kopie en 0 fouten dankzij regelmatige verificatie/restore-tests.
- 3-2-1-1: variant van 3-2-1 met expliciet één immutabele of offline kopie tegen ransomware.
- 4-3-2-regel (strenger): 4 kopieën, 3 media, 2 off-site voor zeer kritieke data/regelgevingen.
- GFS-rotatie (Grandfather-Father-Son): dagelijks/wekelijkse/maandelijkse back-ups met langere retentie voor “vaders/opa’s” om historiek te bewaren.
- 3-2-1-0/“zero-error” principe: naast het kopie-model is er de regelmatige test-restore en checksum-controle zodat je back-ups daadwerkelijk herstelbaar zijn.

## What is the difference between a full vs an incremental backup?

- Full backup: een volledige kopie van alle data elke run.
Voordelen: snelste en eenvoudigste restore (één set), robuust tegen kettingfouten.
Nadelen: traag, groot opslag- en bandbreedteverbruik, lange backup-window.

- Incremental backup: alleen wijzigingen sinds de vorige backup (meestal de vorige incremental).
Voordelen: snelle, kleine backups; minder opslag en netwerk.
Nadelen: langzamere/complexere restore (laatste full + hele keten), kettingbreuk = dataverliesrisico, meer beheer om integriteit te bewaken.

## Extra questions

Why do some people state that "synchronisation with a cloud service (OneDrive, Dropbox, Google Drive) is not a synonym for backups"?
Synchronisatie houdt mappen gelijk: wijzigingen én fouten (verwijderingen, corruptie, ransomware-encryptie) worden meteen overal doorgezet, waardoor je geen gegarandeerd point-in-time herstel of immutabele kopie hebt. Cloudsync biedt vaak beperkte versiebehoud/retentie en beschermt niet tegen accountcompromis of grootschalige fouten. Daarom is het geen vervanging voor een echte back-up volgens 3-2-1(-1-0).

When using a cloud storage to store backups, like OneDrive, you can assume they provide backups and maybe have a backup strategy in place. Why do cyber security experts state it is still not enough to only store critical data on (this) one place? In other words why is putting 100% trust on a cloud provider a (potential) bad idea?
Omdat één cloudprovider nog steeds één plek is en dus een single point of failure blijft: jouw account kan worden gecompromitteerd of per ongeluk opgeschoond, ransomware of massadeleties synchroniseren mee, retentie/versies zijn beperkt of misgeconfigureerd, en je bent afhankelijk van provider-fouten, outages, beleid of account-suspensies. Daarom adviseren experts altijd een onafhankelijke tweede/derde kopie (3-2-1), bij voorkeur met immutability/offline en regelmatige restore-tests, in plaats van 100% te vertrouwen op één cloud.

## Lab

Dit commando downloadt 4 bestanden en slaat ze lokaal op met precies de bestandsnaam van de URL.

- curl --remote-name-all (--remote-name = -O) zegt: “schrijf de response naar een bestand met de remote naam, voor alle opgegeven URL’s.”
- De 4 URLs worden dus opgeslagen in je huidige map als:
  - bf1f3fb5-b119-4f9f-9930-8e20e892b898-720.mp4
  - 100.txt.utf-8
  - 996.txt.utf-8
  - Toreador_song_cleaned.ogg

https://borgbackup.readthedocs.io/en/stable/quickstart.html#a-step-by-step-example

Op web:

- sudo dnf install epel-release
- sudo dnf config-manager --set-enabled crb
- sudo dnf install borgbackup

Op db:

- sudo apk add borgbackup
- mkdir backups

Op web:

- borg init --encryption repokey vagrant@172.30.0.15:/home/vagrant/backups (ww = arminarmin)
- borg key export --paper vagrant@172.30.0.15:/home/vagrant/backups > encrypted-key-backup.txt
- borg create vagrant@172.30.0.15:backups::first /home/vagrant/important-files
- borg info vagrant@172.30.0.15:backups
- borg list vagrant@172.30.0.15:backups
- borg list vagrant@172.30.0.15:backups::first
- borg check -v vagrant@172.30.0.15:backups
- borg mount vagrant@172.30.0.15:backups::first /home/vagrant/restore
- borg umount /home/vagrant/restore
