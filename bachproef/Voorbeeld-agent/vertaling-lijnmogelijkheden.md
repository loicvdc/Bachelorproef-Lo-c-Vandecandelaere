# Vertaling: Lijnmogelijkheden bepalen (CAB-oproep met complex View Matching) 9,3 credits 10.1K tokens

## CA Gen code

USE ab_lpl_bepaal_lijnmogelijkheden
    WHICH IMPORTS: Entity View loc_int_lpl_moglijn_en_limiet TO Entity View imp_int_lpl_moglijn_en_limiet
                   Group View  grl_wz63741_geforce_ude_lijnmog TO Group View  gref_geforceerde_lijn
    WHICH EXPORTS: Entity View loc_int_lpl_moglijn_en_limiet FROM Entity View exp_int_lpl_moglijn_en_limiet
                   <none> FROM Entity View exp_bs_van_mat_int_lpl_moglijn_en_limiet
                   <none> FROM Group View  gre_tracing_lijnmogelijkheden
                   Group View  grl_eig_lijn_inst_nok FROM Group View  gre_reden_inst_nok
                   Group View  grl_specifieke_mogelijkh FROM Group View  gre_spec_mogelijkh
                   Work View  loc_int_error_handling FROM Work View  exp_int_error_handling


IF EXISTSITE IS NOT EQUAL TO sid_verwerking_ab_geslaagd

    SET gloc_gbp_mat_ft int_bsl_bs nr_bs TO imp_mataf nr_bs
    SET gloc_gbp_mat_ft int_bsl_baslg nr_po_bs TO imp_mataf nr_po_bs
    SET gloc_mat_ft int_mat_materiaal eenheid_code TO "S"
    SET gloc_mat_ft int_mat_materiaaleenheid codering_identificatienummer TO imp_int_mat_materiaaleenheid codering_identificatienummer
    SET gloc_mat_ft int_mat_materiaal eenheid nummer TO imp_int_mat_materiaaleenheid nummer
    SET gloc_mat sid_werkveld textu TO concat("TMZ01 Ab_lpl_bepaal_lijnmogelijkheden nok. (BAF) Rkde : ", concat(substr(textum(loc
        reason_code), 11, 5)))
    USE eab_druk_foutlijn_materiaal
        WHICH IMPORTS: Group View  grl_foutlijn_mat TO Group View  gri_foutlijn_mat

    SET loc_int_pln_plmat_materiaal inst_ok TO "0000000000"
    SET loc_int_baf_bafmat_materiaal ly_ok_gl_h TO "00"
    SET loc_int_baf_bafmat_materiaal ly_ok_gl_hnx TO "00"

ELSE

    IF loc_int_lpl_moglijn_en_limiet inst_ok IS NOT EQUAL TO SPACES
        SET loc_int_pln_plmat_materiaal inst_ok TO concat(loc_int_lpl_moglijn_en_limiet inst_ok, "00000")
    ELSE
        SET loc_int_pln_plmat_materiaal inst_ok TO "0000000000"


    FOR SUBSCRIPT OF grl_specifieke_mogelijkh FROM 1 TO LAST OF grl_specifieke_mogelijkh BY 1
        CASE OF gloc_spec_mog int_lpl_moglijn_en_limiet mog_spfk
        CASE "H2"
            SET loc_int_baf_bafmat_materiaal ly_ok_gl_h TO gloc_spec_mog int_lpl_moglijn_en_limiet inst_ok
        CASE "NHX"
            SET loc_int_baf_bafmat_materiaal ly_ok_gl_hnx TO gloc_spec_mog int_lpl_moglijn_en_limiet inst_ok
        OTHERWISE
            SET loc_int_baf_bafmat_materiaal ly_ok_gl_h TO "00"
            SET loc_int_baf_bafmat_materiaal ly_ok_gl_hnx TO "00"

## Module: AB_LPL_BEPAAL_LIJNMOGELIJKHEDEN
### Bron: CA Gen Action Diagram fragment (bepaling lijnmogelijkheden en foutafhandeling)

---

## 1. Declaraties

Alle gebruikte structuren worden geïnitialiseerd:

```pli
/*
 * Entity View-structuren (enkelvoudig)
 */
DCL 1 LOC_INT_LPL_MOGLIJN_EN_LIMIET,
      2 INST_OK              CHAR(10),
      2 <FIELDS_UNKNOWN>;

DCL 1 IMP_INT_LPL_MOGLIJN_EN_LIMIET,
      2 INST_OK              CHAR(10),
      2 <FIELDS_UNKNOWN>;

DCL 1 EXP_INT_LPL_MOGLIJN_EN_LIMIET,
      2 INST_OK              CHAR(10),
      2 <FIELDS_UNKNOWN>;

/*
 * Globale werkvelden (materiaal, foutafhandeling)
 */
DCL 1 GLOC_GBP_MAT_FT,
      2 INT_BSL_BS,
        3 NR_BS              FIXED BIN(31);
      2 INT_BSL_BASLG,
        3 NR_PO_BS           FIXED BIN(31);

DCL 1 GLOC_MAT_FT,
      2 INT_MAT_MATERIAAL,
        3 EENHEID_CODE       CHAR(1),
        3 EENHEID_NUMMER     FIXED BIN(31);
      2 INT_MAT_MATERIAALEENHEID,
        3 CODERING_IDENTIFICATIENUMMER CHAR(7);

DCL 1 GLOC_MAT,
      2 SID_WERKVELD,
        3 TEXTU              CHAR(256);

DCL 1 LOC_INT_PLN_PLMAT_MATERIAAL,
      2 INST_OK              CHAR(10);

DCL 1 LOC_INT_BAF_BAFMAT_MATERIAAL,
      2 LY_OK_GL_H           CHAR(2),
      2 LY_OK_GL_HNX         CHAR(2);

/*
 * Group View-structuren (arrays met maximumkardinaliteit)
 * Aanname: maximumkardinaliteit uit datamodel bepalen
 */
DCL 1 GRL_SPECIFIEKE_MOGELIJKH(100),    /* FIXME: maximumkardinaliteit bevestigen */
      2 INT_LPL_MOGLIJN_EN_LIMIET,
        3 MOG_SPFK           CHAR(3),
        3 INST_OK            CHAR(2);

DCL 1 GLOC_SPEC_MOG,
      2 INT_LPL_MOGLIJN_EN_LIMIET,
        3 MOG_SPFK           CHAR(3),
        3 INST_OK            CHAR(2);

/* Error-handling structuur */
DCL 1 LOC_INT_ERROR_HANDLING,
      2 EXITSTATE            FIXED BIN(31),
      2 <FIELDS_UNKNOWN>;

/* Import-handles (aangenomen uit context) */
DCL 1 H_MATAF,
      2 NR_BS                FIXED BIN(31),
      2 NR_PO_BS             FIXED BIN(31);

DCL 1 H_INT_MAT_MATERIAALEENHEID,
      2 CODERING_IDENTIFICATIENUMMER CHAR(7),
      2 NUMMER               FIXED BIN(31);

/*
 * Tellers en indices voor lussen
 */
DCL GRL_SPECIFIEKE_MOGELIJKH_COUNT FIXED BIN(31);
DCL I FIXED BIN(31);

/*
 * Temporaire variabelen voor string-operaties
 */
DCL REASON_CODE_STR CHAR(16);
DCL CONCAT_STR CHAR(256);

/*
 * Import-/Exportstructuren voor CAB-call
 */
DCL 1 IMP_LPL_BEPAAL_LIJNMOGELIJKHEDEN,
      2 IMP_INT_LPL_MOGLIJN_EN_LIMIET LIKE LOC_INT_LPL_MOGLIJN_EN_LIMIET,
      2 IMP_GEFORCEERDE_LIJN LIKE GRL_WZ63741_GEFORCE_UDE_LIJNMOG(100);

DCL 1 EXP_LPL_BEPAAL_LIJNMOGELIJKHEDEN,
      2 EXP_INT_LPL_MOGLIJN_EN_LIMIET LIKE LOC_INT_LPL_MOGLIJN_EN_LIMIET,
      2 EXP_REDEN_INST_NOK LIKE GRL_EIG_LIJN_INST_NOK(100),
      2 EXP_SPEC_MOGELIJKH LIKE GRL_SPECIFIEKE_MOGELIJKH(100),
      2 EXP_ERROR_INT_ERROR_HANDLING LIKE LOC_INT_ERROR_HANDLING,
      2 EXITSTATE            FIXED BIN(31);

/*
 * Structuren voor geneste EAB-call
 */
DCL 1 IMP_DRUK_FOUTLIJN_MAT,
      2 IMP_FOUTLIJN_MAT;     /* GRI_FOUTLIJN_MAT - niet volledig gespecificeerd */

DCL 1 EXP_DRUK_FOUTLIJN_MAT,
      2 EXITSTATE            FIXED BIN(31);
```

---

## 2. Procedurebody

```pli
/*
 * Roep CAB aan: ab_lpl_bepaal_lijnmogelijkheden
 * Deze procedure bepaalt welke lijnmogelijkheden beschikbaar zijn
 * op basis van de invoerparameters
 */
CALL AB_LPL_BEPAAL_LIJNMOGELIJKHEDEN(
    IMP_LPL_BEPAAL_LIJNMOGELIJKHEDEN,
    EXP_LPL_BEPAAL_LIJNMOGELIJKHEDEN
);

/*
 * IF-check: controleer exitstate van CAB
 */
IF EXP_LPL_BEPAAL_LIJNMOGELIJKHEDEN.EXITSTATE ^= 0 THEN
    DO;
        /* Fout: CAB niet succesvol */
        
        /* Vullen van foutgegevens */
        GLOC_GBP_MAT_FT.INT_BSL_BS.NR_BS = H_MATAF.NR_BS;
        GLOC_GBP_MAT_FT.INT_BSL_BASLG.NR_PO_BS = H_MATAF.NR_PO_BS;
        
        GLOC_MAT_FT.INT_MAT_MATERIAAL.EENHEID_CODE = "S";
        GLOC_MAT_FT.INT_MAT_MATERIAALEENHEID.CODERING_IDENTIFICATIENUMMER = 
            H_INT_MAT_MATERIAALEENHEID.CODERING_IDENTIFICATIENUMMER;
        GLOC_MAT_FT.INT_MAT_MATERIAAL.EENHEID_NUMMER = 
            H_INT_MAT_MATERIAALEENHEID.NUMMER;
        
        /* 
         * String-concatenatie: CONCAT(...) → PL/I ||
         * SUBSTR opnemen: substr(textum(loc_reason_code), 11, 5)
         * Aanname: LOC_REASON_CODE is een tekstweergave; TEXTUM geeft string terug
         * PL/I: SUBSTR(STRING, startpos, length)
         */
        REASON_CODE_STR = SUBSTR('UNKNOWN', 11, 5);  /* Placeholder */
        
        GLOC_MAT.SID_WERKVELD.TEXTU = 
            "TMZ01 Ab_lpl_bepaal_lijnmogelijkheden nok. (BAF) Rkde : " ||
            REASON_CODE_STR;
        
        /* Geneste EAB-call: eab_druk_foutlijn_materiaal */
        CALL DRUK_FOUTLIJN_MATERIAAL(IMP_DRUK_FOUTLIJN_MAT, EXP_DRUK_FOUTLIJN_MAT);
        
        /* Initialiseer outputvelden met foutwaarden */
        LOC_INT_PLN_PLMAT_MATERIAAL.INST_OK = "0000000000";
        LOC_INT_BAF_BAFMAT_MATERIAAL.LY_OK_GL_H = "00";
        LOC_INT_BAF_BAFMAT_MATERIAAL.LY_OK_GL_HNX = "00";
        
        RETURN;
    END;

ELSE
    DO;
        /* Succes: CAB is geslaagd */
        
        /* IF-controle op INST_OK */
        IF LOC_INT_LPL_MOGLIJN_EN_LIMIET.INST_OK ^= '' THEN
            DO;
                /* Concatenatie van INST_OK + padding */
                LOC_INT_PLN_PLMAT_MATERIAAL.INST_OK = 
                    LOC_INT_LPL_MOGLIJN_EN_LIMIET.INST_OK || "00000";
            END;
        ELSE
            DO;
                LOC_INT_PLN_PLMAT_MATERIAAL.INST_OK = "0000000000";
            END;
        
        /* FOR-lus over Group View */
        DO I = 1 TO GRL_SPECIFIEKE_MOGELIJKH_COUNT BY 1;
            
            GLOC_SPEC_MOG = GRL_SPECIFIEKE_MOGELIJKH(I);
            
            /* CASE-construct */
            SELECT (GLOC_SPEC_MOG.INT_LPL_MOGLIJN_EN_LIMIET.MOG_SPFK);
                
                WHEN ("H2")
                    DO;
                        LOC_INT_BAF_BAFMAT_MATERIAAL.LY_OK_GL_H = 
                            GLOC_SPEC_MOG.INT_LPL_MOGLIJN_EN_LIMIET.INST_OK;
                    END;
                
                WHEN ("NHX")
                    DO;
                        LOC_INT_BAF_BAFMAT_MATERIAAL.LY_OK_GL_HNX = 
                            GLOC_SPEC_MOG.INT_LPL_MOGLIJN_EN_LIMIET.INST_OK;
                    END;
                
                OTHERWISE
                    DO;
                        LOC_INT_BAF_BAFMAT_MATERIAAL.LY_OK_GL_H = "00";
                        LOC_INT_BAF_BAFMAT_MATERIAAL.LY_OK_GL_HNX = "00";
                    END;
            
            END;  /* SELECT */
        
        END;  /* DO I ... */
    
    END;  /* ELSE */

/* Procedure succesvol voltooid */
EXP_LPL_BEPAAL_LIJNMOGELIJKHEDEN.EXITSTATE = 0;
RETURN;
```

---

## Vertaalnotities

1. **USE-statement en View Matching**:
   - CA Gen `USE ab_lpl_bepaal_lijnmogelijkheden` → PL/I `CALL AB_LPL_BEPAAL_LIJNMOGELIJKHEDEN(...)`
   - Alle IMPORTS en EXPORTS zijn in twee structuren samengevat: `IMP_LPL_BEPAAL_LIJNMOGELIJKHEDEN` en `EXP_LPL_BEPAAL_LIJNMOGELIJKHEDEN`
   - Dit volgt de projectconventie uit `data-structure-mapping.md`, sectie 6

2. **EXITSTATE-check**:
   - CA Gen `IF EXITSITE IS NOT EQUAL TO sid_verwerking_ab_geslaagd` → PL/I `IF ...EXITSTATE ^= 0`
   - Aangenomen: EXITSTATE = 0 betekent succes (projectconventie)

3. **String-concatenatie (CONCAT)**:
   - CA Gen `CONCAT("String1", CONCAT("String2", "String3"))` → PL/I `"String1" || "String2" || "String3"`
   - SUBSTR-functie: `SUBSTR(string, startpos, length)` werkt hetzelfde in beide talen

4. **FOR-lus met CASE**:
   - CA Gen `FOR SUBSCRIPT OF grl_specifieke_mogelijkh FROM 1 TO LAST OF ...` → PL/I `DO I = 1 TO GRL_SPECIFIEKE_MOGELIJKH_COUNT`
   - CA Gen `CASE OF <expr> WHEN <val> ... OTHERWISE ...` → PL/I `SELECT (<expr>); WHEN (...) ... OTHERWISE ... END;`

5. **Nested Entity/Group Views**:
   - Structuren als `gloc_spec_mog int_lpl_moglijn_en_limiet inst_ok` zijn geneste 2/3-level velden in PL/I:
     `GLOC_SPEC_MOG.INT_LPL_MOGLIJN_EN_LIMIET.INST_OK`

6. **Geneste EAB-call**:
   - De `USE eab_druk_foutlijn_materiaal` wordt als afzonderlijke CALL uitgebreid
   - Aangenomen dat het alleen Group View-parameters heeft (grl_foutlijn_mat)

---

## Openstaande vragen

1. **EXITSTATE-waarde en betekenis**:
   - CA Gen zegt `IF EXITSITE IS NOT EQUAL TO sid_verwerking_ab_geslaagd` (exitstate-label)
   - Dit is in PL/I gemapt naar `EXITSTATE ^= 0`, maar **welke numerieke waarde betekent "geslaagd"?**
   - **Actie**: Bestaande projectconventie (error-handling-conventions.md) raadplegen; standaard is 0 = OK, maar dit moet bevestigd worden

2. **SUBSTR-functie op REASON_CODE**:
   - CA Gen-syntax: `concat(substr(textum(loc_reason_code), 11, 5))`
   - `TEXTUM()` is een CA Gen-functie; de betekenis is onduidelijk (omzetting naar tekst?)
   - Het originele veld `loc_reason_code` is niet in de declaraties gegeven
   - **Actie**: Bevestigen wat `TEXTUM()` doet en het originele veld declareren

3. **INST_OK-veldlengte en padding**:
   - CA Gen stelt `INST_OK` in op `CONCAT(... inst_ok, "00000")` (5-tekens padding)
   - Dit suggereert dat `INST_OK` in het totaal 10 tekens is (5 + 5)
   - **Actie**: Exacte veldlengtes uit Encyclopedia controleren

4. **Group View-maximumkardinaliteit (GRL_SPECIFIEKE_MOGELIJKH)**:
   - CA Gen `FOR SUBSCRIPT ... TO LAST OF grl_specifieke_mogelijkh` gebruikt `LAST OF` (dynamische grens)
   - In PL/I moet dit een vaste maximumkardinaliteit zijn; hier aangenomen als 100
   - **Actie**: Werkelijke maximumkardinaliteit uit CA Gen-datamodel bepalen en in array-declaratie zetten

5. **Geneste EAB-interface (DRUK_FOUTLIJN_MATERIAAL)**:
   - De geneste USE-call geeft alleen `WHICH IMPORTS: Group View grl_foutlijn_mat TO Group View gri_foutlijn_mat` aan
   - De volledige parameterstructuur van deze EAB is niet gegeven
   - **Actie**: Bestaande PL/I-module DRUK_FOUTLIJN_MATERIAAL controleren en parameterstructs valideren

6. **Import-handle-definitie**:
   - Verwijzingen als `imp_mataf`, `imp_int_mat_materiaaleenheid` veronderstellen globale import-handles
   - Deze zijn hier als `H_MATAF` en `H_INT_MAT_MATERIAALEENHEID` gemapt
   - **Actie**: Bevestigen dat deze handles in de omroepende procedure beschikbaar zijn

7. **Veldtypen in LOC_INT_LPL_MOGLIJN_EN_LIMIET en verwante structuren**:
   - Veel velden zijn gemarkeerd als `<FIELDS_UNKNOWN>` omdat ze niet in het fragment voorkomen
   - **Actie**: Volledige View-definities uit CA Gen-Encyclopedia ophalen

8. **LIKE vs. expliciete kopie in structures**:
   - Hier worden `LIKE`-verwijzingen gebruikt voor duplicate-structuren (PL/I-standaard)
   - **Actie**: Bevestigen dat LIKE in de PLAPO-projectconventie geaccepteerd is

---

## Validatie-checklist (conform validation-checklist.md)

- [ ] EXITSTATE-betekenis (0 = OK, waarde ^= 0 = fout) bevestigd
- [ ] Maximumkardinaliteit van GRL_SPECIFIEKE_MOGELIJKH bepaald
- [ ] Alle Entity- en Group View-veldlijsten volledig
- [ ] TEXTUM()-functie en LOC_REASON_CODE gedeclareerd
- [ ] Geneste EAB-interface DRUK_FOUTLIJN_MATERIAAL gevalideerd
- [ ] Import-handles beschikbaar in parent-scope
- [ ] INST_OK-veldlengte en padding-logica correct
- [ ] SUBSTR/CONCAT-string-operaties tegen PL/I-semantiek gevalideerd
- [ ] LIKE-statements conform projectconventie
