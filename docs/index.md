# sciebo@Learnweb
Diese Website dokumentiert die Integration von [sciebo](https://www.sciebo.de/) in das [Learnweb](https://www.uni-muenster.de/LearnWeb/learnweb2/)
von dem Projektseminar [sciebo@Learnweb](https://www.wi.uni-muenster.de/de/studierende/lehrangebot/227197), das im Wintersemester 2016/17 an der [Westfälischen Wilhelms-Universität Münster](http://www.uni-muenster.de/) stattgefunden hat.

Sciebo ist ein Cloud-Dienst, der gemeinsam von 26 Hochschulen und Forschungseinrichtungen in NRW angeboten wird. Als Grundlage dient die Enterprise-Edition von [ownCloud](https://owncloud.org/).
Das Learnweb ist ein auf der Open-Source-Lösung [Moodle](https://moodle.org/) basierendes E-Learning System der Westfälischen Wilhelms-Universität Münster.

Eine mögliche Lösung sollte dabei möglichst allgemein einsetzbar sein, da die Kombination von Moodle und ownCloud an zahlreichen Universitäten genutzt wird.

## Allgemeine Motivation für die Arbeit
Die grundsätzliche Motivation für das Projektseminar war es, die beiden Systeme [sciebo](https://www.sciebo.de/) und [Learnweb](https://www.uni-muenster.de/LearnWeb/learnweb2/) passwortlos miteinander kommunizieren zu lassen.
Bedeutet, dass einmalig ein Passwort eingegeben werden muss, welches aber nicht im Klartext auf dem jeweils anderen System gespeichert wird.
Der Zugriff sollte also über ein tokenbasiertes Authentifizierungsverfahren wie zum Beispiel OAuth 2.0 ablaufen.
Dadurch sollte es unter Anderem ermöglicht werden, dass Dateien aus Sciebo im Learnweb abgerufen werden können und kollaborative Ordner in sciebo vom Learnweb aus erstellt werden können.

## Struktur der Dokumentation
Die hier vorliegende Dokumentation ist in mehrere Teile strukturiert. Zuerst die Aufteilung in **Home**, **ownCloud** und **Moodle**. Sie befinden sich gerade im Abschnitt **Home**,
welcher grundlegende Informationen zum Projektseminar enthält und einen groben Überblick verschafft, was in der Dokumentation enthalten ist. Die Abschnitte **ownCloud** und **Moodle** dokumentieren
jeweils die spezifische **Benutzung** (Teil der Dokumention, welcher eher an die Nutzer gerichtet ist), und die jeweilige **Technische Umsetzung**, welche die Dokumentation für Entwickler bereitstellt. Der Punkt **Technische Umsetzung** im Abschnitt
**ownCloud** enthält die Themen [OAuth 2.0](/owncloud/technische-umsetzung/oauth2), [Softwarearchitektur](/owncloud/technische-umsetzung/softwarearchitektur), [OAuth 2.0 App](/owncloud/technische-umsetzung/oauth2-app)
und [Core Anpassungen](/owncloud/technische-umsetzung/core-anpassungen). Der gleiche Unterpunkt im Abschnitt **Moodle** behandelt die Themen [Softwarearchitektur](/moodle/technische-umsetzung/softwarearchitektur),
[Admin Tool](/moodle/technische-umsetzung/admin-tool), [Repository](/moodle/technische-umsetzung/repository) und [Collaborative Folders](/moodle/technische-umsetzung/acitivity).

## Integrationsszenarien
Als Integrationsszenarien der Systeme wurden verschiedene User Stories entwickelt.
Diese haben wir nach Schwierigkeit, Interesse, Benutzbarkeit und Implementationsaufwand priorisiert.
Als Integrationsrichtung konzentrierte wir uns auf die Richtung Learnweb <i class="fa fa-long-arrow-right" aria-hidden="true"></i> sciebo.
### Realisierte Szenarien

#### 1. Als **Nutzer** möchte ich OAuth2 benutzen können, um mich im Learnweb als ownCloud Nutzer anzumelden.

Dieser Use Case implementiert die grundlegende Authentifizierung mit dem [OAuth 2.0](https://oauth.net/2/) Verfahren.
Im Rahmen unserer Vorbereiungsphase auf das Projektseminar haben wir verschiedene Authentifizierungsmethoden evaluiert. Im Abschnitt
 [Authentifizierung und Autorisierung](#authentifizierung-und-autorisierung) finden sie hierfür genauere Informationen. Hierfür ist in Moodle das
 [Admin Tool](/moodle/technische-umsetzung/admin-tool.md) implementiert worden. Es übernimmt sämtliche Aufgaben der Authentifizierung. Alle anderen Plugins nutzen dieses Tool.
 In OwnCloud wurde hierfür eine [App](/owncloud/technische-umsetzung/oauth2-app.md) entwickelt.

#### 2. Als **Nutzer** möchte ich in der Dateiauswahl im Learnweb eine Datei aus meiner ownCloud Instanz hochladen.

Der vorherige Use Case war für den Benutzer der Moodle Instanz noch nicht sichtbar. Das wichtigste Anwendungsszenario ist das Nutzer im Learnweb Dateien aus ihrer
ownCloud Account hochladen können. Hierfür haben wir in Moodle ein [Repository-Plugins](/moodle/technische-umsetzung/repository.md) entwickelt. Sobald das Reposiotory einer Moodle
Instanz hinzugefügt wurde kann der Nutzer über einen anmelde Button Moodle autorisieren Dateien aus dem privaten ownCloud account anzuzeigen
und mit Hilfe des File Pickers können nun Dateien ausgewählt werden.
<div class="alert alert-danger">
  <strong>TODO:</strong> BILDER! (Wenn get_listing wieder funktioniert)
</div>

#### 3. Als **Nutzer** möchte ich in der Dateiauswahl im Learnweb eine Datei aus meiner ownCloud Instanz verlinken.

Eine Erweiterung des vorherigen Szenarios ist die Verlinkung von Dateien. Hier wird zu der bestehenden Datei ein public-link erstellt.
Dieser kann nun von Nutzern angeklickt werden. Diese Funktionalität ist auch Teil des [Repository-Plugins](/moodle/technische-umsetzung/repository.md). Das Verlinken ist
jedoch nur in der Aktivität URL verfügbar.

#### 4. Als **Lehrender** möchte ich Studierenden oder Gruppen von Studierenden Ordner für kollaboratives Arbeiten bereitstellen.

Die letzte User Story die wir implementiert haben, wurde vermehrt von Lehrenden der Universität Münster nachgefragt. Ein Lehrender kann mir Hilfe der
Aktivität [collaborativefolders](/moodle/technische-umsetzung/acitivity.md) in einem Kurs Ordner erstellen auf den Studierende Zugriff haben und den sie ihrem sciebo account hinzufügen können.

Weitere User Storys die jedoch nicht im Rahmen diese Projektseminars implementiert werden konnten finden sie unter dem Abschnitt
[Weitere Anwendungsszenarien](#weitere-anwendungsszenarien).

## Authentifizierung und Autorisierung
Grundlegend für die Integration beider Systeme ist die Authentifizierung und Autorisierung. Unter den Verfahren, die untersucht wurden, befinden sich:

* [OAuth 2.0](https://oauth.net/2/)
* [JSON Web Tokens](https://jwt.io/)
* [Macaroons](https://research.google.com/pubs/pub41892.html)
* Federated Single Sign-on mit beispielsweise [Shibboleth](https://shibboleth.net/)

Für das Projekt wurde das OAuth 2.0 Verfahren ausgewählt, da mit ihm ein standardisiertes Verfahren zur tokenbasierten Authentifizierung und
Autorisierung vorliegt, das sich gut in bestehende Applikationen auf Basis vieler verschiedener Programmiersprachen einfügen lässt und damit keine weiteren Anforderungen an die Infrastruktur stellt.

## Komponenten
<div class="alert alert-danger">
  <strong>TODO:</strong> Würde ich nicht in den Index packen, sondern einzelne Komponenten nur in dem Moodle und ownCloud Unterpunkt beschreiben.
</div>

### ownCloud

Die Komponenten, die wir benötigen um eine Lösung anzubieten, sind die von uns implementierte oauth2 App und der ownCloud Core mit Änderungen aus unserem Pull request,
damit die oauth2 App die volle Funktionalität bereitstellen kann. Die oauth2 App implementiert den OAuth 2.0 Prozessfluss.
Sie agiert als Endpunkt für die Authorisierung und die Bereitstellung von Access Tokens und Refresh Tokens.
Die App ist also das Mittel zur Verbindung von Resource Owner, Authorization Server und auch Resource Server mit dem Client.



### Moodle

<div class="alert alert-danger">
  <strong>TODO:</strong> Bestandteile der Lösung in moodle auflisten und kurz die Funktion anschneiden.
</div>

## Zusammenspiel

<div class="alert alert-danger">
  <strong>TODO:</strong> Wie arbeiten die einzelnen Komponenten zusammen, um das Problem zu lösen? (WebDAV: Wie wird der
  komplette OAuth Protokollablauf erfüllt?).
</div>

## Weitere Integrationsszenarien

Im Rahmen unseres Projektseminars haben wir uns auf die für uns wichtigsten Integrationsszenarien konzentriert. Im Folgenden werden weitere Szenarien erläutert, über deren Implementierung wir nachgedacht haben und die eine Erweiterung
zu den bestehenden Szenarien bilden könnten.

1. Als **Lehrender** möchte ich auf einen Button klicken, um hochgeladene Dateien zu aktualisieren.

2. Als **Studierender** möchte ich Dateien aus dem Learnweb direkt in meiner ownCloud Instanz speichern können.

3. Als **Studierender** möchte ich anderen Studierenden Schreib- und Lese-Rechte geben um kollaborativ zu arbeiten.

4. Als **Lehrender** möchte ich Nutzern auf Modul-Basis das Recht entziehen eine Datei zu verlinken.

5. Als **Nutzer** möchte ich in der Dateiauswahl im Learnweb einen Ordner aus meiner ownCloud Instanz hochladen.

6. Als **Lehrender** möchte ich in der Dateiauswahl im Learnweb einen Ordner aus meiner ownCloud Instanz verlinken.

7. Als **Studierender** möchte ich Lehrenden Schreib- oder Lese-Rechte auf mein Dokument geben können um Feedback zu erhalten.

8. Als **Lehrender** möchte ich die Möglichkeit haben, einen Zielordner in ownCloud zur Speicherung auszuwählen, um Abgaben herunterzuladen.

9. Als **Lehrender** möchte ich ownCloud als primären Speicher für alle Dateien im Kurs verwenden können.

10. Als **Lehrender** möchte ich in ownCloud die Teilen-Funktion nutzen, um Dateien oder Dokumente für Kursteilnehmer freigeben zu können.

## Zusammenfassung
<div class="alert alert-danger">
  <strong>TODO:</strong> Ausführung.
</div>
Erfüllungsgrad der User Stories, Geschaffene Funktionalitäten, (bekannte, ggf. bewusst gewählte) Einschränkungen, Ausblick auf weitere Entwicklungsmöglichkeiten.
Einschränkungen festhalten; den Rest ganz zum Schluss des PS beschreiben
