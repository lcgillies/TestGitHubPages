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
   
<A href="commonServicesAuditFlow">
####Common Audit Flow</A>
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

![swt_CommonServicesWrapper.commonServicesWrapper.commonServicesAuditFlow] [g101]

<A name="globalCommonExceptionStrategy">
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

![swt_CommonServicesWrapper.commonServicesWrapper.globalCommonExceptionStrategy] [g102]

<A name="commonServicesExceptionFlow">
####Common Services Exception Flow</A>
    commonServicesExceptionFlow
   1. Same as <A href="#globalCommonExceptionStrategy">Global Common Exception Strategy</A> above, except with a Source

![swt_CommonServicesWrapper.commonServicesWrapper.commonServicesExceptionFlow] [g103]

<A name="NotificationSub_Flow">
####Notification SubFlow</A>
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

![swt_CommonServicesWrapper.commonServicesWrapper.NotificationSub_Flow] [g104]

<A name="RemovePayloadContent">
####Remove Payload Contnet</A>
    RemovePayloadContent
   1. Hash the payload JSON using java.util.HashMap
   1. Build an expression for `message.payload.Auditing.originalPayload` setting it to the `message.payload`
   1. Convert java object to JSON

![swt_CommonServicesWrapper.commonServicesWrapper.RemovePayloadContent] [g105]

<A name="callExceptionAPI">
####Call Exception API</A>
    callExceptionAPI
   1. Post to /Exception via `CommonDBInsert_HTTP_Request_Configuration`
   1. Convert byte array to string, log payload as INFO

![swt_CommonServicesWrapper.commonServicesWrapper.callExceptionAPI] [g106]

<A name="RemoveExcpPayloadStackTrace">
####Remove Exception Payload Stack Trace</A>
    RemoveExcpPayloadStackTrace
   1. Run string to byte array
   1. Hash the payload JSON using java.util.HashMap
   1. Build an expression for `message.payload.Auditing.originalPayload` setting it to the `message.payload`
   1. Convert java object to JSON
   1. Send exception metadata (payload) to console as ERROR

![swt_CommonServicesWrapper.commonServicesWrapper.RemoveExcpPayloadStackTrace] [g107]
   
[g101]: ./assets/swt_CommonServicesWrapper.commonServicesWrapper.commonServicesAuditFlow.png
[g102]: ./assets/swt_CommonServicesWrapper.commonServicesWrapper.globalCommonExceptionStrategy.png
[g103]: ./assets/swt_CommonServicesWrapper.commonServicesWrapper.commonServicesExceptionFlow.png
[g104]: ./assets/swt_CommonServicesWrapper.commonServicesWrapper.NotificationSub_Flow.png
[g105]: ./assets/swt_CommonServicesWrapper.commonServicesWrapper.RemovePayloadContent.png
[g106]: ./assets/swt_CommonServicesWrapper.commonServicesWrapper.callExceptionAPI.png
[g107]: ./assets/swt_CommonServicesWrapper.commonServicesWrapper.RemoveExcpPayloadStackTrace.png
