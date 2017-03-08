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
1. Perform commonServicesAuditFlow
