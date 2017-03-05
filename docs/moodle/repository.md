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

## Tests und CI
