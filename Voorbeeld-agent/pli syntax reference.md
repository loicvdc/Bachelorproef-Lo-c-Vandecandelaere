# PL/I Syntax Reference

Referentiebestand met de PL/I-syntax en mainframe-conventies die relevant zijn voor de migratie-agent.
Dit is bewust **beperkt tot wat nodig is** voor het vertalen van CA Gen Action Diagrams — geen volledige
taalspecificatie. Voor de exacte mapping per CA Gen-construct: zie `construct-mapping.md`.

## 1. Declaraties (DCL)

### Enkelvoudige variabele

```pli
DCL VARNAAM FIXED BIN(31);
DCL TEKSTVELD CHAR(20);
```

### Geneste structuur (1/2-niveau)

```pli
DCL 1 KLANT_STRUCT,
      2 KLANT_ID       FIXED BIN(31),
      2 KLANT_NAAM     CHAR(40),
      2 KLANT_STATUS   CHAR(1);
```

Structuren worden gebruikt om Views/Groups uit het datamodel te representeren (zie
`data-structure-mapping.md` voor de exacte veldmapping).

### Veelgebruikte types binnen dit project (PLAPO-conventie)

| CA Gen-type | PL/I-declaratie |
|---|---|
| Numeriek (integer) | `FIXED BIN(31)` |
| Numeriek (decimaal) | `FIXED DEC(p,s)` |
| Tekst | `CHAR(n)` (vaste lengte gelijk aan CA Gen-veldlengte) |
| Datum | `CHAR(10)` of project-specifiek datumformaat — zie `data-structure-mapping.md` |

### Arrays (voor Group Views)

```pli
DCL 1 REGEL_TABEL(50),
      2 REGEL_ID     FIXED BIN(31),
      2 REGEL_BEDRAG FIXED DEC(11,2);
```

Het getal tussen haakjes is de vaste maximumkardinaliteit — moet exact overeenkomen met de
maximumkardinaliteit van de bijbehorende Group View in CA Gen.

## 2. Toekenningen

```pli
X = Y;
KLANT_STRUCT = ANDERE_STRUCT;   /* volledige structuurtoekenning */
```

Let op impliciete typeconversie tussen `FIXED BIN` en `FIXED DEC`, en tussen `CHAR` van verschillende
lengtes (padding/truncatie gebeurt stilzwijgend in PL/I).

## 3. Controlestructuren

### IF/THEN/ELSE

```pli
IF conditie THEN
    DO;
        <statements>
    END;
ELSE
    DO;
        <statements>
    END;
```

### SELECT (CASE-equivalent)

```pli
SELECT (expressie);
    WHEN (waarde1)
        DO;
            <statements>
        END;
    WHEN (waarde2)
        DO;
            <statements>
        END;
    OTHERWISE
        DO;
            <statements>
        END;
END;
```

### DO-lussen

```pli
DO I = B TO C BY D;
    <statements>
END;

DO WHILE (conditie);
    <statements>
END;
```

### LEAVE / meerdere niveaus verlaten

`LEAVE;` verlaat **één** niveau van de dichtstbijzijnde lus. Voor het verlaten van meerdere geneste
niveaus (CA Gen `ESCAPE n`) is een expliciet label nodig:

```pli
BUITENLUS: DO I = 1 TO 10;
    DO J = 1 TO 10;
        IF conditie THEN
            LEAVE BUITENLUS;
    END;
END BUITENLUS;
```

## 4. Procedures en CALL

```pli
CALL MODULENAAM(IMP_KLANT, EXP_KLANT);
```

Parameters worden **expliciet** meegegeven — in tegenstelling tot CA Gen's impliciete View Matching moet
in PL/I elke Import- en Export-parameter met naam in de CALL staan. Conventie binnen dit project:
per aangeroepen module één `IMP_*`-structuur en één `EXP_*`-structuur (zie `data-structure-mapping.md`).

## 5. Embedded SQL

### Enkelvoudige lees-actie

```pli
EXEC SQL
    SELECT KLANT_NAAM, KLANT_STATUS
    INTO :KLANT_NAAM, :KLANT_STATUS
    FROM KLANT
    WHERE KLANT_ID = :KLANT_ID;
```

### Cursor + fetch-lus (voor READ EACH / Group Views)

```pli
EXEC SQL DECLARE C1 CURSOR FOR
    SELECT REGEL_ID, REGEL_BEDRAG
    FROM REGEL
    WHERE KLANT_ID = :KLANT_ID;

EXEC SQL OPEN C1;

I = 1;
DO WHILE (SQLCODE = 0 & I <= 50);   /* 50 = maximumkardinaliteit */
    EXEC SQL FETCH C1
        INTO :REGEL_ID(I), :REGEL_BEDRAG(I);
    IF SQLCODE = 0 THEN
        I = I + 1;
END;

EXEC SQL CLOSE C1;
```

Twee eindcondities zijn verplicht: `SQLCODE = 100` (geen rijen meer) **en** het bereiken van de
maximumkardinaliteit van de array. Beide moeten altijd samen gecontroleerd worden om buffer-overflow te
voorkomen.

### CREATE / UPDATE / DELETE

```pli
EXEC SQL INSERT INTO KLANT (KLANT_ID, KLANT_NAAM) VALUES (:KLANT_ID, :KLANT_NAAM);
EXEC SQL UPDATE KLANT SET KLANT_STATUS = :KLANT_STATUS WHERE KLANT_ID = :KLANT_ID;
EXEC SQL DELETE FROM KLANT WHERE KLANT_ID = :KLANT_ID;
```

## 6. Foutafhandeling en exitstate-conventie

PL/I heeft geen ingebouwd exitstate-mechanisme zoals CA Gen. Project-conventie:

- Elke `EXP_*`-structuur bevat een veld `EXITSTATE` (`CHAR(1)` of `FIXED BIN(31)`, projectconventie
  volgen — zie `data-structure-mapping.md`).
- Standaardcodes: `0` = OK, `4` = waarschuwing, `8` = fout (tenzij een module-specifieke mapping-tabel
  anders aangeeft — zie `error-handling-conventions.md`).
- Na elke `EXEC SQL`-actie wordt `SQLCODE` gecontroleerd en, indien relevant, vertaald naar het
  `EXITSTATE`-veld van de eigen procedure.

Voorbeeld:

```pli
EXEC SQL SELECT ... INTO ... FROM ... WHERE ...;

IF SQLCODE = 100 THEN
    EXP_EXITSTATE = 8;   /* niet gevonden -> fout, conform mapping-tabel */
ELSE IF SQLCODE ^= 0 THEN
    EXP_EXITSTATE = 8;
ELSE
    EXP_EXITSTATE = 0;
```

## 7. Commentaar en headers

Volgens PLAPO-conventie krijgt elke gemigreerde module een vaste header met minimaal: modulenaam,
oorspronkelijke CA Gen-procedurenaam, migratiedatum, en een korte functionele omschrijving. Exact formaat:
zie `coding-standards.md`.

## 8. Wat dit bestand niet dekt

- Volledige PL/I-taalspecificatie (bv. multitasking, geavanceerde storage classes, PICTURE-clausules
  buiten wat binnen PLAPO gebruikt wordt).
- CICS- of IMS-specifieke constructen, tenzij expliciet aangetroffen in de bron — in dat geval flaggen
  voor menselijke review.