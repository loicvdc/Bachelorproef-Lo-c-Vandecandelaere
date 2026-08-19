# Vertaling: Olieverwerking (Opmerking "Haal oliegegevens") 10,3 credits

## CA Gen code

NOTE *** Haal oliegegevens ***
     -------------------------

SET loc_toep wz06151 cod_toep TO "PLNBAF"
SET loc_mat_input wz06151 cod_ni_wr_ro TO "PSKN"
SET loc_mat_input wz06151 rt_hfd_ro TO ro rt_hfd_ro
SET loc_bs_input wz06151 cod_frm_gg_po_bs TO imp mataf cod_frm_gg_po_bs
SET loc_bs_input wz06151 nr_bs TO imp mataf nr_bs
SET loc_bs_input wz06151 nr_po_bs TO imp mataf nr_po_bs
SET loc_bs_input wz06151 nr_gg_prd TO 1
SET loc_bs_input wz06151 nr_ty_prd_alt TO 1

USE eab_oproep_tpolie
    WHICH IMPORTS: Work View  loc_toep wz06151 TO Work View  imp_toep wz06151
                   Work View  loc_mat_input wz06151 TO Work View  imp_mat_input wz06151
                   Work View  loc_bs_input wz06151 TO Work View  imp_bs_input wz06151
                   Work View  loc_mat_output wz06151 TO Work View  imp_mat_output wz06151
                   Work View  loc_bs_output wz06151 TO Work View  imp_bs_output wz06151
                   Work View  loc_dos_pc wz06151 TO Work View  imp_dos_pc wz06151
    WHICH EXPORTS: Work View  loc_toep wz06151 FROM Work View  exp_toep wz06151
                   Work View  loc_mat_input wz06151 FROM Work View  exp_mat_input wz06151
                   Work View  loc_bs_input wz06151 FROM Work View  exp_bs_input wz06151
                   Work View  loc_mat_output wz06151 FROM Work View  exp_mat_output wz06151
                   Work View  loc_bs_output wz06151 FROM Work View  exp_bs_output wz06151
                   Work View  loc_dos_pc wz06151 FROM Work View  exp_dos_pc wz06151
                   <none> FROM Work View  exp_brk_emul wz06151
                   Work View  loc_int_error_handling FROM Work View  exp_error_int_error_handling

## Module: EAB_OPROEP_TPOLIE
### Bron: CA Gen Action Diagram fragment (comment: Haal oliegegevens)

---

## 1. Declaraties

Alle werkview-structuren die in het fragment worden gebruikt:

```pli
/*
 * Lokale werkviews voor olie-verwerking (WZ06151 context)
 */
DCL 1 LOC_TOEP_WZ06151,
      2 COD_TOEP         CHAR(8);

DCL 1 LOC_MAT_INPUT_WZ06151,
      2 COD_NI_WR_RO     CHAR(4),
      2 RT_HFD_RO        <DATATYPE_UNKNOWN>;

DCL 1 LOC_BS_INPUT_WZ06151,
      2 COD_FRM_GG_PO_BS CHAR(4),
      2 NR_BS            FIXED BIN(31),
      2 NR_PO_BS         FIXED BIN(31),
      2 NR_GG_PRD        FIXED BIN(31),
      2 NR_TY_PRD_ALT    FIXED BIN(31);

DCL 1 LOC_MAT_OUTPUT_WZ06151;
       2 <FIELDS_UNKNOWN>;

DCL 1 LOC_BS_OUTPUT_WZ06151;
       2 <FIELDS_UNKNOWN>;

DCL 1 LOC_DOS_PC_WZ06151;
       2 <FIELDS_UNKNOWN>;

DCL 1 LOC_INT_ERROR_HANDLING,
      2 EXITSTATE        FIXED BIN(31);

/*
 * Import-structuren (aangenomen mappin naar relevante entiteiten)
 */
DCL 1 H_MATAF,               /* Materiaal-affine entiteit */
      2 COD_FRM_GG_PO_BS CHAR(4),
      2 NR_BS            FIXED BIN(31),
      2 NR_PO_BS         FIXED BIN(31);

DCL 1 H_RO,                  /* Rollen/processen */
      2 RT_HFD_RO        <DATATYPE_UNKNOWN>;

/*
 * Export-structuren van EAB_OPROEP_TPOLIE (interface aangenomen)
 */
DCL 1 IMP_TPOLIE,
      2 IMP_TOEP_WZ06151      LIKE LOC_TOEP_WZ06151,
      2 IMP_MAT_INPUT_WZ06151 LIKE LOC_MAT_INPUT_WZ06151,
      2 IMP_BS_INPUT_WZ06151  LIKE LOC_BS_INPUT_WZ06151,
      2 IMP_MAT_OUTPUT_WZ06151 LIKE LOC_MAT_OUTPUT_WZ06151,
      2 IMP_BS_OUTPUT_WZ06151 LIKE LOC_BS_OUTPUT_WZ06151,
      2 IMP_DOS_PC_WZ06151    LIKE LOC_DOS_PC_WZ06151;

DCL 1 EXP_TPOLIE,
      2 EXP_TOEP_WZ06151      LIKE LOC_TOEP_WZ06151,
      2 EXP_MAT_INPUT_WZ06151 LIKE LOC_MAT_INPUT_WZ06151,
      2 EXP_BS_INPUT_WZ06151  LIKE LOC_BS_INPUT_WZ06151,
      2 EXP_MAT_OUTPUT_WZ06151 LIKE LOC_MAT_OUTPUT_WZ06151,
      2 EXP_BS_OUTPUT_WZ06151 LIKE LOC_BS_OUTPUT_WZ06151,
      2 EXP_DOS_PC_WZ06151    LIKE LOC_DOS_PC_WZ06151,
      2 EXP_ERROR_HANDLING    LIKE LOC_INT_ERROR_HANDLING,
      2 EXITSTATE             FIXED BIN(31);
```

---

## 2. Procedurebody

```pli
/*
 * *** Haal oliegegevens ***
 */

/* Initialiseer input-structuren */
LOC_TOEP_WZ06151.COD_TOEP = "PLNBAF";

LOC_MAT_INPUT_WZ06151.COD_NI_WR_RO = "PSKN";
LOC_MAT_INPUT_WZ06151.RT_HFD_RO = H_RO.RT_HFD_RO;

LOC_BS_INPUT_WZ06151.COD_FRM_GG_PO_BS = H_MATAF.COD_FRM_GG_PO_BS;
LOC_BS_INPUT_WZ06151.NR_BS            = H_MATAF.NR_BS;
LOC_BS_INPUT_WZ06151.NR_PO_BS         = H_MATAF.NR_PO_BS;
LOC_BS_INPUT_WZ06151.NR_GG_PRD        = 1;
LOC_BS_INPUT_WZ06151.NR_TY_PRD_ALT    = 1;

/*
 * Roep EAB aan voor oliegegevens-verwerking
 * (View Matching: alle loc_* werken zijn gekoppeld aan corresponderende imp_/exp_ views)
 */
CALL OPROEP_TPOLIE(IMP_TPOLIE, EXP_TPOLIE);

/*
 * Foutafhandeling nach EAB-call
 */
IF EXP_TPOLIE.EXITSTATE ^= 0 THEN
    DO;
        LOC_INT_ERROR_HANDLING.EXITSTATE = EXP_TPOLIE.EXITSTATE;
        RETURN;
    END;

/* Vervolg procedurelogica kan hier volgen */
```

---

## Vertaalnotities

1. **Work View naamgeving**: 
   - `loc_toep wz06151` → `LOC_TOEP_WZ06151` (conform glossary-patroon `LOC_<FUNCTIE>_WZ<CODE>`)
   - Evenzo voor alle `loc_*` werk views

2. **Structuur-veldmapping**:
   - CA Gen-syntax `loc_toep wz06151 cod_toep` vertaald naar `LOC_TOEP_WZ06151.COD_TOEP`
   - Veldnamen geheel naar uppercase (conform PLAPO-standaard)

3. **Import-references** (bv. `imp mataf cod_frm_gg_po_bs`):
   - Aangenomen dat deze verwijzen naar entiteitsvelden die in de import-scope beschikbaar zijn
   - In dit voorbeeld gemapt naar `H_MATAF.COD_FRM_GG_PO_BS` (H = "Handle"/"Header", MATAF = materiaal-affine entiteit)
   - Dit patroon is afgeleid van het worked-example; exacte mapping hangt af van het import-model

4. **EAB-call interface**:
   - CA Gen `USE eab_oproep_tpolie` → PL/I `CALL OPROEP_TPOLIE(IMP_TPOLIE, EXP_TPOLIE);`
   - Aangenomen dat alle IMPORTS in één `IMP_*`-struct en alle EXPORTS in één `EXP_*`-struct passen
   - Alternatief: als EAB meer parameters verwacht, moet interface worden vastgesteld

5. **Exitstate-afhandeling**:
   - Na CALL wordt `EXP_TPOLIE.EXITSTATE` gecontroleerd
   - Niet-nul waarde triggert error-handling (conform error-handling-conventions)

---

## Openstaande vragen

1. **Veldtypes in LOC_MAT_INPUT_WZ06151 en H_RO**:
   - `RT_HFD_RO` wordt hier als `<DATATYPE_UNKNOWN>` gemarkeerd
   - Dit veld moet uit het CA Gen-datamodel worden bepaald (waarschijnlijk `FIXED BIN(31)` of `CHAR(n)`)
   - **Actie**: Datatype uit Encyclopedia overnemen

2. **Uitvoervelden in LOC_MAT_OUTPUT_WZ06151, LOC_BS_OUTPUT_WZ06151, LOC_DOS_PC_WZ06151**:
   - Deze structuren hebben geen velden gespecificeerd in het CA Gen-fragment (alleen imports/exports worden gezet)
   - **Actie**: Volledige veldlijsten uit de CA Gen-werkview-definities bepalen

3. **Import-entiteitsmapping**:
   - `imp mataf`, `imp mat`, `ro` worden hier aangenomen als globale import-handles (H_MATAF, H_RO)
   - **Actie**: Bevestigen dat deze handles in de omroepende procedure beschikbaar zijn, en dat hun structuurdefiniënties overeenkomen

4. **EAB-interface validatie**:
   - De aangenomen `CALL OPROEP_TPOLIE(IMP_TPOLIE, EXP_TPOLIE)` moet tegen de werkelijke EAB-interface (PL/I header) gecontroleerd worden
   - Het is mogelijk dat de EAB meer/minder parameters verwacht, of andere parameterstructuren vereist
   - **Actie**: Bestaande PL/I-module `OPROEP_TPOLIE` raadplegen (zie chatmode sectie 4: "Wanneer escaleren")

5. **View Matching semantiek**:
   - Het CA Gen-statement bevat 6 IMPORTS en 7 EXPORTS (waarvan 1 `<none>`)
   - Dit suggereert complexe view-koppeling; het is onduidelijk of één `IMP_`/`EXP_`-struct volstaat of dat afzonderlijke parameters nodig zijn
   - **Actie**: Shared-modules-glossary en bestaande EAB-interface controleren

6. **Veldlengtes**:
   - Hardcoded lengtes in deze vertaling (bv. `CHAR(8)` voor COD_TOEP, `CHAR(4)` voor COD_NI_WR_RO) zijn aannames
   - **Actie**: Exacte lengtes uit CA Gen-datamodel bepalen

---

## Validatie-checklist (conform validation-checklist.md)

- [ ] Alle velden in LOC_* en H_* structuren hebben datatypes vastgesteld
- [ ] Import-entities (mataf, ro, etc.) zijn in context beschikbaar
- [ ] EAB-interface OPROEP_TPOLIE is gevalideerd tegen bestaande PL/I-module
- [ ] Exitstate-waarde 0 = OK, niet-nul = fout (projectconventie bevestigd)
- [ ] Veldlengtesen types conform Encyclopedia (niet geraden)
- [ ] RETURN na error-handling is correct (geen exit-code-propagatie nodig?)
