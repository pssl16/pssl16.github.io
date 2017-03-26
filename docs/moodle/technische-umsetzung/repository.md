<div><img alt="" align=right src="../../images/icon_repository.svg" width=20% position=right>
<h1> Repository: <span class=code>owncloud</span></h1>
</div>


## Zweck

Der Plugintyp [Repository](https://docs.moodle.org/dev/Repository_plugins) wird in Moodle unter anderem verwendet, um Nutzern die Möglichkeit zu schaffen Zugang zu Dateien aus externen Quellen zu erhalten. Das Repository Plugin `ownCloud` kann somit folgende [Integrationsszenarien] realisieren:

> Als **Nutzer** möchte ich in der Dateiauswahl in Moodle eine Datei aus meiner ownCloud Instanz **hochladen**.

> Als **Nutzer** möchte ich in der Dateiauswahl in Moodle eine Datei aus meiner ownCloud Instanz **verlinken**.

Das Plugin soll dem Seiten-Administrator in Moodle ermöglichen, das Repository zu aktivieren und zu deaktivieren. Die Authentifizierung der Nutzer und der direkte Zugriff auf ownCloud Schnittstellen erfolgen über das [Admin Tool](admin-tool/) und dessen OAuth 2.0 ownCloud Client. Nachdem die Authentifizierung erfolgt ist, sollen die Nutzer bequem zwischen ihren Dateien wählen können. Besonders im Vordergrund steht dabei, dass das Plugin möglichst einfach zu bedienen ist, um die Benutzerfreundlichkeit zu erhöhen.

## Vorgegebene Schnittstelle

Wie auch im [Admin Tool](admin-tool/#vorgegebene-schnittstelle) müssen zunächst einige Standardschnittstellen implementiert werden. Für Repository Plugins muss darüber hinaus noch eine Funktionsbibliothek hinzugefügt werden, welche die Schnittstelle zum Moodle Core darstellt und mit Hilfe dessen der [File Picker](https://docs.moodle.org/32/en/File_picker) bespielt werden kann. Die dafür benötigten Methoden werden in in einer Klasse erfasst, welche von der Moodle-internen Klasse `repository` erbt. Diese enthält bereits alle Schnittstellen-Methoden, welche von Respository-Plugins implementiert, überschrieben oder erweitert werden können um die Daten so aufbereiten zu können, dass sie Moodle-intern weiterverarbeitet werden können. 

Die Funktionsbibliothek basiert auf dem bereits in Moodle implementierten [WebDAV Repository Plugin](https://docs.moodle.org/32/de/WebDAV_Repository), da in ownCloud ebenfalls eine WebDAV Schnittstelle für den Datentransfer zur Verfügung gestellt wird. Zusätzlich mussten die Methoden auf das Zusammenspiel mit dem Admin Tool angepasst werden, da, statt direkter Anfragen an den WebDAV Client, bei Zugriffen auf ownCloud zum Admin Tool weitergeleitet werden muss.

## Implementierung

Die Funktionsbibliothek ist in der `lib.php` Datei implementiert. Da von der `repository` Klasse bereits alle Schnittstellen zum Moodle Core vorgegeben sind, werden im Folgenden die wichtigsten Funktionen der Bibliothek erläutert. Dabei liegt der Fokus vor allem auf der Kommunikation mit dem Admin Tool und der Aufbereitung der Daten.

### Konstruktor

#### Initialisierung des Clients

Die Hauptaufgabe des Konstruktors ist die Initialisierung eines OAuth 2.0 ownCloud Clients. Wie im Admin Tool [beschrieben](admin-tool/#oauth-20-client), benötigt der Client zur Initialisierung von der Anwendung, welche ihn benutzt, eine `callback URL`. Diese gibt an, wohin der Client weiterleiten soll, nachdem der Nutzer sich in ownCloud authentifiziert hat. Für Repository Plugins bietet Moodle bereits eine `callback URL` in der Datei `repository_callback.php` an. Im folgenden Codebeispiel wird dargestellt, wie der Client initialisiert werden muss.

``` php
$returnurl = new moodle_url('/repository/repository_callback.php', [
    'callback'  => 'yes',
    'repo_id'   => $repositoryid,
    'sesskey'   => sesskey(),
]);

$this->owncloud = new owncloud($returnurl);
```

Neben der ID des aktuellen Repositories, muss der URL auch ein `session key` angefügt werden, welcher zur Wiederherstellung der Sitzung in Moodle gebraucht wird.

#### Datenprüfung

Nachdem der Client initialisiert worden ist, muss geprüft werden, ob alle benötigten Clientdaten auf der Einstellungsseite des Admin Tools angegeben worden sind. Zu diesem Zweck wird eine im Client implementierte Funktion namens `check_data` verwendet. Sollten die Daten tatsächlich unvollständig sein, so wird bei Aufruf einer Aktivität, welche das Repository verwendet (zum Beispiel Datei oder URL), eine Fehlermeldung angezeigt und das Repository aus der Auswahl im File Picker entfernt, solange die Daten nicht eingegangen sind. Das Entfernen aus dem File Picker erfolgt durch die Schnittstellenmethode.
`is_visible`.

### Login-Status

Nach dem Aufruf des Konstruktors und des File Pickers, aus dem Moodle Core heraus, wird zunächst geprüft, ob der aktuelle Nutzer in ownCloud authentifiziert ist. An dieser Stelle wird die entsprechende Frage im Client beantwortet. Er prüft, ob der aktuelle Nutzer über ein valides Access Token verfügt. Sollte das zutreffen, wird dem Nutzer der Zugriff auf sein ownCloud Verzeichnis gewährt. Andernfalls wird die Methode `print_login` aufgerufen, welche einen Login-Link erstellt, der auf die `authorize` Schnittstelle in ownCloud verweist. Bei Betätigung des Links wird ein Popup-Fenster angezeigt, das den Nutzer zur Authentifizierung auffordert.

Hat sich der Nutzer authentifiziert, so wird das erhaltene Access Token fest für ihn gespeichert. Im Optimalfall muss sich ein Nutzer daher nur ein Mal authentifizieren um anschließend durchgehend Zugriff auf seine ownCloud Daten zu haben.

### Dateiauswahl

Ist ein Nutzer zum Zugriff auf ownCloud autorisiert, so werden ihm im File Picker alle Dateien und Ordner, welche sich in seinem persönlichen Verzeichnis befinden, zur Auswahl angeboten. Diese Dateiauswahl wird in der Methode `get_listing` implementiert.

Als Rückgabe dieser Methode wird ein Array erwartet, welches spezifische Informationen zu allen verfügbaren Dateien und Ordner und dessen Darstellung enthält. Bis auf die Weiterleitung zum OAuth 2.0 ownCloud Client ist der Kern der Funktion identisch zu der des WebDAV Repositories aufgabaut, da dort, der WebDAV Schnittstelle geschuldet, die selben Daten verarbeitet werden müssen.

Zu Beginn werden ansichtsspezifische Einstellungen getätigt:

``` php
$ret['dynload'] = true;
```

* Dies bestätigt dem File Picker, dass Inhalte dynamisch geladen werden. Das heißt, dass wenn z.B. ein Ordner angeklickt wird, der File Picker einen Ajax-Request versendet um den Inhalt des Ordners anzeigen zu können.

``` php
$ret['nosearch'] = true;
```

* Dieser Parameter verbietet die Suche in den Dateien.

``` php
$ret['nologin'] = false;
```

* Dieser Parameter sorgt dafür, dass jeder Nutzer sich einzeln einloggen muss. Zusätzlich wird dadurch automatisch ein Logout-Button im File Picker generiert.

> Im WebDAV Repository kann nur ein Nutzerkonto pro Instanz genutzt werden. Daher können dort Nutzername und Passwort für eine Instanz festgelegt werden, wodurch der Login einzelner Nutzer nicht möglich ist. Diese Option beinhaltet dort also den Wert `true`.

Nach der Festlegung der ansichtsspezifischen Einstellungen, werden mittels der Methode `get_listing` des Clients die nötigen Informationen aus ownCloud beschafft. Anschließend werden sie verarbeitet. Dabei wird zwischen Dateien und Ordnern unterschieden:

``` php
if (!empty($v['resourcetype']) && $v['resourcetype'] == 'collection') {

    if ($path != $v['href']) {

        $folders[strtoupper($title)] = array(
            'title' => rtrim($title, '/'),
            'thumbnail' => $OUTPUT->pix_url(file_folder_icon(90))->out(false),
            'children' => array(),
            'datemodified' => $v['lastmodified'],
            'path' => $v['href']
        );
    }
```

* Falls es sich um einen Ordner handelt, werden der Titel, ein Ordnerbild, der Pfad zum Ordner und der Zeitpunkt des letzten Zugriffs gespeichert.

``` php
} else {

    $size = !empty($v['getcontentlength']) ? $v['getcontentlength'] : '';

    $files[strtoupper($title)] = array(
        'title' => $title,
        'thumbnail' => $OUTPUT->pix_url(file_extension_icon($title, 90))->out(false),
        'size' => $size,
        'datemodified' => $v['lastmodified'],
        'source' => $v['href']
    );
}
```

* Falls es sich um eine Datei handelt wird zusätzlich zu den oben genannten Informationen noch die Dateigröße gespeichert.

Diese Informationen werden für jede im aktuellen Verzeichnis befindliche Datei und jeden Ordner festgehalten. Anschließend werden erst die Ordner und danach die Dateien alphabetisch sortiert in einem Array gespeichert. Dieses Array wird von der Funktion zurückgegeben und Moodle platziert anschließend die entsprechenden Einträge im File Picker.

### Dateidownload

Wenn ein Nutzer eine Datei aus dem File Picker zum Download ausgewählt hat, so wird dessen relativer Dateipfad an die Bibliotheksfunktion `get_file` übergeben. Nachdem für die Datei ein entsprechender Speicherplatz im Moodle Verzeichnis alloziert worden ist, wird die Anfrage nach der Datei an den Client weitergeleitet. Dieser wiederum leitet an einen WebDAV Client weiter, welcher die entsprechende Anfrage an ownCloud stellt. Falls die Operation erfolgreich ist, wird die Datei daraufhin aus ownCloud nach Moodle hochgeladen.

### Links und Referenzen

Anstelle einer Datei soll auch ein Downloadlink zu einer existierenden Datei bereitgestellt werden können. Um dies zu ermöglichen kann einerseits Moodles URL Aktivität genutzt werden. Nach der Auswahl einer Datei im File Picker wird die Methode `get_link` des Repositories mit dem relativen Pfad zu der gewählten Datei aufgerufen. Innerhalb der Methode wird die Anfrage an das Admin Tool weitergeleitet, welches die betreffende Datei öffentlich teilt. Nach einem erfolgreichen Zugriff auf die OCS Share API, generiert der Client einen öffentlichen Link zu der Datei und gibt diesen an das Repository zurück. Diesen übergibt das Repository anschließend an die Schnittstelle zu Moodle.

Eine weitere Methode zur Bereitstellung eines Links, bietet die Datei Aktivität in Moodle. Neben dem direkten [Download](repository/#dateidownload) der ausgewählten Datei, wird auch die Möglichkeit geboten eine Referenz zu der Datei zu erstellen. Sobald eine Datei im File Picker ausgewählt worden ist, wird die folgende Funktion des Repositories aufgerufen:

```php
public function get_file_reference($source) {
    $usefilereference = optional_param('usefilereference', false, PARAM_BOOL);

    $reference = $source;

    if ($usefilereference) {
        $reference = $this->get_link($source);
    }

    return $reference;
}
```

Falls der Nutzer eine Referenz angefordert hat, so wird, wie in der URL Aktivität, ein Link zu der Datei generiert. Dieser wird anschließend über die Schnittstelle zu Moodle hintergründig gespeichert. Wenn der Nutzer dann die Referenz der Datei aufruft, dann wird der Nutzer an die die Adresse, welche der Link enthält, weitergeleitet. Das folgende Codebeispiel zeigt die Methode, welche aufgerufen wird, wenn der Nutzer eine Referenz betätigt:

```php
public function send_file($storedfile, [...]) {
    redirect($storedfile->get_reference());
}
```

### Sonstige Einstellungen

#### Unveränderbare Eigenschaften

Für Repository Plugins gibt es einige Einstellungen die hart kodiert sind und sich nicht auf der Website anpassen lassen. Dazu gehören folgende Einstellungen:

* **`supported_returntypes`:** mögliche Rückgabetypen sind:
    * `FILE_INTERNAL`: Dateien dürfen im Moodle Dateien System hoch und runtergeladen werden.
    * `FILE_EXTERNAL`: Dateien bleiben im externen Repository und werden von dort bezogen.
    * `FILE_REFERENCE`: Dateien werden lokal erstellt, aber werden extern synchronisiert wenn notwendig.

> Alle Rückgabetypen werden von diesem Plugin unterstützt.

* **`supported_filetypes`:** hier wird spezifiziert welche Arten von Dateitypen unterstützt werden.

> Alle Dateitypen werden momentan von dem Plugin unterstützt.

#### Abhängigkeiten in der `version.php`

Um sicherzustellen, dass das Admin Tool bereits installiert und die damit zusammenhängende Abhängigkeit gewährleistet ist, muss diese Abhängigkeit im `owncloud` Repository definiert werden. Dies wird sichergestellt, indem in der `version.php` eine `dependency` gesetzt wird:

``` php
$plugin->dependencies = array(
    'tool_oauth2sciebo' => 2017030905);
```

## Continuous Integration

Als Continuous Integration Tool wurde Travis CI verwendet. Bei jeder Änderung im [GitHub Repository](https://github.com/pssl16/moodle-repository_owncloud) wird ein Build angestoßen, in dem das Plugin gebaut und anschließend in verschiedenen Umgebungen installiert und getestet wird. Folgende Parameter werden variiert:

* **PHP Versionen**: 5.6, 7.0
* **Datenbanken**: PostgreSQL, MySQL, SQLite
* **Branches des Moodle Core**: `MOODLE_31_STABLE`, `MOODLE_32_STABLE`, `master`

Der aktuelle Build-Status ist bei Travis einsehbar: [![Build Status](https://travis-ci.org/pssl16/moodle-repository_owncloud.svg?branch=master)](https://travis-ci.org/pssl16/moodle-repository_owncloud)
