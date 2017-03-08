# Softwarearchitektur

Da wir möglichst keine neue Schnittstelle implementieren wollten, war es das Ziel, die bestehenden Schnittstellen in ownCloud um OAuth 2.0 zu erweitern. Zur Umsetzung der Integrationsszenarien 

WebDAV Schnittstelle um OAuth 2.0 zu erweitern. Die WebDAV Schnittstelle ist als [ownCloud App](https://doc.owncloud.org/server/latest/developer_manual/app/) realisiert worden. Zusätzlich sollte die OCS S

Auf der anderen Seite musste das OAuth 2.0 Protokoll mit seinen Schnittstellen bereitgestellt werden, um die Authentifizierung in der WebDAV App um OAuth 2.0 zu erweitern zu können. Dafür wurde eine weitere ownCloud App implementiert.
