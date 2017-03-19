# Synch Product (PPA) to Apttus

    swt-onprem-apttus-product-sync
   
## Flows Outline

* swt-onprem-attus-product-sync-main
   * HTTP: swt-onprem-attus-product-sync-httpListenerConfig
      * Path: /api/*
      * Allowed: POST
   * APIkit Router: swt-onprem-apttus-product-sync-config
   
* post:/:swt-onprem-apttus-product-sync-config
   * swt-onprem-apttus-product-syncSub_Flow
   
* swt-onprem-apttus-product-syncFlow
   * HTTP:
      * Path: /upsert
      * Allowed: POST
   * swt-onprem-apttus-product-syncSub_Flow

* swt-onprem-apttus-product-sync-apiKitGlobalExceptionMapping
   * 404
   * 405
   * 415
   * 406
   * 400
   
* swt-apptus-onprem-product-syncFlow
   * swt-onprem-apttus-product-syncBatch
   
* swt-onprem-apttus-product-syncBatch
   * swt-onprem-apttus-product-syncBatch
