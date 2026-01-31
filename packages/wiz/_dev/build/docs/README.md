# Wiz Integration for Elastic

> **Note**: This documentation was generated using AI and should be reviewed for accuracy.

## Overview

The Wiz integration for Elastic enables you to centralize your cloud security posture data, providing a unified view of risks across multi-cloud environments. By ingesting high-fidelity alerts and configuration findings into the Elastic Stack, your security teams can correlate cloud vulnerabilities with runtime telemetry for comprehensive threat hunting. By combining Wiz's graph-based analysis with Elastic's search power, you'll be able to focus on "toxic combinations"—the specific risks that present the highest danger to your environment.

This integration facilitates:
- Continuous compliance monitoring: Monitor cloud configuration findings against industry benchmarks like CIS or SOC2 by ingesting posture data to ensure resources remain compliant over time.
- Real-time threat detection: Utilize the `defend` data stream to receive immediate notifications of runtime threats, such as container escapes or lateral movement, directly within the Elastic Security SIEM.
- Vulnerability prioritization: Analyze vulnerability logs alongside active threat intelligence in Elastic to prioritize patching efforts based on actual exposure and resource criticality.
- Audit and governance: Maintain a long-term record of administrative actions and platform mutations—including write, edit, and delete actions—to meet regulatory requirements and support forensic investigations.

### Compatibility

This integration is compatible with the following vendor specifications:
- Wiz API version v1.
- Wiz Defend runtime threat detection module.
- Elastic Agent managed by Fleet or Agentless (Serverless/Elastic Cloud) ingestion methods.

### How it works

This integration collects data from the Wiz platform using both API polling and HTTP webhooks. Each data stream maps specific Wiz security signals to the Elastic Common Schema (ECS) to ensure consistency across your security operations.

The integration uses the following collection methods:
- API polling: The `audit`, `cloud_configuration_finding`, `cloud_configuration_finding_full_posture`, `issue`, and `vulnerability` data streams use the Common Expression Language (`cel`) input to poll the Wiz API for the latest security data and configuration states.
- HTTP webhooks: The `defend` data stream uses the `http_endpoint` input to receive real-time detection events and runtime signals from Wiz Defend, such as suspicious process executions or network anomalies.

Once you've configured the integration, the Elastic Agent processes these signals and forwards them to your Elastic deployment. You'll then be able to monitor your cloud security posture using pre-built dashboards and automated security rules.

## What data does this integration collect?

The Wiz integration collects various types of logs and security signals to help you monitor and secure your cloud environment. You'll receive data through the following data streams:

*   Audit logs: The integration collects audit logs in the `audit` data stream using the `cel` input. This stream records administrative actions within the Wiz portal, such as user logins and API mutation calls like write, edit, and delete actions.
*   Cloud configuration findings: You'll receive logs in the `cloud_configuration_finding` data stream when your cloud resources fail specific configuration rules. These signals indicate potential security misconfigurations that you may need to remediate.
*   Cloud configuration finding full posture: The `cloud_configuration_finding_full_posture` data stream provides a comprehensive view of the posture state for all your cloud configuration findings, giving you total visibility into your cloud environment.
*   Defend logs: You can collect real-time detection events and runtime signals through the `defend` data stream. This stream uses an `http_endpoint` input to capture threats such as suspicious process executions or network anomalies identified by Wiz Defend.
*   Issue logs: The integration tracks active security risks and threats, such as exposed secrets, orphaned resources, or overly permissive identities, in the `issue` data stream.
*   Vulnerability logs: You'll get detailed information about software weaknesses and known vulnerabilities (CVEs) in the `vulnerability` data stream. This includes vulnerabilities identified across your cloud workloads, container images, and virtual machine instances.

### Supported use cases

Integrating Wiz logs with the Elastic Stack helps you enhance your security posture and operational visibility. You can use this data for the following:

*   Security posture management: You'll be able to use the configuration and posture data to maintain a secure cloud environment and quickly identify resources that violate your security policies.
*   Real-time threat detection: By correlating Wiz Defend detection events with other security data in Elastic, you can identify and respond to active threats and suspicious runtime activity as they occur.
*   Compliance and auditing: The `audit` data stream allows you to track administrative changes and user activity, helping you meet compliance requirements and perform internal security audits.
*   Vulnerability management: You can prioritize your remediation efforts by analyzing software vulnerabilities across your entire cloud estate within Kibana dashboards.

## What do I need to use this integration?

To collect logs and events from Wiz, you must prepare your Wiz environment and ensure your Elastic Stack meets the necessary requirements.

### Wiz prerequisites

You must configure the following settings in your Wiz portal before setting up the integration:
- A Wiz user account with **Project Admin** or **Global Admin** privileges to manage Service Accounts and Webhooks.
- A Service Account with a **Client ID** and **Client Secret**, which you can create by navigating to **Settings > Service Accounts**.
- The following scopes assigned to your Service Account: `admin:audit`, `read:issues`, `read:vulnerabilities`, and `read:cloud_configuration`.
- Outbound network connectivity from the Elastic Agent host to the Wiz API endpoint (for example, `https://api.us1.app.wiz.io`) and the authentication endpoint `https://auth.app.wiz.io/oauth/token`.
- If you're using the `defend` data stream for webhook-based logs, ensure your Elastic Agent host is reachable from Wiz's infrastructure on the configured **Listen Port** (the default is `9588`).

### Elastic prerequisites

Your Elastic environment must meet these requirements:
- An Elastic Agent must be installed and enrolled in Fleet, or configured as a standalone agent.
- The Elastic Agent requires outbound HTTPS access on port `443` to communicate with the Wiz API endpoint and the OAuth2 token URL.

## How do I deploy this integration?

### Agent-based deployment

You must install Elastic Agent to use this integration. For detailed steps, check the Elastic Agent [installation instructions](https://www.elastic.co/guide/en/fleet/current/elastic-agent-installation.html). You'll only need to install one Elastic Agent per host.

Elastic Agent is required to stream data from the syslog or log file receiver and ship it to Elastic. Once the data reaches Elastic, the integration's ingest pipelines'll process the events.

### Agentless deployment

Agentless deployments are only supported in Elastic Serverless and Elastic Cloud environments. Agentless deployments provide a means to ingest data while avoiding the orchestration, management, and maintenance needs associated with standard ingest infrastructure. Using an agentless deployment makes manual agent deployment unnecessary, allowing you to focus on your data instead of the agent that collects it.

For more information, refer to [Agentless integrations](https://www.elastic.co/guide/en/serverless/current/security-agentless-integrations.html) and [Agentless integrations FAQ](https://www.elastic.co/guide/en/serverless/current/agentless-integration-troubleshooting.html).

### Set up steps in Wiz

You need to configure Wiz to share data with Elastic. Depending on the logs you want to collect, you'll use either API polling or a webhook.

#### API polling (Audit, Issue, Vulnerability, and Cloud Configuration)

Follow these steps to set up API access:

1.  Log in to your Wiz tenant and navigate to your **User Profile** (top right) > **User Settings** > **Tenant**.
2.  Copy the **API Endpoint URL** (for example, `https://api.us1.app.wiz.io`) so you can use it later in Kibana.
3.  Navigate to **Settings > Service Accounts** and click **Add Service Account**.
4.  Provide a descriptive name like `Elastic-Integration` and set the type to **Custom Integration (GraphQL API)**.
5.  In the Scopes section, assign the following permissions:
    *   `admin:audit` (for Audit logs)
    *   `read:issues` (for Issues)
    *   `read:vulnerabilities` (for Vulnerabilities)
    *   `read:cloud_configuration` (for Cloud Configuration findings)
6.  Click **Add Service Account**.
7.  Immediately copy the **Client ID** and **Client Secret** provided in the confirmation dialog. You'll need to store these securely because you can't retrieve the secret again.

#### Webhook collection (Defend logs)

Follow these steps to set up a webhook for detection events:

1.  Log in to the Wiz console and navigate to **Settings > Integrations**.
2.  Click **Add Integration** and select the **Webhook** option under SIEM & Automation Tools.
3.  In the New Integration page, enter a name like `Elastic Agent Defend`.
4.  In the **URL** field, enter the destination address of your Elastic Agent (for example, `http://<AGENT_IP>:9588/`).
5.  Choose an **Authentication** method. We recommend using **Token** for easier configuration within the Elastic Agent. Enter a strong, unique string as the token.
6.  (Optional) Configure Project Scopes or Filters to limit the alerts sent to Elastic.
7.  Click **Add Integration**.
8.  Ensure your Defend policies in Wiz are configured to send alerts to this specific integration.

#### Vendor resources

For more information, refer to the following Wiz documentation:

*   [Wiz Documentation Portal](https://docs.wiz.io/)

### Set up steps in Kibana

You'll configure the Wiz integration in Kibana to begin ingesting data. Follow these steps:

1.  In Kibana, navigate to **Management > Integrations**.
2.  Search for **Wiz** and select the integration.
3.  Click **Add Wiz**.
4.  Configure the global settings using the credentials you created in Wiz:
    *   **Client ID**: The Client ID for your Wiz Service Account.
    *   **Client Secret**: The Client Secret for your Wiz Service Account.
    *   **URL**: The base URL of the Wiz API (for example, `https://api.us1.app.wiz.io`).

Choose the configuration steps below for the specific data streams you want to enable.

#### Collecting Wiz logs via API (Cloud Configuration Finding full posture)

Configure these settings for full posture logs:

1.  **Preserve original event**: (Default: `false`) Enable this if you want to save the raw JSON in `event.original`.
2.  **Batch Size**: (Default: `500`) The number of records to fetch per API request.
3.  **HTTP Client Timeout**: (Default: `30s`) The time to wait before the connection times out.
4.  **Maximum Pages Per Interval**: (Default: `1000`) The limit for the number of pages fetched per polling interval.
5.  **Enable request tracing**: (Default: `false`) Only enable this for debugging to log requests and responses.
6.  **Tags**: List of tags to add to the events (Default: `forwarded`, `wiz-cloud_configuration_finding_full_posture`).
7.  **Preserve duplicate custom fields**: (Default: `false`) Keeps original Wiz fields that were mapped to ECS.

#### Collecting Wiz logs via API (Cloud Configuration Finding logs)

Configure these settings for finding logs:

1.  **Initial Interval**: (Default: `24h`) How far back to look for logs during the first collection.
2.  **Interval**: (Default: `5m`) The frequency of API polling.
3.  **Preserve original event**: (Default: `false`) Enable this to save the raw JSON in `event.original`.
4.  **Batch Size**: (Default: `500`) The number of records per request.
5.  **HTTP Client Timeout**: (Default: `30s`) The connection timeout duration.
6.  **Maximum Pages Per Interval**: (Default: `1000`) The maximum pages per polling cycle.
7.  **Tags**: (Default: `forwarded`, `wiz-cloud_configuration_finding`).

#### Collecting Wiz logs via API (Audit logs)

Configure these settings for audit logs:

1.  **Initial Interval**: (Default: `24h`) The lookback period for the initial ingest.
2.  **Interval**: (Default: `5m`) The polling frequency.
3.  **Preserve original event**: (Default: `false`) Enable this to save the raw JSON.
4.  **Batch Size**: (Default: `500`) The records per request.
5.  **HTTP Client Timeout**: (Default: `30s`) The timeout settings.
6.  **Tags**: (Default: `forwarded`, `wiz-audit`).

#### Collecting Wiz logs via API (Issue logs)

Configure these settings for issue logs:

1.  **Initial Interval**: (Default: `24h`) The initial data lookback.
2.  **Interval**: (Default: `5m`) The API poll frequency.
3.  **Preserve original event**: (Default: `false`) Enable this to save raw JSON.
4.  **Batch Size**: (Default: `500`) The records per call.
5.  **Maximum Pages Per Interval**: (Default: `1000`) The page limit per interval.
6.  **Tags**: (Default: `forwarded`, `wiz-issue`).

#### Collecting Wiz logs via API (Vulnerability logs)

Configure these settings for vulnerability logs:

1.  **Include All Vulnerability Statuses**: (Default: `false`) Toggle this to fetch `RESOLVED`, `REJECTED`, and `IN_PROGRESS` vulnerabilities in addition to `OPEN` ones.
2.  **Initial Interval**: (Default: `24h`) The initial ingest lookback.
3.  **Interval**: (Default: `5m`) The polling frequency.
4.  **Preserve original event**: (Default: `false`) Save a raw copy in `event.original`.
5.  **Batch Size**: (Default: `500`) The maximum batch size.
6.  **Tags**: (Default: `forwarded`, `wiz-vulnerability`).

#### Collecting Detection events from Wiz Defend via HTTP Endpoint

Configure these settings for the webhook listener:

1.  **Listen Address**: (Default: `localhost`) The address the agent'll listen on. Use `0.0.0.0` for all interfaces.
2.  **Listen Port**: (Default: `9588`) The port number for the webhook listener.
3.  **Authentication (Basic)**: (Default: `false`) Enable this for username and password authentication.
4.  **Username**: The username if Basic Auth is enabled.
5.  **Password**: The password if Basic Auth is enabled.
6.  **Authentication (Token)**: The secret token value required to authenticate the Wiz webhook.
7.  **URL**: (Default: `/`) The path where the webhook's accepted.
8.  **Tags**: (Default: `forwarded`, `wiz-defend`).
9.  **Preserve duplicate custom fields**: (Default: `false`) Keeps original fields after ECS mapping.

### Validation

After you've completed the configuration, verify that data's flowing correctly.

#### Trigger data flow on Wiz

Follow these steps to generate events in Wiz for testing:

*   **Generate audit event**: Log out and log back into the Wiz administration interface to trigger an authentication audit log.
*   **Generate issue event**: Navigate to an existing issue in the Wiz UI and change its status or add a comment to trigger an update.
*   **Trigger vulnerability scan**: If you can, initiate a manual scan on a specific resource to generate new vulnerability logs.
*   **Test webhook**: Navigate to **Settings > Integrations** in Wiz, locate your Webhook integration, and click the **Test** button to send a sample payload to the Elastic Agent.

#### Check data in Kibana

Follow these steps to verify the data in your Elastic environment:

1.  Navigate to **Analytics > Discover**.
2.  Select the `logs-*` data view.
3.  Enter the KQL filter `data_stream.dataset : "wiz.audit"` (or substitute the dataset you're testing).
4.  Verify that logs appear and confirm these fields are present:
    *   `event.dataset` (should match `wiz.audit`, `wiz.issue`, etc.)
    *   `source.ip` (populated for `wiz.defend` webhook events)
    *   `event.action` (for example, `info-system-login`)
    *   `message` (contains the raw log payload)
5.  Navigate to **Analytics > Dashboards** and search for **Wiz** to verify that the out-of-the-box dashboards are populating with data.

## Troubleshooting

For help with common Elastic ingest issues, see [Common problems](https://www.elastic.co/docs/troubleshoot/ingest/fleet/common-problems).

### Common configuration issues

You may encounter the following issues when configuring or running the Wiz integration:

- `event.ingested` field is missing: When you use standalone Elastic Agents, this field isn't added automatically. This can cause transforms to fail. You should manually add this field using a custom ingest pipeline like `@custom`.
- Webhook port is unreachable: If you don't see `defend` logs, check that the Elastic Agent host firewall allows inbound traffic on the configured port, which defaults to `9588`. Ensure you set the **Listen Address** to `0.0.0.0` if the agent is receiving traffic from outside the local host.
- Specific data streams are missing data: If some data streams like `vulnerability` are empty while others like `issue` work, check your Wiz Service Account. Verify that you've assigned the exact required scopes, such as `read:vulnerabilities`, in the Wiz portal.
- Vulnerability logs do not appear immediately: These logs are fetched based on the polling interval. If they don't appear after setup, check the agent logs for `403 Forbidden` or `401 Unauthorized` errors, which usually indicate credential or scope issues.
- Parsing failures occur: Check the `error.message` field in Kibana Discover. These can happen if the Wiz API schema changes. You should ensure the integration is updated to the latest version to resolve schema mismatches.
- OAuth token failures occur: Confirm that the **Token URL** is correct, typically `https://auth.app.wiz.io/oauth/token`. If you see `401 Unauthorized` errors in the agent logs, your **Client ID** or **Client Secret** might be incorrect or expired.

### Vendor resources

For more information about configuring Wiz integrations, see these resources:

- [Wiz Webhook Integration Documentation](https://docs.wiz.io/docs/webhook-integration)
- [Wiz API Documentation](https://docs.wiz.io/docs/wiz-api)

## Performance and scaling

To ensure optimal performance in high-volume cloud security environments, you should consider the following configuration adjustments:
- This integration uses a hybrid model. It uses the `cel` input to pull findings like `audit`, `issue`, and `vulnerability` logs via the GraphQL API, and the `http_endpoint` input to receive pushed `defend` logs. For API collection, you can tune the `Batch Size` (default `500`) and `Maximum Pages Per Interval` (default `1000`) based on the size of your cloud environment. High volumes of vulnerability data may require you to increase the `HTTP Client Timeout` beyond the default `30s`.
- You can manage data volume by configuring Wiz Service Account scopes to only include the specific projects or data types you need. For the `vulnerability` data stream, use the `Include All Vulnerability Statuses` toggle selectively; by default, the integration uses the API's default filters to reduce noise. Limiting the `Initial Interval` (default `24h`) can prevent excessive data retrieval during your initial synchronization.
- For high-throughput environments, especially those receiving many `defend` webhook alerts, you can deploy multiple Elastic Agents behind a network load balancer to distribute the incoming HTTPS traffic. Ensure your Agent host has enough resources to process the CEL script logic and large JSON payloads, particularly if you've enabled `Preserve original event`.

For more information on architectures that can be used for scaling this integration, check the [Ingest Architectures](https://www.elastic.co/docs/manage-data/ingest/ingest-reference-architectures) documentation.

## Reference

### Inputs used

{{ inputDocs }}

### API usage

These APIs are used with this integration:
* [Wiz GraphQL API](https://docs.wiz.io/docs/graphql-api-overview)

### Data streams

#### audit

The `audit` data stream provides events from Wiz of the following types: audit logs.

##### audit fields

{{ fields "audit" }}

##### audit sample event

{{ event "audit" }}

#### cloud_configuration_finding

The `cloud_configuration_finding` data stream provides events from Wiz of the following types: cloud configuration findings.

##### cloud_configuration_finding fields

{{ fields "cloud_configuration_finding" }}

##### cloud_configuration_finding sample event

{{ event "cloud_configuration_finding" }}

#### cloud_configuration_finding_full_posture

The `cloud_configuration_finding_full_posture` data stream provides events from Wiz of the following types: cloud configuration findings.

##### cloud_configuration_finding_full_posture fields

{{ fields "cloud_configuration_finding_full_posture" }}

##### cloud_configuration_finding_full_posture sample event

{{ event "cloud_configuration_finding_full_posture" }}

#### defend

The `defend` data stream provides events from Wiz of the following types: detection events from Wiz Defend.

##### defend fields

{{ fields "defend" }}

##### defend sample event

{{ event "defend" }}

#### issue

The `issue` data stream provides events from Wiz of the following types: security issues.

##### issue fields

{{ fields "issue" }}

##### issue sample event

{{ event "issue" }}

#### vulnerability

The `vulnerability` data stream provides events from Wiz of the following types: software vulnerabilities.

##### vulnerability fields

{{ fields "vulnerability" }}

##### vulnerability sample event

{{ event "vulnerability" }}
