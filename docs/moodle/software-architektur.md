# Software Architektur

Zur Umsetzung des Projektziels, musste moodle um nötige Funktionalitäten ergänzt werden, die weder durch den moodle Core 
bereitgestellt werden, noch mit Hilfe von externen Plugins hinzugefügt werden konnten. Im Folgenden wird beschrieben, 
wie moodle im Rahmen des Projektes erweitert wurde und welche Auswirkungen sich daraus ergaben.

## Übersicht über die Plugin-Struktur

Moodle Plugins dienen dazu um im Core angebotene Funktionalitäten dahingehend zu erweitern, sodass diese den 
individuellen Bedürfnissen des Nutzers entsprechen. 
In moodle wird ein plugin einer Kategorie zugeordnet, welche eine Bestimmte Art von Funktionalität repräsentiert. 

Im Laufe des Projekts hat man sich auf folgende zu implementierende Plugins geeinigt:

| Plugintyp                                                   | Beschreibung                                        | Zweck                                                 |
|-------------------------------------------------------------|-----------------------------------------------------|-------------------------------------------------------|
| [`admin tool`](https://docs.moodle.org/dev/Activity_modules)| Bietet Dienste zur Site-Administration an           | Verwaltung der Authentifizierung mittels OAuth2       |
| [`repository`](https://docs.moodle.org/dev/Admin_tools)     | Stellt Verbindung zu einer externen Datenquelle her | Datenbeschaffung aus Sciebo                           |
| [`activity`](https://docs.moodle.org/dev/Admin_tools)       | Stellt Aktivität in einem Kurs zur Verfügung        | Bereitstellung eines Ordners für kollaborative Arbeit |


Zwar bietet das [Repository Plugin](https://docs.moodle.org/dev/Admin_tools) die nötige Funktionalität über die gegebene 
[Schnittstelle](repository/) um Dateien aus Sciebo nach moodle hochzuladen, jedoch kann über die Schnittstelle 
hinaus keine weitere Funktionalität darin implementiert werden. Weil daher nur eine beschränkte Anzahl von 
Integrationsszenarien abgedeckt werden würde, hat man sich für ein ergänzendes [Activity Module](https://docs.moodle.org/dev/Admin_tools) 
enschieden. Basierend auf dieser Entscheidung erschien es als sinnvoll den Authentifizierungsprozess ebenfalls zentral 
zu implementieren, sodass allen zusätzlichen Plugins der Zugriff auf das Verfahren ermöglicht wird.