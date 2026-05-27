# Konvertierungsregeln: H-Spec → `ctl-opt`

> **Zweck dieser Datei:** LLM-Kontext für den RPG-IV → Fully Free Konverter.  
> Vollständige Mapping-Tabelle und Sonderfälle für die H-Spec-Konvertierung.

---

## 1. Einleitung

Die **H-Spec (Header Specification)** existiert im Fully Free Format nicht mehr.  
Alle H-Spec-Keywords werden durch das `ctl-opt`-Statement ersetzt.

### Wesentliche Unterschiede

| Merkmal | H-Spec (Fixed Format) | `ctl-opt` (Fully Free) |
|---|---|---|
| Spec-Kennzeichen | `H` in Spalte 6 | Keines — freier Text |
| Zeilenlänge | Max. 80 Zeichen | Max. 100 Zeichen |
| Mehrere Zeilen | Mehrere H-Spec-Zeilen erlaubt | Mehrere `ctl-opt`-Zeilen oder eine mit `;` |
| Abschluss | Keine Semikolons | Jedes `ctl-opt`-Statement mit `;` abschließen |
| Kommentar | `*` in Spalte 7 | `//` am Zeilenanfang oder Zeilenende |

### Grundprinzip der Konvertierung

```
H DFTACTGRP(*NO) ACTGRP(*NEW)
→
ctl-opt dftactgrp(*no) actgrp(*new);
```

Jede H-Spec-Zeile liefert Keywords, die in einem oder mehreren `ctl-opt`-Statements zusammengefasst werden.  
Die Keywords selbst sind in Fully Free identisch — nur Groß-/Kleinschreibung und der Zeilenrahmen ändern sich.

---

## 2. Vollständige Mapping-Tabelle

### 2.1 Direkte 1:1-Keywords (Syntax unverändert)

Diese Keywords übernehmen Namen und Werte identisch. Nur der `H`-Präfix entfällt und ein `;` wird am Ende des Statements ergänzt.

| H-Spec-Keyword | `ctl-opt`-Keyword | Werte | Beschreibung |
|---|---|---|---|
| `DFTACTGRP(*NO)` | `dftactgrp(*no)` | `*YES`, `*NO` | `*NO` = ILE-Programm; `*YES` = OPM-Kompatibilität |
| `ACTGRP(*NEW)` | `actgrp(*new)` | `*NEW`, `*CALLER`, `'name'` | Activation Group |
| `BNDDIR('name')` | `bnddir('name')` | Binding Directory Name | z. B. `'QC2LE'`, `'MYBNDDIR'` |
| `OPTION(opt1 : opt2)` | `option(opt1 : opt2)` | Compiler-Optionen | Mehrere Optionen mit `:` trennen |
| `EXPROPTS(*RESDECPOS)` | `expropts(*resdecpos)` | `*RESDECPOS`, `*MAXDIGITS` | Dezimalstellenbehandlung bei Ausdrücken |
| `DATEDIT(*DMY/)` | `datedit(*dmy/)` | `*DMY/`, `*MDY/`, `*YMD/`, `*JUL/` | Datumsbearbeitungscode für Edit-Wörter |
| `DATFMT(*ISO)` | `datfmt(*iso)` | `*ISO`, `*USA`, `*EUR`, `*JIS`, `*DMY`, `*MDY`, `*YMD` | Standard-Datumsformat |
| `TIMFMT(*HMS:)` | `timfmt(*hms:)` | `*HMS`, `*ISO`, `*USA`, `*EUR`, `*JIS` | Standard-Zeitformat |
| `DECEDIT('0,')` | `decedit('0,')` | `'0,'`, `'0.'`, `','`, `'.'` | Dezimalbearbeitungscode |
| `TRUNCNBR(*NOWARN)` | `truncnbr(*nowarn)` | `*YES`, `*NO`, `*NOWARN` | Behandlung nummerischer Abschneidung |
| `DEBUG(*YES)` | `debug(*yes)` | `*YES`, `*NO`, `*INPUT`, `*DUMP`, `*XMLSAX` | Debug-Informationen generieren |
| `COPYNEST(n)` | `copynest(n)` | Ganzzahl 1–4096 | Maximale /COPY-Verschachtelungstiefe |
| `FIXNBR(*ZONED)` | `fixnbr(*zoned)` | `*ZONED`, `*INPUTPACKED`, `*NONE` | Korrektur ungültiger Zahlen |
| `NOMAIN` | `nomain` | — | Modul ohne Hauptprozedur (Service Program) |
| `MAIN(procName)` | `main(procName)` | Prozedurname | Einstiegsprozedur (lineares ILE-Hauptprogramm) |
| `ALTSEQ(*SRC : seqtbl)` | `altseq(*src : seqtbl)` | `*NONE`, `*SRC : name` | Alternatives Sortiersequenz-Tabelle |
| `ALWNULL(*USRCTL)` | `alwnull(*usrctl)` | `*USRCTL`, `*NO`, `*INPUTONLY` | Behandlung null-fähiger Felder |
| `CCSID(*GRAPH : n)` | `ccsid(*graph : n)` | CCSID-Nummer | DBCS-CCSID für Graphic-Daten |
| `CURSYM('€')` | `cursym('€')` | Währungszeichen | Währungssymbol für Edit-Codes |
| `CVTOPT(*DATETIME)` | `cvtopt(*datetime)` | `*DATETIME`, `*NODATETIME`, `*VARCHAR`, `*NOVARCHAR` | Konvertierung bei externer Datenbeschreibung |
| `ENBPFRCOL(*PEP)` | `enbpfrcol(*pep)` | `*NONE`, `*PEP`, `*ENTRYEXIT`, `*FULL` | Performance Collection aktivieren |
| `EXTBININT(*YES)` | `extbinint(*yes)` | `*YES`, `*NO` | Externe Binär-Felder als Integer interpretieren |
| `FLTDIV(*NO)` | `fltdiv(*no)` | `*YES`, `*NO` | Fließkomma-Division erzwingen |
| `FORMSALIGN(*YES)` | `formsalign(*yes)` | `*YES`, `*NO` | Formularausrichtung für Drucker |
| `FTRANS(filename)` | `ftrans(filename)` | Dateiname | Datei-Übersetzungstabelle |
| `GENLVL(n)` | `genlvl(n)` | 0–20 | Schweregrad für Codegenerierung abbrechen |
| `INDENT('|')` | `indent('|')` | Zeichen oder `*NONE` | Einrückungszeichen in der Quellauflistung |
| `INTPREC(n)` | `intprec(n)` | `10`, `20` | Intermediate Integer-Präzision (Stellen) |
| `LANGID(*JOBRUN)` | `langid(*jobrun)` | `*JOBRUN`, `*JOB`, Sprach-ID | Sprach-ID für ALTSEQ und SRTSEQ |
| `NOXREF` | — | — | **Kein Äquivalent** — siehe Abschnitt 3 |
| `OPENOPT(*INZOFL)` | `openopt(*inzofl)` | `*INZOFL`, `*NOINZOFL` | Overflow-Indikator-Verhalten beim Open |
| `OPTIMIZE(*FULL)` | `optimize(*full)` | `*NONE`, `*BASIC`, `*FULL` | Optimierungsstufe |
| `PGMINFO(*PCML : *MODULE)` | `pgminfo(*pcml : *module)` | PCML-Optionen | Programm-Informationen generieren |
| `PRFDTA(*COL)` | `prfdta(*col)` | `*COL`, `*NOCOL` | Profiling-Daten sammeln |
| `SRTSEQ(*LANGIDUNQ)` | `srtseq(*langidunq)` | `*HEX`, `*JOB`, `*JOBRUN`, `*LANGIDUNQ`, `*LANGIDSHR`, `'seqtbl'` | Sortiersequenz |
| `STGMDL(*INHERIT)` | `stgmdl(*inherit)` | `*SNGLVL`, `*TERASPACE`, `*INHERIT` | Speichermodell |
| `TEXT('text')` | `text('text')` | Zeichenkette max. 50 Zeichen | Beschreibungstext des Moduls |
| `THREAD(*CONCURRENT)` | `thread(*concurrent)` | `*CONCURRENT`, `*SERIALIZE` | Thread-Sicherheits-Unterstützung |
| `TIMEZONE(*UTCQ)` | `timezone(*utcq)` | `*UTCQ`, `*JOB`, `'UTC-offset'` | Zeitzone für Timestamp-Operationen |
| `TRUNCNBR(*NO)` | `truncnbr(*no)` | `*YES`, `*NO`, `*NOWARN` | Abschneiden von Zahlen verhindern |
| `USRPRF(*OWNER)` | `usrprf(*owner)` | `*USER`, `*OWNER` | Benutzerprofil für Objekt-Autorität |

### 2.2 Häufige `OPTION`-Werte (gleich in H-Spec und `ctl-opt`)

| Option-Wert | Bedeutung |
|---|---|
| `*NODEBUGIO` | Keine Debug-I/O-Zeilen generieren |
| `*DEBUGIO` | Debug-I/O-Zeilen generieren |
| `*SRCSTMT` | Quellzeilennummern im Debug verwenden |
| `*NOSRCSTMT` | Interne Statement-Nummern im Debug |
| `*NOSHOWCPY` | /COPY-Member nicht in Quellauflistung zeigen |
| `*SHOWCPY` | /COPY-Member in Quellauflistung anzeigen |
| `*EXPDDS` | Externe Datenbeschreibungen expandieren |
| `*NOEXPDDS` | Externe Datenbeschreibungen nicht expandieren |
| `*NOUNREF` | Nicht-referenzierte Symbole nicht aufführen |
| `*UNREF` | Nicht-referenzierte Symbole aufführen |

---

## 3. Keywords ohne direkte Entsprechung

Diese H-Spec-Keywords haben kein 1:1-Äquivalent in Fully Free und werden weggelassen oder kommentiert:

| H-Spec-Keyword | Verhalten in Fully Free | Empfehlung |
|---|---|---|
| `NOXREF` | `ctl-opt` kennt kein `NOXREF` — Cross-Reference-Ausgabe wird über Compile-Befehl (`CRTBNDRPG`) gesteuert, nicht über die Quelle | Weglassen — Compile-Befehlsoption `XREF(*NO)` verwenden |
| `REALLOC(*NO)` | Nicht in `ctl-opt` vorhanden | Weglassen — kein Einfluss auf Fully Free Verhalten |
| `CHARCOUNT` | Nicht als `ctl-opt`-Keyword verfügbar | Weglassen |
| `AUT(*LIBCRTAUT)` | Wird über den Compile-Befehl gesetzt, nicht über Quelle | Weglassen — im Compile-Befehl `CRTBNDRPG AUT(...)` angeben |

> **Regel:** Unbekannte H-Spec-Keywords, die nicht in der Mapping-Tabelle stehen, werden mit einem TODO-Kommentar markiert:
> ```rpgle
> // TODO: H-Spec-Keyword 'UNBEKANNT(wert)' konnte nicht automatisch konvertiert werden — manuell prüfen
> ```

---

## 4. Syntaktische Konvertierungsregeln

### Regel 1: Groß-/Kleinschreibung

H-Spec-Keywords sind großgeschrieben. In Fully Free ist beides gültig, Konvention ist Kleinschreibung.

```rpgle
// Fixed Format (H-Spec)
H DFTACTGRP(*NO) ACTGRP(*NEW)

// Fully Free (ctl-opt) — Kleinschreibung bevorzugt
ctl-opt dftactgrp(*no) actgrp(*new);
```

### Regel 2: Semikolon-Abschluss

Jedes `ctl-opt`-Statement **muss** mit `;` abgeschlossen werden. H-Spec hatte kein Semikolon.

### Regel 3: Kommentare

H-Spec-Kommentare (`*` in Spalte 7) werden zu `//`-Kommentaren in Fully Free:

```rpgle
// Fixed Format
     * Compiler-Optionen
     H DFTACTGRP(*NO)

// Fully Free
// Compiler-Optionen
ctl-opt dftactgrp(*no);
```

### Regel 4: Zusammenführung mehrerer Keywords

Mehrere Keywords können auf einer `ctl-opt`-Zeile oder auf mehreren `ctl-opt`-Zeilen stehen. Beides ist korrekt.

---

## 5. Sonderfall: Mehrere H-Spec-Zeilen → `ctl-opt`

### Variante A: Alle Keywords in einem `ctl-opt`-Statement

```rpgle
// Fixed Format — 3 H-Spec-Zeilen
     H DFTACTGRP(*NO) ACTGRP(*NEW)
     H OPTION(*NODEBUGIO : *SRCSTMT)
     H BNDDIR('QC2LE') DATEDIT(*DMY/)

// Fully Free — alle Keywords in einem Statement
ctl-opt dftactgrp(*no) actgrp(*new)
        option(*nodebugio : *srcstmt)
        bnddir('QC2LE') datedit(*dmy/);
```

### Variante B: Mehrere `ctl-opt`-Statements (thematisch gruppiert)

```rpgle
// Fixed Format — 3 H-Spec-Zeilen
     H DFTACTGRP(*NO) ACTGRP(*NEW)
     H OPTION(*NODEBUGIO : *SRCSTMT)
     H BNDDIR('QC2LE') DATEDIT(*DMY/)

// Fully Free — mehrere ctl-opt (thematische Gruppierung)
ctl-opt dftactgrp(*no) actgrp(*new);
ctl-opt option(*nodebugio : *srcstmt);
ctl-opt bnddir('QC2LE') datedit(*dmy/);
```

**Empfehlung:** Variante A (ein Statement) bevorzugen, wenn die Zeile 100 Zeichen nicht überschreitet.  
Variante B bei langen Keyword-Listen oder wenn thematische Trennung die Lesbarkeit verbessert.

---

## 6. Sonderfall: H-Spec fehlt komplett

Wenn die Fixed-Format-Quelldatei **keine H-Spec** enthält, gilt:

- Das Programm läuft im OPM-Kompatibilitätsmodus (`DFTACTGRP(*YES)`)
- Keine `ctl-opt`-Zeile ist technisch korrekt — aber problematisch für ILE-Features

### Empfehlung: Minimale `ctl-opt`-Vorlage einfügen

```rpgle
// Automatisch generiert — H-Spec war im Original nicht vorhanden
// TODO: Activation Group und Binding Directory prüfen und anpassen
ctl-opt dftactgrp(*no) actgrp(*caller);
```

### Erklärung der Standardwerte

| Keyword | Gewählter Standardwert | Begründung |
|---|---|---|
| `dftactgrp(*no)` | `*no` | Ermöglicht ILE-Features (Prozeduren, Service Programs) |
| `actgrp(*caller)` | `*caller` | Sicherstes Default — erbt die Activation Group des Aufrufers |

> **Wichtig:** `actgrp(*caller)` ist konservativer als `actgrp(*new)`. Das Original ohne H-Spec lief im OPM-Modus. Falls das Programm später als eigenständiges Programm verwendet wird, sollte `actgrp(*new)` evaluiert werden.

---

## 7. Beispiele

### Beispiel 1: Einfache H-Spec → `ctl-opt`

**Input (Fixed Format):**
```rpgle
     H DFTACTGRP(*NO) ACTGRP(*NEW) BNDDIR('QC2LE')
```

**Output (Fully Free):**
```rpgle
**FREE
ctl-opt dftactgrp(*no) actgrp(*new) bnddir('QC2LE');
```

---

### Beispiel 2: Vollständige H-Spec-Sektion → `ctl-opt`

**Input (Fixed Format):**
```rpgle
     H DFTACTGRP(*NO)
     H ACTGRP(*NEW)
     H OPTION(*NODEBUGIO : *SRCSTMT : *NOSHOWCPY)
     H BNDDIR('QC2LE' : 'MYBNDDIR')
     H DATEDIT(*DMY/) DATFMT(*ISO)
     H TIMFMT(*HMS:)
     H DEBUG(*YES)
```

**Output (Fully Free):**
```rpgle
**FREE
ctl-opt dftactgrp(*no) actgrp(*new)
        option(*nodebugio : *srcstmt : *noshowcpy)
        bnddir('QC2LE' : 'MYBNDDIR')
        datedit(*dmy/) datfmt(*iso)
        timfmt(*hms:)
        debug(*yes);
```

---

### Beispiel 3: H-Spec mit Keywords ohne Entsprechung

**Input (Fixed Format):**
```rpgle
     H DFTACTGRP(*NO) ACTGRP(*NEW)
     H NOXREF
     H AUT(*LIBCRTAUT)
```

**Output (Fully Free):**
```rpgle
**FREE
ctl-opt dftactgrp(*no) actgrp(*new);
// TODO: H-Spec-Keyword 'NOXREF' hat kein ctl-opt-Äquivalent — beim Compile-Befehl CRTBNDRPG XREF(*NO) setzen
// TODO: H-Spec-Keyword 'AUT(*LIBCRTAUT)' hat kein ctl-opt-Äquivalent — beim Compile-Befehl CRTBNDRPG AUT(*LIBCRTAUT) setzen
```

---

### Beispiel 4: Keine H-Spec vorhanden

**Input (Fixed Format):** *(keine H-Spec-Zeilen)*
```rpgle
     F KUNDENDTA   IF   E           K DISK
     C                   ...
```

**Output (Fully Free):**
```rpgle
**FREE
// Automatisch generiert — H-Spec war im Original nicht vorhanden
// TODO: Activation Group und Binding Directory prüfen und anpassen
ctl-opt dftactgrp(*no) actgrp(*caller);

dcl-f KUNDENDTA keyed;
...
```

---

## 8. Zusammenfassung der Konvertierungsschritte

```
1. Alle H-Spec-Zeilen sammeln und ihre Keywords extrahieren
2. Keywords gegen Mapping-Tabelle (Abschnitt 2) prüfen
3. Keywords mit Entsprechung → in ctl-opt-Statement übertragen (Kleinschreibung)
4. Keywords ohne Entsprechung → TODO-Kommentar einfügen (Abschnitt 3)
5. `**FREE` als erste Zeile setzen
6. Ein oder mehrere ctl-opt-Statements direkt nach **FREE platzieren
7. Jedes ctl-opt-Statement mit `;` abschließen
8. Fehlende H-Spec → Standard-ctl-opt-Vorlage einfügen (Abschnitt 6)
```
