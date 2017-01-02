# Technische Umsetzung

Da man möglichst keine neue Schnittstelle implementieren wollte, war es das Ziel, die bestehende WebDAV Schnittstelle um OAuth 2.0 zu erweitern. Die WebDAV Schnittstelle ist als [ownCloud App](https://doc.owncloud.org/server/latest/developer_manual/app/) realisiert worden und nutzt die [sabre/dav](http://sabre.io/) Bibliothek. Auf der anderen Seite musste das OAuth 2.0 Protokoll mit seinen Schnittstellen bereitgestellt werden, um die Authentifizierung in der WebDAV App um OAuth 2.0 zu erweitern zu können. Dafür wurde eine weitere ownCloud App implementiert.

## Implementierung der `oauth2` App

In der App sollte der häufig für Webapplikationen eingesetzte [Authorization Code Flow](https://tools.ietf.org/html/rfc6749#section-4.1) implementiert werden. Dazu mussten folgende User Stories umgesetzt werden:

* **Clientregistrierung:** Als ownCloud-Administrator möchte ich Clients in den Administrator-Einstellungen hinzufügen und löschen können, um die Kontrolle über erlaubte Clients zu haben.
* **Authorization URL:** Als Client-Entwickler möchte ich eine Authorization URL zur Verfügung haben, um Authorization Codes anfordern zu können.
* **Access Token URL:** Als Client-Entwickler möchte ich eine Access Token URL zur Verfügung haben, um Access Tokens anfordern zu können.
* **Verwaltung autorisierter Applikationen**: Als ownCloud-Nutzer möchte ich in den persönlichen Einstellungen autorisierte Applikationen verwalten können, um einen Überblick zu haben und Autorisierungen widerrufen zu können.

### Datenmodell

Zunächst musste ein Datenmodell zur Speicherung der benötigten Daten aufgestellt werden. Gemäß dem Authorization Code Flow wurden folgende Entitäten mit Attributen definiert:

* **[`client`](https://tools.ietf.org/html/rfc6749#section-1.1):** Die Applikation, die für den Zugriff auf die WebDAV Schnittstelle autorisiert werden soll.
	* `identifier`: Zeichenkette, die einen Client eindeutig identifiziert.
	* `secret`: Zeichenkette, mit der ein Client sich beim Anfordern eines Access Tokens authentifizieren kann.
	* `redirect_uri`: URI, an die nach erfolgter Autorisierung des Nutzers weitergeleitet wird.
* **[`authorization_code`](https://tools.ietf.org/html/rfc6749#section-1.3.1):** Ein [Authorization Grant](https://tools.ietf.org/html/rfc6749#section-1.3), mit dem der Client die Autorisierung des Nutzers darlegen und somit ein Access Token anfordern kann.
	* `code`: Zeichenkette, die als Authorization Code dient.
	* `client_id`: Client Identifier des Clients, für den der Authorization Code ausgegeben wird.
	* `user_id`: User ID des ownCloud-Nutzers, der den Client autorisiert hat.
	* `expires`: Zeitpunkt, zu dem der Authorization Code ungültig wird (optional).
* **[`access_token`](https://tools.ietf.org/html/rfc6749#section-1.4):** Eine Zeichenkette, die den Zugriff auf die WebDAV Schnittstelle erlaubt.
	* `code`: Zeichenkette, die als Access Token dient.
	* `client_id`: Client Identifier des Clients, für den der Access Token ausgegeben wird.
	* `user_id`: User ID des ownCloud-Nutzers, der den Client autorisiert hat.
	* `expires`: Zeitpunkt, zu dem der Access Token ungültig wird (optional).
* **[`refresh_token`](https://tools.ietf.org/html/rfc6749#section-1.5):** Eine Zeichenkette, mit der ein abgelaufener Access Token gegen einen neuen ausgetauscht werden kann.
	* `code`: Zeichenkette, die als Refresh Token dient.
	* `client_id`: Client Identifier des Clients, für den der Access Token ausgegeben wird.
	* `user_id`: User ID des ownCloud-Nutzers, der den Client autorisiert hat.
	* `expires`: Zeitpunkt, zu dem der Refresh Token ungültig wird (optional).

Folgendes Entity-Relationship-Modell fasst das Datenmodell nochmal grafisch zusammen.

<div class="alert alert-danger">
  <strong>TODO:</strong> ERM einfügen.
</div>

### Mapper und Entities

Für den Datenbank-Zugriff im PHP-Code ist es in ownCloud möglich, [Mapper](https://doc.owncloud.org/server/latest/developer_manual/app/database.html#mappers) und [Entities](https://doc.owncloud.org/server/latest/developer_manual/app/database.html#entities) zu schreiben. Dadurch werden Tupel in einer Datenbank-Tabelle automatisch in ein Objekt umgewandelt.

Folgendes Codebeispiel zeigt am Beispiel des Entitys `Client`, wie eine PHP-Klasse dazu aussehen muss.

```php
<?php
namespace OCA\OAuth2\Db;

use OCP\AppFramework\Db\Entity;

/**
 * Class Client
 *
 * @method string getIdentifier()
 * @method void setIdentifier(string $identifier)
 * @method string getSecret()
 * @method void setSecret(string $secret)
 * @method string getRedirectUri()
 * @method void setRedirectUri(string $redirectUri)
 * @method string getName()
 * @method void setName(string $name)
 */
class Client extends Entity {

    protected $identifier;
    protected $secret;
    protected $redirectUri;
    protected $name;

    public function __construct() {
        $this->addType('id', 'int');
        $this->addType('identifier', 'string');
        $this->addType('secret', 'string');
        $this->addType('redirect_uri', 'string');
        $this->addType('name', 'string');
    }

}
```

Wichtig ist, dass die Klasse von [`Entity`](https://doc.owncloud.org/api/classes/OCP.AppFramework.Db.Entity.html) erbt und sowohl der Klassenname als auch die Attribute mit denen der Tabelle übereinstimmen. Pascal bzw. Camel case im PHP-Code wird automatisch zu Snake case für die Datenbank umgewandelt. Getter und Setter werden ebenfalls automatisch generiert. Die PHPDoc Kommentare dienen lediglich dazu, in der Entwicklungsumgebung eine automatische Vervollständigung zu haben. Die Angabe von [Typen](https://doc.owncloud.org/server/latest/developer_manual/app/database.html#types) im Konstruktor dienen dazu, beim Lesen aus der Datenbank die richtige Umwandlung zu erhalten.

Das folgende Codebeispiel zeigt einen Ausschnitt aus dem zur `Client` Entity gehörenden Mapper.

```php
<?php
namespace OCA\OAuth2\Db;

use InvalidArgumentException;
use OCP\AppFramework\Db\Entity;
use OCP\IDb;
use OCP\AppFramework\Db\Mapper;

class ClientMapper extends Mapper {

	/**
	 * ClientMapper constructor.
	 *
	 * @param IDb $db Database Connection.
	 */
	public function __construct(IDb $db) {
		parent::__construct($db, 'oauth2_clients');
	}

	/**
	 * Selects a client by its ID.
	 *
	 * @param int $id The client's ID.
	 *
	 * @return Entity The client entity.
	 *
	 * @throws \OCP\AppFramework\Db\DoesNotExistException if not found.
	 * @throws \OCP\AppFramework\Db\MultipleObjectsReturnedException if more than one result.
	 */
	public function find($id) {
		if (!is_int($id)) {
			throw new InvalidArgumentException('id must not be null');
		}

		$sql = 'SELECT * FROM `' . $this->tableName . '` WHERE `id` = ?';
		return $this->findEntity($sql, array($id), null, null);
	}

	/**
	 * Selects clients by the given user ID.
	 *
	 * @param string $userId The user ID.
	 *
	 * @return array The client entities.
	 */
	public function findByUser($userId) {
		if (!is_string($userId)) {
			throw new InvalidArgumentException('userId must not be null');
		}

		$sql = 'SELECT * FROM `' . $this->tableName . '` '
			. 'WHERE `id` IN ( '
				. 'SELECT `client_id` FROM `oc_oauth2_authorization_codes` WHERE `user_id` = ? '
				. 'UNION '
				. 'SELECT `client_id` FROM `oc_oauth2_access_tokens` WHERE `user_id` = ? '
			.')';
		return $this->findEntities($sql, array($userId, $userId), null, null);
	}

}
```

Beim Mapper ist es wichtig, dass die Klasse von [`Mapper`](https://doc.owncloud.org/api/classes/OCP.AppFramework.Db.Mapper.html) erbt und eine Entity-Klasse zu ihm existiert. Dazu wird das Wort vor „Mapper“ als Entityname verwendet. Im Konstruktur wird der Tabellenname angegeben. Die beiden Funktionen `find` und `findByUser` demonstrieren `SELECT`-Anweisungen. Dazu wird die SQL-Anweisungen zusammen mit benötigten Parametern an `findEntity` bzw. `findEntities` übergeben, abhängig davon, ob mehrere Entities im Ergebnis enthalten sein sollten. Funktionen zum löschen, einfügen und updaten werden von der Oberklasse bereits implementiert und mussten nicht angepasst werden.

### Schnittstellen und Routes

Um in einer ownCloud App Schnittstellen anzubieten, müssen [Routes](https://doc.owncloud.org/server/latest/developer_manual/app/routes.html) registriert werden. Zur Umsetzung der erwähnten User Stories waren folgende Routes notwendig:

| Methode | Endpunkt              | Beschreibung                                                                                              |
|---------|-----------------------|-----------------------------------------------------------------------------------------------------------|
| `GET`   | `authorize`           | Endpunkt, zu dem der Client den Nutzer weiterleitet, um die Autorisierung anzufragen (Authorization URL). |
| `POST`  | `authorize`           | Endpunkt, der aufgerufen wird, sobald der Nutzer den Client autorisiert hat.                              |
| `POST`  | `api/v1/token`        | Endpunkt, an dem ein Access Token angefordert wird (Access Token URL).                                    |
| `POST`  | `clients`             | Endpunkt, durch den der Administrator einen Client hinzufügen kann.                                       |
| `POST`  | `clients/{id}/delete` | Endpunkt, durch den der Administrator den Client mit der ID `id` löschen kann.                            |
| `POST`  | `clients/{id}/revoke` | Endpunkt, durch den der Nutzer die Autorisierung des Clients mit der ID `id` widerrufen kann.             |

Registriert werden die Routes in der Datei `routes.php`, indem ein Array mit den Routes zurückgegeben wird. Nachfolgendes Codebeispiel zeigt einige der obigen Routes:

```php
<?php
return [
	'routes' => [
		['name' => 'page#authorize', 'url' => '/authorize', 'verb' => 'GET'],
		['name' => 'o_auth_api#generate_token', 'url' => '/api/v1/token', 'verb' => 'POST'],
		['name' => 'settings#deleteClient', 'url' => '/clients/{id}/delete', 'verb' => 'POST']
    ]
];

```

Durch `name` wird für jede Route der Name des dazugehörigen [Controllers](#controller) sowie die aufzurufende Funktion angegeben. Vor dem `#`-Zeichen steht der Controllername in Snake case und hinter dem `#`-Zeichen steht der Funktionsname (ebenfalls in Snake case). Mithilfe von `url` wird der Endpunkt festgelegt und `verb` definiert die HTTP-Methode.

### Controller

Wenn an einem Endpunkt eine HTTP-Anfrage ankommt, so wird der in den Routes definierte [Controller](https://doc.owncloud.org/server/latest/developer_manual/app/controllers.html) aufgerufen. Wichtig ist hierbei, dass von der Klasse [`Controller`](https://doc.owncloud.org/api/classes/OCP.AppFramework.Controller.html) oder einer Unterklasse wie [`ApiController`](https://doc.owncloud.org/api/classes/OCP.AppFramework.ApiController.html) geerbt wird.

Für den Controller notwendige Parameter wie [Mapper](#mapper-und-entities) können im Konstruktor als Parameter angegeben und so durch [Dependency Injection](https://doc.owncloud.org/server/latest/developer_manual/app/container.html) erhalten werden. Nachfolgendes Codebeispiel zeigt den Konstruktor vom `PageController`.

```php
/**
 * PageController constructor.
 * 
 * @param string $AppName The name of the app.
 * @param IRequest $request The request.
 * @param ClientMapper $clientMapper The client mapper.
 * @param AuthorizationCodeMapper $authorizationCodeMapper The authorization code mapper.
 * @param string $UserId The user ID.
 */
public function __construct($AppName, IRequest $request, ClientMapper $clientMapper,
	AuthorizationCodeMapper $authorizationCodeMapper, $UserId) {
	parent::__construct($AppName, $request);

	$this->clientMapper = $clientMapper;
	$this->authorizationCodeMapper = $authorizationCodeMapper;
	$this->userId = $UserId;
}
```

Die hier notwendigen Parameter sind der Name der App, eine `ClientMapper` Instanz, eine `AuthorizationCodeMapper` Instanz und die ID des Nutzers, um bei der Autorisierung des Clients speichern zu können, welcher Nutzers dies veranlasst hat.

Die mit den Routes verknüpften Funktionen können zur Zugriffskontrolle mit [PHPDoc Annotationen](https://doc.owncloud.org/server/latest/developer_manual/app/controllers.html#authentication) versehen werden. Folgendes Codebeispiel zeigt die Annotationen für die Funktion `generateToken` im `OAuthApiController`.

```php
/**
 * Implements the OAuth 2.0 Access Token Response.
 *
 * @param string $code The authorization code.
 * @return JSONResponse The Access Token or an empty JSON Object.
 *
 * @NoAdminRequired
 * @NoCSRFRequired
 * @PublicPage
 * @CORS
 */
public function generateToken($code) { }
```

Die Annotationen haben dabei folgende Bedeutungen.

| Annotation         | Bedeutung                                                         |
|--------------------|-------------------------------------------------------------------|
| `@NoAdminRequired` | Aufruf auch von normalen Nutzern möglich.                         |
| `@NoCSRFRequired`  | Zeigt an, dass die Überprüfung des CSRF Tokens nicht gewollt ist. |
| `@PublicPage`      | Zugriff auch ohne Login möglich.                                  |
| `@CORS`            | Aufruf der API durch andere Web Applikationen von außen möglich.  |


In den Controller-Funktionen können verschiedene Inhalte zurückgegeben werden. Hier genutzte Rückgabetypen sind in der folgenden Tabelle zusammengefasst.

| Typ                                                                                                          | Beschreibung                                                        |
|--------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------|
| [`TemplateResponse`](https://doc.owncloud.org/server/latest/developer_manual/app/controllers.html#templates) | Zur Rückgabe eines Templates, das dem Nutzer angezeigt werden soll. |
| [`RedirectResponse`](https://doc.owncloud.org/server/latest/developer_manual/app/controllers.html#redirects) | Zur Weiterleitung des Nutzers an eine andere URL.                   |
| [`JSONResponse`](https://doc.owncloud.org/server/latest/developer_manual/app/controllers.html#json)          | Zur Rückgabe eines JSON Strings.                                    |

Ein Beispiel für die Rückgabetypen `TemplateResponse` und `RedirectResponse` gibt die Funktion `authorize` im `PageController`, die im folgenden Codebeispiel zu sehen ist.

```php
/**
 * Shows a view for the user to authorize a client.
 *
 * @param string $response_type The expected response type.
 * @param string $client_id The client identifier.
 * @param string $redirect_uri The redirect URI.
 * @param string $state The state.
 * @param string $scope The scope.
 *
 * @return TemplateResponse|RedirectResponse The authorize view or a
 * redirection to the ownCloud main page.
 *
 * @NoAdminRequired
 * @NoCSRFRequired
 */
public function authorize($response_type, $client_id, $redirect_uri, $state = null, $scope = null) {
	if (!is_string($response_type) || !is_string($client_id)
		|| !is_string($redirect_uri) || (isset($state) && !is_string($state))
		|| (isset($scope) && !is_string($scope))) {
		return new RedirectResponse(OC_Util::getDefaultPageUrl());
	}

	try {
		/** @var Client $client */
		$client = $this->clientMapper->findByIdentifier($client_id);
	} catch (DoesNotExistException $exception) {
		return new RedirectResponse(OC_Util::getDefaultPageUrl());
	}

	if (strcmp($client->getRedirectUri(), urldecode($redirect_uri)) !== 0) {
		return new RedirectResponse(OC_Util::getDefaultPageUrl());
	}
	if (strcmp($response_type, 'code') !== 0) {
		return new RedirectResponse(OC_Util::getDefaultPageUrl());
	}

	return new TemplateResponse('oauth2', 'authorize', ['client_name' => $client->getName()]);
}
```

Hier werden zunächst die Parameter auf Gültigkeit überprüft. Sollten die Parameter nicht gültig sein (beispielsweise deshalb, weil der angegebene Client nicht existiert oder dessen Redirect URI falsch angegeben wurde) wird mit einem `RedirectResponse` auf die ownCloud Startseite umgeleitet. Andernfalls wird ein `TemplateResponse` für das Template `authorize` zurückgegeben. Für das Rendern des Templates können Parameter wie hier `client_name` für den Namen des Clients übergeben werden.

Der Rückgabetyp `JSONResponse` wird für die Rückgabe des Access Tokens in der Funktion `generateToken` im `OAuthApiController` genutzt, wie nachfolgendes Codebeispiel zeigt. Zudem ist das Zusammenspiel mit Entities und Mappern zu sehen.

```php
/**
 * Implements the OAuth 2.0 Access Token Response.
 *
 * @param string $code The authorization code.
 * @return JSONResponse The Access Token or an empty JSON Object.
 *
 * @NoAdminRequired
 * @NoCSRFRequired
 * @PublicPage
 * @CORS
 */
public function generateToken($code) {
	if (is_null($code) || is_null($_SERVER['PHP_AUTH_USER'])
		|| is_null($_SERVER['PHP_AUTH_PW'])) {
		return new JSONResponse(['message' => 'Missing credentials.'], Http::STATUS_BAD_REQUEST);
	}

	try {
		/** @var Client $client */
		$client = $this->clientMapper->findByIdentifier($_SERVER['PHP_AUTH_USER']);
	} catch (DoesNotExistException $exception) {
		return new JSONResponse(['message' => 'Unknown credentials.'], Http::STATUS_BAD_REQUEST);
	}

    if (strcmp($client->getSecret(), $_SERVER['PHP_AUTH_PW']) !== 0) {
        return new JSONResponse(['message' => 'Unknown credentials.'], Http::STATUS_BAD_REQUEST);
    }

	try {
		/** @var AuthorizationCode $authorizationCode */
		$authorizationCode = $this->authorizationCodeMapper->findByCode($code);
	} catch (DoesNotExistException $exception) {
		return new JSONResponse(['message' => 'Unknown credentials.'], Http::STATUS_BAD_REQUEST);
	}

    if (strcmp($authorizationCode->getClientId(), $client->getId()) !== 0) {
        return new JSONResponse(['message' => 'Unknown credentials.'], Http::STATUS_BAD_REQUEST);
    }

	$token = Utilities::generateRandom();
	$userId = $authorizationCode->getUserId();
	$accessToken = new AccessToken();
	$accessToken->setToken($token);
	$accessToken->setClientId($authorizationCode->getClientId());
	$accessToken->setUserId($userId);
	$this->accessTokenMapper->insert($accessToken);

    $this->authorizationCodeMapper->delete($authorizationCode);

    return new JSONResponse(
        [
            'access_token' => $token,
            'token_type' => 'Bearer',
			'user_id' => $userId
        ]
    );
}
```

Nach erfolgreicher Überprüfung des Authorization Codes und der Angaben zur Client Authentication im Authorization Header wird eine neuer Access Token erstellt und in der Datenbank gespeichert. Der verwendete Authorization Code wird zudem gelöscht. Im JSON Response wird dann der Access Token, der Token Typ und die ID des Nutzers zurückgegeben. Nachfolgend ist ein Beispiel dazu angegeben.

```json
{
	"access_token" : "1vtnuo1NkIsbndAjVnhl7y0wJha59JyaAiFIVQDvcBY2uvKmj5EPBEhss0pauzdQ",
	"token_type" : "Bearer",
	"user_id" : "admin"
}
```

Für die Token-Generierung wurde die Hilfsklasse `Utilities` mit der statischen Funktion `generateRandom` geschrieben, die mithilfe einer ownCloud-internen Funktion 64-stellige Zeichenketten erzeugt. Folgendes Codebeispiel zeigt diese Klasse.

```php
<?php
namespace OCA\OAuth2;

class Utilities {

    /**
     * Generates a random string with 64 characters.
     *
     * @return string The random string.
     */
    public static function generateRandom() {
        return \OC::$server->getSecureRandom()->generate(64,
            'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789');
    }

}
```

Zusammenfassend werden im folgenden UML-Klassendiagramm die Controller mit ihren Beziehungen zu den Entities und Mappern dargestellt.

<div class="alert alert-danger">
  <strong>TODO:</strong> Klassendiagramm einfügen.
</div>

### Templates

Für die zweckmäßige Nutzung der OAuth2 App sind Templates von Nöten. Relevant sind diese als Darstellung des Authorisierungsfenster, sprich der Hauptfunktionalität der App, für die Einstellungen des Admins und in den persönlichen Einstellungen. 
    `authorize` stellt ein umrahmtes Fenster mit entsprechendem Text zur Authorisierung und der Möglichkeit die Authorisierung zu akzeptieren oder abzulehnen dar.
    `settings-admin` stellt eine tabellarische Auflistung der registrierten Clients und ihrer jeweiligen redirectURIs, Identifiers und Secrets mit Möglichkeit die entsprechende Registrierung zu löschen dar, oder eine Meldung, dass kein Client registriert sind. Zusätzlich gibt es eine Eingabemaske zur Registrierung von neuen Clients, die einen Namen und eine redirectURI fordert.
    `settings-personal` stellt eine tabellarische Auflistung der autorisierten Applikationen mit Möglichkeit die entsprechende Authorisierung zu löschen dar, oder eine Meldung, dass keine Applikationen authorisiert sind.

<div class="alert alert-danger">
<strong>TODO:</strong> Screenshots einfügen.
</div>

### Protokollablauf

### Tests

Zum Testen der PHP-Klassen wurde das Framework [PHPUnit](https://phpunit.de/) verwendet. Die aktuelle Testabdeckung ist bei Codecov einsehbar: [![codecov](https://codecov.io/gh/pssl16/owncloud-app_oauth2/branch/master/graph/badge.svg)](https://codecov.io/gh/pssl16/owncloud-app_oauth2).

### Continuous Integration

Als Continuous Integration Integration Tool wurde Travis CI verwendet. Bei jeder Änderung im [GitHub Repository](https://github.com/pssl16/owncloud-app_oauth2) wird ein Build angestoßen, in dem die App mithilfe eines Makefiles für den App Store gebaut wird und anschließend in verschiedenen Umgebungen installiert und getestet wird. Folgende Parameter werden variiert:

* **PHP Versionen**: 5.6, 7.0, 7.1, nightly
* **Datenbanken**: PostgreSQL, MySQL, SQLite
* **Branches des ownCloud Core**: `stable9.1`, `master`

Der aktuelle Build-Status ist bei Travis einsehbar: [![Build Status](https://travis-ci.org/pssl16/owncloud-app_oauth2.svg?branch=master)](https://travis-ci.org/pssl16/owncloud-app_oauth2).

## Anpassung der `dav` App

### Authentication Backend

### Tests
