# Repository: `sciebo`

## Zweck des Plugins
Der Plugin Typ *Repository* wird in Moodle unter anderem verwendet um Nutzer die Möglichkeit zu schaffen Zugang zu
Dateien aus externen Quellen zu bekommen. Das Repository Plugin Sciebo kann somit die User Stories [Zwei](/index.md, "2. Als <b>Nutzer</b> möchte ich in der Dateiauswahl im Learnweb eine Datei aus meiner sciebo Instanz hochladen.") und [Drei](/index.md, "Als Nutzer möchte ich in der Dateiauswahl im Learnweb eine Datei aus meiner sciebo Instanz verlinken.") realisieren. Sobald der Admin das Plugin unter Site administration `> Plugins > Repositories > Manage repositories` aktiviert hat, sieht der Nutzer im File Picker folgende Optionen:

*(Mehr Informationen zum File Picker finden sie in der [Moodle Dokumentation](https://docs.moodle.org/32/en/File_picker))*
<div class="alert alert-danger">
  <strong>TODO:</strong> BILD nach richtiger Benennung.
</div>
Wenn der Nutzer auf den anmelde Button klickt, wird er zu einem Pop-up Window weitergeleitet, dass ihn auffordert sich in der zugehörigen ownCloud Istanz zu authentifizieren. Sobald der Nutzer sich hier einmalig authentifiziert hat, werden ihm im File Picker seine Dateien angezeigt. Das Plugin bietet dem Benutzer zusätzliche Vorteile
dadruch, dass die Authentifizierung nur einmalig erfolgt. Durch den OAuth2 Protokollablauf werden danach Refresh Tokens angefordert ohne dass der Nutzer dies im Front-End zu sehen bekommt. Somit kann der Nutzer angemeldet bleiben während verschiedenen Sessions.

## Vorgegebene Schnittstelle

Wie auch im [Admin Tool](/moodle/admin-tool.md) müssen zunächst einige Standartdateien implementiert werden:

* **`version.php`:** Beschreibt die Versionsnummer des Plugins, die benötigte moodle Version und Abhängigkeiten des Plugins.
* **`db/access.php`:** Legt die Berechtigungen für definierte Aktionen innerhalb des Plugins anhand von Nutzerrollen fest.
* **`lang/en/repository__sciebo.php`:** Beinhaltet Sprachstrings für unterschiedliche Regionen und Sprachen, sodass definierte Strings,
abhängig von der jeweiligen Sprache, dynamisch angezeigt werden können. Als Standard wird eine englische Sprachdatei erwartet.

Für Repository-Plugins müssen außerdem folgende Dateien implementiert werden:
* **`pix/icon.png`:** Hier wird ein 16x16 icon platziert, welches für das Plugin genutzt wird.
* **`lib.php`:** Hier wird eine Klasse `repository_sciebo extends repository` deklariert, die als Hauptaufgabe die Integration in den File Picker verwaltet.

## Implementierung der vorgegebenen Schnittstelle
### Implementierung der `lib.php`:
In der lib.php wird eine Klasse definiert die von der abstrakten Klasse `repository` erbt.
#### `function __construct()`
Diese Funktion wird jedes mal aufgerufen, wenn eine Instanz des Plugins erstellt wird. Hier wird ein Objekt der sciebo Klasse des Admin tools erzeugt, das als Parameter eine returnurl übergeben bekommt.
``` php
$returnurl = new moodle_url('/repository/repository_callback.php', [
    'callback'  => 'yes',
    'repo_id'   => $repositoryid,
    'sesskey'   => sesskey(),
]);
$this->sciebo = new sciebo($returnurl);
```
Der callback url werden als Parameter noch zusätzlich die id und der sesskey übergeben, um einen callback zu ermöglichen.

Desweiteren wird die Parent Methode aufgerufen, die die nötigen Datenbankeinträge tätigt.
#### `function get_file()`:
Diese Funktion stellt eine Scnittstelle uzm oauht2 Objekt bereit. Die Funktion überprüft ob schon eine offene Verbindung besteht mit Hilfe der Methode `sciebo->dav->open()`. Falls keine Verbindung besteht wird die Funktion `$this->sciebo->get_file();` aufgerufen. Die Funktion ähnelt sehr der Funktion des `WebDAV Repository`, statt Basic Authentication wird jedoch das OAuth2 Protokoll benutzt. Danach wird der Nutzer  mit Hilfe der `logout()` Funtion ausgeloggt.
#### `function get_listing()`
``` php/**
 * This function does exactly the same as in the WebDAV repository. The only difference is, that
 * the Sciebo OAuth2 client uses OAuth2 instead of Basic Authentication.
 * @param string $path relative path the the directory or file.
 * @param string $page
 * @return array directory properties.
 */
public function get_listing($path='', $page = '') {
    global $CFG, $OUTPUT;
    $list = array();
    $ret  = array();
    $ret['dynload'] = true;
    $ret['nosearch'] = true;
    $ret['nologin'] = false;
    $ret['path'] = array(array('name' => get_string('owncloud', 'repository_sciebo'), 'path' => ''));
    $ret['list'] = array();
    $ret['manage'] = $CFG->wwwroot.'/'.$CFG->admin.'/tool/oauth2sciebo/index.php';

    if (!$this->sciebo->dav->open()) {
        return $ret;
    }
    $webdavpath = rtrim('/'.ltrim(get_config('tool_oauth2sciebo', 'path'), '/ '), '/ ');
    if (empty($path) || $path == '/') {
        $path = '/';
    } else {
        $chunks = preg_split('|/|', trim($path, '/'));
        for ($i = 0; $i < count($chunks); $i++) {
            $ret['path'][] = array(
                'name' => urldecode($chunks[$i]),
                'path' => '/'. join('/', array_slice($chunks, 0, $i + 1)). '/'
            );
        }
    }

    // The WebDav methods are getting outsourced and encapsulated to the sciebo class.
    $dir = $this->sciebo->get_listing($webdavpath. urldecode($path));

    if (!is_array($dir)) {
        return $ret;
    }
    $folders = array();
    $files = array();
    foreach ($dir as $v) {
        if (!empty($v['lastmodified'])) {
            $v['lastmodified'] = strtotime($v['lastmodified']);
        } else {
            $v['lastmodified'] = null;
        }

        // Remove the server URL from the path (if present), otherwise links will not work - MDL-37014.
        $server = preg_quote(get_config('tool_oauth2sciebo', 'server'));
        $v['href'] = preg_replace("#https?://{$server}#", '', $v['href']);
        // Extracting object title from absolute path.
        $v['href'] = substr(urldecode($v['href']), strlen($webdavpath));
        $title = substr($v['href'], strlen($path));

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
    }
    ksort($files);
    ksort($folders);
    $ret['list'] = array_merge($folders, $files);
    return $ret;
}
```
#### `function get_link`
``` php
/**
 * Method to generate a downloadlink for a chosen file (in the file picker).
 * Creates a share for the chosen file and fetches the specific file ID through
 * the OCS Share API (ownCloud).
 * @param string $url relative path to the chosen file
 * @return string the generated downloadlink.
 * @throws repository_exception if $url is empty an exception is thrown.
 */
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
#### `function print_login()`
``` php
/**
 * Prints a simple Login Button which redirects to a Authorization window from ownCloud.
 * @return array login window properties.
 */
public function print_login() {
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
}
```
#### additional admin settings funtions:
``` php

    /**
     * Searching is not available at the moment.
     * @return bool false
     */
    public function global_search() {
        return false;
    }




    /**
     * Method that generates a reference link to the chosen file.
     * TODO Find another method then just calling the get_link function.
     */
    public function send_file($storedfile, $lifetime=86400 , $filter=0, $forcedownload=false, array $options = null) {
        $ref = $storedfile->get_reference();
        $ref = $this->get_link($ref);
        header('Location: ' . $ref);
        $this->logout();
    }

    /**
     * Function which checks whether the user is logged in on the Sciebo instance.
     * @return bool false, if no Access Token is set or can be requested.
     */
    public function check_login() {
        return $this->sciebo->is_logged_in();
    }


    /**
     * Deletes the held Access Token and prints the Login window.
     * @return array login window properties.
     */
    public function logout() {
        $this->sciebo->log_out();

        return $this->print_login();
    }

    /**
     * Sets up access token after the redirection from ownCloud.
     */
    public function callback() {
        $this->sciebo->callback();
    }

    /**
     * Is this repository accessing private data?
     *
     * @return bool
     */
    public function contains_private_data() {
        return false;
    }


      @return string '*' means this repository support any files

    public function supported_filetypes() {
        return '*';
    }

    /**
     * Method to define which Files are supported (hardcoded can not be changed in Admin Menü)
     *
     * Can choose FILE_REFERENCE|FILE_INTERNAL|FILE_EXTERNAL
     * FILE_INTERNAL - the file is uploaded/downloaded and stored directly within the Moodle file system
     * FILE_EXTERNAL - the file stays in the external repository and is accessed from there directly
     * FILE_REFERENCE - the file may be cached locally, but is automatically synchronised, as required,
     *                 with any changes to the external original
     * @return int return type bitmask supported
     */
    public function supported_returntypes() {
        return FILE_INTERNAL | FILE_EXTERNAL;
    }
    ```
### Implementierung der `db/access.php`:
Standardmäßig muss nur eine `capability` in einem Repository-Plugin definiert werden. Diese heißt view capability und beschreibt wer das Repository sehen darf, sobald es vom Site Admin freigegeben und aktiviert wurde.
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

### Besonderheiten der `version.php`:

Um sicherzustellen, dass die Authentifizierung korrekt abläuft muss das admin-tool oauth2sciebo installiert sein. Dies wird sichergestellt indem in der `version.php` eine Abhängigkeit gesetzt wird:
``` php
$plugin->dependencies = array(
    'tool_oauth2sciebo' => ANY_VERSION);
```
Für genauere Informationen zu der version.php in Moodle sehen sie sich hier die offizielle Dokumentation der [`version.php`](https://docs.moodle.org/dev/version.php) an


## Tests und CI
