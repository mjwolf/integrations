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
<details>
<summary>http_endpoint</summary>

## Setup

For more details about the Http endpoint input settings, check the [Filebeat documentation](https://www.elastic.co/docs/reference/beats/filebeat/filebeat-input-http_endpoint).

### Collecting logs from Http Endpoint

To collect logs via Http Endpoint, select **Collect logs via Http Endpoint** and configure the following parameters:

- Listen Address: Bind address for the HTTP listener. Use 0.0.0.0 to listen on all interfaces.
- Listen port: Bind port for the listener.
</details>


### API usage

These APIs are used with this integration:
* [Wiz GraphQL API](https://docs.wiz.io/docs/graphql-api-overview)

### Data streams

#### audit

The `audit` data stream provides events from Wiz of the following types: audit logs.

##### audit fields

**Exported fields**

| Field | Description | Type |
|---|---|---|
| @timestamp | Date/time when the event originated. This is the date/time extracted from the event, typically representing when the event was generated by the source. If the event source has no original timestamp, this value is typically populated by the first time the event was received by the pipeline. Required field for all events. | date |
| data_stream.dataset | The field can contain anything that makes sense to signify the source of the data. Examples include `nginx.access`, `prometheus`, `endpoint` etc. For data streams that otherwise fit, but that do not have dataset set we use the value "generic" for the dataset value. `event.dataset` should have the same value as `data_stream.dataset`. Beyond the Elasticsearch data stream naming criteria noted above, the `dataset` value has additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |
| data_stream.namespace | A user defined namespace. Namespaces are useful to allow grouping of data. Many users already organize their indices this way, and the data stream naming scheme now provides this best practice as a default. Many users will populate this field with `default`. If no value is used, it falls back to `default`. Beyond the Elasticsearch index naming criteria noted above, `namespace` value has the additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |
| data_stream.type | An overarching type for the data stream. Currently allowed values are "logs" and "metrics". We expect to also add "traces" and "synthetics" in the near future. | constant_keyword |
| event.dataset | Name of the dataset. If an event source publishes more than one type of log or events (e.g. access log, error log), the dataset is used to specify which one the event comes from. It's recommended but not required to start the dataset name with the module name, followed by a dot, then the dataset name. | constant_keyword |
| event.module | Name of the module this data is coming from. If your monitoring agent supports the concept of modules or plugins to process events of a given source (e.g. Apache logs), `event.module` should contain the name of this module. | constant_keyword |
| input.type | Type of filebeat input. | keyword |
| log.offset | Log offset. | long |
| wiz.audit.action |  | keyword |
| wiz.audit.action_parameters.client_id |  | keyword |
| wiz.audit.action_parameters.groups |  | flattened |
| wiz.audit.action_parameters.name |  | keyword |
| wiz.audit.action_parameters.products |  | keyword |
| wiz.audit.action_parameters.role |  | keyword |
| wiz.audit.action_parameters.scopes |  | keyword |
| wiz.audit.action_parameters.user.email |  | keyword |
| wiz.audit.action_parameters.user.id |  | keyword |
| wiz.audit.action_parameters.userpool_id |  | keyword |
| wiz.audit.id |  | keyword |
| wiz.audit.request_id |  | keyword |
| wiz.audit.service_account.id |  | keyword |
| wiz.audit.service_account.name |  | keyword |
| wiz.audit.source_ip |  | ip |
| wiz.audit.status |  | keyword |
| wiz.audit.timestamp |  | date |
| wiz.audit.user.id |  | keyword |
| wiz.audit.user.name |  | keyword |
| wiz.audit.user_agent |  | keyword |


##### audit sample event

An example event for `audit` looks as following:

```json
{
    "@timestamp": "2023-07-21T07:07:21.105Z",
    "agent": {
        "ephemeral_id": "ea58853f-b6e9-4a45-86ba-9551c6aec28f",
        "id": "83d115a5-188d-46b5-95ce-7c8e49e04018",
        "name": "elastic-agent-37311",
        "type": "filebeat",
        "version": "8.18.0"
    },
    "data_stream": {
        "dataset": "wiz.audit",
        "namespace": "68164",
        "type": "logs"
    },
    "ecs": {
        "version": "8.11.0"
    },
    "elastic_agent": {
        "id": "83d115a5-188d-46b5-95ce-7c8e49e04018",
        "snapshot": true,
        "version": "8.18.0"
    },
    "event": {
        "action": "login",
        "agent_id_status": "verified",
        "category": [
            "authentication"
        ],
        "dataset": "wiz.audit",
        "id": "hhd8ab9c-f1bf-4a80-a1e1-13bc8769caf4",
        "ingested": "2025-04-22T09:53:49Z",
        "kind": "event",
        "original": "{\"action\":\"Login\",\"actionParameters\":{\"clientID\":\"afsdafasmdgj5c\",\"groups\":null,\"name\":\"example\",\"products\":[\"*\"],\"role\":\"\",\"scopes\":[\"read:issues\",\"read:reports\",\"read:vulnerabilities\",\"update:reports\",\"create:reports\",\"admin:audit\"],\"userEmail\":\"\",\"userID\":\"afsafasdghbhdfg5t35fdgs\",\"userpoolID\":\"us-east-2_GQ3gwvxsQ\"},\"id\":\"hhd8ab9c-f1bf-4a80-a1e1-13bc8769caf4\",\"requestId\":\"hhd8ab9c-f1bf-4a80-a1e1-13bc8769caf4\",\"serviceAccount\":{\"id\":\"mlipebtwsndhxdmnzdwrxzmiolvzt6topjvv4nugzctcsyarazrhg\",\"name\":\"elastic\"},\"sourceIP\":null,\"status\":\"SUCCESS\",\"timestamp\":\"2023-07-21T07:07:21.105685Z\",\"user\":null,\"userAgent\":null}",
        "outcome": "success",
        "type": [
            "info"
        ]
    },
    "http": {
        "request": {
            "id": "hhd8ab9c-f1bf-4a80-a1e1-13bc8769caf4"
        }
    },
    "input": {
        "type": "cel"
    },
    "related": {
        "user": [
            "afsafasdghbhdfg5t35fdgs",
            "us-east-2_GQ3gwvxsQ"
        ]
    },
    "tags": [
        "preserve_original_event",
        "preserve_duplicate_custom_fields",
        "forwarded",
        "wiz-audit"
    ],
    "wiz": {
        "audit": {
            "action": "Login",
            "action_parameters": {
                "client_id": "afsdafasmdgj5c",
                "name": "example",
                "products": [
                    "*"
                ],
                "scopes": [
                    "read:issues",
                    "read:reports",
                    "read:vulnerabilities",
                    "update:reports",
                    "create:reports",
                    "admin:audit"
                ],
                "user": {
                    "id": "afsafasdghbhdfg5t35fdgs"
                },
                "userpool_id": "us-east-2_GQ3gwvxsQ"
            },
            "id": "hhd8ab9c-f1bf-4a80-a1e1-13bc8769caf4",
            "request_id": "hhd8ab9c-f1bf-4a80-a1e1-13bc8769caf4",
            "service_account": {
                "id": "mlipebtwsndhxdmnzdwrxzmiolvzt6topjvv4nugzctcsyarazrhg",
                "name": "elastic"
            },
            "status": "SUCCESS",
            "timestamp": "2023-07-21T07:07:21.105Z"
        }
    }
}
```

#### cloud_configuration_finding

The `cloud_configuration_finding` data stream provides events from Wiz of the following types: cloud configuration findings.

##### cloud_configuration_finding fields

**Exported fields**

| Field | Description | Type |
|---|---|---|
| @timestamp | Date/time when the event originated. This is the date/time extracted from the event, typically representing when the event was generated by the source. If the event source has no original timestamp, this value is typically populated by the first time the event was received by the pipeline. Required field for all events. | date |
| data_stream.dataset | The field can contain anything that makes sense to signify the source of the data. Examples include `nginx.access`, `prometheus`, `endpoint` etc. For data streams that otherwise fit, but that do not have dataset set we use the value "generic" for the dataset value. `event.dataset` should have the same value as `data_stream.dataset`. Beyond the Elasticsearch data stream naming criteria noted above, the `dataset` value has additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |
| data_stream.namespace | A user defined namespace. Namespaces are useful to allow grouping of data. Many users already organize their indices this way, and the data stream naming scheme now provides this best practice as a default. Many users will populate this field with `default`. If no value is used, it falls back to `default`. Beyond the Elasticsearch index naming criteria noted above, `namespace` value has the additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |
| data_stream.type | An overarching type for the data stream. Currently allowed values are "logs" and "metrics". We expect to also add "traces" and "synthetics" in the near future. | constant_keyword |
| event.dataset | Name of the dataset. If an event source publishes more than one type of log or events (e.g. access log, error log), the dataset is used to specify which one the event comes from. It's recommended but not required to start the dataset name with the module name, followed by a dot, then the dataset name. | constant_keyword |
| event.module | Name of the module this data is coming from. If your monitoring agent supports the concept of modules or plugins to process events of a given source (e.g. Apache logs), `event.module` should contain the name of this module. | constant_keyword |
| input.type | Type of filebeat input. | keyword |
| log.offset | Log offset. | long |
| resource.id |  | keyword |
| resource.name |  | keyword |
| resource.sub_type |  | keyword |
| resource.type |  | keyword |
| result.evaluation |  | keyword |
| result.evidence.cloud_configuration_link |  | text |
| result.evidence.configuration_path |  | text |
| result.evidence.current_value |  | text |
| result.evidence.expected_value |  | text |
| rule.remediation |  | keyword |
| tags | List of keywords used to tag each event. | keyword |
| wiz.cloud_configuration_finding.analyzed_at |  | date |
| wiz.cloud_configuration_finding.evidence.cloud_configuration_link |  | text |
| wiz.cloud_configuration_finding.evidence.configuration_path |  | text |
| wiz.cloud_configuration_finding.evidence.current_value |  | text |
| wiz.cloud_configuration_finding.evidence.expected_value |  | text |
| wiz.cloud_configuration_finding.id |  | keyword |
| wiz.cloud_configuration_finding.resource.cloud_platform |  | keyword |
| wiz.cloud_configuration_finding.resource.id |  | keyword |
| wiz.cloud_configuration_finding.resource.name |  | keyword |
| wiz.cloud_configuration_finding.resource.native_type |  | keyword |
| wiz.cloud_configuration_finding.resource.provider_id |  | keyword |
| wiz.cloud_configuration_finding.resource.region |  | keyword |
| wiz.cloud_configuration_finding.resource.subscription.cloud_provider |  | keyword |
| wiz.cloud_configuration_finding.resource.subscription.external_id |  | keyword |
| wiz.cloud_configuration_finding.resource.subscription.name |  | keyword |
| wiz.cloud_configuration_finding.resource.type |  | keyword |
| wiz.cloud_configuration_finding.result |  | keyword |
| wiz.cloud_configuration_finding.rule.description |  | text |
| wiz.cloud_configuration_finding.rule.id |  | keyword |
| wiz.cloud_configuration_finding.rule.name |  | keyword |
| wiz.cloud_configuration_finding.rule.remediation_instructions |  | text |
| wiz.cloud_configuration_finding.rule.short_id |  | keyword |


##### cloud_configuration_finding sample event

An example event for `cloud_configuration_finding` looks as following:

```json
{
    "@timestamp": "2024-08-07T12:55:52.012Z",
    "agent": {
        "ephemeral_id": "3fdb83a8-3bce-4186-8cee-72dd95c25b4d",
        "id": "4815c547-4daf-42b8-a256-e931be9bc655",
        "name": "elastic-agent-89828",
        "type": "filebeat",
        "version": "8.18.0"
    },
    "cloud": {
        "account": {
            "id": "998231069301",
            "name": "wiz-integrations"
        },
        "provider": "aws",
        "service": {
            "name": "eks"
        }
    },
    "data_stream": {
        "dataset": "wiz.cloud_configuration_finding",
        "namespace": "30878",
        "type": "logs"
    },
    "ecs": {
        "version": "8.11.0"
    },
    "elastic_agent": {
        "id": "4815c547-4daf-42b8-a256-e931be9bc655",
        "snapshot": true,
        "version": "8.18.0"
    },
    "event": {
        "agent_id_status": "verified",
        "category": [
            "configuration"
        ],
        "created": "2024-08-07T12:55:52.012Z",
        "dataset": "wiz.cloud_configuration_finding",
        "id": "1243196d-a365-589a-a8aa-13817c9877b2",
        "ingested": "2025-04-22T09:54:52Z",
        "kind": "state",
        "original": "{\"analyzedAt\":\"2024-08-07T12:55:52.012378Z\",\"id\":\"1243196d-a365-589a-a8aa-13817c9877b2\",\"remediation\":null,\"resource\":{\"cloudPlatform\":\"EKS\",\"id\":\"f0f4163d-cbd7-517c-ba9e-f96bb90ab5ea\",\"name\":\"Root user\",\"nativeType\":\"rootUser\",\"providerId\":\"arn:aws:iam::998231069301:root\",\"region\":null,\"subscription\":{\"cloudProvider\":\"AWS\",\"externalId\":\"998231069301\",\"id\":\"94e76baa-85fd-5928-b829-1669a2ca9660\",\"name\":\"wiz-integrations\"},\"tags\":[],\"type\":\"USER_ACCOUNT\"},\"result\":\"PASS\",\"rule\":{\"description\":\"This rule checks if the AWS Root Account has access keys. \\nThis rule fails if `AccountAccessKeysPresent` is not set to `0`. Note that it does not take into consideration the status of the keys if present. \\nThe root account should avoid using access keys. Since the root account has full permissions across the entire account, creating access keys for it increases the chance that they will be compromised. Instead, it is recommended to create IAM users with predefined roles.\\n\\u003e**Note** \\nSee Cloud Configuration Rule `IAM-207` to see if the Root account's access keys are active.\",\"id\":\"563ed717-4fb6-47fd-929e-9c794e201d0a\",\"name\":\"Root account access keys should not exist\",\"remediationInstructions\":\"Perform the following steps, while being signed in as the Root user, in order to delete the root user's access keys via AWS CLI: \\n1. Use the following command to list the Root user's access keys. \\nCopy the `AccessKeyId` from the output and paste it into the `access-key-id` value in the next step. \\n```\\naws iam list-access-keys\\n```\\n2. Use the following command to delete the access key(s). \\n```\\naws iam delete-access-key /\\n --access-key-id \\u003cvalue\\u003e\\n```\\n\\u003e**Note** \\nOnce an access key is removed, any application using it will not work until a new one is configured for it.\",\"shortId\":\"IAM-006\"},\"severity\":\"MEDIUM\"}",
        "outcome": "success",
        "type": [
            "info"
        ],
        "url": "https://app.wiz.io/findings/configuration-findings/cloud#~(filters~(status~()~rule~(equals~(~'563ed717-4fb6-47fd-929e-9c794e201d0a)))~groupBy~(~)~entity~(~'1243196d-a365-589a-a8aa-13817c9877b2*2cCONFIGURATION_FINDING))"
    },
    "input": {
        "type": "cel"
    },
    "message": "This rule checks if the AWS Root Account has access keys. \nThis rule fails if `AccountAccessKeysPresent` is not set to `0`. Note that it does not take into consideration the status of the keys if present. \nThe root account should avoid using access keys. Since the root account has full permissions across the entire account, creating access keys for it increases the chance that they will be compromised. Instead, it is recommended to create IAM users with predefined roles.\n>**Note** \nSee Cloud Configuration Rule `IAM-207` to see if the Root account's access keys are active.",
    "observer": {
        "vendor": "Wiz"
    },
    "resource": {
        "id": "arn:aws:iam::998231069301:root",
        "name": "Root user",
        "sub_type": "rootUser",
        "type": "USER_ACCOUNT"
    },
    "result": {
        "evaluation": "passed"
    },
    "rule": {
        "description": "This rule checks if the AWS Root Account has access keys. \nThis rule fails if `AccountAccessKeysPresent` is not set to `0`. Note that it does not take into consideration the status of the keys if present. \nThe root account should avoid using access keys. Since the root account has full permissions across the entire account, creating access keys for it increases the chance that they will be compromised. Instead, it is recommended to create IAM users with predefined roles.\n>**Note** \nSee Cloud Configuration Rule `IAM-207` to see if the Root account's access keys are active.",
        "id": "IAM-006",
        "name": "Root account access keys should not exist",
        "remediation": "Perform the following steps, while being signed in as the Root user, in order to delete the root user's access keys via AWS CLI: \n1. Use the following command to list the Root user's access keys. \nCopy the `AccessKeyId` from the output and paste it into the `access-key-id` value in the next step. \n```\naws iam list-access-keys\n```\n2. Use the following command to delete the access key(s). \n```\naws iam delete-access-key /\n --access-key-id <value>\n```\n>**Note** \nOnce an access key is removed, any application using it will not work until a new one is configured for it.",
        "uuid": "563ed717-4fb6-47fd-929e-9c794e201d0a"
    },
    "tags": [
        "preserve_original_event",
        "preserve_duplicate_custom_fields",
        "forwarded",
        "wiz-cloud_configuration_finding"
    ],
    "user": {
        "id": "arn:aws:iam::998231069301:root",
        "name": "Root user"
    },
    "wiz": {
        "cloud_configuration_finding": {
            "analyzed_at": "2024-08-07T12:55:52.012Z",
            "id": "1243196d-a365-589a-a8aa-13817c9877b2",
            "resource": {
                "cloud_platform": "EKS",
                "id": "f0f4163d-cbd7-517c-ba9e-f96bb90ab5ea",
                "name": "Root user",
                "native_type": "rootUser",
                "provider_id": "arn:aws:iam::998231069301:root",
                "subscription": {
                    "cloud_provider": "AWS",
                    "external_id": "998231069301",
                    "name": "wiz-integrations"
                },
                "type": "USER_ACCOUNT"
            },
            "result": "PASS",
            "rule": {
                "description": "This rule checks if the AWS Root Account has access keys. \nThis rule fails if `AccountAccessKeysPresent` is not set to `0`. Note that it does not take into consideration the status of the keys if present. \nThe root account should avoid using access keys. Since the root account has full permissions across the entire account, creating access keys for it increases the chance that they will be compromised. Instead, it is recommended to create IAM users with predefined roles.\n>**Note** \nSee Cloud Configuration Rule `IAM-207` to see if the Root account's access keys are active.",
                "id": "563ed717-4fb6-47fd-929e-9c794e201d0a",
                "name": "Root account access keys should not exist",
                "remediation_instructions": "Perform the following steps, while being signed in as the Root user, in order to delete the root user's access keys via AWS CLI: \n1. Use the following command to list the Root user's access keys. \nCopy the `AccessKeyId` from the output and paste it into the `access-key-id` value in the next step. \n```\naws iam list-access-keys\n```\n2. Use the following command to delete the access key(s). \n```\naws iam delete-access-key /\n --access-key-id <value>\n```\n>**Note** \nOnce an access key is removed, any application using it will not work until a new one is configured for it.",
                "short_id": "IAM-006"
            }
        }
    }
}
```

#### cloud_configuration_finding_full_posture

The `cloud_configuration_finding_full_posture` data stream provides events from Wiz of the following types: cloud configuration findings.

##### cloud_configuration_finding_full_posture fields

**Exported fields**

| Field | Description | Type |
|---|---|---|
| @timestamp | Date/time when the event originated. This is the date/time extracted from the event, typically representing when the event was generated by the source. If the event source has no original timestamp, this value is typically populated by the first time the event was received by the pipeline. Required field for all events. | date |
| data_stream.dataset | The field can contain anything that makes sense to signify the source of the data. Examples include `nginx.access`, `prometheus`, `endpoint` etc. For data streams that otherwise fit, but that do not have dataset set we use the value "generic" for the dataset value. `event.dataset` should have the same value as `data_stream.dataset`. Beyond the Elasticsearch data stream naming criteria noted above, the `dataset` value has additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |
| data_stream.namespace | A user defined namespace. Namespaces are useful to allow grouping of data. Many users already organize their indices this way, and the data stream naming scheme now provides this best practice as a default. Many users will populate this field with `default`. If no value is used, it falls back to `default`. Beyond the Elasticsearch index naming criteria noted above, `namespace` value has the additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |
| data_stream.type | An overarching type for the data stream. Currently allowed values are "logs" and "metrics". We expect to also add "traces" and "synthetics" in the near future. | constant_keyword |
| event.dataset | Name of the dataset. If an event source publishes more than one type of log or events (e.g. access log, error log), the dataset is used to specify which one the event comes from. It's recommended but not required to start the dataset name with the module name, followed by a dot, then the dataset name. | constant_keyword |
| event.module | Name of the module this data is coming from. If your monitoring agent supports the concept of modules or plugins to process events of a given source (e.g. Apache logs), `event.module` should contain the name of this module. | constant_keyword |
| input.type | Type of filebeat input. | keyword |
| log.offset | Log offset. | long |
| resource.id |  | keyword |
| resource.name |  | keyword |
| resource.sub_type |  | keyword |
| resource.type |  | keyword |
| result.evaluation |  | keyword |
| result.evidence.cloud_configuration_link |  | text |
| result.evidence.configuration_path |  | text |
| result.evidence.current_value |  | text |
| result.evidence.expected_value |  | text |
| rule.remediation |  | keyword |
| tags | List of keywords used to tag each event. | keyword |
| wiz.cloud_configuration_finding_full_posture.analyzed_at |  | date |
| wiz.cloud_configuration_finding_full_posture.evidence.cloud_configuration_link |  | text |
| wiz.cloud_configuration_finding_full_posture.evidence.configuration_path |  | text |
| wiz.cloud_configuration_finding_full_posture.evidence.current_value |  | text |
| wiz.cloud_configuration_finding_full_posture.evidence.expected_value |  | text |
| wiz.cloud_configuration_finding_full_posture.id |  | keyword |
| wiz.cloud_configuration_finding_full_posture.name |  | keyword |
| wiz.cloud_configuration_finding_full_posture.resource.cloud_platform |  | keyword |
| wiz.cloud_configuration_finding_full_posture.resource.id |  | keyword |
| wiz.cloud_configuration_finding_full_posture.resource.name |  | keyword |
| wiz.cloud_configuration_finding_full_posture.resource.native_type |  | keyword |
| wiz.cloud_configuration_finding_full_posture.resource.provider_id |  | keyword |
| wiz.cloud_configuration_finding_full_posture.resource.region |  | keyword |
| wiz.cloud_configuration_finding_full_posture.resource.subscription.cloud_provider |  | keyword |
| wiz.cloud_configuration_finding_full_posture.resource.subscription.external_id |  | keyword |
| wiz.cloud_configuration_finding_full_posture.resource.subscription.name |  | keyword |
| wiz.cloud_configuration_finding_full_posture.resource.type |  | keyword |
| wiz.cloud_configuration_finding_full_posture.result |  | keyword |
| wiz.cloud_configuration_finding_full_posture.rule.description |  | text |
| wiz.cloud_configuration_finding_full_posture.rule.id |  | keyword |
| wiz.cloud_configuration_finding_full_posture.rule.name |  | keyword |
| wiz.cloud_configuration_finding_full_posture.rule.remediation_instructions |  | text |
| wiz.cloud_configuration_finding_full_posture.rule.short_id |  | keyword |
| wiz.cloud_configuration_finding_full_posture.status |  | keyword |


##### cloud_configuration_finding_full_posture sample event

An example event for `cloud_configuration_finding_full_posture` looks as following:

```json
{
    "@timestamp": "2025-04-22T09:55:55.722365112Z",
    "agent": {
        "ephemeral_id": "5f4b4a3b-5fe7-41c7-ae81-1859e2eb9fcf",
        "id": "54fad7af-68b0-41e9-ba13-01893279295d",
        "name": "elastic-agent-30873",
        "type": "filebeat",
        "version": "8.18.0"
    },
    "cloud": {
        "account": {
            "id": "998231069301",
            "name": "wiz-integrations"
        },
        "provider": "aws",
        "service": {
            "name": "eks"
        }
    },
    "data_stream": {
        "dataset": "wiz.cloud_configuration_finding_full_posture",
        "namespace": "26487",
        "type": "logs"
    },
    "ecs": {
        "version": "8.11.0"
    },
    "elastic_agent": {
        "id": "54fad7af-68b0-41e9-ba13-01893279295d",
        "snapshot": true,
        "version": "8.18.0"
    },
    "event": {
        "agent_id_status": "verified",
        "category": [
            "configuration"
        ],
        "created": "2024-08-07T12:55:52.012Z",
        "dataset": "wiz.cloud_configuration_finding_full_posture",
        "id": "1243196d-a365-589a-a8aa-13817c9877b2",
        "ingested": "2025-04-22T09:55:55Z",
        "kind": "state",
        "original": "{\"analyzedAt\":\"2024-08-07T12:55:52.012378Z\",\"id\":\"1243196d-a365-589a-a8aa-13817c9877b2\",\"remediation\":null,\"resource\":{\"cloudPlatform\":\"EKS\",\"id\":\"f0f4163d-cbd7-517c-ba9e-f96bb90ab5ea\",\"name\":\"Root user\",\"nativeType\":\"rootUser\",\"providerId\":\"arn:aws:iam::998231069301:root\",\"region\":null,\"subscription\":{\"cloudProvider\":\"AWS\",\"externalId\":\"998231069301\",\"id\":\"94e76baa-85fd-5928-b829-1669a2ca9660\",\"name\":\"wiz-integrations\"},\"tags\":[],\"type\":\"USER_ACCOUNT\"},\"result\":\"PASS\",\"rule\":{\"description\":\"description\",\"id\":\"563ed717-4fb6-47fd-929e-9c794e201d0a\",\"name\":\"Root account access keys should not exist\",\"remediationInstructions\":\"instructions\",\"shortId\":\"IAM-006\"},\"severity\":\"MEDIUM\"}",
        "outcome": "success",
        "type": [
            "info"
        ]
    },
    "input": {
        "type": "cel"
    },
    "observer": {
        "vendor": "Wiz"
    },
    "resource": {
        "id": "arn:aws:iam::998231069301:root",
        "name": "Root user",
        "sub_type": "rootUser",
        "type": "USER_ACCOUNT"
    },
    "result": {
        "evaluation": "passed"
    },
    "rule": {
        "description": "description",
        "id": "IAM-006",
        "name": "Root account access keys should not exist",
        "remediation": "instructions",
        "uuid": "563ed717-4fb6-47fd-929e-9c794e201d0a"
    },
    "tags": [
        "preserve_original_event",
        "preserve_duplicate_custom_fields",
        "forwarded",
        "wiz-cloud_configuration_finding_full_posture"
    ],
    "user": {
        "id": "arn:aws:iam::998231069301:root",
        "name": "Root user"
    },
    "wiz": {
        "cloud_configuration_finding_full_posture": {
            "analyzed_at": "2024-08-07T12:55:52.012Z",
            "id": "1243196d-a365-589a-a8aa-13817c9877b2",
            "resource": {
                "cloud_platform": "EKS",
                "id": "f0f4163d-cbd7-517c-ba9e-f96bb90ab5ea",
                "name": "Root user",
                "native_type": "rootUser",
                "provider_id": "arn:aws:iam::998231069301:root",
                "subscription": {
                    "cloud_provider": "AWS",
                    "external_id": "998231069301",
                    "name": "wiz-integrations"
                },
                "type": "USER_ACCOUNT"
            },
            "result": "PASS",
            "rule": {
                "description": "description",
                "id": "563ed717-4fb6-47fd-929e-9c794e201d0a",
                "name": "Root account access keys should not exist",
                "remediation_instructions": "instructions",
                "short_id": "IAM-006"
            }
        }
    }
}
```

#### defend

The `defend` data stream provides events from Wiz of the following types: detection events from Wiz Defend.

##### defend fields

**Exported fields**

| Field | Description | Type |
|---|---|---|
| @timestamp | Event timestamp. | date |
| data_stream.dataset | Data stream dataset. | constant_keyword |
| data_stream.namespace | Data stream namespace. | constant_keyword |
| data_stream.type | Data stream type. | constant_keyword |
| event.dataset | Event dataset. | constant_keyword |
| event.module | Event module. | constant_keyword |
| input.type | Type of Filebeat input. | keyword |
| log.offset | Log offset. | long |
| wiz.defend.cloudOrganizations.cloudProvider |  | keyword |
| wiz.defend.cloudOrganizations.externalId |  | keyword |
| wiz.defend.cloudOrganizations.name |  | keyword |
| wiz.defend.created_at | ISO8601 timestamp for when detection was created. | date |
| wiz.defend.description | Description providing more details on the detection. | keyword |
| wiz.defend.detection_url | URL linking to more details on the detection. | keyword |
| wiz.defend.friendly_name |  | keyword |
| wiz.defend.id | Unique identifier for the detection. | keyword |
| wiz.defend.mitreTactics |  | keyword |
| wiz.defend.mitreTechniques |  | keyword |
| wiz.defend.primary_actor.email | Primary Actor Email. | keyword |
| wiz.defend.primary_actor.external_id | Primary Actor External ID. | keyword |
| wiz.defend.primary_actor.id | Primary Actor ID. | keyword |
| wiz.defend.primary_actor.name | Primary Actor Name. | keyword |
| wiz.defend.primary_actor.native_type | Primary Actor Native Type. | keyword |
| wiz.defend.primary_actor.type | Primary Actor Type. | keyword |
| wiz.defend.primary_resource.cloud_account.cloud_platform | Cloud Platform associated with Cloud Account for primary resource. | keyword |
| wiz.defend.primary_resource.cloud_account.external_id | External ID for cloud account for primary resource. | keyword |
| wiz.defend.primary_resource.cloud_account.id | ID for cloud account for primary resource. | keyword |
| wiz.defend.primary_resource.cloud_provider_url | URL to resource in cloud provider console for primary resource. | keyword |
| wiz.defend.primary_resource.external_id | External ID of primary resource. | keyword |
| wiz.defend.primary_resource.id | ID of primary resource. | keyword |
| wiz.defend.primary_resource.kubernetes_cluster_id | ID of the Kubernetes cluster for primary resource. | keyword |
| wiz.defend.primary_resource.kubernetes_cluster_name | Name of the Kubernetes cluster for primary resource. | keyword |
| wiz.defend.primary_resource.kubernetes_namespace_id | ID of the Kubernetes namespace for primary resource. | keyword |
| wiz.defend.primary_resource.kubernetes_namespace_name | Name of the Kubernetes namespace for primary resource. | keyword |
| wiz.defend.primary_resource.kubernetes_node_id | ID of the Kubernetes node for primary resource. | keyword |
| wiz.defend.primary_resource.kubernetes_node_name | Name of the Kubernetes node for primary resource. | keyword |
| wiz.defend.primary_resource.name | Name of the resource for primary resource. | keyword |
| wiz.defend.primary_resource.native_type | Native type classification for primary resource. | keyword |
| wiz.defend.primary_resource.provider_unique_id | Unique identifier from the provider for primary resource. | keyword |
| wiz.defend.primary_resource.region | Geographic region for primary resource. | keyword |
| wiz.defend.primary_resource.status | Current status of the resource for primary resource. | keyword |
| wiz.defend.primary_resource.type | Type of resource for primary resource. | keyword |
| wiz.defend.severity | Severity level of the detection. | keyword |
| wiz.defend.source | Source of the detection, will be "DETECTIONS". | keyword |
| wiz.defend.tdr_id | TDR identifier. | keyword |
| wiz.defend.tdr_source | TDR source. | keyword |
| wiz.defend.threat_id | ID of the associated threat. | keyword |
| wiz.defend.threat_url | URL linking to more details on the threat. | keyword |
| wiz.defend.timeframe.end | End timeframe for detection frametime. | date |
| wiz.defend.timeframe.start | Start timeframe for detection frametime. | date |
| wiz.defend.title | Title or summary of the detection. | keyword |
| wiz.defend.trigger.rule_id | Triggered Rule ID. | keyword |
| wiz.defend.trigger.rule_name | Triggered Rule Name. | keyword |
| wiz.defend.trigger.source | Triggered Source Name. | keyword |
| wiz.defend.trigger.type | Triggered Source Type. | keyword |
| wiz.defend.triggering_event.actor.acting_as.id | Actor ID. | keyword |
| wiz.defend.triggering_event.actor.acting_as.name | Name of the actor. | keyword |
| wiz.defend.triggering_event.actor.acting_as.native_type | Native type classification. | keyword |
| wiz.defend.triggering_event.actor.acting_as.type | Type of the actor. | keyword |
| wiz.defend.triggering_event.actor.external_id | External ID. | keyword |
| wiz.defend.triggering_event.actor.id | Actor ID. | keyword |
| wiz.defend.triggering_event.actor.name | Name of the actor. | keyword |
| wiz.defend.triggering_event.actor.native_type | Native type classification. | keyword |
| wiz.defend.triggering_event.actor.provider_unique_id | Unique identifier from the provider. | keyword |
| wiz.defend.triggering_event.actor.type | Type of the actor. | keyword |
| wiz.defend.triggering_event.actor_ip | IP address of the actor. | ip |
| wiz.defend.triggering_event.actor_ip_meta.autonomous_system_number | ASN number. | long |
| wiz.defend.triggering_event.actor_ip_meta.autonomous_system_organization | Organization associated with ASN (ASO). | keyword |
| wiz.defend.triggering_event.actor_ip_meta.country | Country of origin for IP. | keyword |
| wiz.defend.triggering_event.actor_ip_meta.is_foreign | Whether IP is from foreign source. | boolean |
| wiz.defend.triggering_event.actor_ip_meta.related_attack_group_names | Attack groups associated with IP. | keyword |
| wiz.defend.triggering_event.actor_ip_meta.reputation | IP reputation rating. | keyword |
| wiz.defend.triggering_event.actor_ip_meta.reputation_description | Description of IP reputation. | keyword |
| wiz.defend.triggering_event.actor_ip_meta.reputation_source | Source of reputation data. | keyword |
| wiz.defend.triggering_event.category | Event category. | keyword |
| wiz.defend.triggering_event.cloud_platform | Cloud platform where event occurred. | keyword |
| wiz.defend.triggering_event.cloud_provider_url | URL to event in cloud provider console. | keyword |
| wiz.defend.triggering_event.description | Description of the event. | keyword |
| wiz.defend.triggering_event.event_time | ISO8601 timestamp of when event occurred. | date |
| wiz.defend.triggering_event.external_id | Event External ID. | keyword |
| wiz.defend.triggering_event.id | Event ID. | keyword |
| wiz.defend.triggering_event.name | Name of the event. | keyword |
| wiz.defend.triggering_event.origin | Origin of the event. | keyword |
| wiz.defend.triggering_event.resources.cloud_account.cloud_platform | Cloud Platform associated with Cloud Account. | keyword |
| wiz.defend.triggering_event.resources.cloud_account.external_id | External ID for cloud account. | keyword |
| wiz.defend.triggering_event.resources.cloud_account.id | ID for cloud account. | keyword |
| wiz.defend.triggering_event.resources.cloud_provider_url | URL to resource in cloud provider console. | keyword |
| wiz.defend.triggering_event.resources.external_id | External ID. | keyword |
| wiz.defend.triggering_event.resources.id | Resource ID. | keyword |
| wiz.defend.triggering_event.resources.kubernetes_cluster_id | ID of the Kubernetes cluster. | keyword |
| wiz.defend.triggering_event.resources.kubernetes_cluster_name | Name of the Kubernetes cluster. | keyword |
| wiz.defend.triggering_event.resources.kubernetes_namespace_id | ID of the Kubernetes namespace. | keyword |
| wiz.defend.triggering_event.resources.kubernetes_namespace_name | Name of the Kubernetes namespace. | keyword |
| wiz.defend.triggering_event.resources.kubernetes_node_id | ID of the Kubernetes node. | keyword |
| wiz.defend.triggering_event.resources.kubernetes_node_name | Name of the Kubernetes node. | keyword |
| wiz.defend.triggering_event.resources.name | Name of the resource. | keyword |
| wiz.defend.triggering_event.resources.native_type | Native type classification. | keyword |
| wiz.defend.triggering_event.resources.provider_unique_id | Unique identifier from the provider. | keyword |
| wiz.defend.triggering_event.resources.region | Geographic region. | keyword |
| wiz.defend.triggering_event.resources.status | Current status of the resource. | keyword |
| wiz.defend.triggering_event.resources.type | Type of resource. | keyword |
| wiz.defend.triggering_event.runtime_details.process_tree.command | Process command line. | keyword |
| wiz.defend.triggering_event.runtime_details.process_tree.container.external_id | Container External ID. | keyword |
| wiz.defend.triggering_event.runtime_details.process_tree.container.id | Container ID. | keyword |
| wiz.defend.triggering_event.runtime_details.process_tree.container.image_external_id | Container Image External ID. | keyword |
| wiz.defend.triggering_event.runtime_details.process_tree.container.image_id | Container Image ID. | keyword |
| wiz.defend.triggering_event.runtime_details.process_tree.container.name | Container Name. | keyword |
| wiz.defend.triggering_event.runtime_details.process_tree.execution_time | ISO8601 timestamp when process executed. | date |
| wiz.defend.triggering_event.runtime_details.process_tree.hash | Executable SHA1 hash. | keyword |
| wiz.defend.triggering_event.runtime_details.process_tree.id | Process Tree ID. | keyword |
| wiz.defend.triggering_event.runtime_details.process_tree.path | Executable path. | keyword |
| wiz.defend.triggering_event.runtime_details.process_tree.size | Executable size in bytes. | long |
| wiz.defend.triggering_event.runtime_details.process_tree.user_id | User ID that executed process. | keyword |
| wiz.defend.triggering_event.runtime_details.process_tree.username | Username that executed process. | keyword |
| wiz.defend.triggering_event.source | Source of the event. | keyword |
| wiz.defend.triggering_event.status | Status of the event. | keyword |
| wiz.defend.triggering_event.subject_resource_id | ID of the primary affected resource. | keyword |
| wiz.defend.triggering_event.subject_resource_ip | IP of the primary affected resource. | ip |
| wiz.defend.triggering_events_count | Count of events that triggered detection. | long |


##### defend sample event

An example event for `defend` looks as following:

```json
{
    "@timestamp": "2025-01-21T18:52:15.838Z",
    "agent": {
        "ephemeral_id": "10c542b8-ed29-40a5-9d04-f32da0fef9bc",
        "id": "c4be22ec-fa52-4247-accb-8c8e1762c834",
        "name": "elastic-agent-50085",
        "type": "filebeat",
        "version": "8.18.0"
    },
    "cloud": {
        "provider": "AWS"
    },
    "data_stream": {
        "dataset": "wiz.defend",
        "namespace": "18676",
        "type": "logs"
    },
    "ecs": {
        "version": "8.17.0"
    },
    "elastic_agent": {
        "id": "c4be22ec-fa52-4247-accb-8c8e1762c834",
        "snapshot": true,
        "version": "8.18.0"
    },
    "event": {
        "action": "created",
        "agent_id_status": "verified",
        "category": [
            "threat"
        ],
        "dataset": "wiz.defend",
        "id": "2b46aa0d-9f46-5cb9-a6ae-e83ca514144a",
        "ingested": "2025-04-28T09:00:38Z",
        "kind": "event",
        "original": "{\"severity\":\"MEDIUM\",\"threatId\":\"733edfe5-db25-5b14-ac58-dc69d6005c81\",\"description\":\"Process executed the touch binary with the relevant command line flag used to modify files date information such as creation time, and last modification time. This could indicate the presence of a threat actor achieving defense evasion using the Timestomping technique.\",\"trigger\":{\"ruleName\":\"Detections Webhook Test Rule\",\"source\":\"DETECTIONS\",\"type\":\"Created\",\"ruleId\":\"a08fe977-3f54-48bf-adcf-f76994739c1f\"},\"tdrId\":\"46fd0cdc-252e-5e69-be6e-66e4851d7ae4\",\"title\":\"Timestomping technique was detected\",\"triggeringEventsCount\":2,\"tdrSource\":\"WIZ_SENSOR\",\"primaryResource\":{\"cloudAccount\":{\"cloudPlatform\":\"AWS\",\"externalId\":\"134653897021\",\"id\":\"5d67ed02-738e-5217-b065-d93642dd2629\"},\"nativeType\":\"ecs#containerinstance\",\"name\":\"test-container\",\"externalId\":\"test-container\",\"id\":\"da259b23-de77-5adb-8336-8c4071696305\",\"region\":\"us-east-1\",\"type\":\"CONTAINER\"},\"mitreTechniques\":[\"T1070.006\"],\"cloudAccounts\":[{\"cloudPlatform\":\"AWS\",\"externalId\":\"134653897021\",\"id\":\"5d67ed02-738e-5217-b065-d93642dd2629\"}],\"timeframe\":{\"start\":\"2025-01-21T18:52:15.838Z\",\"end\":\"2025-01-21T18:52:15.838Z\"},\"createdAt\":\"2025-01-21T18:52:16.819883668Z\",\"mitreTactics\":[\"TA0005\"],\"id\":\"6a440e9b-c8d8-5482-a0e9-da714359aecf\",\"threatURL\":\"https://test.wiz.io/issues#~(issue~'733edfe5-db25-5b14-ac58-dc69d6005c81)\",\"triggeringEvent\":{\"cloudPlatform\":\"AWS\",\"origin\":\"WIZ_SENSOR\",\"externalId\":\"Ptrace##test-container-SensorRuleEngine##sen-id-142-bd820642-34f2-4d3c-90b6-c384df0fd528\",\"description\":\"The program /usr/bin/bash executed the program /usr/bin/touch on container test-container\",\"resources\":[{\"cloudAccount\":{\"cloudPlatform\":\"AWS\",\"externalId\":\"134653897021\",\"id\":\"5d67ed02-738e-5217-b065-d93642dd2629\"},\"nativeType\":\"ecs#containerinstance\",\"name\":\"test-container\",\"externalId\":\"test-container\",\"id\":\"da259b23-de77-5adb-8336-8c4071696305\",\"region\":\"us-east-1\",\"type\":\"CONTAINER\"}],\"source\":\"WizSensorAlert##RuleEngine\",\"runtimeDetails\":{\"processTree\":[{\"container\":{\"imageId\":\"d18500ef-c0f7-5028-8c4c-1cd56c3a6652\",\"name\":\"test-container\",\"externalId\":\"test-container\",\"imageExternalId\":\"sha256:dcad76015854d8bcab3041a631d9d25d777325bb78abfa8ab0882e1b85ad84bb\",\"id\":\"da259b23-de77-5adb-8336-8c4071696305\"},\"executionTime\":\"2025-01-21T18:52:15.838Z\",\"path\":\"/usr/bin/touch\",\"size\":109616,\"id\":\"1560\",\"userId\":\"0\",\"hash\":\"a0d0c6248d07a8fa8e3b6a94e218ff9c8c372ad6\",\"command\":\"touch -r /usr/bin /tmp/uga\",\"username\":\"root\"},{\"container\":{\"imageId\":\"d18500ef-c0f7-5028-8c4c-1cd56c3a6652\",\"name\":\"test-container\",\"externalId\":\"test-container\",\"imageExternalId\":\"sha256:dcad76015854d8bcab3041a631d9d25d777325bb78abfa8ab0882e1b85ad84bb\",\"id\":\"da259b23-de77-5adb-8336-8c4071696305\"},\"executionTime\":\"2025-01-21T18:52:15.838Z\",\"path\":\"/usr/bin/bash\",\"size\":1265648,\"id\":\"1560\",\"userId\":\"0\",\"hash\":\"91fbd9d8c65de48dc82a1064b8a4fc89f5651778\",\"command\":\"/bin/bash -x -c touch -r /usr/bin /tmp/uga\",\"username\":\"root\"}]},\"cloudProviderUrl\":\"https://console.aws.amazon.com/cloudtrail/home?region=us-east-1#/events/Ptrace##test-container-SensorRuleEngine##sen-id-142-bd820642-34f2-4d3c-90b6-c384df0fd528\",\"actor\":{\"nativeType\":\"Microsoft Entra ID Application Service Principal\",\"name\":\"test-actor\",\"externalId\":\"test-actor\",\"id\":\"4e1bd57f-49b2-47a8-a4a7-0e66fe0b770e\",\"type\":\"SERVICE_ACCOUNT\"},\"actorIPMeta\":{\"reputationSource\":\"Recorded Future\",\"country\":\"United States\",\"isForeign\":true,\"reputation\":\"Benign\",\"autonomousSystemNumber\":8075,\"autonomousSystemOrganization\":\"MICROSOFT-CORP-MSN-AS-BLOCK\"},\"name\":\"Timestomping technique was detected\",\"eventTime\":\"2025-01-21T18:52:15.838Z\",\"id\":\"2b46aa0d-9f46-5cb9-a6ae-e83ca514144a\",\"category\":\"Detection\",\"status\":\"Success\",\"actorIP\":\"81.2.69.192\"},\"cloudOrganizations\":[]}",
        "outcome": "success",
        "provider": "WizSensorAlert##RuleEngine",
        "severity": 47,
        "type": [
            "indicator"
        ]
    },
    "input": {
        "type": "http_endpoint"
    },
    "message": "The program /usr/bin/bash executed the program /usr/bin/touch on container test-container",
    "observer": {
        "product": "Defend",
        "vendor": "Wiz"
    },
    "related": {
        "hash": [
            "a0d0c6248d07a8fa8e3b6a94e218ff9c8c372ad6",
            "91fbd9d8c65de48dc82a1064b8a4fc89f5651778"
        ],
        "ip": [
            "81.2.69.192"
        ],
        "user": [
            "4e1bd57f-49b2-47a8-a4a7-0e66fe0b770e",
            "test-actor",
            "0",
            "root"
        ]
    },
    "rule": {
        "id": "a08fe977-3f54-48bf-adcf-f76994739c1f",
        "name": "Detections Webhook Test Rule"
    },
    "source": {
        "as": {
            "number": 8075,
            "organization": {
                "name": "MICROSOFT-CORP-MSN-AS-BLOCK"
            }
        },
        "geo": {
            "city_name": "London",
            "continent_name": "Europe",
            "country_iso_code": "GB",
            "country_name": "United Kingdom",
            "location": {
                "lat": 51.5142,
                "lon": -0.0931
            },
            "region_iso_code": "GB-ENG",
            "region_name": "England"
        }
    },
    "tags": [
        "preserve_original_event",
        "preserve_duplicate_custom_fields",
        "forwarded",
        "wiz-defend"
    ],
    "threat": {
        "indicator": {
            "id": [
                "733edfe5-db25-5b14-ac58-dc69d6005c81"
            ],
            "reference": "https://test.wiz.io/issues#~(issue~'733edfe5-db25-5b14-ac58-dc69d6005c81)"
        },
        "tactic": {
            "id": [
                "TA0005"
            ]
        },
        "technique": {
            "id": [
                "T1070.006"
            ]
        }
    },
    "user": {
        "id": "4e1bd57f-49b2-47a8-a4a7-0e66fe0b770e",
        "name": "test-actor"
    },
    "wiz": {
        "defend": {
            "created_at": "2025-01-21T18:52:16.819Z",
            "description": "Process executed the touch binary with the relevant command line flag used to modify files date information such as creation time, and last modification time. This could indicate the presence of a threat actor achieving defense evasion using the Timestomping technique.",
            "friendly_name": "Detections Webhook Test Rule",
            "id": "6a440e9b-c8d8-5482-a0e9-da714359aecf",
            "mitreTactics": [
                "TA0005"
            ],
            "mitreTechniques": [
                "T1070.006"
            ],
            "primary_resource": {
                "cloud_account": {
                    "cloud_platform": "AWS",
                    "external_id": "134653897021",
                    "id": "5d67ed02-738e-5217-b065-d93642dd2629"
                },
                "external_id": "test-container",
                "id": "da259b23-de77-5adb-8336-8c4071696305",
                "name": "test-container",
                "native_type": "ecs#containerinstance",
                "region": "us-east-1",
                "type": "CONTAINER"
            },
            "severity": "MEDIUM",
            "tdr_id": "46fd0cdc-252e-5e69-be6e-66e4851d7ae4",
            "tdr_source": "WIZ_SENSOR",
            "threat_id": "733edfe5-db25-5b14-ac58-dc69d6005c81",
            "threat_url": "https://test.wiz.io/issues#~(issue~'733edfe5-db25-5b14-ac58-dc69d6005c81)",
            "timeframe": {
                "end": "2025-01-21T18:52:15.838Z",
                "start": "2025-01-21T18:52:15.838Z"
            },
            "title": "Timestomping technique was detected",
            "trigger": {
                "rule_id": "a08fe977-3f54-48bf-adcf-f76994739c1f",
                "rule_name": "Detections Webhook Test Rule",
                "source": "DETECTIONS",
                "type": "Created"
            },
            "triggering_event": {
                "actor": {
                    "external_id": "test-actor",
                    "id": "4e1bd57f-49b2-47a8-a4a7-0e66fe0b770e",
                    "name": "test-actor",
                    "native_type": "Microsoft Entra ID Application Service Principal",
                    "type": "SERVICE_ACCOUNT"
                },
                "actor_ip": "81.2.69.192",
                "actor_ip_meta": {
                    "autonomous_system_number": 8075,
                    "autonomous_system_organization": "MICROSOFT-CORP-MSN-AS-BLOCK",
                    "country": "United States",
                    "is_foreign": true,
                    "reputation": "Benign",
                    "reputation_source": "Recorded Future"
                },
                "category": "Detection",
                "cloud_platform": "AWS",
                "cloud_provider_url": "https://console.aws.amazon.com/cloudtrail/home?region=us-east-1#/events/Ptrace##test-container-SensorRuleEngine##sen-id-142-bd820642-34f2-4d3c-90b6-c384df0fd528",
                "description": "The program /usr/bin/bash executed the program /usr/bin/touch on container test-container",
                "event_time": "2025-01-21T18:52:15.838Z",
                "external_id": "Ptrace##test-container-SensorRuleEngine##sen-id-142-bd820642-34f2-4d3c-90b6-c384df0fd528",
                "id": "2b46aa0d-9f46-5cb9-a6ae-e83ca514144a",
                "name": "Timestomping technique was detected",
                "origin": "WIZ_SENSOR",
                "resources": [
                    {
                        "cloud_account": {
                            "cloud_platform": "AWS",
                            "external_id": "134653897021",
                            "id": "5d67ed02-738e-5217-b065-d93642dd2629"
                        },
                        "external_id": "test-container",
                        "id": "da259b23-de77-5adb-8336-8c4071696305",
                        "name": "test-container",
                        "native_type": "ecs#containerinstance",
                        "region": "us-east-1",
                        "type": "CONTAINER"
                    }
                ],
                "runtime_details": {
                    "process_tree": [
                        {
                            "command": "touch -r /usr/bin /tmp/uga",
                            "container": {
                                "external_id": "test-container",
                                "id": "da259b23-de77-5adb-8336-8c4071696305",
                                "image_external_id": "sha256:dcad76015854d8bcab3041a631d9d25d777325bb78abfa8ab0882e1b85ad84bb",
                                "image_id": "d18500ef-c0f7-5028-8c4c-1cd56c3a6652",
                                "name": "test-container"
                            },
                            "execution_time": "2025-01-21T18:52:15.838Z",
                            "hash": "a0d0c6248d07a8fa8e3b6a94e218ff9c8c372ad6",
                            "id": "1560",
                            "path": "/usr/bin/touch",
                            "size": 109616,
                            "user_id": "0",
                            "username": "root"
                        },
                        {
                            "command": "/bin/bash -x -c touch -r /usr/bin /tmp/uga",
                            "container": {
                                "external_id": "test-container",
                                "id": "da259b23-de77-5adb-8336-8c4071696305",
                                "image_external_id": "sha256:dcad76015854d8bcab3041a631d9d25d777325bb78abfa8ab0882e1b85ad84bb",
                                "image_id": "d18500ef-c0f7-5028-8c4c-1cd56c3a6652",
                                "name": "test-container"
                            },
                            "execution_time": "2025-01-21T18:52:15.838Z",
                            "hash": "91fbd9d8c65de48dc82a1064b8a4fc89f5651778",
                            "id": "1560",
                            "path": "/usr/bin/bash",
                            "size": 1265648,
                            "user_id": "0",
                            "username": "root"
                        }
                    ]
                },
                "source": "WizSensorAlert##RuleEngine",
                "status": "Success"
            },
            "triggering_events_count": 2
        }
    }
}
```

#### issue

The `issue` data stream provides events from Wiz of the following types: security issues.

##### issue fields

**Exported fields**

| Field | Description | Type |
|---|---|---|
| @timestamp | Date/time when the event originated. This is the date/time extracted from the event, typically representing when the event was generated by the source. If the event source has no original timestamp, this value is typically populated by the first time the event was received by the pipeline. Required field for all events. | date |
| data_stream.dataset | The field can contain anything that makes sense to signify the source of the data. Examples include `nginx.access`, `prometheus`, `endpoint` etc. For data streams that otherwise fit, but that do not have dataset set we use the value "generic" for the dataset value. `event.dataset` should have the same value as `data_stream.dataset`. Beyond the Elasticsearch data stream naming criteria noted above, the `dataset` value has additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |
| data_stream.namespace | A user defined namespace. Namespaces are useful to allow grouping of data. Many users already organize their indices this way, and the data stream naming scheme now provides this best practice as a default. Many users will populate this field with `default`. If no value is used, it falls back to `default`. Beyond the Elasticsearch index naming criteria noted above, `namespace` value has the additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |
| data_stream.type | An overarching type for the data stream. Currently allowed values are "logs" and "metrics". We expect to also add "traces" and "synthetics" in the near future. | constant_keyword |
| event.dataset | Name of the dataset. If an event source publishes more than one type of log or events (e.g. access log, error log), the dataset is used to specify which one the event comes from. It's recommended but not required to start the dataset name with the module name, followed by a dot, then the dataset name. | constant_keyword |
| event.module | Name of the module this data is coming from. If your monitoring agent supports the concept of modules or plugins to process events of a given source (e.g. Apache logs), `event.module` should contain the name of this module. | constant_keyword |
| input.type | Type of filebeat input. | keyword |
| log.offset | Log offset. | long |
| wiz.issue.created_at |  | date |
| wiz.issue.due_at |  | date |
| wiz.issue.entity_snapshot.cloud.platform |  | keyword |
| wiz.issue.entity_snapshot.cloud.provider_url |  | keyword |
| wiz.issue.entity_snapshot.external_id |  | keyword |
| wiz.issue.entity_snapshot.id |  | keyword |
| wiz.issue.entity_snapshot.name |  | keyword |
| wiz.issue.entity_snapshot.native_type |  | keyword |
| wiz.issue.entity_snapshot.provider_id |  | keyword |
| wiz.issue.entity_snapshot.region |  | keyword |
| wiz.issue.entity_snapshot.resource_group_external_id |  | keyword |
| wiz.issue.entity_snapshot.status |  | keyword |
| wiz.issue.entity_snapshot.subscription.external_id |  | keyword |
| wiz.issue.entity_snapshot.subscription.name |  | keyword |
| wiz.issue.entity_snapshot.subscription.tags |  | flattened |
| wiz.issue.entity_snapshot.tags |  | flattened |
| wiz.issue.entity_snapshot.type |  | keyword |
| wiz.issue.id |  | keyword |
| wiz.issue.notes.created_at |  | date |
| wiz.issue.notes.service_account.name |  | keyword |
| wiz.issue.notes.text |  | keyword |
| wiz.issue.notes.updated_at |  | date |
| wiz.issue.notes.user.email |  | keyword |
| wiz.issue.notes.user.name |  | keyword |
| wiz.issue.projects.business_unit |  | keyword |
| wiz.issue.projects.id |  | keyword |
| wiz.issue.projects.name |  | keyword |
| wiz.issue.projects.risk_profile.business_impact |  | keyword |
| wiz.issue.projects.slug |  | keyword |
| wiz.issue.resolved_at |  | date |
| wiz.issue.service_tickets.external_id |  | keyword |
| wiz.issue.service_tickets.name |  | keyword |
| wiz.issue.service_tickets.url |  | keyword |
| wiz.issue.severity |  | keyword |
| wiz.issue.source_rules.__typename |  | keyword |
| wiz.issue.source_rules.description |  | keyword |
| wiz.issue.source_rules.id |  | keyword |
| wiz.issue.source_rules.name |  | keyword |
| wiz.issue.source_rules.remediation_instructions |  | keyword |
| wiz.issue.source_rules.resolution_recommendation |  | keyword |
| wiz.issue.source_rules.risks |  | keyword |
| wiz.issue.source_rules.security_sub_categories.category.framework.name |  | keyword |
| wiz.issue.source_rules.security_sub_categories.category.name |  | keyword |
| wiz.issue.source_rules.security_sub_categories.title |  | keyword |
| wiz.issue.source_rules.service_type |  | keyword |
| wiz.issue.source_rules.source_type |  | keyword |
| wiz.issue.source_rules.type |  | keyword |
| wiz.issue.status.changed_at |  | date |
| wiz.issue.status.value |  | keyword |
| wiz.issue.type |  | keyword |
| wiz.issue.updated_at |  | date |


##### issue sample event

An example event for `issue` looks as following:

```json
{
    "@timestamp": "2023-07-21T06:26:08.708Z",
    "agent": {
        "ephemeral_id": "8ed8aec5-5335-4510-a6b6-e98441f37b7a",
        "id": "07d85902-c982-4c69-8827-318fa4d8050f",
        "name": "elastic-agent-67496",
        "type": "filebeat",
        "version": "8.16.6"
    },
    "cloud": {
        "provider": "Kubernetes",
        "region": "us-01"
    },
    "data_stream": {
        "dataset": "wiz.issue",
        "namespace": "49382",
        "type": "logs"
    },
    "ecs": {
        "version": "8.11.0"
    },
    "elastic_agent": {
        "id": "07d85902-c982-4c69-8827-318fa4d8050f",
        "snapshot": false,
        "version": "8.16.6"
    },
    "event": {
        "agent_id_status": "verified",
        "category": [
            "configuration"
        ],
        "created": "2023-08-21T07:56:09.903Z",
        "dataset": "wiz.issue",
        "id": "ggf9cggd-64a7-412c-9445-cf837f4b0b10",
        "ingested": "2026-01-07T08:55:20Z",
        "kind": "event",
        "original": "{\"createdAt\":\"2023-08-21T07:56:09.903743Z\",\"dueAt\":\"2023-08-28T21:00:00Z\",\"entitySnapshot\":{\"cloudPlatform\":\"Kubernetes\",\"cloudProviderURL\":\"https://portal.az.com/#@sectest.on.com/resource//subscriptions/\",\"externalId\":\"k8s/clusterrole/aaa8e7ca2bf9bc85a75d5bbdd8ffd08d69f8852782a6341c3c3519sad45/system:aggregate-to-edit/12\",\"id\":\"f307d472-b7da-5t05-9b25-71a271336b14\",\"name\":\"system:aggregate-to-edit\",\"nativeType\":\"ClusterRole\",\"providerId\":\"k8s/clusterrole/aaa8e7ca2bf9bc85a75d5bbdd8ffd08d69f8852782a6341c3c3519bac0f24ae9/system:aggregate-to-edit/12\",\"region\":\"us-01\",\"resourceGroupExternalId\":\"/subscriptions/cfd132be-3bc7-4f86-8efd-ed53ae498fec/resourcegroups/test-selfmanaged-eastus\",\"status\":\"Active\",\"subscriptionExternalId\":\"998231069301\",\"subscriptionName\":\"demo-integrations\",\"subscriptionTags\":{},\"tags\":{\"kubernetes.io/bootstrapping\":\"rbac-defaults\",\"rbac.authorization.k8s.io/aggregate-to-edit\":\"true\"},\"type\":\"ACCESS_ROLE\"},\"id\":\"ggf9cggd-64a7-412c-9445-cf837f4b0b10\",\"notes\":[{\"createdAt\":\"2023-08-21T07:56:09.903743Z\",\"serviceAccount\":{\"name\":\"rev-ke\"},\"text\":\"updated\",\"updatedAt\":\"2023-09-09T23:10:22.588721Z\"},{\"createdAt\":\"2023-08-07T23:08:49.918941Z\",\"serviceAccount\":{\"name\":\"rev-ke2\"},\"text\":\"updated\",\"updatedAt\":\"2023-08-09T23:10:22.591487Z\"}],\"projects\":[{\"businessUnit\":\"\",\"id\":\"jf77n35n-a7b6-5762-8a53-8e8f59e68bd8\",\"name\":\"Project 2\",\"riskProfile\":{\"businessImpact\":\"MBI\"},\"slug\":\"project-2\"},{\"businessUnit\":\"Dev\",\"id\":\"af52828c-4eb1-5c4e-847c-ebc3a5ead531\",\"name\":\"project 4\",\"riskProfile\":{\"businessImpact\":\"MBI\"},\"slug\":\"project-4\"},{\"businessUnit\":\"Dev\",\"id\":\"d5h1545-aec0-52fc-80ab-bacd7b02f178\",\"name\":\"Project1\",\"riskProfile\":{\"businessImpact\":\"MBI\"},\"slug\":\"project1\"}],\"resolvedAt\":\"2023-08-09T23:10:22.588721Z\",\"serviceTickets\":[{\"externalId\":\"638361121bbfdd10f6c1cbf3604bcb7e\",\"name\":\"SIR0010002\",\"url\":\"https://ven05658.testing.com/nav_to.do?uri=%2Fsn_si_incident.do%3Fsys_id%3D6385248sdsae421\"}],\"severity\":\"INFORMATIONAL\",\"sourceRules\":[{\"__typename\":\"Control\",\"description\":\"These EKS principals assume roles that provide bind, escalate and impersonate permissions. \\n\\nThe `bind` permission allows users to create bindings to roles with rights they do not already have. The `escalate` permission allows users effectively escalate their privileges. The `impersonate` permission allows users to impersonate and gain the rights of other users in the cluster. Running containers with these permissions has the potential to effectively allow privilege escalation to the cluster-admin level.\",\"id\":\"wc-id-1335\",\"name\":\"EKS principals assume roles that provide bind, escalate and impersonate permissions\",\"resolutionRecommendation\":\"To follow the principle of least privilege and minimize the risk of unauthorized access and data breaches, it is recommended not to grant `bind`, `escalate` or `impersonate` permissions.\",\"risks\":[\"INSECURE_KUBERNETES_CLUSTER\",\"VULNERABILITY\"],\"securitySubCategories\":[{\"category\":{\"framework\":{\"name\":\"CIS EKS 1.2.0\"},\"name\":\"4.1 RBAC and Service Accounts\"},\"title\":\"4.1.8 Limit use of the Bind, Impersonate and Escalate permissions in the Kubernetes cluster - Level 1 (Manual)\"},{\"category\":{\"framework\":{\"name\":\"Wiz for Risk Assessment\"},\"name\":\"Identity Management\"},\"title\":\"Privileged principal\"},{\"category\":{\"framework\":{\"name\":\"Wiz\"},\"name\":\"9 Container Security\"},\"title\":\"Container Security\"},{\"category\":{\"framework\":{\"name\":\"Wiz for Risk Assessment\"},\"name\":\"Container \\u0026 Kubernetes Security\"},\"title\":\"Cluster misconfiguration\"}]},{\"__typename\":\"CloudEventRule\",\"description\":\"Process wrote to a security configuration file. This could indicate the presence of a threat actor tampering with security controls.\",\"id\":\"cer-sen-id-002\",\"name\":\"Security configuration file was modified\",\"risks\":[\"INSECURE_KUBERNETES_CLUSTER\",\"VULNERABILITY\"],\"securitySubCategories\":[{\"category\":{\"framework\":{\"name\":\"Wiz for Threat Detection\"},\"name\":\"Defense Evasion\"},\"title\":\"Security tool tampering\"}],\"sourceType\":\"WIZ_SENSOR\",\"type\":\"FILE_INTEGRITY_MONITORING_WORKLOAD_RUNTIME_RULE\"},{\"__typename\":\"CloudEventRule\",\"description\":\"Python process spawned interactive shell. This can indicate the presence of a malicious actor enhancing a basic reverse shell.\",\"id\":\"cer-sen-id-003\",\"name\":\"Python process spawned interactive shell\",\"risks\":[\"INSECURE_KUBERNETES_CLUSTER\",\"VULNERABILITY\"],\"securitySubCategories\":[{\"category\":{\"framework\":{\"name\":\"Wiz for Threat Detection\"},\"name\":\"C2 \\u0026 Exfiltration\"},\"title\":\"Remote shell\"}],\"sourceType\":\"WIZ_SENSOR\",\"type\":\"WORKLOAD_RUNTIME_RULE\"}],\"status\":\"IN_PROGRESS\",\"statusChangedAt\":\"2023-07-21T06:26:08.708199Z\",\"updatedAt\":\"2023-08-14T06:06:18.331647Z\"}",
        "type": [
            "info"
        ],
        "url": "https://app.wiz.io/issues#~(filters~(status~())~issue~'ggf9cggd-64a7-412c-9445-cf837f4b0b10)"
    },
    "input": {
        "type": "cel"
    },
    "message": "These EKS principals assume roles that provide bind, escalate and impersonate permissions. \n\nThe `bind` permission allows users to create bindings to roles with rights they do not already have. The `escalate` permission allows users effectively escalate their privileges. The `impersonate` permission allows users to impersonate and gain the rights of other users in the cluster. Running containers with these permissions has the potential to effectively allow privilege escalation to the cluster-admin level.",
    "tags": [
        "preserve_original_event",
        "preserve_duplicate_custom_fields",
        "forwarded",
        "wiz-issue"
    ],
    "url": {
        "domain": "portal.az.com",
        "fragment": "@sectest.on.com/resource//subscriptions/",
        "original": "https://portal.az.com/#@sectest.on.com/resource//subscriptions/",
        "path": "/",
        "scheme": "https"
    },
    "wiz": {
        "issue": {
            "created_at": "2023-08-21T07:56:09.903Z",
            "due_at": "2023-08-28T21:00:00.000Z",
            "entity_snapshot": {
                "cloud": {
                    "platform": "Kubernetes",
                    "provider_url": "https://portal.az.com/#@sectest.on.com/resource//subscriptions/"
                },
                "external_id": "k8s/clusterrole/aaa8e7ca2bf9bc85a75d5bbdd8ffd08d69f8852782a6341c3c3519sad45/system:aggregate-to-edit/12",
                "id": "f307d472-b7da-5t05-9b25-71a271336b14",
                "name": "system:aggregate-to-edit",
                "native_type": "ClusterRole",
                "provider_id": "k8s/clusterrole/aaa8e7ca2bf9bc85a75d5bbdd8ffd08d69f8852782a6341c3c3519bac0f24ae9/system:aggregate-to-edit/12",
                "region": "us-01",
                "resource_group_external_id": "/subscriptions/cfd132be-3bc7-4f86-8efd-ed53ae498fec/resourcegroups/test-selfmanaged-eastus",
                "status": "Active",
                "subscription": {
                    "external_id": "998231069301",
                    "name": "demo-integrations"
                },
                "tags": {
                    "kubernetes.io/bootstrapping": "rbac-defaults",
                    "rbac.authorization.k8s.io/aggregate-to-edit": "true"
                },
                "type": "ACCESS_ROLE"
            },
            "id": "ggf9cggd-64a7-412c-9445-cf837f4b0b10",
            "notes": [
                {
                    "created_at": "2023-08-21T07:56:09.903Z",
                    "service_account": {
                        "name": "rev-ke"
                    },
                    "text": "updated",
                    "updated_at": "2023-09-09T23:10:22.588Z"
                },
                {
                    "created_at": "2023-08-07T23:08:49.918Z",
                    "service_account": {
                        "name": "rev-ke2"
                    },
                    "text": "updated",
                    "updated_at": "2023-08-09T23:10:22.591Z"
                }
            ],
            "projects": [
                {
                    "id": "jf77n35n-a7b6-5762-8a53-8e8f59e68bd8",
                    "name": "Project 2",
                    "risk_profile": {
                        "business_impact": "MBI"
                    },
                    "slug": "project-2"
                },
                {
                    "business_unit": "Dev",
                    "id": "af52828c-4eb1-5c4e-847c-ebc3a5ead531",
                    "name": "project 4",
                    "risk_profile": {
                        "business_impact": "MBI"
                    },
                    "slug": "project-4"
                },
                {
                    "business_unit": "Dev",
                    "id": "d5h1545-aec0-52fc-80ab-bacd7b02f178",
                    "name": "Project1",
                    "risk_profile": {
                        "business_impact": "MBI"
                    },
                    "slug": "project1"
                }
            ],
            "resolved_at": "2023-08-09T23:10:22.588Z",
            "service_tickets": [
                {
                    "external_id": "638361121bbfdd10f6c1cbf3604bcb7e",
                    "name": "SIR0010002",
                    "url": "https://ven05658.testing.com/nav_to.do?uri=%2Fsn_si_incident.do%3Fsys_id%3D6385248sdsae421"
                }
            ],
            "severity": "INFORMATIONAL",
            "source_rules": [
                {
                    "__typename": "Control",
                    "description": "These EKS principals assume roles that provide bind, escalate and impersonate permissions. \n\nThe `bind` permission allows users to create bindings to roles with rights they do not already have. The `escalate` permission allows users effectively escalate their privileges. The `impersonate` permission allows users to impersonate and gain the rights of other users in the cluster. Running containers with these permissions has the potential to effectively allow privilege escalation to the cluster-admin level.",
                    "id": "wc-id-1335",
                    "name": "EKS principals assume roles that provide bind, escalate and impersonate permissions",
                    "resolution_recommendation": "To follow the principle of least privilege and minimize the risk of unauthorized access and data breaches, it is recommended not to grant `bind`, `escalate` or `impersonate` permissions.",
                    "risks": [
                        "INSECURE_KUBERNETES_CLUSTER",
                        "VULNERABILITY"
                    ],
                    "security_sub_categories": [
                        {
                            "category": {
                                "framework": {
                                    "name": "CIS EKS 1.2.0"
                                },
                                "name": "4.1 RBAC and Service Accounts"
                            },
                            "title": "4.1.8 Limit use of the Bind, Impersonate and Escalate permissions in the Kubernetes cluster - Level 1 (Manual)"
                        },
                        {
                            "category": {
                                "framework": {
                                    "name": "Wiz for Risk Assessment"
                                },
                                "name": "Identity Management"
                            },
                            "title": "Privileged principal"
                        },
                        {
                            "category": {
                                "framework": {
                                    "name": "Wiz"
                                },
                                "name": "9 Container Security"
                            },
                            "title": "Container Security"
                        },
                        {
                            "category": {
                                "framework": {
                                    "name": "Wiz for Risk Assessment"
                                },
                                "name": "Container & Kubernetes Security"
                            },
                            "title": "Cluster misconfiguration"
                        }
                    ]
                },
                {
                    "__typename": "CloudEventRule",
                    "description": "Process wrote to a security configuration file. This could indicate the presence of a threat actor tampering with security controls.",
                    "id": "cer-sen-id-002",
                    "name": "Security configuration file was modified",
                    "risks": [
                        "INSECURE_KUBERNETES_CLUSTER",
                        "VULNERABILITY"
                    ],
                    "security_sub_categories": [
                        {
                            "category": {
                                "framework": {
                                    "name": "Wiz for Threat Detection"
                                },
                                "name": "Defense Evasion"
                            },
                            "title": "Security tool tampering"
                        }
                    ],
                    "source_type": "WIZ_SENSOR",
                    "type": "FILE_INTEGRITY_MONITORING_WORKLOAD_RUNTIME_RULE"
                },
                {
                    "__typename": "CloudEventRule",
                    "description": "Python process spawned interactive shell. This can indicate the presence of a malicious actor enhancing a basic reverse shell.",
                    "id": "cer-sen-id-003",
                    "name": "Python process spawned interactive shell",
                    "risks": [
                        "INSECURE_KUBERNETES_CLUSTER",
                        "VULNERABILITY"
                    ],
                    "security_sub_categories": [
                        {
                            "category": {
                                "framework": {
                                    "name": "Wiz for Threat Detection"
                                },
                                "name": "C2 & Exfiltration"
                            },
                            "title": "Remote shell"
                        }
                    ],
                    "source_type": "WIZ_SENSOR",
                    "type": "WORKLOAD_RUNTIME_RULE"
                }
            ],
            "status": {
                "changed_at": "2023-07-21T06:26:08.708Z",
                "value": "IN_PROGRESS"
            },
            "updated_at": "2023-08-14T06:06:18.331Z"
        }
    }
}
```

#### vulnerability

The `vulnerability` data stream provides events from Wiz of the following types: software vulnerabilities.

##### vulnerability fields

**Exported fields**

| Field | Description | Type |
|---|---|---|
| @timestamp | Date/time when the event originated. This is the date/time extracted from the event, typically representing when the event was generated by the source. If the event source has no original timestamp, this value is typically populated by the first time the event was received by the pipeline. Required field for all events. | date |
| data_stream.dataset | The field can contain anything that makes sense to signify the source of the data. Examples include `nginx.access`, `prometheus`, `endpoint` etc. For data streams that otherwise fit, but that do not have dataset set we use the value "generic" for the dataset value. `event.dataset` should have the same value as `data_stream.dataset`. Beyond the Elasticsearch data stream naming criteria noted above, the `dataset` value has additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |
| data_stream.namespace | A user defined namespace. Namespaces are useful to allow grouping of data. Many users already organize their indices this way, and the data stream naming scheme now provides this best practice as a default. Many users will populate this field with `default`. If no value is used, it falls back to `default`. Beyond the Elasticsearch index naming criteria noted above, `namespace` value has the additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |
| data_stream.type | An overarching type for the data stream. Currently allowed values are "logs" and "metrics". We expect to also add "traces" and "synthetics" in the near future. | constant_keyword |
| event.dataset | Name of the dataset. If an event source publishes more than one type of log or events (e.g. access log, error log), the dataset is used to specify which one the event comes from. It's recommended but not required to start the dataset name with the module name, followed by a dot, then the dataset name. | constant_keyword |
| event.module | Name of the module this data is coming from. If your monitoring agent supports the concept of modules or plugins to process events of a given source (e.g. Apache logs), `event.module` should contain the name of this module. | constant_keyword |
| input.type | Type of filebeat input. | keyword |
| log.offset | Log offset. | long |
| package.fixed_version |  | keyword |
| resource.id |  | keyword |
| resource.name |  | keyword |
| vulnerability.cwe |  | keyword |
| vulnerability.package.fixed_version |  | keyword |
| vulnerability.package.name |  | keyword |
| vulnerability.package.version |  | keyword |
| vulnerability.title |  | keyword |
| wiz.vulnerability.cve_description |  | keyword |
| wiz.vulnerability.cvss_severity |  | keyword |
| wiz.vulnerability.data_source_name |  | keyword |
| wiz.vulnerability.description |  | keyword |
| wiz.vulnerability.detailed_name |  | keyword |
| wiz.vulnerability.detection_method |  | keyword |
| wiz.vulnerability.epss.percentile |  | double |
| wiz.vulnerability.epss.probability |  | double |
| wiz.vulnerability.epss.severity |  | keyword |
| wiz.vulnerability.exploitability_score |  | double |
| wiz.vulnerability.first_detected_at |  | date |
| wiz.vulnerability.fixed_version |  | keyword |
| wiz.vulnerability.has_cisa_kev_exploit |  | boolean |
| wiz.vulnerability.has_exploit |  | boolean |
| wiz.vulnerability.id |  | keyword |
| wiz.vulnerability.ignore_rules.enabled |  | boolean |
| wiz.vulnerability.ignore_rules.expired_at |  | date |
| wiz.vulnerability.ignore_rules.id |  | keyword |
| wiz.vulnerability.ignore_rules.name |  | keyword |
| wiz.vulnerability.impact_score |  | double |
| wiz.vulnerability.last_detected_at |  | date |
| wiz.vulnerability.layer_metadata.details |  | keyword |
| wiz.vulnerability.layer_metadata.id |  | keyword |
| wiz.vulnerability.layer_metadata.is_base_layer |  | boolean |
| wiz.vulnerability.link |  | keyword |
| wiz.vulnerability.location_path |  | keyword |
| wiz.vulnerability.name |  | keyword |
| wiz.vulnerability.portal_url |  | keyword |
| wiz.vulnerability.projects.business_unit |  | keyword |
| wiz.vulnerability.projects.id |  | keyword |
| wiz.vulnerability.projects.name |  | keyword |
| wiz.vulnerability.projects.risk_profile.business_impact |  | keyword |
| wiz.vulnerability.projects.slug |  | keyword |
| wiz.vulnerability.remedation |  | keyword |
| wiz.vulnerability.resolution_reason |  | keyword |
| wiz.vulnerability.resolved_at |  | date |
| wiz.vulnerability.score |  | double |
| wiz.vulnerability.status |  | keyword |
| wiz.vulnerability.validated_in_runtime |  | boolean |
| wiz.vulnerability.vendor_severity |  | keyword |
| wiz.vulnerability.version |  | keyword |
| wiz.vulnerability.vulnerable_asset.cloud.platform |  | keyword |
| wiz.vulnerability.vulnerable_asset.cloud.provider_url |  | keyword |
| wiz.vulnerability.vulnerable_asset.has_limited_internet_exposure |  | boolean |
| wiz.vulnerability.vulnerable_asset.has_wide_internet_exposure |  | boolean |
| wiz.vulnerability.vulnerable_asset.id |  | keyword |
| wiz.vulnerability.vulnerable_asset.ip_addresses |  | ip |
| wiz.vulnerability.vulnerable_asset.is_accessible_from.other_subscriptions |  | boolean |
| wiz.vulnerability.vulnerable_asset.is_accessible_from.other_vnets |  | boolean |
| wiz.vulnerability.vulnerable_asset.is_accessible_from.vpn |  | boolean |
| wiz.vulnerability.vulnerable_asset.name |  | keyword |
| wiz.vulnerability.vulnerable_asset.operating_system |  | keyword |
| wiz.vulnerability.vulnerable_asset.provider_unique_id |  | keyword |
| wiz.vulnerability.vulnerable_asset.region |  | keyword |
| wiz.vulnerability.vulnerable_asset.status |  | keyword |
| wiz.vulnerability.vulnerable_asset.subscription.external_id |  | keyword |
| wiz.vulnerability.vulnerable_asset.subscription.id |  | keyword |
| wiz.vulnerability.vulnerable_asset.subscription.name |  | keyword |
| wiz.vulnerability.vulnerable_asset.tags.name |  | keyword |
| wiz.vulnerability.vulnerable_asset.type |  | keyword |


##### vulnerability sample event

An example event for `vulnerability` looks as following:

```json
{
    "@timestamp": "2023-08-16T18:40:57.000Z",
    "agent": {
        "ephemeral_id": "4c555afd-d62f-4893-8145-235a7a2aa42e",
        "id": "c3610579-6628-4346-bac5-22eb264323cb",
        "name": "elastic-agent-39585",
        "type": "filebeat",
        "version": "8.18.0"
    },
    "cloud": {
        "account": {
            "name": "wiz-integrations"
        },
        "provider": "AWS",
        "region": "us-east-1"
    },
    "data_stream": {
        "dataset": "wiz.vulnerability",
        "namespace": "50935",
        "type": "logs"
    },
    "device": {
        "id": "c828de0d-4c42-5b1c-946b-2edee094d0b3"
    },
    "ecs": {
        "version": "8.11.0"
    },
    "elastic_agent": {
        "id": "c3610579-6628-4346-bac5-22eb264323cb",
        "snapshot": true,
        "version": "8.18.0"
    },
    "event": {
        "agent_id_status": "verified",
        "category": [
            "vulnerability"
        ],
        "dataset": "wiz.vulnerability",
        "id": "5e95ff50-5490-514e-87f7-11e56f3230ff",
        "ingested": "2025-04-22T10:01:05Z",
        "kind": "alert",
        "original": "{\"CVEDescription\":\"In LibTIFF, there is a memory malloc failure in tif_pixarlog.c. A crafted TIFF document can lead to an abort, resulting in a remote denial of service attack.\",\"CVSSSeverity\":\"MEDIUM\",\"dataSourceName\":\"data Source\",\"description\":\"Thepackage`libtiff`version`4.0.3-35.amzn2`wasdetectedin`YUMpackagemanager`onamachinerunning`Amazon2(Karoo)`isvulnerableto`CVE-2020-35522`,whichexistsinversions`\\u003c4.0.3-35.amzn2.0.1`.\\n\\nThevulnerabilitywasfoundinthe[OfficialAmazonLinuxSecurityAdvisories](https://alas.aws.amazon.com/AL2/ALAS-2022-1780.html)withvendorseverity:`Medium`([NVD](https://nvd.nist.gov/vuln/detail/CVE-2020-35522)severity:`Medium`).\\n\\nThevulnerabilitycanberemediatedbyupdatingthepackagetoversion`4.0.3-35.amzn2.0.1`orhigher,using`yumupdatelibtiff`.\",\"detailedName\":\"libtiff\",\"detectionMethod\":\"PACKAGE\",\"epssPercentile\":46.2,\"epssProbability\":0.1,\"epssSeverity\":\"LOW\",\"exploitabilityScore\":1.8,\"firstDetectedAt\":\"2022-05-01T11:36:10.063767Z\",\"fixedVersion\":\"4.0.3-35.amzn2.0.1\",\"hasCisaKevExploit\":false,\"hasExploit\":false,\"id\":\"5e95ff50-5490-514e-87f7-11e56f3230ff\",\"ignoreRules\":{\"enabled\":true,\"expiredAt\":\"2023-08-16T18:40:57Z\",\"id\":\"aj3jqtvnaf\",\"name\":\"abc\"},\"impactScore\":3.6,\"lastDetectedAt\":\"2023-08-16T18:40:57Z\",\"layerMetadata\":{\"details\":\"xxxx\",\"id\":\"5e95ff50-5490-514e-87f7-11e56f3230ff\",\"isBaseLayer\":true},\"link\":\"https://alas.aws.amazon.com/AL2/ALAS-2022-1780.html\",\"locationPath\":\"package/library/file\",\"name\":\"CVE-2020-3333\",\"portalUrl\":\"https://app.wiz.io/explorer/vulnerability-findings#~(entity~(~'xxx-xxx*2cSECURITY_TOOL_FINDING))\",\"projects\":[{\"businessUnit\":\"\",\"id\":\"83b76efe-a7b6-5762-8a53-8e8f59e68bd8\",\"name\":\"Project2\",\"riskProfile\":{\"businessImpact\":\"MBI\"},\"slug\":\"project-2\"},{\"businessUnit\":\"Dev\",\"id\":\"af52828c-4eb1-5c4e-847c-ebc3a5ead531\",\"name\":\"project4\",\"riskProfile\":{\"businessImpact\":\"MBI\"},\"slug\":\"project-4\"},{\"businessUnit\":\"Dev\",\"id\":\"d6ac50bb-aec0-52fc-80ab-bacd7b02f178\",\"name\":\"Project1\",\"riskProfile\":{\"businessImpact\":\"MBI\"},\"slug\":\"project1\"}],\"remediation\":\"yumupdatelibtiff\",\"resolutionReason\":\"resolutionReason\",\"resolvedAt\":\"2023-08-16T18:40:57Z\",\"score\":5.5,\"status\":\"OPEN\",\"validatedInRuntime\":true,\"vendorSeverity\":\"MEDIUM\",\"version\":\"4.0.3-35.amzn2\",\"vulnerableAsset\":{\"cloudPlatform\":\"AWS\",\"cloudProviderURL\":\"https://us-east-1.console.aws.amazon.com/ec2/v2/home?region=us-east-1#InstanceDetails:instanceId=i-0a0f7e1451da5f4a3\",\"hasLimitedInternetExposure\":true,\"hasWideInternetExposure\":true,\"id\":\"c828de0d-4c42-5b1c-946b-2edee094d0b3\",\"ipAddresses\":[\"89.160.20.112\",\"89.160.20.128\"],\"isAccessibleFromOtherSubscriptions\":false,\"isAccessibleFromOtherVnets\":false,\"isAccessibleFromVPN\":false,\"name\":\"test-4\",\"operatingSystem\":\"Linux\",\"providerUniqueId\":\"arn:aws:ec2:us-east-1:998231069301:instance/i-0a0f7e1451da5f4a3\",\"region\":\"us-east-1\",\"status\":\"Active\",\"subscriptionExternalId\":\"998231069301\",\"subscriptionId\":\"94e76baa-85fd-5928-b829-1669a2ca9660\",\"subscriptionName\":\"wiz-integrations\",\"tags\":{\"Name\":\"test-4\"},\"type\":\"VIRTUAL_MACHINE\"}}",
        "type": [
            "info"
        ]
    },
    "host": {
        "name": "test-4",
        "os": {
            "family": "Linux"
        }
    },
    "input": {
        "type": "cel"
    },
    "message": "Thepackage`libtiff`version`4.0.3-35.amzn2`wasdetectedin`YUMpackagemanager`onamachinerunning`Amazon2(Karoo)`isvulnerableto`CVE-2020-35522`,whichexistsinversions`<4.0.3-35.amzn2.0.1`.\n\nThevulnerabilitywasfoundinthe[OfficialAmazonLinuxSecurityAdvisories](https://alas.aws.amazon.com/AL2/ALAS-2022-1780.html)withvendorseverity:`Medium`([NVD](https://nvd.nist.gov/vuln/detail/CVE-2020-35522)severity:`Medium`).\n\nThevulnerabilitycanberemediatedbyupdatingthepackagetoversion`4.0.3-35.amzn2.0.1`orhigher,using`yumupdatelibtiff`.",
    "observer": {
        "vendor": "Wiz"
    },
    "package": {
        "fixed_version": "4.0.3-35.amzn2.0.1",
        "name": "libtiff",
        "version": "4.0.3-35.amzn2"
    },
    "related": {
        "ip": [
            "89.160.20.112",
            "89.160.20.128"
        ]
    },
    "resource": {
        "id": "arn:aws:ec2:us-east-1:998231069301:instance/i-0a0f7e1451da5f4a3",
        "name": "test-4"
    },
    "tags": [
        "preserve_original_event",
        "preserve_duplicate_custom_fields",
        "forwarded",
        "wiz-vulnerability"
    ],
    "vulnerability": {
        "cwe": "CVE-2020-3333",
        "description": "In LibTIFF, there is a memory malloc failure in tif_pixarlog.c. A crafted TIFF document can lead to an abort, resulting in a remote denial of service attack.",
        "id": "CVE-2020-3333",
        "package": {
            "fixed_version": "4.0.3-35.amzn2.0.1",
            "name": "libtiff",
            "version": "4.0.3-35.amzn2"
        },
        "reference": "https://alas.aws.amazon.com/AL2/ALAS-2022-1780.html",
        "score": {
            "base": 5.5
        },
        "severity": "MEDIUM",
        "title": "Vulnerability found - CVE-2020-3333"
    },
    "wiz": {
        "vulnerability": {
            "cve_description": "In LibTIFF, there is a memory malloc failure in tif_pixarlog.c. A crafted TIFF document can lead to an abort, resulting in a remote denial of service attack.",
            "cvss_severity": "MEDIUM",
            "data_source_name": "data Source",
            "description": "Thepackage`libtiff`version`4.0.3-35.amzn2`wasdetectedin`YUMpackagemanager`onamachinerunning`Amazon2(Karoo)`isvulnerableto`CVE-2020-35522`,whichexistsinversions`<4.0.3-35.amzn2.0.1`.\n\nThevulnerabilitywasfoundinthe[OfficialAmazonLinuxSecurityAdvisories](https://alas.aws.amazon.com/AL2/ALAS-2022-1780.html)withvendorseverity:`Medium`([NVD](https://nvd.nist.gov/vuln/detail/CVE-2020-35522)severity:`Medium`).\n\nThevulnerabilitycanberemediatedbyupdatingthepackagetoversion`4.0.3-35.amzn2.0.1`orhigher,using`yumupdatelibtiff`.",
            "detailed_name": "libtiff",
            "detection_method": "PACKAGE",
            "epss": {
                "percentile": 46.2,
                "probability": 0.1,
                "severity": "LOW"
            },
            "exploitability_score": 1.8,
            "first_detected_at": "2022-05-01T11:36:10.063Z",
            "fixed_version": "4.0.3-35.amzn2.0.1",
            "has_cisa_kev_exploit": false,
            "has_exploit": false,
            "id": "5e95ff50-5490-514e-87f7-11e56f3230ff",
            "ignore_rules": {
                "enabled": true,
                "expired_at": "2023-08-16T18:40:57.000Z",
                "id": "aj3jqtvnaf",
                "name": "abc"
            },
            "impact_score": 3.6,
            "last_detected_at": "2023-08-16T18:40:57.000Z",
            "layer_metadata": {
                "details": "xxxx",
                "id": "5e95ff50-5490-514e-87f7-11e56f3230ff",
                "is_base_layer": true
            },
            "link": "https://alas.aws.amazon.com/AL2/ALAS-2022-1780.html",
            "location_path": "package/library/file",
            "name": "CVE-2020-3333",
            "portal_url": "https://app.wiz.io/explorer/vulnerability-findings#~(entity~(~'xxx-xxx*2cSECURITY_TOOL_FINDING))",
            "projects": [
                {
                    "id": "83b76efe-a7b6-5762-8a53-8e8f59e68bd8",
                    "name": "Project2",
                    "risk_profile": {
                        "business_impact": "MBI"
                    },
                    "slug": "project-2"
                },
                {
                    "business_unit": "Dev",
                    "id": "af52828c-4eb1-5c4e-847c-ebc3a5ead531",
                    "name": "project4",
                    "risk_profile": {
                        "business_impact": "MBI"
                    },
                    "slug": "project-4"
                },
                {
                    "business_unit": "Dev",
                    "id": "d6ac50bb-aec0-52fc-80ab-bacd7b02f178",
                    "name": "Project1",
                    "risk_profile": {
                        "business_impact": "MBI"
                    },
                    "slug": "project1"
                }
            ],
            "remedation": "yumupdatelibtiff",
            "resolution_reason": "resolutionReason",
            "resolved_at": "2023-08-16T18:40:57.000Z",
            "score": 5.5,
            "status": "OPEN",
            "validated_in_runtime": true,
            "vendor_severity": "MEDIUM",
            "version": "4.0.3-35.amzn2",
            "vulnerable_asset": {
                "cloud": {
                    "platform": "AWS",
                    "provider_url": "https://us-east-1.console.aws.amazon.com/ec2/v2/home?region=us-east-1#InstanceDetails:instanceId=i-0a0f7e1451da5f4a3"
                },
                "has_limited_internet_exposure": true,
                "has_wide_internet_exposure": true,
                "id": "c828de0d-4c42-5b1c-946b-2edee094d0b3",
                "ip_addresses": [
                    "89.160.20.112",
                    "89.160.20.128"
                ],
                "is_accessible_from": {
                    "other_subscriptions": false,
                    "other_vnets": false,
                    "vpn": false
                },
                "name": "test-4",
                "operating_system": "Linux",
                "provider_unique_id": "arn:aws:ec2:us-east-1:998231069301:instance/i-0a0f7e1451da5f4a3",
                "region": "us-east-1",
                "status": "Active",
                "subscription": {
                    "external_id": "998231069301",
                    "id": "94e76baa-85fd-5928-b829-1669a2ca9660",
                    "name": "wiz-integrations"
                },
                "tags": {
                    "name": "test-4"
                },
                "type": "VIRTUAL_MACHINE"
            }
        }
    }
}
```
