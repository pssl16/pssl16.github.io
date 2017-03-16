# Aktivität: `collaborative folders`

## Zweck

Das Aktivitäts Modul *Collaborative Folders* soll Lehrenden die Möglichkeit geben, für Studierende oder Gruppen von Studierenden Ordner für kollaboratives Arbeiten zu erstellen.
Die Autorisierung und Authentifizierung erfolgt über das `oauth2sciebo admin_tool`. Diese Aktivität `collaborativefolders` implementiert die User Story 4:

<ol start="4">
  <li>Als <b>Lehrender</b> möchte ich Studierenden oder Gruppen von Studierenden Ordner für kollaboratives Arbeiten bereitstellen.</li>
</ol>

Der Administrator muss zunächst einen Speicherort für die Ordner festlegen. Hierfür authentifiziert er sich mit einem technischen Nutzer.
Der Lehrende kann nun einem Kurs beliebig viele Instanzen der Aktivität *Collaborative Folders* hinzufügen. In den Einstellungen kann der Lehrende
nun festlegen wie der Ordner im Moodle Kurs benannt werden soll und wie der Ordner in der Sciebo Instanz benannt werden soll.

![Settings for the activity collaborativefolders](activity_settings/settings.svg "Settings for the activity collaborativefolders")

Desweiteren hat der Lehrende die Möglichkeit Ordner nur für einzelne Gruppen zu erstellen. Dies erfolgt über den Moodle `group_mode`. Wenn der Lehrende diesen auswäht wird für jede Gruppe ein eigener Ordner erstellt. Der Lehrende hat entweder Zugriff auf alle oder auf keinen Ordner, selbst wenn er in den Gruppen eingeschrieben ist.

## Vorgegebene Schnittstelle
Zur Implementierung des Integrationszenario benutzen wir die `mod_form.php` um die Einstellungen für einzelne Instanzen des Plugins festzulegen.
In der `settings.php` kann der Administrator den technischen Nutzer des Modules festlegen. Die `lib.php` bietet eine Schnittstelle um auf das Hinzufügen, Ändern und Löschen von Instanzen zu reagieren.
<div class="alert alert-danger">
  <strong>TODO:</strong> nicht zu genau beschreiben die moodle Dokumentationist sehr gut lieber auf die Implementation mehr eingehen.
</div>
Für genauere Informationen besuchen sie die Moodle Dokumentation von [Aktivity modules](https://docs.moodle.org/dev/Activity_modules "Activity Modules")
## Implementierung
### Anmelden des technischen Nutzers

Der Admin der Moodle Seite kann in der Seiten Administration einen technischen Nutzer hinzufügen. Spätere Änderungen sind möglich, aber nicht empfohlen und mit einer Warnmeldung versehen, da dies zu Kompilierungs-Problemen mit bestehenden Instanzen des Moduls führen würde.

### Erstellen von Ordnern
Wir haben einen Observer implementiert der darauf reagiert wenn eine Instanz der Aktivität `collaborativefolders` erstellt wird.
Dieser muss zwei Szenarien betrachten.
Im ersten Szenario wird ein Ordner für alle Studierenden eines Kurses erstellt.
Die Anforderung an das Modul ist also einen Ordner in dem gespeicherten Account zu erstellender der eindeutig Identifizierbar ist.
Damit keine Synchronisationskonflikte entstehen, erstellt das Modul eine Ordner Struktur. Der jeweils erforderliche Ordner wird in einem
Ordner erstellt der nach der `course_module_id` benannt ist, diese ist einzigartig für jede Aktivität die in Moodle in einem Kurs erzeugt wird. So kann es soweit manuell
keine Ordner erstellt werden zu keinen Synchronisationskonflikten kommen.

``` php
$paths = array();
            $paths['cmid'] = $cmid;

            list ($course, $cm) = get_course_and_cm_from_cmid($cmid, 'collaborativefolders');
```

Im zweiten Szenario sollen Ordner nur für Gruppen innerhalb eines Kurses erstellt werden.
Auch hier muss erst ein eindeutig identifizierbarer Ordner erstellt werden. Deswegen wird ein Überordner erstellt mit der `course_module_id`. In diesem Ordner sollen nun zu jeder Gruppe Unterordner erstellt werden.
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
Zu diesem Zweck wird in der Variable `$path` die einem Cronjob übergeben wird nicht nur die `course_module_id` gespeichert sondern auch für jede `$groupid` ein Feld.
Der Cronjob der nun die Ordner erstellt kann anhand der Einträge im Array erkennen wie viele und welche Ordner er erstellen muss.
Wir haben uns für einen Cronjob entschieden um eine Überlastung der Server zu vermeiden.

## Tests und Continuous Integration
