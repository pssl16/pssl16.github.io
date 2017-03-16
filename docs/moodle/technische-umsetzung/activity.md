# Aktivität: `collaborative folders`

## Zweck

Das Aktivitäts Modul *Collaborative Folders* soll Lehrenden die Möglichkeit geben, für Studierende oder Gruppen von Studierenden Ordner für kollaboratives Arbeiten zu erstellen.
Die Autorisierung und Authentifizierung erfolgt über das `oauth2sciebo admin_tool`. Diese Aktivität `collaborativefolders` implementiert die User Story 4:

<ol start="4">
  <li>Als <b>Lehrender</b> möchte ich Studierenden oder Gruppen von Studierenden Ordner für kollaboratives Arbeiten bereitstellen.</li>
</ol>
Im Folgenden wird erklärt wozu einzelne Beteiligte des Integrationszenarioin der Lage sein müssen.
Dafür ist es erforderlich einen neutralen Speicherort für die geteilten Ordner bereitzustellen. Falls der Lehrende keinen Zugriff auf die Ordner haben soll, dürfen sie nicht in seiner Instanz gespeichert sein. Um die Privatsphäre der Kursteilnehmer zu sichern darf ein Lehrender nicht unter ihrem Namen Ordner erstellen. Aus diesem Grund haben wir unsere Lösung so implementiert, dass der Moodle Administrator einen technischen Nutzer festlegen kann. In dessen Namen werden alle Ordner erstellt.
Der Lehrende wird in der Lage sein einem Kurs beliebig viele Instanzen der Aktivität *Collaborative Folders* hinzufügen. In den Einstellungen kann der Lehrende
nun festlegen wie der Ordner im Moodle Kurs benannt werden soll und wie der Ordner in der ownCloud Instanz benannt werden soll. Mit dieser Aktivität kann er Kursteilnehmer ermutigen kollaborativ zu arbeiten. Zusätzlich hat die Option die laufende Arbeit zu betreuen, indem er sich selbst Zugriff auf die Ordner gewährt. In diesem Fall werden die Kursteilnehmer darüber in Kenntniss gesetzt, dass der Ordner Lehrenden zugänglich ist.
Kursteilnehmer haben weniger Organisationsaufwand und werden bei ihrer Gruppenarbeit unterstützt. Zusätzlich muss kein Kursteilnehmer eigenen Speicherplatz zur Verfügung stellen.


## Vorgegebene Schnittstelle
Um die User Story zu realisieren haben wir ein Aktivity Plugin für Moodle entwickelt. Instanzen von Aktivity Plugins können Kursen beliebig oft hinzugefügt werden.
Für genauere Informationen besuchen sie die Moodle Dokumentation von [Aktivity modules](https://docs.moodle.org/dev/Activity_modules "Activity Modules")
In der `mod_form.php` fragen wir alle Einstellungen ab, die vor dem Erstellen der Ordner bekannt sein müssen. Dies beeinhaltet Name des Ordners in Moodle, Zugriff des Lehrenden auf die erstellten Ordner und ob für Gruppen seperate Ordner erzeugt werden sollen.
In der `settings.php` kann der Administrator den technischen Nutzer des Plugins festlegen. Dieser gilt für alle Instanzen, also global für das gesammte Plugin.
Die `lib.php` bietet eine Schnittstelle um auf das Hinzufügen, Ändern und Löschen von Instanzen zu reagieren.
<div class="alert alert-danger">
  <strong>TODO:</strong> nicht zu genau beschreiben die moodle Dokumentationist sehr gut lieber auf die Implementation mehr eingehen.
</div>

## Implementierung
### Anmelden des technischen Nutzers

Der Admin der Moodle Seite kann in der Seiten Administration einen technischen Nutzer hinzufügen. Über `Site administration ► Plugins ► Activity modules ► collaborativefolders` kommt er zu den passenden Einstellungen. Dort wird er durch einen Login-Button auf die ownCloud Seite zum autorisieren der App weitergeleitet. Spätere Änderungen des technischen Nutzers sind durch ein Logout Button möglich. Dies ist nicht empfohlen und mit einer Warnmeldung versehen, da Kompilierungs-Problemen mit bestehenden Instanzen entstehen würden. Wir haben uns dafür entschieden den Logout trotzdem bereitzustellen, für den Fall das ein falscher Account authentifiziert wird. Hierbei sind wir insbesondere über folgendes Szenario gestoßen:

Der Seiten Administrator will den technischen Nutzer einloggen, ist aber noch mit seinem eigenen ownCloud Account oder dem Administrator account der ownCloud authentifiziert. Er bemerkt nicht, dass er mit dem falschen Account eingeloggt ist autorisiert das Plugin. Als er seinen Fehler bemerkt, möchte er den technischen Nutzer so schnell wie möglich ändern, obwohl bestehende Instanzen neu erstellt werden müssen. 

Der Administrator kommt

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
