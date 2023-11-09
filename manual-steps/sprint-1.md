# Sprint 1 - Manual Steps

## Pre-deployment steps

### [SOC-559](https://employinc.atlassian.net/browse/SOC-559)

Install Nebula Logger unlocked package:

Sandbox - https://test.salesforce.com/packaging/installPackage.apexp?p0=04t5Y000001Mjx5QAC

PROD - https://login.salesforce.com/packaging/installPackage.apexp?p0=04t5Y000001Mjx5QAC

When installing, select `Install for Admins Only` and in Advanced Options select `Compile only the Apex in the package`.

### [SOC-570](https://employinc.atlassian.net/browse/SOC-570)

Install Trigger Handler unlocked package:

Sandbox - https://test.salesforce.com/packaging/installPackage.apexp?p0=04t08000000ga52AAA

PROD - https://login.salesforce.com/packaging/installPackage.apexp?p0=04t08000000ga52AAA

When installing, select `Install for Admins Only` and in Advanced Options select `Compile only the Apex in the package`.

## Post-deployment steps

### [SOC-559](https://employinc.atlassian.net/browse/SOC-559)

Execute following Apex code to create default settings for Nebula Logger:

```java
insert new LoggerSettings__c(
    SetupOwnerId = UserInfo.getOrganizationId(),
    DefaultLogOwner__c = null,
    DefaultLogPurgeAction__c = 'Delete',
    DefaultLogShareAccessLevel__c = 'Read',
    DefaultNumberOfDaysToRetainLogs__c = 14,
    DefaultPlatformEventStorageLocation__c = 'CUSTOM_OBJECTS',
    DefaultPlatformEventStorageLoggingLevel__c = null,
    DefaultSaveMethod__c = 'EVENT_BUS',
    DefaultScenario__c = null,
    EndTime__c = null,
    IsAnonymousModeEnabled__c = false,
    IsApexSystemDebugLoggingEnabled__c = false,
    IsDataMaskingEnabled__c = false,
    IsEnabled__c = true,
    IsJavaScriptConsoleLoggingEnabled__c = false,
    IsRecordFieldStrippingEnabled__c = false,
    IsSavingEnabled__c = true,
    LoggingLevel__c = 'FINEST',
    StartTime__c = null
);
```
