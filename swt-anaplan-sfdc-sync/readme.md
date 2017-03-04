 
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
   1.   Update the following Anaplan model values to the Saleforce Account object based on ID:
      * Account ID -> ID
      * CMT L1 -> SWT_CMT_1__c
      * CMT L2 -> SWT_CMT_2__c
      * Region -> SWT_Region__c
      * Account Number -> AccountNumber
      * Accounts -> Name
      * Business Area Group -> SWT_Business_Unit__c
  
   ####Sales coverage segments
   1.   Convert JSON to a POJO
   1.   Upsert the following Anaplan model values to the Salesforce SWT_Sales_Coverage_Segment__c object based on salesSegmentId:
      * Sales Segment ID -> Id
      * Business Area Group -> SWT_Business_Area_Group__c
      * Customer Segment -> SWT_Customer_Segment__c
      * Sales Coverage Segment -> SWT_Sales_Coverage_Segment_Name__c
      * Sales Territory ID -> ???
      * Account ID -> SWT_Account__c
