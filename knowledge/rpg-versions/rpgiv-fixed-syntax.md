# RPG IV Fixed Format — Vollständige Syntax-Referenz

> **Zweck dieser Datei:** LLM-Kontext für den RPG-IV → Fully Free Konverter.  
> Quellformat aller Konvertierungen. Dient als alleinige Quelle für korrekte Fixed Format Syntax.

---

## 1. Einleitung: Spaltenbasiertes Format

**RPG IV** (ILE RPG) ist die vierte Generation der RPG-Sprache auf IBM i (AS/400).  
Das Fixed Format ist das **klassische, spaltengebundene** Programmiermodell, bei dem jede Zeile streng nach Spaltenposition interpretiert wird.

### Historischer Kontext

| Generation | Name | Plattform | Besonderheit |
|---|---|---|---|
| RPG II | RPG/400 | System/34, System/36 | Urformat, stark codeworientiert |
| RPG III | RPG/400 | AS/400 | Verbesserungen, Structured Operations |
| **RPG IV** | ILE RPG | AS/400 / IBM i | Modernes ILE-Framework, frei formatierbarer Code möglich |
| RPG IV Free | ILE RPG | IBM i 7.1+ | `/FREE`-Blöcke in C-Spec |
| Fully Free | ILE RPG | IBM i 7.1 TR7 / 7.2+ | `**FREE` — gesamte Datei frei |

Fixed Format RPG IV ist das **häufigste Quell-Format** in Legacy-Codebasen.  
Erkennungsmerkmale: Spec-Buchstabe in Spalte 6, keine `**FREE`-Direktive am Dateianfang.

### Spec-Typen im Überblick

| Spalte 6 | Spec-Typ | Zweck |
|---|---|---|
| `H` | Header-Spec | Compiler- und Laufzeit-Optionen (= H-Spec) |
| `F` | File-Spec | Dateidefinitionen |
| `D` | Definition-Spec | Variablen, Data Structures, Prototypen |
| `C` | Calculation-Spec | Programm-Logik / Anweisungen |
| `O` | Output-Spec | Ausgabeformate (historisch, selten in neuem Code) |
| `P` | Procedure-Spec | Prozedur-Beginn und -Ende |
| ` ` (leer) | — | Continuation-Zeile (z. B. D-Spec Subfeld) |

---

## 2. Allgemeine Spaltenstruktur

Jede Zeile in einer Fixed Format RPG IV Quelldatei ist **80 Zeichen breit** (Standard).  
Spalten werden **1-basiert** gezählt.

```
Spalte:  1234567890123456789012345678901234567890...80
         |----||+------------------------- Nutzdaten (je nach Spec) ------+|
```

### Spalten 1–5: Zeilennummer (optional)

- Numerische Sequenznummer (0001–9999)
- In modernem Code meist leer gelassen
- Vom Compiler ignoriert — rein dokumentarisch

### Spalte 6: Spec-Typ

- Pflichtfeld: identifiziert den Spec-Typ (H / F / D / C / O / P)
- `*` in Spalte 7 → Kommentarzeile (unabhängig vom Spec-Typ)

### Spalte 7: Kommentar-/Continuation-Indikator

| Zeichen | Bedeutung |
|---|---|
| `*` | Kommentarzeile — gesamte Zeile wird ignoriert |
| `/` | Direktiven-Zeile (`/FREE`, `/END-FREE`, `/COPY`, `/INCLUDE`, `/DEFINE`, etc.) |
| `+` | Continuation — setzt vorherige Zeile fort (nur in bestimmten Specs) |
| leer | Normale Zeile |

### Kommentare

```rpgle
     * Das ist ein Kommentar in Spalte 7 (kein Spec-Typ nötig)
     C                   EVAL      x = 1            * Inline-Kommentar ab Spalte 81 (selten)
```

---

## 3. H-Spec (Header Specification)

### Zweck

Definiert programmweite Compiler- und Laufzeit-Einstellungen.  
Entspricht im Fully Free Format dem `ctl-opt`-Statement.

### Spaltenstruktur

```
Spalte:  1  6  7                                                          80
         ----H+Keywords++++++++++++++++++++++++++++++++++++++++++++++++++++
```

- Spalte 6: `H`
- Spalten 7–80: Keywords (in RPG IV bereits free-form innerhalb der Spalten 7–80)

### Syntax

Keywords werden in Spalte 7–80 als Schlüsselwort-Ausdruck notiert:

```rpgle
     H DFTACTGRP(*NO) ACTGRP(*NEW)
     H OPTION(*NODEBUGIO : *SRCSTMT)
     H BNDDIR('QC2LE') DATEDIT(*DMY/)
```

Mehrere H-Spec-Zeilen sind erlaubt und werden zusammengefasst.

### Gängige H-Spec-Keywords

| Keyword | Werte | Beschreibung |
|---|---|---|
| `DFTACTGRP(*NO)` | `*YES`, `*NO` | `*NO` = ILE-Programm; `*YES` = OPM-kompatibel |
| `ACTGRP(*NEW)` | `*NEW`, `*CALLER`, `'name'` | Activation Group |
| `BNDDIR('name')` | Binding Directory Name | z. B. `'QC2LE'`, `'MYBNDDIR'` |
| `OPTION(*NODEBUGIO)` | Compiler-Optionen | Mehrere mit `:` trennen |
| `EXPROPTS(*RESDECPOS)` | `*RESDECPOS`, `*MAXDIGITS` | Dezimalstellenbehandlung bei Ausdrücken |
| `DATEDIT(*DMY/)` | `*DMY/`, `*MDY/`, `*YMD/`, `*JUL/` | Datumsbearbeitungscode |
| `DATFMT(*ISO)` | `*ISO`, `*USA`, `*EUR`, `*JIS`, `*DMY`, `*MDY` | Standard-Datumsformat |
| `TIMFMT(*HMS)` | `*HMS`, `*ISO`, `*USA`, `*EUR`, `*JIS` | Standard-Zeitformat |
| `DECEDIT('0,')` | `'0,'`, `'0.'`, `','`, `'.'` | Dezimalbearbeitungscode |
| `TRUNCNBR(*NOWARN)` | `*YES`, `*NO`, `*NOWARN` | Behandlung bei Nummerischer Abschneidung |
| `DEBUG(*YES)` | `*YES`, `*NO` | Debug-Informationen generieren |
| `COPYNEST(n)` | Zahl | Maximale /COPY-Verschachtelungstiefe |
| `FIXNBR(*ZONED)` | `*ZONED`, `*INPUTPACKED` | Verhalten bei ungültigen Zahlen |
| `NOMAIN` | — | Modul ohne Hauptprozedur (Service Program) |

**Häufige `OPTION`-Werte:**

| Option | Bedeutung |
|---|---|
| `*NODEBUGIO` | Keine Debug-I/O-Zeilen generieren |
| `*SRCSTMT` | Quellzeilennummern im Debug verwenden |
| `*NOSHOWCPY` | /COPY-Member nicht in Quellauflistung anzeigen |
| `*EXPDDS` | Externe Datenbeschreibungen expandieren |

---

## 4. F-Spec (File Specification)

### Zweck

Definiert alle Dateien, die das Programm verwendet: Disk-Dateien, Drucker, Workstation-Dateien.  
Entspricht im Fully Free Format dem `dcl-f`-Statement.

### Spaltenstruktur

```
Spalte:  1  6  7          17 18 19 20 21 22    28    34 36   43              80
         ----F+Dateiname++ITEASequNFORMKLenDFDeviceKeywords+++++++++++++++++++
```

| Spalten | Inhalt | Beschreibung |
|---|---|---|
| 7–16 | Dateiname | Max. 10 Zeichen, linksbündig |
| 17 | Dateityp | I, O, U, C |
| 18 | File Designation | blank, P, S, R, T |
| 19 | End-of-File | blank oder E |
| 20 | File Addition | blank oder A |
| 21 | Sequence | blank, A–Z (für sequentielle Dateien) |
| 22 | File Format | blank = Program-Described, E = Extern |
| 23–27 | Reserviert | leer |
| 28–32 | Record Length | Satzlänge (nur bei program-described) |
| 33 | Limits Processing | blank, K oder A |
| 34–35 | Schlüssellänge | Länge des Schlüssels (bei K) |
| 36 | Record Address Type | blank, A, P, K, D, G, T, Z |
| 37–42 | Device | DISK, PRINTER, WORKSTN, COMB, SEQ, SPECIAL |
| 43–80 | Keywords | Datei-Keywords (free-form in diesen Spalten) |

### Dateitypen (Spalte 17)

| Zeichen | Typ | Beschreibung |
|---|---|---|
| `I` | Input | Nur lesen |
| `O` | Output | Nur schreiben |
| `U` | Update | Lesen und schreiben (Satz-Update) |
| `C` | Combined | Lesen und schreiben (z. B. Workstation) |

### File Designation (Spalte 18)

| Zeichen | Designation | Beschreibung |
|---|---|---|
| blank | Full Procedural | Explizite I/O-Operationen im Programm |
| `P` | Primary | Steuert den Verarbeitungszyklus (RPG-Zyklus, veraltet) |
| `S` | Secondary | Sekundäre Zyklus-Datei |
| `R` | Record Address | Adressdatei für ADDROUT |
| `T` | Array/Table | Laden bei Programmstart |

In modernem ILE RPG: fast immer **blank** (Full Procedural).

### Record Address Type (Spalte 36)

| Zeichen | Typ |
|---|---|
| blank | Keine / sequentiell |
| `K` | Schlüssel (Keyed) |
| `A` | Zeichenkette (Char-Schlüssel) |
| `P` | Gepackt-numerisch (Packed-decimal-Schlüssel) |
| `D` | Datum |
| `G` | Grafik / DBCS |
| `T` | Zeit |
| `Z` | Timestamp |

### Devices (Spalten 37–42)

| Device | Typ |
|---|---|
| `DISK` | Physische oder logische Datenbankdatei |
| `PRINTER` | Druckdatei (PRTF) |
| `WORKSTN` | Display File (DSPF) / interaktive Ein-/Ausgabe |
| `SEQ` | Sequentielle Datei |
| `SPECIAL` | Benutzergeschriebener Gerätehandler |
| `COMB` | Kombiniert (Sequential/Record Address) |

### Gängige F-Spec-Keywords (Spalten 43–80)

| Keyword | Beschreibung |
|---|---|
| `KEYED` | Schlüsselzugriff (entspricht K in Spalte 36) |
| `USROPN` | Datei muss mit OPEN explizit geöffnet werden |
| `EXTDESC('name')` | Externer Dateideskriptor (anderer Dateiname im System) |
| `EXTFILE(var)` | Dateiname aus Variable zur Laufzeit |
| `RENAME(extName : intName)` | Externen Satznamen umbenennen |
| `QUALIFIED` | Satznamen sind vollständig qualifiziert |
| `PREFIX(prfx)` | Feldnamen-Präfix für externe Felder |
| `OFLIND(*IN01)` | Overflow-Indikator für Druckdateien |
| `COMMIT` | Commitsteuerung aktivieren |
| `INFDS(dsName)` | File Information Data Structure |
| `INFSR(srName)` | I/O-Fehler-Subroutine |
| `FORMLEN(n)` | Formularlänge (Printer) |
| `FORMOFL(n)` | Overflow-Zeile (Printer) |

### Beispiele

```rpgle
     * Einfache Eingabedatei (sequentiell, extern beschrieben)
     FCUSTMAST  IF   E           K DISK

     * Update-Datei (keyed)
     FORDTAB    UF   E           K DISK    COMMIT

     * Druckdatei mit Overflow-Indikator
     FREPORT    O    F  132        PRINTER  OFLIND(*IN01)

     * Datei mit laufzeitbestimmtem Namen
     FMYDYNFILE IF   E           K DISK    EXTFILE(wFileName)

     * Workstation-Datei (interaktiv)
     FSCRN01    CF   E             WORKSTN
```

---

## 5. D-Spec (Definition Specification)

### Zweck

Definiert Variablen (Standalone), Data Structures, Named Constants, Prototypen und Procedure Interfaces.  
Entspricht im Fully Free Format `dcl-s`, `dcl-ds`, `dcl-c`, `dcl-pr`, `dcl-pi`.

### Spaltenstruktur

```
Spalte:  1  6  7                  22 24 26      33  40 44              80
         ----D+Name++++++++++++++EUDsFrom+++To/L+++IDc.Keywords++++++++++
```

| Spalten | Inhalt | Beschreibung |
|---|---|---|
| 7–21 | Name | Variablenname, max. 15 Zeichen (linksbündig) |
| 22 | External Type | `E` = extern beschrieben, sonst blank |
| 23 | Reserved | leer |
| 24–25 | Spec-Untertyp | DS, S, C, PR, PI (oder leer für Subfeld) |
| 26–32 | From-Position | Startposition (für positionale DS) |
| 33–39 | To-Position / Length | Endposition oder Länge |
| 40 | Interner Datentyp | Typkürzel (siehe Tabelle) |
| 41–42 | Dezimalstellen | Anzahl Dezimalstellen |
| 43 | Reserved | leer |
| 44–80 | Keywords | Definition-Keywords (free-form) |

### D-Spec Untertypen (Spalten 24–25)

| Kürzel | Bedeutung | Fully Free Äquivalent |
|---|---|---|
| `S` | Standalone Variable | `dcl-s` |
| `DS` | Data Structure | `dcl-ds` |
| `C` | Named Constant | `dcl-c` |
| `PR` | Prototype | `dcl-pr` |
| `PI` | Procedure Interface | `dcl-pi` |
| blank | Subfeld (innerhalb einer DS) | Unterfeld in `dcl-ds` |

### Interne Datentypen (Spalte 40)

| Kürzel | Datentyp | Fully Free Typ | Beispiel-Deklaration |
|---|---|---|---|
| `A` | Character | `CHAR(n)` | `DCL-S name CHAR(20)` |
| `B` | Binary Integer | `INT(5)` oder `INT(10)` | `DCL-S zahl INT(10)` |
| `C` | UCS-2 Character | `UCS2(n)` | `DCL-S utext UCS2(10)` |
| `D` | Date | `DATE` | `DCL-S datum DATE` |
| `F` | Float (Binär-Gleitkomma) | `FLOAT(4)` / `FLOAT(8)` | `DCL-S flt FLOAT(8)` |
| `G` | Graphic (DBCS) | `GRAPH(n)` | `DCL-S gtext GRAPH(10)` |
| `I` | Integer (vorzeichenbehaftet) | `INT(3/5/10/20)` | `DCL-S cnt INT(10)` |
| `N` | Indicator | `IND` | `DCL-S flag IND` |
| `O` | Object | `OBJECT(*JAVA ...)` | (selten) |
| `P` | Packed Decimal | `PACKED(p:d)` | `DCL-S betrag PACKED(9:2)` |
| `S` | Zoned Decimal | `ZONED(p:d)` | `DCL-S menge ZONED(7:2)` |
| `T` | Time | `TIME` | `DCL-S uhrzeit TIME` |
| `U` | Unsigned Integer | `UNS(3/5/10/20)` | `DCL-S pos UNS(10)` |
| `W` | Timestamp | `TIMESTAMP` | `DCL-S ts TIMESTAMP` |
| `Z` | Timestamp (alternativ) | `TIMESTAMP` | Selten verwendet |
| blank | Char (Default) oder aus Keywords | abgeleitet | – |

> **Hinweis:** Längen für Integer/Unsigned:
> - 3 = `INT(5)` / `UNS(5)` (−32768 bis 32767 / 0 bis 65535)
> - 5 = `INT(5)` / `UNS(5)` — In RPG IV Fixed Format: Länge 5 → 16-bit
> - 9 = `INT(10)` / `UNS(10)` — 32-bit
> - 19 = `INT(20)` / `UNS(20)` — 64-bit
>
> Genauer: Spalte 40 + Längenfeld gemeinsam bestimmen den Typ.

### Gängige D-Spec-Keywords

| Keyword | Beschreibung |
|---|---|
| `INZ` / `INZ(wert)` | Initialisierungswert (`*ZEROS`, `*BLANKS`, `*NULL`, Literalwert) |
| `CONST(wert)` | Für `C`-Typ: Konstantenwert |
| `CONST` | Parameter ist konstant (kein Rückschreiben) |
| `VALUE` | Parameter wird by Value übergeben |
| `VARYING` | Variable-Length String |
| `DIM(n)` | Array-Dimension |
| `ASCEND` / `DESCEND` | Sortierreihenfolge für Arrays |
| `OVERLAY(name:pos)` | Überlagert ein anderes Feld in der DS |
| `POS(n)` | Startposition in der DS (absolut) |
| `BASED(pointer)` | Variable basiert auf Pointer |
| `OCCURS(n)` | Multiple-Occurrence Data Structure (alt, → `DIM`) |
| `LIKE(feldname)` | Gleicher Typ und Länge wie anderes Feld |
| `LIKEDS(dsname)` | Gleiche Struktur wie andere DS |
| `LIKEREC(recname:*INPUT/*OUTPUT/*ALL)` | Wie ein Dateifeld-Satz |
| `EXTNAME(datei:format)` | Felder aus externer Datei übernehmen |
| `QUALIFIED` | Subfelder nur per `ds.feld` zugreifbar |
| `TEMPLATE` | DS als Vorlage (keine Speicherzuweisung) |
| `NOOPT` | Kein Optimieren dieser Variable durch Compiler |
| `PACKEVEN` | Packed Decimal mit gerader Stellenzahl |
| `DATFMT(*ISO)` | Datumsformat für Date-Felder |
| `TIMFMT(*HMS)` | Zeitformat für Time-Felder |
| `ALTSEQ` | Alternative Sortiersequenz |
| `EXTPROC('name')` | Externe Prozedur für Prototype |
| `EXTPGM('name')` | Externes Programm für Prototype |
| `OPDESC` | Operational Descriptors übergeben |
| `RTNPARM` | Rückgabewert als Parameter |

### Standalone Variable (S)

```rpgle
     D kundenNr       S              7P 0           INZ(0)
     D name           S             30A             INZ(*BLANKS)
     D betrag         S              9P 2
     D datum          S               D             DATFMT(*ISO)
     D flag           S               N             INZ(*OFF)
     D zähler         S             10I 0           INZ(1)
     D tabelle        S             10A             DIM(100)
```

### Data Structure (DS)

```rpgle
     D adresse        DS
     D  strasse                      30A
     D  plz                           5P 0
     D  ort                          20A

     * Mit EXTNAME (extern beschrieben)
     D kundeDS        E DS                          EXTNAME(KUNDTAB)

     * Mit LIKEDS
     D kundeDS2       DS                          LIKEDS(adresse)

     * Qualifiziert (Subfelder: adresse.strasse)
     D adresse2       DS                          QUALIFIED
     D  strasse                      30A
     D  plz                           5P 0
```

### Named Constant (C)

```rpgle
     D MAXZEILEN      C                   CONST(9999)
     D TRENNZEICHEN   C                   CONST(' : ')
     D MWST           C                   CONST(0.19)
```

### Prototype (PR) und Procedure Interface (PI)

```rpgle
     * Prototype für externe Prozedur
     D berechne       PR            10P 2
     D  wert1                        9P 2           VALUE
     D  wert2                        9P 2           VALUE

     * Procedure Interface (innerhalb der Prozedur)
     D berechne       PI            10P 2
     D  wert1                        9P 2           VALUE
     D  wert2                        9P 2           VALUE
```

---

## 6. C-Spec (Calculation Specification)

### Zweck

Enthält die Programm-Logik: Berechnungen, Dateioperationen, Schleifen, Bedingungen, Prozeduraufrufe.  
Entspricht im Fully Free Format freien Anweisungen (kein `dcl-`-Statement, sondern direkte Ausdrücke).

### Spaltenstruktur

```
Spalte:  1  6  78 9   12              26          36              50        64    69 71  73  75
         ----CLN01Factor1++++++++++++Opcode(E)++++Factor2+++++++++Result+++++++++Len++DcHiLoEq
```

| Spalten | Inhalt | Beschreibung |
|---|---|---|
| 6 | Spec-Typ | `C` |
| 7–8 | Control Level | blank, L0–L9, LR, SR |
| 9–11 | Conditioning Indicators | Zwei- bis dreistellige Indikatoren |
| 12–25 | Faktor 1 | Erster Operand (14 Zeichen) |
| 26–35 | Opcode | Operationscode (10 Zeichen, inkl. `(E)` für Fehlerbehandlung) |
| 36–49 | Faktor 2 | Zweiter Operand (14 Zeichen) |
| 50–63 | Ergebnis-Feld | Resultat-Variable (14 Zeichen) |
| 64–68 | Feldlänge | Länge des Ergebnisfeldes |
| 69–70 | Dezimalstellen | Dezimalstellen des Ergebnisfeldes |
| 71–72 | Hi/Plus-Indikator | Resulting Indicator 1 (positiv / größer) |
| 73–74 | Lo/Minus-Indikator | Resulting Indicator 2 (negativ / kleiner) |
| 75–76 | Eq/Zero-Indikator | Resulting Indicator 3 (gleich / null) |

### Control Level (Spalten 7–8)

| Wert | Bedeutung |
|---|---|
| blank | Keine Steuerung |
| `SR` | Ausführbar innerhalb einer Subroutine |
| `L0` | Detaileebene (immer) |
| `L1`–`L9` | Control Level Break (Zyklus, veraltet) |
| `LR` | Last Record — Endverarbeitung |

### Conditioning Indicators (Spalten 9–11)

Conditioning Indicators steuern, ob eine C-Spec-Zeile ausgeführt wird:

```
09: Indikator-Nummer (1–2 Ziffern oder 2-Zeichen-Code)
10: blank oder N (negieren)
11: zweite Stelle der Indikator-Nummer
```

Beispiel: `N01` → Zeile wird ausgeführt wenn Indikator 01 **nicht** gesetzt ist.

```rpgle
     C   01            MOVE      'Gefunden'    meldung
     C   N01           MOVE      'Nicht found' meldung
```

### Erweiterter Faktor 2 (Extended Factor 2)

Wenn Opcode `EVAL`, `CALLP`, `IF`, `DOW`, `DOU`, etc. verwendet wird, entfallen Faktor 1 und Ergebnis-Feld.  
Der gesamte Ausdruck steht in Faktor 2 (Spalten 36–80 verfügbar):

```rpgle
     C                   EVAL      ergebnis = wert1 + wert2
     C                   IF        betrag > 0 AND menge < 100
```

---

## 7. Häufige C-Spec Opcodes

### Zuweisung & Arithmetik

| Opcode | Faktor 1 | Faktor 2 | Ergebnis | Beschreibung |
|---|---|---|---|---|
| `EVAL` | — | Ausdruck | — | Ausdruck auswerten und zuweisen |
| `MOVE` | — | Quelle | Ziel | Rechtsbündige Zuweisung (Datentyp-Konvertierung) |
| `MOVEL` | — | Quelle | Ziel | Linksbündige Zuweisung |
| `Z-ADD` | — | Wert | Ziel | Ziel = 0 + Wert (Wert zuweisen) |
| `Z-SUB` | — | Wert | Ziel | Ziel = 0 − Wert (negieren) |
| `ADD` | Summand1 | Summand2 | Summe | Summe = F1 + F2 (oder F2 += F1 wenn kein Ergebnis) |
| `SUB` | Minuend | Subtrahend | Differenz | Differenz = F1 − F2 |
| `MULT` | Faktor1 | Faktor2 | Produkt | Produkt = F1 × F2 |
| `DIV` | Dividend | Divisor | Quotient | Quotient = F1 / F2 |
| `MVR` | — | — | Rest | Rest nach letzter DIV-Operation |

### Vergleich & Bedingungen

| Opcode | Faktor 1 | Faktor 2 | Resulting Ind. | Beschreibung |
|---|---|---|---|---|
| `COMP` | Operand 1 | Operand 2 | Hi / Lo / Eq | Vergleich, setzt Resulting Indicators |
| `IFEQ` | F1 | F2 | — | If F1 = F2 (alt) |
| `IFNE` | F1 | F2 | — | If F1 ≠ F2 (alt) |
| `IFGT` | F1 | F2 | — | If F1 > F2 (alt) |
| `IFLT` | F1 | F2 | — | If F1 < F2 (alt) |
| `IFGE` | F1 | F2 | — | If F1 ≥ F2 (alt) |
| `IFLE` | F1 | F2 | — | If F1 ≤ F2 (alt) |
| `IF` | — | Ausdruck | — | Freier If-Ausdruck (modernes RPG IV) |
| `ELSE` | — | — | — | Else-Zweig |
| `ELSEIF` | — | Ausdruck | — | Else-If (modernes RPG IV) |
| `ENDIF` | — | — | — | End If |
| `AND` | — | — | — | (kombiniert mit `IFxx`/`ANDxx`) |
| `OR` | — | — | — | (kombiniert mit `IFxx`/`ORxx`) |

> **Hinweis:** `IFxx`-Opcodes (IFEQ, IFNE, …) sind **veraltet** und werden in neuem Code durch `IF`-Ausdrücke ersetzt.

### Schleifen

| Opcode | Beschreibung |
|---|---|
| `DOWxx` / `DOW` | Do While — Schleife solange Bedingung wahr (Prüfung vor Schleife) |
| `DOUxx` / `DOU` | Do Until — Schleife bis Bedingung wahr (Prüfung nach Schleife) |
| `DO` | Do N-mal (mit Counter) |
| `FOR` | For-Schleife mit Zähler (modernes RPG IV) |
| `ITER` | Nächste Iteration (continue) |
| `LEAVE` | Schleife verlassen (break) |
| `ENDDO` | Ende einer DO/DOW/DOU/DO-Schleife |
| `ENDFOR` | Ende einer FOR-Schleife |

### SELECT / WHEN

| Opcode | Beschreibung |
|---|---|
| `SELECT` | Beginn einer Select-Gruppe |
| `WHENxx` / `WHEN` | Bedingungszweig |
| `OTHER` | Else-Zweig |
| `ENDSL` | Ende der Select-Gruppe |
| `CASxx` | Case-Aufruf (veraltet, → `SELECT/WHEN`) |
| `ENDCS` | Ende eines CAS-Blocks |

### Dateioperationen

| Opcode | Faktor 1 | Faktor 2 | Ergebnis | Beschreibung |
|---|---|---|---|---|
| `READ(E)` | — | Dateiname / Satzname | — | Nächsten Satz lesen |
| `READP(E)` | — | Dateiname / Satzname | — | Vorherigen Satz lesen |
| `READE(E)` | Suchschlüssel | Dateiname / Satzname | — | Gleichen Satz lesen (Equal Read) |
| `READPE(E)` | Suchschlüssel | Dateiname / Satzname | — | Gleichen vorherigen Satz lesen |
| `CHAIN(E)` | Schlüssel | Dateiname / Satzname | — | Direktzugriff per Schlüssel |
| `SETLL` | Schlüssel | Dateiname / Satzname | LO/EQ-Ind. | Set Lower Limit — Cursor positionieren |
| `SETGT` | Schlüssel | Dateiname / Satzname | HI-Ind. | Set Greater Than — nach Schlüssel positionieren |
| `WRITE(E)` | — | Satzname | — | Neuen Satz schreiben |
| `UPDATE(E)` | — | Satzname | — | Gelesenen Satz aktualisieren |
| `DELETE(E)` | Schlüssel | Dateiname / Satzname | — | Satz löschen |
| `OPEN(E)` | — | Dateiname | — | Datei öffnen (bei `USROPN`) |
| `CLOSE(E)` | — | Dateiname | — | Datei schließen |
| `EXCEPT` | Ausnahme-Name | — | — | Output-Spec-Satz ausgeben |

> **(E)** nach dem Opcode aktiviert Fehlerbehandlung via `%ERROR` anstelle von Resulting Indicators.

### Unterprogramme & Prozeduren

| Opcode | Faktor 1 | Faktor 2 | Beschreibung |
|---|---|---|---|
| `EXSR` | — | Subroutinen-Name | Subroutine aufrufen |
| `BEGSR` | Subroutinen-Name | — | Beginn einer Subroutine |
| `ENDSR` | — | — | Ende einer Subroutine |
| `CALLP` | — | Prozedur(Parameter) | Procedure-Prototype-Aufruf |
| `CALL` | — | `'Programmname'` | Altes Programm-Aufruf (ohne Prototype) |
| `PARM` | Rückgabe | Parameter | Parameter für CALL definieren (alt) |
| `PLIST` | Parameterlisten-Name | — | Parameterliste benennen (alt) |
| `RETURN` | — | Rückgabewert | Prozedur/Programm beenden |

### String-Operationen

| Opcode | Beschreibung | Fully Free Äquivalent |
|---|---|---|
| `CAT` | Strings verketten | `string1 + string2` |
| `SUBST(E)` | Teilstring extrahieren | `%SUBST(str:start:len)` |
| `SCAN(E)` | In String suchen | `%SCAN(such:str:start)` |
| `CHECK(E)` | Zeichen prüfen (von links) | `%CHECK(vergl:str:start)` |
| `CHECKR(E)` | Zeichen prüfen (von rechts) | `%CHECKR(vergl:str:start)` |
| `XLATE(E)` | Zeichen übersetzen | `%XLATE(von:nach:str)` |

### Fehlerbehandlung

| Opcode | Beschreibung |
|---|---|
| `MONITOR` | Beginn eines Monitor-Blocks (Fehlerüberwachung) |
| `ON-ERROR` | Fehlerbehandlungs-Zweig |
| `ENDMON` | Ende des Monitor-Blocks |

### Sonstiges

| Opcode | Beschreibung | Hinweis |
|---|---|---|
| `GOTO` | Sprung zu einer TAG-Markierung | Manuelle Refaktorierung nötig |
| `TAG` | Sprungmarke definieren | Ziel für GOTO |
| `DUMP` | Speicherauszug für Debug | Selten, `DUMP` in Fully Free beibehalten |
| `DEBUG` | Debug-Punkt | In Fully Free entfernen |
| `TIME` | Systemzeit/-datum holen | → `%TIME()`, `%DATE()`, `%TIMESTAMP()` |
| `TEST(E)` | Datums-/Zeitfeld validieren | → `MONITOR`-Block oder `TEST` |
| `RESET` | Variable auf Initialwert zurücksetzen | `RESET varName` |
| `CLEAR` | Variable/DS auf Nullwert setzen | `CLEAR varName` |
| `SORTA` | Array sortieren | `SORTA array` |
| `LOOKUP` | In Array suchen | → `%LOOKUP` |
| `XFOOT` | Array summieren | → `FOR`-Schleife mit `+=` |
| `OCCUR` | Multiple-Occurrence DS zugreifen | → `DIM` + Array-Index |

---

## 8. O-Spec (Output Specification)

> **Konvertierungsfokus:** O-Specs werden in Phase 0 **nicht** konvertiert.  
> Vorkommen in input.rpgle werden als `// TODO: O-Spec — manuelle Konvertierung erforderlich` markiert.

### Kurzbeschreibung

O-Specs definieren das Ausgabelayout für **programmbeschriebene** Dateien (kein `E` in F-Spec Spalte 22).

```
Spalte:  1  6  7         16 17  21 23 24 25 26 29 30 43      51 53
         ----OFilename++++ExcNam+++SBSaEdcEdcConstant/editword/DTformat+
```

Typischer Aufbau:
1. **Satzzeile** (Spalte 17 blank oder `H`/`D`/`T`/`B`): Definiert eine Ausgabezeile, welche Indikatoren aktiv sein müssen
2. **Feldzeile** (Spalte 17 blank, Feldname in Spalten 30–43): Definiert ein einzelnes Feld der Ausgabe

In modernen RPG-Projekten werden O-Specs durch externe Datenbeschreibungen (E in F-Spec) oder `WRITE` auf `PRINTER`-Dateien mit Formatnamen ersetzt. Vorkommen in Legacy-Code sind ein Hinweis auf sehr alten Code.

---

## 9. P-Spec (Procedure Specification)

### Zweck

Definiert Beginn und Ende von **Subprozeduren** (ILE-Prozeduren).  
Entspricht im Fully Free Format `dcl-proc` und `end-proc`.

### Spaltenstruktur

```
Spalte:  6  7                 22   44              80
         P+Name++++++++++++++B/E++Keywords++++++++++
```

| Spalten | Inhalt | Beschreibung |
|---|---|---|
| 6 | `P` | Spec-Typ |
| 7–21 | Prozedurname | Name der Prozedur (15 Zeichen) |
| 24 | Beginn/Ende | `B` = Begin, `E` = End |
| 44–80 | Keywords | Prozedur-Keywords |

### P-Spec-Keywords

| Keyword | Beschreibung |
|---|---|
| `EXPORT` | Prozedur ist außerhalb des Moduls aufrufbar |
| `OPDESC` | Operational Descriptors verwenden |

### Aufbau einer Prozedur

```rpgle
     * Prozedur-Beginn
     P berechne       B                   EXPORT

     * Procedure Interface
     D berechne       PI            10P 2
     D  wert1                        9P 2           VALUE
     D  wert2                        9P 2           VALUE

     * Lokale Variablen
     D result         S             10P 2

     * Logik
     C                   EVAL      result = wert1 + wert2
     C                   RETURN    result

     * Prozedur-Ende
     P berechne       E
```

---

## 10. Indicators

### Was sind Indicators?

Indicators sind **boolesche Flags** (`*ON` / `*OFF`), die in RPG-Programmen den Programmfluss steuern.  
Sie wurden in frühen RPG-Versionen anstelle von echten booleschen Variablen verwendet.

### Typen von Indicators

| Typ | Bezeichnung | Bereich | Beschreibung |
|---|---|---|---|
| Numeric | Numerische Indikatoren | `01`–`99` | Allgemeine boolsche Flags |
| Resulting | Resulting Indicators | `01`–`99` | Werden durch Operationen gesetzt |
| Overflow | Overflow-Indicator | `OA`–`OG`, `OV` | Drucker-Overflow |
| Last Record | Last Record Indicator | `LR` | Letzter Datensatz, Programmende |
| Return | Return Indicator | `RT` | Rücksprung aus Unterprogramm |
| Level | Level Break | `L1`–`L9` | Steuerfeld-Wechsel |
| Level 0 | Level 0 | `L0` | Immer true |
| Function Key | Funktionstaste | `KA`–`KN`, `KP`–`KY` | Workstation-Tasten (Phase 2) |
| Field | Field Indicator | `*INxx` | Direktzugriff als Array-Element |

### `*IN`-Array

Alle Indikatoren sind über `*IN(nn)` oder `*INxx` als boolesche Variable erreichbar:

```rpgle
     * Indikator 01 setzen
     C                   EVAL      *IN01 = *ON

     * Indikator 01 prüfen
     C   01            EVAL      ergebnis = 'Gefunden'

     * Via *IN-Array
     C                   IF        *IN(01) = *ON
```

### Resulting Indicators bei Dateioperationen

| Indikator-Position | Datei-Op | Bedeutung |
|---|---|---|
| Hi (71–72) | CHAIN, SETLL | Schlüssel existiert (→ `%FOUND`) |
| Lo (73–74) | CHAIN, READ, READE | Dateiende (→ `%EOF`) |
| Eq (75–76) | SETLL, CHAIN | Gleich (→ `%EQUAL`) |
| Hi | COMP | Faktor 1 > Faktor 2 |
| Lo | COMP | Faktor 1 < Faktor 2 |
| Eq | COMP | Faktor 1 = Faktor 2 |

### Conditioning Indicators in C-Spec

Conditioning Indicators in Spalten 9–11 bedingen, ob eine Zeile ausgeführt wird:

```rpgle
     * Nur wenn Indikator 05 gesetzt
     C   05            EVAL      x = x + 1
     * Nur wenn Indikator 05 NICHT gesetzt (N = negiert)
     C   N05           EVAL      x = 0
```

### Spezielle Indicators im Scope (Phase 0)

| Indicator | Bedeutung | Konvertierung |
|---|---|---|
| `*INLR` | Last Record — Programm normal beenden | Ersetzen durch `RETURN` am Programmende |
| `*INRT` | Return — Signalisiert Rückkehr | Programm- oder Prozedurabhängig prüfen |
| `*IN01`–`*IN99` | Allgemeine boolesche Flags | Ersetzen durch `DCL-S isXxx IND` |

> **Nicht in Phase 0:** Display-File-Indicators (`*INKx`, `*INOA`–`*INOW`) — werden in Phase 2 behandelt.

---

## 11. Free-Form innerhalb Fixed Format (`/FREE` … `/END-FREE`)

### Zweck

Vor der Einführung von Fully Free (`**FREE`) konnten ab einem bestimmten Release **freie Ausdrücke** in der C-Spec mit `/FREE` und `/END-FREE` verwendet werden.  
Dies wird als **Mixed Free** oder **Partial Free** bezeichnet.

### Syntax

```rpgle
     H DFTACTGRP(*NO)
     FCUSTMAST  IF   E           K DISK
     D name           S             30A
     ...

      /FREE
         // Freie C-Spec Ausdrücke ohne Spaltenrestriktionen
         name = 'Max Muster';
         READ custmast;
         DOW NOT %EOF(custmast);
           // Verarbeitung
           READ custmast;
         ENDDO;
         *INLR = *ON;
         RETURN;
      /END-FREE
```

### Besonderheiten

- H-, F-, D-Specs bleiben **spaltenbasiert** (Fixed Format)
- Nur der Code zwischen `/FREE` und `/END-FREE` ist frei formatiert
- Kommentare im Free-Block mit `//`
- In Fully Free (`**FREE`) entfällt `/FREE`/`/END-FREE` komplett — der gesamte Code ist frei

### Erkennungsmerkmale im Quellcode

| Merkmal | Bedeutung |
|---|---|
| Zeile beginnt mit `**FREE` | Fully Free — keine F/D/H-Specs mehr |
| Zeile beginnt mit `/FREE` | Mixed Free — H/F/D bleiben Fixed |
| Keine `**FREE`-Zeile, keine `/FREE`-Zeile | Reines Fixed Format |

---

## 12. Compiler-Direktiven

| Direktive | Beschreibung |
|---|---|
| `/COPY member` | Quelldatei einbinden (Member-Name, bei SRCFILE) |
| `/INCLUDE member` | Wie `/COPY` (neuere Alternative) |
| `/DEFINE symbol` | Compile-Zeit-Symbol definieren |
| `/UNDEFINE symbol` | Symbol entfernen |
| `/IF DEFINED(symbol)` | Bedingte Kompilierung — wenn Symbol definiert |
| `/IF NOT DEFINED(symbol)` | Bedingte Kompilierung — wenn Symbol nicht definiert |
| `/ELSEIF DEFINED(symbol)` | Alternativ-Zweig |
| `/ELSE` | Sonst-Zweig |
| `/ENDIF` | Ende bedingter Kompilierung |
| `/EOF` | Vorzeitiges Dateiende |
| `/SPACE n` | n Leerzeilen in Auflistung |
| `/EJECT` | Seitenvorschub in Auflistung |
| `/TITLE text` | Titel für Auflistung |

---

## 13. Typisches Programm-Skelett (Fixed Format)

```rpgle
      *-----------------------------------------------------------------
      * MEINPROG — Beispielprogramm im Fixed Format
      *-----------------------------------------------------------------
     H DFTACTGRP(*NO) ACTGRP(*NEW)
     H OPTION(*NODEBUGIO : *SRCSTMT)

      *----- Dateien -----------------------------------------------
     FCUSTMAST  IF   E           K DISK

      *----- Variablen ----------------------------------------------
     D kundNr         S              7P 0
     D kundName       S             30A
     D gefunden       S               N             INZ(*OFF)

      *----- Hauptprogramm -----------------------------------------
     C                   EVAL      kundNr = 1234567
     C     kundNr        CHAIN     CUSTMAST
     C                   IF        %FOUND(CUSTMAST)
     C                   EVAL      gefunden = *ON
     C                   EVAL      kundName = KNAME
     C                   ENDIF

      *----- Programmende ------------------------------------------
     C                   EVAL      *INLR = *ON
     C                   RETURN
```

---

## Zusammenfassung: Spec-Typen auf einen Blick

| Spec | Spalte 6 | Spalten 7–80 | Fully Free Äquivalent |
|---|---|---|---|
| Header | `H` | Keywords (free-form) | `ctl-opt` |
| File | `F` | Dateiname, Typ, Device, Keywords | `dcl-f` |
| Definition | `D` | Name, Typ, Länge, Datentyp, Keywords | `dcl-s` / `dcl-ds` / `dcl-c` / `dcl-pr` / `dcl-pi` |
| Calculation | `C` | Factor1, Opcode, Factor2, Result, Ind. | Freie Anweisungen |
| Output | `O` | Satzname, Felder, Indikatoren | (Phase 2) |
| Procedure | `P` | Name, B/E, Keywords | `dcl-proc` / `end-proc` |
