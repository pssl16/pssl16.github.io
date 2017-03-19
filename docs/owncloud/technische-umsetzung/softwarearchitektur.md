# Softwarearchitektur

Da wir möglichst keine neue Schnittstelle implementieren wollten, war es das Ziel, die bestehenden Schnittstellen in ownCloud um OAuth 2.0 zu erweitern. Zunächst war dafür die Implementierung des OAuth 2.0 Protokolls mit seinen Schnittstellen notwendig. Außerdem musste zur Umsetzung der [Integrationsszenarien](../..#integrationsszenarien) der Zugriff auf [WebDAV](https://doc.owncloud.org/server/9.1/user_manual/files/access_webdav.html) und die [OCS Share API](https://doc.owncloud.org/server/9.1/developer_manual/core/ocs-share-api.html) über OAuth 2.0 bereitgestellt werden.

## Bestandteile

### OAuth 2.0 App

Zur Implementierung von OAuth 2.0 entschieden wir uns in Absprache mit einem ownCloud Entwickler, eine [ownCloud App](https://doc.owncloud.org/server/9.1/developer_manual/app/) namens `oauth2` zu erstellen. Der Vorteil einer solchen Lösung ist, dass OAuth 2.0 zu einer bestehenden ownCloud Installation einfach hinzugefügt und auch wieder entfernt werden kann. Die Möglichkeit, [RESTful APIs](https://doc.owncloud.org/server/9.1/developer_manual/app/api.html) und eigene [Datenmodelle](https://doc.owncloud.org/server/9.1/developer_manual/app/schema.html) in Apps zu definieren, stellten alle notwendigen Werkzeuge für die Umsetzung des OAuth 2.0 Protokolls bereit. [Background Jobs](https://doc.owncloud.org/server/9.1/developer_manual/app/backgroundjobs.html), [Hooks](https://doc.owncloud.org/server/9.1/developer_manual/app/hooks.html) und [Logging](https://doc.owncloud.org/server/9.1/developer_manual/app/logging.html) erlaubten die Implementierung weiterer Funktionen.

### Core Anpassungen

Um auf die WebDAV Schnittstelle und die OCS Share API über OAuth 2.0 zugreifen zu können, untersuchten wir die Implementierungen dieser Schnittstellen im [ownCloud Core](https://github.com/owncloud/core). Dabei haben wir Änderungen bei der ownCloud-internen `dav` App und dem Authentifizierungsmechanismus für ownCloud APIs durchgeführt. Dabei war es ein wichtiges Anliegen, keine Abhängigkeiten zwischen dem ownCloud Core und der OAuth 2.0 App einzuführen. Für die Änderungen wurde der Pull Request [owncloud/core#26742](https://github.com/owncloud/core/pull/26742) erstellt. Ein Backport für ownCloud 9.1 ist im Pull Request [owncloud/core#27370](https://github.com/owncloud/core/pull/27370) zu finden.

Folgende Tabelle fasst die Funktionen der einzelnen Bestandteile zusammen.

| Bestandteil                          | Funktion                                                                                                                             |
|:-------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------|
| [OAuth 2.0 App](oauth2-app)          | Implementierung des OAuth 2.0 Protokolls mit der notwendigen Authentifizierungslogik für die WebDAV Schnittstelle und ownCloud APIs  |
| [Core Anpassungen](core-anpassungen) | Erweiterung der Authentifizierung in der `dav` App und für ownCloud APIs um das Laden benutzerdefinierter Authentifizierungsmethoden |
