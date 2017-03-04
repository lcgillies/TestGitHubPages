##Process account and sales coverage segments from Anaplan to Salesforce

###swt-anaplan-sfdc-sync

1. Once a day, poll Anaplan
1. Export the Account Segmentation model (which one is defined in the {accountsegmentation.modelname} global
1. Payload is converted to JSON and logged to the Audit flow (async)
1. A null payload logs the INFO "No Account and SalescoverageSegment Records are fetched from Anaplan"
1. Content is batch processed in parallel for  
   * Accounts  
   * Sales coverage segments  

   ####Accounts
   1.   Convert JSON to a POJO
   1.   Load the following Anaplan model values to the Saleforce Account object:
      * Account ID -> ID
      * CMT L1 -> SWT_CMT_1__c
      * CMT L2 -> SWT_CMT_2__c
      * Region -> SWT_Region__c
      * Account Number -> AccountNumber
      * Accounts -> Name
      * Business Area Group -> SWT_Business_Unit__c
