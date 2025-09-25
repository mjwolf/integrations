{{- generatedHeader }}
# 1Password Integration for Elastic

## Overview

The 1Password integration for Elastic collects account activity logs from your 1Password Business account. By using the 1Password Events API, this integration fetches security-related events, enabling you to monitor and analyze them within the Elastic Stack.

This integration facilitates a range of security use cases, including:
- Monitoring sign-in attempts to detect suspicious activity.
- Tracking usage of sensitive items stored in vaults.
- Auditing administrative actions and changes within your 1Password account.
- Correlating 1Password events with other data sources in Elastic Security for comprehensive threat hunting.

### Compatibility

This integration requires a **1Password Business** account. It is compatible with all regional 1Password Events API endpoints (US, Canada, and Europe).

This integration is compatible with Kibana versions `^8.18.0` or `^9.0.0`.

### How it works

The integration periodically polls the 1Password Events API to retrieve new events. The collected events are then ingested into Elastic, where they are parsed, enriched, and stored in dedicated data streams. This process is managed by Elastic Agent.

## What data does this integration collect?

This integration collects several types of log data from the 1Password Events API:

*   **Sign-in Attempts:** Captures details about attempts to sign in to a 1Password account, including successful and failed attempts, user information, and IP addresses.
*   **Item Usages:** Records events related to the access, modification, or use of items stored in shared vaults. This includes who accessed an item, when it was accessed, and from where.
*   **Audit Events:** Collects a detailed log of actions performed by team members, such as account updates, invitations, device authorizations, and changes to vault permissions.

### Supported use cases

By centralizing 1Password logs in Elastic, you can:
- **Enhance Security Monitoring:** Build dashboards and alerts in Kibana to monitor for anomalous sign-in patterns or unusual item access.
- **Simplify Compliance and Auditing:** Maintain a long-term, searchable archive of all administrative and user actions within 1Password.
- **Accelerate Incident Response:** Use Elastic Security to correlate 1Password events with data from other security tools, providing a unified view for investigations.
- **Control Data Retention:** Manage your 1Password data retention policies within Elastic to meet your organization's specific needs.

## What do I need to use this integration?

- A **1Password Business** account with owner or administrator privileges.
- A **Bearer Token** for the 1Password Events API.

## How do I deploy this integration?

### Agent-based deployment

Elastic Agent must be installed. For more details, check the Elastic Agent [installation instructions](docs-content://reference/fleet/install-elastic-agents.md). You can install only one Elastic Agent per host.

Elastic Agent is required to stream data from the 1Password Events API and ship it to Elastic, where the events will then be processed via the integration's ingest pipelines.

### Agentless deployment

Agentless deployments are only supported in Elastic Serverless and Elastic Cloud environments. Agentless deployments provide a means to ingest data while avoiding the orchestration, management, and maintenance needs associated with standard ingest infrastructure. Using an agentless deployment makes manual agent deployment unnecessary, allowing you to focus on your data instead of the agent that collects it.

For more information, refer to [Agentless integrations](https://www.elastic.co/guide/en/serverless/current/security-agentless-integrations.html) and [Agentless integrations FAQ](https://www.elastic.co/guide/en/serverless/current/agentless-integration-troubleshooting.html). This functionality is in beta and is subject to change.

### Onboard / configure

#### 1. Configure 1Password

To generate the necessary credentials, you must set up the Events API integration within your 1Password Business account.

1.  Sign in to your account on `1Password.com`.
2.  From the sidebar, navigate to **Integrations**.
3.  Under the "Events Reporting" section, choose **Elastic**.
4.  Enter a name for the integration (e.g., "Elastic SIEM") and click **Create Integration**.
5.  On the next screen, copy the generated **Bearer Token**. This token is required to authenticate with the 1Password Events API and will be used in the next step.

For more details, refer to the official [1Password Events Reporting documentation](https://support.1password.com/events-reporting-elastic/).

#### 2. Configure the Elastic Integration

1.  In Kibana, navigate to **Management > Integrations**.
2.  In the search bar, type **1Password** and select the integration.
3.  Click **Add 1Password**.
4.  Configure the integration settings:
    *   **URL of 1Password Events API Server**: Select the URL corresponding to your 1Password account's region (`https://events.1password.com`, `https://events.1password.ca`, or `https://events.1password.eu`).
    *   **1Password Authorization Token**: Paste the Bearer Token you copied from the 1Password admin console.
5.  Click **Save and continue**. This will enroll the agent in a policy to begin collecting data.

### Validation

1.  In Kibana, navigate to the **Discover** tab.
2.  Select the appropriate data view, such as `logs-1password.audit_events-*`.
3.  Verify that log events are being populated. It may take a few minutes for the initial API poll to occur.
4.  Alternatively, navigate to the pre-built dashboards for 1Password to see visualizations of the event data.

## Troubleshooting

For help with Elastic ingest tools, check [Common problems](https://www.elastic.co/docs/troubleshoot/ingest/fleet/common-problems).

- **401 Unauthorized Errors**: This typically indicates an invalid Bearer Token. Verify that the token was copied correctly and has not been revoked in the 1Password admin console.
- **No Data Appearing**: Ensure the correct regional URL for the 1Password Events API Server is selected in the integration configuration.

## Scaling

For more information on architectures that can be used for scaling this integration, check the [Ingest Architectures](https://www.elastic.co/docs/manage-data/ingest/ingest-reference-architectures) documentation. The `httpjson` input is lightweight, but for very large 1Password organizations, you may need to adjust the polling interval and consider a dedicated Elastic Agent for this task.

## Reference

### audit_events

The `audit_events` data stream collects information about actions performed by team members, such as account updates, access and invitations, device authorization, and changes to vault permissions.

#### audit_events fields

{{ fields "audit_events" }}

#### audit_events event sample

{{ event "audit_events" }}

### item_usages

The `item_usages` data stream retrieves information about items in shared vaults that have been modified, accessed, or used.

#### item_usages fields

{{ fields "item_usages" }}

#### item_usages event sample

{{ event "item_usages" }}

### signin_attempts

The `signin_attempts` data stream retrieves information about sign-in attempts, including successful and failed attempts.

#### signin_attempts fields

{{ fields "signin_attempts" }}

#### signin_attempts event sample

{{ event "signin_attempts" }}

### Inputs used

{{ inputDocs }}

### API usage

This integration uses the [1Password Events API](https://developer.1password.com/docs/events/get-started).
