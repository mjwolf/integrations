# Microsoft Office 365 Integration for Elastic

> **Note**: This documentation was generated using AI and should be reviewed for accuracy.

## Overview
The Microsoft Office 365 integration for Elastic enables you to collect and analyze activity data from across the Microsoft 365 ecosystem. This provides visibility into user actions, administrative changes, and security events by ingesting data using the [Office 365 Management Activity API](https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-reference). By centralizing these logs, you can monitor your environment for security threats, ensure compliance, and troubleshoot administrative issues.

This integration facilitates:
- Security monitoring and threat detection: You can monitor for unauthorized access attempts, suspicious login activity in Microsoft Entra ID (Azure AD), and unusual administrative privilege escalations.
- Data loss prevention (DLP) oversight: You'll collect and analyze DLP events to identify when sensitive information is shared or accessed in violation of corporate policies across Exchange and SharePoint.
- Compliance and auditing: You can maintain a long-term record of all system and user actions required for regulatory compliance, such as GDPR, HIPAA, or SOC2.
- Service administration tracking: You'll track changes to mailbox permissions in Exchange Online, file sharing activities in SharePoint, and configuration modifications in your tenant.

### Compatibility
This integration is compatible with the following:
- Microsoft Office 365 Management Activity API `v1.0`
- Microsoft Entra ID (formerly Azure Active Directory)
- Microsoft Purview (for Audit and DLP features)
- Supported workloads: `Audit.AzureActiveDirectory`, `Audit.Exchange`, `Audit.SharePoint`, `Audit.General`, and `DLP.All`

### How it works
This integration works by collecting user, admin, system, and policy actions, as well as events from Office 365 and Microsoft Entra ID activity logs. It retrieves this information by making API calls to the [Office 365 Management Activity API](https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-reference). An Elastic Agent configured with this integration periodically polls the API for new content blobs, downloads the event data, and forwards it to your Elastic deployment. The integration uses the modern Common Expression Language (CEL) input to provide rich telemetry formatted as JSON objects, ensuring efficient and reliable data collection.

## What data does this integration collect?

The Microsoft Office 365 integration collects log messages of the following types:
*   Microsoft Office 365 audit logs: Comprehensive records of user and admin activities across various Office 365 services, collected via the [Office 365 Management Activity API](https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-reference). These are the same logs that are available under [Audit Log Search](https://learn.microsoft.com/en-us/purview/audit-search) in the Microsoft Purview portal.
*   Authentication logs: Detailed records of sign-in attempts, MFA challenges, and application access events from Azure Active Directory/Microsoft Entra ID.
*   Content-specific logs: Activity logs for specific workloads including SharePoint file operations, Exchange mailbox access, and general Office 365 service events.
*   DLP events: Specialized logs detailing matches against Data Loss Prevention (DLP) policies across all supported Office 365 services.

This integration includes the following data streams:
*   `audit`: This data stream collects Microsoft Office 365 audit logs using the modern Common Expression Language (CEL) input. It provides rich telemetry including user actions, system events, and policy changes formatted as `json` objects.
*   Deprecated `audit`: A legacy data stream used for older installations. You should deactivate this option and instead use the one described above.

### Supported use cases

Integrating Microsoft Office 365 with Elastic provides a powerful solution for enhancing your security posture and operational visibility. You can use this integration for the following use cases:
*   Security monitoring and analysis: Leverage Elastic SIEM to collect and analyze audit logs for monitoring and analysis, which you can then visualize in Kibana.
*   Real-time threat detection: Identify suspicious activities such as unusual sign-in attempts, unauthorized file access, or unauthorized changes to security policies.
*   Compliance and auditing: Maintain a searchable, long-term archive of audit logs to meet regulatory compliance requirements and conduct thorough security audits.
*   Incident response: Accelerate incident investigations by correlating Office 365 data with other security and observability data sources within Elastic.

## What do I need to use this integration?

To use this integration, you'll need the following:

- An active Elastic Agent installed and enrolled in Fleet.
- Secure network connectivity between the Elastic Agent and the Elastic Stack.
- Outbound HTTPS access (port `443`) from the Elastic Agent host to `manage.office.com` and `login.microsoftonline.com`.
- [Audit logging enabled](https://learn.microsoft.com/en-us/purview/audit-log-enable-disable) in your Microsoft 365 tenant.
- A registered application in [Microsoft Entra ID](https://www.microsoft.com/en-us/security/business/identity-access/microsoft-entra-id) with administrative access to grant API permissions.

Once you register the Microsoft Entra ID application, you can configure its credentials and permissions to gather the information required by the integration:

1. Record the `Application (client) ID` and `Directory (tenant) ID` from the registered application's `Overview` page.
2. Create a new client secret to authenticate your application:
    - Navigate to the `Certificates & secrets` section under `Manage`.
    - Select `New client secret`, provide a description, and create the secret.
    - Record the secret `Value` immediately. This is required for the integration setup and won't be visible again after you leave the page.
3. Add the required permissions to your registered application. For more details, refer to the [Office 365 Management API documentation](https://learn.microsoft.com/en-us/office/office-365-management-api/get-started-with-office-365-management-apis#specify-the-permissions-your-app-requires-to-access-the-office-365-management-apis):
    - Navigate to the `API permissions` page under `Manage`. Under `Configured permissions`, select `Add a permission`.
    - Select the `Office 365 Management APIs` tile.
    - Select `Application permissions`.
    - Under `ActivityFeed`, select the `ActivityFeed.Read` permission. This is the minimum requirement to read organization audit logs. You can optionally select `ActivityFeed.ReadDlp` to read DLP policy events.
    - Select `Add permissions`.
    - If the `User.Read` permission under the `Microsoft Graph` tile is not added by default, add it.
    - A tenant administrator must grant consent for these permissions in the Azure portal.

These steps assume you're collecting data from your own tenant. If you're collecting data from a different tenant, you'll need to obtain tenant admin consent for the required permissions. The API documentation describes [a method of gathering consent via redirect URLs](https://learn.microsoft.com/en-us/office/office-365-management-api/get-started-with-office-365-management-apis#get-office-365-tenant-admin-consent).

## How do I deploy this integration?

### Agent-based deployment

Elastic Agent must be installed. For more details, check the Elastic Agent [installation instructions](https://www.elastic.co/guide/en/fleet/current/elastic-agent-installation.html). You can install only one Elastic Agent per host.

Elastic Agent is required to stream data from the syslog or log file receiver and ship the data to Elastic, where the events will then be processed via the integration's ingest pipelines.

### Agentless deployment

Agentless deployments are only supported in Elastic Serverless and Elastic Cloud environments. Agentless deployments provide a means to ingest data while avoiding the orchestration, management, and maintenance needs associated with standard ingest infrastructure. Using an agentless deployment makes manual agent deployment unnecessary, allowing you to focus on your data instead of the agent that collects it.

For more information, refer to [Agentless integrations](https://www.elastic.co/guide/en/serverless/current/security-agentless-integrations.html) and [Agentless integrations FAQ](https://www.elastic.co/guide/en/serverless/current/agentless-integration-troubleshooting.html).

### Set up steps in Microsoft Office 365

Follow these steps to configure your Microsoft environment for API-based collection:

1.  **Register an application in Microsoft Entra ID:**
    - Log in to the [Azure Portal](https://portal.azure.com).
    - Navigate to **Microsoft Entra ID > App registrations > New registration**.
    - Provide a name, such as `Elastic-O365`, and select **Accounts in this organizational directory only (Single tenant)**.
    - Click **Register**.
    - Save the **Application (client) ID** and **Directory (tenant) ID** for later use.
2.  **Configure API permissions:**
    - In the app menu, navigate to **API permissions > Add a permission**.
    - Select **Office 365 Management APIs > Application permissions**.
    - Expand the **ActivityFeed** section and check both `ActivityFeed.Read` and `ActivityFeed.ReadDlp`.
    - Click **Add permissions**, then click **Grant admin consent for <your-organization>** (replace with your actual value) and confirm.
3.  **Create a client secret:**
    - Go to **Certificates & secrets > New client secret**.
    - Enter a description and select an expiration period, then click **Add**.
    - Immediately copy the **Secret Value**. You won't be able to see it again after you leave this page.
4.  **Enable auditing in Microsoft Purview:**
    - Access the [Microsoft Purview portal](https://purview.microsoft.com).
    - Navigate to **Solutions > Audit**.
    - If auditing isn't already enabled, click the banner to **Start recording user and admin activity**. Note that activation can take up to 48 hours.

#### Vendor resources

- [Get started with Office 365 Management APIs](https://learn.microsoft.com/en-us/office/office-365-management-api/get-started-with-office-365-management-apis)

### Set up steps in Kibana

You can configure the integration in Kibana to collect logs using the modern Management Activity API or the legacy method if you have specific requirements.

#### Collect audit logs using the Management Activity API

1.  In Kibana, navigate to **Management > Integrations** and search for **Microsoft Office 365**.
2.  Click **Add Microsoft Office 365** and select an Elastic Agent policy.
3.  Configure the following settings for the primary input:
    - **Base URL of Office Management API**: The base endpoint for the API. Default is `https://manage.office.com`.
    - **Interval**: How frequently the API is polled for new data. Default is `3m`.
    - **Directory (tenant) ID**: The unique Directory (tenant) ID from your Entra ID portal.
    - **Application (client) ID**: The Client ID used for OAuth 2.0 authentication.
    - **Client Secret**: The secret value generated in your Entra ID app registration.
    - **OAuth 2.0 Token URL**: The base endpoint for generating tokens. Default is `https://login.microsoftonline.com`.
    - **Content Type**: A comma-separated list of workloads to fetch. Supported types include `Audit.AzureActiveDirectory, Audit.Exchange, Audit.SharePoint, Audit.General, DLP.All`.
    - **Initial Interval**: The initial window for the first API call (maximum 7 days). Default is `167h55m`.
    - **Batch Interval**: The interval for each individual API request (maximum 24 hours). Default is `1h`.
    - **Preserve original event**: If enabled, this stores the raw JSON payload in the `event.original` field.
4.  Expand **Advanced options** for further configuration:
    - **Token Scopes**: The scopes required for the OAuth 2.0 token. Default is `['https://manage.office.com/.default']`.
    - **Resource SSL Configuration**: Configure SSL settings, including certificate authorities or verification modes.
    - **Resource Proxy**: Set up a proxy if your environment requires one using the format `http[s]://<user>:<password>@<server>:<port>`.
    - **Enable request tracing**: Use this for debugging API interactions. It logs requests and responses, which may expose sensitive information.
5.  Click **Save and continue**.

#### Legacy configuration (Deprecated)

This option uses a deprecated method to collect logs. Only use this if you have a legacy requirement that cannot be met by the standard API input.

1.  Provide the **Application (client) ID** and **Client secret**.
2.  If using certificate-based authentication, specify the **Path to certificate file** and **Path to private key file**.
3.  Enter the **Directory (tenant) IDs** as a list, such as `['<your-tenant-id>']` (replace with your actual value).
4.  Select the **Content types** you wish to collect.
5.  Click **Save and continue**.

### Validation

To verify the integration is working correctly, check the status of your Elastic Agent and then confirm data flow in Kibana.

#### Verify agent status

1.  In Kibana, navigate to **Management > Fleet > Agents**.
2.  Locate the Elastic Agent running the Microsoft Office 365 integration.
3.  Ensure the status is **Healthy**. If it's in a **Warning** or **Offline** state, check the agent logs for errors related to the Office 365 input.

#### Trigger test events

Perform actions in your Microsoft 365 tenant to generate audit logs:
- **Entra ID:** Create a test user or modify a group's membership in the **Microsoft Entra admin center**.
- **Exchange:** Send an email between internal accounts or change a mailbox permission in the **Exchange admin center**.
- **SharePoint:** Upload a file to a site or change a folder's sharing settings to trigger `FileUploaded` or `FileModified` events.

#### Check data in Kibana

1.  Navigate to **Analytics > Discover**.
2.  Select the `logs-*` data view.
3.  Filter the results using the following KQL query: `data_stream.dataset : "o365.audit"`
4.  Verify that events appear with recent timestamps. Expand a document and confirm the following:
    - `event.dataset` is set to `o365.audit`.
    - `o365.audit.UserId` contains the email or ID of the user who triggered the event.
    - `event.action` matches the activity performed (e.g., `Add user`).
5.  Navigate to **Analytics > Dashboards** and search for **Microsoft Office 365** to view pre-built visualizations. Check the **Office 365 Audit Dashboard** to ensure charts and tables are populating with data.

## Troubleshooting

For help with Elastic ingest tools, check [Common problems](https://www.elastic.co/docs/troubleshoot/ingest/fleet/common-problems).

### Common configuration issues

You might encounter the following issues when configuring or running the Microsoft Office 365 integration:

- No data appears immediately after setup: When you first enable a subscription, there's a documented Microsoft API delay of up to 12 hours. Don't troubleshoot connectivity until this window has passed.
- API returns 403 Forbidden errors: This usually happens if you added permissions in the Azure Portal but didn't click **Grant admin consent**. Ensure the status shows "Granted" in the portal for all required permissions.
- API returns empty results: If you haven't turned on the Purview Audit log, the API returns empty results even if you've correctly configured the integration. Verify the auditing status in the Microsoft Purview portal.
- Configuration fails after upgrading to version 1.18.0 or newer: Ensure your Elastic Stack version is at least 8.7.1. Note that certificate-based authentication isn't supported in the newer CEL input.
- Events are missing or have error messages: Check your Elastic Agent logs for CEL expression errors. These errors typically indicate a change in the upstream JSON schema from Microsoft.
- Specific fields like `o365.audit.FileName` are missing: Some fields only exist for specific workloads. For example, `o365.audit.FileName` is only available for SharePoint and OneDrive logs. Check the `event.dataset` to confirm you're viewing the correct event type.
- Troubleshooting complex authentication or data flow issues: You can enable the `enable_request_tracer` setting in the integration configuration to log raw requests and responses to the agent's filesystem. Only use this for temporary debugging as it can capture sensitive data.
- OAuth flow fails with `invalid_client`: Verify that the `client_secret` in your configuration hasn't expired in the Azure Portal.
- API authentication fails with token URL issues: Ensure the `token_url` setting matches your Azure environment, which is typically `https://login.microsoftonline.com`.
- Authentication failures on the legacy input: The deprecated `o365audit` input used ADAL, which Microsoft is retiring. If you encounter authentication issues on the legacy input, migrate to the modern CEL-based input.

### Vendor resources

Use these resources for more information about the Microsoft Office 365 Management APIs:

- [Office 365 Management Activity API Reference](https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-reference)
- [Audit Log Record Type Documentation](https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-schema#auditlogrecordtype)
- [Microsoft Purview Audit Log Setup](https://learn.microsoft.com/en-us/purview/audit-log-enable-disable)

## Performance and scaling

For more information on architectures that can be used for scaling this integration, check the [Ingest Architectures](https://www.elastic.co/docs/manage-data/ingest/ingest-reference-architectures) documentation.

To ensure optimal performance in high-volume environments, consider the following:

- Transport and collection considerations: This integration relies on API polling via the Office 365 Management Activity API. You configure the `Interval` (default `3m`) and `Batch Interval` (default `1h`) to balance data freshness with API rate limits. Be aware that Microsoft documents a latency of up to 12 hours for some data to become available for collection; adjusting the interval too low may result in empty API responses if the data isn't yet published by Microsoft.
- Data volume management: Use the `Content Type` configuration to filter for only required workloads at the source. For example, if identity data is your priority, collecting only `Audit.AzureActiveDirectory` significantly reduces volume compared to enabling all types. Additionally, use the `Processors` setting to drop unnecessary fields or entire events at the Elastic Agent level before they're transmitted to the Elastic Stack.
- Elastic Agent scaling: For high-throughput environments or large tenants with thousands of users, a single agent may reach resource limits. You can deploy multiple Elastic Agents and distribute the workload by configuring different agent policies to collect specific `Content Types` or by assigning different agents to monitor different `Directory (tenant) IDs`.

## Reference

### Inputs used

These inputs can be used with this integration:
<details>
<summary>cel</summary>

## Setup

For more details about the CEL input settings, check the [Filebeat documentation](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-input-cel.html).

Before configuring the CEL input, make sure you have:
- Network connectivity to the target API endpoint
- Valid authentication credentials (API keys, tokens, or certificates as required)
- Appropriate permissions to read from the target data source

### Collecting logs from CEL

To configure the CEL input, you must specify the `request.url` value pointing to the API endpoint. The interval parameter controls how frequently requests are made and is the primary way to balance data freshness with API rate limits and costs. Authentication is often configured through the `request.headers` section using the appropriate method for the service.

NOTE: To access the API service, make sure you have the necessary API credentials and that the Filebeat instance can reach the endpoint URL. Some services may require IP whitelisting or VPN access.

To collect logs via API endpoint, configure the following parameters:

- API Endpoint URL
- API credentials (tokens, keys, or username/password)
- Request interval (how often to fetch data)
</details>


### API usage

You'll use these APIs with this integration:
- [Office 365 Management Activity API](https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-reference): This API provides visibility into user, admin, system, and policy actions and events from Microsoft 365 and Microsoft Entra ID activity logs.

### Data streams

#### audit

The `audit` data stream provides events from the Office 365 Management Activity API of the following types: SharePoint, Exchange, Microsoft Entra ID, and general Office 365 audit records. You'll use this data stream to monitor user activity and configuration changes across your Microsoft 365 tenant.

##### audit fields

You can find a complete list of fields exported by the `audit` data stream in the following table:

**Exported fields**

| Field | Description | Type |
|---|---|---|
| @timestamp | Event timestamp. | date |
| application.name | Name of the application. | keyword |
| cloud.image.id | Image ID for the cloud instance. | keyword |
| data_stream.dataset | Data stream dataset name. | constant_keyword |
| data_stream.namespace | Data stream namespace. | constant_keyword |
| data_stream.type | Data stream type. | constant_keyword |
| event.dataset | Event dataset | constant_keyword |
| event.module | Event module | constant_keyword |
| host.containerized | If the host is a container. | boolean |
| host.os.build | OS build information. | keyword |
| host.os.codename | OS codename, if any. | keyword |
| input.type | Type of Filebeat input. | keyword |
| log.flags | Flags for the log file. | keyword |
| log.offset | Offset of the entry in the log file. | long |
| o365.audit.AadAppId |  | keyword |
| o365.audit.Actions |  | flattened |
| o365.audit.Activity |  | keyword |
| o365.audit.Actor.ID |  | keyword |
| o365.audit.Actor.Type |  | keyword |
| o365.audit.ActorContextId |  | keyword |
| o365.audit.ActorInfoString |  | keyword |
| o365.audit.ActorIpAddress |  | keyword |
| o365.audit.ActorUserId |  | keyword |
| o365.audit.ActorYammerUserId |  | keyword |
| o365.audit.AdditionalData.Name |  | keyword |
| o365.audit.AdditionalData.Value |  | keyword |
| o365.audit.AdditionalInfo.\* |  | object |
| o365.audit.AirAdminActionSource |  | keyword |
| o365.audit.AirAdminActionType |  | keyword |
| o365.audit.AlertEntityId |  | keyword |
| o365.audit.AlertId |  | keyword |
| o365.audit.AlertLinks |  | flattened |
| o365.audit.AlertType |  | keyword |
| o365.audit.AppAccessContext.\* |  | object |
| o365.audit.AppId |  | keyword |
| o365.audit.Application |  | keyword |
| o365.audit.ApplicationDisplayName |  | keyword |
| o365.audit.ApplicationId |  | keyword |
| o365.audit.Approver |  | keyword |
| o365.audit.AttachmentData.FileName |  | keyword |
| o365.audit.AttachmentData.FileType |  | keyword |
| o365.audit.AttachmentData.FileVerdict |  | keyword |
| o365.audit.AttachmentData.MalwareFamily |  | keyword |
| o365.audit.AttachmentData.SHA256 |  | keyword |
| o365.audit.AuthDetails.Name |  | keyword |
| o365.audit.AuthDetails.Value |  | keyword |
| o365.audit.AuthenticationType |  | keyword |
| o365.audit.AzureActiveDirectoryEventType |  | keyword |
| o365.audit.BCLValue |  | keyword |
| o365.audit.BulkApprovalId |  | keyword |
| o365.audit.Category |  | keyword |
| o365.audit.ClientAppId |  | keyword |
| o365.audit.ClientApplication |  | keyword |
| o365.audit.ClientIP |  | keyword |
| o365.audit.ClientIPAddress |  | keyword |
| o365.audit.ClientInfoString |  | keyword |
| o365.audit.ClientRequestId |  | keyword |
| o365.audit.CmdletVersion |  | keyword |
| o365.audit.Comments |  | text |
| o365.audit.Connector |  | keyword |
| o365.audit.CorrelationId |  | keyword |
| o365.audit.CreationTime |  | keyword |
| o365.audit.CustomUniqueId |  | boolean |
| o365.audit.Data.ad |  | keyword |
| o365.audit.Data.af |  | keyword |
| o365.audit.Data.aii |  | keyword |
| o365.audit.Data.ail |  | keyword |
| o365.audit.Data.alk |  | keyword |
| o365.audit.Data.als |  | keyword |
| o365.audit.Data.an |  | keyword |
| o365.audit.Data.at |  | date |
| o365.audit.Data.cid |  | keyword |
| o365.audit.Data.cpid |  | keyword |
| o365.audit.Data.dm |  | keyword |
| o365.audit.Data.dpn |  | keyword |
| o365.audit.Data.eid |  | keyword |
| o365.audit.Data.etps |  | keyword |
| o365.audit.Data.etype |  | keyword |
| o365.audit.Data.f3u |  | keyword |
| o365.audit.Data.flattened | The full Data document. | flattened |
| o365.audit.Data.fvs |  | keyword |
| o365.audit.Data.imsgid |  | keyword |
| o365.audit.Data.lon |  | keyword |
| o365.audit.Data.mat |  | keyword |
| o365.audit.Data.md |  | date |
| o365.audit.Data.ms |  | keyword |
| o365.audit.Data.od |  | keyword |
| o365.audit.Data.op |  | keyword |
| o365.audit.Data.ot |  | keyword |
| o365.audit.Data.plk |  | keyword |
| o365.audit.Data.pud |  | keyword |
| o365.audit.Data.reid |  | keyword |
| o365.audit.Data.rid |  | keyword |
| o365.audit.Data.sev |  | keyword |
| o365.audit.Data.sict |  | keyword |
| o365.audit.Data.sid |  | keyword |
| o365.audit.Data.sip |  | ip |
| o365.audit.Data.sitmi |  | keyword |
| o365.audit.Data.srt |  | keyword |
| o365.audit.Data.ssic |  | keyword |
| o365.audit.Data.suid |  | keyword |
| o365.audit.Data.tdc |  | keyword |
| o365.audit.Data.te |  | date |
| o365.audit.Data.thn |  | keyword |
| o365.audit.Data.tht |  | keyword |
| o365.audit.Data.tid |  | keyword |
| o365.audit.Data.tpid |  | keyword |
| o365.audit.Data.tpt |  | keyword |
| o365.audit.Data.trc |  | keyword |
| o365.audit.Data.ts |  | date |
| o365.audit.Data.tsd |  | keyword |
| o365.audit.Data.ttdt |  | date |
| o365.audit.Data.ttr |  | keyword |
| o365.audit.Data.upfc |  | keyword |
| o365.audit.Data.upfv |  | keyword |
| o365.audit.Data.ut |  | keyword |
| o365.audit.Data.von |  | keyword |
| o365.audit.Data.wl |  | keyword |
| o365.audit.Data.zfh |  | keyword |
| o365.audit.Data.zfn |  | keyword |
| o365.audit.Data.zmfh |  | keyword |
| o365.audit.Data.zmfn |  | keyword |
| o365.audit.Data.zu |  | keyword |
| o365.audit.DataType |  | keyword |
| o365.audit.DatabaseType |  | keyword |
| o365.audit.DeepLinkUrl |  | keyword |
| o365.audit.DeliveryAction |  | keyword |
| o365.audit.Description |  | match_only_text |
| o365.audit.DetectionMethod |  | keyword |
| o365.audit.DetectionType |  | keyword |
| o365.audit.DeviceName |  | keyword |
| o365.audit.Directionality |  | keyword |
| o365.audit.EffectiveOrganization |  | keyword |
| o365.audit.EndTimeUtc |  | date |
| o365.audit.EntityType |  | keyword |
| o365.audit.ErrorNumber |  | keyword |
| o365.audit.EventData |  | keyword |
| o365.audit.EventDeepLink |  | keyword |
| o365.audit.EventSource |  | keyword |
| o365.audit.ExceptionInfo.\* |  | object |
| o365.audit.ExchangeAggregatedFolders.FolderItems.Id | Item ID | keyword |
| o365.audit.ExchangeAggregatedFolders.FolderItems.ImmutableId | Immutable ID of the item | keyword |
| o365.audit.ExchangeAggregatedFolders.FolderItems.InternetMessageId | Internet message ID | keyword |
| o365.audit.ExchangeAggregatedFolders.FolderItems.SizeInBytes | Size of the item in bytes | long |
| o365.audit.ExchangeAggregatedFolders.Id | Folder ID | keyword |
| o365.audit.ExchangeAggregatedFolders.Path | Path of the folder | keyword |
| o365.audit.ExchangeAggregatedMessages.Id | Message ID | keyword |
| o365.audit.ExchangeAggregatedMessages.MessageItems.Id | Message item ID | keyword |
| o365.audit.ExchangeAggregatedMessages.MessageItems.SizeInBytes | Size of the message item in bytes | long |
| o365.audit.ExchangeAggregatedMessages.Path | Path of the message | keyword |
| o365.audit.ExchangeMetaData.\* |  | long |
| o365.audit.ExchangeMetaData.CC |  | keyword |
| o365.audit.ExchangeMetaData.MessageID |  | keyword |
| o365.audit.ExchangeMetaData.Sent |  | date |
| o365.audit.ExchangeMetaData.Subject |  | keyword |
| o365.audit.ExchangeMetaData.To |  | keyword |
| o365.audit.ExchangeMetaData.UniqueID |  | keyword |
| o365.audit.Experience |  | keyword |
| o365.audit.ExtendedProperties.\* |  | object |
| o365.audit.ExtendedProperties.RequestType |  | keyword |
| o365.audit.ExtendedProperties.additionalDetails |  | object |
| o365.audit.ExternalAccess |  | boolean |
| o365.audit.FileExtension |  | keyword |
| o365.audit.FileSize |  | keyword |
| o365.audit.FileSizeBytes |  | long |
| o365.audit.FilteringDate |  | date |
| o365.audit.GroupName |  | keyword |
| o365.audit.Id |  | keyword |
| o365.audit.ImplicitShare |  | keyword |
| o365.audit.IncidentId |  | keyword |
| o365.audit.InsightData.Type |  | keyword |
| o365.audit.InsightId |  | keyword |
| o365.audit.InterSystemsId |  | keyword |
| o365.audit.InternalLogonType |  | keyword |
| o365.audit.InternetMessageId |  | keyword |
| o365.audit.IntraSystemId |  | keyword |
| o365.audit.InvestigationId |  | keyword |
| o365.audit.InvestigationName |  | keyword |
| o365.audit.InvestigationType |  | keyword |
| o365.audit.InvestigationUrn |  | keyword |
| o365.audit.Item.\* |  | object |
| o365.audit.Item.\*.\* |  | object |
| o365.audit.ItemName |  | keyword |
| o365.audit.ItemType |  | keyword |
| o365.audit.KesMailId |  | keyword |
| o365.audit.Language |  | keyword |
| o365.audit.LastUpdateTimeUtc |  | date |
| o365.audit.LatestDeliveryLocation |  | keyword |
| o365.audit.ListBaseType |  | keyword |
| o365.audit.ListId |  | keyword |
| o365.audit.ListItemUniqueId |  | keyword |
| o365.audit.LogonError |  | keyword |
| o365.audit.LogonType |  | keyword |
| o365.audit.LogonUserSid |  | keyword |
| o365.audit.MailboxGuid |  | keyword |
| o365.audit.MailboxOwnerMasterAccountSid |  | keyword |
| o365.audit.MailboxOwnerSid |  | keyword |
| o365.audit.MailboxOwnerUPN |  | keyword |
| o365.audit.Members |  | flattened |
| o365.audit.MessageDate |  | keyword |
| o365.audit.MessageTime |  | keyword |
| o365.audit.ModifiedProperties |  | object |
| o365.audit.ModifiedProperties.\*.\* |  | object |
| o365.audit.ModifiedProperties.Role_DisplayName.NewValue |  | keyword |
| o365.audit.Name |  | keyword |
| o365.audit.NetworkMessageId |  | keyword |
| o365.audit.NewValue |  | keyword |
| o365.audit.NonPIIParameters |  | keyword |
| o365.audit.ObjectDisplayName |  | keyword |
| o365.audit.ObjectId |  | keyword |
| o365.audit.ObjectType |  | keyword |
| o365.audit.Operation |  | keyword |
| o365.audit.OperationCount |  | long |
| o365.audit.OperationId |  | keyword |
| o365.audit.OperationProperties |  | object |
| o365.audit.OrganizationId |  | keyword |
| o365.audit.OrganizationName |  | keyword |
| o365.audit.OriginalDeliveryLocation |  | keyword |
| o365.audit.OriginatingDomain |  | keyword |
| o365.audit.OriginatingServer |  | keyword |
| o365.audit.P1Sender |  | keyword |
| o365.audit.P1SenderDomain |  | keyword |
| o365.audit.P2Sender |  | keyword |
| o365.audit.P2SenderDomain |  | keyword |
| o365.audit.Parameters |  | object |
| o365.audit.Parameters.\* |  | object |
| o365.audit.Parameters.AccessRights |  | keyword |
| o365.audit.Parameters.AllowFederatedUsers |  | keyword |
| o365.audit.Parameters.AllowGuestUser |  | keyword |
| o365.audit.Parameters.Enabled |  | keyword |
| o365.audit.Parameters.ForwardAsAttachmentTo |  | keyword |
| o365.audit.Parameters.ForwardTo |  | keyword |
| o365.audit.Parameters.From |  | keyword |
| o365.audit.Parameters.RedirectTo |  | keyword |
| o365.audit.PhishConfidenceLevel |  | keyword |
| o365.audit.Platform |  | keyword |
| o365.audit.Policy |  | keyword |
| o365.audit.PolicyAction |  | keyword |
| o365.audit.PolicyDetails |  | flattened |
| o365.audit.PolicyId |  | keyword |
| o365.audit.Recipients |  | keyword |
| o365.audit.RecordType |  | keyword |
| o365.audit.RelativeUrl |  | keyword |
| o365.audit.RequestId |  | keyword |
| o365.audit.RescanResult.Id |  | keyword |
| o365.audit.RescanResult.RescanVerdict |  | keyword |
| o365.audit.RescanResult.Timestamp |  | keyword |
| o365.audit.ResultCount |  | keyword |
| o365.audit.ResultStatus |  | keyword |
| o365.audit.RunningTime |  | keyword |
| o365.audit.SecurityComplianceCenterEventType |  | keyword |
| o365.audit.SenderIP |  | keyword |
| o365.audit.SenderIp |  | keyword |
| o365.audit.SensitiveInfoDetectionIsIncluded |  | boolean |
| o365.audit.SensitivityLabelEventData.ActionSourceDetail |  | long |
| o365.audit.SensitivityLabelEventData.ContentType |  | keyword |
| o365.audit.SensitivityLabelEventData.LabelEventType |  | long |
| o365.audit.SensitivityLabelEventData.SensitivityLabelId |  | keyword |
| o365.audit.SessionId |  | keyword |
| o365.audit.Severity |  | keyword |
| o365.audit.Sha1 |  | keyword |
| o365.audit.Sha256 |  | keyword |
| o365.audit.SharePointMetaData.\* |  | object |
| o365.audit.Site |  | keyword |
| o365.audit.SiteUrl |  | keyword |
| o365.audit.Source |  | keyword |
| o365.audit.SourceFileExtension |  | keyword |
| o365.audit.SourceFileName |  | keyword |
| o365.audit.SourceRelativeUrl |  | keyword |
| o365.audit.StartTime |  | keyword |
| o365.audit.StartTimeUtc |  | keyword |
| o365.audit.Status |  | keyword |
| o365.audit.SubAirAdminActionTypeMail |  | keyword |
| o365.audit.Subject |  | keyword |
| o365.audit.SubmissionConfidenceLevel |  | keyword |
| o365.audit.SubmissionContentSubType |  | keyword |
| o365.audit.SubmissionContentType |  | keyword |
| o365.audit.SubmissionId |  | keyword |
| o365.audit.SubmissionState |  | keyword |
| o365.audit.SubmissionType |  | keyword |
| o365.audit.Submitter |  | keyword |
| o365.audit.SubmitterId |  | keyword |
| o365.audit.SupportTicketId |  | keyword |
| o365.audit.SystemOverrides.Details |  | keyword |
| o365.audit.SystemOverrides.FinalOverride |  | keyword |
| o365.audit.SystemOverrides.Result |  | keyword |
| o365.audit.SystemOverrides.Source |  | keyword |
| o365.audit.Target.ID |  | keyword |
| o365.audit.Target.Type |  | keyword |
| o365.audit.TargetContextId |  | keyword |
| o365.audit.TargetFilePath |  | keyword |
| o365.audit.TargetUserOrGroupName |  | keyword |
| o365.audit.TargetUserOrGroupType |  | keyword |
| o365.audit.TeamGuid |  | keyword |
| o365.audit.TeamName |  | keyword |
| o365.audit.ThreatDetectionMethods |  | keyword |
| o365.audit.Timestamp |  | keyword |
| o365.audit.TokenObjectId |  | keyword |
| o365.audit.TokenTenantId |  | keyword |
| o365.audit.UniqueSharingId |  | keyword |
| o365.audit.UserAgent |  | keyword |
| o365.audit.UserId |  | keyword |
| o365.audit.UserKey |  | keyword |
| o365.audit.UserType |  | keyword |
| o365.audit.Verdict |  | keyword |
| o365.audit.Version |  | keyword |
| o365.audit.WebId |  | keyword |
| o365.audit.Workload |  | keyword |
| o365.audit.WorkspaceId |  | keyword |
| o365.audit.WorkspaceName |  | keyword |
| o365.audit.YammerNetworkId |  | keyword |
| session.id | The unique identifier for the authentication session. | keyword |
| token.id | The unique token identifier of the API call used to make the audited change. | keyword |


##### audit sample event

This is a sample event for the `audit` data stream:

An example event for `audit` looks as following:

```json
{
    "@timestamp": "2020-02-07T16:43:53.000Z",
    "agent": {
        "ephemeral_id": "f173faa4-2d61-4c41-8670-06930fd22753",
        "id": "e9670ccb-33fc-41b0-90c1-b67dcf953c6a",
        "name": "elastic-agent-35357",
        "type": "filebeat",
        "version": "8.18.0"
    },
    "client": {
        "address": "213.97.47.133",
        "ip": "213.97.47.133"
    },
    "data_stream": {
        "dataset": "o365.audit",
        "namespace": "12427",
        "type": "logs"
    },
    "device": {
        "id": "62eedfc0-b73c-206c-a59d-16457c7ebcd8"
    },
    "ecs": {
        "version": "8.11.0"
    },
    "elastic_agent": {
        "id": "e9670ccb-33fc-41b0-90c1-b67dcf953c6a",
        "snapshot": false,
        "version": "8.18.0"
    },
    "event": {
        "action": "PageViewed",
        "agent_id_status": "verified",
        "category": [
            "web"
        ],
        "code": "SharePoint",
        "dataset": "o365.audit",
        "id": "99d005e6-a4c6-46fd-117c-08d7abeceab5",
        "ingested": "2025-12-15T13:36:59Z",
        "kind": "event",
        "original": "{\"ClientIP\":\"213.97.47.133\",\"CorrelationId\":\"622b339f-4000-a000-f25f-92b3478c7a25\",\"CreationTime\":\"2020-02-07T16:43:53\",\"CustomUniqueId\":true,\"EventSource\":\"SharePoint\",\"ExtendedProperties\":[{\"Name\":\"additionalDetails\",\"Value\":\"{\\\"DeviceId\\\":\\\"62eedfc0-b73c-206c-a59d-16457c7ebcd8\\\",\\\"DeviceOSType\\\":\\\"Linux\\\",\\\"DeviceTrustType\\\":\\\"\\\"}\"}],\"Id\":\"99d005e6-a4c6-46fd-117c-08d7abeceab5\",\"ItemType\":\"Page\",\"ListItemUniqueId\":\"59a8433d-9bb8-cfef-6edc-4c0fc8b86875\",\"ObjectId\":\"https://testsiem-my.sharepoint.com/personal/asr_testsiem_onmicrosoft_com/_layouts/15/onedrive.aspx\",\"Operation\":\"PageViewed\",\"OrganizationId\":\"b86ab9d4-fcf1-4b11-8a06-7a8f91b47fbd\",\"RecordType\":4,\"Site\":\"d5180cfc-3479-44d6-b410-8c985ac894e3\",\"UserAgent\":\"Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:72.0) Gecko/20100101 Firefox/72.0\",\"UserId\":\"asr@testsiem.onmicrosoft.com\",\"UserKey\":\"i:0h.f|membership|1003200096971f55@live.com\",\"UserType\":0,\"Version\":1,\"WebId\":\"8c5c94bb-8396-470c-87d7-8999f440cd30\",\"Workload\":\"OneDrive\"}",
        "outcome": "success",
        "provider": "OneDrive",
        "type": [
            "info"
        ]
    },
    "host": {
        "id": "b86ab9d4-fcf1-4b11-8a06-7a8f91b47fbd",
        "name": "testsiem.onmicrosoft.com"
    },
    "input": {
        "type": "cel"
    },
    "network": {
        "type": "ipv4"
    },
    "o365": {
        "audit": {
            "CorrelationId": "622b339f-4000-a000-f25f-92b3478c7a25",
            "CreationTime": "2020-02-07T16:43:53",
            "CustomUniqueId": true,
            "EventSource": "SharePoint",
            "ExtendedProperties": {
                "additionalDetails": {
                    "DeviceId": "62eedfc0-b73c-206c-a59d-16457c7ebcd8",
                    "DeviceOSType": "Linux"
                }
            },
            "ItemType": "Page",
            "ListItemUniqueId": "59a8433d-9bb8-cfef-6edc-4c0fc8b86875",
            "ObjectId": "https://testsiem-my.sharepoint.com/personal/asr_testsiem_onmicrosoft_com/_layouts/15/onedrive.aspx",
            "RecordType": "4",
            "Site": "d5180cfc-3479-44d6-b410-8c985ac894e3",
            "UserId": "asr@testsiem.onmicrosoft.com",
            "UserKey": "i:0h.f|membership|1003200096971f55@live.com",
            "UserType": "0",
            "Version": "1",
            "WebId": "8c5c94bb-8396-470c-87d7-8999f440cd30"
        }
    },
    "organization": {
        "id": "b86ab9d4-fcf1-4b11-8a06-7a8f91b47fbd"
    },
    "related": {
        "hosts": [
            "testsiem.onmicrosoft.com"
        ],
        "ip": [
            "213.97.47.133"
        ],
        "user": [
            "asr",
            "asr@testsiem.onmicrosoft.com"
        ]
    },
    "source": {
        "ip": "213.97.47.133"
    },
    "tags": [
        "preserve_original_event",
        "forwarded",
        "o365-cel"
    ],
    "user": {
        "domain": "testsiem.onmicrosoft.com",
        "email": "asr@testsiem.onmicrosoft.com",
        "id": "asr@testsiem.onmicrosoft.com",
        "name": "asr"
    },
    "user_agent": {
        "device": {
            "name": "Mac"
        },
        "name": "Firefox",
        "original": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:72.0) Gecko/20100101 Firefox/72.0",
        "os": {
            "full": "Mac OS X 10.14",
            "name": "Mac OS X",
            "version": "10.14"
        },
        "version": "72.0"
    }
}
```

### Vendor documentation links

You can find more information about the Microsoft Office 365 APIs and audit logs in these resources:
- [Office 365 Management Activity API Reference](https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-reference)
- [Audit Log Record Type Documentation](https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-schema#auditlogrecordtype)
- [Microsoft Purview Audit Log Setup](https://learn.microsoft.com/en-us/purview/audit-log-enable-disable)
- [Register an App in Microsoft Entra ID](https://learn.microsoft.com/en-us/entra/identity-platform/quickstart-register-app)
- [Elastic CEL Input Documentation](https://www.elastic.co/docs/reference/beats/filebeat/filebeat-input-cel)
