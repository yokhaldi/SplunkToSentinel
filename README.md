Overview:

Azure-Sentinel Add-On allows Azure Log Analytics and Sentinel users to ingest all security logs from Splunk platform using the Azure HTTP Data Collector API, more information can be found here https://docs.microsoft.com/en-us/azure/azure-monitor/platform/data-collector-api. 

Follow the setup and configuration steps in the 'Details' tab of this add-on to use it.
Release Notes
Version 1.0.2
Oct. 20, 2020

Details:

When you add data to Splunk, the Splunk indexer processes it and stores it in a designated index (either, by default, in the main index or in the one that you identify). Searching in Splunk involves using the indexed data for the purpose of creating metrics, dashboards and alerts.

Let’s assume that your security team wants to collect data from Splunk platform to use Azure Sentinel log analytics as their centralized log analytics platform. In order to implement this scenario, you can rely on data stored in Splunk index then create a scheduled custom alert to push this data to the Azure Sentinel API. 

In order to send data from Splunk to Azure Sentinel log analytics, the idea was to use the HTTP Data Collector API, more information can be found here https://docs.microsoft.com/en-us/azure/azure-monitor/platform/data-collector-api. You can use the HTTP Data Collector API to send log data to a Log Analytics workspace from any client that can call a REST API.
All data in the Log Analytics workspace is stored as a record with a particular record type. You format your data to send to the HTTP Data Collector API as multiple records in JSON. When you submit the data, an individual record is created in the repository for each record in the request payload.

Based on Splunk Add-on Builder here https://splunkbase.splunk.com/app/2962/, we created an add-on which trigger an action based on the alert in Splunk. You can use Alert actions to define third-party integrations (like Azure Sentinel log analytics).

Splunk Add-on Builder uses Python code to create your alert action, here is the code we used within the Add-on: https://docs.microsoft.com/en-us/azure/azure-monitor/platform/data-collector-api#python-3-sample

To make this simple we have created an Add-on for you to use. You need just to install it in your Splunk platform.

Let’s start the configuration!


Preparation & Use
The following tasks describe the necessary preparation and configurations steps.
•	Onboarding of Splunk instance (latest release),can be found here https://docs.splunk.com/Documentation/Splunk/8.0.5/Installation/Beforeyouinstall
•	Get the Log Analytics workspace parameters: Workspace ID and Primary Key from here https://docs.microsoft.com/en-us/azure/azure-monitor/platform/log-analytics-agent#workspace-id-and-key
•	Install the Azure Sentinel Add-on for Splunk 


Onboard Azure Sentinel
Onboarding of Azure Sentinel is not part of this blog post, however required guidance can be found here https://docs.microsoft.com/en-us/azure/sentinel/quickstart-onboard.

Add-on Installation in Splunk Enterprise

First download the Azure-Sentinel-add-on-1.0.2 Add-on from Splunk base which allows you to ingest Splunk logs from. Use the following steps to install the app in Splunk.
 
Login with provided login credentials (username / password) during the installation of Splunk.

In Splunk portal click to Manage Apps

In Manage Apps click to Install app from file and use the downloaded file TA-sentinel2.tar before for the installation, and click Upload.

Preparation Steps in Splunk
https://docs.splunk.com/Documentation/Splunk/8.0.6/Alert/DefineRealTimeAlerts

As mentioned above, we will use Splunk alerts to send logs to Azure Sentinel. To validate the integration, as example, we will use the audit index as example “_audit”, this repository stores events from the file system change monitor, auditing, and all user search history.

You can query the data by using index=”_audit” in search field.
 

Then use a scheduled or real-time alert to monitor events or event patterns as they happen. You can create real-time alerts with per-result triggering or rolling time window triggering. Real-time alerts can be costly in terms of computing resources, so consider using a scheduled alert when possible.

Set up alert actions, which help you respond to triggered alerts. You can enable one or more alert actions. In our case, select “send_to_sentinel” action, it was added when we installed the Azure-Sentinel add-on

Fill in the required parameters:

•	Customer_id: Azure Sentinel log analytics Workspace ID 
•	Shared_key: Azure Sentinel log analytics Primary Key
•	Log_Type: Azure Sentinel custom log name

Note that these parameters are required and will be used by the application to send data to Azure Sentinel through the HTTP Data Collector API.


View Splunk Data in Azure Sentinel

The results will be added to a custom Azure Sentinel table called ‘Splunk_Audit_Events_CL’.
It can take few minutes for events to be available.

 
With the logs now in Azure Sentinel you can easily query the data

Splunk_Audit_Events_CL 
 | summarize count() by user_s, action_s
 | render barchart 

 


When a correlation search included in the Splunk Enterprise Security (or added by a user) identifies an event or pattern of events, it creates an incident called notable event. Correlation searches filter the IT security data and correlate across events to identify a particular type of incident (or pattern of events) and then create notable events.
Correlation searches run at regular intervals (for example, every hour) or continuously in real-time and search events for a particular pattern or type of activity. The notable event is stored in a dedicated notable index.

You can import all notable events into Azure Sentinel, we will use the same procedure described above.
The results will be added to a custom Azure Sentinel table called ‘Splunk_Notable_Events_CL’.
 

You can easily query Splunk incidents in Azure Sentinel:

Splunk_Notable_Events_CL 
| extend Origin_Time= extract("([0-9]{2}\\/[0-9]{2}\\/[0-9]{4} [0-9]{2}:[0-9]{2}:[0-9]{2})", 0, orig_raw_s )
| project TimeGenerated=Origin_Time , search_name_s, threat_description_s, risk_object_type_s


 





