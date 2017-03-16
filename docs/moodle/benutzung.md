# Benutzung

Plugins können in Moodle zusätzlich installiert werden. Auf der [offiziellen Seite](https://moodle.org/plugins/ "Moodle Plugin Page")
von Moodle können alle zurzeit angebotenen Plugins gefunden werden. Für mehr Informationen zur allgemeinen Installation von Moodle Plugins
benutzen sie die Dokumentation der [Moodle Seite](https://docs.moodle.org/25/de/Plugins_installieren "Installation von Moodle Plugins").
## Installation
Wenn sie die Plugins heruntergeladen haben und richtig platziert haben zeigt der Moodle Plugin Manager Ihnen die verfügbaren Plugins an.
Sie benötigen in jedem Fall das oauth2_admin_tool, alle anderen Plugins sind von diesem abhängig.

## Admin Tool oauth2sciebo

### Admin Einstellungen

Damit das OAuth 2 Protokoll reibungslos ablaufen kann, muss zuerst der Client in den Einstellungen registriert werden.

Hierfür muss der Administrator das Formular des Plugins, das unter `Website-Administration>Plugins>Authentifizierung>Sciebo OAuth 2.0 Configuration` zu finden ist, ausfüllen.

<img alt="OAuth 2.0 Formular" src="../images/OAuth2Form.png" width=100%>

Als erstes Feld muss die Client ID eingegeben werden. Diese findet man in ownCloud, sobald ein neuer Client registriert wurde. Dasselbe gilt für das nächste Feld, hier wird das Secret angegeben, dass sich auch aus der ownCloud App kopieren lässt.

<img alt="WebDAV Formular" src="../images/WebDAVForm.png" width=100%>

Nun werden die Einstellungen für den WebDAV Zugriff festgelegt.
Als erstes wird die Adresse des ownCloud Servers angegeben.
Im nächsten Feld wird der Pfad zur WebDAV Schnittstelle angegeben in ownCloud endet diese typischerweise mit `remote.php/webdav/`.
Als Protokolltyp kann http oder https angegeben werden. Wenn keine Angabe gemacht wird, wird von https ausgegangen.
Als letztes kann der Port angegeben werden.

## Repository sciebo

### Admin Einstellungen
Sobald das Admin tool installiert wurde, kann das Repository installiert werden. Bitte beachten Sie, dass die oben genannten Einträge getätigt wurden, ansonsten funktioniert die Authentifizierung des Repositorys nicht. Repositorys Plugins müssen in Moodle von einem Administrator unter dem Menüpunkt `Website-Administration>PluginsRepositories>Übersicht` aktiviert werden. Der Administrator kann dem Repository zusätzlich unter `Einstellungen` einen globalen Namen geben.

### Nutzer Sicht

Das Repository ist sowohl in den Kursen als auch für private Instanzen verfügbar und muss nicht mehr hinzugefügt werden. Kurs Administratoren können das Repository jedoch unter `Speicherorte` löschen. Die Nutzung lässt sich nicht auf bestimmte Nutzer oder Aktivitäten im Kurs einschränken. Im File Picker muss man sich zunächst anmelden.
Danach sieht man nach der Anmeldung eine tabellarische Auflistung der vorhandenen Dateien:

<img alt="FilePicker" src="../images/FilePickerredblock.png" width=100%>

Im roten Kasten sehen sie Buttons um den Inhalt neu zu laden, sich auszuloggen und nur als Admin sieht man den letzten Button in dem man die Einstellungen des OAuth2 admin_tool bearbeiten kann.

## collaborative_folders

### Einstellungen
#### Admin Einstellungen
Für die Aktivität collaborative_folders wird ein technischer Nutzer der ownCloud Instanz benötigt. Bei diesem Nutzer werden alle Ordner die erstellt werden gespeichert. Um den Nutzer festzulegen muss in `Website-Administration>Plugins>Aktivitäten>collaborativefolders` ein technischer Nutzer mit Hilfe des OAuth2 Protokolls authentifizieren. Über einen Login Button werden Sie aufgefordert sich in ownCloud zu authentifizieren. Falls Sie nicht richtig weitergeleitet werden, sind die Einstellungen im Admin tool oauth2sciebo fehlerhaft, bitte überprüfen Sie diese. Achten Sie darauf, dass Sie sich nicht mit Ihrem normalen Account sondern mit dem technischen Nutzer anmelden.

#### Sicht des Lehrenden
Die Aktivität ist in jedem Moodle Kurs verfügbar. Wenn ein Lehrender die Aktivität dem Kurs hinzufügt muss er dem Ordner einen Namen für die Moodle Instanz und einen für die ownCloud Instanz geben. Danach kann festlegt werden ob Lehrende des Kurses Zugriff auf alle erstellten Ordner haben. Eins der wichtigsten Integrationsszenarien ist, dass nur für Gruppen  von Studierenden ein Ordner erstellt wird. Dies ist möglich, wenn der Lehrende den Zugriff auf bestimmte Gruppen beschränkt. In diesem Fall werden nur für die gewählten Gruppen einzelne Ordner erstellt.


#### Sicht der Studierenden
Wenn ein Ordner für einen Studierenden freigegeben wurde, sieht dieser die Aktivität in dem Kurs. Wählt er diese aus, kann er sich den Ordner herunterladen, oder sich den Ordner in seine Instanz duplizieren. Hat der Studierende keinen Account in der ownCloud Instanz kann er den Ordner herunterladen jedoch nichts in den Ordner laden.
