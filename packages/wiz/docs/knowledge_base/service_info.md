# Service Info

## Common use cases
The Wiz integration for Elastic enables organizations to centralize their cloud security posture data, providing a unified view of risks across multi-cloud environments. By ingesting high-fidelity alerts and configuration findings into the Elastic Stack, security teams can correlate cloud vulnerabilities with runtime telemetry for comprehensive threat hunting.

*   **Continuous Compliance Monitoring:** Monitor cloud configuration findings against industry benchmarks (like CIS or SOC2) by ingesting posture data to ensure resources remain compliant over time. This helps in maintaining a "always-compliant" state and simplifies reporting for audits.
*   **Real-time Threat Detection:** Utilize the Wiz Defend data stream to receive immediate notifications of runtime threats, such as container escapes or lateral movement, directly within the Elastic Security SIEM.
*   **Vulnerability Prioritization:** Analyze vulnerability logs alongside active threat intelligence in Elastic to prioritize patching efforts based on actual exposure and resource criticality. By combining Wiz's graph-based analysis with Elastic's search power, teams can focus on the "toxic combinations" that present the highest risk.
*   **Audit and Governance:** Maintain a long-term record of administrative actions and platform mutations via Audit logs to meet regulatory requirements and support forensic investigations. Track who accessed what and what changes were made within the Wiz platform.

## Data types collected
This integration collects several data types from the Wiz platform using API polling (CEL) and HTTP webhooks. Each data stream is designed to map specific Wiz security signals to Elastic:

*   **Audit logs (logs):** Collects Audit logs from Wiz via the `cel` input. This stream records administrative actions within the Wiz portal, including user logins and mutation API calls like write, edit, and delete actions.
*   **Cloud Configuration Finding full posture (logs):** Collects the full Cloud Configuration Finding posture from Wiz using the `cel` input. This provides a comprehensive data stream for the full posture state of cloud configuration findings, useful for total visibility of the cloud environment.
*   **Cloud Configuration Finding logs (logs):** Collects Cloud Configuration Finding logs from Wiz via the `cel` input. These results are generated when cloud resources fail specific configuration rules, indicating potential security misconfigurations that need remediation.
*   **Defend logs (logs):** Collects Detection events from Wiz Defend via the `http_endpoint` input. This provides real-time detection events and visibility into runtime signals and cloud activity threats, such as suspicious process execution or network anomalies.
*   **Issue logs (logs):** Collects Issue logs from Wiz via the `cel` input. This stream tracks active security risks or threats identified within the cloud environment, such as exposed secrets, orphaned resources, or overly permissive identities.
*   **Vulnerability logs (logs):** Collects Vulnerability logs from Wiz via the `cel` input. This includes detailed information regarding software weaknesses and known vulnerabilities (CVEs) identified across cloud workloads, including container images and virtual machine instances.

## Compatibility

The Wiz integration is compatible with the following versions and requirements:

The Wiz integration is compatible with the following vendor specifications:
- **Wiz** API Version v1.
- **Wiz Defend** runtime threat detection module.
- Supports both **Elastic Agent** managed by Fleet and **Agentless** (Serverless/Elastic Cloud) ingestion methods.

## Scaling and Performance

To ensure optimal performance in high-volume cloud security environments, consider the following:
- **Transport/Collection Considerations:** This integration utilizes a hybrid model. The `cel` input is used for "pulling" findings (Audit, Issues, Vulnerabilities) via the GraphQL API, while the `http_endpoint` input "pushes" Defend logs. For API collection, the **Batch Size** (default `500`) and **Maximum Pages Per Interval** (default `1000`) should be tuned based on the size of your cloud environment. High volumes of vulnerability data may require increasing the **HTTP Client Timeout** beyond `30s`.
- **Data Volume Management:** To optimize ingestion, configure Wiz Service Account scopes to only include the specific projects or data types required. For the Vulnerability data stream, use the **Include All Vulnerability Statuses** toggle selectively; by default, the integration uses the API's default filters to reduce noise. Limiting the **Initial Interval** (default `24h`) can prevent excessive data retrieval during the initial synchronization.
- **Elastic Agent Scaling:** For high-throughput environments, particularly those receiving a high volume of Defend Webhook alerts, deploy multiple Elastic Agents behind a network load balancer. This distributes the incoming HTTPS traffic evenly. Ensure that the Agent host has sufficient resources to process the CEL script logic and large JSON payloads, especially when **Preserve original event** is enabled.

# Set Up Instructions

## Vendor prerequisites

1. **Administrative Access**: Ensure you have a Wiz user account with **Project Admin** or **Global Admin** privileges to manage Service Accounts and Webhooks.
2. **Service Account Credentials**:
   - Navigate to **Settings > Service Accounts** in the Wiz portal.
   - Create a Service Account to obtain a **Client ID** and **Client Secret**.
   - Assign the following scopes: `admin:audit`, `read:issues`, `read:vulnerabilities`, and `read:cloud_configuration`.
3. **API Access**: The Elastic Agent must have outbound connectivity to the Wiz API endpoint (e.g., `https://api.us1.app.wiz.io`) and the authentication endpoint (`https://auth.app.wiz.io/oauth/token`).
4. **Network Configuration**: For Defend (Webhook) logs, ensure the Elastic Agent host is reachable from Wiz's infrastructure on the configured **Listen Port** (default `9588`).

## Elastic prerequisites
1.  **Elastic Agent Enrollment:** An Elastic Agent must be installed and enrolled in Fleet, or configured as a Standalone agent.
2.  **Network Access:** The Elastic Agent requires outbound HTTPS access (port `443`) to the Wiz API endpoint and the OAuth2 token URL (`https://auth.app.wiz.io/oauth/token`).

## Vendor set up steps

### For API Polling (Audit, Issue, Vulnerability, Cloud Configuration):
1.  Log in to your Wiz tenant and navigate to your **User Profile** (top right) > **User Settings** > **Tenant**.
2.  Copy the **API Endpoint URL** (e.g., `https://api.us1.app.wiz.io`) for later use in Kibana.
3.  Navigate to **Settings > Service Accounts** and click **Add Service Account**.
4.  Provide a descriptive name such as `Elastic-Integration` and set the type to **Custom Integration (GraphQL API)**.
5.  In the Scopes section, assign the following permissions:
    *   `admin:audit` (for Audit logs)
    *   `read:issues` (for Issues)
    *   `read:vulnerabilities` (for Vulnerabilities)
    *   `read:cloud_configuration` (for Cloud Configuration findings)
6.  Click **Add Service Account**. 
7.  Immediately copy the **Client ID** and **Client Secret** provided in the confirmation dialog. Store them securely as the secret cannot be retrieved again.

### For Webhook Collection (Defend Logs):
1.  Log in to the Wiz console and navigate to **Settings > Integrations**.
2.  Click **Add Integration** and select the **Webhook** option under SIEM & Automation Tools.
3.  In the New Integration page, enter a Name such as `Elastic Agent Defend`.
4.  In the **URL** field, enter the destination address of your Elastic Agent (e.g., `http://<AGENT_IP>:9588/`).
5.  Choose an **Authentication** method. **Token** is recommended for ease of configuration within the Elastic Agent. Enter a strong unique string as the token.
6.  (Optional) Configure Project Scopes or Filters to limit the alerts sent to Elastic.
7.  Click **Add Integration**. Note that for "Defend" logs, you must also ensure your Defend policies in Wiz are configured to send alerts to this specific integration.

## Kibana set up steps

1. In Kibana, navigate to **Integrations** and search for **Wiz**.
2. Click **Add Wiz**.
3. Configure the global settings:
   - **Client ID**: The Client ID for your Wiz Service Account.
   - **Client Secret**: The Client Secret for your Wiz Service Account.
   - **URL**: The base URL of the Wiz API (e.g., `https://api.us1.app.wiz.io`).

### Collecting Wiz logs via API (Cloud Configuration Finding full posture)
1.  **Preserve original event**: (Default: `False`) Enable to save the raw JSON in `event.original`.
2.  **Batch Size**: (Default: `500`) Number of records to fetch per API request.
3.  **HTTP Client Timeout**: (Default: `30s`) Time to wait before connection timeout.
4.  **Maximum Pages Per Interval**: (Default: `1000`) Limit the number of pages fetched per polling interval.
5.  **Enable request tracing**: (Default: `False`) Enable only for debugging to log requests/responses.
6.  **Tags**: List of tags to add to the events (Default: `forwarded`, `wiz-cloud_configuration_finding_full_posture`).
7.  **Preserve duplicate custom fields**: (Default: `False`) Keeps original Wiz fields that were mapped to ECS.

### Collecting Wiz logs via API (Cloud Configuration Finding logs)
1.  **Initial Interval**: (Default: `24h`) How far back to look for logs during the first collection.
2.  **Interval**: (Default: `5m`) Frequency of API polling.
3.  **Preserve original event**: (Default: `False`) Enable to save the raw JSON in `event.original`.
4.  **Batch Size**: (Default: `500`) Number of records per request.
5.  **HTTP Client Timeout**: (Default: `30s`) Connection timeout duration.
6.  **Maximum Pages Per Interval**: (Default: `1000`) Max pages per polling cycle.
7.  **Tags**: (Default: `forwarded`, `wiz-cloud_configuration_finding`).

### Collecting Wiz logs via API (Audit logs)
1.  **Initial Interval**: (Default: `24h`) Lookback period for initial ingest.
2.  **Interval**: (Default: `5m`) Polling frequency.
3.  **Preserve original event**: (Default: `False`) Enable to save the raw JSON.
4.  **Batch Size**: (Default: `500`) Records per request.
5.  **HTTP Client Timeout**: (Default: `30s`) timeout settings.
6.  **Tags**: (Default: `forwarded`, `wiz-audit`).

### Collecting Wiz logs via API (Issue logs)
1.  **Initial Interval**: (Default: `24h`) Initial data lookback.
2.  **Interval**: (Default: `5m`) API poll frequency.
3.  **Preserve original event**: (Default: `False`) Enable to save raw JSON.
4.  **Batch Size**: (Default: `500`) records per call.
5.  **Maximum Pages Per Interval**: (Default: `1000`) Page limit per interval.
6.  **Tags**: (Default: `forwarded`, `wiz-issue`).

### Collecting Wiz logs via API (Vulnerability logs)
1.  **Include All Vulnerability Statuses**: (Default: `False`) Toggle to fetch RESOLVED, REJECTED, and IN_PROGRESS vulnerabilities in addition to OPEN ones.
2.  **Initial Interval**: (Default: `24h`) Initial ingest lookback.
3.  **Interval**: (Default: `5m`) Polling frequency.
4.  **Preserve original event**: (Default: `False`) Save raw copy in `event.original`.
5.  **Batch Size**: (Default: `500`) Max batch size.
6.  **Tags**: (Default: `forwarded`, `wiz-vulnerability`).

### Collecting Detection events from Wiz Defend via HTTP Endpoint
1.  **Listen Address**: (Default: `localhost`) The address the agent listens on. Use `0.0.0.0` for all interfaces.
2.  **Listen Port**: (Default: `9588`) The port number for the webhook listener.
3.  **Authentication (Basic)**: (Default: `False`) Enable for username/password auth.
4.  **Username**: Username if Basic Auth is enabled.
5.  **Password**: Password if Basic Auth is enabled.
6.  **Authentication (Token)**: The secret token value required to authenticate the Wiz webhook.
7.  **URL**: (Default: `/`) The path where the webhook is accepted.
8.  **Tags**: (Default: `forwarded`, `wiz-defend`).
9.  **Preserve duplicate custom fields**: (Default: `False`) Keeps original fields after ECS mapping.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on Wiz:
- **Generate Audit Event:** Log out and log back into the Wiz administration interface to trigger an authentication audit log.
- **Generate Issue Event:** Navigate to an existing Issue in the Wiz UI and change its status or add a comment to trigger an update.
- **Trigger Vulnerability Scan:** If possible, initiate a manual scan on a specific resource to generate new vulnerability logs.
- **Test Webhook:** Navigate to **Settings > Integrations** in Wiz, locate your Webhook integration, and click the **Test** button to send a sample payload to the Elastic Agent.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "wiz.audit"`
4. Verify logs appear. Expand a log entry and confirm these fields are present and populated:
   - `event.dataset` (should match `wiz.audit`, `wiz.issue`, etc.)
   - `source.ip` (populated for `wiz.defend` webhook events)
   - `event.action` (e.g., `info-system-login`)
   - `message` (contains the raw log payload)
5. Navigate to **Analytics > Dashboards** and search for "Wiz" to verify that the out-of-the-box dashboards are populating with data.

# Troubleshooting

## Common Configuration Issues
- **Missing event.ingested Field**: 📌 Action Required. If using **standalone** Elastic Agents, the `event.ingested` field is not added automatically because the Fleet-managed pipeline is not executed. This will cause transforms to fail. You must manually add this field via a custom ingest pipeline (e.g., `@custom`).
- **Webhook Port Unreachable**: If Defend logs are not appearing, verify that the Elastic Agent host firewall allows inbound traffic on the configured port (default 9588). Ensure the "Listen Address" is set to `0.0.0.0` rather than `localhost` if the Agent is receiving traffic from an external network.
- **Incomplete API Scopes**: If specific data streams (like Vulnerabilities) are missing while others (like Issues) work, verify that the Wiz Service Account has the exact required scope (e.g., `read:vulnerabilities`) assigned in the Wiz portal.

## Ingestion Errors
- **Vulnerability Data Latency**: Vulnerability data is typically fetched based on the polling interval. If logs do not appear immediately after setup, check the Agent logs for any "403 Forbidden" or "401 Unauthorized" errors which indicate scope or credential issues.
- **Parsing Failures**: Check the `error.message` field in Kibana Discover. If the Wiz API returns an unexpected GraphQL schema change, the CEL input might fail to parse specific fields. Ensure the integration is updated to the latest version.

## API Authentication Errors

- **OAuth Token Failures**: Ensure the `Token URL` is correct (typically `https://auth.app.wiz.io/oauth/token`). Check the Elastic Agent logs for "401 Unauthorized" errors, which indicate the `Client ID` or `Client Secret` is incorrect or has expired.
- **Custom Header Limitations**: Note that the integration only supports the standard `Authorization` header. If your environment requires custom headers for API traffic, the integration may not function as expected.

## Vendor Resources

- Official Wiz Website
- [Wiz Webhook Integration Documentation](https://docs.wiz.io/docs/webhook-integration)
- Refer to the official vendor website for additional resources.

# Documentation sites
- [Wiz Webhook Integration Documentation](https://docs.wiz.io/docs/webhook-integration) - Specific instructions for configuring the Wiz side of the webhook.
- Refer to the official vendor website for additional general product guides.
