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
       
   1. Log "Audit Invoked" message as INFO
   1. Set the following in the message header:
      * transactionID <- message.id
      * timeStamp <- current date/time
      * serverName <- server.host
      * userName <- server.userName
   1. Test message.outboundProperties.isAuditReq for "Y", log outboundProperties.test as INFO if not
   1. If audit is required
      1. Create audit message:  
            *Header:*
	 * `actionName` <- "Audit"
	 * `transactionId` <- `outboundProperties.transactionId`
	 * `timeStamp` <- `outboundProperties.timeStamp`
	 * `sourceSystemName` <- `outboundProperties.sourceSystemName`
	 * `targetSystemName` <- `outboundProperties.targetSystemName`
	 * `serviceName` <- `outboundProperties.serviceName`
	 * `flowName` <- `outboundProperties.flowName`
	 * `protocol` <- `outboundProperties.protocol`
	 * `format` <- `outboundProperties.format`				
	 * `userName` <- `outboundProperties.userName`
	 * `serverName` <- `outboundProperties.serverName`								
	 * `recordId` <- "12",
	 * `batchId` <- "55555",  						
	 * `businessId` <- "OrderId"  
   	    *Auditing:*
	 * `transactionLevel` <- `outboundProperties.transactionLevel`
	 * `originalPayload` <- `flowVars.originalPayload`
   1. Test message.outboundProperties.isAuditPayldReq for "N", post to /Audit via `CommonDBInsert_HTTP_Request_Configuration`
   1. Perform <A href="#RemovePayloadContent">RemovePayloadContent</A> flow
   
   ![swt_CommonServicesWrapper.commonServicesWrapper.commonServicesAuditFlow] [g101]
   
   [g101]: ./assets/swt_CommonServicesWrapper.commonServicesWrapper.commonServicesAuditFlow.png
   
