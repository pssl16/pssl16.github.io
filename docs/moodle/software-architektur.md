# Software Architektur

[Moodle](https://moodle.de/) ist eine Open Source Online Lehr- und Lernplattform in Form einer Webapplikation, welche an zahlreichen 
Universitäten und diversen anderen Institutionen im Bildungssektor weltweit als Kommunikationswerkzeug eingesetzt wird. 

Zur Umsetzung des Projektziels, musste moodle um nötige Funktionalitäten ergänzt werden, die weder durch den [moodle Core](https://github.com/moodle/moodle)
bereitgestellt werden, noch mit Hilfe von externen Plugins hinzugefügt werden konnten. Im Folgenden wird beschrieben, 
wie moodle im Rahmen des Projektes erweitert wurde und welche Auswirkungen sich daraus ergaben.

## Übersicht über die Plugin-Struktur

Moodle [Plugins](https://moodle.org/plugins/) dienen dazu um im Core angebotene Funktionalitäten dahingehend zu erweitern, sodass diese den 
individuellen Bedürfnissen des Nutzers entsprechen. Daher eignete sich die Implementierung solcher Plugins ideal zur 
Umsetzung der definierten Integrationsszenarien.
In moodle wird ein plugin einer Kategorie zugeordnet, welche eine Bestimmte Art von Funktionalität repräsentiert. 

Im Laufe des Projekts hat man sich auf folgende zu implementierende Plugins geeinigt:

| Plugintyp                                                   | Beschreibung                                        | Zweck für das Projekt                                 |
|-------------------------------------------------------------|-----------------------------------------------------|-------------------------------------------------------|
| [`admin tool`](https://docs.moodle.org/dev/Activity_modules)| Bietet Dienste zur Site-Administration an           | Verwaltung der Authentifizierung mittels OAuth 2.0    |
| [`repository`](https://docs.moodle.org/dev/Admin_tools)     | Stellt Verbindung zu einer externen Datenquelle her | Datenbeschaffung aus Sciebo                           |
| [`activity`](https://docs.moodle.org/dev/Admin_tools)       | Stellt Aktivität in einem Kurs zur Verfügung        | Bereitstellung eines Ordners für kollaborative Arbeit |


Zwar bietet das [Repository Plugin](https://github.com/pssl16/moodle-repository_sciebo) die nötige Funktionalität über die gegebene 
[Schnittstelle](repository/) um Dateien aus Sciebo nach moodle hochzuladen, jedoch kann über die Schnittstelle 
hinaus keine weitere Funktionalität darin implementiert werden. Weil daher nur eine beschränkte Anzahl von 
Integrationsszenarien abgedeckt werden würde, hat man sich für ein ergänzendes [Activity Module](https://github.com/pssl16/moodle-mod_collaborativefolders) 
enschieden. Basierend auf dieser Entscheidung erschien es als sinnvoll den [Authentifizierungsprozess](https://github.com/pssl16/moodle-tool_oauth2sciebo) ebenfalls zentral 
zu implementieren, sodass allen zusätzlichen Plugins der Zugriff auf das Verfahren ermöglicht wird.

## Abhängigkeiten

Die aus der Aufteilung der Funktionen in verschiedene Plugins resultierenden Abhängigkeiten werden in folgender Abbildung
dargestellt:

![Plugin-Struktur](images/plugin-struktur.svg)

Zu beachten ist, dass eine möglichst hohe Flexibilität und Modularität bei dem Entwurf der Software Architektur erzielt
werden sollte. Das führt zwar mit sich, dass funktionale Plugins (also das `repository` und das `activity` Plugin) nicht
eigenständig ohne das `admin tool` existieren können, allerdings wird weiteren, in Zukunft entwickelten Plugins ebenfalls
Zugriff auf die OAuth 2.0 Schnittstelle in [Sciebo](https://www.sciebo.de/) bzw. [ownCloud](https://owncloud.org/) ermöglicht,
wodurch sich das Projektergebnis zu einer guten Wiederverwendbarkeit qualifiziert.

## Funktionsübersicht

Die in den verschiedenen Plugins angebotenen Funktionalitäten können wie folgt zusammengefasst werden:

* **`Admin Tool`:** [`oauth2sciebo`](admin-tool/)
    * Umfasst sowohl OAuth 2.0, als auch WebDAV Client.
    * Steuert Protokollablauf von OAuth 2.0 und verwaltet alle dazu nötigen Informationen.
    * Stellt das Verbindungsstück von moodle zu Sciebo bzw. ownCloud
* **`Repository`:** [`sciebo`](repository/)
    * Bewerkstelligt die Datenbeschaffung aus Sciebo bzw. ownCloud nach moodle.
    * Ermöglicht den Upload von Dateien aus einer persönlichen Sciebo Instanz.
    * Ermöglicht die Verlinkung von Dateien aus Sciebo in moodle.
* **`Activity Module`:** [`collaborative folders`](activity/)
    * Ermöglicht die Erstellung und Freigabe von Ordern in Sciebo für bestimmte Gruppen in moodle.
    
<div class="alert alert-danger">
  <strong>TODO:</strong> Liste im Laufe weiterer Sprints erweitern.
</div>