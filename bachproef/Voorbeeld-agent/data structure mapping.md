# Data Structure Mapping: CA Gen Views → PL/I DCL

Dit bestand beschrijft hoe CA Gen-datastructuren (Views, Group Views, Elements/Attributes) expliciet
worden vertaald naar PL/I `DCL`-structuren. Dit is doorgaans de **grootste bronfoutbron** bij migratie —
volg deze regels strikt en flag afwijkende gevallen altijd voor review in plaats van te improviseren.

## 1. Basisprincipe

| CA Gen-concept | PL/I-equivalent |
|---|---|
| View | `DCL 1`-structuur |
| Element/Attribute binnen een View | `2`-niveau veld binnen de structuur |
| Group View | Array van een `DCL 1`-structuur met vaste `(n)`-dimensie |
| Import View van een procedure | `IMP_<procedurenaam>`-structuur |
| Export View van een procedure | `EXP_<procedurenaam>`-structuur, incl. `EXITSTATE`-veld |

## 2. Datatype-mapping (Element → PL/I)

| CA Gen Element-type | PL/I-declaratie | Toelichting |
|---|---|---|
| Numeric (integer, geen decimalen) | `FIXED BIN(31)` | Standaard voor identifiers, counters |
| Numeric (met decimalen) | `FIXED DEC(p,s)` | `p` = totale lengte, `s` = aantal decimalen, exact uit het datamodel overnemen |
| Character | `CHAR(n)` | `n` = exacte veldlengte uit CA Gen, nooit afronden of verkorten |
| Date | `CHAR(10)` (formaat `YYYY-MM-DD`) tenzij projectconventie anders voorschrijft | Zie `coding-standards.md` voor het definitieve datumformaat binnen PLAPO |
| Time | `CHAR(8)` (formaat `HH:MM:SS`) | Idem, projectconventie volgen |
| Indicator / boolean-achtig veld | `CHAR(1)` met vaste waarden (`'Y'`/`'N'` of `'1'`/`'0'`) | CA Gen kent geen natief boolean-type; conventie in het datamodel controleren |

**Regel**: het datatype wordt altijd 1-op-1 afgeleid uit het CA Gen-datamodel (Encyclopedia), nooit uit de
gegenereerde broncode of uit aannames over "waarschijnlijke" lengtes.

## 3. Null-indicatoren

CA Gen-attributen kunnen "optional" (nullable) zijn. PL/I zelf heeft geen native NULL-concept voor
gewone variabelen buiten pointers. Projectconventie:

- Elk optioneel element krijgt een bijbehorend indicatorveld: `<VELDNAAM>_IND` (`CHAR(1)`, `'Y'` = waarde
  aanwezig, `'N'` = null).
- Bij `EXEC SQL`-operaties wordt dit gekoppeld aan een SQL-indicatorvariabele (`:VELD :VELD_IND`).

```pli
DCL 1 KLANT_STRUCT,
      2 KLANT_ID          FIXED BIN(31),
      2 KLANT_EMAIL       CHAR(60),
      2 KLANT_EMAIL_IND   CHAR(1);
```

```pli
EXEC SQL SELECT KLANT_EMAIL
    INTO :KLANT_EMAIL :KLANT_EMAIL_IND
    FROM KLANT
    WHERE KLANT_ID = :KLANT_ID;
```

## 4. Group Views → arrays

Een Group View heeft altijd een vaste maximumkardinaliteit in het datamodel. Deze wordt letterlijk
overgenomen als array-dimensie:

```pli
DCL 1 REGEL_TABEL(50),          /* 50 = maximumkardinaliteit uit het datamodel */
      2 REGEL_ID       FIXED BIN(31),
      2 REGEL_BEDRAG   FIXED DEC(11,2);

DCL AANTAL_REGELS FIXED BIN(31);   /* huidige vulling van de array, apart bijhouden */
```

**Verplicht**: naast de array zelf wordt altijd een teller (`AANTAL_REGELS` of vergelijkbaar) bijgehouden
die aangeeft hoeveel elementen daadwerkelijk gevuld zijn — arrays in PL/I hebben geen ingebouwd
lengtebegrip zoals CA Gen Group Views dat impliciet wel hebben.

## 5. Redefines / hersubtypering

Als een CA Gen-attribuut in het datamodel via een subtype-attribuut of redefinitie meerdere
interpretaties heeft (bv. een generiek codeveld dat per context anders wordt geïnterpreteerd), wordt dit
in PL/I opgelost met `DEFINED`:

```pli
DCL CODE_VELD CHAR(4);
DCL NUM_CODE FIXED BIN(15) DEFINED CODE_VELD;
```

Gebruik dit **alleen** wanneer het datamodel expliciet een redefine aangeeft — niet als generieke
workaround voor typemismatches.

## 6. Import-/Exportparameterstructs per procedure

Conform de USE/View-Matching-conventie (zie `cagen-syntax-reference.md`, sectie 6) krijgt elke procedure
die wordt aangeroepen een eigen, expliciete parameterstructuur:

```pli
DCL 1 IMP_BEREKEN_KORTING,
      2 KLANT_ID        FIXED BIN(31),
      2 ORDERBEDRAG     FIXED DEC(11,2);

DCL 1 EXP_BEREKEN_KORTING,
      2 KORTINGSBEDRAG  FIXED DEC(11,2),
      2 EXITSTATE       FIXED BIN(31);

CALL BEREKEN_KORTING(IMP_BEREKEN_KORTING, EXP_BEREKEN_KORTING);
```

Regels:

- Naamgeving: `IMP_<doelprocedure>` / `EXP_<doelprocedure>`.
- Elk veld dat in CA Gen impliciet via View Matching werd doorgegeven, moet hier **expliciet** aanwezig
  zijn — controleer dit tegen de Import/Export View-definities van zowel de aanroepende als de
  aangeroepen procedure.
- Bij EAB's: de bestaande PL/I-interface van de EAB is leidend; de `IMP_*`/`EXP_*`-structuur moet daarop
  aansluiten, niet andersom.

## 7. Checklist bij elke structuur-mapping

- [ ] Elk datamodel-element heeft een PL/I-veld met exact hetzelfde datatype en dezelfde lengte.
- [ ] Elk optioneel element heeft een `_IND`-indicatorveld.
- [ ] Elke Group View heeft een array met de juiste vaste dimensie én een aparte teller.
- [ ] Elke `EXP_*`-structuur bevat een `EXITSTATE`-veld conform `error-handling-conventions.md`.
- [ ] Geen enkel veld is "verzonnen" of vrij geïnterpreteerd — alles herleidbaar tot het datamodel of de
      bestaande EAB-interface.

Twijfelgevallen (bv. een attribuut met een ongebruikelijk redefine, of een Group View zonder duidelijke
maximumkardinaliteit) worden nooit zelfstandig opgelost — flag deze voor menselijke review conform
`agent-instructions.md`.