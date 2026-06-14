# MeshCore Rosenheim – Konfiguration für Neueinsteiger

## Grundlagen

**Was ist MeshCore?**
MeshCore ist ein dezentrales LoRa-Funknetz. Geräte kommunizieren direkt miteinander ohne Internet oder Mobilfunk – ideal für Outdoor, Notfunk und lokale Kommunikation.

**Was ist ein Repeater?**
Ein Repeater ist ein ortsfester Knoten, der Nachrichten empfängt und weiterleitet. Er vergrößert die Reichweite des Netzes und läuft in der Regel dauerhaft.

**Was ist ein Companion?**
Ein Companion ist ein mobiles Endgerät (z. B. mit Heltec oder LILYGO-Board). Damit können Nachrichten gesendet und empfangen werden – verbunden über Bluetooth oder USB.

> **Wichtig:** Alle Geräte im selben Netz müssen identische Frequenz- und Funkkonfiguration verwenden. Abweichungen führen dazu, dass Geräte sich gegenseitig nicht hören können.

---

## Frequenz & Funkparameter

In Deutschland gilt das 869 MHz ISM-Band (EU-Vorschriften, Subband 869.4–869.65 MHz). Das Gerät muss für diesen Bereich konfiguriert sein.

| Parameter        | Wert                | Hinweis                              |
|------------------|---------------------|--------------------------------------|
| Frequenzband     | 869 MHz             | EU-ISM-Subband 869.4–869.65 MHz      |
| Frequenz         | 869.618 MHz         | Festgelegter Wert für DE             |
| Spreading Factor | SF8                 | Festgelegter Wert für DE             |
| Bandwidth        | 62.5 kHz            | Festgelegter Wert für DE             |
| Coding Rate      | CR8                 | Festgelegter Wert für DE             |
| TX Power         | max. 25 mW (14 dBm) | Gesetzlich erlaubtes Maximum in DE   |

---

## Regions

Regions sind in MeshCore hauptsächlich für Flood-Nachrichten relevant – also Nachrichten in öffentlichen Channels, die von Repeater zu Repeater weitergeleitet werden. Beim Senden einer Nachricht in einem öffentlichen Channel wird ein Region-Scope mitgegeben, der festlegt, wie weit die Nachricht geflutet wird. Repeater können so konfiguriert werden, dass sie nur Pakete bestimmter Regions weiterleiten – Pakete anderer Regions werden verworfen.

Direkte Nachrichten zwischen zwei Geräten (Peer-to-Peer) sind von der Regions-Einstellung nicht betroffen.

| Region    | Gebiet                      | Beschreibung                                                                  |
|-----------|-----------------------------|-------------------------------------------------------------------------------|
| `de-by-ro` | Landkreis Rosenheim *(empfohlen)* | Empfohlene Region für öffentliche Channels im Landkreis Rosenheim inkl. Stadt Rosenheim. |
| `de-sued` | Oberbayern Süd / Alpenraum  | Region für den gesamten südlichen Bereich Bayerns – Alpenvorland, Inn-Salzach, Chiemgau, Berchtesgadener Land. |
| `de-by`   | Bayern übergreifend         | Bayernweite Basisregion für Flood-Nachrichten mit größerer Reichweite.        |
| `de-by-muc` | Grenzgebiet München       | Relevant für Standorte im Grenzbereich Richtung München (z. B. Rosenheimer Nordlandkreis, nahe A8). |

> **Tipp:** Beim Senden in einem öffentlichen Channel sollte die engste passende Region gewählt werden, um unnötigen Flood-Traffic im Netz zu vermeiden.

---

## Channels

Channels sind Gesprächsgruppen innerhalb des Meshs. Der Channel-Name muss bei allen Teilnehmern exakt gleich geschrieben sein (Groß-/Kleinschreibung beachten).

| Parameter    | Wert / Hinweis                                    |
|--------------|---------------------------------------------------|
| Channel-Name | `#rosenheim` (Hashtag und Kleinschreibung beachten) |
| Zweck        | Lokale Kommunikation im Raum Rosenheim            |

> **Hinweis:** Hashtag-Channels benötigen keinen zusätzlichen Schlüssel. Der Channel-Name allein reicht aus – alle Teilnehmer, die denselben Namen eingeben, sind in derselben Gruppe.

---

## Repeater-Konfiguration

> **Hinweis:** Repeater können keine Channels abonnieren. Die Steuerung, welche Pakete weitergeleitet werden, erfolgt ausschließlich über Flood-Region-Regeln (siehe unten).

### Basiseinstellungen

| Parameter        | Wert                  | Hinweis                                                                 |
|------------------|-----------------------|-------------------------------------------------------------------------|
| Gerätemodus      | `REPEATER`            | Im Firmware-Setup als „Repeater" oder „Router" wählen                  |
| Frequenz         | 869.618 MHz           | Auf Übereinstimmung mit Nachbarknoten achten                            |
| Spreading Factor | SF8                   | Festgelegter Wert für DE                                                |
| Bandwidth        | 62.5 kHz              | Festgelegter Wert für DE                                                |
| Coding Rate      | CR8                   | Festgelegter Wert für DE                                                |
| TX Power         | 14 dBm (25 mW)        | Gesetzliches Maximum in DE – nicht überschreiten                        |
| path.hash.bytes  | 2                     | 2 Byte für Path-Hashes – spart Paket-Overhead                           |
| Knotenname       | frei wählbar          | Schema: `DE.BY.RO <Knotenname>`, z. B. `DE.BY.RO Hochries`             |
| GPS / Position   | wenn möglich aktiv    | Ermöglicht Kartendarstellung für andere Nutzer                          |

### Flood-Region-Regeln

Über Flood-Region-Regeln wird festgelegt, welche Pakete der Repeater weiterleitet und welche er verwirft. Die Regeln werden der Reihe nach geprüft – die erste passende Regel greift.

| Prio | Regel              | Aktion       | Hinweis                                                                           |
|------|--------------------|--------------|-----------------------------------------------------------------------------------|
| 1    | ohne Region-Scope  | flood DENY   | Pakete ohne Regionsangabe werden nicht weitergeleitet. Schützt vor unkontrollierter netzweiter Ausbreitung. |
| 2    | `de-by`            | flood ALLOW  | Pakete mit bayernweitem Scope werden weitergeleitet.                              |
| 3    | `de-by-ro`         | flood ALLOW  | Pakete mit Rosenheimer Scope werden weitergeleitet.                               |
| 4    | `de-by-muc`        | flood ALLOW  | Optional – nur für Repeater im Grenzgebiet Richtung München relevant.             |

### Advert-Intervalle

| Parameter                  | Wert     | Hinweis                                                                                                 |
|----------------------------|----------|---------------------------------------------------------------------------------------------------------|
| Flood Advert Intervall     | 150 h    | Alle 150 Stunden sendet der Repeater einen Flood-Advert ins gesamte Netz – informiert entfernte Knoten über seine Existenz. |
| Zero-Hop Advert Intervall  | 240 min  | Alle 240 Minuten (4 Stunden) sendet der Repeater einen lokalen Advert nur an direkt erreichbare Nachbarknoten (0 Hops). |

### Flood-Limits

| Parameter          | Wert | Hinweis                                                                                              |
|--------------------|------|------------------------------------------------------------------------------------------------------|
| `flood.max`        | 12   | Maximale Hop-Anzahl für allgemeine Flood-Pakete – begrenzt die netzweite Ausbreitung.                |
| `flood.max.unscoped` | 5  | Maximale Hops für Pakete ohne Scope-Einschränkung – schützt vor unkontrollierter regionübergreifender Ausbreitung. |
| `flood.max.advert` | 8    | Maximale Hops für Advert-Pakete – Adverts werden weniger weit geflutet als normale Nachrichten, um den Kanal zu schonen. |

---

## Companion-Konfiguration

| Parameter        | Wert                    | Hinweis                                               |
|------------------|-------------------------|-------------------------------------------------------|
| Gerätemodus      | `COMPANION` / `CLIENT`  | Je nach Firmware-Version unterschiedlich bezeichnet   |
| Frequenz         | 869.618 MHz             | Identisch mit Repeater-Einstellung                    |
| Spreading Factor | SF8                     | Gleicher Wert wie bei den lokalen Repeatern           |
| Bandwidth        | 62.5 kHz                | Gleicher Wert wie bei den lokalen Repeatern           |
| Coding Rate      | CR8                     | Gleicher Wert wie bei den lokalen Repeatern           |
| TX Power         | 14 dBm (25 mW)          | Gesetzliches Maximum – Firmware begrenzt automatisch  |
| path.hash.bytes  | 2                       | Muss mit den Repeatern übereinstimmen (ebenfalls 2 Byte) |
| Knotenname       | Rufzeichen / Spitzname  | Wird anderen Nutzern im Netz angezeigt                |
| Channels         | `#rosenheim`            | Weitere Channels nach Bedarf hinzufügen               |
| Verbindung       | Bluetooth (BLE)         | MeshCore-App für Android/iOS über Bluetooth koppeln   |

---

## Weiterführende Anleitungen & Dokumentation

### Deutschsprachige Ressourcen (Community)

- **[MeshCore Wiki DE – Startseite](https://meshcore-de.fyi/)**
  Umfassendes deutschsprachiges Wiki der MeshCore-Community. Enthält Anleitungen zu Firmware, Regions, Scopes, Gerätekompatibilität und vielem mehr.

- **[MeshCore Wiki DE – Handbuch](https://meshcore-de.fyi/meshcore:handbuch)**
  Das deutschsprachige Handbuch als zentraler Einstiegspunkt für alle Konfigurationsthemen.

- **[MeshCore Wiki DE – Firmware flashen](https://meshcore-de.fyi/meshcore:allgemeines:meshcore-firmware-flashen)**
  Schritt-für-Schritt-Anleitung zum Flashen der MeshCore-Firmware auf ESP32- und nRF52-basierten Geräten, inklusive Hinweisen zu USB-Kabeln und Treibern.

- **[MeshCore Wiki DE – Regions & Scopes](https://meshcore-de.fyi/meshcore:allgemeines:regions)**
  Erklärt das Regions- und Scope-Konzept ausführlich: warum Regions wichtig sind, wie Flood-Limits funktionieren und wie die Community-Konsensregeln entstanden sind.

- **[MeshCore Wiki DE – Deutschlandweite Basis-Regions](https://meshcore-de.fyi/meshcore:allgemeines:regions:basis)**
  Übersicht der deutschlandweit einheitlich verwendeten Basis-Regions (`europe`, `de`, `de-by` usw.) und ihrer Bedeutung für die Netzwerk-Interoperabilität.

- **[MeshCore Wiki DE – FAQ](https://meshcore-de.fyi/meshcore:faq)**
  Häufige Fragen zur MeshCore-Technik auf Deutsch, u. a. zu Routing, Schlüsseln, Repeater-Konfiguration und Störungen.

- **[Repeater aufbauen – HanseMesh](https://hansemesh.de/anleitungen/repeater-aufbauen/)**
  Praxisnahe deutschsprachige Anleitung zum Bau und zur Konfiguration eines Repeaters: Firmware, Scopes/Regionen setzen, Standortwahl und Montage-Tipps.

- **[Repeater-Setup – Mesh Rheinland](https://www.meshrheinland.de/meshcore/repeater-setup)**
  Kompakte Konfigurationsreferenz mit CLI-Befehlen speziell für deutsche Repeater, inklusive Beispielen für Frequenz, Duty Cycle und Advert-Intervall.

- **[MeshCore für Einsteiger – LocalMesh DE](https://www.localmesh.de/meshcore-einrichten/)**
  Deutschsprachige Einsteiger-Anleitung für den Companion: Gerät flashen, App koppeln und ins deutsche MeshCore-Netz einsteigen.

- **[MeshCore Repeater-Einstellungen – LocalMesh DE](https://www.localmesh.de/meshcore-repeater-einstellungen/)**
  Deutschsprachige Anleitung speziell für Repeater: Flashen, Preset wählen und häufige Fehler vermeiden.

- **[CLI-Konfiguration – LocalMesh DE](https://www.localmesh.de/meshcore-cli-konfiguration/)**
  Anleitung zur Konfiguration von Repeatern über die serielle Konsole und das meshcore-cli Python-Tool, auf Deutsch.

### Offizielle Ressourcen (englisch)

- **[MeshCore Dokumentation](https://docs.meshcore.io/)**
  Offizielle englischsprachige Dokumentation mit FAQ, Protokollbeschreibungen und CLI-Referenz.

- **[CLI-Befehlsreferenz für Repeater & Room Server](https://docs.meshcore.io/cli_commands/)**
  Vollständige Übersicht aller Konsolenbefehle: Frequenz, Regionen, Flood-Limits, Advert-Intervalle und mehr.

- **[MeshCore Web-Flasher](https://flasher.meshcore.io/)**
  Firmware direkt im Browser flashen – kein lokales Tool nötig (Chromium-Browser erforderlich).

- **[MeshCore Web-Konfigurator für Repeater](https://config.meshcore.io/)**
  Repeater nach dem Flashen über USB im Browser konfigurieren, ohne Kommandozeile.

---

## Hinweise & Rechtliches

Das 869-MHz-ISM-Subband (869.4–869.65 MHz) ist in Deutschland lizenzfrei nutzbar (§ 55 TKG, Allgemeinzuteilung). Die TX-Leistung von maximal 25 mW (14 dBm) ist einzuhalten.

*Diese Seite ist eine Community-Ressource und keine offizielle Veröffentlichung des MeshCore-Projekts.*
