# KN02: Cloud-Onboarding und SSH

## A: AWS Academy Learner Lab

Das Learner Lab wurde gestartet; die Anzeige zeigt den grünen Status und das Budget. In der AWS-Konsole wurde EC2 in der Region Nord-Virginia (`us-east-1`) geöffnet.

![Gestartetes Learner Lab und Budget](img/Screenshot%202026-11-23%20104738.png)
![EC2-Konsole mit Benutzername](img/Screenshot%202026-11-23%20105022.png)

## B: SSH-Schlüssel-Experiment

Die erste Instanz `KN02` läuft mit Ubuntu 26.04.1 LTS und dem Typ `t3.micro` (2 vCPUs, 1 GiB RAM). Der beim Start ausgewählte Schlüssel heisst `lionel1`. Das Login zeigt 6.61 GB nutzbaren Speicher auf dem Root-Dateisystem; die konfigurierte EBS-Volumengrösse ist auf den vorhandenen Screenshots nicht ersichtlich.

![Zugewiesenes Schlüsselpaar lionel1](img/Screenshot%202026-11-23%20105848.png)
![Laufende Instanz, Typ und öffentliche IP](img/Screenshot%202026-11-23%20105939.png)

Die privaten Schlüssel `lionel1.pem` und `lionel2.pem` bleiben lokal im persönlichen `.ssh`-Ordner. Nur der öffentliche Schlüssel wird auf dem Server hinterlegt. Windows-Dateiberechtigungen wurden eingeschränkt, damit OpenSSH die privaten Schlüssel akzeptiert.

Der Login als `ubuntu` mit `lionel1` funktioniert:

![SSH-Login mit lionel1 erfolgreich](img/Screenshot%202026-11-23%20110533.png)

Mit `lionel2` wird der Login abgelehnt:

![SSH-Login mit lionel2 abgelehnt](img/Screenshot%202026-11-23%20110820.png)

Der öffentliche Schlüssel wurde mit `ssh-keygen -y` aus dem privaten Schlüssel abgeleitet. Der Vergleich zeigt denselben Schlüssel in `lionel1.pub` und in `~/.ssh/authorized_keys` des Ubuntu-Benutzers. AWS hat den zu `lionel1` gehörenden öffentlichen Schlüssel hinterlegt, weil dieses Schlüsselpaar beim Start der Instanz ausgewählt wurde.

![Extrahierter öffentlicher Schlüssel](img/Screenshot%202026-11-23%20111034.png)
![Authorized keys auf dem Server](img/Screenshot%202026-11-23%20111021.png)

## C: cloud-init und persönliches Template

Die zweite Instanz `KN02-cloud-init` wurde mit Ubuntu 26.04, `t3.micro`, dem AWS-Schlüsselpaar `lionel2` und der [AWS-Version der cloud-init-Datei](cloud-init-kn01.yaml) als User Data erstellt. Cloud-init hinterlegt den öffentlichen Schlüssel von `lionel1` beim zusätzlichen Benutzer `lionel`.

![Zweite Instanz und öffentliche IP](img/Screenshot%202026-11-23%20111901.png)
![Zugewiesenes Schlüsselpaar lionel2](img/Screenshot%202026-11-23%20111944.png)

Die SSH-Tests zeigen die getrennten Zugänge:

| Benutzer | Schlüssel | Ergebnis |
| --- | --- | --- |
| `ubuntu` | `lionel2` | erfolgreich |
| `lionel` | `lionel1` | erfolgreich |
| `lionel` | `lionel2` | abgelehnt |

![Ubuntu-Login mit lionel2](img/Screenshot%202026-11-23%20112104.png)
![Lionel-Login mit lionel1](img/Screenshot%202026-11-23%20112256.png)
![Lionel-Login mit lionel2 abgelehnt](img/Screenshot%202026-11-23%20112409.png)

Die Info-Datei aus KN01 ist auch auf der AWS-Instanz vorhanden. Das cloud-init-Log wurde mit `sudo` ausgelesen.

![Nachweis der Info-Datei](img/Screenshot%202026-11-23%20112718.png)
![Anfang des cloud-init-Logs](img/Screenshot%202026-11-23%20112816.png)

### Persönliches Template

Das [cloud-init-Template](cloud-init-template.yaml) enthält meinen öffentlichen Schlüssel und den öffentlichen Schlüssel von Van Essen für den Benutzer `lionel`. `ssh_pwauth: false` deaktiviert die SSH-Passwortanmeldung, `disable_root: true` verhindert die direkte Root-Anmeldung und `package_update: true` aktualisiert beim Einrichten die Paketlisten.

### Reflexion

Die cloud-init-Konfiguration kann lokal mit Multipass und auf einer AWS-Instanz verwendet werden. Die Datei beschreibt den gewünschten Zustand unabhängig von den einzelnen Klicks in der Oberfläche. Dadurch kann ich die Einrichtung wiederholen und Änderungen in Git nachvollziehen. Bei einer reinen GUI-Einrichtung gibt es diese übertragbare Konfigurationsdatei nicht; die Einstellungen müssten erneut von Hand gesetzt werden. Die verwendete Umgebung und das verwendete Image müssen cloud-init unterstützen.
