#Update SFDC Account, Apttus Quote, and Zuora Invoice Statuses from TR GTOneSource

    swt-gtonesource-status-update

##Flows Outline

* <A href="#gtonesrcsoapsvcFlow">gtonesrcsoapsvcFlow</A> - perform audit flow and recognize target system for update (SFDC/APPTUS/ZUORA)

* <A href="#updateAccountStatusToSFDC>updateAccountStatusToSFDC</A> - 

* <A href="#updateQuoteStatusToApttus>updateQuoteStatusToApttus</A> -

* <A href="#ResponseToGTforAccountOrQuote">ResponseToGTforAccountOrQuote</A> -

* <A href="#updateInvoiceStatusToZuora">updateInvoiceStatusToZuora</A>

* <A href="#sendingGTResponseForInvoice">sendingGTResponseForInvoice</A>

* <A href="#amendmentCreation">amendmentCreation</A>

* <A href="#settingInterfaceProperties>settingInterfaceProperties</A>

* <A href="#sendingErrorEmail>sendingErrorEmail</A>

<A name="gtonesrcsoapsvcFlow">
##Start GTOneSource SOAP Service Flow</A>
1. Listen via HTTP on `/gtonesrc`
1. Connect message to proxy service for GTOneSource using `GTONESRCService`
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
1. Transform message:
   * Id <- flowVars.systemID
   * SWT_RPL_Status__c <- 
      * "Hold" if flowVars.status is 'Rejected'
      * "Released" if flowVars.status is 'Verified'
      * otherwise nothing
   * SWT_RPL_Detail__c <- flowVars.statusMessage when .status is "Rejected", otherwise blank
   * SWT_RPL_Last_Sync__c <- current date


[1]: https://github.com/lcgillies/TestGitHubPages/tree/dev/CommonServicesWrapper#common-audit-flow
