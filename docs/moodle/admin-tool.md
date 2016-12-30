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
Admin Tools gebraucht wird:

<div class="alert alert-danger">
  <strong>TODO:</strong> Grafik für Ordnerstruktur einfügen.
</div>

## Implementierung der vorgegebenen Schnittstelle

## Tests und CI

# Änderungen an Core Bibliotheken

## Anpassung des WebDAV Clients