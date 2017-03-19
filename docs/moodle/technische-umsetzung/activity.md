# Aktivität: `collaborative folders`

## Zweck

Das Aktivitäts Modul *Collaborative Folders* soll Lehrenden die Möglichkeit geben, für Studierende oder Gruppen von Studierenden Ordner für kollaboratives Arbeiten zu erstellen.
Die Autorisierung und Authentifizierung erfolgt über das `oauth2sciebo admin_tool`. Diese Aktivität `collaborativefolders` implementiert die User Story 4:

<ol start="4">
  <li>Als <b>Lehrender</b> möchte ich Studierenden oder Gruppen von Studierenden Ordner für kollaboratives Arbeiten bereitstellen.</li>
</ol>
Im Folgenden wird erklärt wozu einzelne Beteiligte des Integrationsszenario in der Lage sein müssen.
Zunächst ist es erforderlich einen neutralen Speicherort für die geteilten Ordner anzugeben. Falls der Lehrende keinen Zugriff auf die Ordner haben soll, dürfen sie nicht in seiner Instanz gespeichert sein. Um die Privatsphäre der Kursteilnehmer zu sichern darf ein Lehrender nicht unter ihrem Namen Ordner erstellen. Aus diesem Grund haben wir unsere Lösung so implementiert, dass der Moodle Administrator einen technischen Nutzer festlegen kann. In dessen Namen werden alle Ordner erstellt.

Der Lehrende soll in der Lage sein einem Kurs beliebig viele Instanzen der Aktivität *Collaborative Folders* hinzuzufügen. In den Einstellungen kann der Lehrende nun festlegen wie der Ordner im Moodle Kurs benannt werden soll. Außerdem kann der Lehrende entscheiden ob er Zugriff auf die Ordner bekommt.

Kursteilnehmer sollen den Ordner in Ihren eigenen ownCloud Account kopieren können. Dabei sollen sie dem Ordner einen eigene Namen geben können.

Mit dieser Aktivität kann der Lehrende Kursteilnehmer ermutigen kollaborativ zu arbeiten. Zusätzlich hat die Option die laufende Arbeit zu betreuen, indem er sich selbst Zugriff auf die Ordner gewährt. In diesem Fall werden die Kursteilnehmer darüber in Kenntnis gesetzt, dass der Ordner Lehrenden zugänglich ist.

Kursteilnehmer haben weniger Organisationsaufwand und werden bei ihrer Gruppenarbeit unterstützt. Zusätzlich muss kein Kursteilnehmer eigenen Speicherplatz zur Verfügung stellen.

## Vorgegebene Schnittstelle
Um die User Story zu realisieren haben wir ein Aktivity Plugin für Moodle entwickelt. Instanzen von Aktivity Plugins können Kursen beliebig oft hinzugefügt werden.
Für genauere Informationen besuchen sie die Moodle Dokumentation von [Aktivity modules](https://docs.moodle.org/dev/Activity_modules "Activity Modules")
In der `collaborativefolders/mod_form.php` fragen wir alle Einstellungen ab, die vor dem Erstellen der Ordner bekannt sein müssen. Dies beinhaltet Name des Ordners in Moodle, Zugriff des Lehrenden auf die erstellten Ordner und ob für Gruppen separate Ordner erzeugt werden sollen.
In der `collaborativefolders/settings.php` kann der Administrator den technischen Nutzer des Plugins festlegen. Dieser gilt für alle Instanzen, also global für das gesammte Plugin.
Die `collaborativefolders/lib.php` bietet eine Schnittstelle um auf das Hinzufügen, Ändern und Löschen von Instanzen zu reagieren.

## Implementierung
### Anmelden des technischen Nutzers

Der Admin der Moodle Seite kann in der Seiten Administration einen technischen Nutzer hinzufügen. Über `Website-Administration ► Plugins ► Aktivitäten ► collaborativefolders` kommt er zu den passenden Einstellungen. Dort wird er durch einen Login-Button auf die ownCloud Seite zum autorisieren der App weitergeleitet. Spätere Änderungen des technischen Nutzers sind durch ein Logout Button möglich. Dies ist nicht empfohlen und mit einer Warnmeldung versehen, da Kompilierungs-Problemen mit bestehenden Instanzen entstehen würden. Wir haben uns dafür entschieden den Logout trotzdem bereitzustellen, für den Fall das ein falscher Account authentifiziert wird. Hierbei sind wir insbesondere über folgendes Szenario gestoßen:

Der Seiten Administrator will den technischen Nutzer einloggen, ist aber noch mit seinem eigenen ownCloud Account oder dem Administrator Account der ownCloud authentifiziert. Er bemerkt nicht, dass er mit dem falschen Account eingeloggt ist autorisiert das Plugin. Als er seinen Fehler bemerkt, möchte er den technischen Nutzer so schnell wie möglich ändern, obwohl bestehende Instanzen neu erstellt werden müssen.

Implementiert haben wir dies in der `collaborativefolders/settings.php`. Diese erstellt eine Seite in den Admin Settings der Moodle Instanz. Hier müssen wir drei verschiedene Fälle beachten:

1. **Der technische Nutzer ist bereits angemeldet aber soll die Möglichkeit haben ausgeloggt zu werden.**
    Die `check_login()` Methode des `oauth2_owncloud` Plugins überprüft ob ein technischer Nutzer registriert ist. Falls der Nutzer angemeldet ist, kann der Administrator den Nutzer mit einem Logout-Button ausloggen. Dieser leitet den Nutzer auf eine neue Seite. Diese enthält eine Warnung da alte Instanzen des Plugins nicht länger genutzt werden können wenn der technische Nutzer geändert wird.

2. **Der technische Nutzer wird erstmals registriert.**

    Der token des technischen Nutzers wird für das Plugin in den `config` Einstellungen gespeichert, da er zu keinem Nutzer, aber zu dem Plugin, gehört. Zur Sicherheit setzten wir diese auf null, und zeigen dem Admin einen Login-Button an. Die Speicherung des token erfolgt durch den callback des oauth2 Protokolls.

3. **Der technische Nutzer soll ausgeloggt werden.**

    Dem Administrator wird dasselbe Login Fenster angezeigt wie bei der erstmaligen Registrierung. Genauso wird der bisherige *Access token* gelöscht. Zusätzlich wird jedoch ein `logout Event` ausgelöst. Diese Event wird geloggt, damit der Vorgang später nachverfolgt werden kann.

``` php
    $logoutevent = \mod_collaborativefolders\event\technical_user_loggedout::create($params);
    $logoutevent->trigger();
```

### Hinzufügen eine Instanz
Die Schnittstelle in Moodle für das Einstellungs Formular ist sehr ausführlich. Den Standard Einstellungen haben wir nur eine Checkbox hinzugefügt die bestimmt ob der Lehrende Zugriff auf die Ordner hat.
Außerdem benutzen wird den Moodle internen *group_mode*. Dieser ermöglicht es Gruppen separat zu bearbeiten. Wie wir für separate Gruppen Ordner erstellen erklären wir unter anderem im nächsten Abschnitt, der sich mit dem Erstellen von Ordnern auseinander setzt. Falls sie mehr Informationen zu der `mod_form.php` in Moodle benötigen besuchen sie [diese Seite](https://docs.moodle.org/dev/Activity_modules#mod_form.php).

### Erstellen von Ordnern
Zum Erstellen von Ordnern haben einen Observer implementiert, der aufgerufen wird wenn eine Instanz der Aktivität `collaborativefolders` erstellt wird.
In `collaborativefolders/db/events.php` können in einem array alle Observer registriert werden und werden von Moodle verwaltet.
```php
$observers = array(
        array(
                // Zuerst wird das Event, dass beobachtet werden soll festgelegt.
                'eventname'   => '\core\event\course_module_created',
                // Als nächstes wird der Observer festgelegt.
                'callback'    => 'mod_collaborativefolders\observer::collaborativefolders_created',
                'internal'  => false,
                'priority'  => 1000
        )

);
```
Somit wird sobald das Event `course_module_created` erzeugt wird, der Observer aufgerufen. Es gibt zwei Hauptgründe warum dies erforderlich war. Zunächst brauchten wir für das Erstellen der Ordner die `course_module_id`. Diese ist nur verfügbar wenn die Instanz schon erstellt wurde, somit kann nicht die Schnittstelle der `collaborativefolders/lib.php` `add_instance()` genutzt werden. Desweiteren ruft unser Observer einen CronJob auf. Dieser sorgt dafür, dass Ordner zeitlich verzögert erstellt werden, somit wird der Server nicht überlastet. Falls Ordner für große Kurse mit über 50 Gruppen erstellt werden sollen könnte es hier sonst zu Schwierigkeiten kommen.
Beim Erstellen müssen zwei unterschiedliche Szenarien betrachtet werden:

Im ersten Szenario wird ein Ordner für alle Studierenden eines Kurses erstellt.
Die Anforderung an das Modul ist also einen Ordner in dem gespeicherten Account zu erstellender der eindeutig Identifizierbar ist. Dies ist gesichert indem wir die Ordner nach der `course_module_id` benennen. Diese ist einzigartig für jede Aktivität, die in Moodle erzeugt wird. So kann es, soweit manuell
keine Ordner erstellt werden, zu keinen Synchronisationskonflikten kommen.

``` php
$paths = array();
            $paths['cmid'] = $cmid;

            list ($course, $cm) = get_course_and_cm_from_cmid($cmid, 'collaborativefolders');
```

Im zweiten Szenario sollen Ordner für Gruppen innerhalb eines Kurses erstellt werden.
Auch hier müssen eindeutig identifizierbare Ordner erstellt werden. Deswegen wird ein Überordner erstellt mit der `course_module_id`. In diesem Ordner wird nun zu jeder Gruppe ein Unterordner erstellt.
``` php
if (groups_get_activity_groupmode($cm) != 0) {
    $grid = $cm->groupingid;
    $groups = groups_get_all_groups($course->id, 0, $grid);

    foreach ($groups as $group) {
        $path = $cmid . '/' . $group->id;
        $paths[$group->id] = $path;
    }
}
```
Zu diesem Zweck wird in der Variable `$path` die einem Cronjob übergeben wird nicht nur die `course_module_id` gespeichert, sondern auch für jede `$groupid` ein Feld.
Der Cronjob, der nun die Ordner erstellt, kann anhand der Einträge im Array erkennen wie viele und welche Ordner er erstellen muss.

Der Cronjob findet sich in `collaborativefolders/classes/task/collaborativefolders_create.php`. Er erbt von der abstrakten Klasse `\core\task\adhoc_task`.
Dieser ließt aus den Daten die er übergeben bekommt alle Ordner aus die erstellt werden müssen.
``` php
foreach ($data as $key => $value) {
            $code = $oc->handle_folder('make', $value);
            if ($code == false) {
                throw new \coding_exception('Folder ' . $value . ' not created.');
            } else {
                mtrace('Folder: ' . $value . ', Code: ' . $code);
                if (($code != 201) && ($code != 405)) {
                    throw new \coding_exception('Folder ' . $value . ' not created.');
                }
            }
        }
```
### Ansicht der bestehenden Instanzen

Nun da der Lehrende alle notwendigen Einstellungen tätigen konnte, mussten wir die Ansicht der Kursteilnehmer auf die Aktivität mit allen notwendigen Funktionalitäten implementieren. Dies beinhaltet die individuelle Namensvergabe für Ordner in ownCloud und das Hinzufügen dieser Ordner zur eigenen Instanz.
Lehrende sollen entweder eine Übersicht aller Ordner haben, oder keinen Zugriff auf die Ordner haben. Alle Funktionalitäten sind in der `view.php` implementiert.

#### Sicht der Studierenden
Für Studierende wird zunächst überprüft, ob der `group_mode` aktiviert ist. Wenn dies der Fall ist, wird an den Pfad an dem später der Ordner gefunden werden soll die Gruppenid angefügt. Moodle hat hierfür intern eine Methode `groups_get_activity_group()` Die zu einer Instanz der Aktivität angibt in welcher Gruppe der Studierende ist.
Die `view.php` wird zu verschiedenen Zwecken aufgerufen die behandelt werden müssen.

1. **Der Ordnername wird erstmals gespeichert**

    Wir speichern den Ordnernamen in den moodle `user_preferences`. Diese werden für jeden Nutzer einzeln gespeichert und können beliebig geändert oder gelöscht werden. Die Eingabemaske für einen Ordnernamen wird nur angezeigt wenn bis jetzt kein Name gesetzt wurde oder der Nutzer explizit ausgewählt hat, das der Name zurückgesetzt werden soll.

2. **Der Name des Ordners wird geändert**

    Wenn der Name des Ordners zurückgesetzt wird, wird ein URL Parameter *reset=1* an die URL übergeben. In diesem Fall wird dem Kursteilnehmer eine Eingabemaske angezeigt. Diese ist als eigene Klasse in dem Ordner `collaborativefolders/classes` implementiert. Sie erbt von der abstrakten Klasse `moodleform`. Es muss nun sichergestellt werden das vergebene Namen kompatible mit ownCloud sind. Moodle unterstützt die zugelassenen Eingaben durch Form Element Regeln zu begrenzen.

    ```
    $mform->addRule('namefield', get_string('err_alphanumeric', 'form'), 'alphanumeric', null, 'client');
    ```

    Diese Regel verbietet andere Eingaben zu speichern, als Buchstaben und Zahlen.

3. **Der Nutzer logt sich aus seinem aktuell gespeichertem Account aus**
    Der Nutzer muss mit Hilfe des `oauth2owncloud` admin_tools aus-geloggt werden, und der accesstoken wird auf null gesetzt.

    ```
    $ocs->owncloud->log_out();
    set_user_preference('oC_token', null);
    ```

4. **Kursteilnehmer rufen die Seite auf, obwohl die Ordner noch nicht vom CronJob erstellt wurden.**

    Für jeden Ordner wird überprüft, ob der Ordner schon erstellt wurde:Zur Information wird dem Nutzer angezeigt, dass die Ordner noch nicht erstellt wurden.

    ```
    $content = json_decode($element->customdata);
    $cmidoftask = $content->cmid;
    if ($id == $cmidoftask) {
        $created = false;
    }
    ```

#### Sicht der Lehrenden

Falls der Lehrende sich selbst keinen Zugriff gewährt hat, sieht er den Ordner auch nicht wenn er in einer Gruppe eingeschrieben ist. Hat der Lehrende Zugriff auf die Ordner sieht er eine tabellarische Auflistung aller bestehenden Ordner. Zusätzlich kann er genau wie Studierende seinen ownCloud Account ändern, den Namen des Ordners ändern und Ordner per Klick der eigenen Instanz hinzufügen. In dem Fall, dass Gruppenordner erstellt wurden wird dem Lehrenden der Überordner aller Gruppenordner hinzugefügt.

## Tests und Continuous Integration
