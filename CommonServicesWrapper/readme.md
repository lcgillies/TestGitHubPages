#Common Services Wrapper
    swt_CommonServicesWrapper
    
##Flows Outline
   * commonServicesAuditFlow
      * RemovePayloadContent
      
   * globalCommonExceptionStrategy
      * callExceptionAPI (Async)
      * RemoveExcpPayloadStackTrace (Async)
   
   * commonServicesExceptionFlow
      * callExceptionAPI (Async)
      * RemoveExcpPayloadStackTrace (Async)

   * NotificationSub_Flow
   
   * RemovePayloadContent
   
   * callExceptionAPI
   
   * RemoveExcpPayloadStackTrace
   
   ##Common Audit Flow
       commonServicesAuditFlow
       
   1. 
