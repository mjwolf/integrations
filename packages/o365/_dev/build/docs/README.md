# Microsoft Office 365 Integration for Elastic

## Overview
The Microsoft Office 365 integration for Elastic enables you to collect audit logs from the [Office 365 Management Activity API](https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-reference). This provides visibility into user, admin, system, and policy actions and events from your Office 365 and Microsoft Entra ID (formerly Azure Active Directory) environments.

This integration facilitates security monitoring, threat detection, and compliance auditing by ingesting detailed activity logs into the Elastic Stack.

### Compatibility
This integration is compatible with any Office 365 subscription that supports the Office 365 Management Activity API.

To use this integration, you need an Elastic Stack version 8.18.0 or later. The `ingest-geoip` and `ingest-user_agent` Elasticsearch plugins are required for full data enrichment.

### How it works
The integration works by creating a subscription for each enabled content type within the Office 365 Management Activity API. It then periodically checks each subscription for available data and downloads any new logs. When a subscription is first created, it can take up to 12 hours for the first data to become available from the API.

## What data does this integration collect?
The Microsoft Office 365 integration collects audit logs, which include detailed records of events and actions from Office 365 and Microsoft Entra ID.

The `audit` data stream provides events from the Office 365 Management Activity API.

### Supported use cases
- **Security Monitoring:** Track user activities, administrative changes, and system events to detect suspicious behavior.
- **Threat Detection:** Identify potential security threats by analyzing sign-in patterns, file access, and other activities.
- **Compliance Auditing:** Maintain a comprehensive log of all activities to meet regulatory and compliance requirements.
- **Incident Response:** Investigate security incidents with detailed audit trails of user and admin actions.

## What do I need to use this integration?
- An Office 365 subscription with Audit Log search enabled. See [Enable Audit Log Search](https://learn.microsoft.com/en-us/purview/audit-log-enable-disable) for instructions.
- A registered application in Microsoft Entra ID with the required API permissions and credentials.

## How do I deploy this integration?

### Agent-based deployment

Elastic Agent must be installed to collect logs and forward them to your Elastic deployment. For more details, check the Elastic Agent [installation instructions](docs-content://reference/fleet/install-elastic-agents.md).

### Onboard / configure

#### 1. Configure Microsoft Entra ID (Azure AD)

Before configuring the integration in Elastic, you must register an application in Microsoft Entra ID and grant it the necessary permissions.

1.  **Register a new application** in your Microsoft Entra ID tenant.
2.  From the application's **Overview** page, note the **Application (client) ID** and **Directory (tenant) ID**. These will be required to configure the integration.
3.  Navigate to the **Certificates & secrets** section.
4.  Click **New client secret**, provide a description, and create the new secret.
5.  Immediately copy the secret's **Value**. This is the only time it will be fully displayed.
6.  Navigate to the **API permissions** page and click **Add a permission**.
7.  Select **Office 365 Management APIs**.
8.  Click **Application permissions**.
9.  Under `ActivityFeed`, select `ActivityFeed.Read`. This is the minimum required permission. Optionally, select `ActivityFeed.ReadDlp` to read DLP policy events.
10. Click **Add permissions**.
11. After adding the permissions, an administrator must grant admin consent for them. Click the **Grant admin consent for [Your Tenant]** button.

#### 2. Configure the Integration in Elastic

1.  In Kibana, navigate to **Management > Integrations**.
2.  Search for "Microsoft Office 365" and select it.
3.  Click **Add Microsoft Office 365**.
4.  Provide a name and description for the integration policy.
5.  Under the integration settings, enter the credentials you obtained from the Microsoft Entra ID application:
    *   **Directory (tenant) ID**: The Directory (tenant) ID from step 2.
    *   **Application (client) ID**: The Application (client) ID from step 2.
    *   **Client Secret**: The client secret `Value` you created in step 5.
6.  Adjust the **Initial Interval** if needed. This setting controls how far back the integration will look for data when it first starts. The default is 7 days, which is the maximum allowed by the API.
7.  Click **Save and continue**.

### Validation
After configuring the integration, allow some time for data to be collected. Note that it can take up to 12 hours for the first logs to become available from the Microsoft API after initial setup.

1.  In Kibana, navigate to **Dashboards**.
2.  Search for `[O365]` to find the pre-built dashboards for this integration.
3.  Select a dashboard and verify that it is populated with data.

## Troubleshooting

For help with Elastic ingest tools, check [Common problems](https://www.elastic.co/docs/troubleshoot/ingest/fleet/common-problems).

If you suspect a permissions issue, it can be useful to decode the OAuth 2.0 token values using a tool like [https://jwt.ms/](https://jwt.ms/). The decoded token should include a `roles` section containing the permissions you configured (e.g., `ActivityFeed.Read`).

## Scaling

For more information on architectures that can be used for scaling this integration, check the [Ingest Architectures](https://www.elastic.co/docs/manage-data/ingest/ingest-reference-architectures) documentation.

## Reference

### audit

The `audit` data stream collects audit messages from Office 365 and Azure AD activity logs. These are the same logs that are available under Audit Log Search in the Security and Compliance Center.

#### audit fields

{{ fields "audit" }}

#### audit sample event

{{ event "audit" }}


### Inputs used
{{ inputDocs }}

### API usage
This integration uses the following API:
* [Office 365 Management Activity API](https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-reference)
