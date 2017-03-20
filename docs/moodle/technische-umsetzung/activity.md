<div><img alt="" align=right src="../../images/icon_mod_collaborativefolders.svg" width=20% position=right>
<h1> Aktivität: <span class=code>collaborativefolders</span></h1>
</div>


## Zweck

Das Aktivitäts Modul **Collaborative Folders** soll Lehrenden die Möglichkeit geben, für Studierende oder Gruppen von Studierenden Ordner für kollaboratives Arbeiten zu erstellen.
Sowohl die Autorisierung und Authentifizierung, als auch der Zugriff auf ownCloud Schnittstellen erfolgen über das `oauth2owncloud` [Admin Tool](admin-tool/). Somit implementiert dieses Plugin das folgende Integrationsszenario:

> Als **Lehrender** möchte ich Studierenden oder Gruppen von Studierenden Ordner für kollaboratives Arbeiten bereitstellen.

Dabei soll der Lehrende beliebig viele Instanzen der Aktivität erstellen können bei denen er jeweils selbst entscheiden kann, ob er Ordner für einzelne Gruppen freigibt und ob er selbst auf die erstellten kollaborativen Ordner Zugriff haben möchte. Zusätzlich sollen die Teilnehmer der Aktivität jeweils selbst über den Namen des Ordners und dessen Freigabe für sie entscheiden können.

Mit dieser Aktivität kann der Lehrende Kursteilnehmer ermutigen kollaborativ zu arbeiten. Zusätzlich hat er die Option die laufende Arbeit zu betreuen, indem er sich selbst Zugriff auf die Ordner gewährt. Somit haben kursteilnehmer insgesamt weniger Organisationsaufwand und werden bei ihrer Gruppenarbeit unterstützt.

## Speicherort und Zugriff

Der zur Lagerung der erstellten kollaborativen Ordner genutzte Speicherplatz sollte neutral und unabhängig von einzelnen, Nutzer-spezifischen ownCloud Instanzen. Diese Vorüberlegung geht aus den folgenden Gründen hervor:

* Der **Lehrende** muss nicht zwingend Zugriff auf die Ordner haben.
	* Seine persönliche Instanz ist damit ungeeignet.
* Für den benötigten Speicherplatz könnte eine persönliche Instanz nicht ausreichen.
	* **Kapazitäten** sind häufig begrenzt.
* Die gespeicherten Daten sollen **langfristig** erhalten bleiben.
	* Dies ist besonders wichtig, da die Ordner, obgleich mit den Nutzern geteilt, von dem Zustand des **geteilten Ordners** abhängig sind.

Daraus ergibt sich, dass für die Speicherung der kollaborativen Ordner zunächst ein neutraler Speicherort angegeben muss. Dieser Speicherort muss sich auf dem selben ownCloud Server befinden um das Teilen von Inhalten zwischen diesem Speicherort und den Ordnernutzern zu gewährleisten. Dabei werden die nötigen Server-Angaben dem OAuth 2.0 ownCloud Client entnommen, welcher auch für Nutzer-seitige Zugriffe verantwortlich ist. 

Bevor von einem Lehrenden ein kollaborativer Ordner erstellt werden kann, muss der Administrator zunächst für das Plugin global eben diesen neutralen Speicherort in Form eines Nutzer-Accounts hinterlegen. In dessen Namen werden anschließend alle kollaborativen Ordner erstellt und mit den entsprechenden Nutzern geteilt. Im Folgenden wird dieser Nutzer-Account als **technischer Nuter** bezeichnet.

Um nun auf die erstellen Ordner zugreifen zu können, müssen diese von dem technischen Nutzer mit den Personen geteilt werden, welche zum Zugriff zuvor berechtigt worden sind. Dabei kann es sich um alle Kursteilnehmer oder nur spezifische Gruppen einschließlch oder ausschließlich dem Lehrenden handeln. Einerseits bringt dies den Vorteil, dass kein Speicherplatz in den persönlichen ownCloud Instanzen für die Gruppenordner gebraucht wird; andererseits birgt es aber auch den Nachteil, dass der geteilte Ordner gelöscht werden kann, ohne dass die Nutzer jeweils eine Kopie den gespeicherten Daten erhalten. Daher ist es notwendig, dass der technische Nutzer die Daten über einen längeren Zeitraum hinweg sichern kann.

## Vorgegebene Schnittstelle

Das Integrationsszenario wurde im Rahmen eines [Activity Modules](https://docs.moodle.org/dev/Activity_modules "Activity Modules") in Moodle umgesetzt. Instanzen solcher Plugins können in Kursen von einem Lehrenden beliebig oft hinzugefügt werden und stellen für die Studierenden eine Interaktion innerhalb eines Kurses dar.

Ähnlich wie bei anderen Moodle Plugins, muss zunächst eine mindestens erforderliche Schnittstelle inerhalb der vorgegebenen Ordner und Dateien implementiert werden, welche das Plugin mit dem Moodle Core verbindet. Die für `collaborativefolders` relevantesten Dateien sind die folgenden:

* **`mod_form.php`:** Definiert ein Formular, dessen Eingaben vor dem Erstellen eines Ordners bekannt sein müssen.
	* Name der Aktivität, Zugriffberechtigungen (**Lehrender** und **Gruppen**), [Gruppenmodus](https://docs.moodle.org/32/en/Groups#Group_modes)
* **`settings.php`:** Beinhaltet eine Einstellungsseite für globale Konfigurationen von `collaborativefolders`.
	* Wird verwendet um den technischen Nutzer zu verwalten.
* **`lib.php`:** Bietet eine Schnittstelle zum Erstellen, Ändern und Löschen von Instanzen der Aktivität.
	* Definiert, was im Fall einer der Operationen getan werden muss.
* **`view.php`:** Konstruiert die Ansichtsseite der Aktivität abhängig von dem Status der Aktivität und der Berechtigungen des aktuellen Nutzers.

Sonstige Standarddateien werden auch in der Dokumentation zum [Admin Tool](admin-tool/#vorgegebene-schnittstelle) beschrieben.

## Implementierung

Da die Implementierung der vorgegebenen Schnittstelle von Zugriffen auf ownCloud und zusätzlichen Hilfsklassen abhängt, umfasst sie vorgegebenen und nicht-vorgegebenen Schnittstellen und Inhalte gemeinsam. 

### Zugriff auf ownCloud

Zur Umsetzung des Integrationsszenarios ist Zugriff zu ownCloud notwendig. Der zuvor implementierte OAuth 2.0 ownCloud Client bietet bereits die Möglichkeit zur Weiterleitung von WebDAV und OCS Share API Anfragen. Einige von diesen Anfragen wurden, gebündelt in der speziell dafür vorgesehenen Klasse `owncloud_access`, zusammengefasst um die Verwendung dieser zu vereinfachen und von der technischen Ebene zu abstrahieren. Die folgenden Funkionen wurden dabei implementiert und werden von dieser Aktivität verwendet:

* **`handle_folder`:** Diese Methode kann benutzt werden um einen Ordner, welcher dem technischen Nutzer der Aktivität gehört, in ownCloud zu erstellen oder zu löschen.
* **`generate_share`:** Mit Hilfe dieser Methode wird für einen Nutzer ein Ordner aus dem ownCloud Verzeichnis des technischen Nutzers freigegeben.
* **`rename`:** Diese Methode benennt einen Ordner im ownCloud Verzeichnis des aktuellen Nutzers um.
* **`share_and_rename`:** Hier werden die Methoden `generate_share` und `rename` vereint.

Die Methoden liefern bei Erfolg jeweils das gewünschte Ergebnis und im Fall eines Fehlschlags eine angemessene Fehlermeldung.

### Anmelden des technischen Nutzers

Der Administrator der Moodle Seite kann in der Seiten-Administration einen technischen Nutzer hinzufügen. Über `Website-Administration ► Plugins ► Aktivitäten ► collaborativefolders` kommt er zu den entsprechenden Einstellungen. An dieser Stelle kann er mittels eines Login-Links einen technischen Nutzer authentifizieren und autorisieren. Damit wird der technische Nutzer mit allen Instanzen der `collaborativefolders` Aktivität [verknüpft](activity/#speicherort-und-zugriff). Zwar sind spätere Änderungen des technischen Nutzers durch einen Logout möglich, jedoch ist dies nicht empfohlen und mit einer Warnmeldung versehen, da Kompilierungs-Probleme mit bestehenden Instanzen entstehen würden. Für den Fall, dass im Moment des Logins zum Bespiel ein falscher Nutzer in ownCloud authentifiziert ist, wird der Logout des technischen Nutzer dennoch bereitgestellt. Das folgende Szenario begründet die Entscheidung:

> Der Seiten Administrator will den technischen Nutzer einloggen, ist aber noch mit seinem eigenen ownCloud Account oder dem Administrator Account in ownCloud authentifiziert. Er bemerkt nicht, dass er mit dem falschen Account eingeloggt ist und autorisiert das Plugin. Als er seinen Fehler bemerkt, möchte er den technischen Nutzer so schnell wie möglich ändern, obwohl bestehende Instanzen neu erstellt werden müssen.

Die Verwaltung des Login-Status übernimmt intern der OAuth 2.0 ownCloud Client, welcher innerhalb der `settings.php` in die Einstellungsseite des Plugins eingebettet ist. Abhängig von dem Login-Status des technischen Nutzers werden durch den Client verschiedene Operationen durchgeführt und dem Administrator dementsprechend verschiedene Möglichkeiten angeboten. Das folgende Codebeispiel zeigt abstrahiert, wie der Client agiert.

```php
if ($logout === true) {

	set_config('token', null, 'mod_collaborativefolders');
	print_login();

} else {

	if ($owncloud->check_login('mod_collaborativefolders')) {

    		print_logout();


	} else {

    		set_config('token', null, 'mod_collaborativefolders');
    		print_login();

	}
}
```

Die Variable `$owncloud` stellt in dem Code den Client dar. Falls zuvor der Logout-Link betätigt worden ist, enthält die Variable `$logout` den Wert `true`. Die folgenden Szenarien stellt das Codebeispiel dar:

1. **Der technische Nutzer ist nicht angemeldet.**

    Falls der technische Nutzer noch nicht angemeldet worden ist, wurde für ihn auch noch kein [Plugin-spezifisches](admin-tool/#speicherung-nutzer-spezifischer-access-tokens) Access Token hinterlegt. Wenn nun also der Client prüft, ob das Access Token noch gültig, ist, wird er kein Token vorfinden und daher `false` zurückgeben. Folgerichtig wird ein Login-Link für den technischen Nutzer angezeigt.

2. **Der technische Nutzer ist angemeldet.**

    Das Token des technischen Nutzers wird in Plugin-spezifischen Einstellungen gespeichert, da es zu keinem persönlichen Nutzer, sondern zum Plugin gehört. Dieses Szenario wird sowohl in dem Fall, dass das bereits erhaltene Access Token gültig ist, als auch beim Erhalt eines gültigen Authorization Codes aufgerufen. Wenn der Client das aktuelle Access Token also für gültig erklärt, gilt der technische Nutzer als eingeloggt und der Administrator darf ihn bei Bedarf ausloggen.

3. **Der technische Nutzer wird ausgeloggt.**

    Es wird registriert, dass der Logout-Link betätigt wurde (`$logout === true`). Das aktuelle Access Token des technischen Nutzers wird daraufhin in den Einstellungen gelöscht und dem Administrator erneut ein Login-Link angezeigt. Zusätzlich wird ein Event ausgelöst, welches den Logout dokumentiert.

### Hinzufügen einer Instanz

Die Schnittstelle in Moodle für das [Erstellungsformular](https://docs.moodle.org/dev/Activity_modules#mod_form.php) einer Aktivität ist sehr ausführlich. Den Standardeinstellungen wurden nur ein Textfeld zur Eingabe des gewünschten, angezeigten Namens der Aktivität und eine Checkbox hinzugefügt, die festlegt, ob der Lehrende Zugriff auf die Ordner haben soll. 
Unter den bereits vorgegebenen Einstellungen für die Erstellung einer Instanz der Aktivität, wird vor allem von dem Moodle-internen [Gruppenmodus](https://docs.moodle.org/32/en/Groups#Group_modes) in der `collaborativefolders` Aktivität Gebrauch gemacht. Anhand des eingestellten Gruppenmodus wird in dem Plugin bestimmt, welche Ordner erstellt werden müssen und wer zum Zugriff zu der Aktivität und damit den kollaborativen Ordnern haben soll.

Die vorgenommenen Einstellungen werden nach dem Hinzufügen der Instanz durch die Methode `add_instance` der `lib.php` in einer für die Aktivität vorgesehenen Datenbanktabelle hinterlegt.

### Erstellen von Ordnern

Für jede Instanz von `collaborativefolder` muss zunächst ein Hauptordner erstellt werden. Dieser fasst alle anschließend erstellten Gruppenordner, falls eine der Gruppenmodus aktiviert worden ist. Sollte das nicht der Fall sein, ist dieser Hauptordner der einzige kollaborative Ordner, der erstellt wird und mit allen Kursteilnehmern geteilt werden muss. Unabhängig davon, ob der Gruppenmodus aktiv ist, wird dieser Ordner, falls vom Lehrenden gewünscht, für ihn freigegeben um die Arbeit in den/dem kollaborativen Ordner/-n zu beaufsichtigen.

Zusätzlich wird für jede teilnehmende Gruppe ebenfalls ein Ordner erstellt. 

#### Observer

Nachdem die Instanz von einem Lehrenden hinzugefügt worden ist, müssen anhand des angegebenen Gruppenmodus die entsprechenden Ordner in ownCloud über den technischen Nutzer erstellt werden. Zwar sind die Informationen zum Gruppenmodus bereits innerhalb der Methode `add_instance` verfügbar, allerdings hat man an dieser Stelle noch keinen Zugriff auf die [Course Module ID](https://docs.moodle.org/dev/Course_module#Usage), weil die entsprechende Datenbanktabelle noch nicht aktualisiert worden ist. Die Course Module ID (`cmid`) wird verwendet um den übergeordneten Ordner zu benennen, welcher die kollaborativen Gruppenordner beinhaltet. Die `cmid` ist deswegen notwendig, weil sie innerhalb eines Moodle-Lebenszyklus nie doppelt belegt wird und sich daher Ordnernamen in ownCloud nicht wiederholen können. 

Um die Ordner nachgezogen erstellen zu können wurde ein [Observer](https://docs.moodle.org/dev/Event_2#Event_observers) implementiert, der aufgerufen wird wenn eine Instanz der Aktivität `collaborativefolders` erstellt wird.
In `db/events.php` kann der Observer registriert werden.

```php
$observers = array(
        array(
                'eventname'   => '\core\event\course_module_created',
                'callback'    => 'mod_collaborativefolders\observer::collaborativefolders_created',
                'internal'  => false,
                'priority'  => 1000
        )
);
```

Es wird angegeben, dass auf das Event gehört werden soll, welches das Hinzufügen einer Aktivität signalisiert. Daraufhin werden, sobald alle nötigen Datenbankzugriffe erfolgt sind, die Eventdaten an den implementierten Observer weitergeleitet.

Die Aufgabe des Observers besteht zunächst darin die Namen für alle zu erstellenden Gruppen zu ermitteln. Diese werden in einem Array zusammengetragen, wie das folgende Codebeispiel zeigt.

```php
$paths = array();
$paths['cmid'] = $cmid;

list ($course, $cm) = get_course_and_cm_from_cmid($cmid, 'collaborativefolders');

if (groups_get_activity_groupmode($cm) != 0) {

	$groupingid = $cm->groupingid;

	$groups = groups_get_all_groups($course->id, 0, $grid);

	foreach ($groups as $group) {

    		$path = $cmid . '/' . $group->id;
    		$paths[$group->id] = $path;
	}
}
```

Falls ein Gruppenmodus für die betreffende Instanz der Aktivität aktiv ist, werden die teilnehmenden Gruppen ermittelt. Hierzu muss geprüft werden, ob ein [Grouping](https://docs.moodle.org/32/en/Groupings) in der Aktivität gewählt worden ist. Sollte dies der Fall sein, werden alle Gruppen aus dem Grouping einbezogen, ansonsten alle Gruppen des Kurses.

Sobald alle teilnehmenden Gruppen ermittelt worden sind, wird ein [Ad Hoc Taks](https://docs.moodle.org/dev/Task_API#Adhoc_tasks) mit der Erstellung der entsprechenden Ordner im ownCloud Verzeichnis des technischen Nutzers beauftragt.

#### Ad Hoc Task

Der aufgerufene Ad Hoc Task befindet sich in der Datei `classes/task/collaborativefolders_create.php`. Beim Aufruf erhält er bereits ein Array mit allen zu erstellenden Ordnernamen. Die Aufgabe des Tasks ist nun die Ordner einzeln zu erstellen. Zu diesem Zweck wird mit jedem Ordnernamen die Methode `handle_folder` der Klasse [`owncloud_access`](activity/#zugriff-auf-owncloud) aufgerufen. Das folgende Codebeispiel zeigt die Schleife, welche dazu benötigt wurde.

Dieser liest aus den Daten, die er übergeben bekommt, alle Ordner aus, die erstellt werden müssen.
``` php
foreach ($folderpaths as $key => $path) {
	$code = $oc->handle_folder('make', $path);
        if ($code == false) {
            throw new \coding_exception('Folder ' . $path . ' not created. 
		The WebDAV socket could not be opened.');
        } else {
            mtrace('Folder: ' . $path . ', Code: ' . $code);
            if (($code != 201) && ($code != 405)) {
                throw new \coding_exception('Folder ' . $path . ' not created. 
			An unexpected status code was received.');
            }
        }
}
```

Die Variable `$folderpath` stellt dabei das Array mit den Ordnernamen und `$oc` ein Objekt der Klasse `owncloud_access` dar.

Falls der Zugriff auf ownCloud nicht erfolgreich durchgeführt werden kann, wird der Ad Hoc Task abgebrochen und eine entsprechende Fehlermeldung geworfen. Solange der Task fehlschlägt, wird er regelmäßig wiederholt bis jeder Ordner erstellt worden ist. Zusätzlich wird der Statuscode, welcher von ownCloud als Antwort ankommt angezeigt. Im Fall eines Erfolges wird ein Event geschaltet, welches dokumentiert, dass die Aktivität nun zur Benutzung zur Verfügung steht.

### Ansicht der bestehenden Instanzen

Nun, da der Lehrende alle notwendigen Einstellungen tätigen konnte, musste die Ansicht der Kursteilnehmer auf die Aktivität mit allen notwendigen Funktionalitäten implementiert werden. Dies beinhaltet die individuelle Namensvergabe für Ordner in ownCloud und das Hinzufügen dieser Ordner zur eigenen Instanz.
Lehrende sollen entweder eine Übersicht aller Ordner haben, oder keinen Zugriff auf die Ordner haben. Alle Funktionalitäten sind in der `view.php` implementiert.

#### Sicht der Studierenden
Für Studierende wird zunächst überprüft, ob der `group_mode` aktiviert ist. Wenn dies der Fall ist, wird an den Pfad, über den später der Ordner gefunden werden soll, die GruppenID angefügt. Moodle hat hierfür intern eine Methode `groups_get_activity_group()` die zu einer Instanz der Aktivität angibt in welcher Gruppe der Studierende ist.
Die `view.php` wird zu verschiedenen Zwecken aufgerufen die behandelt werden müssen.

1. **Der Ordnername wird erstmals gespeichert**

    Der Ordnername wird in den moodle `user_preferences` gespeichert. Diese werden für jeden Nutzer einzeln gespeichert und können beliebig geändert oder gelöscht werden. Die Eingabemaske für einen Ordnernamen wird nur angezeigt wenn bis jetzt kein Name gesetzt wurde oder der Nutzer explizit ausgewählt hat, das der Name zurückgesetzt werden soll.

2. **Der Name des Ordners wird geändert**

    Wenn der Name des Ordners zurückgesetzt wird, wird ein URL Parameter *reset=1* an die URL übergeben. In diesem Fall wird dem Kursteilnehmer eine Eingabemaske angezeigt. Diese ist als eigene Klasse in dem Ordner `collaborativefolders/classes` implementiert. Sie erbt von der abstrakten Klasse `moodleform`. Es muss nun sichergestellt werden das vergebene Namen kompatibel mit ownCloud sind. Moodle unterstützt die zugelassenen Eingaben durch Begrenzung nach *Form Element* Regeln.

    ```
    $mform->addRule('namefield', get_string('err_alphanumeric', 'form'), 'alphanumeric', null, 'client');
    ```

    Diese Regel verbietet andere Eingaben zu speichern, als Buchstaben und Zahlen.

3. **Der Nutzer loggt sich aus seinem aktuell gespeichertem Account aus**
    Der Nutzer muss mit Hilfe des `oauth2owncloud` admin_tools ausgeloggt werden, und der `access token` wird auf null gesetzt.

    ```
    $ocs->owncloud->log_out();
    set_user_preference('oC_token', null);
    ```

4. **Kursteilnehmer rufen die Seite auf, obwohl die Ordner noch nicht vom CronJob erstellt wurden.**

    Für jeden Ordner wird überprüft, ob der Ordner schon erstellt wurde: Zur Information wird dem Nutzer angezeigt, dass die Ordner noch nicht erstellt wurden.

    ```
    $content = json_decode($element->customdata);
    $cmidoftask = $content->cmid;
    if ($id == $cmidoftask) {
        $created = false;
    }
    ```

#### Sicht der Lehrenden

Falls der Lehrende sich selbst keinen Zugriff gewährt hat, sieht er den Ordner auch nicht wenn er in einer Gruppe eingeschrieben ist. Hat der Lehrende Zugriff auf die Ordner, so sieht er eine tabellarische Auflistung aller bestehenden Ordner. Zusätzlich kann er genau wie Studierende seinen ownCloud Account ändern, den Namen des Ordners ändern und Ordner per Klick der eigenen Instanz hinzufügen. In dem Fall, dass Gruppenordner erstellt wurden, wird dem Lehrenden der Überordner aller Gruppenordner hinzugefügt.

## Tests und Continuous Integration
