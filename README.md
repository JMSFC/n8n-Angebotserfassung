# KI-gestützte Angebotserfassung mit Protokollierung und Fehlerbehandlung

## 1. Kurzbeschreibung

Der n8n-Workflow automatisiert die Erfassung und Auswertung von Angeboten in einem Unternehmen.

Angebote im PDF-, DOCX- und PNG-Format werden aus einem überwachten Ordner eingelesen. Der Workflow prüft anhand eines SHA-256-Hashwerts, ob die Datei bereits in Notion erfasst wurde. Neue Angebote werden ausgelesen, mithilfe von Google Gemini strukturiert und positionsweise in einer Notion-Datenbank gespeichert.

Zusätzlich ermöglicht der Workflow einen KI-gestützten Angebotsvergleich sowie die Abfrage gespeicherter Dokumentinhalte über einen KI-Chat mit RAG-Technologie.

**Erweiterung:** Die Angebotserfassung wurde um eine Protokollierung erfolgreicher und fehlgeschlagener Notion-Speichervorgänge sowie eine gezielte Fehlerbehandlung ergänzt.

## 2. Ziel und Nutzen

Ziel ist es, die Angebotserfassung effizienter, nachvollziehbarer und zuverlässiger zu gestalten.

Der Workflow bietet folgende Vorteile:

- **Zeitersparnis:** Automatisierte Erfassung statt manueller Übertragung von Angebotsdaten.
- **Qualitätsverbesserung:** Einheitliche Strukturierung der Angebotsinformationen.
- **Vermeidung von Duplikaten:** Bereits erfasste Dateien werden anhand ihres SHA-256-Hashwerts erkannt.
- **Nachvollziehbarkeit:** Erfolgreiche und fehlgeschlagene Speichervorgänge werden dokumentiert.
- **Fehlererkennung:** Fehler bei der Notion-Speicherung können gezielt untersucht werden.
- **Entscheidungsunterstützung:** Angebote können KI-gestützt ausgewertet und verglichen werden.

Konkrete Zeit- oder Kosteneinsparungen wurden bisher nicht gemessen.

## 3. Beschreibung der Erweiterung

Der ursprüngliche Workflow wurde um folgende Funktionen erweitert:

### 3.1 Protokollierung

Erfolgreiche und fehlgeschlagene Notion-Speichervorgänge werden automatisch in der n8n Data Table `M25_Angebotsprotokoll` erfasst.

Protokolliert werden:

- Dateiname
- Angebotsposition
- Status (Erfolgreich oder Fehler)
- Fehlergrund bei fehlgeschlagener Speicherung
- Notion-ID bei erfolgreicher Speicherung
- SHA-256-DateiHash
- Erstellungs- und Änderungszeitpunkt

### 3.2 Fehlerbehandlung

Der Node `Create a database page` verwendet die n8n-Einstellung **Continue (using error output)**.

Bei erfolgreicher Speicherung wird der Node `Insert row - Erfolgreich` ausgeführt.

Bei einem Speicherfehler wird der separate Fehlerausgang verwendet und der Node `Insert row - Fehler` ausgeführt.

Dadurch werden Notion-Speicherfehler protokolliert, ohne dass ein eigener Fehler-Workflow benötigt wird.

### 3.3 Anpassung der Duplikatsprüfung

Die ursprüngliche Duplikatsprüfung wurde durch einen Abgleich mit den in Notion gespeicherten SHA-256-Hashwerten ersetzt.

Bereits erfasste Dateien werden übersprungen. Die Prüfung berücksichtigt damit auch frühere Workflow-Ausführungen, solange die zugehörigen Notion-Datensätze vorhanden sind.

## 4. Ablauf des Workflows

Der Workflow besteht aus drei Funktionsbereichen.

### 4.1 Automatische Angebotserfassung

1. Der `Schedule Trigger` startet die Überprüfung des überwachten Ordners.
2. Neue und bereits vorhandene Dateien werden eingelesen und mit einem SHA-256-Hashwert versehen.
3. Der Hashwert wird mit den gespeicherten Werten in Notion verglichen. Bereits erfasste Dateien werden übersprungen.
4. Neue Dateien werden entsprechend ihrem Dateiformat verarbeitet:
   - PDF: Textextraktion über `Extract from File`
   - DOCX: Textextraktion über `DOCX to Text`
   - PNG: Texterkennung mithilfe von Google Gemini
5. Der extrahierte Text wird durch einen KI-Agenten analysiert und in ein einheitliches JSON-Schema übertragen.
6. Die einzelnen Angebotspositionen werden in Notion gespeichert.
7. Erfolgreiche oder fehlgeschlagene Notion-Speichervorgänge werden in der n8n-Protokolltabelle dokumentiert.
8. Die Dokumenttexte werden zusätzlich mithilfe von Google Gemini Embeddings im Simple Vector Store für die KI-gestützte Suche bereitgestellt.

### 4.2 KI-gestützter Angebotsvergleich

Dieser Bereich wird manuell gestartet.

Die gespeicherten Angebote werden aus Notion geladen, zusammengefasst und durch einen KI-Agenten verglichen.

Dabei berücksichtigt der Agent unter anderem Preise, Leistungsumfang, Angebotsbedingungen und mögliche Risiken.

Die Ergebnisse werden in einer separaten Notion-Datenbank gespeichert.

### 4.3 KI-Chat mit RAG

Über den Chat können Fragen zu den im Simple Vector Store verfügbaren Angebotsdokumenten gestellt werden.

Ein KI-Agent durchsucht die gespeicherten Dokumentinhalte und erstellt daraus Antworten.

### Workflow-Übersicht

<img width="1768" height="961" alt="Workflow_Uebersicht" src="https://github.com/user-attachments/assets/300064c2-a787-46fe-9566-062f90cdca85" />

*Abbildung 1: Gesamtübersicht des n8n-Workflows mit KI-Chat, Angebotsvergleich und automatisierter Angebotserfassung.*

## 5. Tools und KI-Modelle

| Tool / Technologie | Verwendung |
|---|---|
| n8n | Automatisierung und Steuerung der Workflows |
| Docker | Lokaler Betrieb der n8n-Umgebung |
| Google Gemini | Texterkennung, Angebotsanalyse und Angebotsvergleich |
| Google Gemini Embeddings | Umwandlung von Dokumenttexten in Vektoren |
| Structured Output Parser | Strukturierte Ausgabe der KI-Ergebnisse |
| Notion | Speicherung und Bereitstellung der Angebotsdaten und Vergleiche |
| n8n Data Tables | Protokollierung von Speichervorgängen |
| Simple Vector Store | Temporäre Speicherung und Suche von Dokumentinhalten |
| JavaScript-Nodes | Dateiverarbeitung, Hashwert-Abgleich und Aufbereitung der Daten |

Für die Bildanalyse wird `gemini-3.1-flash-lite` verwendet. Bei den KI-Agenten kommt ebenfalls Google Gemini zum Einsatz, unter anderem `gemini-3.5-flash-lite`.

Google Gemini wurde gewählt, weil sich damit Dokumentinhalte analysieren, strukturieren und für den Angebotsvergleich verarbeiten lassen.

n8n ermöglicht die Integration der unterschiedlichen Komponenten in einem grafischen Workflow. Notion dient als übersichtliche Datenbank für die weitere Bearbeitung.

## 6. Daten

Verarbeitet werden Angebotsdokumente im PDF-, DOCX- und PNG-Format.

Diese können folgende Informationen enthalten:

- Anbieter und Angebotsdatum
- Projektbezeichnungen
- Leistungen, Mengen und Einheiten
- Einzel- und Gesamtpreise
- Ausführungszeiträume
- Vertragsbedingungen und besondere Hinweise

In der n8n-Protokolltabelle werden zusätzlich technische Informationen wie Dateiname, Hashwert, Speicherstatus und Fehlermeldungen gespeichert.

**Datenschutz:** Angebote können vertrauliche Geschäftsinformationen und personenbezogene Daten enthalten, beispielsweise Namen oder Kontaktdaten.

Die Daten werden in Notion gespeichert und für die KI-Verarbeitung teilweise an Google Gemini übertragen. Deshalb müssen vor einem produktiven Einsatz Datenschutz, Zugriffsberechtigungen und die Verarbeitung durch externe Dienstleister geprüft werden.

Für die bisherigen Funktionstests wurden Testangebote verwendet.

## 7. Integration im Unternehmen

Für den betrieblichen Einsatz sind folgende Voraussetzungen erforderlich:

- Eine betriebsbereite n8n-Umgebung mit Zugriff auf den überwachten Dateiordner
- Berechtigte Zugänge zu Notion und Google Gemini
- Eine festgelegte Ordnerstruktur für eingehende Angebote
- Geregelte Zugriffsrechte auf Angebots- und Protokolldaten
- Festgelegte Zuständigkeiten für Betrieb, Fehlerbearbeitung und Wartung

**Organisatorische Verantwortlichkeiten:**

- **IT / Administration:** Betrieb, Wartung, technische Fehlerbehebung und Datensicherung
- **Fachabteilung / Einkauf:** Prüfung der erfassten Angebote und fachliche Freigabe
- **Unternehmensverantwortliche:** Freigabe des produktiven Einsatzes
- **Datenschutzverantwortliche:** Prüfung der datenschutzrechtlichen Anforderungen

Die Rollen müssen vor einem produktiven Einsatz konkret zugewiesen werden.

Die Protokolltabelle sollte regelmäßig kontrolliert werden, da der Workflow keine automatische Benachrichtigung bei Fehlern enthält.

## 8. Governance und Compliance

Beim Einsatz im Unternehmen sind insbesondere folgende Anforderungen zu beachten:

### Datenschutz (DSGVO)

Personenbezogene Daten dürfen nur auf einer geeigneten Rechtsgrundlage und für festgelegte Zwecke verarbeitet werden.

Es müssen insbesondere Datenminimierung, Zugriffsrechte, Aufbewahrungs- und Löschfristen sowie die erforderlichen Vereinbarungen mit externen Dienstleistern geprüft werden.

### Informationssicherheit

API-Schlüssel und Zugangsdaten dürfen nicht öffentlich gespeichert werden.

Der Zugriff auf Notion, n8n und die Angebotsdokumente muss auf berechtigte Personen beschränkt sein. Zusätzlich sind geeignete Datensicherungen und Schutzmaßnahmen erforderlich.

### EU AI Act

Die Anforderungen des EU AI Act sind anhand des konkreten Einsatzzwecks zu prüfen.

Der Workflow dient der Angebotserfassung und Entscheidungsunterstützung. Er trifft keine automatischen Vergabeentscheidungen.

Mitarbeitende müssen über die Möglichkeiten, Grenzen und Risiken der verwendeten KI informiert sein.

### Urheberrecht und Vertraulichkeit

Angebotsdokumente dürfen nur im Rahmen bestehender Berechtigungen und Vereinbarungen verarbeitet werden. Vertrauliche Inhalte dürfen nicht unkontrolliert veröffentlicht werden.

### Mitbestimmung und interne Richtlinien

Eine mögliche Beteiligung des Betriebsrats ist abhängig vom konkreten betrieblichen Einsatz zu prüfen, insbesondere wenn eine Verarbeitung oder Überwachung von Mitarbeiterdaten möglich ist.

Bestehende Unternehmensrichtlinien zur IT-Sicherheit, KI-Nutzung und Datenverarbeitung sind einzuhalten.

**Wichtig:** Die technische Funktionsfähigkeit des Workflows bedeutet nicht automatisch, dass sämtliche rechtlichen Anforderungen erfüllt sind. Vor einem produktiven Einsatz ist eine entsprechende Prüfung notwendig.

## 9. Risiken und Gegenmaßnahmen

| Risiko | Gegenmaßnahme |
|---|---|
| Fehlerhafte KI-Auswertung oder Texterkennung | Fachliche Kontrolle wichtiger Angebotsdaten |
| Fehlerhafte Notion-Speicherung | Protokollierung des Speicherfehlers und manuelle Prüfung |
| Doppelte Angebotserfassung | SHA-256-basierte Duplikatsprüfung |
| Unvollständige Speicherung einzelner Angebotspositionen | Kontrolle der Protokolle und Abgleich mit Notion |
| Unberechtigter Datenzugriff | Rollenbasierte Zugriffsrechte und sichere Zugangsdaten |
| Verlust von Daten | Regelmäßige Datensicherungen |
| Unzutreffende KI-Empfehlungen | Menschliche Prüfung vor Entscheidungen |

**Technische Einschränkungen des aktuellen Workflows:**

Die neue Fehlerprotokollierung erfasst Notion-Speicherfehler, jedoch nicht sämtliche möglichen Fehler der vorgelagerten Verarbeitung.

Bei einer nur teilweise erfolgreichen Angebotsspeicherung kann die bestehende Hashwert-Prüfung eine automatische erneute Verarbeitung verhindern. Solche Fälle müssen anhand der Protokolle kontrolliert und gegebenenfalls manuell korrigiert werden.

Vorhandene Notion-Datensätze ohne DateiHash können zum Abbruch der Duplikatsprüfung führen. Solche Datensätze müssen vor der Verarbeitung geprüft und gegebenenfalls bereinigt werden.

Der verwendete Simple Vector Store speichert Daten nur im Arbeitsspeicher. Bei einem Neustart von n8n gehen diese Daten verloren. Aufgrund der Duplikatsprüfung werden bereits in Notion erfasste Dateien nicht automatisch erneut in den Vektorspeicher aufgenommen.

**Für einen dauerhaften produktiven Unternehmenseinsatz wäre deshalb ein persistenter Vektorspeicher sowie ein geregeltes Wiederherstellungs- und Fehlerbehandlungsverfahren erforderlich.**

## 10. Menschliche Kontrolle

Der Workflow unterstützt die Bearbeitung und den Vergleich von Angeboten, ersetzt aber keine fachliche Entscheidung.

Folgende Ergebnisse müssen durch eine berechtigte Person geprüft werden:

- Vollständigkeit und Richtigkeit der erfassten Angebotsdaten
- Preise, Mengen und Leistungsbeschreibungen
- Besondere Angebotsbedingungen
- Auffälligkeiten und mögliche Fehler
- KI-generierte Bewertungen und Empfehlungen

Die abschließende Auswahl eines Anbieters und die Freigabe einer Beauftragung erfolgen durch die zuständige Fachabteilung beziehungsweise die hierfür verantwortliche Person.

Eine automatische Vergabeentscheidung ist nicht Bestandteil des Workflows.

## 11. Test und Erfolgsmessung

Die Erweiterung wurde anhand vorhandener Angebotsdaten und gezielter Funktionstests überprüft.

### Test 01 – Duplikatsprüfung

**Ziel:** Bereits gespeicherte Angebote dürfen nicht erneut verarbeitet werden.

**Durchführung:** Drei bereits in Notion vorhandene Angebotsdateien wurden erneut eingelesen.

**Ergebnis:** Alle drei Dateien wurden anhand ihrer SHA-256-Hashwerte als Duplikate erkannt. Der Node `Hash-Abgleich mit Notion` gab keine Items aus. Die weitere Verarbeitung wurde nicht gestartet.

**Status:** Bestanden

<img width="1742" height="539" alt="Test_01_Duplikatspruefung" src="https://github.com/user-attachments/assets/f61b51ce-d71a-4dee-acc6-d3d540a56939" />

*Abbildung 2: Erfolgreiche Duplikatsprüfung – bereits gespeicherte Angebote werden erkannt und nicht erneut verarbeitet.*

### Test 02 – Fehlerprotokollierung

**Ziel:** Fehler bei der Notion-Speicherung sollen automatisch protokolliert werden.

**Durchführung:** Ein Testangebot mit einem absichtlich ungültigen Angebotsdatum wurde verarbeitet, um einen Speicherfehler zu provozieren.

**Ergebnis:** Die Notion-Speicherung lieferte einen Fehler. Der Fehlerausgang wurde verwendet und der Node `Insert row - Fehler` erfolgreich ausgeführt.

In der Protokolltabelle wurde folgender Eintrag erstellt:

- Dateiname: `Test_02_Fehlerprotokollierung.docx`
- Angebotsposition: `Testleistung`
- Status: `Fehler`
- Fehlergrund: `Bad request - please check your parameters`
- DateiHash: Vorhanden
- Notion-ID: Nicht vorhanden

**Status:** Bestanden

<img width="1808" height="556" alt="Test_02_Fehlerprotokollierung" src="https://github.com/user-attachments/assets/13e3884e-b35d-4a97-95b7-b10deddcc731" />

*Abbildung 3: Fehlerprotokollierung in der n8n Data Table mit Dateiname, Fehlerstatus und Fehlergrund.*

### Erfolgreiche Angebotserfassung

In der n8n-Protokolltabelle sind zusätzlich 16 erfolgreiche Speichervorgänge dokumentiert. Darunter befinden sich Angebote im PDF-, DOCX- und PNG-Format.

Bei erfolgreichen Speichervorgängen werden die betroffene Angebotsposition, der Dateiname, der DateiHash und die zugehörige Notion-ID dokumentiert.

### Erfolgsmessung

Die wesentlichen technischen Prüfkriterien wurden erfüllt:

- Erkennung bereits gespeicherter Angebotsdateien
- Erfolgreiche Protokollierung von Notion-Speichervorgängen
- Erkennung und Protokollierung eines gezielt ausgelösten Notion-Speicherfehlers

Eine quantitative Messung der Zeitersparnis, Fehlerquote oder Wirtschaftlichkeit wurde nicht durchgeführt.

Die Tests belegen die geprüften Funktionen, jedoch keine vollständige Produktionsreife.

## 12. Installation und Nutzung

### Voraussetzungen

- Docker mit einer funktionsfähigen n8n-Installation
- Zugriff auf den lokalen Dateiordner
- Google-Gemini-API-Zugang
- Notion-Integration mit den erforderlichen Berechtigungen
- Die in Notion benötigten Datenbanken
- n8n Data Table `M25_Angebotsprotokoll`
- Installierter Community-Node `DOCX to Text`

### Einrichtung

1. Die Workflow-JSON-Datei aus diesem Repository in n8n importieren.
2. Die erforderlichen Google-Gemini- und Notion-Zugangsdaten in n8n einrichten beziehungsweise neu zuordnen.
3. Die Notion-Datenbanken `Aufgabe-Angebotszusammenfassung` und `Aufgabe – Angebotsvergleich` bereitstellen. Die Datenbankeigenschaften müssen den Zuordnungen in den Notion-Nodes entsprechen.
4. In n8n die Data Table `M25_Angebotsprotokoll` mit folgenden Spalten bereitstellen:
   - `Dateiname`
   - `Angebotsposition`
   - `Status`
   - `Fehlergrund`
   - `NotionID`
   - `DateiHash`
5. Die Protokollierungs-Nodes mit dieser Data Table verbinden.
6. Den Zugriff auf den Dateiordner sicherstellen. In der verwendeten lokalen Docker-Umgebung wird folgender Ordner genutzt:

   Windows: `C:\TEMP\_n8n-files\M25_Dateitypentest\`

   n8n-Container: `/files/M25_Dateitypentest/`

   Der Windows-Ordner muss über die Docker-Konfiguration im Container unter `/files` verfügbar sein.
7. Den Simple Vector Store mit dem Speicher-Schlüssel `M25_Angebote` verwenden und die erforderlichen Gemini-Embeddings zuordnen.
8. Die Konfiguration und Zugriffsberechtigungen überprüfen.

### Nutzung

**Angebote erfassen:**

Neue Angebotsdateien in den überwachten Ordner kopieren und den Workflow ausführen.

Für den automatischen Betrieb muss der Workflow mit dem Schedule Trigger aktiviert sein.

**Angebote vergleichen:**

Den manuellen Trigger des Angebotsvergleichs ausführen. Die Vergleichsergebnisse werden in der entsprechenden Notion-Datenbank gespeichert.

**KI-Chat nutzen:**

Den Chat-Trigger verwenden, um Fragen zu den aktuell im Vektorspeicher verfügbaren Angeboten zu stellen.

**Fehler kontrollieren:**

Die n8n Data Table `M25_Angebotsprotokoll` öffnen und Einträge mit dem Status `Fehler` prüfen.

### Sicherheitshinweis

Im Git-Repository dürfen keine echten API-Schlüssel, Passwörter, Zugangstoken oder vertraulichen Angebotsdokumente abgelegt werden.

Zugangsdaten werden ausschließlich über die dafür vorgesehenen n8n-Credentials eingerichtet.

---

**Fazit:** Der bestehende Workflow wurde erfolgreich um eine Protokollierung und Fehlerbehandlung bei der Notion-Speicherung erweitert. Die Erweiterung erhöht die Nachvollziehbarkeit und erleichtert die Identifikation fehlgeschlagener Speichervorgänge. Die geforderten Funktionen wurden anhand praktischer Tests überprüft.

Für einen dauerhaften Unternehmenseinsatz sind die beschriebenen technischen Einschränkungen sowie die organisatorischen und rechtlichen Voraussetzungen zu berücksichtigen.
