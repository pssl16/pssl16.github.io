# Core Anpassungen

## Zweck

Um das durch die `oauth2` App implementierte OAuth 2.0 Protokoll für den Zugriff auf die [WebDAV Schnittstelle](https://doc.owncloud.org/server/9.1/user_manual/files/access_webdav.html) und die [OCS Share API](https://doc.owncloud.org/server/9.1/developer_manual/core/ocs-share-api.html) nutzen zu können, mussten Anpassungen am [ownCloud Core](https://github.com/owncloud/core) durchgeführt werden. Diese Änderungen liegen im Pull Request [owncloud/core#26742](https://github.com/owncloud/core/pull/26742) vor. Ein Backport für ownCloud 9.1 ist im Pull Request [owncloud/core#27370](https://github.com/owncloud/core/pull/27370) zu finden.

## Vorgegebene Schnittstelle

Bevor wir ownCloud um OAuth 2.0 erweitern konnten, mussten wir die vorgegebene Schnittstelle untersuchen. Dazu schauten wir uns die Implementierungen von WebDAV und der OCS Share API an.

### WebDAV

Bei der Untersuchung der WebDAV Schnittstelle stellte sich heraus, dass hierfür eine ownCloud App namens `dav` verwendet wurde, die die [sabre/dav](http://sabre.io) Bibliothek zur Implementierung des WebDAV-Serves nutzt. Da die Bibliothek den Austausch der Authentifizierung über eigene Authentication Backends unterstützt, entschieden wir uns dafür, diesen Mechanismus auch für die Bereitstellung von OAuth 2.0 zu nutzen. Die Implementierung eines solchen Backends wird auf der Seite zur [OAuth 2.0 App](oauth2-app/#authentifizierungslogik) beschrieben.

### ownCloud APIs

Die Implementierung der OCS Share API wird größtenteils durch die ownCloud App `files_sharing` bereitgestellt. Daher mussten Änderungen auf Ebene von ownCloud Apps und im Speziellen bei der Authentifizierung der von ihnen definierten APIs durchgeführt werden. Wir folgten dem Vorschlag eines ownCloud Entwicklers, dafür einen Plugin Mechanismus zu implementieren, sodass Apps weitere Authentifizierungsmethoden bereitstellen können.

## Implementierung

### WebDAV

Um eigene Authentication Backends für die `dav` App bekannt zu machen und gleichzeitig keine Abhängigkeit zu schaffen, nutzten wir Event Listener. Vor dem Start des WebDAV-Servers in der Datei `appinfo/v1/webdav.php` lösten wir das `authInit`-Event aus, wie folgendes Codebeispiel zeigt.

```php
<?php
$event = new \OCP\SabrePluginEvent($server);
\OC::$server->getEventDispatcher()->dispatch('OCA\DAV\Connector\Sabre::authInit', $event);
```

Dadurch ist es ownCloud Apps möglich, das Event abzugreifen und vor dem Start des WebDAV-Servers weitere Authentication Backends hinzuzufügen. Wie dies in der OAuth 2.0 App implementiert wurde, beschreibt [dessen Seite](oauth2-app/#authentifizierungslogik).

### ownCloud APIs

Die Authentifizierung von API-Zugriffen in ownCloud geschieht in der Datei `lib/private/legacy/api.php`, welche auf `lib/private/User/Session.php` zurückgreift. Analog zu den bestehenden Funktionen `tryTokenLogin` und `tryBasicAuthLogin` fügten wir die Funktion `tryAuthModuleLogin` hinzu. Ein `AuthModule` kann dabei von ownCloud Apps bereitgestellt werden, indem eine Implementierung des Interfaces `IAuthModule` (zu finden in der Datei `lib/public/Authentication/IAuthModule.php`) registriert wird. Nachfolgendes Codebeispiel zeigt dieses Interface.

```php
<?php
namespace OCP\Authentication;

use OCP\IRequest;
use OCP\IUser;

/**
 * Interface IAuthModule
 *
 * @package OCP\Authentication
 * @since 10.0.0
 */
interface IAuthModule {

	/**
	 * Authenticates a request.
	 *
	 * @param IRequest $request The request.
	 *
	 * @return null|IUser The user if the request is authenticated, null otherwise.
	 * @since 10.0.0
	 */
	public function auth(IRequest $request);
	
	/**
	 * Returns the user's password.
	 *
	 * @param IRequest $request The request.
	 *
	 * @return String The user's password.
	 * @since 10.0.0
	 */
	public function getUserPassword(IRequest $request);
	
}
```

Die Funktion `auth` authentifiziert eine Anfrage und ermittelt (bei erfolgreicher Authentifizierung) den zugehörigen Nutzer. Zusätzlich gibt es noch die Funktion `getUserPassword`, welche zu einer Anfrage das Passwort des Nutzers bestimmt. Sie wurde hinzugefügt, da die App `encryption` in bestimmten Nutzungsszenarien auf das Passwort des Nutzers angewiesen ist.

In der Funktion `tryAuthModuleLogin` der Klasse `Session` werden nun alle von ownCloud Apps bereitgestellten Implementierungen von `IAuthModule` geladen und zur Authentifizierung genutzt, wie im folgenden Codebeispiel zu sehen.

```php
<?php
/**
 * Tries to login with an AuthModule provided by an app
 *
 * @param IRequest $request The request
 * @return bool True if request can be authenticated, false otherwise
 * @throws Exception If the auth module could not be loaded
 */
public function tryAuthModuleLogin(IRequest $request) {
	/** @var IAppManager $appManager */
	$appManager = OC::$server->query('AppManager');
	$allApps = $appManager->getInstalledApps();
	
	foreach ($allApps as $appId) {
		$info = $appManager->getAppInfo($appId);
		
		if (isset($info['auth-modules'])) {
			$authModules = $info['auth-modules'];
			
			foreach ($authModules as $class) {
				try {
					if (!OC_App::isAppLoaded($appId)) {
						OC_App::loadApp($appId);
					}
					
					/** @var IAuthModule $authModule */
					$authModule = OC::$server->query($class);
					
					if ($authModule instanceof IAuthModule) {
						return $this->loginUser(
							$authModule->auth($request),
							$authModule->getUserPassword($request)
						);
					} else {
						throw new Exception("Could not load the auth module $class");
					}
				} catch (QueryException $exc) {
					throw new Exception("Could not load the auth module $class");
				}
			}
		}
	}
	
	return false;
}
```

Hier wird in allen installierten Apps in der `info.xml` nach dem Tag `auth-modules` gesucht. Durch Nutzung von Registrierungen in der `info.xml` war es möglich, Abhängigkeiten zu anderen Apps zu vermeiden. Falls die App ein `AuthModule` implementiert und registriert hat, wird die registrierte Klasse geladen und für die Funktion `loginUser` genutzt.

Die Funktion `loginUser` ist analog zu der bestehenden Funktion `loginWithToken` implementiert worden und nutzt die Rückgabewerte der beiden Funktionen aus dem `AuthModule`, wie folgendes Codebeispiel zeigt.

```php
/**
 * Logs a user in
 *
 * @param IUser $user The user
 * @param String $password The user's password
 * @return boolean True if the user can be authenticated, false otherwise
 * @throws LoginException if an app canceled the login process or the user is not enabled
 */
private function loginUser($user, $password) {
	if (is_null($user)) {
		return false;
	}
	
	$this->manager->emit('\OC\User', 'preLogin', [$user, $password]);
	
	if (!$user->isEnabled()) {
		$message = \OC::$server->getL10N('lib')->t('User disabled');
		throw new LoginException($message);
	}
	
	$this->setUser($user);
	$this->setLoginName($user->getDisplayName());
	
	$this->manager->emit('\OC\User', 'postLogin', [$user, $password]);
	
	if ($this->isLoggedIn()) {
		$this->prepareUserLogin(false);
	} else {
		$message = \OC::$server->getL10N('lib')->t('Login canceled by app');
		throw new LoginException($message);
	}
	
	return true;
}
```

Die Implementierung und Registrierung eines `AuthModule`s in der OAuth 2.0 App wird auf [dessen Seite](oauth2-app/#authentifizierungslogik) beschrieben.

## Tests und Continuous Integration

Die Änderungen in den Pull Requests durchliefen die Tests zum ownCloud Core, die bei [Jenkins](https://jenkins.owncloud.org/job/owncloud-core/job/core/) und [Travis](https://travis-ci.org/owncloud/core/) durchgeführt werden.
