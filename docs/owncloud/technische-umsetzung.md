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

Beim Mapper ist es wichtig, dass die Klasse von [`Mapper`](https://doc.owncloud.org/api/classes/OCP.AppFramework.Db.Mapper.html) erbt und eine Entity-Klasse zu ihm existiert. Dazu wird das Wort vor „Mapper“ als Entityname verwendet. Im Konstruktur wird der Tabellenname angegeben. Die beiden Funktionen `find($id)` und `findByUser($userId)` demonstrieren `SELECT`-Anweisungen. Dazu wird die SQL-Anweisungen zusammen mit benötigten Parametern an `findEntity` bzw. `findEntities` übergeben, abhängig davon, ob mehrere Entities im Ergebnis enthalten sein sollten. Funktionen zum löschen, einfügen und updaten werden von der Oberklasse bereits implementiert und mussten nicht angepasst werden.

### Schnittstellen und Routes

### Controller

### Templates

### Tests

### Continuous Integration

## Anpassung der `dav` App

### Authentication Backend

### Tests

## Protokollablauf im Detail
