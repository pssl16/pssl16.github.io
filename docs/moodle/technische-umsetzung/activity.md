# Aktivität: `collaborative folders`

## Zweck

Das Aktivitäts Modul *Collaborative Folders* soll Lehrenden die Möglichkeit geben, für Studierende oder Gruppen von Studierenden Ordner für kollaboratives Arbeiten zu erstellen.
Die Autorisierung und Authentifizierung erfolgt über das *oauth2sciebo* admin tool. Diese Aktivität implementiert die User Story 4:

<ol start="4">
  <li>Als <b>Lehrender</b> möchte ich Studierenden oder Gruppen von Studierenden Ordner für kollaboratives Arbeiten bereitstellen.</li>
</ol>

Der Lehrende kann einem Kurs beliebig viele Instanzen der Aktivität *Collaborative Folders* hinzufügen. In den Einstellungen kann der Lehrende
nun festlegen wie der Ordner im Moodle Kurs benannt werden soll und wie der Ordner in der Sciebo Instanz benannt werden soll.

![Settings for the activity collaborativefolders](activity_settings/settings.svg "Settings for the activity collaborativefolders")

Desweiteren hat der Lehrende die Möglichkeit Ordner nur für einzelne Gruppen zu erstellen. In den Einstellungen werden unter dem Menüpunkt
*Create Shares for single groups* alle Gruppen des Kurses aufgelistet. Mit einer Checkbox können die Gruppen makiert werden, für die pro Gruppe jeweils
ein Ordner erstellt werden soll.

## Vorgegebene Schnittstelle

Um sich grundlegend mit der Struktur von [Aktivity modules](https://docs.moodle.org/dev/Activity_modules "Activity Modules") auseinander zu setzen, benutzen sie die Moodle Dokumentation.
Um das hinzufügen von Ordnern zu ermöglichen muss ein neutraler Speicherplatz für die Ordner gewählt werden. Es bietet sich an einen hypothetischen Nutzer zu erstellen unter dessen Adresse alle
Ordner gespeichert werden. Der Admin der Moodle Seite kann in den Admin Einstellungen einmalig einen Nutzer hinzufügen. Spätere Änderungen sind nicht mehr möglich, da
dies zu Kompilierungsproblemen mit bestehenden Instanzen des Moduls führen würde.

Die User Story erfordert, zwei Szenarien zu betrachten. Im ersten Szenario wird ein Ordner für alle Studierenden eines Kurses erstellt.
Die Anforderung an das Modul ist also einen Ordner in dem gespeicherten Account zu erstellender eindeutig Identifizierbar ist.
Damit keine Synchronisationskonflikte entstehen, erstellt das Modul eine Ordner Struktur. Der jeweils erforderliche Ordner wird in einem
Ordner erstellt der nach der collaborativefolders_id benannt ist. Dies ist eine Id die jeder Instanz individuell gegeben wird. So kann es soweit manuell
keine Ordner erstellt werden zu keinen Synchronisationskonflikten kommen.

``` php
$directory = $webdavpath . $id;
            $name = $webdavpath . $id . '/' . $foldername;

            // If one of the folders could not be created, false is returned.
            if (($this->sciebo->make_folder($directory)) != 201) {
                return false;
            }
            if (($this->sciebo->make_folder($name)) != 201) {
                return false;
            }
```

Im zweiten Szenario sollen Ordner nur für Gruppen innerhalb eines Kurses erstellt werden. Zu diesem Zweck gibt es den Menüpunkt *Create Shares for single groups*.
Hier wird für jede Gruppe ein einzelnes Checkbox Element erstellt.

``` php
foreach ($arrayofgroups as $id => $group){
            $mform->addElement('advcheckbox', $group['id'] , $group['name'], ' Number of participants: ' . $group['numberofparticipants'], array(), array(0, 1));
        }
```

Beim Auswerten der Form wird nun überprüft, ob eines der Elemente ausgewählt wurde. Ist dies
der Fall wird nur der entsprechenden Gruppe oder den entsprechenden Gruppen ein Ordner erstellt.

## Implementierung

## Tests und Continuous Integration
