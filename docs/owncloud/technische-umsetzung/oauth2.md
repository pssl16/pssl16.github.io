# OAuth 2.0

Wie [eingangs begründet](../../#authentifizierung-und-autorisierung), wurde OAuth 2.0 als Verfahren für die Authentifizierung und Autorisierung zwischen den beiden Systemen Moodle und ownCloud gewählt. Diese Seite erläutert den allgemeinen Protokollablauf von OAuth 2.0.

## Allgemeiner Protokollablauf

Der allgemeine [OAuth 2.0 Protokollablauf](https://tools.ietf.org/html/rfc6749#section-1.2) ist in der nachfolgenden Abbildung dargestellt.

![Authorization Code Flow](images/oauth-allgemein.svg)

<div align="right">
	<small>[vgl. <a href="https://tools.ietf.org/html/rfc6749#page-7">RFC 6749, S. 7f</a>]</small>
</div>

Zunächst muss sich der Client (Moodle), der im Namen des Resource Owners (ownCloud Nutzer) auf eine geschützte Ressource auf dem Resource Server (ownCloud) zugreifen möchte, bei dem Authorization Server (ownCloud) [registrieren](https://tools.ietf.org/html/rfc6749#section-2). Danach werden nach dem Protokoll folgende Schritte durchlaufen:

1. **Authorization Request**: Der Client fordert eine Autorisierung vom Resource Owner an.
2. **Authorization Response**: Der Client erhält einen [Authorization Grant](https://tools.ietf.org/html/rfc6749#section-1.3) vom Resource Owner, mit dem er ein Access Token anfordern kann.
3. **Access Token Request**: Der Client fordert ein Access Token vom Authorization Server an. Hierfür nutzt er den Authorization Grant.
4. **Access Token Response**: Der Authorization Server authentifiziert den Client und prüft den Authorization Grant. Ist die Prüfung erfolgreich, wird ein [Access Token](https://tools.ietf.org/html/rfc6749#section-1.4) ausgestellt.
5. **Anfrage mit Access Token**: Der Client fragt die geschützten Daten beim Resource Server an. Zur Authentifizierung benutzt er den Access Token.
6. **Geschützte Ressource**: Der Resource Server prüft den Access Token und stellt, wenn gültig, die geschützte Ressource zur Verfügung.
