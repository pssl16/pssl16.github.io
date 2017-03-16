# Softwarearchitektur

Da wir möglichst keine neue Schnittstelle implementieren wollten, war es das Ziel, die bestehenden Schnittstellen in ownCloud um OAuth 2.0 zu erweitern. Zunächst war dafür die Implementierung des OAuth 2.0 Protokolls mit seinen Schnittstellen notwendig. Außerdem musste zur Umsetzung der [Integrationsszenarien](../..) der Zugriff auf [WebDAV](https://doc.owncloud.org/server/9.1/user_manual/files/access_webdav.html) und die [OCS Share API](https://doc.owncloud.org/server/9.1/developer_manual/core/ocs-share-api.html) über OAuth 2.0 bereitgestellt werden.

## Bestandteile

### OAuth 2.0 App

Zur Implementierung von OAuth 2.0 entschieden wir uns in Absprache mit einem ownCloud Entwickler, eine [ownCloud App](https://doc.owncloud.org/server/9.1/developer_manual/app/) namens `oauth2` zu erstellen. Der Vorteil einer solchen Lösung ist, dass OAuth 2.0 zu einer bestehenden ownCloud Installation einfach hinzugefügt und auch wieder entfernt werden kann. Die Möglichkeit, [RESTful APIs](https://doc.owncloud.org/server/9.1/developer_manual/app/api.html) und eigene [Datenmodelle](https://doc.owncloud.org/server/9.1/developer_manual/app/schema.html) in Apps zu definieren, stellten alle notwendigen Werkzeuge für die Umsetzung des OAuth 2.0 Protokolls bereit.

### Core Anpassungen

Zur Erweiterung der WebDAV Schnittstelle um OAuth 2.0 wurde die Implementierung von WebDAV im [ownCloud Core](https://github.com/owncloud/core) untersucht. 
Es stellte sich heraus, dass hierfür auch eine ownCloud App namens `dav` verwendet wurde, die die [sabre/dav](http://sabre.io) Bibliothek zur Implementierung des WebDAV-Serves nutzt. 
Da die Bibliothek den Austausch der Authentifizierung über eigene Authentication Backends unterstützt, entschieden wir uns für die Implementierung eines Backends für OAuth 2.0, 
das wir ebenfalls über die `oauth2` App bereitstellen wollten. Um eigene Authentication Backends für die `dav` App bekannt zu machen und gleichzeitig keine Abhängigkeit zwischen den Apps zu schaffen, 
nutzten wir Event Listener. Notwendig dafür war die Registrierung des OAuth 2.0 Backends durch die `oauth2` App. Außerdem musste die `dav` App dahingehend angepasst werden, zusätzliche Authentication Backends zu laden. Hierfür wurde der Pull Request [owncloud/core#26742](https://github.com/owncloud/core/pull/26742) erstellt.

Schließlich musste noch die OCS Share API um OAuth 2.0 erweitert werden. Die Implementierung dieser API wird größtenteils durch die ownCloud App `files_sharing` bereitgestellt. Daher mussten Änderungen in der Authentifizierung auf Ebene von ownCloud Apps durchgeführt werden. Das bedeutete, dass auch hierfür Änderungen am ownCloud Core notwendig waren. Gleichzeitig sollten aber keine Abhängigkeiten zwischen dem Core und der `oauth2` App geschaffen werden. Da keine entsprechende Lösung im Core existierte, folgten wir dem Vorschlag eines ownCloud Entwicklers für die Implementierung eines Plugin Mechanismus. Auch diese Änderungen sind im Pull Request [owncloud/core#26742](https://github.com/owncloud/core/pull/26742) zu finden.

Folgende Tabelle fasst die Funktionen der einzelnen Bestandteile zusammen.

| Bestandteil                          | Funktion                                                                                                                                |
|--------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------|
| [OAuth 2.0 App](oauth2-app)          | Implementierung des OAuth 2.0 Protokolls mit der notwendigen Authentifizierungslogik für die WebDAV Schnittstelle und die OCS Share API |
| [Core Anpassungen](core-anpassungen) | Erweiterung der `dav` App und der Authentifizierung für ownCloud Apps um das Laden benutzerdefinierter Authentifizierungsmethoden       |
