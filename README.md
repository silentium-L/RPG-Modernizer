# RPG-Modernizer

> Ein KI-Agent zur Modernisierung von IBM i RPG-Anwendungen — von Legacy-Code zu Fully Free RPG und modernem DB2 SQL.

---

## Überblick

RPG-Modernizer ist ein agentenbasiertes System, das Entwicklungsteams dabei unterstützt, bestehende IBM i RPG-Anwendungen schrittweise zu modernisieren. Der Agent vereint spezialisierte Skills für Code-Konvertierung, Analyse, Refactoring und SQL-Optimierung in einem einheitlichen Werkzeug.

Das Ziel ist keine vollautomatische Konvertierung — die ist bei Legacy-RPG unrealistisch — sondern eine **hochwertige, assistierte Modernisierung**, bei der der Agent die schwere Arbeit übernimmt und der Entwickler die fachliche Kontrolle behält.

---

## Warum modernisieren?

IBM i RPG ist in vielen Unternehmen tief verwurzelt. Gleichzeitig sorgt der Fachkräftemangel für einen steigenden Druck, bestehenden Code wartbarer zu machen:

| Problem | Auswirkung |
|---|---|
| Spaltenbasierter Fixed-Format-Code | Schwer lesbar, fehleranfällig, keine IDE-Unterstützung |
| Implicit Indicators (`*IN50`, `*INLR`) | Versteckte Programmlogik, kaum testbar |
| Embedded SQL ohne Parameter | SQL-Injection-Risiko, kein Query-Plan |
| Monolithische Cycle-Programme | Nicht modularisierbar, kein Unit-Testing |
| Undokumentierte GOTO-Strukturen | Wartungs-Alptraum |

Modern geschriebenes RPG — Fully Free mit strukturierten Prozeduren und sauberem SQL — ist deutlich wartbarer und lässt sich mit modernen Tools wie RDi, VS Code (Code for IBM i) und Git entwickeln.

---

## RPG-Versionslandschaft

RPG hat über Jahrzehnte mehrere Evolutionsstufen durchlaufen. Für die Modernisierung ist das Verständnis dieser Stufen entscheidend:

```
RPG II (1969)
  └─ RPG III (1979)
       └─ RPG/400 (1988) — Klassisches AS/400-RPG
            └─ RPG IV / ILE RPG (1994) — Fixed Format, H/F/D/C/O/P-Specs
                 ├─ Partial Free (/FREE ... /END-FREE) — ab V5R1 (2001)
                 └─ Fully Free RPG (**FREE) — ab IBM i 7.1 TR7 / 7.2 (2013)
```

### Übersicht der Unterschiede

| Merkmal | RPG/400 | RPG IV Fixed | Fully Free RPG |
|---|---|---|---|
| Spaltenstruktur | 1–80, fest | 1–100, fest | Keine |
| Freie Ausdrücke | Nein | Teilweise | Ja |
| Prozedurale Struktur | Cycle only | Procedures | Procedures |
| Modulares Design | Eingeschränkt | ILE-fähig | Voll ILE-fähig |
| Leserlichkeit | Gering | Mittel | Hoch |
| IDE-Unterstützung | Kaum | Eingeschränkt | Gut |

---

## Skills

Der Agent besteht aus unabhängigen Skills, die einzeln oder kombiniert eingesetzt werden können.

### Geplante Skills

| Skill | Status | Beschreibung |
|---|---|---|
| `rpg-convert` | In Entwicklung | Konvertierung von Fixed Format / RPG/400 nach Fully Free RPG |
| `rpg-analyze` | Geplant | Analyse von Programmstruktur, Indicators, GOTOs und Komplexität |
| `rpg-refactor` | Geplant | Extraktion von Prozeduren, Eliminierung von Indicators |
| `sql-modernize` | Geplant | Embedded SQL analysieren, parametrisieren und optimieren |
| `db2-schema` | Geplant | DDS nach DDL konvertieren, Constraints und Indizes prüfen |
| `rpg-document` | Geplant | Automatische Dokumentation von Programmen und Prozeduren |
| `rpg-test` | Geplant | Generierung von Unit-Tests (z.B. für RPGUnit) |

---

## Skill: RPG-Konvertierung nach Fully Free

### Muss für jede RPG-Version ein eigener Konverter her?

**Kurze Antwort: Nein** — ein einheitlicher Konverter mit Versions-Erkennung ist der bessere Ansatz.

**Ausführliche Begründung:**

Die Konvertierung nach Fully Free ist im Kern eine **syntaktische Transformation**. Die Unterschiede zwischen RPG/400, RPG IV Fixed und Fully Free liegen hauptsächlich im Format (Spaltenpositionen, Spec-Buchstaben), nicht in völlig verschiedenen Paradigmen. Deshalb ist ein einheitlicher Konverter mit einer Erkennungs- und Normalisierungsschicht praktikabler als separate Konverter.

```
Eingabe
  │
  ▼
┌─────────────────────────┐
│  Version-Detector        │  Erkennt: RPG II, RPG III, RPG/400,
│                          │  RPG IV Fixed, Partial Free, Fully Free
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│  Spec-Parser             │  Liest H/F/D/I/C/O/P-Specs
│  (versionsspezifisch)    │  inkl. Spaltenoffsets je Version
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│  AST / IR                │  Einheitliche interne Darstellung
│  (versionsneutral)       │  des Programms (Prozeduren, Variablen,
│                          │  Anweisungen, Conditions, SQL-Blöcke)
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│  Fully Free Generator    │  Erzeugt sauberes, modernes RPG
│                          │  mit **FREE, DCL-S, DCL-DS, usw.
└──────────┬──────────────┘
           │
           ▼
        Ausgabe
```

### Was ist pro Version unterschiedlich?

| Komponente | RPG/400 | RPG IV Fixed | Partial Free |
|---|---|---|---|
| Spec-Spaltenoffsets | Abweichend | Standard | Standard |
| H-Spec Schlüsselwörter | Eingeschränkt | Voll | Voll |
| `/FREE`-Blöcke | Nein | Nein/Ja | Ja |
| CHAIN/READE-Syntax | Positional | Positional | Gemischt |
| Data Structures | Eingeschränkt | Voll | Voll |

Nur der **Parser** muss versions-aware sein — der Rest der Pipeline arbeitet auf einer neutralen Zwischendarstellung. RPG II und RPG III sind die Ausnahme: Hier sind strukturelle Unterschiede (kein ILE, andere Cycle-Logik) so signifikant, dass eine halbautomatische Konvertierung mit menschlicher Überprüfung realistischer ist.

### Konvertierungs-Prioritäten

| Quellformat | Automatisierungsgrad | Anmerkung |
|---|---|---|
| RPG IV Fixed → Fully Free | Hoch (>90%) | Primärer Use-Case, rein syntaktisch |
| RPG/400 Fixed → Fully Free | Mittel (70–80%) | Spec-Unterschiede, aber machbar |
| Partial Free → Fully Free | Sehr hoch (>95%) | Nur verbleibende Fixed-Blöcke |
| RPG III → Fully Free | Niedrig (40–60%) | Strukturelle Unterschiede, Review nötig |
| RPG II → Fully Free | Gering (<30%) | Nur Diagnose + Leitfaden, kein Auto-Convert |

---

## Architektur

```
rpg-modernizer/
├── agent/                  # Haupt-Agent (Orchestrierung, CLI, API)
│   ├── core/               # Agent-Loop, Tool-Routing, Kontext-Management
│   └── mcp/                # MCP-Server für IDE-Integration
│
├── skills/
│   ├── rpg-convert/        # Fixed → Fully Free Konverter
│   │   ├── detector/       # Versions-Erkennung
│   │   ├── parser/         # Spec-Parser (H/F/D/C/O/P)
│   │   ├── ir/             # Interne Repräsentation (AST)
│   │   └── generator/      # Fully Free Code-Generator
│   ├── rpg-analyze/        # Statische Analyse
│   ├── sql-modernize/      # DB2 SQL Modernisierung
│   └── db2-schema/         # DDS → DDL
│
├── tests/
│   ├── fixtures/           # Beispiel-RPG-Dateien aller Versionen
│   └── ...
│
└── docs/
    ├── rpg-versions.md     # Detailierte RPG-Versions-Referenz
    └── conversion-guide.md # Leitfaden für die Konvertierung
```

---

## Beispiel

### Eingabe: RPG IV Fixed Format

```rpgle
     H DFTACTGRP(*NO) ACTGRP(*CALLER)

     FCUSTOMER  IF   E           K DISK

     D CustomerDS      DS
     D  CustNo                        7P 0
     D  CustName                     30A
     D  CustCity                     20A

     C                   READ      CUSTOMER
     C                   DOW       NOT %EOF(CUSTOMER)
     C                   IF        CustCity = 'BERLIN'
     C                   EXSR      ProcessCustomer
     C                   ENDIF
     C                   READ      CUSTOMER
     C                   ENDDO

     C     ProcessCustomer BEGSR
     C                   DSPLY                   CustName
     C                   ENDSR
```

### Ausgabe: Fully Free RPG

```rpgle
**FREE
ctl-opt dftactgrp(*no) actgrp(*caller);

dcl-f CUSTOMER keyed usage(*input);

dcl-ds CustomerDS;
  CustNo    packed(7 : 0);
  CustName  char(30);
  CustCity  char(20);
end-ds;

read CUSTOMER;
dow not %eof(CUSTOMER);
  if CustCity = 'BERLIN';
    ProcessCustomer();
  endif;
  read CUSTOMER;
enddo;

dcl-proc ProcessCustomer;
  dsply CustName;
end-proc;
```

---

## Voraussetzungen

- Node.js 20+ oder Python 3.11+  *(wird noch entschieden)*
- Zugang zur Claude API (Anthropic)
- Optional: IBM i Zugang für direkte QSYS-Integration

---

## Status

Dieses Projekt befindet sich in aktiver früher Entwicklung. Die Architektur und die Skill-Schnittstellen sind noch nicht stabil.

---

## Mitarbeit

Beiträge sind willkommen — insbesondere:
- Beispiel-RPG-Programme verschiedener Versionen als Testfälle
- Erfahrungsberichte aus realen Modernisierungsprojekten
- Feedback zur Priorität der Skills

---

## Lizenz

MIT
