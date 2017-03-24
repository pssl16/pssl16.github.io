# sciebo@Learnweb
Diese Website dokumentiert die Integration von [sciebo](https://www.sciebo.de/) in das [Learnweb](https://www.uni-muenster.de/LearnWeb/learnweb2/), die durch das Projektseminar [sciebo@Learnweb](https://www.wi.uni-muenster.de/de/studierende/lehrangebot/227197) im Wintersemester 2016/17 an der [Westfälischen Wilhelms-Universität Münster](http://www.uni-muenster.de/) realisiert wurde.

Sciebo ist ein Cloud-Dienst, der gemeinsam von 26 Hochschulen und Forschungseinrichtungen in NRW angeboten wird. Als Grundlage dient die Enterprise-Edition von [ownCloud](https://owncloud.org/). Das Learnweb ist ein auf der Open-Source-Lösung [Moodle](https://moodle.org/) basierendes E-Learning System der Westfälischen Wilhelms-Universität Münster.

## Struktur der Dokumentation
Die vorliegende Dokumentation ist in mehrere Teile gegliedert. Neben der Startseite (auf der Sie sich momentan befinden), gibt es die Abschnitte **ownCloud** und **Moodle**.

Die Startseite enthält grundlegende Informationen zum Projektseminar und verschafft einen Überblick über die Dokumentation. Die Abschnitte **ownCloud** und **Moodle** dokumentieren jeweils die **technische Umsetzung** und **Benutzung** der beiden Systeme. Während die technische Umsetzung für Entwickler interessant ist, ist der Teil Benutzung als Bedienungsanleitung an die Nutzer gerichtet.

Der Unterabschnitt **technische Umsetzung** im Abschnitt **ownCloud** enthält die Themen [OAuth 2.0](/owncloud/technische-umsetzung/oauth2), [Softwarearchitektur](/owncloud/technische-umsetzung/softwarearchitektur), [OAuth 2.0 App](/owncloud/technische-umsetzung/oauth2-app) und [Core Anpassungen](/owncloud/technische-umsetzung/core-anpassungen). Auf Seiten von **Moodle** werden die Themen [Softwarearchitektur](/moodle/technische-umsetzung/softwarearchitektur), [Admin Tool](/moodle/technische-umsetzung/admin-tool), [Repository](/moodle/technische-umsetzung/repository) und [Collaborative Folders](/moodle/technische-umsetzung/activity) behandelt.

## Motivation
Da im Learnweb sehr häufig mit Dateien umgegangen wird, lag es nahe, dem Nutzer nicht nur den Upload seiner lokalen Dateien anzubieten, sondern zusätzlich seinen Cloudspeicher zu integrieren (ganz abgesehen von den anderen Möglichkeiten, die die Integration eines Cloudspeichers bietet). Zur Kommunikation zwischen den beiden Systemen musste bisher jedoch das Passwort des Nutzers in beiden Systemen gespeichert werden, was ein Sicherheitsrisiko darstellt. Daher mussten andere Lösungen gefunden werden.

Das grundlegende Ziel bei allen umzusetzenden [Integrationsszenarien](#integrationsszenarien) war daher, die beiden Systeme sciebo und Learnweb passwortlos miteinander kommunizieren zu lassen. Dies bedeutet, dass beispielsweise für einen Zugriff vom Learnweb auf sciebo das Passwort des Nutzers nicht im Learnweb vorliegt, sondern die Authentifizierung und Autorisierung auf anderem Wege stattfindet.

Des Weiteren sollte die zu entwickelnde Lösung möglichst allgemein einsetzbar sein, da die Kombination von Moodle und ownCloud an zahlreichen Universitäten genutzt wird. Aus diesem Grunde wird nachfolgend nicht mehr von Learnweb und sciebo, sondern von Moodle und ownCloud gesprochen.

## Authentifizierung und Autorisierung
Grundlegend für die Integration beider Systeme ist die Authentifizierung und Autorisierung. Unter den Verfahren, die untersucht wurden, befinden sich:

* [OAuth 2.0](https://oauth.net/2/)
* [JSON Web Tokens](https://jwt.io/)
* [Macaroons](https://research.google.com/pubs/pub41892.html)
* Federated Single Sign-on mit beispielsweise [Shibboleth](https://shibboleth.net/)

Für das Projekt wurde das OAuth 2.0 Verfahren ausgewählt, da mit ihm ein standardisiertes Verfahren zur tokenbasierten Authentifizierung und
Autorisierung vorliegt, das sich gut in bestehende Applikationen auf Basis vieler verschiedener Programmiersprachen einfügen lässt und damit keine weiteren Anforderungen an die Infrastruktur stellt.

## Integrationsszenarien
Zu Beginn wurden verschiedene Integrationsszenarien zwischen den beiden Systemen entwickelt. Diese haben wir nach Schwierigkeit, Interesse, Benutzbarkeit und Implementierungsaufwand priorisiert. Als Integrationsrichtung konzentrierte wir uns dabei auf die Richtung Learnweb <i class="fa fa-long-arrow-right" aria-hidden="true"></i> sciebo.

Es wurden folgende Integrationsszenarien realisiert.

1. Als **Nutzer** möchte ich OAuth 2.0 benutzen können, um Moodle Zugriff auf ownCloud zu gewähren. <p></p> 
Dieses Integrationsszenario behandelt die passwortlose Authentifizierung mit dem OAuth 2.0 Verfahren, was die Grundlage für alle weiteren Szenarien war. Auf Seiten von ownCloud wurde dazu eine [App](/owncloud/technische-umsetzung/oauth2-app) entwickelt und [Anpassungen am ownCloud Core](owncloud/technische-umsetzung/core-anpassungen) durchgeführt. Auf der anderen Seite ist in Moodle ein [Admin Tool](moodle/technische-umsetzung/admin-tool) implementiert worden, das sämtliche Aufgaben zur Authentifizierung und Autorisierung mittels OAuth 2.0 übernimmt. Alle anderen entwickelten Plugins nutzen dieses Tool.
   
2. Als **Nutzer** möchte ich in der Dateiauswahl in Moodle eine Datei aus meiner ownCloud Instanz **hochladen**. </p></p>
Für dieses Szenario wurde für Moodle ein [Repository Plugin](/moodle/technische-umsetzung/repository) entwickelt, mithilfe dessen der Nutzer, nach Autorisierung von Moodle für den Zugriff auf ownCloud, Dateien aus ownCloud hochladen kann.

3. Als **Nutzer** möchte ich in der Dateiauswahl in Moodle eine Datei aus meiner ownCloud Instanz **verlinken**. </p></p>
Eine Erweiterung des vorherigen Szenarios ist die Verlinkung von Dateien. Hierzu wird zu einer Datei ein öffentlicher Link erstellt. Dieser kann daraufhin von Nutzern angeklickt werden. Diese Funktionalität ist auch Teil des [Repository Plugins](/moodle/technische-umsetzung/repository). Das Verlinken ist
jedoch nur in der Moodle-Aktivität „URL“ verfügbar.

4. Als **Lehrender** möchte ich Studierenden oder Gruppen von Studierenden Ordner für kollaboratives Arbeiten bereitstellen. </p></p>
Nach diesem Integrationsszenario wurde vermehrt von Lehrenden der Universität Münster gefragt. Ein Lehrender kann mithilfe eines [Activity Plugins](moodle/technische-umsetzung/activity) für Moodle in einem Kurs Ordner erstellen, auf den Studierende Zugriff haben und den sie zu ihrer ownCloud hinzufügen können.

## Komponenten
Als Überblick werden nachfolgend die entwickelten Komponenten in den beiden Systemen dargestellt.

* **ownCloud**
	* [App `oauth2`](owncloud/technische-umsetzung/oauth2-app/): Implementierung des OAuth 2.0 Protokolls.
	* [Core Anpassungen](owncloud/technische-umsetzung/core-anpassungen/): Ermöglichen den Zugriff auf verschiedene Schnittstellen von ownCloud via OAuth 2.0.
* **Moodle**
	* [Admin Tool `oauth2owncloud`](moodle/technische-umsetzung/admin-tool): Verwaltung der Authentifizierung und Autorisierung mit OAuth 2.0.
	* [Repository Plugin `owncloud`](moodle/technische-umsetzung/repository): Herunterladen und Verlinken von Dateien aus ownCloud.
	* [Activity Plugin `collaborativefolders`](moodle/technische-umsetzung/activity): Erstellen von Ordnern in ownCloud für kollaboratives Arbeiten.

## Weitere Integrationsszenarien
Wegen zeitlicher Restriktionen konnten nicht alle Integrationsszenarien, über deren Implementierung wir nachgedacht haben, realisiert werden. Zukünftige Arbeiten könnten sich jedoch mit folgenden Erweiterungen beschäftigen.

1. Als **Lehrender** möchte ich auf einen Button klicken, um hochgeladene Dateien zu aktualisieren.

2. Als **Studierender** möchte ich Dateien aus Moodle direkt in meiner ownCloud Instanz speichern können.

3. Als **Studierender** möchte ich anderen Studierenden Schreib- und Lese-Rechte geben um kollaborativ zu arbeiten.

4. Als **Lehrender** möchte ich Nutzern auf Modul-Basis das Recht entziehen, eine Datei zu verlinken.

5. Als **Nutzer** möchte ich in der Dateiauswahl in Moodle einen ganzen Ordner aus meiner ownCloud Instanz hochladen.

6. Als **Lehrender** möchte ich in der Dateiauswahl in Moodle einen Ordner aus meiner ownCloud Instanz verlinken.

7. Als **Studierender** möchte ich Lehrenden Schreib- oder Lese-Rechte auf mein Dokument geben können, um Feedback zu erhalten.

8. Als **Lehrender** möchte ich die Möglichkeit haben, einen Zielordner in ownCloud zur Speicherung auszuwählen, um Abgaben herunterzuladen.

9. Als **Lehrender** möchte ich ownCloud als primären Speicher für alle Dateien im Kurs verwenden können.

10. Als **Lehrender** möchte ich in ownCloud die Teilen-Funktion nutzen, um Dateien oder Dokumente für Kursteilnehmer freigeben zu können.

## Zusammenfassung
In diesem Projektseminar wurden die beiden Systeme ownCloud und Moodle integriert. Als Grundlage diente dabei das OAuth 2.0 Protokoll, um eine passwortlose Authentifizierung und Autorisierung zu ermöglichen. Es wurden verschiedene Integrationsszenarien betrachtet und vier von ihnen realisiert. Außerdem wurde die implementierte OAuth 2.0 ownCloud App in die offizielle [ownCloud GitHub Organisation](https://github.com/owncloud/) übernommen, sowie die Anpassungen am ownCloud Core akzeptiert. Das in Kürze erscheinende ownCloud 10.0 wird die Änderungen enthalten und vollständig kompatibel mit der OAuth 2.0 App sein. Durch den akzeptierten Backport wird dies ebenfalls für ownCloud 9.1.5 der Fall sein.

Am 15. März 2017 wurde die Arbeit präsentiert. Nachfolgend finden Sie eine Aufzeichnung der Abschlusspräsentation (wir entschuldigen uns für die Mikrofon-Probleme). Die Folien dazu können Sie sich [hier](https://pssl16.github.io/abschlusspraesentation/) ansehen.

<div align="center">
	<iframe width="560" height="315" src="https://www.youtube.com/embed/2GYvoP36vFc" frameborder="0" allowfullscreen></iframe>
</div>
