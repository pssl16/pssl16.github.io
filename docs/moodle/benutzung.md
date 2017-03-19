# Benutzung

Plugins können in Moodle zusätzlich zum *moodle core* installiert werden. Im Folgenden wird beschrieben, wie die Plugins heruntergeladen werden können und anschließend in den einzelnen Plugins die richtigen Einstellungen getätigt können. 
Zusätzlich werden verschiedene Ansichten der Plugins dargestellt um die Funktionalitäten der jeweiligen Plugins zu verdeutlichen.

## Download
Die Plugins sind zurzeit in einem öffentlichen *GitHub repository* verfügbar.
Von dort aus können die Plugins auf zwei Wegen heruntergeladen werden.

1. Als Zip-Datei
    Am rechten Rand in dem Menü `clone or download` auf **Download ZIP** klicken. In Moodle können ZIP-Ordner einfach installiert werden, indem sie vom Seiten-Administrator unter dem Menüpunkt `Website-Administration Plugins Plugin installieren` hochgeladen werden. Das Plugin wird automatisch richtig platziert.

2. Mit dem Befehl `git clone`
    In der Moodle Instanz in den zugehörigen Ordner navigieren. Bei Unsicherheiten können die richtigen Ordner in den Beschreibungen der einzelnen Plugins nachgelesen werden. Nun den folgenden Befehl ausführen:
    ```
    git clone git@github.com:pssl16/name_of_the_repository.github.git
    ```
    Nun befindet sich der Inhalt an der richtigen Stelle, der Ordner muss nur noch umbenannt werden. Das letzte Wort des Ordners ist der Name des Plugins und muss auch der Name des Ordners sein.

Der Vorteil des zweiten Weges ist, dass mittels `git pull` Änderung schnell nachgepflegt können. Bei Unerfahrenheit mit *git* sollte jedoch lieber der erste Weg gewählt werden.

Nach Herunterladen und richtiger Platzierung der Plugins werden dem Administrator der Moodle-Website im Plugin Manager die neuen verfügbaren Updates angezeigt.
Um die Plugins zu benutzen wird in jedem Fall das `oauth2owncloud_admin_tool`benötigt , alle anderen Plugins lassen sich nicht ohne dieses installieren. Dies ist bei der Reihenfolge der Installation bei Installation der Plugins als ZIP-Ordner zu beachten.


### Icons 
Die vorgegebenen Icons können einfach ersetzt werden, indem die in `plugin/pix/icon.svg` vorhandene Datei durch eine gleichnamige Datei ausgetauscht wird. Es können auch andere Formate als *.svg* verwendet werden. Das Bild darf jedoch nicht die Größe von 16x16 px überschreiten.

## Admin Tool oauth2owncloud
[*Downloadlink*](https://github.com/pssl16/moodle-tool_oauth2owncloud)
### Admin Einstellungen

Damit das OAuth 2.0 Protokoll reibungslos ablaufen kann, muss zuerst der Client in den Einstellungen registriert werden.

Hierfür muss der Administrator das Formular des Plugins, das unter `Website-Administration ► Plugins ► Authentifizierung ► Sciebo OAuth 2.0 Configuration` zu finden ist, ausfüllen.

<img class='moodleimage' alt="OAuth 2.0 Formular" src="../images/OAuth2Form.png" width=70%>

Als erstes Feld muss die Client ID eingegeben werden. Diese findet man in ownCloud, sobald ein neuer Client registriert wurde. Dasselbe gilt für das nächste Feld, hier wird das Secret angegeben, dass sich auch aus der ownCloud App kopieren lässt.

<img class='moodleimage' alt="WebDAV Formular" src="../images/WebDAVForm.png" width=100%>

Nun werden die Einstellungen für den WebDAV Zugriff festgelegt.
Als erstes wird die Adresse des ownCloud Servers angegeben.
Im nächsten Feld wird der Pfad zur WebDAV Schnittstelle angegeben in ownCloud endet diese typischerweise mit `remote.php/webdav/`.
Als Protokolltyp kann *http* oder *https* angegeben werden. Wird keine Angabe gemacht, so wird von *https* ausgegangen.
Als letztes kann der Port angegeben werden.

## Repository sciebo
[*Downloadlink*](https://github.com/pssl16/moodle-repository_owncloud)

### Admin Einstellungen
Sobald das Admin tool installiert wurde, kann das Repository installiert werden. Es ist zu beachten, dass die oben genannten Einträge getätigt wurden, da ansonsten die Authentifizierung des Repositorys nicht funktioniert. Repository Plugins müssen in Moodle von einem Administrator unter dem Menüpunkt `Website-Administration ► Plugins ► Repositories ► Übersicht` aktiviert werden. Der Administrator kann dem Repository zusätzlich unter `Einstellungen` einen globalen Namen geben.

### Nutzer Sicht

Das Repository ist sowohl in den Kursen als auch für private Instanzen verfügbar und muss nicht mehr hinzugefügt werden. Kurs-Administratoren können das Repository jedoch unter `Speicherorte` löschen. Die Nutzung lässt sich nicht auf bestimmte Nutzer oder Aktivitäten im Kurs einschränken. Im File Picker muss man sich zunächst anmelden.

<img class='moodleimage' alt="FilePicker" src="../images/FilePickerlogin.png" width=70%>

Wird der Button betätigt, erscheint ein Pop-up Window oder ein neuer Tab, der den Nutzer auffordert sich in ownCloud anzumelden. Anschließend wird der Nutzer gefragt ob er die App autorisieren möchte. Der Nutzer wird nun zurückgeleitet und sieht eine tabellarische Auflistung der vorhandenen Dateien:

<img class='moodleimage' alt="FilePicker" src="../images/05.png" width=70%>

Im roten Kasten sieht der Nutzer Buttons um den Inhalt neu zu laden und sich auszuloggen. Nur als Admin sieht man den letzten Button, mit dem man die Einstellungen des OAuth2 admin_tool bearbeiten kann.

## collaborative_folders
[*Downloadlink*](https://github.com/pssl16/moodle-mod_collaborativefolders)

### Einstellungen
#### Admin Einstellungen
Für die Aktivität collaborative_folders wird ein technischer Nutzer der ownCloud Instanz benötigt. Bei diesem Nutzer werden alle Ordner, die erstellt werden, gespeichert. Um den Nutzer festzulegen muss in `Website-Administration ► Plugins ► Aktivitäten ► collaborativefolders` ein technischer Nutzer mit Hilfe des OAuth 2.0 Protokolls authentifiziert werden. Über einen Login Button muss sich in ownCloud authentifiziert werden. Bei nicht erfolgter Weiterleitung sind die Einstellungen im Admin tool oauth2owncloud fehlerhaft und diese sind zu überprüfen. Es ist darauf zu achten, dass sich nicht mit dem regulären privaten Account, sondern mit dem des technischen Nutzers angemeldet wird.

#### Sicht des Lehrenden
Die Aktivität ist in jedem Moodle Kurs verfügbar. Wenn ein Lehrender die Aktivität dem Kurs hinzufügt muss er dem Ordner einen Namen für die Moodle Instanz und einen für die ownCloud Instanz geben. Danach kann festlegt werden, ob Lehrende des Kurses Zugriff auf alle erstellten Ordner haben. Eins der wichtigsten Integrationsszenarien ist, dass nur für Gruppen  von Studierenden ein Ordner erstellt wird. Dies ist möglich, wenn der Lehrende den Zugriff auf bestimmte Gruppen beschränkt. In diesem Fall werden nur für die gewählten Gruppen einzelne Ordner erstellt.


#### Sicht der Studierenden
Wenn ein Ordner für einen Studierenden freigegeben wurde, sieht dieser die Aktivität in dem Kurs. Ordner werden von dem Plugin zeitverzögert erstellt um den Server nicht zu überlasten. Somit kann es passieren, dass der Nutzer, wenn er die Aktivität auswählt, eine Nachricht bekommt, dass die Ordner noch nicht erstellt wurden. Sobald die Ordner erstellt wurden, wird der Nutzer aufgefordert dem Ordner einen individuellen Namen
zu geben, wenn er die Aktivität auswählt. Dieser Name wird nur für den derzeitigen Nutzer gespeichert.

<img class='moodleimage' alt="Name Form" src="../images/NameForm.png" width=80%>

Danach kann der Nutzer unter dem Menüpunkt `Change Folder Name` den gewählten Namen ändern. Wenn der Nutzer noch nicht eingeloggt ist, kann er sich über einen Login Button authentifizieren. Ansonsten kann er sich aus dem aktuellen Account ausloggen und sich mit einem anderen Account authentifizieren.
Wenn er authentifiziert ist, kann der Ordner unter dem Menüpunkt `Generate Link to Folder` dem privaten Account hinzugefügt werden.

<img class='moodleimage' alt="AddForm" src="../images/AddForm.png" width=60%>

Nun kann der Nutzer über einen Link in Moodle oder über den normalen ownCloud Login auf
den Ordner zugreifen.
