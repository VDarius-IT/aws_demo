# AWS-Infrastrukturdokumentation: Demo-Webseite (VDarius.IT.com)

## Überblick

Dieses Repository dokumentiert die AWS-Cloud-Infrastruktur, die entworfen und implementiert wurde, um eine Demo-Webseite, VDarius.IT.com, zu hosten. Dieses Projekt dient als praktische Lernübung zur Bereitstellung von Webanwendungen auf AWS, wobei der Fokus auf der Nutzung des AWS Free Tier liegt und grundlegende Best Practices für Sicherheit, Netzwerk und Betrieb berücksichtigt werden.

Die Infrastruktur unterstützt eine Node.js-basierte Webanwendung, die ein relationales Datenbank-Backend benötigt. Wichtige Überlegungen während des Entwurfs und der Implementierung waren Kosteneffizienz, grundlegende Sicherheitsmaßnahmen und die Schaffung der Grundlage für zukünftige Erweiterungen wie Hochverfügbarkeit und robustere Überwachung.

## Architektur

Die Infrastruktur wird innerhalb der AWS-Region **EU Frankfurt (eu-central-1)** bereitgestellt, wobei für bestimmte Komponenten mehrere Verfügbarkeitszonen genutzt werden, um die Ausfallsicherheit zu erhöhen.

Ein Überblick über die Architektur auf hoher Ebene umfasst:

-   Eine **Virtual Private Cloud (VPC)**, die mit spezifischen Subnetzen und Routing konfiguriert ist.
-   Eine **EC2-Instanz**, die den Webserver (Nginx) und die Node.js-Anwendung hostet.
-   Eine **RDS MySQL-Instanz**, die den Datenbankdienst bereitstellt.
-   Mehrschichtige **Sicherheitskontrollen** einschließlich Sicherheitsgruppen und hostbasierter Firewalls.
-   **IAM** für sicheres Zugriffsmanagement.
-   (Teilweise implementiert) **Cognito** für Benutzerauthentifizierungsabläufe.

## Netzwerkkonfiguration

Die Netzwerkinfrastruktur basiert auf einer einzelnen VPC, die in mehrere Subnetze über verschiedene Verfügbarkeitszonen hinweg unterteilt ist, um logische Isolation zu bieten und auf Konfigurationen mit höherer Verfügbarkeit vorzubereiten.

### Virtual Private Cloud (VPC)

-   **Region**: eu-central-1 (Frankfurt)
-   **VPC CIDR-Block**: `/18` - Konfiguriert, um alle innerhalb der VPC verwendeten Subnetze zu umfassen und die interne Kommunikation zu erleichtern.
-   **Genutzte Verfügbarkeitszonen**: `eu-central-1a`, `eu-central-1b`, `eu-central-1c`

### Subnetzkonfiguration

Subnetze sind strategisch innerhalb der VPC und über AZs hinweg platziert:

| Subnetztyp            | CIDR-Block | Genutzte Verfügbarkeitszonen | Zweck                                                                  |
| :-------------------- | :--------- | :--------------------------- | :--------------------------------------------------------------------- |
| Öffentlich/Anwendung | `/22`      | Mehrere AZs                  | Hostet die EC2-Instanz und andere öffentlich zugängliche Ressourcen.   |
|                       |            |                              | Wird für Ressourcen verwendet, die direkten Internetzugang benötigen. |
| Datenbank (RDS) - AZ A | `/25`      | eu-central-1a                | Dediziertes Subnetz für die RDS-Instanzplatzierung (Teil einer DB Subnet |
|                       |            |                              | Group).                                                                |
| Datenbank (RDS) - AZ B | `/25`      | eu-central-1b                | Dediziertes Subnetz für die RDS-Instanzplatzierung (Teil einer DB Subnet |
|                       |            |                              | Group).                                                                |
| Datenbank (RDS) - AZ C | `/25`      | eu-central-1c                | Dediziertes Subnetz für die RDS-Instanzplatzierung (Teil einer DB Subnet |
|                       |            |                              | Group).                                                                |

*Hinweis: Die Datenbank-Subnetze sind kleinere `/25`-Blöcke, die aus dem größeren `/18` VPC CIDR herausgeschnitten wurden.*

### Routing

-   **Internet Gateway (IGW)**: An die VPC angehängt, um die Kommunikation zwischen Ressourcen in der VPC und dem öffentlichen Internet zu ermöglichen.
-   **Routing-Tabellen**:
    -   **Öffentliche Routing-Tabelle**: Dem öffentlichen/Anwendungs-Subnetz zugeordnet. Enthält eine Standardroute (`0.0.0.0/0`), die auf das Internet Gateway zeigt, wodurch Ressourcen in diesem Subnetz (wie die EC2-Instanz) auf das Internet zugreifen und von diesem aus erreichbar sind (vorbehaltlich der Regeln von Sicherheitsgruppen/NACLs).
    -   **Private Routing-Tabelle**: Den Datenbank-Subnetzen zugeordnet. Enthält nur Routen für die lokale VPC-Kommunikation, wodurch direkter Internetzugriff auf die Datenbankinstanzen verhindert wird.
-   **NAT Gateway**: Wurde während der Ersteinrichtung getestet, aber anschließend entfernt, da es für die aktuelle Architektur nicht erforderlich war (keine Instanzen in privaten Subnetzen benötigten von ihnen initiierten ausgehenden Internetzugriff).

## Sicherheitskonfiguration

Sicherheit wird in Schichten implementiert, vom Zugriff auf die AWS-Konsole bis hinunter zur Instanzebene.

### AWS-Konto- und Konsolenzugriff

-   **MFA-Schutz**: Multi-Faktor-Authentifizierung ist für alle Anmeldungen an der AWS-Konsole erzwungen.
-   **Passwortrichtlinie**: Starke Passwortanforderungen werden durchgesetzt.
-   **Überwachung verdächtiger Aktivitäten**: E-Mail-Benachrichtigungen sind konfiguriert, um bei erkannten verdächtigen Anmeldeaktivitäten zu warnen.

### Netzwerkzugriffskontrolle

-   **Sicherheitsgruppen (Zustandsbehaftete Firewall)**:
    -   **EC2-Sicherheitsgruppe**:
        -   Eingehend:
            -   HTTP (Port 80): Erlaubt von `0.0.0.0/0` (überall) - für öffentlichen Webzugriff.
            -   HTTPS (Port 443): Erlaubt von `0.0.0.0/0` (überall) - für öffentlichen Webzugriff.
            -   SSH (Port 22): Nur auf eine spezifische Administrator-IP-Adresse beschränkt - zur Sicherung des Verwaltungszugriffs.
        -   Ausgehend:
            -   Gesamter Datenverkehr: Erlaubt an `0.0.0.0/0` (überall) - für allgemeinen Internetzugriff (z. B. Updates).
            -   MySQL/Aurora (Port 3306): *Nur* an die **RDS-Sicherheitsgruppe** erlaubt - stellt sicher, dass die Anwendung eine Verbindung zur Datenbank herstellen kann, aber nicht zu anderen externen DBs.
    -   **RDS-Sicherheitsgruppe**:
        -   Eingehend:
            -   MySQL/Aurora (Port 3306): *Nur* von der **EC2-Sicherheitsgruppe** erlaubt - beschränkt den Datenbankzugriff ausschließlich auf den Anwendungsserver.
        -   Ausgehend: Eingeschränkt (erlaubt normalerweise ausgehenden Verkehr zu erforderlichen AWS-Diensten oder ist je nach Bedarf begrenzt).

-   **Netzwerk-Zugriffskontrolllisten (NACLs) (Zustandslose Firewall)**:
    -   Auf Subnetzebene konfiguriert.
    -   Regeln beinhalten:
        -   Regel 100 (Eingehend/Ausgehend): Erlaubt spezifischen Verkehr im Zusammenhang mit HTTP/HTTPS und ephemeren Ports für zurückkehrenden Webverkehr.
        -   Standardregeln (`*`): Verweigern explizit allen anderen Verkehr (dies ist das Standardverhalten für NACLs).
      *(Hinweis: NACLs arbeiten zustandslos, was bedeutet, dass sowohl eingehende als auch ausgehende Regeln den Verkehr in beide Richtungen explizit erlauben müssen, einschließlich ephemerer Ports für den Rückverkehr.)*

### Sicherheit auf Instanzebene

-   **SSH-Schlüsselauthentifizierung**: Passwortauthentifizierung für SSH ist deaktiviert; der Zugriff erfordert ein spezifisches öffentliches/privates Schlüsselpaar.
-   **UFW Firewall**: Uncomplicated Firewall ist auf der EC2-Instanz aktiviert und bietet eine zusätzliche Schicht hostbasierter Firewall-Schutz.
-   **Nginx-Konfiguration**: Der Nginx-Webserver ist sicherheitsbewusst konfiguriert und dient als Reverse-Proxy für die Node.js-Anwendung.

## Datenbankkonfiguration

Die Anwendung nutzt eine Amazon RDS für MySQL-Instanz.

-   **Datenbank-Engine**: MySQL (Free Tier-berechtigt)
-   **Bereitstellung**: Konfiguriert innerhalb einer **DB Subnet Group**, die sich über drei Verfügbarkeitszonen erstreckt (`eu-central-1a`, `eu-central-1b`, `eu-central-1c`). Dieses Setup ist eine Voraussetzung für Multi-AZ-Bereitstellungen und ermöglicht potenzielle zukünftige HA-Upgrades.
-   **Erreichbarkeit**: Die Instanz ist als "privat" konfiguriert (nicht öffentlich zugänglich), befindet sich in den dedizierten Datenbank-Subnetzen und ist nur von der EC2-Instanz über die konfigurierten Sicherheitsgruppenregeln erreichbar.
-   **Automatisierte Backups**: Tägliche automatisierte Backups sind aktiviert und bieten Point-in-Time-Recovery-Funktionen.

## Identity and Access Management (IAM)

IAM wird verwendet, um den Zugriff auf AWS-Ressourcen zu verwalten.

-   **Admin-Benutzergruppe**: Eine dedizierte Gruppe für Administratoren (den Hauptbenutzer) mit Richtlinien, die die erforderlichen Berechtigungen zur Verwaltung der Infrastruktur gewähren.
-   **Prinzip der geringsten Rechte**: Es wird darauf geachtet, nur die minimal notwendigen Berechtigungen über benutzerdefinierte oder verwaltete Richtlinien zuzuweisen.
-   **Cognito-Integration**: Es wurde versucht, AWS Cognito User Pools und Identity Pools zur Verwaltung der Benutzerregistrierung und -anmeldung für die Webseite zu integrieren. Diese Implementierung ist derzeit aufgrund technischer Herausforderungen bei der Token-Handhabung und unterschiedlichen APIs pausiert, stellt aber eine geplante Funktion für die sichere Benutzerauthentifizierung dar.

## Überwachung und Alarmierung

Grundlegende Überwachungs- und Alarmierungsmechanismen sind vorhanden, mit Plänen zur Erweiterung.

-   **Kostenwarnungen**: Eine Kostenwarnung ist auf 0 $ gesetzt und löst Benachrichtigungen aus, um unerwartete Kosten zu vermeiden und die Einhaltung der Free Tier-Limits sicherzustellen.
-   **Anmeldeüberwachung**: E-Mail-Benachrichtigungen sind für verdächtige Anmeldeversuche auf dem AWS-Konto konfiguriert.
-   **Grundlegende CloudWatch-Metriken**: AWS stellt standardmäßige CloudWatch-Metriken für EC2- und RDS-Instanzen bereit (z. B. CPU-Auslastung, Netzwerkverkehr, Datenbankverbindungen).
-   **CloudWatch-Alarme**: Festlegen von Schwellenwerten für Schlüsselmetriken (z. B. hohe CPU, geringer Festplattenspeicher, hoher Netzwerkverkehr, hohe Datenbankverbindungen), um Benachrichtigungen auszulösen.
-   **CloudWatch Logs**: Sammeln von Systemprotokollen (z. B. `syslog`), Anwendungsprotokollen (Node.js-Ausgabe) und Webserverprotokollen (Nginx Zugriffs-/Fehlerprotokolle) zur Analyse und Fehlerbehebung.
-   **CloudWatch Dashboards**: Erstellen visueller Zusammenfassungen von Schlüsselmetriken und Alarmen für eine schnelle Betriebsübersicht.
-   **Überwachung auf Anwendungsebene**: Implementierung von Protokollierung und Metriken innerhalb der Node.js-Anwendung selbst zur Verfolgung von Leistung und Fehlern.

## Hochverfügbarkeit und Notfallwiederherstellung

Dieses initiale Infrastrukturdesign priorisiert Kosteneffizienz und Lernen innerhalb der Einschränkungen des AWS Free Tier. Während diese aktuelle Konfiguration eine funktionale Umgebung für eine Demo-Webseite bietet, wurde sie mit dem Verständnis von branchenüblichen Prinzipien der Hochverfügbarkeit (HA) und Notfallwiederherstellung (DR) erstellt. Überlegungen zur Verbesserung der Ausfallsicherheit in der Zukunft sind dokumentiert, wobei die Kompromisse anerkannt werden, die in der aktuellen kostengünstigen Implementierung gemacht wurden.

-   **Aktuelles Design & Resilienzmerkmale**:
    -   Die aktuelle Anwendungsebene, die eine **einzelne EC2-Instanz** verwendet, stellt einen Single Point of Failure dar. Sollte diese Instanz ausfallen, würde die Webseite eine Ausfallzeit erleben.
    -   Die EC2-Instanz ist mit einer **dynamischen öffentlichen IP-Adresse** konfiguriert. Änderungen an dieser IP (z. B. beim Stoppen/Starten oder Ersetzen der Instanz) erfordern manuelle Aktualisierungen externer DNS-Einträge, was während des Aktualisierungsprozesses zu einer potenziellen Nichtverfügbarkeit führen kann.
    -   Die RDS-Datenbank ist in einer **Multi-AZ Subnet Group** platziert, was eine Voraussetzung für eine Multi-AZ-Bereitstellung ist. Unter den Einschränkungen des Free Tier ist sie jedoch derzeit als **Single-AZ-Instanz** konfiguriert. Dies bedeutet, dass ein automatisches Failover zu einer Standby-Replik in einer anderen Verfügbarkeitszone nicht aktiviert ist und die Wiederherstellung nach einem Ausfall der Datenbankinstanz eine Wiederherstellung aus Backups erfordert.
    -   Wiederherstellungsverfahren im aktuellen Setup basieren hauptsächlich auf der Wiederherstellung aus vorhandenen Backups (automatisiert von AWS für RDS, benutzerdefiniertes Skript für Anwendungsdateien) und der manuellen Neukonfiguration/Neustart von Komponenten, was manuelles Eingreifen und potenzielle Ausfallzeiten beinhaltet.

-   **Zukünftige Verbesserungen für Hochverfügbarkeit & Notfallwiederherstellung**:
    Um ein Verständnis für robuste Cloud-Architektur und Resilienzstrategien zu demonstrieren, umfasst die Roadmap zur Verbesserung dieser Infrastruktur:

    *   **Hochverfügbarkeit der Anwendungsebene (In-Region)**: Implementierung einer **Auto Scaling Group (ASG)**, die über mehrere Verfügbarkeitszonen (`eu-central-1a`, `eu-central-1b`, `eu-central-1c`) verteilt ist. Dies würde automatisch die gewünschte Anzahl von Instanzen aufrechterhalten und fehlerhafte ersetzen. Ein **Load Balancer (ALB)** würde vor der ASG implementiert, um eingehenden Verkehr zu verteilen und einen stabilen, hochverfügbaren Endpunkt-DNS-Namen bereitzustellen, wodurch manuelle DNS-Updates bei Instanzänderungen entfallen.
    *   **Hochverfügbarkeit der Datenbankebene (In-Region)**: Upgrade der RDS-Instanz auf eine **Multi-AZ-Bereitstellung**. Dies stellt automatisch eine synchrone Standby-Replik in einer anderen Verfügbarkeitszone bereit und verwaltet diese. Im Falle eines Ausfalls der primären Instanz oder eines AZ-Ausfalls übernimmt AWS automatisch das Failover auf die Standby-Instanz mit minimaler Wiederherstellungszeit. (Hinweis: Diese Verbesserung führt normalerweise dazu, dass die RDS-Instanz die Berechtigung für den Free Tier überschreitet).
    *   **Datenresilienz**: Überprüfung und Verbesserung der Datenpersistenzstrategien, um nicht nur Datenbank-Backups (bereits vorhanden) sicherzustellen, sondern auch gemeinsam genutzte Dateisysteme wie **Amazon EFS** zu berücksichtigen, falls die Anwendung einen gemeinsamen Zustand über mehrere potenzielle Instanzen in einer ASG-Konfiguration hinweg erfordert.
    *   **Notfallwiederherstellung (Regionenübergreifend)**: Für eine umfassendere Wiederherstellungsstrategie gegen regionale Ausfälle würde die Implementierung eines Dienstes wie **AWS Elastic Disaster Recovery (DRS)** in Betracht gezogen. DRS bietet eine kostengünstige Lösung für die Notfallwiederherstellung, indem es eine kontinuierliche Replikation von Instanzen auf Blockebene in einen kostengünstigen Staging-Bereich in einer anderen AWS-Region ermöglicht. Dies erlaubt eine schnelle und zuverlässige Wiederherstellung mit minimalem Datenverlust (niedrigem RPO) und schneller Wiederherstellung von Diensten (niedrigem RTO), indem der Prozess des Startens von Instanzen in der Wiederherstellungsregion unter Verwendung der replizierten Daten automatisiert wird. Obwohl dies für das aktuelle Demo-Projekt nicht notwendig ist und Kosten über den Free Tier hinaus verursacht (für Replikation und Staging), ist das Verständnis und die Planung einer robusten DR-Lösung wie DRS für produktionsreife Systeme entscheidend und stellt einen signifikanten Fortschritt gegenüber der alleinigen Abhängigkeit von Backups dar.

Die Implementierung dieser dokumentierten HA/DR-Strategien stellt einen Fortschritt hin zu einer widerstandsfähigeren und produktionsreifen Architektur dar und demonstriert die Fähigkeit, Systeme zu entwerfen, die verschiedenen Ausfallszenarien standhalten können.

## Bereitstellung und Betrieb

Verfahren zur Bereitstellung von Updates und zur Datenverwaltung sind vorhanden.

-   **Anwendungsbereitstellung**: Updates der Node.js-Anwendung und der Nginx-Konfiguration werden hauptsächlich über SSH-Zugriff auf die EC2-Instanz gehandhabt, wobei Tools wie Git verwendet werden, um den neuesten Code abzurufen.
-   **Datenbankmanagement**: Datenbankschema-Änderungen und Datenmanipulationen werden manuell über Standard-MySQL-Clients oder potenziell bei Bedarf über `.sh`-Skripte durchgeführt.
-   **Backup & Wiederherstellung**:
    -   RDS: Automatisierte tägliche Backups durch AWS. Die Wiederherstellung kann über die RDS-Konsole initiiert werden.
    -   Anwendung: Benutzerdefinierte `.sh`-Skripte sind auf der EC2-Instanz verfügbar, um Webseiten-Dateien und den Anwendungscode zu sichern, zusammen mit einem entsprechenden Skript zur Wiederherstellung aus diesen Backups, sowohl lokal als auch in meinem privaten GitHub-Repository.
-   **Wartung**: Patching des Betriebssystems und Software-Updates auf der EC2-Instanz werden manuell verwaltet. Das Patching der Datenbank-Engine wird typischerweise von AWS für RDS verwaltet, aber Wartungsfenster sollten überwacht werden.

## Kostenmanagement

Der Betrieb innerhalb der Grenzen des AWS Free Tier ist ein Hauptziel dieses Projekts.

-   **Nutzung des Free Tier**: Alle bereitgestellten Dienste (VPC, EC2 t2.micro, RDS db.t2.micro, Speicher, Datenübertragung innerhalb der Grenzen) werden so ausgewählt, dass sie unter die Berechtigung für den AWS Free Tier fallen.
-   **Kostenwarnung**: Ein Null-Dollar-Budget ist mit Benachrichtigungen eingerichtet, um sofortige Warnungen zu gewährleisten, falls die Kosten die Schwellenwerte des Free Tier überschreiten.
-   **Regelmäßige Überwachung**: Die regelmäßige Überprüfung des AWS Cost Explorer-Dashboards hilft zu bestätigen, dass die Nutzung innerhalb der Grenzen bleibt.

## Zukünftige Erweiterungen

Basierend auf der aktuellen Implementierung und den Lernzielen umfassen potenzielle zukünftige Erweiterungen:

-   Abschluss der AWS Cognito-Integration für eine robuste und skalierbare Benutzerauthentifizierung.
-   Erkundung von Infrastructure as Code (IaC) unter Verwendung von Tools wie AWS CloudFormation oder Terraform für eine automatisierte, wiederholbare Bereitstellung und Verwaltung der Infrastruktur.
-   Implementierung der besprochenen Hochverfügbarkeits- und Notfallwiederherstellungsstrategien, einschließlich Auto Scaling Groups, Load Balancers und potenziell RDS Multi-AZ (im Bewusstsein, dass dies über den Free Tier hinausgeht).
-   Einrichtung einer Continuous Integration/Continuous Deployment (CI/CD)-Pipeline für automatisierte Code-Bereitstellungen.
-   Weitere Verfeinerung der Sicherheitskonfigurationen.

## Fazit

Dieses Projekt demonstriert die Fähigkeit, eine grundlegende Infrastruktur für Webanwendungen auf AWS zu entwerfen, bereitzustellen und zu verwalten, wobei Free Tier-Ressourcen genutzt und grundlegende Cloud-Konzepte wie VPC-Netzwerk, Sicherheitsgruppen, verwaltete Datenbanken und Zugriffskontrolle integriert werden. Es dient als Grundlage zur Erkundung fortgeschrittenerer AWS-Dienste und Architekturmuster, insbesondere in den Bereichen Automatisierung, Überwachung und Hochverfügbarkeit.

---
