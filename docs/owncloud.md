# ownCloud

Um auf Dateien in ownCloud zuzugreifen, wird standardmäßig [eine WebDAV Schnittstelle angeboten](https://doc.owncloud.org/server/latest/user_manual/files/access_webdav.html). Da diese jedoch nur über [Basic Authentication](https://tools.ietf.org/html/rfc7617) ansprechbar ist, musste zusätzlich das [OAuth 2.0 Protokoll](https://oauth.net/2/) implementiert werden.

## Technische Umsetzung

### Implementierung der `oauth2` App

### Anpassung der `dav` App

## Benutzung

### Installation

Da die Änderungen zum aktuellen Zeitpunkt noch nicht in den Core aufgenommen wurden, muss der `dav-oauth`-Branch des [geforkten Repositorys](https://github.com/pssl16/core) geklont werden:

<pre><code class="nohighlight">$ git clone -b dav-oauth https://github.com/pssl16/core</code></pre>

Danach müssen die Dependencies installiert werden. Dazu genügt es, im Verzeichnis des Repositorys folgenden Befehl auszuführen:

<pre><code class="nohighlight">$ make</code></pre>

Die restlichen Installationsschritte unterscheiden sich nicht von denen im [ownCloud Handbuch](https://doc.owncloud.org/server/latest/admin_manual/installation/index.html).

### Clientregistrierung

### Authorization Code Flow
Die nachfolgende Abbildung stellt den durch die `oauth2` App implementierten [OAuth 2.0 Authorization Code Flow](https://tools.ietf.org/html/rfc6749#section-4.1) dar.
 
![Authorization Code Flow](images/authorization-code-flow.svg)

<div class="alert alert-danger">
  <strong>TODO:</strong> Beschreibung der Schritte einfügen.
</div>

### Angepasste WebDAV Schnittstelle