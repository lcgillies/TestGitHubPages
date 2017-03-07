#Update SFDC Account, Apttus Quote, and Zuora Invoice Statuses from TR GTOneSource

    swt-gtonesource-status-update

##Flows Outline

* <A href="#gtonesrcsoapsvcFlow">gtonesrcsoapsvcFlow</A> - perform audit flow and recognize target system for update (SFDC/APPTUS/ZUORA)

* updateAccountStatusToSFDC

* updateQuoteStatusToApttus

* ResponseToGTforAccountOrQuote

* updateInvoiceStatusToZuora

* sendingGTResponseForInvoice

* amendmentCreation

* settingInterfaceProperties

* sendingErrorEmail

<A name="gtonesrcsoapsvcFlow">
##Start GTOneSource SOAP Service Flow</A>
