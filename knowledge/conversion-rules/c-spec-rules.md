# C-Spec → Fully Free Konvertierungsregeln

> Herzstück der Konvertierung. Alle C-Spec-Opcodes mit Mapping auf freie Syntax.

---

## Einleitung

### C-Spec Spaltenstruktur (Wiederholung)

```
Spalten  Inhalt
1–5      Zeilennummer (optional)
6        Spec-Typ: C
7–8      Conditioning Indicators (z.B. 10, LR)
9–17     Factor 1 (linker Operand)
18–26    Opcode (+ Extender: z.B. READ(E), CHAIN(N))
27–35    Factor 2 (rechter Operand / Dateiname)
36–49    Result Field (Ergebnis-Variable)
50–51    Result Field Länge
52–53    Result Field Dezimalstellen
54–55    Resulting Indicator HI (oder 01)
56–57    Resulting Indicator LO (oder 02)
58–59    Resulting Indicator EQ (oder 03)
```

**Opcode-Extender** stehen in Klammern direkt nach dem Opcode:
- `(E)` — Error-Handling aktivieren (`%ERROR` / `%STATUS` nutzbar)
- `(N)` — No-lock bei READ-Operationen
- `(H)` — Half-adjust (Runden)
- `(P)` — Pad (bei String-Operationen)

### Konvertierungsprinzip

Im Fully Free Format gibt es keine Spalten mehr. Die Logik wird in freie Ausdrücke übersetzt:

1. **Factor 1 + Opcode + Factor 2 → freier Ausdruck** (z.B. `A ADD B` → `result += b`)
2. **Conditioning Indicators → IF-Block** um die betroffene Operation
3. **Resulting Indicators → boolesche Variablen oder Built-in-Functions** (`%FOUND`, `%EOF`, `%ERROR`)
4. **Opcode-Extender `(E)`** → `MONITOR/ON-ERROR/ENDMON`-Block oder `%ERROR`-Prüfung

---

## 1. Zuweisung & Arithmetik

### `EVAL` → direkte Zuweisung

```rpgle
// Fixed Format
C                   EVAL      result = a + b

// Fully Free
result = a + b;
```

`EVAL` fällt einfach weg — der Ausdruck bleibt unverändert. In Fully Free ist die Zuweisung (`=`) direkt gültig.

**Sonderfall `EVAL(H)` (Half-adjust/Runden):**
```rpgle
// Fixed Format
C                   EVAL(H)   result = a / b

// Fully Free — %DEC mit gewünschten Stellen oder explizites Runden
result = %dech(a / b : 7 : 2);   // Dezimaldarstellung mit Runden
```

---

### `MOVE` / `MOVEL` → Zuweisung + ggf. `%SUBST`

`MOVE` und `MOVEL` kopieren Werte zeichenweise von rechts (MOVE) bzw. von links (MOVEL). In Fully Free wird das je nach Kontext unterschiedlich übersetzt.

#### Einfacher Fall: gleicher Typ, gleiche Länge
```rpgle
// Fixed Format
C                   MOVE      source        target

// Fully Free
target = source;
```

#### Numerisch → Zeichen oder umgekehrt
```rpgle
// Fixed Format
C                   MOVE      numVar        charVar

// Fully Free
charVar = %char(numVar);

// Zeichen → Numerisch
C                   MOVE      charVar       numVar
numVar = %int(charVar);   // oder %dec, %uns je nach Zieltyp
```

#### Partial Move (unterschiedliche Längen)
```rpgle
// Fixed Format — MOVE: rechtsbündig, überschreibt rechten Teil
C                   MOVE      'ABC'         result         // result ist 10 Stellen

// Fully Free — Zielfeld wird rechtsbündig befüllt (Padding links)
// Meist ist eine direkte Zuweisung ausreichend, da RPG automatisch auffüllt:
result = 'ABC';

// Wenn Teile erhalten bleiben sollen:
result = %subst(result : 1 : %len(result) - 3) + 'ABC';
```

**Wichtiger Hinweis:** `MOVE`/`MOVEL` mit Typkonvertierungen (Datum/Zeit, numerisch/alphanumerisch) sind komplex. Im Zweifelsfall TODO-Marker setzen und manuell prüfen.

---

### `Z-ADD` → Zuweisung mit Null-Initialisierung

```rpgle
// Fixed Format: Setzt result auf 0, dann addiert Factor2
C                   Z-ADD     sourceVal     result

// Fully Free
result = sourceVal;
// (Z-ADD = result wird auf 0 gesetzt, dann source dazu addiert → entspricht einfacher Zuweisung)
```

---

### `Z-SUB` → Vorzeichenwechsel

```rpgle
// Fixed Format: Setzt result auf 0, dann subtrahiert Factor2
C                   Z-SUB     sourceVal     result

// Fully Free
result = -sourceVal;
```

---

### `ADD` → `+=`

```rpgle
// Fixed Format: result = Factor1 + Factor2 (Factor1 optional → result = result + Factor2)
C                   ADD       addend        result
C     base          ADD       addend        result

// Fully Free
result += addend;
result = base + addend;
```

---

### `SUB` → `-=`

```rpgle
// Fixed Format
C                   SUB       subtrahend    result
C     base          SUB       subtrahend    result

// Fully Free
result -= subtrahend;
result = base - subtrahend;
```

---

### `MULT` → `*=`

```rpgle
// Fixed Format
C                   MULT      factor        result
C     val1          MULT      val2          result

// Fully Free
result *= factor;
result = val1 * val2;
```

---

### `DIV` → `/=`

```rpgle
// Fixed Format
C                   DIV       divisor       result
C     dividend      DIV       divisor       result

// Fully Free
result /= divisor;
result = dividend / divisor;
```

**Wichtig:** Nach `DIV` kann `MVR` folgen (s. unten). Den Rest-Wert ggf. mit `%REM` vorab sichern.

---

### `MVR` → `%REM`

`MVR` liefert den Restbetrag der letzten `DIV`-Operation.

```rpgle
// Fixed Format
C     dividend      DIV       divisor       quotient
C                   MVR                     remainder

// Fully Free
quotient  = dividend / divisor;
remainder = %rem(dividend : divisor);
```

---

## 2. Vergleich & Bedingungen

### `COMP` → `IF`-Ausdruck

`COMP` vergleicht Factor 1 mit Factor 2 und setzt Resulting Indicators (HI/LO/EQ).

```rpgle
// Fixed Format: Spalten 54–59 = HI/LO/EQ Indikatoren
C     fieldA        COMP      fieldB                    10 20 30
// *IN10 = *ON wenn fieldA > fieldB
// *IN20 = *ON wenn fieldA < fieldB
// *IN30 = *ON wenn fieldA = fieldB

// Fully Free — Indicators durch boolesche Variablen ersetzen:
dcl-s isHigh  ind;
dcl-s isLow   ind;
dcl-s isEqual ind;

isHigh  = (fieldA > fieldB);
isLow   = (fieldA < fieldB);
isEqual = (fieldA = fieldB);
```

Wenn nur ein Indicator genutzt wird:
```rpgle
// Häufig reicht direkte Prüfung:
if fieldA > fieldB;
  // HI-Zweig
elseif fieldA < fieldB;
  // LO-Zweig
else;
  // EQ-Zweig
endif;
```

---

### `IFxx` → `IF`

Alle `IFxx`-Varianten werden zu `IF`-Ausdrücken:

| Fixed Format Opcode | Operator Fully Free |
|---------------------|---------------------|
| `IFEQ` | `=` |
| `IFNE` | `<>` |
| `IFGT` | `>` |
| `IFLT` | `<` |
| `IFGE` | `>=` |
| `IFLE` | `<=` |

```rpgle
// Fixed Format
C     fieldA        IFEQ      fieldB
C     fieldA        IFGT      100

// Fully Free
if fieldA = fieldB;
if fieldA > 100;
```

---

### `ANDxx` / `ORxx` → `AND` / `OR`

`ANDxx`/`ORxx` folgen in Fixed Format auf `IFxx`, `DOWxx` oder `DOUxx`.

```rpgle
// Fixed Format
C     status        IFEQ      'ACTIVE'
C     count         IFGT      0

// Fully Free
if status = 'ACTIVE' and count > 0;
```

```rpgle
// Fixed Format
C     typeA         IFEQ      'X'
C     typeB         OREQ      'Y'

// Fully Free
if typeA = 'X' or typeB = 'Y';
```

---

### `ELSE` → `ELSE`

```rpgle
// Fixed Format
C                   ELSE

// Fully Free
else;
```

---

### `END` / `ENDIF` → `ENDIF`

```rpgle
// Fixed Format
C                   END
C                   ENDIF

// Fully Free
endif;
```

---

## 3. Schleifen

### `DOWxx` → `DOW`

`DOWxx` = Do-While mit Vergleichs-Opcode.

| Fixed Format | Fully Free |
|---|---|
| `DOWEQ` | `dow a = b;` |
| `DOWNE` | `dow a <> b;` |
| `DOWGT` | `dow a > b;` |
| `DOWLT` | `dow a < b;` |
| `DOWGE` | `dow a >= b;` |
| `DOWLE` | `dow a <= b;` |

```rpgle
// Fixed Format
C     *IN10         DOWEQ     *OFF

// Fully Free
dow not isFound;   // oder: dow *in10 = *off;
```

---

### `DOUxx` → `DOU`

`DOUxx` = Do-Until (Schleife läuft bis Bedingung *wahr* ist).

```rpgle
// Fixed Format
C     *INLR         DOUEQ     *ON

// Fully Free
dou *inlr = *on;   // oder: dou isLastRecord;
```

---

### `DO` (mit Counter) → `FOR`

`DO` mit Factor 1 (Anzahl) → `FOR`-Schleife.

```rpgle
// Fixed Format
C     10            DO                      idx

// Fully Free
for idx = 1 to 10;
```

```rpgle
// Mit Startwert und Ende-Variable:
C     startVal      DO        endVal        idx

// Fully Free
for idx = startVal to endVal;
```

---

### `ITER` → `ITER`

```rpgle
// Fixed Format
C                   ITER

// Fully Free
iter;
```

---

### `LEAVE` → `LEAVE`

```rpgle
// Fixed Format
C                   LEAVE

// Fully Free
leave;
```

---

### `ENDDO` → `ENDDO`

```rpgle
// Fixed Format
C                   ENDDO

// Fully Free
enddo;
```

---

## 4. SELECT / WHEN

### `CASxx` → `SELECT / WHEN`

`CASxx` ist eine ältere Form der Fallunterscheidung (vor `SELECT` eingeführt). Wird zu `SELECT/WHEN` konvertiert.

```rpgle
// Fixed Format
C     fieldX        CASEI     'A'           proc1
C     fieldX        CASNE     'A'           proc2
C                   CAS                     defaultProc
C                   ENDCS

// Fully Free
select;
  when fieldX = 'A';
    proc1();
  when fieldX <> 'A';
    proc2();
  other;
    defaultProc();
endsl;
```

---

### `SELECT` / `WHENxx` / `OTHER` / `ENDSL`

```rpgle
// Fixed Format
C                   SELECT
C     status        WHENEQ    'OPEN'
C                   ...
C     status        WHENEQ    'CLOSED'
C                   ...
C                   OTHER
C                   ...
C                   ENDSL

// Fully Free
select;
  when status = 'OPEN';
    // ...
  when status = 'CLOSED';
    // ...
  other;
    // ...
endsl;
```

`WHENxx`-Varianten folgen denselben Operator-Mappings wie `IFxx` (s. Tabelle oben).

**Fully Free bietet zusätzlich `WHEN`-Ausdrücke mit `AND`/`OR`:**
```rpgle
when status = 'OPEN' and count > 0;
```

---

## 5. Unterprogramme & Prozeduren

### `EXSR` → Prozeduraufruf

Subroutinen (`BEGSR`/`ENDSR`) werden in Fully Free zu eigenständigen Prozeduren (`dcl-proc`).

```rpgle
// Fixed Format
C                   EXSR      calcTotal

// Fully Free
calcTotal();
```

---

### `BEGSR` / `ENDSR` → `dcl-proc` / `end-proc`

```rpgle
// Fixed Format
C     calcTotal     BEGSR
C                   ...
C                   ENDSR

// Fully Free
dcl-proc calcTotal;
  // ...
end-proc;
```

**Wichtig:** Subroutinen können auf globale Programmvariablen zugreifen. Bei der Umwandlung in `dcl-proc` bleiben globale Variablen weiterhin zugänglich (Prozeduren im Hauptprogramm teilen den globalen Scope).

---

### `CALLP` → direkter Prozeduraufruf

```rpgle
// Fixed Format
C                   CALLP     myProc(param1 : param2)

// Fully Free
myProc(param1 : param2);
```

---

### `CALL` (altes Programmaufruf-Format) → `CALLP` mit Prototype

Das alte `CALL`-Format (mit `PARM`-Zeilen) sollte auf modernes `CALLP` mit Prototype umgestellt werden.

```rpgle
// Fixed Format (alt)
C                   CALL      'PGMNAME'
C                   PARM                    param1
C                   PARM                    param2

// Fully Free — bevorzugt: Prototype anlegen
dcl-pr pgmName extpgm('PGMNAME');
  param1 char(10);
  param2 packed(5:0);
end-pr;

// Dann Aufruf:
pgmName(param1 : param2);
```

**Wenn Prototype nicht möglich (Legacy-Kontext):**
```rpgle
// TODO: CALL ohne Prototype — manuelle Prüfung erforderlich
// Original: CALL 'PGMNAME' + PARM
callp pgmName(param1 : param2);   // nur wenn Prototype vorhanden
```

---

### `PARM` → Parameter in Prozedur-Signatur

`PARM`-Zeilen in `*ENTRY PLIST` definieren Eingabeparameter des Programms.

```rpgle
// Fixed Format
C     *ENTRY        PLIST
C                   PARM                    inParam1
C                   PARM                    inParam2

// Fully Free — Procedure Interface im globalen Scope:
dcl-pi *n;
  inParam1 char(10);
  inParam2 packed(5:0);
end-pi;
```

---

### `RETURN` → `RETURN`

```rpgle
// Fixed Format
C                   RETURN

// Fully Free
return;

// Mit Rückgabewert:
return returnValue;
```

---

## 6. Dateioperationen

Dateioperationen bleiben in Fully Free fast identisch — die Opcodes sind gleich, fallen aber als freie Anweisungen ohne Spaltenstruktur.

### `READ` → `READ`

```rpgle
// Fixed Format
C                   READ      fileName                       LR
// *INLR = *ON wenn EOF

// Fully Free
read fileName;
if %eof(fileName);
  isLastRecord = *on;
endif;
```

Mit Error-Extender:
```rpgle
// Fixed Format
C                   READ(E)   fileName

// Fully Free
read(e) fileName;
if %error;
  // Fehlerbehandlung
endif;
```

---

### `READP` → `READP`

```rpgle
// Fixed Format — liest vorherigen Datensatz
C                   READP     fileName                       BF
// *INBF = *ON wenn Begin-of-File

// Fully Free
readp fileName;
if %bof(fileName);
  // Anfang der Datei
endif;
```

---

### `READE` → `READE`

```rpgle
// Fixed Format — liest nächsten Datensatz mit gleichem Key
C     keyField      READE     fileName                       LR

// Fully Free
reade %kds(keyDs) fileName;
// oder mit direktem Key-Wert:
reade keyField fileName;
if %eof(fileName);
  leave;
endif;
```

---

### `READPE` → `READPE`

```rpgle
// Fixed Format — liest vorherigen Datensatz mit gleichem Key
C     keyField      READPE    fileName

// Fully Free
readpe keyField fileName;
if %bof(fileName);
  // Anfang der Datei
endif;
```

---

### `WRITE` → `WRITE`

```rpgle
// Fixed Format
C                   WRITE     recordFormat                      10
// *IN10 = *ON bei Fehler

// Fully Free
write(e) recordFormat;
if %error;
  // Fehlerbehandlung
endif;
```

---

### `UPDATE` → `UPDATE`

```rpgle
// Fixed Format
C                   UPDATE    recordFormat

// Fully Free
update recordFormat;

// Mit bestimmten Feldern:
update %fields(field1 : field2) recordFormat;
```

---

### `DELETE` → `DELETE`

```rpgle
// Fixed Format
C                   DELETE    recordFormat

// Fully Free
delete recordFormat;
```

---

### `CHAIN` → `CHAIN`

```rpgle
// Fixed Format
C     keyField      CHAIN     fileName                       NF
// *INNF = *ON wenn nicht gefunden

// Fully Free
chain keyField fileName;
if not %found(fileName);
  // Datensatz nicht gefunden
endif;
```

Mit Error-Extender:
```rpgle
chain(e) keyField fileName;
if %error;
  // Fehler
endif;
```

---

### `SETLL` / `SETGT` → `SETLL` / `SETGT`

```rpgle
// Fixed Format
C     keyField      SETLL     fileName                       NF
C     keyField      SETGT     fileName

// Fully Free
setll keyField fileName;
if %equal(fileName);
  // Exakter Treffer am Anfang
endif;

setgt keyField fileName;
```

---

### `OPEN` / `CLOSE` → `OPEN` / `CLOSE`

```rpgle
// Fixed Format
C                   OPEN      fileName
C                   CLOSE     fileName

// Fully Free
open fileName;
close fileName;
```

Voraussetzung: Datei in `dcl-f` mit `USROPN`-Keyword deklariert.

---

### `EXCEPT` → Hinweis: manuell prüfen

`EXCEPT` ruft einen Output-Spec-Eintrag auf (O-Spec). O-Spec-Konvertierung ist nicht Teil von Phase 0.

```rpgle
// Fixed Format
C                   EXCEPT    outputRecord

// Fully Free — TODO: O-Spec manuell prüfen
// TODO: EXCEPT outputRecord — O-Spec-Konvertierung erforderlich (Phase 2)
// Original-Zeile: C                   EXCEPT    outputRecord
write outputRecord;   // nur wenn outputRecord ein Datei-Format ist
```

---

## 7. String-Operationen

### `CAT` → String-Konkatenation `+`

```rpgle
// Fixed Format: Factor1 CAT Factor2:blanks → Result
C     strA          CAT       strB:0        result
// :0 = 0 Leerzeichen zwischen den Strings

C     strA          CAT       strB:1        result
// :1 = 1 Leerzeichen zwischen den Strings

// Fully Free
result = strA + strB;
result = strA + ' ' + strB;   // mit Leerzeichen
result = %trimr(strA) + strB; // ohne trailing spaces
```

---

### `SUBST` → `%SUBST`

```rpgle
// Fixed Format: SUBST Factor2:start → Result (Länge aus Result-Feld oder Factor1)
C     length        SUBST     sourceStr:startPos   result
C                   SUBST     sourceStr:startPos   result   // Länge = Rest des Strings

// Fully Free
result = %subst(sourceStr : startPos : length);
result = %subst(sourceStr : startPos);   // bis zum Ende
```

---

### `SCAN` → `%SCAN`

```rpgle
// Fixed Format: Factor1 SCAN Factor2:startPos → Result (Position)
C     'search'      SCAN      sourceStr     position

// Fully Free
position = %scan('search' : sourceStr);
position = %scan('search' : sourceStr : startPos);   // ab Position

// Prüfung ob gefunden:
if %scan('search' : sourceStr) > 0;
  // gefunden
endif;
```

---

### `CHECK` / `CHECKR` → `%CHECK` / `%CHECKR`

```rpgle
// Fixed Format: prüft auf ungültige Zeichen
C     validChars    CHECK     sourceStr     position

// Fully Free
position = %check(validChars : sourceStr);
position = %check(validChars : sourceStr : startPos);

// CHECKR (von rechts):
position = %checkr(validChars : sourceStr);
```

---

### `XLATE` → `%XLATE`

```rpgle
// Fixed Format: Zeichenübersetzung
C     'abcde'       XLATE     'ABCDE':source   result

// Fully Free
result = %xlate('abcde' : 'ABCDE' : source);
result = %xlate('abcde' : 'ABCDE' : source : startPos);
```

---

### `TRIM` / `TRIML` / `TRIMR` → `%TRIM` / `%TRIML` / `%TRIMR`

```rpgle
// Fixed Format
C                   EVAL      result = %trim(sourceStr)   // (schon Fully Free Syntax in /FREE)

// Fully Free
result = %trim(sourceStr);    // beide Seiten
result = %triml(sourceStr);   // links
result = %trimr(sourceStr);   // rechts
```

---

## 8. Sonstiges

### `GOTO` / `TAG` → manuelle Refaktorierung

`GOTO`/`TAG` können nicht mechanisch konvertiert werden — sie entsprechen keiner strukturierten Kontrollstruktur in Fully Free.

```rpgle
// Fixed Format
C                   GOTO      errorLabel
C                   ...
C     errorLabel    TAG
C                   ...

// Fully Free — NICHT automatisch konvertierbar
// TODO: GOTO/TAG — manuelle Refaktorierung erforderlich
// Mögliche Alternativen:
// - LEAVE/ITER (in Schleifen)
// - MONITOR/ON-ERROR (bei Fehlersprüngen)
// - Extraktion in Subroutinen/Prozeduren
// - Boolesche Flagge + strukturierte Bedingungen
```

**Muster: GOTO für Fehlersprung → MONITOR**
```rpgle
// Fixed Format Muster
C                   CHAIN     key           file                   10
C     *IN10         IFEQ      *ON
C                   GOTO      notFound
C                   ENDIF
C                   ... (Verarbeitung)
C     notFound      TAG

// Fully Free
chain(e) key file;
if %error or not %found(file);
  // notFound-Behandlung
else;
  // Verarbeitung
endif;
```

---

### `DUMP` → `DUMP`

```rpgle
// Fixed Format
C                   DUMP

// Fully Free
dump;
dump(a);   // mit aktiviertem Dump auch ohne Fehler
```

---

### `DEBUG` → entfernen

`DEBUG` (veraltet) hat in modernem RPG keine Funktion mehr.

```rpgle
// Fixed Format
C                   DEBUG

// Fully Free — einfach weglassen, kein Äquivalent nötig
// (war nur für altes Debug-System relevant)
```

---

### `TIME` → `%TIME()` / `%DATE()` / `%TIMESTAMP()`

```rpgle
// Fixed Format: setzt Result auf aktuelle Zeit/Datum
C                   TIME      currentTime   // 12-stellige numerische HHMMSSMMDDYY

// Fully Free — je nach gewünschtem Typ:
currentTime      = %time();        // TIME-Typ (hhmmss)
currentDate      = %date();        // DATE-Typ
currentTimestamp = %timestamp();   // TIMESTAMP-Typ
```

---

### `TEST` → `TEST`-Opcode oder `MONITOR`-Block

`TEST` validiert Datum/Zeit-Felder auf Gültigkeit.

```rpgle
// Fixed Format
C     *DMY          TEST(E)   dateField                      50
// *IN50 = *ON wenn ungültig

// Fully Free
test(e) *dmy dateField;
if %error;
  // ungültiges Datum
endif;

// Moderner Ansatz: MONITOR-Block
monitor;
  testDate = %date(dateField : *dmy);
on-error;
  // ungültiges Datum
endmon;
```

---

### `MONITOR` / `ON-ERROR` / `ENDMON`

Bereits eine strukturierte Konstruktion — direkt in Fully Free übertragbar.

```rpgle
// Fixed Format (/FREE-Block innerhalb Fixed Format)
/FREE
  monitor;
    chain key file;
  on-error;
    errMsg = 'Fehler beim Lesen';
  endmon;
/END-FREE

// Fully Free (identisch, kein /FREE nötig)
monitor;
  chain key file;
on-error;
  errMsg = 'Fehler beim Lesen';
endmon;
```

---

### Resulting Indicators → `%ERROR`, `%FOUND`, boolesche Variable

Resulting Indicators (Spalten 54–59) müssen elimiert werden.

| Kontext | Resulting Indicator | Fully Free Ersatz |
|---------|--------------------|--------------------|
| Datei-READ | LR-Indicator (EOF) | `%eof(fileName)` |
| CHAIN/READ | NF-Indicator (Not Found) | `not %found(fileName)` |
| SETLL | EQ-Indicator (Equal) | `%equal(fileName)` |
| COMP | HI/LO/EQ | Boolesche Variable oder direktes `IF` |
| Dateioperationen mit `(E)` | Fehler-Indicator | `%error` / `%status` |
| `*INLR` | Last Record | `*inlr = *on` oder `isLastRecord = *on` |

**Conditioning Indicators** (Spalten 7–8) steuern, ob eine C-Spec-Zeile ausgeführt wird:

```rpgle
// Fixed Format
C     10            READ      fileName                       LR
// Zeile wird nur ausgeführt wenn *IN10 = *ON

// Fully Free
if *in10 = *on;   // oder: if isCondition;
  read fileName;
  if %eof(fileName);
    *inlr = *on;
  endif;
endif;
```

---

## 9. Vollständiges Konvertierungsbeispiel

### Fixed Format Original

```rpgle
     H DFTACTGRP(*NO) ACTGRP(*CALLER)
     FCUSTMST   IF   E           K DISK
     D custNo          S              7P 0
     D custName        S             50A
     D totalAmt        S             11P 2
     D counter         S              5P 0
     D isFound         S               N
      *
     C     *ENTRY        PLIST
     C                   PARM                    pCustNo           7P 0
      *
     C                   Z-ADD     0             totalAmt
     C                   Z-ADD     0             counter
      *
     C     pCustNo       CHAIN     CUSTMST                        50
     C     *IN50         IFEQ      *ON
     C                   EXSR      notFoundRtn
     C                   RETURN
     C                   ENDIF
      *
     C                   EVAL      custNo   = CUSNO
     C                   EVAL      custName = CUSNAM
     C                   ADD       CUSAMT        totalAmt
     C                   ADD       1             counter
      *
     C                   SETON                                        LR
      *
     C     notFoundRtn   BEGSR
     C                   EVAL      custName = 'NOT FOUND'
     C                   ENDSR
```

### Fully Free Konvertierung

```rpgle
**FREE
ctl-opt dftactgrp(*no) actgrp(*caller);

dcl-f custmst if e k disk;

dcl-s custNo    packed(7:0);
dcl-s custName  char(50);
dcl-s totalAmt  packed(11:2);
dcl-s counter   packed(5:0);

dcl-pi *n;
  pCustNo packed(7:0);
end-pi;

totalAmt = 0;
counter  = 0;

chain pCustNo custmst;
if not %found(custmst);
  notFoundRtn();
  return;
endif;

custNo   = cusno;
custName = cusnam;
totalAmt += cusamt;
counter  += 1;

*inlr = *on;
return;

dcl-proc notFoundRtn;
  custName = 'NOT FOUND';
end-proc;
```

**Konvertierungsschritte:**
1. `H DFTACTGRP(*NO)` → `ctl-opt dftactgrp(*no);`
2. `FCUSTMST IF E K DISK` → `dcl-f custmst if e k disk;`
3. D-Spec Variablen → `dcl-s` mit Typmapping
4. `*ENTRY PLIST` / `PARM` → `dcl-pi *n; ... end-pi;`
5. `Z-ADD 0 var` → `var = 0;`
6. `CHAIN ... *IN50` → `chain ... ; if not %found(...);`
7. `EXSR notFoundRtn` → `notFoundRtn();`
8. `EVAL` → direkte Zuweisung
9. `ADD` → `+=`
10. `SETON LR` → `*inlr = *on;`
11. `BEGSR/ENDSR` → `dcl-proc/end-proc`
