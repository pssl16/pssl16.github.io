# Softwarearchitektur

Zur Umsetzung des Projektziels musste Moodle um nötige Funktionalitäten ergänzt werden. Im Folgenden erhalten Sie eine strukturierte Übersicht zu den Plugins, die in Moodle entwickelt wurden und welche Funktionalitäten sie bereit stellen.

## Übersicht über die Plugin-Struktur

Moodle [Plugins](https://moodle.org/plugins/) dienen dazu im Core angebotene Funktionalitäten dahingehend zu erweitern, dass diese den
individuellen Bedürfnissen des Nutzers entsprechen. Daher eignete sich die Implementierung solcher Plugins ideal zur Umsetzung der definierten Integrationsszenarien.
In Moodle wird ein Plugin einer [Kategorie](https://docs.moodle.org/dev/Plugin_types) zugeordnet, welche eine bestimmte Art von Funktionalität repräsentiert.
Da unsere Integrationsszenarien verschiedene Arten von Funktionalitäten benötigen haben wir uns für verschiedene Plugintypen entschieden.


| Plugintyp                                                   | Beschreibung                                        | Zweck für das Projekt                                 |
|-------------------------------------------------------------|-----------------------------------------------------|-------------------------------------------------------|
| [`admin tool`](https://docs.moodle.org/dev/Admin_tools) oauth2owncloud| Bietet Dienste zur Site-Administration an, d.h. die Einstellungen gelten global und können nur vom Administrator der Seite getätigt werden           | Verwaltung der Authentifizierung mittels OAuth 2.0    |
| [`repository`](https://docs.moodle.org/dev/Repository_plugins) owncloud| Stellt die Verbindung zu einer externen Datenquelle her | Datenbeschaffung aus ownCloud                           |
| [`activity`](https://docs.moodle.org/dev/Activity_modules) collaborativefolders| Stellt Aktivitäten in einem Kurs zur Verfügung        | Erstellung von Ordnern für kollaboratives Arbeiten |


## Funktionsübersicht

Die in den verschiedenen Plugins angebotenen Funktionalitäten können wie folgt zusammengefasst werden:

* **`Admin Tool`:** [`oauth2owncloud`](admin-tool/)
    * Umfasst sowohl OAuth 2.0, als auch einen WebDAV Client.
    * Steuert den Protokollablauf von OAuth 2.0 und verwaltet alle dazu nötigen Informationen.
        * globale Speicherung des Secrets, der ClientID und der WebDAV Zugangsdaten
    * Stellt das Verbindungsstück von moodle zu ownCloud zur Verfügung
* **`Repository`:** [`ownCloud`](repository/)
    * Bewerkstelligt die Datenbeschaffung aus ownCloud nach Moodle.
    * Ermöglicht den Upload von Dateien aus einer persönlichen ownCloud Instanz.
    * Ermöglicht die Verlinkung von Dateien aus ownCloud in Moodle.
* **`Activity Module`:** [`collaborative folders`](activity/)
    * Ermöglicht die Erstellung und Freigabe von Ordern in ownCloud für Kursteilnehmer in Moodle.
    * Ermöglicht das Erzeugen von separaten Ordnern für Gruppen.
    * Ermöglicht den Zugriff des Lehrenden auf diese Ordner sowie das Verbot des Zugriffes.

## Abhängigkeiten

Die aus der Aufteilung der Funktionen in verschiedene Plugins resultierenden Abhängigkeiten werden in folgender Abbildung
dargestellt:


![Plugin-Struktur](images/plugin-struktur.svg)

Zu beachten ist, dass eine möglichst hohe Flexibilität und Modularität bei dem Entwurf der Softwarearchitektur erzielt
werden sollte. Das führt zwar mit sich, dass funktionale Plugins (also das `repository` und das `activity` Plugin) nicht
eigenständig ohne das `admin tool` existieren können, allerdings wird weiteren, in Zukunft entwickelten Plugins ebenfalls
Zugriff auf die OAuth 2.0 Schnittstelle in ownCloud bzw. sciebo ermöglicht,
wodurch sich das Projektergebnis zu einer guten Wiederverwendbarkeit qualifiziert.
