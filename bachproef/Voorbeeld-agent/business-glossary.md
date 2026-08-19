# Business glossary: CA Gen → PL/I

Deze glossary bevat de project-specifieke naamgeving die in de bronlogica en daarmee in de vertaling wordt gebruikt. De entries zijn bedoeld als vaste vertaalbasis voor de agent en moeten per project worden bevestigd met de datamatrix, maar ze vormen hier de concrete basis voor de hervertaling.

## 1. Procedure- en exitstate-namen

| CA Gen | PL/I | Opmerking |
|---|---|---|
| sid_verwerking_ab_gestart_rb | SID_VERWERKING_AB_GESTART_RB | Procedure/exitstate naam |
| mtstbo_nf | MTSTBO_NF | Not found exitstate |
| sid_werkveld | SID_WERKVELD | Werkveld-naam in lokale opslag |

## 2. Entiteiten en recordnamen

| CA Gen | PL/I | Opmerking |
|---|---|---|
| mtstbo | MTSTBO | Stamboom/tabel voor materiaal-/procesgegevens |
| mat | MAT | Materialen-/elementtabel |
| matwr | MATWR | Werkrecord / procesrecord voor materiaal |
| imp_ro | IMP_RO | Import-structuur voor proces-/rolcontext |
| exp_ro | EXP_RO | Export-structuur voor procesresultaat |
| exp_stamboom_gg | EXP_STAMBOOM_GG | Extern stamboom-gebied |
| int_bsn_bstuyprd | INT_BSN_BSTUYPRD | Interne bedrijfsstructuur |

## 3. Veld- en attribuutnamen

| CA Gen | PL/I | Opmerking |
|---|---|---|
| cod_frm_gg_eh_mat | COD_FRM_GG_EH_MAT | Formule-/groepcode voor materiaal |
| nr_eh_mat_dl | NR_EH_MAT_DL | Nummer materiaal- en detailrecord |
| cod_frm_gg | COD_FRM_GG | Code formulier-/groepsveld |
| cod_nr_eh_mat | COD_NR_EH_MAT | Code voor identificatie materiaal |
| cod_prc_cre_dl_mat | COD_PRC_CRE_DL_MAT | Code voor creatieproces materiaal |
| ty_dl_mat | TY_DL_MAT | Type detailmateriaal |
| cod_dl_mat | COD_DL_MAT | Code detailmateriaal |
| nr_dl_mat | NR_DL_MAT | Nummer detailmateriaal |
| cod_dl_moe | COD_DL_MOE | Code detailmoeder |
| nr_dl_moe | NR_DL_MOE | Nummer detailmoeder |
| cod_eh_mat_dl | COD_EH_MAT_DL | Materiaal-/detailcode |
| nr_prc_cre_dl_mat | NR_PRC_CRE_DL_MAT | Procesnummer voor creatie |
| dt_cre_eh_mat_dl | DT_CRE_EH_MAT_DL | Creatiedatum materiaal-detail |
| cod_tg | COD_TG | Type-/statuscode |
| cod_rd | COD_RD | Reden-/resultaatcode |
| pct_lg_skp_na | PCT_LG_SKP_NA | Percentage legging/skip |
| brk_emul_skp | BRK_EMUL_SKP | Break-emulatie skip |
| i_lg_skp | I_LG_SKP | Indirecte lengte/skip indicator |
| lg_skp | LG_SKP | Lengte/skip |
| emul_brk_skp | EMUL_BRK_SKP | Emulatie break-skip |
| cod_prc_l_ro | COD_PRC_L_RO | Code van huidige proces-/rol |
| nr_wr_eh_mat | NR_WR_EH_MAT | Werkrecordnummer materiaal |

## 4. Work View / lokale werkvelden

| CA Gen | PL/I | Opmerking |
|---|---|---|
| loc_imp_mat056_wz54601 | LOC_IMP_MAT056_WZ54601 | Input work view voor MAT056 |
| loc_exp_mat056_wz54601 | LOC_EXP_MAT056_WZ54601 | Export work view voor MAT056 |
| loc_exp_ind_mat056 | LOC_EXP_IND_MAT056 | Indicator-export voor MAT056 |
| loc_exp_wz30911 | LOC_EXP_WZ30911 | Export work view voor WZ30911 |
| mat_ind_wz54601 | MAT_IND_WZ54601 | Indicator-arbeidsstructuur voor WZ54601 |
| wz54601 | WZ54601 | Output-/regelstructuur voor materiaalverwerking |
| wz30911 | WZ30911 | Resultaatstructuur voor detailcontrole |
| gloc_mat | GLOC_MAT | Globale tekstbuffer |
| gloc_mat_ft | GLOC_MAT_FT | Formaat-/tekstbuffer voor uitvoer |

## 5. EAB / module-naamgeving

| CA Gen | PL/I | Opmerking |
|---|---|---|
| eab_oproep_mat056_lees_prcskin | EAB_OPROEP_MAT056_LEES_PRCSKIN | EAB voor lezen van proces-skin |
| eab_druk_foutlijn_materiaal | EAB_DRUK_FOUTLIJN_MATERIAAL | EAB voor foutlijnuitvoer materiaal |
| grl_foutlijn_mat | GRL_FOUTLIJN_MAT | Groepview / lijst foutlijnen materiaal |
| gri_foutlijn_mat | GRI_FOUTLIJN_MAT | Geïnstanceerde lijst foutlijnen materiaal |

## 6. Transacties en logische operators

| CA Gen | PL/I | Opmerking |
|---|---|---|
| IS EQUAL TO | = | Vergelijking |
| IS NOT EQUAL TO | ^= | Niet-gelijk |
| IS NULL | = '' of NULL | Afhankelijk van datatype en SQL-variabele |
| CONCAT(...) | || | Stringconcatenering |
| EXIT STATE IS ... WITH ROLLBACK | EXIT_STATE = ...; ROLLBACK; RETURN; | Projectconventie |

## 7. Voorbeeld-vertaling uit de bron

- `mtstbo cod_frm_gg_eh_mat` → `MTSTBO.COD_FRM_GG_EH_MAT`
- `loc_exp_wz30911 cod_tg` → `LOC_EXP_WZ30911.COD_TG`
- `exp_stamboom_gg int_bsn_bstuyprd pct_lg_skp_na` → `EXP_STAMBOOM_GG.INT_BSN_BSTUYPRD.PCT_LG_SKP_NA`
- `loc_exp_mat056 wz54601 lg_skp` → `LOC_EXP_MAT056_WZ54601.LG_SKP`
- `loc_exp_mat056 wz54601 emul_brk_skp` → `LOC_EXP_MAT056_WZ54601.EMUL_BRK_SKP`

Deze glossary is de basis voor de hervertaling in de agent en moet bij verdere bouwstappen alleen worden aangevuld met projectspecifieke ontbrekende datamodellering of interface-structuren.