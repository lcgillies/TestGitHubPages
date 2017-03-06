#Synch Anaplan to Salesforce

    swt-anaplan-sfdc-sync
   
##Flow Outline

* global.xml - contains environment properties (anaplan cert, sfdc oauth, smtp connector)

* swt-anaplan-sfdc-account-salescoveragesegment-sync.xml - process account and sales coverage segment changes
   * processAccountandSalescoveragesegmentsync (synch)
      * **EXT**: [commonServicesAuditFlow] [1]
      * processAcountBatchProcessing (ORDERED_SEQUENTIAL by 100)
         * sfdcFailuresupport_Subflow (Process Records)
         * FailedRecordsFlow (On Complete)
      * processSalescoveragesegmentBatchProcessing (ORDERED_SEQUENTIAL by 100)
         * sfdcFailuresupport_Subflow (Process Records)
         * FailedRecordsFlow (On Complete)
      
* swt-anaplan-sfdc-salesterritory-sync.xml - process sales territory updates
   * processSalesterritorysyncflow (synch)
      * **EXT**: commonServicesAuditFlow
      * processSalesterritorySyncBatchProcessing (ORDERED_SEQUENTIAL by 100)
         * Map_Salesterritory_Batch_Step (Process Records)
            * swt-anaplan-sfdc-salesterritorySub_Flow
            * sfdcFailuresupport_Subflow (Process Records)
            * FailedRecordsFlow (On Complete)
      
* supporting_flow.xml - support flows
   * loggingSupportFlow
   * GlobalExceptionStrategy
      * Transformation Exception
      * Connection Exception Strategy
      * InternalProcessingException Strategy
   * FailedRecordsFlow
   * RealtimeEmailFlow
   * BatchEmailFlow
   * sfdcFailuresupport_Subflow

##Process account and sales coverage segments from Anaplan to Salesforce
    swt-anaplan-sfdc-account-salescoveragesegment-sync  
  
1. Poll Anaplan every 1 day
1. Export the Account Segmentation model (defined in the `accountsegmentation.modelname` global)
1. Convert **payload** to JSON and log to the [Audit flow] [1] (async)
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
   1.   Set **payload** to any failed records
   1.   Build record count message as a variable
   1.   Perform <A href="#FailureSupport">Failed Records</A> flow
  
   ####Sales coverage segments
   1.   Convert JSON to a POJO
   1.   *Upsert* the following Anaplan model values to the Salesforce `SWT_Sales_Coverage_Segment__c` object based on `salesSegmentId`:
      * `Sales Segment ID` -> `Id`
      * `Business Area Group` -> `SWT_Business_Area_Group__c`
      * `Customer Segment` -> `SWT_Customer_Segment__c`
      * `Sales Coverage Segment` -> `SWT_Sales_Coverage_Segment_Name__c`
      * `Sales Territory ID` -> ???
      * `Account ID` -> `SWT_Account__c`
   1.   Set **payload** to any failed records
   1.   Build record count message as a variable
   1.   Perform <A href="#FailureSupport">Failed Records</A> flow

  
##Process sales territory from Anaplan to Salesforce
    swt-anaplan-sfdc-salesterritory-sync
    
1. Export the Account Segmentation model (defined in the `salesterriroty.modelname` global)
1. Convert **payload** to JSON and log to the Audit flow (async)
1. A null **payload** logs the INFO "No SalesTerritory Records found in Anaplan"
1. Convert JSON to a POJO
1. Look for the Territory ID in `SWT_Sales_Territory__c`
1. A null **payload** logs the INFO "No existing record"
1. *UPSERT* the following Anaplan model values to the Salesforce `SWT_Sales_Territory__c` object based on `Id`
  * `Territory ID` -> `Id`
  * `Sales Territories` -> `SWT_Sales_Territory_Name`
  * `Sales Territories` -> `SWT_Sales_Territory_Name`
  * `Region` -> `SWT_Territory_Region__c`
  * `Country` -> `SWT_Territory_Country__c`
  * `Start Date` -> blank if null or blank, otherwise express in local time and increment (?) by 13.5 hours
  * `End Date` ->  blank if null or blank, otherwise express in local time and increment (?) by 13.5 hours
1. Log an INFO message containing the payload
1. Set **payload** to any failed records
1. Build record count message as a variable
1. Perform <A href="#FailureSupport">Failed Records</A> flow


##Shared processes
    supporting_flow

<A name="FailureSupport">
####Failure Support
</A>
   1.   Test for payload success---if false, log the errors and queue a failure to Failure_Out
   1.   Pick up Failure_Out queue, test for records
   1.   If Failure_Out records exist, map the following values to a CSV and batch email 
      * `RecordId` <- `$.RecordId`
      * `Exception` <- `$.message`
      * `StatusCode`<- `$.statusCode`
      * `BatchId` <- `sessionVars.CorrelationID`
      * `Status` <- `"Error"`
      * `ApplicationName` <- `p('name')`
      * `TargetSystem` <- `p('target')`
      * `SourceSystem` <- `p('source')`

[1]: https://github.com/lcgillies/TestGitHubPages/edit/dev/CommonServicesWrapper/
