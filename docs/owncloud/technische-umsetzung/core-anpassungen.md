# Core Anpassungen

## Zweck

Um das durch die `oauth2` App implementierte OAuth 2.0 Protokoll für den Zugriff auf die [WebDAV Schnittstelle](https://doc.owncloud.org/server/9.1/user_manual/files/access_webdav.html) und die [OCS Share API](https://doc.owncloud.org/server/9.1/developer_manual/core/ocs-share-api.html) nutzen zu können, mussten Anpassungen am [ownCloud Core](https://github.com/owncloud/core) durchgeführt werden. Diese Änderungen liegen im Pull Request [owncloud/core#26742](https://github.com/owncloud/core/pull/26742) vor. Ein Backport für ownCloud 9.1 ist im Pull Request [owncloud/core#27370](https://github.com/owncloud/core/pull/27370) zu finden.

## Vorgegebene Schnittstelle

Bevor wir um OAuth 2.0 erweitern konnten, mussten wir die vorgegebene Schnittstelle untersuchen. Dazu schauten wir uns die Implementierungen von WebDAV und der OCS Share API an.

### WebDAV

### ownCloud APIs

## Implementierung

### WebDAV

### ownCloud APIs

## Tests und Continuous Integration
