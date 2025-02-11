# sap-get-external-api
Get data from external API in SAP ABAP

*&---------------------------------------------------------------------*
*& Report ZABAP_HTTP
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
REPORT zabap_http.

TRY.
* Get data from external API
* Define API url
  /
  `
    DATA(lv_url) = |https://airport-info.p.rapidapi.com/airport?iata=CGK|.
    DATA : o_client TYPE REF TO if_http_client.
  `

* Create HTTP object
    cl_http_client=>create_by_url( EXPORTING
                                       url = lv_url
                                    IMPORTING
                                       client = o_client
                                    EXCEPTIONS
                                       argument_not_found = 1
                                       plugin_not_active  = 2
                                       internal_error     = 3
                                       OTHERS             = 4 ).
    IF SY-SUBRC <> 0.
      o_client->close( ).
    ENDIF.
    
    IF o_client IS BOUND.
* Set HTTP method
      o_client->request->set_method( if_http_request=>co_request_method_get ).

* Set Header fields
      o_client->request->set_header_field( name  = 'x-rapidapi-host'
                                           value = 'airport-info.p.rapidapi.com' ).
      o_client->request->set_header_field( name  = 'x-rapidapi-key'
                                           value = 'f61c245171msh6444c033ee60464p1cb66cjsnd2548ee06057' ).

* Set timeout
      o_client->send( timeout = if_http_client=>co_timeout_default ).

* Read response
      o_client->receive( ).
      
      DATA : lv_http_status TYPE i.
      o_client->response->get_status( IMPORTING
                                         code = lv_http_status ).
      
      IF lv_http_status = 200.
        DATA(lv_result) = o_client->response->get_cdata( ).
      ENDIF.
      
      o_client->close( ).
    ENDIF.

    CATCH cx_root INTO DATA(e_txt).
      WRITE : / 'Terjadi kesalahan :', e_txt->get_text( ).
  ENDTRY.

* Declaration of variable and types
  TYPES : BEGIN OF ty_data,
            id   TYPE int4, 
            iata TYPE string,
            name TYPE string,
            location TYPE string,
            county TYPE string,
            state TYPE string,
            country TYPE string,
            website TYPE string,
          END OF ty_data.

   DATA : lt_nav TYPE STANDARD TABLE OF ty_data WITH HEADER LINE.

* JSON->ABAP
   /ui2/cl_json=>deserialize( EXPORTING
                                json        = lv_result
                                pretty_name = /ui2/cl_json=>pretty_mode-camel_case
                              CHANGING
                                data = lt_nav ).
   
   cl_demo_output=>write_data( lt_nav-id ).
   cl_demo_output=>write_data( lt_nav-iata ).
   cl_demo_output=>write_data( lt_nav-name ).
   cl_demo_output=>write_data( lt_nav-location ).
   cl_demo_output=>write_data( lt_nav-county ).
   cl_demo_output=>write_data( lt_nav-state ).
   cl_demo_output=>write_data( lt_nav-country ).
   cl_demo_output=>write_data( lt_nav-website ).
   cl_demo_output=>display( lt_nav ).
                                
