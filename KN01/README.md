# KN01: Virtualisierung und automatisierte VM

## Umgebung

- Host: Windows Enterprise, Intel Core Ultra 7 268V
- 8 logische Prozessoren, 33'895'956'480 Bytes RAM (ca. 31.6 GiB)
- Multipass auf Windows; die VM meldet Microsoft Hyper-V als Hypervisor.
- Multipass-Backend: HCS, gemäss der gewählten Installation. Die genaue Version wurde nicht festgehalten.
- Repository: [github.com/lionel0021/m346](https://github.com/lionel0021/m346).

## A: Multipass-Test

Die VM `smoke` wurde mit einer CPU, 1 GB RAM, 5 GB Disk und Ubuntu 26.04 gestartet. `multipass list` zeigt sie als laufend; `lsb_release -a` zeigt Ubuntu 26.04.1 LTS.

![Test-VM und Ubuntu-Version](img/Screenshot%202026-11-23%20084732.png)

Der Screenshot zeigt nicht die Multipass-Version, das Backend oder die Windows-Edition.

## B: Hypervisor-Experiment

Die Laptops der Gruppe hatten gleiche Ressourcen. Der Host hat acht logische Prozessoren und rund 31.6 GiB RAM:

![Host-Ressourcen](img/Screenshot%202026-11-23%20084935.png)

Die reguläre Test-VM sieht vier CPUs und etwa 3.8 GiB RAM:

![CPU der Test-VM](img/Screenshot%202026-11-23%20085038.png)
![Weitere CPU-Informationen](img/Screenshot%202026-11-23%20085051.png)
![RAM der Test-VM](img/Screenshot%202026-11-23%20085137.png)

Der Startversuch mit zwölf CPUs schlug mit einem allgemeinen Fehler fehl. Der Screenshot allein erklärt nicht, warum:

![Fehler bei zwölf CPUs](img/Screenshot%202026-11-23%20085232.png)

Die VM mit 40 GB zugewiesenem RAM startete und zeigte rund 39 GiB an. Zum Zeitpunkt der Aufnahme verwendete sie etwa 711 MiB; die Ausgabe beweist nicht, dass der gesamte zugewiesene Speicher gleichzeitig genutzt werden kann.

![RAM-Zuweisung](img/Screenshot%202026-11-23%20085935.png)

Virtuelle CPUs können Prozessorzeit in Zeitscheiben teilen. Die höhere CPU-Zuweisung wurde hier durch einen Startfehler begrenzt; der genaue Grund ist nicht belegt. Eine grössere RAM-Zuweisung erzeugt keinen zusätzlichen physischen Speicher. Bei Speicherdruck kann ein System bei entsprechender Einrichtung Speicher auslagern (Swap), was langsamer ist. In den gezeigten VMs war Gast-Swap deaktiviert. Wie sich der Host bei Speicherdruck verhält, wurde nicht getestet.

Vermutung: Hyper-V ist ein Typ-1-Hypervisor, der auf der Hardware-Virtualisierung aufsetzt; Windows wird in einer privilegierten Partition ausgeführt. Das CPU-Ergebnis allein beweist den Typ nicht. Die Backends der Gruppenmitglieder wurden nicht festgehalten.

## C1: Basis-Konfiguration

Mit der [Basisdatei](cloud-init-basis.yaml) wurden `fastfetch` und `tree` installiert und `/home/ubuntu/greeting.txt` erstellt. Die ersten beiden Zeilen stammen aus `write_files`; `runcmd` hängt die Abschlusszeile mit Datum an. Der Screenshot zeigt auch den Anfang des cloud-init-Logs.

![Basis-VM, Programme, Datei und Log](img/Screenshot%202026-11-23%20094207.png)

Hinweis: Der Auftrag nennt die Datei `/home/ubuntu/begruessung.txt`; in diesem Versuch wurde stattdessen `greeting.txt` verwendet.

## C2: Erweiterte Konfiguration

Die [kommentierte cloud-init-Datei](cloud-init-kn01.yaml) behält den Standardbenutzer bei und ergänzt `lionel`, `jq`, eine Info-Datei und einen eigenen Befehl. Der Screenshot zeigt die Benutzer- und sudo-Einstellungen, die installierte jq-Version sowie den Inhalt der beiden zusätzlichen Dateien:

![Nachweise der Erweiterungen](img/Screenshot%202026-11-23%20101310.png)

Der erfolgreiche Start der VM und die gezeigten Ergebnisse belegen, dass die Konfiguration bei diesem Versuch verarbeitet wurde. Eine separate YAML-Syntaxprüfung wurde nicht dokumentiert.

## D: Manuell und automatisiert

Für den Vergleich soll auf einer neuen VM ohne cloud-init derselbe Zustand von Hand eingerichtet werden. Danach werden beide VMs gelöscht und die automatisierte VM erneut erstellt. Gemessen wird jeweils die aktive Arbeitszeit ohne Wartezeit.

- Manuelle Einrichtung: geschätzt 20 Minuten aktive Arbeit.
- Automatisierte Einrichtung mit vorhandener YAML: geschätzt 2 Minuten aktive Arbeit.

Diese Werte sind Schätzungen; eine Zeitmessung wurde nicht dokumentiert.

Bei der manuellen Einrichtung können beispielsweise ein Paket oder eine Benutzerberechtigung vergessen werden. Auch Tippfehler in Dateinamen oder Befehlen sind möglich. Eine getestete cloud-init-Datei kann auf mehreren neuen VMs gleich ausgeführt werden. Für zehn VMs lässt sich die Konfiguration wiederverwenden, statt alle Schritte zehnmal einzugeben. Ein Fehler in der Konfiguration kann allerdings ebenfalls auf alle VMs übertragen werden. Cloud-init läuft normalerweise nur beim ersten Start; spätere Änderungen an der YAML-Datei erfordern für diesen Test eine neue VM.
