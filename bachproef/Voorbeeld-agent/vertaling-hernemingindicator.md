# Vertaling: Hernemingindicator-controle 8,3 credits 6.1k tokens

## CA Gen code

IF loc_herneming_indicator ief_supplied flag IS EQUAL TO "J"

    NOTE *** indien er hernemingen zijn worden deze hier toegevoegd aan de plapo ***
    SET loc_match ief_supplied flag TO "0"
    FOR SUBSCRIPT OF grl_tab_prdo FROM 1 TO LAST OF grl_tab_prdo BY 1

        IF gloc_gen int_pln_plnprdo_productie_order nr_ty_prd_alt_bs IS GREATER THAN 1
            NOTE *** nr_ty_prd_alt_bs is voor te hernemen materiaal > 1 ***
            SET loc_match ief_supplied flag TO "1"
            ESCAPE

END IF

## Module: Hernemingindicator-verwerking
### Bron: CA Gen Action Diagram fragment (hernemingindicator check en tabel-scan)

---

## 1. Declaraties

```pli
/*
 * Werkstrukturen voor herneming-verwerking
 */
DCL 1 LOC_HERNEMING_INDICATOR,
      2 IEF_SUPPLIED_FLAG   CHAR(1);    /* "J" = ja, "N" = nee */

DCL 1 LOC_MATCH,
      2 IEF_SUPPLIED_FLAG   CHAR(1);    /* "0" = niet gevonden, "1" = gevonden */

/*
 * Global structuur voor gegenereerde data
 */
DCL 1 GLOC_GEN,
      2 INT_PLN_PLNPRDO_PRODUCTIE_ORDER,
        3 NR_TY_PRD_ALT_BS  FIXED BIN(31);

/*
 * Group View: tabel met productie-orders
 * Aanname: maximumkardinaliteit uit datamodel bepalen
 */
DCL 1 GRL_TAB_PRDO(100),           /* FIXME: maximumkardinaliteit bevestigen */
      2 INT_PLN_PLNPRDO_PRODUCTIE_ORDER,
        3 NR_TY_PRD_ALT_BS  FIXED BIN(31);

/*
 * Teller en index voor FOR-lus
 */
DCL GRL_TAB_PRDO_COUNT FIXED BIN(31);
DCL I FIXED BIN(31);
```

---

## 2. Procedurebody

```pli
/*
 * Controleer of hernemingindicator aan staat
 */
IF LOC_HERNEMING_INDICATOR.IEF_SUPPLIED_FLAG = "J" THEN
    DO;
        /* 
         * *** Indien er hernemingen zijn worden deze hier toegevoegd aan de plapo ***
         */
        LOC_MATCH.IEF_SUPPLIED_FLAG = "0";
        
        /*
         * Scan tabel op hernemingen (nr_ty_prd_alt_bs > 1)
         */
        DO I = 1 TO GRL_TAB_PRDO_COUNT BY 1;
            
            IF GLOC_GEN.INT_PLN_PLNPRDO_PRODUCTIE_ORDER.NR_TY_PRD_ALT_BS > 1 THEN
                DO;
                    /*
                     * *** nr_ty_prd_alt_bs is voor te hernemen materiaal > 1 ***
                     */
                    LOC_MATCH.IEF_SUPPLIED_FLAG = "1";
                    
                    /* Verlaat de FOR-lus */
                    LEAVE;
                END;
        
        END;  /* DO I = 1 TO ... */
    
    END;  /* IF LOC_HERNEMING_INDICATOR ... */
```

---

## Vertaalnotities

1. **IF-check op vlagveld**:
   - CA Gen `IF loc_herneming_indicator ief_supplied flag IS EQUAL TO "J"` → PL/I `IF LOC_HERNEMING_INDICATOR.IEF_SUPPLIED_FLAG = "J" THEN`
   - Structuurveld-syntax: `structuur.veld` in PL/I (flat notation)

2. **NOTE → Commentaar**:
   - CA Gen `NOTE ***  ... ***` → PL/I `/* ... */`
   - Commentaren zijn volledig opgenomen voor leesbaarheid

3. **FOR SUBSCRIPT met index**:
   - CA Gen `FOR SUBSCRIPT OF grl_tab_prdo FROM 1 TO LAST OF grl_tab_prdo BY 1` → PL/I `DO I = 1 TO GRL_TAB_PRDO_COUNT BY 1;`
   - Aangenomen dat `LAST OF` wordt vertaald naar een teller `GRL_TAB_PRDO_COUNT` (standaard praktijk in PL/I)

4. **ESCAPE naar LEAVE**:
   - CA Gen `ESCAPE` (lus verlaten) → PL/I `LEAVE;`
   - Dit verlaat de dichtstbijzijnde DO-lus (geen label nodig voor single-level escape)

5. **IF met GREATER THAN**:
   - CA Gen `IF gloc_gen int_pln_plnprdo_productie_order nr_ty_prd_alt_bs IS GREATER THAN 1` → PL/I `IF GLOC_GEN.INT_PLN_PLNPRDO_PRODUCTIE_ORDER.NR_TY_PRD_ALT_BS > 1`
   - Vergelijkingsoperator: `>` in PL/I (equivalent aan CA Gen `IS GREATER THAN`)

6. **SET-statements**:
   - CA Gen `SET loc_match ief_supplied flag TO "0"` → PL/I `LOC_MATCH.IEF_SUPPLIED_FLAG = "0";`
   - Direct toewijzing (geen conversie nodig voor string-velden)

---

## Openstaande vragen

1. **Group View-maximumkardinaliteit (GRL_TAB_PRDO)**:
   - CA Gen gebruikt `LAST OF grl_tab_prdo` (dynamische grens)
   - PL/I vereist vaste declaratie `(n)`. Hier aangenomen als 100
   - **Actie**: Werkelijke maximumkardinaliteit uit CA Gen-datamodel bepalen

2. **Hernemingindicator-waarden**:
   - CA Gen checkt op `"J"` (aangenomen: "Ja"/Ja)
   - Wat zijn de geldige waarden? Alleen "J"/"N", of ook andere codes?
   - **Actie**: Projectconventie voor vlagwaarden controleren (zie business-glossary.md)

3. **GRL_TAB_PRDO inhoud en bron**:
   - Het fragment toont een scan over `grl_tab_prdo`, maar niet hoe deze tabel gevuld wordt
   - Is dit een Group View die als import meekomen, of lokaal gevuld?
   - **Actie**: Bepalen waar deze tabel vandaan komt (parameter, cursor-fetch, etc.)

4. **Logica na de FOR-lus**:
   - Na de `DO I = 1 TO ...` zijn geen statements meer gegeven
   - Wat gebeurt er na de hernemingindicator-verwerking? 
   - **Actie**: Context van omringende procedure helderder krijgen

5. **Veldtype van NR_TY_PRD_ALT_BS**:
   - Hier aangenomen als `FIXED BIN(31)`, maar type is niet bevestigd
   - **Actie**: Datatype uit Encyclopedia controleren

6. **Scope van LOC_MATCH**:
   - LOC_MATCH wordt ingesteld op "0" of "1"
   - Wordt deze waarde gebruikt door een volgende procedure-statement, of geëxporteerd?
   - **Actie**: Omvang van variabele-scope bepalen

---

## Validatie-checklist (conform validation-checklist.md)

- [ ] Maximumkardinaliteit van GRL_TAB_PRDO vastgesteld
- [ ] Hernemingindicator-waarden ("J"/"N" of andere) bevestigd
- [ ] GRL_TAB_PRDO-bron en vulling bepaald
- [ ] Vervolg-logica na FOR-lus vastgesteld
- [ ] Datatypes van alle velden (met name NR_TY_PRD_ALT_BS) gevalideerd
- [ ] Scope van LOC_MATCH en doelgebruik bepaald
- [ ] Nested-structuur GLOC_GEN correct gemapt
