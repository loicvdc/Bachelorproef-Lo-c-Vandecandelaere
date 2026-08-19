# Voorbeelden van vertalingen door expert

## CA Gen voorbeeld 1

FOR SUBSCRIPT OF grl_mat_prdo FROM 1 TO MAX OF grl_mat_prdo BY 1
    SET gloc_prdo int_pln_plnprdo_productie_order id_bs_prd TO 0
    SET gloc_prdo int_mat_materiaaleenheid nummer TO 0
    SET gloc_prdo int_mat_materiaaleenheid codering_identificatienummer TO SPACES
    SET gloc_update ief_supplied flag TO SPACES

## PL/I vertaling voorbeeld 1

DCL 1 INT_PLN_PLNPRDO_PRODUCTIE_ORDER,
    2 ID_BS_PRD FIXED              BIN(31);
DCL 1 INT_MAT_MATERIAALEENHEID,
    2 NUMMER FIXED                 BIN(31),
    2 CODERING_IDENTIFICATIENUMMER CHAR(7);
DCL FLAG                           CHAR(8);

DO GRL_MAT_PRDO = 1 TO MAX(GRL_MAT_PRDO) BY 1;
    INT_PLN_PLNPRDO_PRODUCTIE_ORDER.ID_BS_PRD = 0;
    INT_MAT_MATERIAALEENHEID.NUMMER = 0; 
    INT_MAT_MATERIAALEENHEID.CODERING_IDENTIFICATIENUMMER = '';
    FLAG = '';
END;

## CA Gen voorbeeld 2

WHILE nr_vkm IS LESS OR EQUAL TO q_vkm
    REPEAT
        SET SUBSCRIPT OF grl_ttis TO 1
        SET loc_exp_mat nr_bs_af TO gloc_tti nr_bs
        SET loc_exp_mat nr_po_bs_aff TO gloc_tti nr_po_bs
        
        IF frm_afd_prd IS NOT EQUAL TO “SID”
        ESCAPE

## PL/I vertaling voorbeeld 2

/* WHILE */
DO WHILE (INT_AFF_WERKVELDEN.NR_VKM <= INT_AFF_WERKVELDEN.Q_VKM);
/* REPEAT */
    GRL_TTIS = 1;
    INT_AFF_MATERIAALEENHEID.NR_BS_AFF    = INT_AFF_TTI.NR_BS;
    INT_AFF_MATERIAALEENHEID.NR_PO_BS_AFF = INT_AFF_TTI.NR_PO_BS;
        
    IF INT_AFF_MATERIAALEENHEID.FRM_AFD_PRD ^= “SID”
    THEN
    /* ESCAPE */
        LEAVE;
END;

## CA Gen voorbeeld 3

USE ab_lpl_app_pp_sgm_plnprdo
    WHICH IMPORTS: Work View  loc_comm sid_werkveld TO Work View imp_comm sid_werkveld
    Group View grl_prdo TO Group View gri_prdo
    WHICH EXPORTS: <none> FROM Work View exp int_error_handling

## PL/I vertaling voorbeeld 3

CALL(SID_WERKVELD, GRI_PRDO);

## CA Gen voorbeeld 4

READ matwr
SORTED BY ASCENDING matwr dt_wr_eh_mat
SORTED BY ASCENDING matwr tyd_wr_eh_mat
SORTED BY ASCENDING matwr tms_atk_wr
WHERE DESIRED matwr wordt_uitgevoerd_op CURRENT imp mat
 AND DESIRED matwr ty_wr_eh_mat IS EQUAL TO “P”

## PL/I vertaling voorbeeld 4

SELECT *
FROM MATWR JOIN MAT
WHERE MATWR.TY_WR_EH_MAT = 'P'
SORT BY MATWR.DT_WR_EH_MAT ASC,
        MATWR.TYD_WR_EH_MAT ASC,
        MATWR.TMS_ATK_WR ASC;

## CA Gen voorbeeld 5

SET loc wz56161 ser_frm_afd TO "SID"
SET loc wz56161 cod_afd TO "GLO"
SET loc wz56161 cod_frm_gg_ro TO "S"
SET loc wz56161 cod_nr_ro TO imp int_mat_materiaaleenheid codering_identificatienummer
SET loc wz56161 nr_ro TO imp int_mat_materiaaleenheid nummer
SET loc wz56161 nr_bs_gr_po_fam TO imp mataf nr_bs
SET loc wz56161 nr_po_bs_gr_po_fam TO imp mataf nr_po_bs

USE eab_oproep_tpbafgg
    WICH IMPORTS: Work View loc wz56161 TO Work View imp wz56161
    WHICH EXPORTS: Work View loc wz56161 FROM Work View exp wz56161

    IF loc wz56161 cod_tg IS NOT EQUAL TO 0
    
    USE eab_druk_foutlijn_bestelling
        WHICH IMPORTS: Group View grl_foutlijn TO Group View gri_foutlijn
    ESCAPE

## PL/I vertaling voorbeeld 5

WZ56161                    = '';
WZ56161.SER_FRM_AFD        = 'SID';
WZ56161.COD_AFD            = 'GLO';
WZ56161.COD_FRM_GG_RO      = 'S';
WZ56161.COD_NR_RO          = H_MAT.COD_NR;
WZ56161.NR_RO              = H_RO.NR_RO;
WZ56161.NR_BS_GR_PO_FAM    = H_MATAF.NR_BS;
WZ56161.NR_PO_BS_GR_PO_FAM = H_MATAF.NR_PO_BS;

CALL TPBAFGG(WZ56161);

IF WZ56161.COD_TG != 0
    CALL FOUT(WZ56161.COD_TG);

## CA Gen voorbeeld 6

EXIT STATE IS sid_verwerking_ab_gestart_rb WITH ROLLBACK

READ mtstbo
    WHERE SOME mat IS CURRENT imp ro
    AND DESIRED mtstbo cod_frm_gg_eh_mat IS EQUAL TO THAT mat cod_frm_gg
    AND DESIRED mtstbo nr_eh_mat_dl IS EQUAL TO THAT mat cod_nr_eh_mat
    AND DESIRED mtstbo nr_eh_mat_dl IS EQUAL TO THAT mat nr_eh_mat
    AND DESIRED mtstbo cod_prc_cre_dl_mat IS EQUAL TO CURRENT imp ro 
    cod_prc_l_ro
    AND SOME matwr is_laatste_proces_voor CURRENT imp ro
    AND THAT matwr nr_wr_eh_mat IS EQUAL TO DESIRED mtstbo 
    nr_prc_cre_dl_mat
    AND DESIRED mtstbo ty_dl_mat IS EQUAL TO "M"

WHEN successful
    WHILE mtstbo cod_eh_mat_dl IS NOT EQUAL TO "W"
        SET loc_imp_mat056 wz54601 cod_frm_gg TO "S"
        SET loc_imp_mat056 wz54601 cod_nr_ro TO "K"
        SET loc_imp_mat056 wz54601 nr_ro TO mtstbo nr_eh_mat_dl
        SET loc_imp_mat056 wz54601 nr_prc_ro TO mtstbo nr_prc_cre_dl_mat
        SET loc_imp_mat056 wz54601 dt_cre_ro TO mtstbo dt_cre_eh_mat_dl
        USE eab_oproep_mat056_lees_prcskin
            WHICH IMPORTS: Work View loc_imp_mat056 wz54601 TO Work View
             imp wz54601
            WHICH EXPORTS: Work View loc_exp_mat056 wz54601 FROM Work 
            View exp wz54601
                             Work View loc_exp_ind_mat056 
                             mat_ind_wz54601 FROM Work View 
                             mat_ind_wz54601
                             Work View loc_exp_wz30911 FROM Work View 
                             exp wz30911

        IF loc_exp_wz30911 cod_tg IS EQUAL TO 1
            AND loc_exp_wz30911 cod_rd IS EQUAL TO 0

* * * * SET gloc_mat sid_werkveld text80 TO concat("Oproep 
eab_oproep_mat055_lees_prcskin gelukt Return: "
* * * * SET gloc_mat_ft int_mat_materiaaleenheid code_frma TO 
loc_imp_mat056 wz54601 cod_frm_gg
* * * * SET gloc_mat_ft int_mat_materiaaleenheid 
codering_identificatienummer TO loc_imp_mat056 wz54601 cod_
* * * * SET gloc_mat_ft int_mat_materiaaleenheid nummer TO 
loc_imp_mat056 wz54601 nr_ro
* * * * USE eab_druk_foutlijn_materiaal
* * * *     WHICH IMPORTS: Group View grl_foutlijn_mat TO Group 
View gri_foutlijn_mat

            IF exp_stamboom_gg int_bsn_bstuyprd pct_lg_skp_na IS 
            EQUAL TO 0
                IF loc_exp_ind_mat056 mat_ind_wz54601 i_lg_skp IS 
                EQUAL TO -1
                ELSE
                    SET exp_stamboom_gg int_bsn_bstuyprd pct_lg_skp_na 
                    TO loc_exp_mat056 wz54601 lg_skp

            IF exp_stamboom_gg int_bsn_bstuyprd brk_emul_skp IS EQUAL 
            TO SPACES
                SET exp_stamboom_gg int_bsn_bstuyprd brk_emul_skp TO 
                loc_exp_mat056 wz54601 emul_brk_skp

        IF mtstbo nr_dl_moe IS NULL
            ESCAPE

        READ mtstbo
            WHERE DESIRED mtstbo cod_dl_mat IS EQUAL TO CURRENT 
            mtstbo cod_dl_moe
            AND DESIRED mtstbo nr_dl_mat IS EQUAL TO CURRENT 
            mtstbo nr_dl_moe

        WHEN successful
        WHEN not found
            ESCAPE

WHEN not found
    EXIT STATE IS mtstbo_nf WITH ROLLBACK

ESCAPE

## PL/I vertaling voorbeeld 6

BEPAAL_SKP_GEGS_VIA_MTSTBO:
PROC;

  #TRACE_ENTRY;

  /* Init variabelen */
  LG_SKP = 0;
  BRK_EMUL_SKP = '';

  H_MTSTBO.COD_FRM_GG_EH_MAT  = H_RO_IMP.COD_FRM_GG_RO;
  H_MTSTBO.COD_NR_EH_MAT_DL   = H_RO_IMP.COD_NR_RO;
  H_MTSTBO.NR_EH_MAT_DL       = H_RO_IMP.NR_RO;
  H_MTSTBO.COD_PRC_CRE_DL_MAT = H_RO_IMP.COD_PRC_L_RO;
  H_MTSTBO.NR_PRC_CRE_DL_MAT  = H_RO_IMP.NR_PRC_L_RO;
  H_MTSTBO.TY_DL_MAT          = 'M';  /* MATDEEL */

  CALL SQL_OPEN_CURSOR('MTSTBO_MATDL_PRC_L');
  CALL SQL_FETCH_CURSOR('MTSTBO_MATDL_PRC_L');

  SCAN_LAATSTE_PROCES:
    DO WHILE(SQLCA.SQLCODE = 0);

    VERWERK_MAT_DEEL:
      DO WHILE(SQLCA.SQLCODE = 0 & H_MTSTBO.COD_NR_EH_MAT_DL = 'K');

        CALL OPROEP_MAT056;

        IF WZ30911.COD_TG = 1 & WZ30911.COD_RD = 0
        THEN
        DO;
          IF LG_SKP = 0
          THEN
            IF WZ54601.GRP_EXP_IND.LG_SKP ^= -1
            THEN
              LG_SKP = WZ54601.GRP_EXP.LG_SKP;

          IF BRK_EMUL_SKP = ''
          THEN
            BRK_EMUL_SKP = WZ54601.GRP_EXP.EMUL_BRK_SKP;
        END;

        IF I_MTSTBO.NR_DL_MOE >= 0
        THEN
          CALL SELECT_MTSTBO_MOEDER;
        ELSE
          LEAVE VERWERK_MAT_DEEL;

    END VERWERK_MAT_DEEL;

    CALL SQL_FETCH_CURSOR('MTSTBO_MATDL_PRC_L');

    END SCAN_LAATSTE_PROCES;

    CALL SQL_CLOSE_CURSOR('MTSTBO_MATDL_PRC_L');