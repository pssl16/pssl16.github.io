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

Um nun die Eingabeeinstellungen dem Admin Tree anzuhängen, muss jedes vorgesehene Eingabefeld der `admin_settingpage` 
mittels der Methode `add` hinzugefügt werden. Der dafür notwendige Code sieht wie folgt aus:

```php
<?php

$temp = new admin_settingpage('oauth2sciebo', new lang_string('pluginname', 'tool_oauth2sciebo'));
$temp->add(new admin_setting_heading('coursebank_proxy_head',
        get_string('configplugin', 'tool_oauth2sciebo'),
        ''
        ));
$temp->add(new admin_setting_configtext('tool_oauth2sciebo/clientid',
        get_string('clientid', 'tool_oauth2sciebo'),
        '', ''
        ));
$temp->add(new admin_setting_configtext('tool_oauth2sciebo/secret',
        get_string('secret', 'tool_oauth2sciebo'),
        '', ''
        ));
$temp->add(new admin_setting_configtext('tool_oauth2sciebo/server',
        get_string('server', 'tool_oauth2sciebo'),
        '', ''
    ));
$temp->add(new admin_setting_configtext('tool_oauth2sciebo/path',
        get_string('path', 'tool_oauth2sciebo'),
        '', ''
    ));
$temp->add(new admin_setting_configtext('tool_oauth2sciebo/port',
        get_string('port', 'tool_oauth2sciebo'),
        '', ''
    ));
$temp->add(new admin_setting_configselect('tool_oauth2sciebo/auth',
        get_string('auth', 'tool_oauth2sciebo'),
        '', '', array('basic' => 'Basic', 'bearer' => 'Bearer')
    ));
    
$ADMIN->add('authsettings', $temp);
```

Hierbei sind die Klassen `admin_setting_heading`, `admin_configtext` und `admin_configselect` ebenfalls Teil der `adminlib.php`
und die Überschrift der Einstellungen, eine Eingabetextfeld und eine Eingabeauswahl. Beim Erstellen eines Objektes der Klassen 
`admin_configtext` und `admin_configselect` müssen neben dem einzigartigen Einstellungsnamen Anzeigename, Beschreibung und Standartwert
übergeben werden, wobei bei letzterem die Auswahlmöglichkeiten angegeben werden müssen.

Sobald alle nötigen Optionen erstellt worden und der `admin_settingpage` hinzugefügt worden sind, muss diese dem Globalen
`ADMIN` Objekt hinzugefügt werden. Dabei wird auch die Stelle übergeben, an der die Einstellungen in moodle gefunden werden
können. Im Fall des hier implementierten Admin Tools, werden die Einstellungen unter den Authentifizierungs-Optionen
gelistet. Damit ist die Erstellung der Eingabemaske abgeschlossen.

### OAuth 2.0 Client

Den funktionalen Kern des Plugins stellt der OAuth 2.0 Client dar. Dieser befindet sich in Form der Klasse `sciebo` in der
Datei `sciebo.php` in dem `classes` Ordner des Plugins. Diese Klasse steuert sowohl den moodle-seitigen Protokollablauf
von OAuth 2.0, als auch den Verbindungsaufbau zu ownCloud mittels WebDAV. Dadurch, dass `sciebo` von der im moodle Core
enthaltenen Klasse `oauth2_client` erbt, ist ein Großteil des Protokollablaufs bereits abgedeckt.
Der Konstruktor der Klasse `oauth2_client` muss mit den `Client ID` und `Secret` Daten aufgerufen werden. 
Diese werden aus den zuvor angewandten Einstellungen beschafft:

```php
<?php

namespace tool_oauth2sciebo;

defined('MOODLE_INTERNAL') || die();

require_once($CFG->libdir . '/oauthlib.php');

use tool_oauth2sciebo\sciebo_client;

class sciebo extends \oauth2_client {

    /**
     * Create the DropBox API Client.
     *
     * @param   string      $key        The API key
     * @param   string      $secret     The API secret
     * @param   string      $callback   The callback URL
     */
    public function __construct($callback) {
        parent::__construct(get_config('tool_oauth2sciebo', 'clientid'),
            get_config('tool_oauth2sciebo', 'secret'), $callback, '');
```

Zu diesem Zweck wird die Methode `get_config` verwendet. Sie gibt den für ein Plugin und einen zuvor einzigartig definierten
Namen aus dem Admin Tree heraus die dazu gespeicherte Einstellung.
Darüber hinaus muss eine `callback URL` angefügt werden, die den Pfad angibt, an den nach der Authentifizierung und Autorisierung
weitergeleitet werden soll. Dieser wird allerdings wird extern in den Plugins erzeugt, die die `sciebo` Klasse benutzen.

Weiterhin müssen die Methoden `auth_url` und `token_url` der Elternklasse zwingend überschrieben werden, um bei der Authentifizierung
auf die richtigen Pfade zu verweisen:

```php
    /**
     * Returns the auth url for OAuth 2.0 request
     * @return string the auth url
     */
    protected function auth_url() {
        // Dynamically generated from the admin tool settings.
        return get_config('tool_oauth2sciebo', 'auth_url');
    }

    /**
     * Returns the token url for OAuth 2.0 request
     * @return string the auth url
     */
    protected function token_url() {
        return get_config('tool_oauth2sciebo', 'token_url');
    }
```

<div class="alert alert-danger">
  <strong>TODO:</strong> Später wird der Pfad aus den gegebenen Daten berechnet.
</div>

Hierfür werden die beiden Pfade aus der Serveraddresse und dem Serverpfad berechnet, da der Endpunkt für die oauth2 App in
ownCloud gleich bleibt.

<div class="alert alert-danger">
  <strong>TODO:</strong> Überschriebene post Methode.
</div>

## Tests und CI

# Änderungen an Core Bibliotheken

## Anpassung des WebDAV Clients