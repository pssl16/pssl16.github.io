# Admin Tool: `oauth2sciebo`

## Zweck des Plugins

Wie bereits im Kapitel [Software Architektur](software-architektur/) angeschnitten, ist der Hauptzweck dieses Plugins
die Schnittstelle zu Sciebo bzw. ownCloud bereitzustellen. Zu diesem Zweck wird die im Projekt implementierte ownCloud 
App [OAuth2](../owncloud/technische-umsetzung/) mit Hilfe eines OAuth 2.0 Clients über die WebDAV Schnittstelle angesprochen.
Gleichzeitig kann dieses Plugin auch für ähnliche externe Datenquellen verwendet werden, sofern diese über die nötigen
OAuth 2.0 und WebDAV Schnittstellen verfügen.

Im Wesentlichen implementiert dieses Plugin folgende [Integrationsszenarien](software-architektur/):

* **Beispiel 1:** ...
* **Beispiel 2:** ...

<div class="alert alert-danger">
  <strong>TODO:</strong> Integrationsszenarien definieren und hier einfügen.
</div>

## Vorgegebene Schnittstelle

Für Admin Tools ist in moodle lediglich eine schwach definierte Schnittstelle gegeben. Wie in jedem anderen moodle Plugin 
auch, müssen zunächst einige Standartdateien implementiert werden: 

* **`version.php`:** Beschreibt die Versionsnummer des Plugins, die benötigte moodle Version und Abhängigkeiten des Plugins.
* **`access.php`:** Legt die Berechtigungen für definierte Aktionen innerhalb des Plugins anhand von Nutzerrollen fest.
* **`tool_oauth2sciebo.php`:** Beinhaltet Sprachstrings für unterschiedliche Regionen und Sprachen, sodass definierte Strings,
abhängig von der jeweiligen Sprache, dynamisch angezeigt werden können.

Zusätzlich zu den allgemeinen Plugindateien, sollte unser Admin Tool auch mindestens noch eine Datei namens `settings.php`
beinhalten. Diese umfasst alle Einstellungen, die für das Admin Tool geltend dem Administrator der moodle Instanz zur 
Verfügung gestellt werden sollen. Nach der Eingabe, wird diese Konfiguration moodle-intern in dem sogenannten [Admin Tree](https://docs.moodle.org/dev/Admin_settings)
gespeichert. Aus dieser Baumstruktur können anschließend benötigte Einstellungen beschafft werden.

Insgesamt ergibt sich folgende Struktur von Ordnern und Dateien, die mindestens für die Implementierung des von uns gebrauchten
Admin Tools notwendig ist:

<div class="alert alert-danger">
  <strong>TODO:</strong> Grafik für Ordnerstruktur einfügen.
</div>

## Implementierung der vorgegebenen Schnittstelle

### Eingabemaske

Um die OAuth 2.0 und WebDAV Clients erfolgreich zum Zugriff auf eine entsprechende Sciebo bzw. ownCloud Instanz zu befähigen,
müssen diese zunächst mit Hilfe benötigter Eingabedaten konfiguriert werden. Diese sollen zentral im Admin Tool eingegeben und
gespeichert werden können, um sie anschließend von anderen Plugins aus nutzen zu können. Dies ist einer der Gegensätze zu
ähnlichen, vor Allem repository Plugins, welche die benötigte Daten auf Plugin-Ebene abfragen und benutzen.

Eine solche Eingabemaske kann im Rahmen der `settings` definiert werden. Zu diesem Zweck muss zunächst ein neues Objekt vom Typ `admin_settingpage`
innerhalb der `settings.php` Datei erstellt werden. Dieses Objekt umfasst eine Gruppe von Einstellungen, welche, sobald hinzugefügt,
in dem Admin Tree eingeordnet und gespeichert werden. Die zugehörige Klasse befindet sich in Funktionsbibliothek
[`adminlib.php`](https://github.com/moodle/moodle/blob/master/lib/adminlib.php), welche Teil des moodle Cores ist. Beim 
Aufruf des Konstruktors müssen Name des Plugins, welcher später dazu verwendet wird die Einstellung im Admin Tree wiederzufinden,
und der Anzeigename für die Einstellungsseite übergeben werden. 

Um den OAuth 2.0 Protokollablauf zu ermöglichen, müssen folgende Daten im Vorfeld erfasst werden:

* **`Client ID`:** wird in ownCloud generiert und dient der Identifizierung eines regstrierten Clients.
* **`Secret`:** wird ebenfalls in ownCloud generiert und zur Authentifizierung verwendet.

Beide Datensätze sind Strings und daher eignet sich für beide ein Textfeld zur Eingabe.

Zur Nutzung des WebDAV Clients werden darüber hinaus folgende Daten benötigt:

* **`Server Addresse`:** Url über die der ownCloud Server erreicht werden kann.
* **`Server Pfad`:** der angehangene Pfad, über den die WebDAV Schnittstelle erreicht werden kann.
* **`Port`:** Port des WebDAV-Servers.
* **`SSL-Verschlüsselung`:** Wahl zwischen HTTP und HTTPS.
* **`Authentifizierung`:** Wahl zwischen Basic und Bearer Authentifizierung.  

Während Server Adresse, Pfad und Port mittels eines Textfeldes abgefragt werden können, sollten die anderen Optionen mit Hilfe
einer Auswahl aus den angebotenen Möglicheiten erfragt werden können.



## Tests und CI

# Änderungen an Core Bibliotheken

## Anpassung des WebDAV Clients