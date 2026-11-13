# literate-fishstick

# Consistency Audit 0.3 – Windows-Prüfwerkzeug für Quellcodeprojekte

**Zweck:** Projektordner automatisch und ausschließlich *statisch* untersuchen. Das Programm liest Dateien, erstellt eine Architektur- und Risikoinventur und liefert pro Befund einen kopierbaren Untersuchungs- und Behebungsauftrag. **Es startet keinerlei Unit-Tests, Build-, Installations- oder Projektprogramme**, verändert keine Projektdateien, ruft keine Cloud-Dienste auf und benötigt keine API-Schlüssel.

Die Ausgabe ist eine **Triage**, kein Beweis der vollständigen Fehlerfreiheit. Insbesondere ist der Unterschied zwischen einem nachgewiesenen syntaktischen/strukturellen Befund, einem plausiblen Code-Indikator und einer offenen fachlichen Frage ausdrücklich angegeben.

## Windows: Doppelklick

**Voraussetzung:** Python 3.9 oder neuer mit `py -3` oder `python`. Das Werkzeug selbst benötigt keine zusätzlichen Pakete.

1. ZIP entpacken, z. B. nach `C:\Tools\ConsistencyAudit`.
2. `SCAN_STARTEN.bat` doppelklicken und den Ordner des zu untersuchenden Projekts eingeben. Alternativ: Projektordner auf `SCAN_STARTEN.bat` ziehen.
3. Der HTML-Bericht wird nach dem Scan möglichst automatisch im lokalen Browser geöffnet. `behebungsauftraege.md` enthält alle kopierbaren Behebungsaufträge.

**Bei der Standardausführung landen Berichte außerhalb des Projekts:** unter `%LOCALAPPDATA%\ConsistencyAudit\reports\<Projektname_Zeitkennung>`; im Terminal wird der exakte Speicherort ausgegeben. Jeder Durchlauf erhält einen separaten Unterordner und überschreibt nicht die Ergebnisse früherer Durchläufe.

## Terminal

```bat
py -3 "C:\Tools\ConsistencyAudit\audit.py" scan "C:\Projekte\SevenVoices"
py -3 "C:\Tools\ConsistencyAudit\audit.py" scan . --open-report
py -3 "C:\Tools\ConsistencyAudit\audit.py" scan . --profile seven-voices-stage1
py -3 "C:\Tools\ConsistencyAudit\audit.py" scan . --profile seven-voices-stage2
py -3 "C:\Tools\ConsistencyAudit\audit.py" scan . --profile seven-voices-midi
py -3 "C:\Tools\ConsistencyAudit\audit.py" scan . --output "C:\Berichte\SevenVoices"
```

**Profile:** `auto` setzt keine Agentenparallelität voraus. Stufe 1 modelliert eine manuell freigegebene Reihenfolge, Stufe 2 die sequenzielle gemeinsame Bearbeitung, `seven-voices-midi` den optional möglichen Zugriff auf gemeinsam genutzte DAW-Ressourcen. Profile formulieren Prüfannahmen und Fragen, **keine Behauptungen über die tatsächlich implementierte Steuerung**. Die drei Stufen können auch getrennt bewertet werden; der Standardscan `auto` eignet sich für beliebige Projekte.

## Was geprüft wird

### Bestehende Prüffelder aus v0.2

- Python-Syntax, JSON-Konfiguration, Testdatei-Inventur, Dokumentation.
- Potenzielle Secrets, unsichere TLS-Konfiguration, dynamische Ausführung, verdächtige Deserialisierung, Logausgaben sensibler Felder.
- Eingabe/Verarbeitung/Ausgabe, Ein- und Ausgabesenken, Vertrauensgrenzen, Eingaben in Datenbank- oder Dateizugriffe.
- HTTP-Aufrufe, Timeouts, Wiederholungsmuster, Fehlerbehandlung und nebenläufige Aufrufe.
- Indikatoren für Datenbanktransaktionen, Dual Writes, Event-Veröffentlichungen und unklare Nachrichtenbestätigung.
- 26 Themen des begleitenden Konsistenz-Referenzwerks mit einer ehrlichen Abdeckungsmatrix.

### Erweiterungen in v0.3

- **Projektweiter Importgraph:** statisch erkennbare Importzyklen in lokalen Python- und JavaScript/TypeScript-Modulen.
- **Funktionsbezogene Prüfhypothesen:** Commit und Event-Veröffentlichung ohne lokal erkennbaren Outbox-Bezug.
- **Kontrolle des Abschlusses asynchroner Arbeit:** nicht verwaltete `asyncio.create_task()`-Aufrufe oder verworfene `gather()`-Ergebnisse.
- **AST-basierte Fehlerbehandlung:** stumm verschluckte Exceptions einschließlich mehrzeiliger `except`-Blöcke.
- **Einfache Datenfluss-Näherung:** externe Eingaben, die in derselben Python-Funktion über lokale Variablen in dynamisch aufgebautes SQL eingehen könnten.
- **Offene Architekturverträge:** projektabhängige Fragen zu fachlichen Invarianten, Zustandsübergabe, Idempotenz, MCP-/MIDI-Ressourcenrechten und Reconciliation. Diese Fragen werden explizit **nicht** als gefundene Fehler gewertet.
- **Separate Berichtablage:** keine Berichte im Quellcodeprojekt bei Standardausführung. Rohquelltext wird in Berichten nicht ausgegeben; Fundstellen beziehen sich auf lokale Datei und Zeile.

## Berichte

| Datei | Inhalt |
| --- | --- |
| `report.html` | Lokale HTML-Ansicht mit Risikoindikatoren, offenen Prüfaufträgen und Fundstellen |
| `report.md` | Alle Befunde sowie einzeln kopierbare Untersuchungs- und Behebungsaufträge |
| `report.json` | Strukturierte Ergebnisse, Status und Abdeckungsmatrix für spätere Erweiterungen |
| `behebungsauftraege.md` | Alle Befunde als separat kopierbare Prompts mit konkreten Fundorten und Freigabevorbehalt |

**Ein Behebungsauftrag ist kein automatisches Reparaturkommando.** Die Vorprüfung, Planung einer minimal-invasiven Änderung, Bewertung von Nebenwirkungen und das Einholen einer gesonderten Freigabe sind ausdrücklich enthalten.

## Betriebsgrenzen und Datenschutz

- Projektcode wird **nie ausgeführt**, es werden keine Tests gestartet und keine Netzwerkverbindungen durch das Werkzeug hergestellt.
- Es folgen keine symbolischen Links und (soweit die Python-Version dies unterstützt) keine Windows-Junctions.
- Übliche Paket-, Build-, Cache- und Versionsverwaltungsverzeichnisse werden übersprungen.
- Maximal **8.000 Dateien**, **768 KiB je Datei** und **128 MiB gelesener Gesamtinhalt** pro Scan. Übersprungene Dateien werden gezählt.
- Komplexe Konfiguration, Framework-Magie, dynamische Imports, Laufzeitfehler und tatsächliche Konsistenzgarantien können statisch unentdeckt bleiben.
- Kein Modell- oder KI-Zugriff, keine CodeQL-, Semgrep-, Trivy-, Shodan- oder Notion-Anbindung in dieser Version; sie dürfen nicht als geprüft gelten.
- Protokollierte Dateipfade sowie Befundbeschreibungen können Rückschlüsse auf die Projektstruktur geben. Berichte deshalb als projektsensibel behandeln.

## Konkreter Entwicklungsstand

Version 0.3 ist ein **ausführbarer statischer Erstprüfer**, kein vollumfängliches Datenflussanalyse-, Modellprüf- oder Security-Zertifizierungswerkzeug. Einige Prüfungen erfolgen allein über reguläre Ausdrücke, andere über Python-AST und einen lokalen Importgraphen. Neue projektspezifische Regeln und externe Analyseadapter sollten erst nach genauer Spezifikation und gesonderter Freigabe implementiert werden.

