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
gespeichert werden können, um sie anschließend von anderen Plugins aus nutzen zu können. Zu diesem Zweck werden globale Optionen
werden, welche instanzübergreifend gelten.

#### Benötigte Eingaben

Um den OAuth 2.0 Protokollablauf zu ermöglichen, müssen folgende Daten im Vorfeld erfasst werden:

* **`Client ID`:** wird in ownCloud generiert und dient der Identifizierung eines regstrierten Clients.
* **`Secret`:** wird ebenfalls in ownCloud generiert und zur Authentifizierung verwendet.

Beide Datensätze sind Strings bestehend aus Buchstaben und Zahlen. Daher eignet sich für beide ein Textfeld, welches ausschließlich 
alphanumerische Werte erwartet zur Eingabe.

Zur Nutzung des WebDAV Clients werden darüber hinaus folgende Daten benötigt:

* **`Server Addresse`:** Url über die der ownCloud Server erreicht werden kann.
* **`Server Pfad`:** der angehangene Pfad, über den die WebDAV Schnittstelle erreicht werden kann.
* **`Port`:** Port des WebDAV-Servers.
* **`SSL-Verschlüsselung`:** Wahl zwischen HTTP und HTTPS. 

Während die Wahl des Protokolls mittels einer Auswahl aus vorhandenen Optionen abgeboten werden kann, müssen die restlichen Werte
in einem Textfeld erfragt werden. Auch in diesem Fall werden die Variablen nach den zu erwartenden Werten gesäubert. Darüber hinaus
werden alle Eingaben, bis auf dem Port, als notwendig angesehen.

#### External Page 

Eine somit notwendige Eingabemaske kann im Rahmen der [`settings.php`] definiert werden. Da zum Zweck der individuellen Validierung und Säuberung
der benötigten Parameter die in der [`adminlib.php`](https://github.com/moodle/moodle/blob/master/lib/adminlib.php) definierten 
Klassen nicht ausreichen, wurde eine [externe Seite](https://docs.moodle.org/dev/Admin_settings#External_pages) speziell zur Darstellung
des notwendigen Formulars erstellt. Die externe Seite, welche über die [`index.php`]() Datei aufgerufen wird, sorgt dafür, dass
die Einstellungen aus dem Formular global in den [admin settings]() gespeichert werden. Von dort aus können sie, sobald nötig ausgelesen
werden. Um die externe Seite nun in die Navigation in der Seitenadministration einzubinden, muss diese in der `settings.php` in den 
Admin Tree eingebunden werden: 

```php
<?php
defined('MOODLE_INTERNAL') || die('moodle_internal not defined');

$ADMIN->add('authsettings', new admin_externalpage('tool_oauth2sciebo/auth',
        'Sciebo OAuth 2.0 Configuration',
        "$CFG->wwwroot/$CFG->admin/tool/oauth2sciebo/index.php"));
```

Das `admin_externalpage` Objekt beschreibt eine externe Seite, die im Admin Tree eingeordnet werden soll. Dazu wird die Seite mit einem
einzigartigen Namen versehen, einem Anzeigenamen und dem Pfad, über den die Seite erreicht werden soll. Neben der externen Seite an
sich, wird bei der Methode `add` zusätzlich übergeben, an welcher Stelle die Verknüpfung erstellt werden soll. In diesem Fall sind es die
Authentifizierungseinstellungen (`authsettings`).

Neben der Darstellung des Formulars, verwaltet die externe Seite auch die Speicherung der eigegebenen Daten. Diese werden, ähnlich wie die
externe Seite zuvor, global in den Admin Settings mit Hilfe der Methode `set_config()` gespeichert. Sobald also das Formular erfolgreich validiert
worden ist, werden die Eingabedaten durch einen Aufruf dieser Methode mit der genauen Bezeichnung der Option und dem Wert, den sie im Fomular
erhalten hat, global abgelegt. Darüber hinaus wird über die externe Seite auch die Rücksetzung der Daten und der Abbruch der Bearbeitung geregelt.
Zuletzt ist die Seite auch dafür zuständig das Formular mit zuvor gesetzten Werten zu füllen, die aus den globalen Einstellungen wiederbeschafft werden.

#### Formular

Um ein geeignetes Formular zu definieren musste die moodle-interne Klasse `moodleform` erweitert und innerhalb der Funktion `definition()`
alle benötigten Eingabefelder definiert werden. Folgende Funktionen wurden dabei verwedet um die Elemente so genau wie möglich zu umreißen:

| Funktion     | Beschreibung                                         | Beispiel                                        |
|--------------|------------------------------------------------------|-------------------------------------------------|
| `addElement` | Fügt ein Element zum Formular hinzu                  | Textfeld, Dropdown Menü, Checkbox               |
| `addRule`    | Versieht ein Element mit einer Regel zur Validierung | erforderlich für die Abgabe, nur alphanumerisch |
| `setDefault` | Setzt den Standartwert für ein Element               | -                                               |
| `setType`    | Legt den Parametertypen der Eingabe fest             | Integer, String, Pfad, roh                      |

Im Folgenden wird anhand von zwei Beispielen die Anwendung dieser Funktionen dargestellt:

```php
<?php
class tool_oauth2sciebo_client_form extends moodleform {

    public function definition() {
        global $CFG;

        $mform = $this->_form;
        // Client ID.
        $mform->addElement('text', 'clientid', get_string('clientid', 'tool_oauth2sciebo'), 
            array('size' => '64'));
        $mform->addRule('clientid', get_string('required'), 'required', null, 'client');
        $mform->addRule('clientid', get_string('err_alphanumeric'), 'alphanumeric', null, 'client');
        $mform->setDefault('clientid', $this->_customdata['clientid']);
        $mform->setType('clientid', PARAM_ALPHANUM);
```
Zunächst wird das Client ID Eingabefeld definiert. Hierzu wird zum Formular ein Textfeld hinzugefügt, welches den eindeutigen Namen
`clientid` trägt und 64 Felder breit ist. Der Anzeigename des Elements wird über ein die Sprachstring-Methode `get_string` beschafft.
Daraufhin wird das Feld als für die Abgabe benötigt markiert und auf alphanumerische Werte beschränkt. Zuletzt wird der Standartwert
für das Textfeld gesetzt, welcher zuvor durch die externe Seite bei Aufruf übergeben wird und zur Säuberung der Eingabe der Typ des
Elements auf alphanumerisch gestellt.

```php
        // Type of server.
        $mform->addElement('select', 'type', get_string('type', 'tool_oauth2sciebo'), 
            array('http' => 'HTTP', 'https' => 'HTTPS'));
        $mform->addRule('type', get_string('required'), 'required', null, 'client');
        $mform->setDefault('type', $this->_customdata['type']);
    }
}
```
Im zweiten Beispiel wird ein `select` Element zum Formular hinzugefügt. Der Unterschied zum Textfeld ist, dass bei einem Dropdown
Menü auch verfügbare Optionen angegeben werden müssen. Außerdem wird auch dieses Element als benötigt markiert und sein Standartwert
Aus den Aufrufparametern beschafft und gesetzt.

Am Ende des Formulares werden zu guter Letzt noch Buttons zur Abgabe des Formulars definiert. Damit ist die Eingabemaske vollständig. 

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
Darüber hinaus muss eine `callback URL` angefügt werden, die den Pfad angibt, an den nach der Authentifizierung und Authorisierung
weitergeleitet werden soll. Dieser wird allerdings wird extern in den Plugins erzeugt, die die `sciebo` Klasse benutzen.

Zu beachten ist, dass für die Klasse `sciebo` ein namespace definiert wird, womit diese effizient in externen Plugins verwendet werden
kann, die einen OAuth 2.0 Client benötigen.

Weiterhin müssen die Methoden `auth_url` und `token_url` der Elternklasse zwingend implementiert werden, um bei der Authentifizierung
auf die richtigen Pfade zu verweisen:

```php
    /**
    * Returns the auth url for OAuth 2.0 request
    * @return string the auth url
    */
    protected function auth_url() {
    // Dynamically generated from the admin tool settings.
        $path = str_replace('remote.php/webdav/', '', get_config('tool_oauth2sciebo', 'path'));
        return get_config('tool_oauth2sciebo', 'type') . '://' . get_config('tool_oauth2sciebo', 'server') . '/' . $path
            . 'index.php/apps/oauth2/authorize';
    }
    
    /**
    * Returns the token url for OAuth 2.0 request
    * @return string the token url
    */
    protected function token_url() {
        $path = str_replace('remote.php/webdav/', '', get_config('tool_oauth2sciebo', 'path'));
        return get_config('tool_oauth2sciebo', 'type') . '://' . get_config('tool_oauth2sciebo', 'server')  . '/' . $path
            . 'index.php/apps/oauth2/api/v1/token';
    }
```

Hierfür werden die beiden Pfade aus der Serveraddresse und dem Serverpfad berechnet, da der Endpunkt für die oauth2 App in
ownCloud gleich bleibt.

## Änderungen an Core Bibliotheken

Du zur Umsetzung des Verfahrens Die vorgegebenen Schnittstellen nicht ausreichten, mussten in Anpassungen in moodles Core 
Bibliotheken vorgenommen werden. Im Folgenden werden diese Änderungen beschrieben.

### Anpassung der `post` Methode

Die moodle-interne Klasse `oauth2client` erbt von einer weiteren Klasse aus dem moodle Core mit dem Namen `curl`, welche mittels
[Curl]() HTTP Requests erstellen und verschicken kann. Dadurch ist die Klasse fähig eigenständig einen Access Token mit einem 
Authorization Code mittels der HTTP POST Methode über die `token` Schnittstelle in ownCloud zu beschaffen. Allerdings
bietet die dafür zuständige Methode `post` nicht die Möglichkeit einen Basic Authorization Header zur Anfrage hinzuzufügen,
welcher Client ID und Secret zur Autorisierung in der `oauth2` ownCloud App mit verschickt. Daher musste die `post` Methode
in der `sciebo` Klasse so überschrieben werden, dass der Header vor dem Aufruf der geerbten Methode gesetzt wird.
In dem zugehörigen Skipt wurde folende Methode ergänzt:

```php
public function post($url, $params = '', $options = array()) {
    
    $this->setHeader(array(
        'Authorization: Basic ' . base64_encode($this->get_clientid() . ':' . $this->get_clientsecret())
        ));
        
    return parent::post($url, $params, $options);
}
```

### Anpassung des WebDAV Clients

### Weiterleitungen

## Tests und CI