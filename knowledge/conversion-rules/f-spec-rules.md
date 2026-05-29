# F-Spec → `dcl-f` Konvertierungsregeln

> Abhängigkeiten: [`rpgiv-fixed-syntax.md`](../rpg-versions/rpgiv-fixed-syntax.md) · [`fully-free-syntax.md`](../rpg-versions/fully-free-syntax.md)

---

## Einleitung

Die **F-Spec (File Specification)** ist Spalte-6-Typ `F` im Fixed Format RPG IV. Sie deklariert alle Dateien, die ein Programm verwendet — Eingabe-, Ausgabe-, Update- und kombinierte Dateien.

Im Fully Free Format ersetzt `dcl-f` die F-Spec vollständig. Die `**FREE`-Direktive macht F-Specs ungültig; jede Datei muss neu als `dcl-f`-Statement deklariert werden.

**Grundprinzip der Konvertierung:**

```
F-Spec-Zeile (fixe Spalten + Keywords-Bereich)
→ dcl-f DATEINAME GERÄT [KEYED] [USAGE(*...)] [weitere Keywords];
```

---

## F-Spec Spaltenstruktur (Kurzreferenz)

| Spalten | Inhalt | Wichtig für Konvertierung |
|---------|--------|--------------------------|
| 6 | `F` (Spec-Typ) | entfällt |
| 7–16 | Dateiname | → `dcl-f DATEINAME` |
| 17 | Dateityp (I/O/U/C) | → `usage(*)` Keyword |
| 18 | Dateibezeichner (E/S/R/T/blank) | → KEYED, Struktur |
| 19 | End-of-File-Marke (E/blank) | → entfällt (wird durch `%EOF` ersetzt) |
| 20 | Sequenz (A/D/blank) | → entfällt |
| 21 | Dateiformat (F/E/blank) | → entfällt (bei `dcl-f` irrelevant) |
| 22–27 | Satzlänge (bei programmbeschriebenen Dateien) | → entfällt (nur in Ausnahmen) |
| 28 | Limits (L/blank) | → entfällt |
| 29–33 | Schlüssellänge / Schlüsselfeld | → KEYLOC (nur programmbeschrieben) |
| 34 | Alternative Sequenz (A/blank) | → entfällt |
| 35–42 | Datei-Addition / Fortsetzungsbereich | → Keywords-Bereich |
| 43 | Record-Adress-Typ | → selten verwendet |
| 44 | Sequenz (U/A/D) | → entfällt |
| 45–51 | Reserviert / Gerätespezifisch | → entfällt |
| 52–57 | Gerätetyp (DISK, PRINTER, WORKSTN…) | → Geräteangabe in `dcl-f` |
| 58–65 | Reserviert | → entfällt |
| 66–68 | Indikatoren | → entfällt (durch `INFDS` ersetzen) |
| 69–80 | Keywords-Bereich | → direkt übertragen / umwandeln |

---

## Dateityp-Mapping (Spalte 17)

Die wichtigste Unterscheidung: Welche Zugriffsart hat die Datei?

| F-Spec Typ | Bedeutung | `dcl-f` Entsprechung |
|-----------|-----------|----------------------|
| `I` | Input (nur Lesen) | `dcl-f NAME GERÄT;` *(usage(*input) ist Standard, entfällt)* |
| `O` | Output (nur Schreiben) | `dcl-f NAME GERÄT usage(*output);` |
| `U` | Update (Lesen + Ändern) | `dcl-f NAME GERÄT usage(*update);` |
| `C` | Combined (Lesen + Schreiben, z. B. WORKSTN) | `dcl-f NAME GERÄT usage(*input:*output);` |

> **Hinweis:** `usage(*input)` muss nicht explizit angegeben werden — es ist der Standardwert. Bei `I`-Dateien das Keyword weglassen.

---

## Dateibezeichner-Mapping (Spalte 18)

| F-Spec Bezeichner | Bedeutung | Konvertierungshinweis |
|-------------------|-----------|----------------------|
| blank | Programmbeschriebene Vollverarbeitung | `dcl-f` ohne `EXTDESC`, Satzlänge direkt im Code |
| `E` | Extern beschrieben (DDS/SQL) | Standard — `dcl-f` ohne besondere Kennzeichnung |
| `S` | Programmbeschrieben sequentiell (veraltet) | Wie blank behandeln |
| `R` | Record-Adress-Datei (veraltet) | → TODO-Marker, manuell prüfen |
| `T` | Array/Tabellendatei (veraltet) | → TODO-Marker, manuell prüfen |

---

## Gerätetyp-Mapping (Spalten 52–57)

| F-Spec Gerät | `dcl-f` Gerät | Hinweis |
|-------------|---------------|---------|
| `DISK` | `disk` | Standardfall |
| `PRINTER` | `printer` | Druckdatei → Sonderfälle beachten |
| `WORKSTN` | `workstn` | Display-File / interaktiv |
| `SPECIAL` | `special(*pgm:HandlerProc)` | Eigener Handler erforderlich |
| `SEQ` | `seq` | Sequenzdatei (z. B. Pipe) |

---

## Keyword-Mapping-Tabelle

Alle gängigen F-Spec-Keywords mit ihrer `dcl-f`-Entsprechung:

| F-Spec Keyword | `dcl-f` Keyword | Kommentar |
|----------------|-----------------|-----------|
| `BLOCK(*YES)` / `BLOCK(*NO)` | `block(*yes)` / `block(*no)` | Puffergröße; normalerweise weglassen |
| `COMMIT` | `commit` | 1:1 übernehmen |
| `DATFMT(fmt)` | `datfmt(fmt)` | 1:1 übernehmen |
| `DEVID(varName)` | `devid(varName)` | 1:1 übernehmen |
| `EXTDESC('name')` | `extdesc('name')` | Extern beschriebene Datei mit abweichendem Namen |
| `EXTFILE(var)` | `extfile(var)` | Dateiname zur Laufzeit |
| `EXTIND(*INxx)` | `extind(*inXX)` | Indikator → wenn möglich durch boolesche Variable ersetzen (→ [`indicator-rules.md`](indicator-rules.md)) |
| `EXTMBR(mbr)` | `extmbr(mbr)` | 1:1 übernehmen |
| `FORMLEN(n)` | `formlen(n)` | Nur PRINTER-Dateien |
| `FORMOFL(n)` | `formofl(n)` | Nur PRINTER-Dateien |
| `IGNORE(recfmt)` | `ignore(recfmt)` | 1:1 übernehmen |
| `INCLUDE(recfmt)` | `include(recfmt)` | 1:1 übernehmen |
| `INFDS(dsName)` | `infds(dsName)` | Datei-Informations-Datenstruktur |
| `INFSR(srName)` | `infsr(*pssr)` oder `infsr(procName)` | Subroutine → muss zu `dcl-proc` werden |
| `KEYED` | `keyed` | 1:1 übernehmen |
| `KEYLOC(n)` | `keyloc(n)` | Nur programmbeschriebene Schlüsseldateien |
| `LIKEFILE(dateiname)` | `likefile(dateiname)` | 1:1 übernehmen |
| `MAXDEV(*ONLY)` / `MAXDEV(*FILE)` | `maxdev(*only)` / `maxdev(*file)` | Nur WORKSTN |
| `OFLIND(*INxx)` | `oflind(überlaufVar)` | Indikator → boolesche Variable (`dcl-s isOfl ind`) |
| `PREFIX(prfx:n)` | `prefix(prfx:n)` | 1:1 übernehmen |
| `PRTCTL(dsName)` | `prtctl(dsName)` | Nur PRINTER-Dateien |
| `QUALIFIED` | `qualified` | Feldnamen nur über `Dateiname.Feld` erreichbar |
| `RECNO(var)` | `recno(var)` | Relative Datensatznummer |
| `RENAME(alt:neu)` | `rename(alt:neu)` | 1:1 übernehmen |
| `SAVEDS(dsName)` | `saveds(dsName)` | 1:1 übernehmen |
| `SAVEIND(n)` | `saveind(n)` | 1:1 übernehmen |
| `SFILE(recfmt:rrn)` | `sfile(recfmt:rrn)` | Subfile; nur WORKSTN |
| `SLN(n)` | `sln(n)` | Startzeile; nur WORKSTN |
| `TEMPLATE` | `template` | 1:1 übernehmen |
| `TIMFMT(fmt)` | `timfmt(fmt)` | 1:1 übernehmen |
| `USROPN` | `usropn` | 1:1 übernehmen; Datei wird manuell geöffnet |

> **Nicht übernehmbare Angaben:**
> - Satzlänge (Spalten 22–27): entfällt bei extern beschriebenen Dateien
> - Schlüssellänge (Spalten 29–33): entfällt bei `KEYED` (Key kommt aus der DDS)
> - Positioning-Indikatoren in Spalten 66–68: durch `INFDS` oder `%STATUS` ersetzen

---

## Sonderfall: WORKSTN-Dateien (Display Files)

WORKSTN-Dateien sind interaktive Bildschirmdateien. Sie sind stets `Combined` (Typ `C`).

**Fixed Format:**
```rpgle
FKUNDANFR  CF   E             WORKSTN
```

**Fully Free:**
```rpgle
dcl-f KUNDANFR workstn usage(*input:*output);
```

Besonderheiten:
- `usage(*input:*output)` immer explizit angeben (CF = Combined)
- Subfile-Deklaration (`SFILE`) bleibt erhalten: `sfile(SFL01:rrn)`
- `MAXDEV(*ONLY)` ist Standard für einfache interaktive Programme
- Display-File-Indikatoren (`*INKx`, `*INOA`–`*INOW`) werden **nicht** in Phase 0 behandelt → TODO-Marker setzen

---

## Sonderfall: PRINTER-Dateien (Druckdateien)

**Fixed Format:**
```rpgle
FQPRINT    O    F  132        PRINTER OFLIND(*IN86)
```

**Fully Free:**
```rpgle
dcl-s isOfl ind;
dcl-f QPRINT printer formlen(66) oflind(isOfl) usage(*output);
```

Besonderheiten:
- `OFLIND(*INxx)` → Indikator durch boolesche Variable ersetzen (`dcl-s isOfl ind`)
- `FORMLEN(n)`: Formularlänge übernehmen
- `PRTCTL(ds)`: Druckersteuerungs-DS übernehmen
- `EXCEPT`-Opcode in der C-Spec → TODO-Marker (kein direktes Äquivalent in Fully Free; stattdessen `write RECFMT`)

---

## Sonderfall: SPECIAL-Dateien

SPECIAL-Dateien haben einen eigenen Verarbeitungs-Handler.

**Fixed Format:**
```rpgle
FMYFILE    IP   F  80         SPECIAL     PLIST(MYLIST)
```

**Fully Free:**
```rpgle
dcl-f MYFILE special(*pgm:MYHANDLER) usage(*input);
```

> **Achtung:** `PLIST` aus der F-Spec ist in Fully Free nicht erlaubt. Die Übergabe an den Handler erfolgt über `dcl-pr` / `dcl-pi`. → TODO-Marker setzen und manuell anpassen.

---

## Sonderfall: Update-Datei (Dateityp `U`)

**Fixed Format:**
```rpgle
FKUNDSTMM  UF   E           K DISK
```

**Fully Free:**
```rpgle
dcl-f KUNDSTMM disk keyed usage(*update);
```

Besonderheiten:
- `U` → `usage(*update)` — ermöglicht `READ`, `CHAIN`, `UPDATE`, `DELETE`
- `K` vor dem Gerät (Spalte 34–35 ca.) → `KEYED`
- `A` hinter dem Dateinamen (Spalte 43) → Datensätze hinzufügen erlaubt → kein eigenes Keyword in `dcl-f`, `WRITE` ist grundsätzlich erlaubt bei Update-Dateien

---

## Mehrere F-Spec-Fortsetzungszeilen

Im Fixed Format können F-Spec-Keywords über mehrere Zeilen verteilt sein (Spalte 6 = `F`, Rest leer, nur Keywords-Bereich genutzt):

**Fixed Format:**
```rpgle
FKUNDSTMM  UF   E           K DISK
F                                     INFDS(FILEINF)
F                                     PREFIX(KD:2)
```

**Fully Free (eine Zeile):**
```rpgle
dcl-f KUNDSTMM disk keyed usage(*update) infds(FILEINF) prefix(KD:2);
```

Regel: Alle Keywords aller Fortsetzungszeilen einer Datei werden in **eine** `dcl-f`-Zeile (oder mit Zeilenumbruch) zusammengeführt.

---

## Vollständige Beispiele

### Beispiel 1: Einfache Eingabedatei (DISK, extern beschrieben)

**Fixed Format:**
```rpgle
FARTIKEL   IF   E           K DISK
```

**Fully Free:**
```rpgle
dcl-f ARTIKEL disk keyed;
```

Erklärung: `I` → usage(*input) = Standard, entfällt. `E` = extern beschrieben = Standard. `K` → `keyed`.

---

### Beispiel 2: Update-Datei mit INFDS und PREFIX

**Fixed Format:**
```rpgle
FKUNDENF   UF   E           K DISK
F                                     INFDS(KundenInfo)
F                                     PREFIX(KD:2)
F                                     USROPN
```

**Fully Free:**
```rpgle
dcl-f KUNDENF disk keyed usage(*update)
               infds(KundenInfo)
               prefix(KD:2)
               usropn;
```

---

### Beispiel 3: Druckdatei mit Überlaufindikator

**Fixed Format:**
```rpgle
FQPRINT    O    F  132        PRINTER OFLIND(*IN86)
```

**Vorbereitende Deklaration (D-Spec → dcl-s):**
```rpgle
dcl-s isOverflow ind;
```

**Fully Free:**
```rpgle
dcl-f QPRINT printer usage(*output) oflind(isOverflow);
```

---

### Beispiel 4: WORKSTN-Datei mit Subfile

**Fixed Format:**
```rpgle
FMYDSPLAY  CF   E             WORKSTN
F                                     SFILE(SFLRCD:rrn)
F                                     INFDS(WsInfo)
```

**Fully Free:**
```rpgle
dcl-f MYDSPLAY workstn usage(*input:*output)
               sfile(SFLRCD:rrn)
               infds(WsInfo);
```

---

## Konvertierungs-Checkliste

Für jede F-Spec-Zeile beim Konvertieren:

- [ ] Dateinamen übernehmen
- [ ] Gerätetyp bestimmen (`disk` / `printer` / `workstn` / `special` / `seq`)
- [ ] Dateityp prüfen: `I` → kein usage; `O` → `usage(*output)`; `U` → `usage(*update)`; `C` → `usage(*input:*output)`
- [ ] `E`-Bezeichner (extern beschrieben)? → kein besonderes Keyword nötig
- [ ] `K` vorhanden? → `keyed`
- [ ] Alle Fortsetzungszeilen-Keywords sammeln und zusammenführen
- [ ] `OFLIND(*INxx)` → neue `dcl-s isXxx ind` + Keyword `oflind(isXxx)`
- [ ] `EXTIND(*INxx)` → neue `dcl-s isXxx ind` + Keyword `extind(isXxx)`
- [ ] `INFSR(srName)` → TODO-Marker: Subroutine muss zu Prozedur werden
- [ ] `R`/`T`-Dateibezeichner → TODO-Marker
- [ ] `SPECIAL` mit `PLIST` → TODO-Marker
