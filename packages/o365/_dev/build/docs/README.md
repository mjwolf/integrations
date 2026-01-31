# Microsoft Office 365 Integration for Elastic

> **Note**: This documentation was generated using AI and should be reviewed for accuracy.

## Overview

The Microsoft Office 365 integration for Elastic enables you to collect and analyze activity data from across the Microsoft 365 ecosystem. By ingesting these logs, you gain visibility into user actions, administrative changes, and security events. This data helps you monitor security threats, audit policy compliance, and troubleshoot service configurations.

This integration facilitates:
- Security monitoring and threat detection: You can monitor for unauthorized access attempts, suspicious login activity in Microsoft Entra ID, and unusual administrative privilege escalations.
- Data loss prevention (DLP) oversight: You can analyze DLP events to identify when sensitive information is shared or accessed in violation of corporate policies.
- Compliance and auditing: You can maintain a long-term record of system and user actions required for regulatory compliance by centralizing Office 365 audit logs.
- Service administration tracking: You can track changes to mailbox permissions, file sharing activities, and configuration modifications in the tenant.

### Compatibility

This integration is compatible with the following services and versions:
- Microsoft Office 365 Management Activity API v1.0
- Microsoft Entra ID (formerly Azure Active Directory)
- Microsoft Purview (for Audit and DLP features)

This integration supports several workloads including:
- Audit.AzureActiveDirectory
- Audit.Exchange
- Audit.SharePoint
- Audit.General
- DLP.All

### How it works

This integration collects activity data using the [Office 365 Management Activity API](https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-reference). It uses a Common Expression Language (CEL) input to fetch user, admin, system, and policy actions from Office 365 and Microsoft Entra ID logs. The Elastic Agent runs on a host and makes secure API calls to retrieve these events as JSON objects. Once ingested, the logs are processed and indexed in your Elastic deployment for searching and visualization.

## What data does this integration collect?

The Microsoft Office 365 integration collects log messages from the [Office 365 Management Activity API](https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-reference). These logs are the same ones available under [Audit Log Search](https://learn.microsoft.com/en-us/purview/audit-search) in the Microsoft Purview portal.

The integration collects the following types of data:
*   Microsoft Office 365 audit logs: Comprehensive records of user and admin activities across various services.
*   Authentication logs: Detailed records of sign-in attempts, multi-factor authentication (MFA) challenges, and application access events from Azure Active Directory and Microsoft Entra ID.
*   Content-specific logs: Activity logs for specific workloads including SharePoint file operations, Exchange mailbox access, and general service events.
*   DLP events: Specialized logs detailing matches against Data Loss Prevention policies across supported services.

This integration includes the following data stream:
*   `audit`: This data stream collects Microsoft Office 365 audit logs using the modern Common Expression Language (CEL) input. It provides rich telemetry including user actions, system events, and policy changes formatted as JSON objects.

### Supported use cases

Integrating Microsoft Office 365 with Elastic provides visibility into your cloud environment for security and operational monitoring. Key use cases include:
*   Security monitoring: You can monitor audit logs for unauthorized access, suspicious administrative changes, or potential threats in real time.
*   Compliance and auditing: You can maintain a searchable archive of your logs to meet regulatory requirements and conduct security audits.
*   Incident response: You can accelerate investigations by correlating activity across different Microsoft 365 services with other data sources in Elastic.
*   Operational visibility: You can gain insights into service usage and track administrative changes across your organization.

## What do I need to use this integration?

Before you can collect logs from Microsoft Office 365, you'll need to complete several prerequisites in both your Microsoft environment and the Elastic Stack.

### Vendor prerequisites

You must configure your Microsoft 365 environment to allow the Elastic Agent to access audit data:

- Microsoft 365 Tenant Admin access: You need administrative permissions to enable auditing and grant admin consent for API permissions.
- Audit logging enabled: You must turn on audit search in the Microsoft Purview portal for logs to be generated. For instructions, see the [Microsoft documentation for enabling audit logs](https://learn.microsoft.com/en-us/purview/audit-log-enable-disable).
- Microsoft Entra ID application registration: You must register an application in the [Microsoft Entra ID portal](https://www.microsoft.com/en-us/security/business/identity-access/microsoft-entra-id) to obtain OAuth 2.0 credentials.
- Application credentials: You'll need the `Application (client) ID`, `Directory (tenant) ID`, and a `Client Secret` value from your registered application.
- API permissions: Your registered application requires `ActivityFeed.Read` (and optionally `ActivityFeed.ReadDlp`) permissions under the Office 365 Management APIs. You must also grant admin consent for these permissions in the Azure portal.
- Network connectivity: The Elastic Agent host requires outbound HTTPS access on port `443` to `manage.office.com` and `login.microsoftonline.com`.

### Elastic prerequisites

The following Elastic Stack components are required:

- Elastic Agent: You must have an active Elastic Agent installed and enrolled in Fleet.
- Connectivity: You'll need secure network communication between the Elastic Agent and the Elastic Stack (Elasticsearch and Kibana).
- Permissions: You must have the appropriate roles in Kibana to add and configure integrations.

## How do I deploy this integration?

This integration supports both agent-based and agentless deployment modes.

### Agent-based deployment

Elastic Agent must be installed. For more details, check the Elastic Agent [installation instructions](https://www.elastic.co/guide/en/fleet/current/elastic-agent-installation.html). You can install only one Elastic Agent per host.

Elastic Agent is required to stream data from the syslog or log file receiver and ship the data to Elastic, where the events will then be processed via the integration's ingest pipelines.

### Agentless deployment

Agentless deployments are only supported in Elastic Serverless and Elastic Cloud environments. Agentless deployments provide a means to ingest data while avoiding the orchestration, management, and maintenance needs associated with standard ingest infrastructure. Using an agentless deployment makes manual agent deployment unnecessary, allowing you to focus on your data instead of the agent that collects it.

This functionality is in beta and is subject to change. Beta features aren't subject to the support SLA of official GA features. For more information, refer to [Agentless integrations](https://www.elastic.co/guide/en/serverless/current/security-agentless-integrations.html) and [Agentless integrations FAQ](https://www.elastic.co/guide/en/serverless/current/agentless-integration-troubleshooting.html).

### Set up steps in Microsoft Office 365

To collect audit logs, you'll need to register an application and configure permissions in the Microsoft Azure portal.

1. **Register Application in Microsoft Entra ID:**
   - Log in to the [Azure Portal](https://portal.azure.com).
   - Navigate to **Microsoft Entra ID > App registrations > New registration**.
   - Name it (e.g., `Elastic-O365`) and select **Accounts in this organizational directory only (Single tenant)**. Click **Register**.
   - Save the `Application (client) ID` and `Directory (tenant) ID`.

2. **Configure API Permissions:**
   - In the App menu, go to **API permissions > Add a permission**.
   - Select **Office 365 Management APIs > Application permissions**.
   - Expand **ActivityFeed** and check `ActivityFeed.Read` and `ActivityFeed.ReadDlp`.
   - Click **Add permissions**, then click **Grant admin consent for <your-org>** and confirm.

3. **Create Client Secret:**
   - Go to **Certificates & secrets > New client secret**.
   - Provide a description and expiration, then click **Add**.
   - Immediately copy the **Secret Value** (don't copy the Secret ID).

4. **Enable Auditing in Microsoft Purview:**
   - Access the [Microsoft Purview portal](https://purview.microsoft.com).
   - Navigate to **Solutions > Audit**.
   - If auditing isn't enabled, click the banner to **Start recording user and admin activity**. Activation may take up to 48 hours.

#### Vendor resources

For more detailed information on these steps, refer to the following Microsoft documentation:
- [Get started with Office 365 Management APIs - Microsoft Learn](https://learn.microsoft.com/en-us/office/office-365-management-api/get-started-with-office-365-management-apis)

### Set up steps in Kibana

You can configure the integration in Kibana by following these steps:

1. In Kibana, navigate to **Management > Integrations** and search for **Microsoft Office 365**.
2. Click **Add Microsoft Office 365** and select an Elastic Agent policy.
3. Configure the input settings based on your collection method.

#### Collect audit logs using the Management Activity API

This is the recommended input for collecting audit logs. Configure the following fields:

- **Base URL of Office Management API** (`url`): The base endpoint for the API. Default: `https://manage.office.com`.
- **Interval** (`interval`): How often the API is polled. Default: `3m`.
- **Directory (tenant) ID** (`azure_tenant_id`): The unique Directory (tenant) ID from the Entra ID portal.
- **Application (client) ID** (`client_id`): Client ID used for OAuth 2.0 authentication.
- **Client Secret** (`client_secret`): Client secret used for OAuth 2.0 authentication.
- **OAuth 2.0 Token URL** (`token_url`): Base URL endpoint for generating tokens. Default: `https://login.microsoftonline.com`.
- **Content Type** (`content_types`): Comma separated list of content types to fetch. Supported types include `Audit.AzureActiveDirectory`, `Audit.Exchange`, `Audit.SharePoint`, `Audit.General`, and `DLP.All`.
- **Initial Interval** (`initial_interval`): Initial interval for the first API call (max 7 days). Default: `167h55m`.
- **Batch Interval** (`batch_interval`): Interval for each API request (max 24h). Default: `1h`.
- **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event in `event.original`. Default: `false`.
- **Token Scopes** (`token_scopes`): Scopes for OAuth 2.0 token. Default: `['https://manage.office.com/.default']`.
- **Enable request tracing** (`enable_request_tracer`): Logs requests and responses for debugging. You should keep this disabled in production as it compromises security by logging sensitive data. Default: `false`.

#### Deprecated audit log collection

Use this input only if you have a legacy configuration requirement. It's recommended that you migrate to the CEL-based input described above.

- **Application (client) ID** (`application_id`): The Client ID of the registered Entra ID application.
- **Client secret (API key)** (`client_secret`): The secret value generated in the Entra ID portal.
- **Directory (tenant) IDs** (`tenants`): List of tenant IDs. Default: `['tenant-id']`.
- **Content types** (`content_type`): List of workloads to collect. Default: `['Audit.AzureActiveDirectory', 'Audit.Exchange', 'Audit.SharePoint', 'Audit.General', 'DLP.All']`.

After configuring your inputs, click **Save and continue**.

### Validation

To verify the integration is working and data is flowing, perform the following actions:

1. **Trigger data flow in Microsoft Office 365:**
   - Generate Administrative Event: Create a new test user or modify a user's group membership in the **Microsoft Entra admin center**.
   - Generate Exchange Event: Send a test email between accounts or modify mailbox permissions in the **Exchange admin center**.
   - Generate SharePoint Event: Upload a document to a **SharePoint** site to trigger a `FileUploaded` event.

2. **Check data in Kibana:**
   - Navigate to **Analytics > Discover**.
   - Select the `logs-*` data view.
   - Enter the KQL filter: `data_stream.dataset : "o365.audit"`.
   - Verify that log entries appear with recent timestamps.
   - Confirm that `event.dataset` is set to `o365.audit` and `o365.audit.UserId` contains the expected user information.

3. **Check dashboards:**
   - Navigate to **Analytics > Dashboards**.
   - Search for **Microsoft Office 365**.
   - Open a dashboard corresponding to your data and verify that the visualizations are populating with your tenant data.

4. **Check transforms:**
   - Navigate to **Management > Stack Management > Data > Transforms**.
   - Search for `o365`.
   - Verify that all related transforms show a status of **Healthy** in the health column.

## Troubleshooting

For help with Elastic ingest tools, check [Common problems](https://www.elastic.co/docs/troubleshoot/ingest/fleet/common-problems).

### Common configuration issues

If you encounter issues while using the Microsoft Office 365 integration, consider these common problems and solutions:

- Initial latency: When you first enable a subscription, Microsoft documents a latency of up to 12 hours for data to become available. Don't start troubleshooting connectivity until this window has passed.
- Admin consent missing: If you added permissions in the Azure Portal but didn't click "Grant admin consent", the API returns 403 Forbidden errors. Ensure the status in the Azure Portal shows as "Granted".
- Auditing not enabled: If the Purview Audit log is turned off, the API returns empty results even when the integration is correctly configured. Verify your auditing status in the Microsoft Purview portal.
- Common Expression Language (CEL) migration errors: If you're upgrading from a version older than 1.18.0, ensure your Elastic Stack is at least version 8.7.1. Note that certificate-based authentication is not supported in the newer CEL input.
- Missing fields: Some fields only exist for specific workloads. For example, `o365.audit.FileName` only exists for SharePoint and OneDrive events. Check the `event.dataset` field to confirm you're viewing the expected event type.
- Token URL mismatch: Ensure the `OAuth 2.0 Token URL` matches your specific Azure environment. The default is `https://login.microsoftonline.com`.
- Expired client secret: Check that your Client Secret in the Azure Portal hasn't expired. An expired secret causes the OAuth flow to fail with an "invalid_client" error.
- ADAL vs MSAL: The legacy `o365audit` input used the ADAL library, which Microsoft is retiring. If you encounter authentication failures on the deprecated input, you should migrate to the CEL-based input immediately.
- CEL parsing failures: If events don't appear or contain `error.message` fields, check the Elastic Agent logs for CEL expression errors. This usually indicates a change in the upstream JSON schema from Microsoft.
- Request tracing: For deep debugging, you can enable "Enable request tracing" in the integration settings to log raw requests and responses to the agent filesystem. Don't leave this enabled in production as it can log sensitive data.

### Vendor resources

The following Microsoft documentation provides additional details on the Management Activity API and audit logs:

- [Office 365 Management Activity API reference](https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-reference)
- [Audit log record type documentation](https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-schema#auditlogrecordtype)
- [Microsoft Purview audit log setup](https://learn.microsoft.com/en-us/purview/audit-log-enable-disable)

## Performance and scaling

For more information on architectures that can be used for scaling this integration, check the [Ingest Architectures](https://www.elastic.co/docs/manage-data/ingest/ingest-reference-architectures) documentation.

To ensure optimal performance in high-volume environments, consider the following:

### Transport and collection considerations
This integration relies on API polling via the Office 365 Management Activity API. When configuring your collection settings, keep the following in mind:
- Configure the `Interval` (default `3m`) and `Batch Interval` (default `1h`) to balance data freshness with API rate limits.
- Microsoft documents a latency of up to 12 hours for some data to become available for collection. If you adjust the interval too low, it may result in empty API responses if the data isn't yet published by Microsoft.

### Data volume management
You can manage the amount of data ingested using several methods:
- Use the `Content Type` configuration to filter for only required workloads at the source. For example, if identity data is your priority, collecting only `Audit.AzureActiveDirectory` significantly reduces volume compared to enabling all types.
- Use the `Processors` setting to drop unnecessary fields or entire events at the Elastic Agent level before they're transmitted to the Elastic Stack.

### Elastic agent scaling
For high-throughput environments or large tenants with thousands of users, a single agent may reach resource limits. You can scale your deployment using these strategies:
- Deploy multiple Elastic Agents and distribute the workload by configuring different agent policies to collect specific `Content Types`.
- Assign different agents to monitor different `Directory (tenant) IDs`.

## Reference

### Inputs used

{{ inputDocs }}

### API usage

This integration dataset uses the following API to collect data:
- `Audit`: [Office 365 Management Activity API](https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-reference)

### Data streams

#### audit

The `audit` data stream provides events from the Microsoft Office 365 Management Activity API of the following types: Audit.AzureActiveDirectory, Audit.Exchange, Audit.SharePoint, Audit.General, and DLP.All.

##### audit fields

{{fields "audit"}}

##### audit sample event

{{event "audit"}}
