# CA Gen Syntax Reference

Referentiebestand met de grammatica en semantiek van CA Gen (voorheen IEF/COOL:Gen) Action Diagrams,
zoals gebruikt door de migratie-agent bij het vertalen naar PL/I. Dit bestand beschrijft **wat** elk
construct betekent; de vertaling naar PL/I staat in `construct-mapping.md` en `data-structure-mapping.md`.

## 1. Opbouw van een Action Diagram

Een Action Diagram (AD) beschrijft de procedurele logica van een CA Gen-procedure (CAB of EAB) als een
geïndenteerde boomstructuur. Elke regel is een statement; indentatie bepaalt nesting (vergelijkbaar met
Python, maar met expliciete open/sluit-constructen zoals `IF`/`END IF`).

Een procedure bestaat typisch uit:

- **IMPORTS-blok** — inkomende parameters (Import View).
- **EXPORTS-blok** — uitgaande parameters (Export View), inclusief exitstate-veld.
- **Local variabelen** — tijdelijke variabelen binnen de procedure-scope.
- **Body** — de eigenlijke Action Diagram-logica.

## 2. Variabelen

| Type | Beschrijving |
|---|---|
| **Attribute-based variabele** | Variabele waarvan het datatype rechtstreeks is afgeleid van een attribuut in het datamodel (Encyclopedia). |
| **Local variabele** | Gedeclareerd in het IMPORTS/LOCALS-blok van de procedure, scope beperkt tot die procedure. |
| **Group View variabele** | Array-achtige structuur met een vaste maximumkardinaliteit, gebruikt bij `READ EACH` en herhalende output. |

## 3. Toekenningen

```
SET <doel> TO <bron>
```

Kent de waarde van `<bron>` toe aan `<doel>`. Kan een enkel veld of een volledige structuur betreffen.
Bij structuur-toekenningen worden alle onderliggende elementen in één keer gekopieerd (veldnamen moeten
overeenkomen op basis van View Matching — zie sectie 6).

## 4. Controlestructuren

### IF/THEN/ELSE

```
IF <conditie>
    <statements>
ELSE
    <statements>
END IF
```

### CASE OF

```
CASE OF <expressie>
WHEN <waarde1>
    <statements>
WHEN <waarde2>
    <statements>
ELSE
    <statements>
END CASE
```

### FOR-lus (index-gebaseerd)

```
FOR SUBSCRIPT OF <array> FROM <b> TO <c> BY <d>
    <statements>
END FOR
```

Itereert over een Group View of array met een expliciete indexvariabele.

### WHILE-lus (conditie-gebaseerd)

```
WHILE <conditie> REPEAT
    <statements>
END WHILE
```

### ESCAPE

```
ESCAPE <n niveaus>
```

Springt uit de lus, optioneel meerdere nesting-niveaus tegelijk. **Let op**: CA Gen staat toe dat één
`ESCAPE` meerdere geneste lussen tegelijk verlaat — dit heeft geen directe 1-op-1 tegenhanger in PL/I
(zie `construct-mapping.md`).

## 5. Entity Actions (database-toegang)

| Statement | Betekenis |
|---|---|
| `READ` | Leest één entity-instantie op basis van een sleutel of conditie. |
| `READ EACH` | Leest een verzameling instanties in een Group View (impliciete cursor-semantiek). |
| `CREATE` | Voegt een nieuwe entity-instantie toe. |
| `UPDATE` | Wijzigt een bestaande entity-instantie. |
| `DELETE` | Verwijdert een entity-instantie. |

Bij `READ` en `READ EACH` genereert CA Gen automatisch de onderliggende SQL uit het datamodel; deze SQL
is niet zichtbaar in de Action Diagram-tekst zelf en moet worden afgeleid uit de datamatrix (zie
project-specifieke documentatie) — niet visueel geraden uit de AD.

## 6. USE-statement en View Matching

```
USE <CAB-of-EAB-naam>
```

Roept een andere procedure aan. CA Gen koppelt Import- en Export-views **impliciet** op basis van
overeenkomende veldnamen en types tussen de aanroepende en aangeroepen procedure (View Matching). Er is
in de Action Diagram-tekst geen expliciete parameterlijst zichtbaar — welke velden precies worden
meegegeven, moet worden afgeleid uit de view-definities van beide procedures.

- **CAB** (Common Action Block) — herbruikbare, binnen het project gedefinieerde procedure.
- **EAB** (External Action Block) — procedure met een vaste, extern beheerde interface die niet gewijzigd
  mag worden bij migratie.

## 7. Exitstates

Elke procedure heeft een ingebouwd exitstate-mechanisme: een statusuitkomst die door de procedure wordt
gezet (bv. impliciet bij een mislukte `READ`, of expliciet via `SET EXITSTATE TO <waarde>`) en die de
aanroepende procedure gebruikt om vervolgstappen te bepalen. Exitstates zijn tekstuele labels
(bijvoorbeeld `SUCCESS`, `FAILURE`, een custom-gedefinieerde waarde) zonder generieke numerieke codering.

## 8. Group Views

Een Group View is een repeterende structuur met een **vaste maximumkardinaliteit**, gedefinieerd in het
datamodel. Wordt gebruikt om:

- Resultaten van een `READ EACH` te bufferen.
- Herhalende Import/Export-data tussen procedures door te geven.

De maximumkardinaliteit is een harde grens — de generator (en dus ook de agent) mag deze nooit
overschrijden bij het vullen van de structuur.

## 9. Wat dit bestand niet dekt

- Screen- en dialoogdefinities (niet relevant voor batch/PL/I-migratie).
- GUI-generatie-specifieke constructen.
- CA Gen Toolset-commando's voor export/import van de Encyclopedia (zie separate tooling-documentatie).

Voor constructen die hier niet expliciet beschreven staan, mag de agent **nooit** een eigen interpretatie
verzinnen — dit moet worden geflagd voor menselijke review (zie `agent-instructions.md`).