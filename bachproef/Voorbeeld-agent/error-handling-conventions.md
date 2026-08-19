### Aandachtspunten per categorie

**Variabelen en datatypes**
Attribute-based variabelen erven hun datatype rechtstreeks van het datamodel. In PL/I dient dit type expliciet gedeclareerd te worden via `DCL` met een geneste 1/2-structuur, conform de bestaande PLAPO-conventie. Numerieke types worden bij voorkeur gedeclareerd als `FIXED BIN(31)`; tekstvelden als `CHAR(n)` met dezelfde lengte als in CA Gen.

**Toekenningen en expressies**
Een `SET ... TO ...` vertaalt direct naar een toewijzing. Aandacht moet gaan naar impliciete typeconversies en naar het verschil tussen het toekennen van een hele structuur en een enkel veld.

**Lussen**
CA Gen kent twee lustypes: `FOR`-lussen op een index en `WHILE`-lussen op een conditie. PL/I biedt voor beide een variant van `DO`. Wanneer in CA Gen een `ESCAPE`-pijl meerdere niveaus omhoog wijst, moet in PL/I een `LEAVE` met expliciet label gebruikt worden om hetzelfde gedrag te bereiken.

**USE en view matching**
In CA Gen worden views impliciet gekoppeld via View Matching. In PL/I moet deze koppeling expliciet gemaakt worden door de Import- en Export-views als parameters van de `CALL` mee te geven. Aanbeveling: per CAB één parameterstructuur per view definiëren (`IMP_*` voor import en `EXP_*` voor export). EAB's blijven onveranderd en behouden hun huidige PL/I-interface.

**Database-operaties**
CA Gen entity-actions worden vertaald naar expliciete `EXEC SQL`-statements. Bij `READ EACH` op een Group View wordt een DB2-cursor gebruikt met een `FETCH`-lus die ophoudt zodra `SQLCODE = 100` of de maximumkardinaliteit van de Group View bereikt is. Het is essentieel om beide eindcondities te respecteren om buffer overflows te voorkomen.

**Exitstates**
CA Gen kent een ingebouwd exitstate-mechanisme dat in PL/I geen directe tegenhanger heeft. Aanbeveling: de exitstate als veld opnemen in de export-parameterstruct, met een afgesproken set codes (bv. `0` = OK, `4` = waarschuwing, `8` = fout).