## Overview

The Hashicorp Vault integration for Elastic enables collection of audit logs, operational logs, and metrics to provide comprehensive visibility into the security and operational health of your Vault environment. You can monitor detailed audit trails, track internal server operations, and analyze performance metrics to ensure stability and compliance.

This integration facilitates:
- **Security monitoring**: Ingests audit logs to track all requests and responses, crucial for threat detection and compliance.
- **Operational visibility**: Collects internal server logs including startup, shutdown, and error messages.
- **Performance analysis**: Gathers telemetry data in Prometheus format to monitor memory usage, request latency, and secret engine activity.

### Compatibility

This integration has been tested with Hashicorp Vault version 1.11. It is expected to be compatible with other recent versions.

### How it works

The integration uses different methods to collect data depending on the source:

- **Audit logs**: Can be collected via:
  - **Syslog**: Vault sends logs to a local syslog daemon, which forwards them to Elastic Agent over TCP (recommended for production).
  - **File**: Elastic Agent monitors the file written by Vault's `file` audit device.
  - **TCP Socket**: Elastic Agent listens on a TCP port, and Vault streams logs directly to it.
- **Operational logs**: Elastic Agent monitors the file where Vault's standard output (stdout) is redirected. Logs must be in JSON format.
- **Metrics**: Elastic Agent periodically scrapes the Prometheus-compatible metrics endpoint exposed by Vault.

## What data does this integration collect?

This integration collects logs and metrics from HashiCorp Vault across three data streams:

* **`audit`**: Captures detailed audit logs, including authentication attempts, secret access, policy changes, and system operations. The logs contain hashed secret values (HMAC-SHA256) for security.
    * **Note**: It is critical to understand how Vault handles audit device failures. If an audit device cannot write logs, Vault may block requests to ensure no action is performed without an audit record. Refer to [Blocked Audit Devices](https://developer.hashicorp.com/vault/docs/audit/socket#configuration) and [Blocked Audit Device Behavior](https://developer.hashicorp.com/vault/tutorials/monitoring/blocked-audit-devices#blocked-audit-device-behavior) for details on configuration and behavior.
* **`log`**: Captures operational logs from the Vault server itself, including errors, warnings, and informational messages about the server's status.
* **`metrics`**: Collects a wide range of performance metrics, such as counters, gauges, and summaries related to runtime, secret engines, and token counts.

### Supported use cases

* **Security auditing**: Track every request and response to Vault to detect unauthorized access, policy violations, and suspicious activity.
* **Operational monitoring**: Monitor the health of your Vault cluster, diagnose issues using operational logs, and set up alerts for critical errors.
* **Performance analysis**: Analyze performance metrics to understand request latency, resource utilization, and identify performance bottlenecks.
* **Compliance**: Maintain a detailed, verifiable log of all activities for compliance with regulations like PCI DSS, SOX, and HIPAA.

## What do I need to use this integration?

Before you begin, ensure you have the following prerequisites configured:

- **Elastic Agent**: You need a Fleet-managed Elastic Agent installed on a host that can communicate with your HashiCorp Vault cluster.
- **HashiCorp Vault**: An active installation of HashiCorp Vault.

### Credentials and Permissions
- **Administrative Access**: To configure audit logging, you need a Vault token with `sudo` capabilities or a policy that allows enabling and configuring audit devices.
- **Metrics Access**: For metrics collection, you need a Vault token with read access to the `/sys/metrics` API endpoint.
  - Example policy capability: `path "sys/metrics" { capabilities = ["read"] }`

### Network Connectivity
- **Metrics Collection**: The Elastic Agent must be able to reach the Vault API endpoint (default: `http://localhost:8200`).
- **Audit Log Collection (Socket)**: If using the socket audit device, the Vault server must be able to initiate a TCP connection to the Elastic Agent (default port: `9007`).
- **Firewall Rules**: Ensure firewall rules allow traffic on the configured ports (e.g., 8200 for API, 9007 for audit logs).

### Vault Configuration
- **Operational Logs**: Vault must be configured to output logs in JSON format (`log_format = "json"`) for proper parsing.
- **Telemetry**: For metrics, Vault telemetry must be enabled with `disable_hostname = true` and `enable_hostname_label = true`.
- **Filesystem Access**: If collecting logs from files, the Elastic Agent requires read access to the Vault log directories (e.g., `/var/log/vault/`).
- **Safety Requirement**: If using the TCP socket audit device, HashiCorp strongly recommends configuring a secondary file-based audit device. This prevents Vault from blocking requests if the TCP connection to the Elastic Agent becomes unavailable.

## How do I deploy this integration?

### Agent-based deployment

The Elastic Agent is a unified agent that collects data from your systems and ships it to Elastic. To deploy this integration:

1. **Install Elastic Agent** on a host that has network access to both your Elastic deployment and the data source.
   - See the [Elastic Agent installation guide](https://www.elastic.co/guide/en/fleet/current/install-fleet-managed-elastic-agent.html).

2. **Enroll the agent** in Fleet:
   - In Kibana, go to **Management** → **Fleet** → **Agents**.
   - Click **Add agent** and follow the enrollment instructions.

3. **Add the integration** to an agent policy:
   - Go to **Management** → **Integrations**.
   - Search for **Hashicorp Vault**.
   - Click **Add Hashicorp Vault** and configure the settings.
   - Assign to an existing policy or create a new one.

**Network Requirements**

| Direction | Protocol | Port | Purpose |
|-----------|----------|------|----------|
| Agent → Elastic | HTTPS | 443 | Data shipping to Elasticsearch |
| Agent (local) | — | — | File read access required (Audit/Operational logs) |
| Source → Agent | TCP | 9007 (configurable) | Socket Audit Device (if enabled) |
| Agent → Source | HTTP | 8200 (configurable) | Metrics collection |

### Set up steps in Hashicorp Vault

Before using the integration, you must configure HashiCorp Vault to expose metrics and logs in a format that Elastic Agent can collect.

### Set up steps in Kibana

1. In Kibana, navigate to **Management** > **Integrations**.
2. Search for "HashiCorp Vault" and select the integration.
3. Click **Add HashiCorp Vault**.
4. Configure the integration inputs based on your setup:

   **For Audit Logs (File)**:
   - Enable the **Logs from file** > **Audit logs (file audit device)** input.
   - Specify the file path (default: `/var/log/vault/audit*.json*`).
   - Optionally enable **Preserve original event** to keep the raw log message.

   **For Audit Logs (TCP Socket)**:
   - Enable the **Logs from TCP socket** input.
   - Configure the **Listen address** (default: `localhost`) and **Port** (default: `9007`).
   - If Vault is connecting from a remote host, change the listen address to `0.0.0.0`.

   **For Operational Logs**:
   - Enable the **Logs from file** > **Operation logs** input.
   - Specify the log file path (default: `/var/log/vault/log*.json*`).

   **For Metrics**:
   - Enable the **Metrics** input.
   - Enter the **Hosts** URL (default: `http://localhost:8200`).
   - Enter the **Token** generated in the previous section.
   - Adjust the **Period** if needed (default: `30s`).

5. select the agent policy where you want to run this integration.
6. Click **Save and continue**.

### Validation

After deployment, you can verify that data is flowing correctly.

1. **Verify Elastic Agent status**
   - In Kibana, navigate to **Management** → **Fleet** → **Agents**.
   - Confirm the agent status shows **Healthy** (green).
   - Click the agent name to verify the Hashicorp Vault integration is listed and shows no errors.

2. **Trigger data flow on HashiCorp Vault**
   - **Generate Authentication Event**: Perform a login operation using the Vault CLI:
     ```bash
     vault login -method=token
     ```
   - **Generate Secret Access Event**: Read a test secret from the KV store:
     ```bash
     vault kv get secret/test_connection
     ```
   - **Generate Administrative Event**: List the enabled audit devices:
     ```bash
     vault audit list
     ```

3. **Check for incoming data in Discover**
   - Go to **Analytics** → **Discover**.
   - Set the time range to **Last 15 minutes** or longer.
   - Search for data from this integration using: `data_stream.dataset:hashicorp_vault*`
   - This will show ALL data streams for the integration (logs, metrics, etc.).
   - Verify documents are appearing with recent timestamps.

4. **Verify specific data streams**
   - **Audit Logs**: Filter for `data_stream.dataset:hashicorp_vault.audit`.
   - **Operational Logs**: Filter for `data_stream.dataset:hashicorp_vault.log`.
   - **Metrics**: Filter for `data_stream.dataset:hashicorp_vault.metrics`.

5. **Check dashboards**
   - Navigate to **Management** → **Integrations** → **Hashicorp Vault**.
   - Click the **Assets** tab to see available dashboards.
   - Open the **[Logs HashiCorp Vault] Audit Overview** dashboard to confirm visualizations are populated with data.

## Troubleshooting

### General debugging steps

### Vendor-specific issues

### Log file input troubleshooting

### TCP/Syslog input troubleshooting

### Prometheus metrics input troubleshooting

## Performance and scaling

For more information on architectures that can be used for scaling this integration, check the [Ingest Architectures](https://www.elastic.co/docs/manage-data/ingest/ingest-reference-architectures) documentation.

The **Hashicorp Vault** integration uses several input types, each with specific performance and scaling considerations.

### Log File Input (`logfile`)
- **Fault Tolerance**: The Elastic Agent tracks its position in each log file, ensuring no data is lost during agent restarts.
- **Scaling Guidance**:
  - For directories with many log files, use glob patterns (e.g., `/var/log/vault/*.json`) to monitor them efficiently.
  - Adjust the `harvester_limit` setting to control the number of concurrently harvested files and manage resource usage.
  - Use the `close_inactive` setting to release file handles for rotated logs. This is particularly important for high-volume systems where log rotation occurs frequently.

### TCP/Syslog Input (`tcp`)
- **Fault Tolerance**: TCP provides reliable, ordered delivery with acknowledgments, making it suitable for production environments where data integrity is critical.
- **Scaling Guidance**:
  - To scale, you can deploy multiple Elastic Agents and use a load balancer to distribute syslog traffic across them.
  - TCP naturally handles backpressure. If Elasticsearch is slow to ingest, the TCP connection will queue data.
  - Monitor connection limits on both the Vault host and the Elastic Agent host to ensure they can handle the required number of connections.
- **Critical Configuration**: When using the socket audit backend, Hashicorp Vault effectively blocks if the audit device (in this case, Elastic Agent) becomes unavailable or cannot read data fast enough. This can cause Vault to stop processing client requests.
  - Review the Hashicorp documentation on [Blocked Audit Devices](https://developer.hashicorp.com/vault/docs/audit/socket#configuration) to understand how to configure the `hmac_accessor`, `socket_type`, and potential fallbacks to prevent service outages.

### Prometheus Metrics Input (`prometheus`)
- **Fault Tolerance**: This input polls the Vault metrics endpoint. If a poll fails, data for that interval is lost, but the next successful poll will resume collection.
- **Scaling Guidance**:
  - Adjust the collection interval (default is often 10s or 1m) to balance data resolution with the load on the Vault API.
  - High-cardinality metrics can increase storage usage in Elasticsearch. If you do not need all metrics, consider filtering specific metric sets.
  - Ensure the Elastic Agent is located close to the Vault cluster (network-wise) to minimize latency during scrapes.

## Reference

### audit

The `audit` data stream provides events from Hashicorp Vault of the following types: audit logs.

These logs contain detailed information about requests and responses to Vault, allowing you to track access and actions performed against the secrets engine.

{{event "audit"}}

{{fields "audit"}}

### log

The `log` data stream provides events from Hashicorp Vault of the following types: operational logs.

Operational logs provide visibility into the behavior of the Vault server itself, including errors, warnings, and informational messages regarding the service state and backend storage.

{{event "log"}}

{{fields "log"}}

### metrics

The `metrics` data stream provides events from Hashicorp Vault of the following types: metrics.

This data stream collects telemetry data from the Vault `/v1/sys/metrics` endpoint, providing insights into performance, runtime statistics, and resource usage.

{{event "metrics"}}

{{fields "metrics"}}

### Inputs used

| Data Stream | Input Type | Description |
|-------------|------------|-------------|
| `audit` | `syslog` | Collects audit logs via syslog forwarding (recommended). |
| `audit` | `logfile` | Alternative method: monitors the audit log file directly. |
| `audit` | `tcp` | Alternative method: listens on a TCP socket for direct log streaming from Vault. |
| `log` | `logfile` | Monitors the operational log file. |
| `metrics` | `prometheus`| Scrapes the `/v1/sys/metrics` endpoint for Prometheus-formatted metrics. |

### API usage

The `metrics` data stream makes periodic HTTP GET requests to the `/v1/sys/metrics?format=prometheus` endpoint on the configured Vault server URL. The frequency of these requests is determined by the **Period** setting in the integration configuration. Be mindful of the load this may place on your Vault server, especially with very short collection intervals.

### Vendor documentation links

- [Blocked Audit Devices](https://developer.hashicorp.com/vault/docs/audit/socket#configuration)
- [Blocked Audit Device Behavior](https://developer.hashicorp.com/vault/tutorials/monitoring/blocked-audit-devices#blocked-audit-device-behavior)
- [HashiCorp Vault Deployment Guide](https://developer.hashicorp.com/vault/tutorials/day-one-raft/raft-deployment-guide)
- [HashiCorp Vault Audit Devices](https://developer.hashicorp.com/vault/docs/audit)
- [HashiCorp Vault File Audit Device](https://developer.hashicorp.com/vault/docs/audit/file)
- [HashiCorp Vault Socket Audit Device](https://developer.hashicorp.com/vault/docs/audit/socket)
- [HashiCorp Vault Telemetry Configuration](https://developer.hashicorp.com/vault/docs/configuration/telemetry)
- [HashiCorp Vault Troubleshooting](https://developer.hashicorp.com/vault/docs/troubleshoot)
- [Syslog - Audit Devices | Vault | HashiCorp Developer](https://developer.hashicorp.com/vault/docs/audit/syslog)
