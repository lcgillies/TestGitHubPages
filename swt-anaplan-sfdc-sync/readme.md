#Synch Anaplan to Salesforce
    swt-anaplan-sfdc-sync

##Process account and sales coverage segments from Anaplan to Salesforce
    swt-anaplan-sfdc-account-salescoveragesegment-sync  
  
1. Poll Anaplan once every 24 hours
1. Export the Account Segmentation model (defined in the `accountsegmentation.modelname` global)
1. Convert **payload** to JSON and log to the Audit flow (async)
1. A null **payload** logs the INFO "No Account and SalescoverageSegment Records are fetched from Anaplan"
1. **Payload** is batch processed in parallel for  
   * Accounts  
   * Sales coverage segments  

   ####Accounts
   1.   Convert JSON to a POJO
   1.   *Update* the following Anaplan model values to the Saleforce `Accounts` object based on ID:
      * `Account ID` -> `ID`
      * `CMT L1` -> `SWT_CMT_1__c`
      * `CMT L2` -> `SWT_CMT_2__c`
      * `Region` -> `SWT_Region__c`
      * `Account Number` -> `AccountNumber`
      * `Accounts` -> `Name`
      * `Business Area Group` -> `SWT_Business_Unit__c`
  
   ####Sales coverage segments
   1.   Convert JSON to a POJO
   1.   *Upsert* the following Anaplan model values to the `Salesforce SWT_Sales_Coverage_Segment__c` object based on `salesSegmentId`:
      * `Sales Segment ID` -> `Id`
      * `Business Area Group` -> `SWT_Business_Area_Group__c`
      * `Customer Segment` -> `SWT_Customer_Segment__c`
      * `Sales Coverage Segment` -> `SWT_Sales_Coverage_Segment_Name__c`
      * `Sales Territory ID` -> ???
      * `Account ID` -> `SWT_Account__c`
  
##Process sales territory from Anaplan to Salesforce
    swt-anaplan-sfdc-salesterriroty-sync
    
1.    
