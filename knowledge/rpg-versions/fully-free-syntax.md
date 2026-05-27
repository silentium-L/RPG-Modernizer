# Fully Free RPG — Vollständige Syntax-Referenz

> **Zweck dieser Datei:** LLM-Kontext für den RPG-IV → Fully Free Konverter.  
> Zielformat aller Konvertierungen. Dient als alleinige Quelle für korrekte Fully Free Syntax.

---

## 1. Einleitung: Was ist Fully Free RPG?

**Fully Free RPG** (auch „Totally Free Format") ist das moderne, spaltenunabhängige Format von RPG.  
Ab **IBM i 7.1 TR7** und **IBM i 7.2** vollständig verfügbar (empfohlen: 7.2+).

Voraussetzungen:
- Compiler-Option `OPTION(*NOMONITOR)` oder neueres Compile-Level
- Datei beginnt zwingend mit `**FREE` in Spalte 1, Zeile 1

Unterschiede zu Fixed Format und Mixed Free:

| Merkmal | Fixed Format | Mixed Free (`/FREE`) | Fully Free (`**FREE`) |
|---|---|---|---|
| Spaltenposition | Pflicht (1–80) | Calc-Sektion frei | Gesamte Datei frei |
| Spec-Typen (H/F/D/C) | Pflicht | H/F/D weiterhin spaltenbasiert | Keine Spec-Typen mehr |
| Kommentar | Spalte 7 = `*` | Spalte 7 = `*` oder `//` | Nur `//` |
| Maximale Zeilenlänge | 80 Zeichen | 80 Zeichen | 100 Zeichen (Standard) |

---

## 2. `**FREE` Direktive

```rpgle
**FREE
```

- **Muss** in Spalte 1, Zeile 1 stehen — keine Leerzeichen davor
- Wirkt auf die gesamte Quelldatei
- Danach sind **keine** spaltenbasierten Spezifikationen mehr erlaubt
- Kommentare ausschließlich mit `//`
- Zeilenlänge bis 100 Zeichen (erweiterbar auf 112 mit Compiler-Option)

---

## 3. `ctl-opt` — Control Options (ersetzt H-Spec)

Definiert programmweite Compiler- und Laufzeit-Optionen. Kann mehrfach vorkommen, typischerweise am Dateianfang.

```rpgle
ctl-opt dftactgrp(*no) actgrp(*new) option(*nodebugio : *srcstmt)
        bnddir('MYBNDDIR') datedit(*dmy/) datfmt(*iso);
```

### Syntax
```
ctl-opt keyword(value) keyword(value) ... ;
```

### Gängige Keywords

| Keyword | Werte | Beschreibung |
|---|---|---|
| `DFTACTGRP(*NO)` | `*YES`, `*NO` | `*NO` = ILE-Programm (Pflicht für `dcl-proc`) |
| `ACTGRP(*NEW)` | `*NEW`, `*CALLER`, `'name'` | Activation Group |
| `BNDDIR('name')` | Bindungsverzeichnis-Name | Binding Directory für Service Programs |
| `OPTION(opt1 : opt2)` | Compiler-Optionen | Mehrere mit `:` trennen |
| `EXPROPTS(*RESDECPOS)` | `*RESDECPOS`, `*MAXDIGITS` | Dezimalstellenbehandlung |
| `DATEDIT(*DMY/)` | `*DMY/`, `*MDY/`, `*YMD/`, `*JUL/` | Datumsbearbeitungscode |
| `DATFMT(*ISO)` | `*ISO`, `*USA`, `*EUR`, `*JIS`, `*DMY`, `*MDY`, `*YMD` | Standard-Datumsformat |
| `TIMFMT(*HMS:)` | `*HMS`, `*ISO`, `*USA`, `*EUR`, `*JIS` | Standard-Zeitformat |
| `DECEDIT('0,')` | `'0,'`, `'0.'`, `','`, `'.'` | Dezimalbearbeitungscode |
| `MAIN(procName)` | Prozedurname | Einstiegsprozedur (ILE-Module) |
| `NOMAIN` | — | Modul ohne Hauptprozedur (Service Program) |
| `COPYRIGHT('text')` | Text | Copyright-String |
| `CCSID(*GRAPH:n)` | CCSID-Nummer | DBCS-CCSID |
| `ALWNULL(*USRCTL)` | `*USRCTL`, `*NO`, `*INPUTONLY` | Null-fähige Felder |
| `STGMDL(*INHERIT)` | `*SNGLVL`, `*TERASPACE`, `*INHERIT` | Speichermodell |

**Häufige `OPTION`-Werte:**

| Option | Bedeutung |
|---|---|
| `*NODEBUGIO` | Keine Debug-I/O-Zeilen |
| `*SRCSTMT` | Quellanweisungsnummern für Debug |
| `*NOSHOWCPY` | Copy-Dateien nicht im Listing anzeigen |
| `*EVENTF` | Error-Datei für IDEs (RDi) |

---

## 4. `dcl-f` — Datei-Deklaration (ersetzt F-Spec)

Deklariert Dateien (Eingabe, Ausgabe, Update, kombiniert).

### Syntax
```
dcl-f dateiname [dateityp] [dateibezeichner] [keywords] ;
```

### Dateitypen

| Keyword | Bedeutung |
|---|---|
| `DISK` | Datenbankdatei (Standard bei Auslass) |
| `PRINTER` | Druckdatei |
| `WORKSTN` | Workstation-Datei (Display File) |
| `SEQ` | Sequentielle Datei |
| `SPECIAL` | Spezielle Datei |

### Dateibezeichner (Zugriffsart)

| Keyword | Bedeutung | F-Spec Äquivalent |
|---|---|---|
| `INPUT` | Nur lesen (Standard) | `I` |
| `OUTPUT` | Nur schreiben | `O` |
| `UPDATE(E)` | Lesen/Schreiben/Löschen | `U` + `E` |
| `(Kein)` | Standard ist Input | — |

### Keywords

| Keyword | Beschreibung |
|---|---|
| `KEYED` | Schlüsselzugriff (statt sequentiell) |
| `USROPN` | Manuelle Öffnung mit `OPEN`-Opcode |
| `EXTDESC('datei')` | Externer Beschreibungsname abweichend vom Dateinamen |
| `EXTFILE(variable)` | Dateiname dynamisch aus Variable |
| `EXTMBR(member)` | Bestimmtes Member öffnen |
| `RENAME(alt : neu)` | Formatnamen umbenennen |
| `QUALIFIED` | Feldnamen über `dateiname.feld` qualifizieren |
| `PREFIX(präfix)` | Präfix zu allen Feldnamen hinzufügen |
| `PREFIX(präfix : n)` | Präfix mit Zeichenauslassung |
| `OFLIND(indikator)` | Überlaufindikator (bei PRINTER) |
| `INDDS(dsName)` | Indikator-Datenstruktur (bei WORKSTN) |
| `STATIC` | Datei bleibt zwischen Aufrufen geöffnet |
| `TEMPLATE` | Template-Deklaration (erzeugt kein I/O) |
| `BLOCK(*YES)` | Record-Blocking aktivieren |
| `SFILE(format : rrn)` | Subfile-Format und RRN-Variable |
| `INFDS(dsName)` | Informations-Datenstruktur |
| `INFSR(subrName)` | Fehler-Subroutine |
| `COMMIT` | Commitment Control |
| `RECNO(variable)` | RRN-Variable (für `WRITE` auf DISK) |

### Beispiele

```rpgle
// Einfache Eingabedatei
dcl-f KDSTAMM disk keyed;

// Update-Datei
dcl-f BESTELLUNG disk keyed update(e);

// Druckdatei mit Überlaufindikator
dcl-f BERICHT printer oflind(*in01);

// Dynamischer Dateiname
dcl-f DATEI disk extfile(dateiName) usropn;

// Qualifizierte Felder
dcl-f ARTIKEL disk keyed qualified prefix(art_);
```

---

## 5. `dcl-s` — Standalone Variable (ersetzt D-Spec Typ S)

Deklariert eine einzelne Variable.

### Syntax
```
dcl-s name datentyp [(länge : dezimalstellen)] [keywords] ;
```

### Datentypen — Vollständige Tabelle

| Fully Free Typ | Kurzzeichen | Beschreibung | Beispiel |
|---|---|---|---|
| `CHAR(n)` | `A` | Alphanumerisch (feste Länge) | `dcl-s name char(50);` |
| `VARCHAR(n)` | — | Alphanumerisch (variable Länge) | `dcl-s text varchar(200);` |
| `GRAPHIC(n)` | `G` | DBCS Graphic | `dcl-s gbez graphic(50) ccsid(835);` |
| `UCS2(n)` | `C` | Unicode UCS-2 | `dcl-s uwort ucs2(20);` |
| `BINDEC(p : s)` | `P` | Packed Decimal | `dcl-s betrag bindec(11:2);` |
| `ZONED(p : s)` | `S` | Zoned Decimal | `dcl-s menge zoned(5:0);` |
| `INT(n)` | `I` | Ganzzahl vorzeichenbehaftet (2/4/8/10/20 Stellen) | `dcl-s zaehler int(10);` |
| `UNS(n)` | `U` | Ganzzahl vorzeichenlos | `dcl-s pos uns(5);` |
| `FLOAT(n)` | `F` | Gleitkomma (4=einfach, 8=doppelt) | `dcl-s faktor float(8);` |
| `DATE` | `D` | Datum | `dcl-s heute date;` |
| `TIME` | `T` | Uhrzeit | `dcl-s jetzt time;` |
| `TIMESTAMP` | `Z` | Zeitstempel | `dcl-s ts timestamp;` |
| `IND` | `N` | Indikator (boolescher Wert `*ON`/`*OFF`) | `dcl-s gefunden ind;` |
| `POINTER` | `*` | Pointer | `dcl-s ptr pointer;` |
| `OBJECT(*JAVA:'class')` | `O` | Java-Objekt-Referenz | — |

**Ganzzahl-Größen für `INT`/`UNS`:**

| Stellen | Typ | C-Äquivalent | Wertebereich (signed) |
|---|---|---|---|
| 3 | `INT(3)` | int8 | -128 bis 127 |
| 5 | `INT(5)` | int16 | -32768 bis 32767 |
| 10 | `INT(10)` | int32 | -2.1 Mrd bis 2.1 Mrd |
| 20 | `INT(20)` | int64 | ca. ±9.2 × 10¹⁸ |

### Keywords für `dcl-s`

| Keyword | Beschreibung |
|---|---|
| `INZ(wert)` | Initialisierungswert (`INZ(*ZEROS)`, `INZ(*BLANKS)`, `INZ(*NULL)`, `INZ(*SYS)`) |
| `DIM(n)` | Array-Dimension (1-dimensionales Array) |
| `DIM(n) PERRCD(m)` | Array mit `m` Elementen pro Record (für DISK/PRINTER) |
| `ASCEND` | Array aufsteigend sortiert |
| `DESCEND` | Array absteigend sortiert |
| `STATIC` | Statischer Speicher (bleibt zwischen Aufrufen erhalten) |
| `STATIC(*ALLTHREAD)` | Thread-übergreifend statisch |
| `LIKE(feldname)` | Gleicher Typ wie Referenzfeld |
| `LIKE(feldname : n)` | Wie Referenzfeld, aber `n` Stellen länger/kürzer |
| `DATFMT(format)` | Datumsformat für DATE-Felder |
| `TIMFMT(format)` | Zeitformat für TIME-Felder |
| `CCSID(n)` | CCSID für Zeichenfelder |
| `BASED(pointer)` | Basiert auf Pointer-Variable |
| `TEMPLATE` | Nur Typ-Vorlage, kein Speicher |
| `NULLIND(indikator)` | Null-Indikator-Variable |
| `VARYING` | Variable Länge (für CHAR) |
| `VARYING(2)` oder `VARYING(4)` | Längen-Präfix 2 oder 4 Bytes |

### Beispiele

```rpgle
dcl-s kundennr    char(10)       inz(*blanks);
dcl-s betrag      bindec(11:2)   inz(*zeros);
dcl-s anzahl      int(10)        inz(0);
dcl-s gefunden    ind            inz(*off);
dcl-s heuteDatum  date           inz(*sys);
dcl-s preisTabelle bindec(9:2)   dim(100);
dcl-s name        varchar(100);
```

---

## 6. `dcl-ds` — Data Structure (ersetzt D-Spec Typ DS)

Deklariert eine Datenstruktur mit Subfeldern.

### Syntax
```
dcl-ds name [keywords] ;
  subfeld1 typ1 [keywords] ;
  subfeld2 typ2 [keywords] ;
  ...
end-ds [name] ;
```

### Keywords für `dcl-ds`

| Keyword | Beschreibung |
|---|---|
| `LIKEDS(dsName)` | Gleiche Struktur wie andere DS |
| `LIKEREC(formatName : *ALL)` | Wie Record-Format einer Datei |
| `LIKEREC(formatName : *INPUT)` | Nur Eingabefelder |
| `LIKEREC(formatName : *OUTPUT)` | Nur Ausgabefelder |
| `EXTNAME('datei' : 'format')` | Externe Beschreibung aus Datei |
| `EXTNAME('datei' : 'format' : *ALL)` | Alle Felder (inkl. I/O-Felder) |
| `QUALIFIED` | Subfeldnamen qualifizieren: `ds.subfeld` |
| `DIM(n)` | Array von Datenstrukturen |
| `INZ` | Initialisieren (`*LIKEDS`, `*LIKEREC`, `*EXTDFT`) |
| `BASED(pointer)` | Basiert auf Pointer |
| `TEMPLATE` | Nur Vorlage |
| `ALIGN` | Felder ausrichten (für Pointer-Kompatibilität) |
| `PREFIX(präfix)` | Präfix für alle Subfelder |
| `OCCURS(n)` | Mehrfachvorkommen (Legacy — `DIM` bevorzugt) |

### Subfeld-Keywords

| Keyword | Beschreibung |
|---|---|
| `POS(n)` | Absolute Position in der DS |
| `OVERLAY(feldName)` | Überlagert anderes Subfeld |
| `OVERLAY(feldName : n)` | Überlagert ab Position `n` |
| `INZ(wert)` | Initialisierungswert |
| `DIM(n)` | Subfeld ist ein Array |
| `LIKE(feldName)` | Typ von Referenzfeld übernehmen |

### Beispiele

```rpgle
// Einfache Data Structure
dcl-ds adresse qualified;
  strasse  char(40);
  plz      char(5);
  ort      char(30);
end-ds adresse;

// Externe Beschreibung (Feldnamen aus Datei KDSTAMM, Format KDREC)
dcl-ds kdRec likerec(KDREC : *input) qualified;
end-ds kdRec;

// Wie eine andere DS
dcl-ds adresse2 likeds(adresse);
end-ds adresse2;

// Array von DS
dcl-ds bestellPos qualified dim(50);
  artnr    char(10);
  menge    bindec(9:2);
  preis    bindec(11:2);
end-ds bestellPos;

// Zugriff auf qualifizierte Subfelder
adresse.strasse = 'Musterstraße 1';
bestellPos(1).menge = 10;
```

---

## 7. `dcl-c` — Named Constant (ersetzt D-Spec Typ C)

Deklariert eine Konstante mit Namen.

### Syntax
```
dcl-c name wert ;
```

### Beispiele

```rpgle
dcl-c MAX_LAENGE    100;
dcl-c STEUERSATZ    0.19;
dcl-c FIRMENNAME    'Musterfirma GmbH';
dcl-c LEER_DATUM    d'0001-01-01';
dcl-c TRUE_WERT     *on;
dcl-c FALSE_WERT    *off;
```

Konstanten können sein:
- Numerisch: `100`, `3.14159`
- Alphanumerisch: `'Text'`
- Hex: `x'0D0A'`
- Date: `d'2024-01-01'`
- Figurativ: `*BLANK`, `*ZERO`, `*ON`, `*OFF`, `*NULL`, `*HIVAL`, `*LOVAL`, `*ALL'x'`

---

## 8. `dcl-proc` / `end-proc` — Prozedur-Deklaration (ersetzt P-Spec)

Definiert eine Subprozedur (lokale oder exportierte Funktion).

### Syntax
```
dcl-proc prozedurName [EXPORT] ;
  dcl-pi prozedurName [rückgabeTyp] [keywords] ;
    parameter1  typ  [CONST] [VALUE] [OPTIONS(*NOPASS)] ;
    ...
  end-pi ;

  // Lokale Deklarationen
  dcl-s lokalVar typ ;

  // Anweisungen
  ...
  return wert ;
end-proc [prozedurName] ;
```

### `dcl-proc` Keywords

| Keyword | Beschreibung |
|---|---|
| `EXPORT` | Prozedur ist außerhalb des Moduls aufrufbar |

### `dcl-pi` Keywords (Procedure Interface)

| Keyword | Beschreibung |
|---|---|
| Rückgabetyp | Optional — Datentyp wie in `dcl-s` |
| `EXTPROC(name)` | Externer Prozedurname abweichend |
| `LIKE(feld)` | Rückgabetyp wie Referenzfeld |

### Parameter-Keywords

| Keyword | Beschreibung |
|---|---|
| `CONST` | Parameter als Konstante (Wert kann nicht geändert werden) |
| `VALUE` | Übergabe per Wert (Kopie, keine Referenz) |
| `OPTIONS(*NOPASS)` | Parameter ist optional |
| `OPTIONS(*OMIT)` | Parameter kann ausgelassen werden (`*OMIT` übergeben) |
| `OPTIONS(*VARSIZE)` | Variable Länge erlaubt |
| `OPTIONS(*STRING)` | Null-terminierter String erlaubt |

### Beispiele

```rpgle
// Prozedur ohne Rückgabewert (Void)
dcl-proc berechneSteuer;
  dcl-pi *n;
    brutto  bindec(11:2) value;
    netto   bindec(11:2);
  end-pi;

  netto = brutto / 1.19;
end-proc berechneSteuer;

// Exportierte Prozedur mit Rückgabewert
dcl-proc pruefeDatum export;
  dcl-pi *n ind;
    eingabe  char(10) const;
  end-pi;

  dcl-s ergebnis ind inz(*off);
  // ... Logik ...
  return ergebnis;
end-proc pruefeDatum;

// Prozedur mit optionalem Parameter
dcl-proc formatBetrag export;
  dcl-pi *n char(20);
    wert     bindec(11:2) value;
    waehrung char(3)      const options(*nopass);
  end-pi;

  if %parms >= 2;
    return %char(wert) + ' ' + waehrung;
  else;
    return %char(wert) + ' EUR';
  endif;
end-proc;
```

**Wichtig:** Der Name in `dcl-pi` kann `*n` sein (anonymes Interface) — üblich wenn `dcl-proc` und `dcl-pr` gleichen Namen haben.

---

## 9. `dcl-pr` — Prototype (ersetzt D-Spec Typ PR)

Deklariert den Aufruf-Prototyp einer externen Prozedur oder eines externen Programms. Ermöglicht dem Compiler Parameterprüfung.

### Syntax
```
dcl-pr prozedurName [rückgabeTyp] [EXTPROC(name) | EXTPGM(name)] ;
  parameter1  typ  [CONST] [VALUE] [OPTIONS(*NOPASS)] ;
  ...
end-pr ;
```

### Keywords

| Keyword | Beschreibung |
|---|---|
| `EXTPROC(name)` | Externer Prozedurname (case-sensitive) |
| `EXTPROC(*CL : name)` | CL-Prozedur |
| `EXTPROC(*CWIDEN : name)` | C-Prozedur mit Widening |
| `EXTPGM(name)` | Externes Programm (OPM-/ILE-Programm) |
| `EXTPGM` | Programmname = Prototyp-Name |
| `RTNPARM` | Rückgabewert als versteckter erster Parameter |
| `OPDESC` | Operational Descriptors |

### Beispiele

```rpgle
// Prototype für eine Prozedur im gleichen Service Program
dcl-pr berechneSteuer;
  brutto  bindec(11:2) value;
  netto   bindec(11:2);
end-pr;

// Prototype für externen Programmaufruf
dcl-pr sendeEmail extpgm('SENDMAIL');
  empfaenger  char(100) const;
  betreff     char(200) const;
  text        varchar(5000) const;
end-pr;

// Prototype für C-Funktion
dcl-pr c_strlen int(10) extproc('strlen');
  str  pointer value options(*string);
end-pr;
```

---

## 10. Freie Ausdrücke — Operatoren und Built-in Functions

### 10.1 Zuweisungs- und Rechenoperatoren

```rpgle
// Zuweisung
variable = ausdruck;

// Arithmetische Operatoren
a = b + c;
a = b - c;
a = b * c;
a = b / c;
a = b ** 2;       // Potenz

// Kombinierte Zuweisungen (RPG: explizit ausschreiben)
anzahl = anzahl + 1;   // Kein ++ in RPG
```

### 10.2 Vergleichsoperatoren

| Operator | Bedeutung |
|---|---|
| `=` | Gleich |
| `<>` | Ungleich |
| `<` | Kleiner als |
| `>` | Größer als |
| `<=` | Kleiner oder gleich |
| `>=` | Größer oder gleich |

### 10.3 Logische Operatoren

```rpgle
if a > 0 and b > 0;   // AND
if a > 0 or  b > 0;   // OR
if not gefunden;       // NOT
```

### 10.4 String-Operatoren

```rpgle
// Verkettung mit +
fullName = firstName + ' ' + lastName;

// Auf Länge prüfen mit %len
if %len(text) > 0;
```

### 10.5 Built-in Functions (BIFs) — Vollständige Übersicht

#### String-BIFs

| BIF | Parameter | Rückgabe | Beschreibung |
|---|---|---|---|
| `%LEN(var)` | Variable | `INT(10)` | Länge des Inhalts (bei VARCHAR) bzw. definierte Länge |
| `%TRIM(str)` | String | CHAR/VARCHAR | Leerzeichen beidseitig entfernen |
| `%TRIML(str)` | String | CHAR/VARCHAR | Führende Leerzeichen entfernen |
| `%TRIMR(str)` | String | CHAR/VARCHAR | Nachfolgende Leerzeichen entfernen |
| `%SUBST(str : start)` | String, Start | CHAR/VARCHAR | Teilstring ab Position |
| `%SUBST(str : start : len)` | String, Start, Länge | CHAR/VARCHAR | Teilstring |
| `%SCAN(such : basis)` | Such, Basis | `INT(10)` | Position des Suchstrings (0 = nicht gefunden) |
| `%SCAN(such : basis : start)` | Such, Basis, Start | `INT(10)` | Suche ab Position |
| `%CHECK(check : basis)` | Check, Basis | `INT(10)` | Position erster Nicht-Check-Zeichen |
| `%CHECK(check : basis : start)` | Check, Basis, Start | `INT(10)` | — |
| `%CHECKR(check : basis)` | Check, Basis | `INT(10)` | Von rechts prüfen |
| `%XLATE(von : nach : str)` | Von, Nach, String | CHAR | Zeichen übersetzen |
| `%XLATE(von : nach : str : start)` | — | CHAR | Ab Position übersetzen |
| `%REPLACE(neu : alt : start)` | Neu, Alt, Start | VARCHAR | Teilstring ersetzen |
| `%REPLACE(neu : alt : start : len)` | — | VARCHAR | Mit Längenangabe |
| `%CONCAT(str1 : str2 : ...)` | Strings | VARCHAR | Strings verbinden |
| `%STR(pointer)` | Pointer | VARCHAR | Null-terminated C-String lesen |
| `%STR(pointer : len)` | Pointer, Max-Länge | VARCHAR | — |
| `%UPPER(str)` | String | VARCHAR | Großbuchstaben (ab 7.2) |
| `%LOWER(str)` | String | VARCHAR | Kleinbuchstaben (ab 7.2) |
| `%CHAR(wert)` | Numerisch/Datum | CHAR | In Zeichenform konvertieren |
| `%CHAR(wert : format)` | — | CHAR | Mit Format-Angabe |
| `%EDITC(wert : code)` | Zahl, Edit-Code | CHAR | Edit-Code anwenden |
| `%EDITW(wert : maske)` | Zahl, Maske | CHAR | Edit-Maske anwenden |

#### Numerische BIFs

| BIF | Parameter | Rückgabe | Beschreibung |
|---|---|---|---|
| `%INT(wert)` | Numeric | `INT(10)` | In Integer konvertieren (truncate) |
| `%INTH(wert)` | Numeric | `INT(10)` | In Integer konvertieren (runden) |
| `%DEC(wert : p : s)` | Wert, Precision, Scale | `BINDEC(p:s)` | In Packed Decimal konvertieren |
| `%DECH(wert : p : s)` | — | `BINDEC(p:s)` | Mit Runden |
| `%DECPOS(wert)` | Numeric | `INT(10)` | Anzahl Dezimalstellen |
| `%FLOAT(wert)` | Numeric | `FLOAT(8)` | In Float konvertieren |
| `%ABS(wert)` | Numeric | Numeric | Absolutwert |
| `%MAX(val1 : val2)` | Zwei Werte | Typ der Operanden | Maximum |
| `%MIN(val1 : val2)` | Zwei Werte | Typ der Operanden | Minimum |
| `%REM(dividend : divisor)` | Zwei Zahlen | Integer | Ganzzahliger Rest |
| `%DIV(dividend : divisor)` | Zwei Zahlen | Integer | Ganzzahldivision |
| `%SQRT(wert)` | Numeric | `FLOAT(8)` | Quadratwurzel |
| `%SIZE(var)` | Variable | `INT(10)` | Größe in Bytes |
| `%ELEM(array)` | Array/DS | `INT(10)` | Anzahl Elemente |
| `%LOOKUP(key : array)` | Schlüssel, Array | `INT(10)` | Array-Position (0 = nicht gefunden) |
| `%LOOKUPLT(key : array)` | — | `INT(10)` | Kleinstes Element < key |
| `%LOOKUPLE(key : array)` | — | `INT(10)` | Kleinstes Element <= key |
| `%LOOKUPGT(key : array)` | — | `INT(10)` | Größtes Element > key |
| `%LOOKUPGE(key : array)` | — | `INT(10)` | Größtes Element >= key |
| `%TLOOKUP(key : array)` | Schlüssel, DS-Array | `INT(10)` | In DS-Array suchen |

#### Datums- und Zeit-BIFs

| BIF | Parameter | Rückgabe | Beschreibung |
|---|---|---|---|
| `%DATE` | — | `DATE` | Aktuelles Datum |
| `%DATE(wert)` | Char/Numeric | `DATE` | Wert in Datum konvertieren |
| `%DATE(wert : format)` | — | `DATE` | Mit Format-Angabe |
| `%TIME` | — | `TIME` | Aktuelle Uhrzeit |
| `%TIME(wert)` | — | `TIME` | Konvertieren |
| `%TIMESTAMP` | — | `TIMESTAMP` | Aktueller Zeitstempel |
| `%TIMESTAMP(wert)` | — | `TIMESTAMP` | Konvertieren |
| `%TIMESTAMP(wert : *ISO0)` | — | `TIMESTAMP` | ISO ohne Trennzeichen |
| `%DIFF(dat1 : dat2 : einheit)` | Zwei Daten, Einheit | Numeric | Differenz (Einheit: `*DAYS`, `*MONTHS`, `*YEARS`, `*HOURS`, `*MINUTES`, `*SECONDS`, `*MSECONDS`) |
| `%HOURS(n)` | Zahl | Dauer | n Stunden als Dauer |
| `%MINUTES(n)` | Zahl | Dauer | n Minuten als Dauer |
| `%SECONDS(n)` | Zahl | Dauer | n Sekunden als Dauer |
| `%DAYS(n)` | Zahl | Dauer | n Tage als Dauer |
| `%MONTHS(n)` | Zahl | Dauer | n Monate als Dauer |
| `%YEARS(n)` | Zahl | Dauer | n Jahre als Dauer |

Datumsarithmetik:
```rpgle
// Datum addieren
dcl-s morgen date;
morgen = %date() + %days(1);

// Differenz
dcl-s tage int(5);
tage = %diff(enddatum : startdatum : *days);
```

#### Status- und I/O-BIFs

| BIF | Kontext | Rückgabe | Beschreibung |
|---|---|---|---|
| `%FOUND` | Nach Datei-Op | `IND` | Datensatz gefunden (CHAIN, SETLL) |
| `%FOUND(dateiName)` | — | `IND` | Für spezifische Datei |
| `%EOF` | Nach READ* | `IND` | Dateiende erreicht |
| `%EOF(dateiName)` | — | `IND` | Für spezifische Datei |
| `%EQUAL` | Nach SETLL/CHAIN | `IND` | Exakte Übereinstimmung |
| `%EQUAL(dateiName)` | — | `IND` | Für spezifische Datei |
| `%ERROR` | Nach Op mit `E` | `IND` | Fehler aufgetreten |
| `%STATUS` | Nach Op | `INT(5)` | Programmstatus-Code |
| `%STATUS(dateiName)` | — | `INT(5)` | Datei-Status-Code |
| `%OPEN(dateiName)` | — | `IND` | Datei ist geöffnet |
| `%PARMS` | In Prozedur | `INT(5)` | Anzahl übergebener Parameter |
| `%PARMNUM(parmName)` | In Prozedur | `INT(5)` | Position des Parameters |

#### Pointer- und Speicher-BIFs

| BIF | Beschreibung |
|---|---|
| `%ADDR(variable)` | Adresse einer Variable |
| `%ADDR(array(n))` | Adresse von Array-Element n |
| `%PADDR(prozName)` | Adresse einer Prozedur (Procedure Pointer) |
| `%NULL` | Null-Pointer-Literal |

#### Sonstige BIFs

| BIF | Rückgabe | Beschreibung |
|---|---|---|
| `%FIELDS(f1 : f2)` | Feldliste | Für `UPDATE`-Opcode (nur bestimmte Felder updaten) |
| `%KDS(dsName)` | Schlüssel-DS | Schlüssel-DS für CHAIN/SETLL |
| `%KDS(dsName : n)` | Schlüssel-DS | n Schlüsselfelder der DS verwenden |
| `%BITAND(a : b)` | `CHAR(1)` | Bitweises AND |
| `%BITOR(a : b)` | `CHAR(1)` | Bitweises OR |
| `%BITXOR(a : b)` | `CHAR(1)` | Bitweises XOR |
| `%BITNOT(a)` | `CHAR(1)` | Bitweises NOT |

---

## 11. Kontrollstrukturen

### 11.1 IF / ELSEIF / ELSE / ENDIF

```rpgle
if bedingung;
  // Anweisungen
elseif andereBedingung;
  // Anweisungen
else;
  // Anweisungen
endif;
```

Bedingungen können beliebig komplex sein:
```rpgle
if (alter >= 18 and land = 'DE') or sonderregel = *on;
  // Volljährig oder Sonderregel
endif;
```

### 11.2 DOW — Do While

Führt Schleife aus, **solange** Bedingung wahr:

```rpgle
dow not %eof(KDSTAMM);
  read KDSTAMM;
  if not %eof(KDSTAMM);
    // Verarbeitung
  endif;
enddo;
```

### 11.3 DOU — Do Until

Führt Schleife aus, **bis** Bedingung wahr (mindestens einmal):

```rpgle
dou %eof(BESTELLUNG);
  read BESTELLUNG;
  // Verarbeitung
enddo;
```

### 11.4 FOR / ENDFOR

Zählschleife:

```rpgle
for i = 1 to 10;
  // i läuft von 1 bis 10
endfor;

for i = 10 downto 1;
  // rückwärts
endfor;

for i = 0 to 100 by 5;
  // Schrittweite 5
endfor;
```

`i` muss ganzzahlig sein. `ITER` und `LEAVE` funktionieren in allen Schleifentypen.

### 11.5 ITER und LEAVE

```rpgle
dow bedingung;
  if sonderfall;
    iter;    // Nächste Iteration (entspricht continue)
  endif;
  if abbruch;
    leave;   // Schleife verlassen (entspricht break)
  endif;
enddo;
```

### 11.6 SELECT / WHEN / OTHER / ENDSL

```rpgle
select;
  when status = '00';
    // Normalfall
  when status = '01' or status = '02';
    // Fehlerklasse 1
  when status >= '10' and status <= '19';
    // Fehlerklasse 10-19
  other;
    // Alle anderen Fälle
endsl;
```

---

## 12. Dateioperationen

### Allgemeines Syntax-Muster

```rpgle
opcode dateiName | formatName [(optionen)] ;
```

Fehlerbehandlung: `(E)` nach dem Dateinamen verhindert Programmabbruch bei Fehler; danach `%ERROR` prüfen.

```rpgle
read KDSTAMM (e);
if %error;
  // Fehlerbehandlung
endif;
```

### Sequentielle Operationen

| Opcode | Syntax | Beschreibung |
|---|---|---|
| `READ` | `read datei;` | Nächsten Datensatz lesen |
| `READP` | `readp datei;` | Vorherigen Datensatz lesen |
| `READE` | `reade suchArg datei;` | Nächsten Datensatz mit gleichem Schlüssel lesen |
| `READPE` | `readpe suchArg datei;` | Vorherigen Datensatz mit gleichem Schlüssel |

Nach READ*: `%EOF(datei)` prüfen.

```rpgle
read KDSTAMM;
dow not %eof(KDSTAMM);
  // Verarbeitung
  read KDSTAMM;
enddo;
```

### Schlüsselbasierte Operationen

| Opcode | Syntax | Beschreibung |
|---|---|---|
| `CHAIN` | `chain (schlüssel) datei;` | Direktzugriff per Schlüssel |
| `SETLL` | `setll (schlüssel) datei;` | Positionierung: unterster Datensatz >= Schlüssel |
| `SETGT` | `setgt (schlüssel) datei;` | Positionierung: unterster Datensatz > Schlüssel |

Nach `CHAIN`: `%FOUND(datei)` prüfen.  
Nach `SETLL`: `%EQUAL(datei)` für exakte Übereinstimmung.

```rpgle
chain (kundennr) KDSTAMM;
if %found(KDSTAMM);
  // Datensatz gefunden, Felder sind geladen
else;
  // Nicht gefunden
endif;
```

### Schreib- und Änderungsoperationen

| Opcode | Syntax | Beschreibung |
|---|---|---|
| `WRITE` | `write formatName;` | Neuen Datensatz schreiben |
| `UPDATE` | `update formatName;` | Zuletzt gelesenen Datensatz ändern |
| `UPDATE` | `update formatName %fields(f1:f2);` | Nur bestimmte Felder ändern |
| `DELETE` | `delete formatName;` | Zuletzt gelesenen Datensatz löschen |

**Wichtig:** `UPDATE` und `DELETE` arbeiten auf dem zuletzt mit `READ`/`CHAIN` gelesenen Datensatz.

```rpgle
chain (bestellnr) BESTELLUNG;
if %found(BESTELLUNG);
  bstStatus = 'E';   // Erledigt
  update BSTREC %fields(bstStatus);
endif;
```

### Datei öffnen/schließen

```rpgle
open KDSTAMM;     // Nur bei USROPN nötig
close KDSTAMM;    // Datei explizit schließen
```

Nach `CLOSE` ist die Datei nicht mehr nutzbar, bis sie wieder geöffnet wird.

---

## 13. Fehlerbehandlung

### 13.1 MONITOR / ON-ERROR / ENDMON

Strukturierte Fehlerbehandlung, vergleichbar mit try/catch:

```rpgle
monitor;
  // Riskante Operationen
  chain (key) DATEI;
  berechneSteuer(brutto : netto);
on-error;
  // Generischer Error-Handler
  meldung = 'Unbekannter Fehler: ' + %char(%status);
on-error 00100 : 00101;
  // Spezifische Statuscodes
  meldung = 'Datenbankfehler';
on-error *file;
  // Alle Dateifehler
  meldung = 'Dateifehler';
on-error *program;
  // Alle Programmfehler
  meldung = 'Programmfehler';
endmon;
```

### 13.2 `%STATUS` — Statuscodes

| Codebereich | Bedeutung |
|---|---|
| 00000 | Kein Fehler |
| 00100–00199 | Zeichenkonvertierungsfehler |
| 00200–00299 | Arithmetische Fehler (Division durch 0: 00102) |
| 01000–01299 | Datei-I/O-Fehler |
| 01021 | Datensatz gesperrt |
| 01211 | Dateiende gelesen |
| 01221 | Datensatz nicht gefunden (CHAIN) |

```rpgle
if %status = 01021;
  // Datensatz ist gesperrt
endif;
```

### 13.3 `%ERROR` — Einfache Fehlerprüfung

Nach Operationen mit `(E)`-Extender:

```rpgle
chain(e) (nr) DATEI;
if %error;
  dsply 'Fehler: ' + %char(%status);
  return;
endif;
```

---

## 14. Subprozeduren — Aufruf, Rückgabewert, Rekursion

### 14.1 Prozeduraufruf

```rpgle
// Mit Rückgabewert
ergebnis = meineProzedur(param1 : param2);

// Ohne Rückgabewert (void)
meineProzedur(param1 : param2);

// Subroutinen aus dem Hauptprogramm (Exsr-Equivalent)
berechnungsDurchfuehren();   // dcl-proc ohne Parameter

// Mit callp (explizite Syntax, optional)
callp meineProzedur(param1 : param2);
```

### 14.2 Rückgabewert

```rpgle
dcl-proc addiere;
  dcl-pi *n bindec(11:2);
    a bindec(9:2) value;
    b bindec(9:2) value;
  end-pi;

  return a + b;
end-proc;
```

### 14.3 Rekursion

Fully Free RPG unterstützt direkte Rekursion:

```rpgle
dcl-proc fakultaet;
  dcl-pi *n int(10);
    n int(5) value;
  end-pi;

  if n <= 1;
    return 1;
  endif;
  return n * fakultaet(n - 1);
end-proc;
```

**Hinweis:** Bei Rekursion lokale Variablen in `dcl-s` deklarieren (kein `STATIC`).

### 14.4 Hauptprogramm-Struktur mit Subprozeduren

```rpgle
**free

ctl-opt dftactgrp(*no) actgrp(*new);

// Globale Deklarationen
dcl-f KDSTAMM disk keyed;

// Prototypen (für interne Prozeduren optional, für externe Pflicht)
dcl-pr berechneSteuer;
  brutto bindec(11:2) value;
  netto  bindec(11:2);
end-pr;

// Hauptprogramm
dcl-s kundenbetrag bindec(11:2);
dcl-s nettoBetrag  bindec(11:2);

chain ('KD001') KDSTAMM;
if %found(KDSTAMM);
  kundenbetrag = KDUMSATZ;
  berechneSteuer(kundenbetrag : nettoBetrag);
endif;

*inlr = *on;
return;

// Subprozeduren
dcl-proc berechneSteuer;
  dcl-pi *n;
    brutto bindec(11:2) value;
    netto  bindec(11:2);
  end-pi;

  netto = brutto / 1.19;
end-proc;
```

---

## 15. I/O-Operationen mit EXTPGM / EXTPROC

### 15.1 EXTPGM — Externes Programm aufrufen

Ruft ein anderes IBM-i-Programm auf (OPM oder ILE). Vorher Prototype mit `EXTPGM` deklarieren:

```rpgle
// Prototype für externes Programm
dcl-pr sendeEmail extpgm('SENDMAIL');
  empfaenger char(100) const;
  betreff    char(200) const;
  text       char(1000) const;
  rueckcode  char(2);
end-pr;

// Aufruf
dcl-s rc char(2);
sendeEmail('user@example.com' : 'Betreff' : 'Text' : rc);
```

### 15.2 EXTPROC — Externe Prozedur aufrufen

Ruft eine Prozedur aus einem anderen Modul/Service Program auf:

```rpgle
// Prototype für Prozedur in anderem Service Program
dcl-pr berechneRabatt bindec(5:2) extproc('RABATTBERECHNUNG_berechneRabatt');
  umsatz    bindec(11:2) value;
  kundentyp char(1)      const;
end-pr;

// Aufruf
dcl-s rabatt bindec(5:2);
rabatt = berechneRabatt(umsatz : kundentyp);
```

### 15.3 CALLP — Programm/Prozedur ohne Prototype

Legacy-Syntax. Prototype bevorzugen.

```rpgle
callp programm('PGMNAME' : param1 : param2);
```

### 15.4 Dynamischer Prozeduraufruf via Procedure Pointer

```rpgle
dcl-pr procTemplate;
  wert int(10) value;
end-pr;

dcl-s procPtr pointer(*proc);

procPtr = %paddr('MEINE_PROZEDUR');
callp procPtr(42);
```

---

## Anhang A: Figurative Konstanten

| Konstante | Bedeutung |
|---|---|
| `*BLANK` / `*BLANKS` | Leerzeichen (für CHAR) |
| `*ZERO` / `*ZEROS` | Null (für alle numerischen Typen, CHAR) |
| `*ON` | Indikator eingeschaltet (`'1'`) |
| `*OFF` | Indikator ausgeschaltet (`'0'`) |
| `*NULL` | Null-Pointer |
| `*HIVAL` | Höchster möglicher Wert des Typs |
| `*LOVAL` | Niedrigster möglicher Wert des Typs |
| `*ALL'x'` | Alle Bytes auf Wert `x` setzen |
| `*OMIT` | Parameter ausgelassen (für `OPTIONS(*OMIT)`) |
| `*NOPASS` | Kennzeichnung übergangener optionaler Parameter |

---

## Anhang B: Compiler-Direktiven

| Direktive | Bedeutung |
|---|---|
| `**FREE` | Fully Free Format aktivieren (Zeile 1) |
| `/COPY member` | Copy-Datei einfügen |
| `/INCLUDE member` | Wie /COPY, aber keine Duplikate |
| `/IF DEFINED(symbol)` | Bedingte Kompilierung |
| `/ELSEIF` | — |
| `/ELSE` | — |
| `/ENDIF` | — |
| `/DEFINE symbol` | Symbol definieren |
| `/UNDEFINE symbol` | Symbol aufheben |
| `/EOF` | Dateiende simulieren |
| `/SPACE n` | Leerzeilen im Listing |
| `/EJECT` | Seitenvorschub im Listing |
| `/TITLE text` | Titelzeile im Listing |

---

## Anhang C: Spezielle RPG-Variablen und Indikatoren

| Variable | Typ | Bedeutung |
|---|---|---|
| `*INLR` | `IND` | Last Record Indicator — Programm beenden |
| `*IN01`–`*IN99` | `IND` | Allgemeine Indikatoren |
| `*INRT` | `IND` | Return-Indikator |
| `*IN` | `CHAR(99)` | Indikator-Array (1 Byte pro Indikator) |
| `*DETIND` | `IND` | Detail-Indikator (für Report) |

Fully Free verzichtet wo möglich auf Indikatoren; Ersatz: boolesche `dcl-s`-Variablen, `%FOUND`, `%EOF`, `%ERROR`.

---

*Letzte Überarbeitung: Phase 0 — Basis-Referenz für Konvertierungs-Engine*
