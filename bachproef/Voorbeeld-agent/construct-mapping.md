## Mappingtabel: CA Gen → PL/I

| CA Gen-construct | PL/I-equivalent | Aandachtspunt |
|---|---|---|
| Attribute-based variabele | `DCL` met geneste 1/2-structuur | Datatype 1-op-1 afleiden uit datamodel |
| Local variabele (IMPORTS-blok) | `DCL` in declaratiesectie | Scope beperken tot procedure |
| `SET x TO y` | `x = y;` | Letten op type-coercion bij gemengde types |
| `FOR SUBSCRIPT OF a FROM b TO c BY d` | `DO i = b TO c BY d;` | Index expliciet declareren |
| `WHILE ... REPEAT` | `DO WHILE (cond); ... END;` | — |
| `ESCAPE` (n niveaus) | `LEAVE` (label gebruiken bij meerdere niveaus) | PL/I-`LEAVE` verlaat slechts één niveau zonder label |
| `IF/THEN/ELSE` | `IF cond THEN ... ELSE ...` | — |
| `CASE OF` | `SELECT; ... WHEN ... OTHER ... END;` | — |
| `USE` (CAB) | `CALL module(args);` | View matching wordt expliciet via parameterstructs |
| `USE` (EAB) | Idem, maar module bestaat al | EAB-interface niet wijzigen |
| `READ` / `READ EACH` | `EXEC SQL SELECT ...` (cursor bij EACH) | Group View → cursor + fetch-loop |
| `CREATE` | `EXEC SQL INSERT ...` | — |
| `UPDATE` | `EXEC SQL UPDATE ...` | — |
| `DELETE` | `EXEC SQL DELETE ...` | — |
| Group View (array) | Array of `DCL`-structuur met `(n)` | Maximumkardinaliteit expliciet vastleggen |
| Exitstate | Return-code/status-veld in import-/exportparameter | Geen 1-op-1 mapping; conventie afspreken |
