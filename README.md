# MANAGED-WITH-UNMANAGED-SAVE
MANAGED WITH UNMANAGED SAVE
*****************************INTERFACE VIEW******************************************************************************
@AbapCatalog.viewEnhancementCategory: [#NONE]
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Data Definition for Document Postings'
@Metadata.ignorePropagatedAnnotations: true
@ObjectModel.usageType:{
    serviceQuality: #X,
    sizeCategory: #S,
    dataClass: #MIXED
}
@VDM.viewType: #BASIC
define root view entity ZFI_I_REVENUE_RECOGN_POSTINGS
  as select distinct from I_BillingDocumentBasic as header
    inner join            I_JournalEntry         as bkpf on  header.CompanyCode        = bkpf.CompanyCode
                                                         and header.FiscalYear         = bkpf.FiscalYear
                                                         and header.AccountingDocument = bkpf.AccountingDocument
{
  key  header.BillingDocument      as BillingDocumentNo,
  key  header.CompanyCode          as CompanyCode,
  key  header.SalesOrganization    as SalesOrganization,
  key  header.AccountingDocument   as AccountingDocument,
       header.BillingDocumentDate  as BillingDate,
       bkpf.FiscalYear             as FiscalYear,
       bkpf.AccountingDocumentType as DocumentType
}
*****************************************CONSUMPTION VIEW***********************************************************************
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Consumption View (ZDDE-00102368)'
@Search.searchable: true
@Metadata.allowExtensions: true
@VDM.viewType: #CONSUMPTION
@ObjectModel.usageType:{
    serviceQuality: #X,
    sizeCategory: #S,
    dataClass: #MIXED
}
define root view entity ZFI_C_REVENUE_RECOGN_POSTINGS
  provider contract transactional_query
  as projection on ZFI_I_REVENUE_RECOGN_POSTINGS

{
      @Search.defaultSearchElement: true
      @Search.ranking: #HIGH
  key BillingDocumentNo,
  key CompanyCode,
  key SalesOrganization,
  key AccountingDocument,
      BillingDate,
      FiscalYear,
      DocumentType
}
******************************************METADATA EXTENSION*******************************************************************
@Metadata.layer: #CORE
@UI: {
    headerInfo: { typeName: 'Sales Cutoff - Posting',
                  typeNamePlural: 'Sales Cutoff - Postings',
                  title: { type: #STANDARD, label: 'Sales Cutoff - Posting' }
                          } }
annotate view ZFI_C_REVENUE_RECOGN_POSTINGS with
{

  @UI: { lineItem: [{ position: 30 },
                     { position: 11, type: #FOR_ACTION, dataAction: 'post', label: 'Post', invocationGrouping: #CHANGE_SET }
                      ],
          identification: [{ position: 30 }],
          selectionField: [{position: 30 }]
        }
  @Consumption.filter.multipleSelections: true
  @Consumption.valueHelpDefinition: [ { entity: { name: 'I_BillingDocumentBasicStdVH', element: 'BillingDocument' } } ]
  BillingDocumentNo;
  @UI: { lineItem: [{ position: 20 , label: 'Sales Organization' }],
       identification: [{ position: 20, label: 'Sales Organization'}],
       selectionField: [{position: 20 }]
       }
  @Consumption.filter.multipleSelections: true
  @Consumption.valueHelpDefinition: [ { entity: { name: 'I_SalesOrganizationValueHelp', element: 'SalesOrganization' } } ]
  SalesOrganization;
  @UI: { lineItem: [{ position: 10 , label: 'Company Code' }],
       identification: [{ position: 10, label: 'Company Code' }],
       selectionField: [{position: 10 }] }

  @Consumption.valueHelpDefinition: [ { entity: { name: 'I_CompanyCodeStdVH', element: 'CompanyCode' } } ]
  @Consumption.filter:{mandatory: true }
  @Consumption.filter.multipleSelections: true
  CompanyCode;
  @UI: { lineItem: [{ position: 40 , label: 'Document Type' }],
       identification: [{ position: 40, label: 'Document Type'}],
       selectionField: [{position: 40 }]
       }
  @Consumption.filter.multipleSelections: true
  DocumentType;
  @UI: { lineItem: [{ position: 50 , label: 'Fiscal Year' }],
       identification: [{ position: 50, label: 'Fiscal Year'}],
       selectionField: [{position: 50 }]
       }
  @Consumption.valueHelpDefinition: [{ entity: { element: 'CalendarYear',name: 'I_CalendarYear'  },
  useForValidation: true }]
  FiscalYear;
  @UI: { lineItem: [{ position: 70 , label: 'BillingDate' }],
       identification: [{ position: 70, label: 'BillingDate'}],
       selectionField: [{position: 70 }] }
  @Consumption.filter:{mandatory: true }
  @Consumption.filter.multipleSelections: true
  BillingDate;

}
************************************ACCESS CONTROL***************************************************************
@EndUserText.label: 'Access Control (ZDDE-00102368)'
@MappingRole: true
define role ZFI_I_REVENUE_RECOGN_POSTINGS {
    grant
        select
            on
                ZFI_I_REVENUE_RECOGN_POSTINGS
                    where
                       (CompanyCode) = aspect pfcg_auth(f_bkpf_buk , bukrs, actvt = '03');
                        
}
***************************************************************************************************************
@EndUserText.label: 'Accces Control - GL SingDraft'
@MappingRole: true
define role ZFI_I_GLACCT_SD_QUERY {
    grant
        select
            on
                ZFI_I_GLACCT_SD_QUERY
                    where
                       ( ) = aspect pfcg_auth(Z_CDS_VIEW, ACTVT = '03', DDLNAME = 'ZFI_I_GLACCT_SD_QUERY'); 
}
**************************************BEHAVIOR DEFINITION**********************************************************
managed implementation in class zbp_fi_i_revenue_recogn_postin unique;
strict ( 2 );

define behavior for ZFI_I_REVENUE_RECOGN_POSTINGS alias Post
with unmanaged save
lock master
authorization master ( instance )
{
  field ( mandatory  ) CompanyCode;

  action ( features : instance ) post parameter zfi_a_acctype result [1] $self;
}
*******************************************BEHAVIOR CONSUMPTION******************************************************
projection;
strict ( 2 );

define behavior for ZFI_C_REVENUE_RECOGN_POSTINGS
{
  use action post;
}
******************************************BEHAVIOR POOL CLASS******************************************************
*---------------------------------------------------------------------*
* Development ID : ZDDE-00102368                                      *
* Functional Spec: ZFSE-00102367                                      *
*---------------------------------------------------------------------*
* Behavior Definition Implementation Class for Recognition Posting App*
*---------------------------------------------------------------------*
* Change Log:                                                         *
*---------------------------------------------------------------------*
* Init     |  Name     |     Date   |     CR       |  Text            *
*---------------------------------------------------------------------*
* CHINTRE3  Reshma Ch    02-Nov-2025  CR: 1100276898  Initial version *
*---------------------------------------------------------------------*
CLASS lhc_post DEFINITION INHERITING FROM cl_abap_behavior_handler FINAL.

  PRIVATE SECTION.

    METHODS get_instance_features FOR INSTANCE FEATURES
      IMPORTING keys REQUEST requested_features FOR post RESULT result.

    METHODS get_instance_authorizations FOR INSTANCE AUTHORIZATION
      IMPORTING keys REQUEST requested_authorizations FOR post RESULT result.

    METHODS post FOR MODIFY
      IMPORTING keys FOR ACTION post~post RESULT result.

ENDCLASS.

CLASS lhc_post IMPLEMENTATION.

  METHOD get_instance_features.

    READ ENTITIES OF zfi_i_revenue_recogn_postings IN LOCAL MODE
     ENTITY post ALL FIELDS WITH
     CORRESPONDING #( keys )
     RESULT DATA(lt_post)
     FAILED DATA(lt_failed).

    result = VALUE #( FOR ls_post IN lt_post
                      LET post_ok = COND #(  WHEN ls_post-accountingdocument NE ''
                                             THEN if_abap_behv=>fc-o-enabled
                                             ELSE if_abap_behv=>fc-o-disabled )
                                             IN
                                             ( %tky = ls_post-%tky
                                               %action-post = post_ok  ) ).


  ENDMETHOD.

  METHOD get_instance_authorizations.

    CONSTANTS: lc_f_bkpf_buk TYPE xuobject   VALUE 'F_BKPF_BUK',
               lc_bukrs      TYPE fieldname  VALUE 'BUKRS',
               lc_actvt      TYPE fieldname  VALUE 'ACTVT',
               lc_02         TYPE activ_auth VALUE '02'.

    READ ENTITIES OF zfi_i_revenue_recogn_postings IN LOCAL MODE
    ENTITY post ALL FIELDS WITH
    CORRESPONDING #( keys )
    RESULT DATA(lt_rev)
    FAILED DATA(lt_failed).
* Check Authorization
    LOOP AT lt_rev ASSIGNING FIELD-SYMBOL(<fs_rev>).
      AUTHORITY-CHECK OBJECT lc_f_bkpf_buk
       ID lc_actvt FIELD lc_02
       ID lc_bukrs FIELD <fs_rev>-companycode.

      IF sy-subrc = 0.
        DATA(lv_update) = abap_true.
      ENDIF.

      APPEND VALUE #( LET  upd_auth = COND #( WHEN lv_update = abap_false
                                              THEN if_abap_behv=>auth-unauthorized
                                              ELSE if_abap_behv=>auth-allowed )
                                               IN
                               %tky            = <fs_rev>-%tky
                               %update         = upd_auth
                               %action-post    = upd_auth
                            ) TO result.
    ENDLOOP.

  ENDMETHOD.

  METHOD post.

    DATA: lt_message     TYPE bapiret2_t,
          lt_post_header TYPE zfi_tt_post_docs,
          lt_head        TYPE zfi_tt_post_docs,
          lr_bukrs       TYPE RANGE OF bukrs,
          lr_vbeln       TYPE RANGE OF vbeln,
          lv_billing_doc TYPE vbeln,
          lr_gjahr       TYPE RANGE OF gjahr,
          lt_posted      TYPE STANDARD TABLE OF zfi_t_posteddocs.

    CONSTANTS: lc_tabname TYPE tabname VALUE 'ZFI_T_POSTEDDOCS',
               lc_e       TYPE char01 VALUE 'E',
               lc_s       TYPE char01 VALUE 'S',
               lc_prog    TYPE repid  VALUE 'ZFI_FM_SALESCUTOFF_POST',
               lc_blart   TYPE zconst VALUE 'BLART',
               lc_msgid   TYPE symsgid VALUE 'ZFSE_00102367',
               lc_000     TYPE symsgno VALUE '000',
               lc_005     TYPE symsgno VALUE '005',
               lc_008     TYPE symsgno VALUE '008',
               lc_009     TYPE symsgno VALUE '009',
               lc_co      TYPE zfi_de_acc_type VALUE 'CO',
               lc_td      TYPE zfi_de_acc_type VALUE 'TD'.

* Fetching Account Type as CO/TD
    DATA(lv_acc_type) = VALUE #( keys[ 1 ]-%param-zacc_type ).

* Fetching Posting Date
    DATA(lv_posting_date) = VALUE #( keys[ 1 ]-%param-budat ).

    IF lv_acc_type EQ lc_co OR lv_acc_type EQ lc_td.
      IF lv_posting_date IS NOT INITIAL.

        READ ENTITIES OF zfi_i_revenue_recogn_postings IN LOCAL MODE
        ENTITY post
        ALL FIELDS
        WITH CORRESPONDING #( keys )
        RESULT DATA(lt_post)
        FAILED DATA(lt_failed).

        lt_post_header = CORRESPONDING #(  lt_post ).

        IF lt_post_header IS INITIAL.
          APPEND VALUE #( %key = keys[ 1 ]-%key
                          %msg = new_message( id = lc_msgid
                                              number = lc_000
                                              severity = if_abap_behv_message=>severity-error )
                                             ) TO reported-post.
        ELSE.
* Company Code
          lr_bukrs = VALUE #( FOR ls_post IN lt_post_header
                              ( sign    = if_fsbp_const_range=>sign_include
                                option  = if_fsbp_const_range=>option_equal
                                low     = ls_post-companycode ) ).

* Billing Document No
          lr_vbeln = VALUE #( FOR ls_post IN lt_post_header
                              ( sign    = if_fsbp_const_range=>sign_include
                                option  = if_fsbp_const_range=>option_equal
                                low     = ls_post-billingdocumentno ) ).

* Fiscal Year
          lr_gjahr = VALUE #( FOR ls_post IN lt_post_header
                              ( sign    = if_fsbp_const_range=>sign_include
                                option  = if_fsbp_const_range=>option_equal
                                low     = ls_post-fiscalyear ) ).

* Fetching data from custom table to validate posting is already done
          SELECT FROM zfi_i_posted_docs
          FIELDS vbeln,
                 bukrs,
                 gjahr,
                 belnrfirev
          WHERE vbeln IN @lr_vbeln
          AND bukrs IN @lr_bukrs
          AND gjahr IN @lr_gjahr
          INTO TABLE @DATA(lt_revposted).

* Fetching data from ZCONSTANTS table
          SELECT *
          FROM zconstants
          WHERE repid = @lc_prog
          INTO TABLE @DATA(lt_const).

* Fetching Document type 'RH'
          DATA(lv_doc_type) = VALUE #( lt_const[ const = lc_blart ]-value OPTIONAL ).

          LOOP AT lt_post_header ASSIGNING FIELD-SYMBOL(<lfs_header>).

            APPEND VALUE #( billingdocumentno = <lfs_header>-billingdocumentno
                            companycode       = <lfs_header>-companycode
                            salesorganization = <lfs_header>-salesorganization
                            accountingdocument = <lfs_header>-accountingdocument
                            billingdate       = <lfs_header>-billingdate
                            fiscalyear        = <lfs_header>-fiscalyear
                            documenttype      = <lfs_header>-documenttype ) TO lt_head.

            lv_billing_doc = <lfs_header>-billingdocumentno.

            DATA(lv_rev_doc) = VALUE #( lt_revposted[ vbeln = <lfs_header>-billingdocumentno
                                                      bukrs = <lfs_header>-companycode
                                                      gjahr = <lfs_header>-fiscalyear ]-belnrfirev OPTIONAL ).

* Proceeding further if reversal/posting is empty in custom table
            IF lv_rev_doc IS INITIAL.

              CALL FUNCTION 'ZFI_FM_SALESCUTOFF_POST' DESTINATION 'NONE'
                EXPORTING
                  iv_billing_doc        = lv_billing_doc
                  iv_acc_type           = lv_acc_type
                  it_post_header        = lt_head
                  iv_posting_date       = lv_posting_date
                IMPORTING
                  et_message            = lt_message
                EXCEPTIONS
                  system_failure        = 1
                  communication_failure = 2
                  OTHERS                = 3.
              IF sy-subrc IS NOT INITIAL.
                APPEND VALUE #( %key = keys[ 1 ]-%key
                         %msg = new_message_with_text( severity = if_abap_behv_message=>severity-error
                                                       text     = TEXT-000 )
                                                      ) TO reported-post.
                APPEND VALUE #( %key = keys[ 1 ]-%key  )  TO failed-post.
              ELSE.
* Displaying Success message with Document number generated via BAPI
                ASSIGN lt_message[ type = lc_s ] TO FIELD-SYMBOL(<lfs_message>) ELSE UNASSIGN.
                IF <lfs_message> IS ASSIGNED.
                  APPEND VALUE #( %key = keys[ 1 ]-%key
                  %msg = new_message_with_text( severity = if_abap_behv_message=>severity-information
                                                text     = <lfs_message>-message && <lfs_message>-message_v2 )
                                               ) TO reported-post.

* Filling data in Custom Table
                  APPEND VALUE #( vbeln = <lfs_header>-billingdocumentno
                                  blart = lv_doc_type
                                  bukrs = <lfs_header>-companycode
                                  belnr = <lfs_header>-accountingdocument
                                  gjahr = <lfs_header>-fiscalyear
                                  budat = lv_posting_date
                                  belnr_fi_rev = <lfs_message>-message_v2+0(10)
                                  gjahr_rev = <lfs_message>-message_v2+14(4)
                                  zacc_type = lv_acc_type
                                  createdby = cl_fdt_obj_system_variables=>get_current_user( )
                                  createdon = cl_fdt_obj_system_variables=>get_current_date( )
                                  ) TO lt_posted.
                ENDIF.
              ENDIF.
            ELSE.
* Document has already Posted
              APPEND VALUE #( %key = keys[ 1 ]-%key
              %msg = new_message( id       = lc_msgid
                                  number   = lc_005
                                  v1       = <lfs_header>-billingdocumentno
                                  v2       = lv_rev_doc
                                  severity = if_abap_behv_message=>severity-error ) ) TO reported-post.
              APPEND VALUE #( %key = keys[ 1 ]-%key  )  TO failed-post.
            ENDIF.
            CLEAR: lt_head,lv_rev_doc.
          ENDLOOP.

          IF lt_posted IS NOT INITIAL.
* Setting Lock
            CALL FUNCTION 'ENQUEUE_E_TABLE'
              EXPORTING
                mode_rstable   = lc_e
                tabname        = lc_tabname
              EXCEPTIONS
                foreign_lock   = 1
                system_failure = 2
                OTHERS         = 3.
            IF sy-subrc = 0.
* Saving records
              MODIFY zfi_t_posteddocs FROM TABLE lt_posted.
            ENDIF.
* Releasing Lock
            CALL FUNCTION 'DEQUEUE_E_TABLE'
              EXPORTING
                mode_rstable = lc_e
                tabname      = lc_tabname.
          ENDIF.

* Sending Error messages
          LOOP AT lt_message ASSIGNING FIELD-SYMBOL(<lfs_error>) ELSE UNASSIGN.
            IF <lfs_error>-type = lc_e .
              APPEND VALUE #( %key = keys[ 1 ]-%key
              %msg = new_message_with_text(
                                       severity = if_abap_behv_message=>severity-error
                                       text     = <lfs_error>-message ) ) TO reported-post.
              APPEND VALUE #( %key = keys[ 1 ]-%key  )  TO failed-post.
            ENDIF.
          ENDLOOP.
        ENDIF.
      ELSE.
* Please Enter Posting Date
        APPEND VALUE #( %key = keys[ 1 ]-%key
              %msg = new_message( id       = lc_msgid
                                  number   = lc_009
                                  severity = if_abap_behv_message=>severity-error ) ) TO reported-post.
        APPEND VALUE #( %key = keys[ 1 ]-%key  )  TO failed-post.
      ENDIF.
    ELSE.
* Invalid Account Type. Select from Value Help.
      APPEND VALUE #( %key = keys[ 1 ]-%key
      %msg = new_message( id       = lc_msgid
                          number   = lc_008
                          v1       = lv_acc_type
                          severity = if_abap_behv_message=>severity-error ) ) TO reported-post.
      APPEND VALUE #( %key = keys[ 1 ]-%key  )  TO failed-post.
    ENDIF.

  ENDMETHOD.

ENDCLASS.

CLASS lsc_zfi_i_revenue_recogn_posti DEFINITION INHERITING FROM cl_abap_behavior_saver FINAL.
  PROTECTED SECTION.

    METHODS save_modified REDEFINITION.

    METHODS cleanup_finalize REDEFINITION.

ENDCLASS.

CLASS lsc_zfi_i_revenue_recogn_posti IMPLEMENTATION.

  METHOD save_modified.
  ENDMETHOD.

  METHOD cleanup_finalize.
  ENDMETHOD.

ENDCLASS.
