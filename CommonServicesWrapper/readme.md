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
   
####Common Audit Flow
   commonServicesAuditFlow
       
   1. Log "Audit Invoked" message as INFO
   1. Set the following in the message header:
      * `transactionID` <- `message.id`
      * `timeStamp` <- `current date/time`
      * `serverName` <- `server.host`
      * `userName` <- `server.userName`
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
   
<A name="#Global-Common-Exception-Strategy">
####Global Common Exception Strategy</A>
    globalCommonExceptionStrategy
   1. Set the following in the message header:
      * `transactionID` <- `message.id`
      * `timeStamp` <- `current date/time`
      * `errorMessageDescription` <- `groovey:exception.getMessage()`
      * `serverName` <- `server.host`
      * `userName` <- `server.userName`
   1. Get flow name via Java utilities
   1. Get exception properties via Java utilities
   1. Create the exception message:  
      *Header:*
      * `actionName` <- "Error"
      * `transactionId` <- `outboundProperties.transactionId`
      * `timeStamp` <- `outboundProperties.timeStamp`
      * `sourceSystemName` <- `p('sourceSystemName')`
      * `targetSystemName` <- `p('targetSystemName')`
      * `serviceName` <- `p('serviceName')`
      * `flowName` <- `outboundProperties.flowName`
      * `protocol` <- `outboundProperties.protocol`
      * `format` <- `outboundProperties.format`
      * `userName` <- `outboundProperties.userName`
      * `serverName` <- `outboundProperties.serverName`
      * `recordId` <- "12"
      * `batchId` <- "55555"
      * `businessId` <- "OrderId"  
	*Error:*
      * `exceptionCode` <- `outboundProperties.exceptionCode`
      * `typeOfError` <- `outboundProperties.typeOfError`
      * `errorMessageDescription` <- `outboundProperties.errorMessageDescription`
      * `info` <- `outboundProperties.info`
      * `exceptionTreeList` <- `outboundProperties.exceptionTreeList`
      * `exceptionRecovery` <- `outboundProperties.exceptionRecovery`
      * `originalayload` <- `flowVars.originalPayload`
   1. Convert to string and delete unwanted properties:
      * errorMessageDescription
      * info
      * exceptionTreeList
   1. Perform <A href="#callExceptionAPI">callExceptionAPI</A> flow (async)
   1. Perform <A href="#RemoveExcpPayloadStackTrace">RemoveExcpPayloadStackTrace</A> flow (async)
   
####Common Services Exception Flow
    commonServicesExceptionFlow
   1. Same as <A href="#Global-Common-Exception-Strategy">Global Common Exception Strategy</A> above, except with a Source
   
####Notification SubFlow
    NotificationSub_Flow
   1. Set flow var `emailSubject` to "ESB Alert <serverName><serviceName><flowName>"
   1. Parse `Emailbody.html` template
   1. Get To list from interface
   1. Get default list from `messsage.outboundProperties.defaulttolist`
   1. Log "Sending email" as INFO
   1. Send email via SMTP
      * Settings contain host, port, user and password for SMTP
      * To list uses default if nothing from interface, a static From and the subject from the `emailSubject` flow var
   1. Log "sent email body" and payload as INFO

####Remove Payload Contnet
    RemovePayloadContent
   1. a

####Call Exception API
    callExceptionAPI
   1. a

####Remove Exception Payload Stack Trace
    RemoveExcpPayloadStackTrace
   1. a

   ![swt_CommonServicesWrapper.commonServicesWrapper.commonServicesAuditFlow] [g101]
   
   [g101]: ./assets/swt_CommonServicesWrapper.commonServicesWrapper.commonServicesAuditFlow.png
   
