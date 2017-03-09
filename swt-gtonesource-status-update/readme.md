#Update SFDC Account, Apttus Quote, and Zuora Invoice Statuses from TR GTOneSource

    swt-gtonesource-status-update

##Flows Outline

* <A href="#gtonesrcsoapsvcFlow">gtonesrcsoapsvcFlow</A> - perform audit flow and recognize target system for update (SFDC/APPTUS/ZUORA)

* <A href="#updateAccountStatusToSFDC">updateAccountStatusToSFDC</A> - update global trade account status from GTOneSource to SFDC

* <A href="#updateQuoteStatusToApttus">updateQuoteStatusToApttus</A> - update global trade quote status from GTOneSource to Apttus

* <A href="#ResponseToGTforAccountOrQuote">ResponseToGTforAccountOrQuote</A> -

* <A href="#updateInvoiceStatusToZuora">updateInvoiceStatusToZuora</A> - update global trade invoice status from GTOneSource to Zuora

* <A href="#sendingGTResponseForInvoice">sendingGTResponseForInvoice</A>

* <A href="#amendmentCreation">amendmentCreation</A>

* <A href="#settingInterfaceProperties">settingInterfaceProperties</A>

* <A href="#sendingErrorEmail">sendingErrorEmail</A>

<A name="gtonesrcsoapsvcFlow">
##Start GTOneSource SOAP Service Flow</A>
1. Listen via HTTP on `/gtonesrc`
1. Connect message to proxy service for GTOneSource using `GTONESRCService`
   * *Namespace:* `http://localhost:9142/gtonesrc/service/1.0`  
   * *Service:* `GTONESRCService`
   * *wsdlLocation:* `gtonesource.wsdl`
   * *doc:name* `CXF`  
1. Convert the DOM to XML
1. Set interface properties for request:
   * `transactionLevel <- "Start of the flow"
   * `flowName` <- `${flowName}`
   * `protocol` <- "HTTP"
   * `format` <- "SOAP"
   * `sourceSystemName` <- `${sourceSystemName}`
   * `targetSystemName` <- `${targetSystemName}`
   * `serviceName` <- `${serviceName}`
   * `isAuditReq` <- `${isAuditReq}`
   * `isAuditPayldReq` <- `${isAuditPayldReq}`
1. Preserve original payload in `originalPayload` flow var
1. Perform the [Audit flow] [1]
1. Update the payload with values from the namespace (e.g., `http://localhost:9142/gtonesrc/service/1.0`):
   * `systemId` <- `update.systemid`
   * `Id` <- `update.id`
   * `objectType` <- `update.objecttype`
   * `targetSystem` <- `update.targetsystem`
   * `status` <- `update.status`
   * `statusMessage` <- `update.message`
1. Update the flow vars named as on the left side of the previous step with payload values
1. Based on flow var `targetSystem`, log no message to INFO if not matched or:
   * "SFDC" - perform flow <A href="#updateAccountStatusToSFDC">updateAccountStatusToSFDC</A>
   * "APTTUS" - perform flow <A href="#updateQuoteStatusToApttus">updateQuoteStatusToApttus</A>
   * "ZUORA" - perform flow <A href="#updateInvoiceStatusToZuora">updateInvoiceStatusToZuora</A>

###Error Handling

####Catch Exception Strategy (`java.lang.IllegalStateException`)
1. Execute flow if there is an exception caused via `java.lang.IllegalStateException`
1. Perform <A href="commonServicesExceptionFlow">commonServicesExceptionFlow</A> flow (async)
1. Perform <A href="sendingErrorEmail">sendingErrorEmail</A> flow
1. Perform <A href="ResponseToGTforAccountOrQuote">ResponseToGTforAccountOrQuote</A> flow

####Catch Exception Strategy
1. Perform <A href="commonServicesExceptionFlow">commonServicesExceptionFlow</A> flow (async)
1. Perform <A href="ResponseToGTforAccountOrQuote">ResponseToGTforAccountOrQuote</A> flow

<A name="updateAccountStatusToSFDC">
##Update SFDC Account Status from GTOneSource</A>
*Output:* `application/java`  
1. Transform message from flow variable values:
   * `Id` <- `systemID`
   * `SWT_RPL_Status__c` <- 
      * "Hold" if status is 'Rejected'
      * "Released" if status is 'Verified'
      * otherwise nothing
   * `SWT_RPL_Detail__c` <- `statusMessage` when `status` is "Rejected", otherwise blank
   * `SWT_RPL_Last_Sync__c` <- current date
1. Update SalesForce `Account` object
1. Capture SalesForce update status from payload:
   * `updateStatus` <- `payload[0].success`
   * `updateMessage` <- `null`   
   * `updateMessage` <- `payload[0].errors[0].message` if `payload[0].success` is false
1. if payload[0].success is true, set these message properties:
   * `transactionLevel` <- "RPL Status for Account is Updated in Salesforce"
   * `flowName` <- `${flowName}`
   * `protocol` <- "HTTP"
   * `format` <- "SOAP"
   * `sourceSystemName` <- `${sourceSystemName}`
   * `targetSystemName` <- `${targetSystemName}`
   * `serviceName` <- `${serviceName}`
   * `isAuditReq` <- `${isAuditReq}`
   * `isAuditPayldReq` <- `${isAuditPayldReq}`
   1. Perform [commonServicesAuditFlow] [1]
   1. Perform <A href="#ResponseToGTforAccountOrQuote">ResponseToGTforAccountOrQuote</A> flow  
*otherwise, set these message properties:*
   * `transactionLevel` <- "Failed to update RPL Status for Account in Salesforce"
   * ...others same as above
   1. Perform [commonServicesAuditFlow] [1] (async)
   1. Via Groovy, throw an `IllegalStateException` passing `updateMessage` and return the payload

<A name="updateQuoteStatusToApttus">
##Update Apttus Quote Status from GTOneSource</A>
*Output:* `application/java`  
1. Transform message from flow variable values:
   * `Id` <- `systemID`
   * `SWT_GTS_Status__c` <- 
      * "Hold" if status is 'Rejected'
      * "Released" if status is 'Verified'
      * otherwise nothing
   * `SWT_GTS_Check_Failure_Reason__c` <- `statusMessage` when `status` is "Rejected", otherwise blank
1. Update Apttus `Apttus_Proposal_Propsal__c` object
1. Capture Apttus update status from payload:
   * `updateStatus` <- `payload[0].success`
   * `updateMessage` <- `null`   
   * `updateMessage` <- `payload[0].errors[0].message` if `payload[0].success` is false
1. if payload[0].success is true, set these message properties:
   * `transactionLevel` <- "Quote Status Updated in Apttus"
   * `flowName` <- `${flowName}`
   * `protocol` <- "HTTP"
   * `format` <- "SOAP"
   * `sourceSystemName` <- `${sourceSystemName}`
   * `targetSystemName` <- `${targetSystemName}`
   * `serviceName` <- `${serviceName}`
   * `isAuditReq` <- `${isAuditReq}`
   * `isAuditPayldReq` <- `${isAuditPayldReq}`
   1. Perform [commonServicesAuditFlow] [1] (async)
   1. Perform <A href="#ResponseToGTforAccountOrQuote">ResponseToGTforAccountOrQuote</A> flow  
*otherwise, set these message properties:*
   * `transactionLevel` <- "Failed to update RPL Status for Account in Salesforce"
   * ...others same as above
   1. Perform [commonServicesAuditFlow] [1]
   1. Via Groovy, throw an `IllegalStateException` passing `updateMessage` and return the payload  

<A name="ResponseToGTforAccountOrQuote">
##Send Response to GTOneSource for Acccount or Quote</A>
*Output:* `application/xml`  
*Namespace:* `http://localhost:9142/gtonesrc/service/1.0`  
1. Transform message from flow variable values:
   * `objecttype` <- `objectType`
   * `id` <- `Id`
   * `status` <- `updateStatus`
   * `message` <- `updateMessage`
1. Perform <A href="#settingInterfaceProperties">settingInterfaceProperties</A> flow 

<A name="updateInvoiceStatusToZuora">
##Update Zuora Invoice Status from GTOneSource</A>
*Output:* `application/xml`  
*Namespace:* `http://api.zuora.com`  
*Namespace:* `http://object.api.zuora.com`  
1. Transform message from flow variable values
   * `Id` <- `systemId`
   * `GTSChecksuccessIndicator__c` <- 
      * "Released" if status is 'Verified' 
      * otherwise "Hold"
   * `Status` <- 
      * "Posted" if status is 'Verified'
      * otherwise null
   * `GTResponseDeniedReason__c` <-
      * `statusMessage` if status is 'Rejected'
      * otherwise a space  

1. Invoke SOAP Service
   * *xmlns:zuora* `http://www.mulesoft.org/schema/mule/zuora`
   * *config-ref* `Zuora__Configuration`
   * *soapMetadataKey* `ZuoraService-Soap-http://api.zuora.com/||update||Invoice-zObject`


[1]: https://github.com/lcgillies/TestGitHubPages/tree/dev/CommonServicesWrapper#common-audit-flow
