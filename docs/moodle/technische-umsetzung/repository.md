# Repository: `owncloud`
<img alt="Name Form" src="../images/icon_repository.svg" width=40% float=right>

## Zweck

Der Plugin Typ *Repository* wird in Moodle unter anderem verwendet um Nutzer die Möglichkeit zu schaffen Zugang zu Dateien aus externen Quellen zu bekommen. Das Repository Plugin ownCloud kann somit folgende Integrationsszenarien realisieren:

> Als **Nutzer** möchte ich in der Dateiauswahl in Moodle eine Datei aus meiner ownCloud Instanz **hochladen**.

> Als **Nutzer** möchte ich in der Dateiauswahl in Moodle eine Datei aus meiner ownCloud Instanz **verlinken**.

Unser Plugin soll dem Administrator der Seite ermöglichen das Repository zu aktivieren und zu deaktivieren.
Alle Nutzer der Moodle Instanz sollen mit Hilfe des Plugins sind in ownCloud authentifizieren können. Danach sollen sie in Moodle bequem zwischen Ihren Dateien wählen können und sich mit verschiedenen Accounts authentifizieren können. Besonders im Vordergrund steht das unser Plugin möglichst einfach zu bedienen ist, um die Benutzerfreundlichkeit zu erhöhen.

## Vorgegebene Schnittstelle

Wie auch im [Admin Tool](/moodle/technische-umsetzung/admin-tool.md) müssen zunächst einige Standartschnittstellen implementiert werden:

* **`version.php`:** Beschreibt die Versionsnummer des Plugins, die benötigte moodle Version und Abhängigkeiten des Plugins.
* **`db/access.php`:** Legt die Berechtigungen für definierte Aktionen innerhalb des Plugins anhand von Nutzerrollen fest.
* **`lang/en/repository__sciebo.php`:** Beinhaltet Sprachstrings für unterschiedliche Regionen und Sprachen, sodass definierte Strings,
abhängig von der jeweiligen Sprache, dynamisch angezeigt werden können. Als Standard wird eine englische Sprachdatei erwartet.

Für Repository-Plugins müssen außerdem folgende Dateien implementiert werden:

* **`pix/icon.png`:** Hier wird ein 16x16 icon platziert, welches für das Plugin genutzt wird. Dies kann auch beliebig angepasst werden.
* **`lib.php`:** Hier wird eine Klasse `repository_sciebo extends repository` deklariert, die als Hauptaufgabe die Integration in den File Picker verwaltet.

## Implementierung

Die WebDAV Zugriffe und der Ablauf des OAuth2.0 Verfahrens wird vom [oauth2owncloud_admin_tool](/moodle/technische-umsetzung/admin-tool.md) geregelt. Somit hat das *Repository* niemals Zugriff auf den Login-Status oder den *Access Token*. Falls Konfigurationen des *Admin Tool* fehlen, ist das *Repository* nicht mehr verfügbar.

### Implementierung der `lib.php`

In der lib.php wird eine Klasse definiert die von der abstrakten Klasse `repository` erbt. Im Folgenden wird darauf eingegangen wie die vorgegebenen Funktionen implementiert wurden.

#### `__construct()`

Diese Funktion wird jedes mal aufgerufen, wenn eine Instanz des Plugins erstellt wird. Hier wird ein Objekt der sciebo Klasse des Admin tools erzeugt, das als Parameter eine returnurl übergeben bekommt.

``` php
$returnurl = new moodle_url('/repository/repository_callback.php', [
    'callback'  => 'yes',
    'repo_id'   => $repositoryid,
    'sesskey'   => sesskey(),
]);
$this->sciebo = new sciebo($returnurl);
```

Der `callback url` werden als Parameter noch zusätzlich die id und der sesskey übergeben, damit die Sitzung nach der Authentifizierung in ownCloud wieder hergestellt werden kann.

Des weiteren wird die Parent Methode aufgerufen, die die nötigen Datenbankeinträge tätigt.

#### `get_file()`

Diese Funktion stellt eine Schnittstelle zum oauht2 Objekt des Admin tools bereit. Die Funktion überprüft ob schon eine offene Verbindung besteht mit Hilfe der Methode `sciebo->dav->open()`. Falls keine Verbindung besteht wird die Funktion `$this->sciebo->get_file();` aufgerufen. Die Funktion ähnelt sehr der Funktion des `WebDAV Repository`, statt Basic Authentication wird jedoch das OAuth2 Protokoll benutzt. Danach wird der Nutzer  mit Hilfe der `logout()` Funktion ausgeloggt. Dies erfolgt auch über das Admin tool.

#### `get_listing()`

Diese Funktion wird aufgerufen um im File Picker die verfügbaren Dateien anzuzeigen. Als Rückgabe wird ein Array aller verfügbaren Dateien mit spezifischen Informationen über diese Dateien erwartet. Bis auf die Authentifizierung funktioniert diese Methode genauso wie die Methode des *WebDAV Repository*. Am Anfang werden noch grundlegende Einstellungen für die Ansicht definiert:

``` php
$ret['dynload'] = true;
```

Dies bestätigt dem File Picker, dass Inhalte dynamisch geladen werden. Das heißt das wenn z.B. ein Ordner angeklickt wird der File Picker einen Ajax-Request versendet um den Inhalt des Ordners anzeigen zu können.

``` php
$ret['nosearch'] = true;
```

Dieser Parameter verbietet die Suche in den Dateien.

``` php
$ret['nologin'] = false;
```

Dieser Parameter sorgt dafür, dass der Login für jede Instanz notwendig ist. Zusätzlich wird dadurch automatisch ein Logout-Button im File Picker generiert.

Mit Hilfe der sciebo Klasse wird überprüft ob es sich um eine Datei oder einen Ordner handelt.

``` php
if (!empty($v['resourcetype']) && $v['resourcetype'] == 'collection') {
    // A folder.
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

Falls es sich um einen Ordner handelt wird der Titel, ein Ordner als Bild, der Pfad zum Ordner und der Zeitpunkt des letzten Zugriffs gespeichert.

``` php
} else {
    // A file.
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

Falls es sich um eine Datei handelt wird zusätzlich zu den oben genannten Informationen noch die Datei Größe gespeichert.

Mit Hilfe einer `foreach()` Schleife wird dies für jede Datei durchgeführt. Anschließend werden zuerst Ordner und danach alphabetisch sortiert die Dateien in einem Array gespeichert. Diese Array wird von der Funktion wiedergegeben, Moodle platziert nun die entsprechenden Einträge im File Picker.

#### `get_link()`

Anstelle einer Datei soll es auch möglich sein, einen Downloadlink zu einer existierenden Datei bereitzustellen. Diese wird von dem Modul URL genutzt. Zusätzlich besteht die Möglichkeit im File Picker Dateien zu verlinken. Die zweite Option erlauben wir in unserem Plugin nicht, da uns die Zeit fehlte die zusätzliche Funktionalität zu implementieren. Dies haben wir in der Methode [`supported_returntypes()`](#repository-spezifische-einstellungen) ausgeschlossen.
Die Implementierung der `get_link()` Methode ist nicht trivial da sich der Link abhängig von den Einstellungen im Admin Tool ändert.
Mit Hilfe der Funktion `get_config()`können in Moodle Einstellungen spezifischer Plugins ausgelesen werden. In der Methode wird wiederum die Methode `get_link()` des Objektes `sciebo` aufgerufen. Um den funktionierenden Downloadlink zurück zu geben muss nun noch der Präfix und die Serveraddresse vor den zurückgegebenen Pfad gesetzt werden. Desweiteren wird hinter den Pfad noch `'public.php?service=files&t=' . $fileid . '&download'` angefügt. Dies ist eine ownCloud spezifische Implementation einen Downloadlink zu generieren. Hier finden sie genauere Informationen zur [ownCloud external API](https://doc.owncloud.org/server/8.2/developer_manual/core/ocs-share-api.html).

``` php
public function get_link($url) {

    $pref = get_config('tool_oauth2sciebo', 'type') . '://';

    $output = $this->sciebo->get_link($url);
    $xml = simplexml_load_string($output);
    $fields = explode("/s/", $xml->data[0]->url[0]);
    $fileid = $fields[1];

    $path = str_replace('remote.php/dav/', '', get_config('tool_oauth2sciebo', 'path'));

    return $pref . get_config('tool_oauth2sciebo', 'server'). '/' . $path .
            'public.php?service=files&t=' . $fileid . '&download';
}
```

#### `print_login()`

Um einen Benutzer zum ersten mal mit dem OAuth 2.0 Protokoll anmelden zu können, muss er einmalig seinen Benutzer Namen und sein Passwort angeben. Sobald der Nutzer auf den Login Button klickt erscheint ein Pop-up Window oder es öffnet sich ein neuer Tab im Browser, indem der Nutzer aufgefordert wird seinen Namen und sein Passwort anzugeben.

``` php
    $url = $this->sciebo->get_login_url();
    if ($this->options['ajax']) {
        $ret = array();
        $btn = new \stdClass();
        $btn->type = 'popup';
        $btn->url = $url->out(false);
        $ret['login'] = array($btn);
        return $ret;
    } else {
        echo html_writer::link($url, get_string('login', 'repository'), array('target' => '_blank'));
    }
```

Als Nächstes muss der Benutzer die App autorisieren. Nun wird er zu Moodle zurückgeleitet. Die Funktion wird nur aufgerufen, falls der Nutzer weder mit seinem token noch mit einem refreshtoken authentifiziert werden kann.

#### Repository spezifische Einstellungen

Für Repository Plugins gibt es einige Einstellungen die hart kodiert sind und sich nicht auf der Website anpassen lassen. Hierzu gehören folgende Funktionen:

* **`supported_returntypes()`**  mögliche Rückgabetypen sind:
    * `FILE_INTERNAL`: Dateien dürfen im Moodle Dateien System hoch und runtergeladen werden.
    * `FILE_EXTERNAL`: Dateien bleiben im externen Repository und werden von dort bezogen.
    * `FILE_REFERENCE`: Dateien werden lokal erstellt, aber werden extern synchronisiert wenn notwendig.

Wir haben `FILE_INTERNAL` und `FILE_EXTERNAL` erlaubt, da die Synchronisation von Dateien einen zu großen Implementationsaufwand für unser Projektseminar darstellte.

* **`supported_filetypes()`**  hier wird spezifiziert welche Arten von Dateitypen unterstützt werden. Wir haben alle Dateitypen erlaubt.

### Implementierung der `db/access.php`
Standardmäßig muss nur eine `capability` in einem Repository-Plugin definiert werden. Diese heißt `view capability` und beschreibt wer das Repository sehen darf, sobald es vom Site Admin freigegeben und aktiviert wurde.

``` php
$capabilities = array(
    'repository/sciebo:view' => array(
        'captype' => 'read',
        'contextlevel' => CONTEXT_MODULE,
        'archetypes' => array(
            'user' => CAP_ALLOW
        )
    )
);
```

In unserem Fall darf jeder Nutzer das Repository sehen.

### Besonderheiten der `version.php`

Um sicherzustellen, dass die Authentifizierung korrekt abläuft muss das admin-tool oauth2sciebo installiert sein. Dies wird sichergestellt indem in der `version.php` eine Abhängigkeit gesetzt wird:

``` php
$plugin->dependencies = array(
    'tool_oauth2sciebo' => ANY_VERSION);
```

Für genauere Informationen zu der version.php in Moodle sehen sie sich hier die offizielle Dokumentation der [`version.php`](https://docs.moodle.org/dev/version.php) an.

## Tests und Continuous Integration
