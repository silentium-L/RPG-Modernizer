# D-Spec → dcl-s / dcl-ds / dcl-c / dcl-pr / dcl-pi Konvertierungsregeln

## Einleitung

Die D-Spec (Definition Specification) ist die Deklarationszeile des RPG IV Fixed Formats. Sie definiert alle Variablen, Konstanten, Data Structures, Prototypen und Procedure Interfaces. Im Fully Free Format werden diese durch dedizierte `dcl-*`-Anweisungen ersetzt.

**D-Spec Grundstruktur (Spalten 7–80):**

```
Spalte  6     : 'D'
Spalten  7–21 : Name (15 Zeichen)
Spalte  22    : External Description (E / leer)
Spalte  23    : Data Structure Type (S = Standalone, DS, C, PR, PI)
Spalten 24–25 : To / Length (bei DS: DS-Typ oder Subfeld-Endpos)
Spalten 26–32 : From Position (Subfelder in DS)
Spalten 33–39 : To Position / Length
Spalte  40    : Decimal Positions
Spalten 41–42 : Data Type Kürzel
Spalten 43–80 : Keywords
```

**D-Spec-Typen (Spalte 23):**

| Zeichen | Bedeutung |
|---------|-----------|
| `S`     | Standalone Variable |
| `DS`    | Data Structure |
| `C`     | Named Constant |
| `PR`    | Prototype |
| `PI`    | Procedure Interface |
| leer    | Subfeld einer Data Structure |

---

## Datentyp-Mapping-Tabelle

Alle Fixed-Format-Typkürzel (Spalte 41–42) und ihre Fully-Free-Äquivalente:

| Fixed Kürzel | Fixed Name          | Fully Free Typ | Beispiel Fully Free       |
|--------------|---------------------|----------------|---------------------------|
| `A`          | Character           | `CHAR(n)`      | `dcl-s name CHAR(30)`     |
| `P`          | Packed Decimal      | `PACKED(p:d)`  | `dcl-s amount PACKED(9:2)`|
| `S`          | Zoned Decimal       | `ZONED(p:d)`   | `dcl-s zoned ZONED(7:2)`  |
| `B`          | Binary              | `INT(n)` / `UNS(n)` | `dcl-s bin INT(5)`  |
| `I`          | Integer (signed)    | `INT(n)`       | `dcl-s cnt INT(10)`       |
| `U`          | Unsigned Integer    | `UNS(n)`       | `dcl-s cnt UNS(10)`       |
| `F`          | Float               | `FLOAT(n)`     | `dcl-s flt FLOAT(8)`      |
| `D`          | Date                | `DATE(*ISO)` o. Format | `dcl-s dt DATE`  |
| `T`          | Time                | `TIME`         | `dcl-s tm TIME`           |
| `Z`          | Timestamp           | `TIMESTAMP`    | `dcl-s ts TIMESTAMP`      |
| `N`          | Indicator           | `IND`          | `dcl-s found IND`         |
| `*`          | Pointer             | `POINTER`      | `dcl-s ptr POINTER`       |
| `O`          | Object              | `OBJECT`       | `dcl-s obj OBJECT`        |
| leer (num.)  | Packed (default)    | `PACKED(p:d)`  | Länge + Dezimal aus Spalten|
| leer (char.) | Character (default) | `CHAR(n)`      | nur Länge in Spalten       |

**Hinweis Integer-Breiten:**

| `INT`/`UNS` n | Wertebereich (signed) |
|---------------|-----------------------|
| `INT(3)`      | -128 … 127            |
| `INT(5)`      | -32.768 … 32.767      |
| `INT(10)`     | -2.147.483.648 … +2.147.483.647 |
| `INT(20)`     | 64-Bit-Bereich        |

---

## Standalone Variable (`S`) → `dcl-s`

**Fixed Format:**
```
DName+++++++++++ETDsFrom+++To/L+++IDc.Keywords++++++++++++++++++++
D varName         S             10A
D amount          S              9P 2
D counter         S             10I 0
D found           S              1N
```

**Fully Free:**
```rpgle
dcl-s varName   CHAR(10);
dcl-s amount    PACKED(9:2);
dcl-s counter   INT(10);
dcl-s found     IND;
```

**Konvertierungsregel:**
1. Name aus Spalten 7–21 übernehmen
2. Typ aus Spalte 41–42 + Länge (Spalten 33–39) + Dezimal (Spalte 40) ableiten
3. Keywords aus Spalten 43–80 direkt übernehmen (Syntax ggf. anpassen)
4. `;` am Ende hinzufügen

---

## Keyword-Mapping-Tabelle (D-Spec → dcl-s/dcl-ds)

| D-Spec Keyword | Fully Free Äquivalent | Hinweis |
|----------------|----------------------|---------|
| `INZ(*ZEROS)`  | `INZ(*ZEROS)`        | Unverändert |
| `INZ(*BLANKS)` | `INZ(*BLANKS)`       | Unverändert |
| `INZ(*ON)`     | `INZ(*ON)`           | Unverändert |
| `INZ(*OFF)`    | `INZ(*OFF)`          | Unverändert |
| `INZ(wert)`    | `INZ(wert)`          | Unverändert |
| `CONST`        | `CONST`              | Unverändert (für Parameter) |
| `VALUE`        | `VALUE`              | Unverändert (für Parameter) |
| `OPTIONS(*NOPASS)` | `OPTIONS(*NOPASS)` | Unverändert |
| `OPTIONS(*OMIT)` | `OPTIONS(*OMIT)`  | Unverändert |
| `DIM(n)`       | `DIM(n)`             | Unverändert |
| `OCCURS(n)`    | `DIM(n)`             | **OCCURS wird zu DIM** |
| `BASED(ptr)`   | `BASED(ptr)`         | Unverändert |
| `LIKE(var)`    | `LIKE(var)`          | Unverändert |
| `LIKEDS(ds)`   | `LIKEDS(ds)`         | Unverändert |
| `LIKEREC(rec)` | `LIKEREC(rec:*ALL)` | Ggf. Qualifikator ergänzen |
| `EXTNAME(file)`| `EXTNAME(file)`      | Unverändert |
| `QUALIFIED`    | `QUALIFIED`          | Unverändert |
| `PREFIX(str)`  | `PREFIX(str)`        | Unverändert |
| `DATFMT(*ISO)` | `DATFMT(*ISO)`       | Unverändert |
| `TIMFMT(*ISO)` | `TIMFMT(*ISO)`       | Unverändert |
| `VARYING`      | `VARYING`            | Unverändert |
| `ALIGN`        | `ALIGN`              | Unverändert |
| `NOOPT`        | `NOOPT`              | Unverändert |
| `PERRCD(n)`    | entfällt / prüfen   | Nur bei Multioccurrence DS |
| `TOFILE(name)` | entfällt             | Nur bei Input Buffer DS |

---

## Data Structure (`DS`) → `dcl-ds`

### Einfache Data Structure

**Fixed Format:**
```
D myDs            DS
D  field1                  1     10A
D  field2                 11     15P 2
D  field3                 16     25A
```

**Fully Free:**
```rpgle
dcl-ds myDs;
  field1  CHAR(10);
  field2  PACKED(5:2);
  field3  CHAR(10);
end-ds myDs;
```

**Hinweis:** Im Fixed Format werden Subfelder durch absolute Positionen (From/To) angegeben. Im Fully Free Format werden sie sequentiell deklariert — der Compiler berechnet die Positionen automatisch aus den Längen.

### Data Structure mit LIKEDS

**Fixed Format:**
```
D orderDs         DS                  LIKEDS(templateDs)
```

**Fully Free:**
```rpgle
dcl-ds orderDs LIKEDS(templateDs);
end-ds orderDs;
```

### Data Structure mit EXTNAME

**Fixed Format:**
```
D fileDs          DS                  EXTNAME(MYFILE)
```

**Fully Free:**
```rpgle
dcl-ds fileDs EXTNAME('MYFILE') END-DS(*AUTO) QUALIFIED;
end-ds;
```

### Data Structure mit QUALIFIED

```rpgle
dcl-ds address QUALIFIED;
  street  CHAR(30);
  city    CHAR(20);
  zip     CHAR(5);
end-ds;
// Zugriff: address.street := 'Hauptstr.';
```

### Data Structure Array (DIM)

**Fixed Format:**
```
D items           DS                  OCCURS(10)
D  itemNo                  1      5P 0
D  itemName                6     35A
```

**Fully Free:**
```rpgle
dcl-ds items DIM(10) QUALIFIED;
  itemNo   PACKED(5:0);
  itemName CHAR(30);
end-ds;
```

---

## Named Constant (`C`) → `dcl-c`

**Fixed Format:**
```
D PI              C                   CONST(3.14159)
D MAXROWS         C                   CONST(9999)
D COMPANY         C                   CONST('ACME Corp.')
```

**Fully Free:**
```rpgle
dcl-c PI      3.14159;
dcl-c MAXROWS 9999;
dcl-c COMPANY 'ACME Corp.';
```

**Hinweis:** Das `CONST`-Keyword entfällt im Fully Free Format bei `dcl-c` — der Wert wird direkt angegeben.

---

## Prototype (`PR`) → `dcl-pr`

**Fixed Format:**
```
D calcTotal       PR            10P 2
D  qty                          5P 0 VALUE
D  price                        9P 2 VALUE
```

**Fully Free:**
```rpgle
dcl-pr calcTotal EXTPGM('CALCTOTAL');
  qty   PACKED(5:0)  VALUE;
  price PACKED(9:2)  VALUE;
end-pr;
```

**Oder für interne Prozedur (kein EXTPGM):**
```rpgle
dcl-pr calcTotal PACKED(10:2);
  qty   PACKED(5:0)  VALUE;
  price PACKED(9:2)  VALUE;
end-pr;
```

**Konvertierungsregel:**
- Erster Eintrag (`PR`) → `dcl-pr name Rückgabetyp;` + `end-pr;`
- Nachfolgende Subfelder ohne Typ (Zeile mit leerem Typ, Name in 7–21) → Parameter-Zeilen
- `EXTPGM` / `EXTPROC` Keyword aus D-Spec übernehmen

---

## Procedure Interface (`PI`) → `dcl-pi`

**Fixed Format (innerhalb eines P-Specs):**
```
D procName        PI            10P 2
D  qty                          5P 0 VALUE
D  price                        9P 2 VALUE
```

**Fully Free:**
```rpgle
dcl-pi procName PACKED(10:2);
  qty   PACKED(5:0)  VALUE;
  price PACKED(9:2)  VALUE;
end-pi;
```

**Hinweis:** `dcl-pi` steht **innerhalb** einer Prozedur (nach `dcl-proc`) und entspricht der Signatur. `dcl-pr` steht **außerhalb** als Prototyp.

---

## Sonderfälle

### BASED-Variable

Eine `BASED`-Variable zeigt auf einen Speicherbereich, auf den ein Pointer verweist.

**Fixed Format:**
```
D dynString       S            100A   BASED(ptrString)
D ptrString       S               *
```

**Fully Free:**
```rpgle
dcl-s ptrString POINTER;
dcl-s dynString CHAR(100) BASED(ptrString);
```

**Reihenfolge:** Der Pointer muss vor der BASED-Variable deklariert sein (oder zumindest bekannt sein). Im Fixed Format war die Reihenfolge flexibler.

### OCCURS → DIM

`OCCURS` aus dem alten RPG III / frühen RPG IV wird im Fully Free Format durch `DIM` ersetzt.

**Fixed Format:**
```
D tableDS         DS                  OCCURS(50)
D  tblKey                  1     10A
D  tblVal                 11     30A
```

**Fully Free:**
```rpgle
dcl-ds tableDS DIM(50) QUALIFIED;
  tblKey CHAR(10);
  tblVal CHAR(20);
end-ds;
// Zugriff: tableDS(i).tblKey
```

### Arrays im Fixed Format → DIM

**Fixed Format:**
```
D myArray         S             10A   DIM(100)
D numArray        S              5P 0 DIM(50)
```

**Fully Free:**
```rpgle
dcl-s myArray  CHAR(10)    DIM(100);
dcl-s numArray PACKED(5:0) DIM(50);
```

### Subfelder mit Overlay

Im Fixed Format können Subfelder durch Positions-Überschneidung (OVERLAY-ähnliches Verhalten) Aliases für dieselben Bytes definieren.

**Fixed Format:**
```
D dateDs          DS
D  dateChar                1      8A
D  dateYear                1      4A
D  dateMon                 5      6A
D  dateDay                 7      8A
```

**Fully Free:**
```rpgle
dcl-ds dateDs;
  dateChar CHAR(8);
  dateYear CHAR(4) OVERLAY(dateChar:1);
  dateMon  CHAR(2) OVERLAY(dateChar:5);
  dateDay  CHAR(2) OVERLAY(dateChar:7);
end-ds;
```

**Hinweis:** Das `OVERLAY`-Keyword ist im Fully Free Format verfügbar und löst Positions-Überschneidungen explizit auf. Der Konverter setzt einen TODO-Marker, wenn Positionen überlappen, damit ein Mensch die Semantik prüft.

---

## Beispiele

### Beispiel 1: Einfache Standalone-Variablen

**Fixed Format:**
```
DName+++++++++++ETDsFrom+++To/L+++IDc.Keywords++++++++++++++++++++
D custName        S             30A
D custAge         S              3P 0
D custBalance     S             11P 2   INZ(*ZEROS)
D isActive        S              1N     INZ(*ON)
D custArr         S             20A     DIM(10)
```

**Fully Free:**
```rpgle
dcl-s custName    CHAR(30);
dcl-s custAge     PACKED(3:0);
dcl-s custBalance PACKED(11:2) INZ(*ZEROS);
dcl-s isActive    IND          INZ(*ON);
dcl-s custArr     CHAR(20)     DIM(10);
```

---

### Beispiel 2: Komplexe Data Structure mit Subfeldern

**Fixed Format:**
```
D address         DS
D  street                  1     30A
D  city                   31     50A
D  country                51     52A
D  zip                    53     57A
D  zipNum                 53     57P 0
```

**Fully Free:**
```rpgle
dcl-ds address;
  street  CHAR(30);
  city    CHAR(20);
  country CHAR(2);
  zip     CHAR(5);
  zipNum  PACKED(5:0) OVERLAY(zip);  // TODO: Overlay prüfen — zipNum und zip teilen dieselben Bytes
end-ds;
```

---

### Beispiel 3: Prototype mit Parametern

**Fixed Format:**
```
D formatDate      PR            10A
D  inDate                        D   VALUE DATFMT(*ISO)
D  inFormat                      4A  VALUE OPTIONS(*NOPASS)
```

**Fully Free:**
```rpgle
dcl-pr formatDate CHAR(10);
  inDate    DATE         VALUE DATFMT(*ISO);
  inFormat  CHAR(4)      VALUE OPTIONS(*NOPASS);
end-pr;
```

---

## Checkliste für den Konverter

- [ ] Typ-Kürzel (Spalte 41–42) korrekt gemappt
- [ ] Länge und Dezimalstellen aus Spalten 33–40 übernommen
- [ ] Keywords aus Spalten 43–80 übertragen
- [ ] `OCCURS` → `DIM` ersetzt
- [ ] Positions-Überschneidungen bei DS-Subfeldern erkannt → `OVERLAY` oder TODO-Marker
- [ ] `BASED`-Variablen: Pointer-Deklaration vor der BASED-Variable
- [ ] Leere Typ-Spalte bei DS-Subfeldern korrekt als Subfeld behandelt (nicht als neue dcl-s)
- [ ] PR-Subfelder (Parameter-Zeilen mit leerem Typ 'S'/'DS') korrekt als Parameter erkannt
