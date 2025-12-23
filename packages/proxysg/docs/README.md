## Overview

The Broadcom ProxySG integration for Elastic enables the collection and analysis of access logs from Broadcom ProxySG appliances. By ingesting these logs into the Elastic Stack, organizations gain comprehensive visibility into web traffic, user activities, and security events. This allows for effective monitoring of usage patterns, enforcement of compliance policies, and rapid detection of security threats.

This integration facilitates:
- **Web Traffic Monitoring and Control**: Monitor, filter, and control web traffic to ensure compliance with organizational policies and enhance security.
- **Data Loss Prevention (DLP)**: Inspect outbound traffic to help prevent sensitive data exfiltration.
- **Malware Protection**: Scan web traffic for malware and block malicious content before it reaches end-users.
- **SSL Inspection**: Identify and block malicious activities hidden within encrypted traffic.
- **Bandwidth Management**: Optimize network performance by analyzing bandwidth usage and cache efficiency.

### Compatibility

This integration is compatible with Broadcom ProxySG appliances that support the following access log formats:
- `main`
- `bcreportermain_v1`
- `bcreporterssl_v1`
- `ssl`

Ensure your ProxySG appliance is configured to output logs in one of these standard formats.

### How it works

This integration collects logs from ProxySG appliances using two primary methods:

1.  **Syslog (UDP/TCP)**: The ProxySG appliance is configured to stream access logs via syslog directly to the Elastic Agent. This is the recommended method for real-time monitoring.
2.  **File-based Collection**: The ProxySG appliance uploads log files to a server (via FTP/SCP) where the Elastic Agent is installed. The agent watches the directory and ingests new log files as they appear.

Once collected, the Elastic Agent parses the logs according to the detected format (e.g., `bcreportermain_v1`) and forwards the structured data to Elasticsearch for storage, analysis, and visualization.

## What data does this integration collect?

The Broadcom ProxySG integration collects **Access Logs**, which provide detailed records of web traffic processed by the proxy. Depending on the log format configured, this data includes:

-   **Traffic Details**: URLs accessed, HTTP methods, bytes transferred, status codes, and action taken (e.g., TCP_HIT, TCP_MISS).
-   **User Information**: Usernames, authentication groups, and client IP addresses.
-   **Security Events**: Blocked sites, malware detection events, threat risk scores, and SSL validation status.
-   **Performance Metrics**: Request duration and connection details.

The integration supports the following log formats:
- `main`
- `bcreportermain_v1`
- `bcreporterssl_v1`
- `ssl`

### Supported use cases

Integrating Broadcom ProxySG with Elastic enables several critical security and operational use cases:

-   **Security Auditing**: Maintain a complete audit trail of all web access to investigate security incidents and policy violations.
-   **Threat Detection**: Correlate proxy logs with other security data to identify complex threats and compromised hosts.
-   **Usage Analytics**: Analyze web traffic trends to optimize network resources and user productivity.
-   **Compliance Reporting**: Generate reports on web usage and blocked content to meet regulatory requirements.

## What do I need to use this integration?

To use this integration, you need:

*   **Broadcom ProxySG Appliance**: You must have administrative access to the ProxySG Management Console to configure access logging.
*   **Elastic Agent**: The agent must be installed on a host that is reachable by the ProxySG appliance (for syslog) or has access to the uploaded log files.
*   **Network Connectivity**:
    *   For syslog (TCP/UDP): A network path must exist between the ProxySG appliance and the Elastic Agent host.
    *   For file-based collection: The ProxySG appliance must be able to upload files to a location readable by the Elastic Agent.
*   **Supported Log Formats**: The ProxySG appliance must be configured to use one of the following log formats:
    *   `main`
    *   `bcreportermain_v1`
    *   `bcreporterssl_v1`
    *   `ssl`

## How do I deploy this integration?

### Agent-based deployment

Elastic Agent must be installed. For more details, check the Elastic Agent [installation instructions](docs-content://reference/fleet/install-elastic-agents.md). You can install only one Elastic Agent per host.

Elastic Agent is required to stream data from the syslog or log file receiver and ship the data to Elastic, where the events will then be processed via the integration's ingest pipelines.

### Set up steps in Broadcom ProxySG

You can configure Broadcom ProxySG to send logs using either Syslog (recommended for real-time monitoring) or File Upload.

### Set up steps in Kibana

1.  In Kibana, navigate to **Management** > **Integrations**.
2.  Search for "ProxySG" and select the **Broadcom ProxySG** integration.
3.  Click **Add Broadcom ProxySG**.
4.  Select the appropriate **Input type** based on your ProxySG configuration:
    *   **Collect logs from ProxySG via UDP**: For UDP syslog collection
    *   **Collect logs from ProxySG via TCP**: For TCP syslog collection
    *   **Collect access logs from ProxySG via logging server file**: For file-based collection
5.  Configure the input settings:
    *   **For UDP/TCP**: Set the **Listen Address** (default: `localhost`, use `0.0.0.0` to bind to all interfaces) and **Listen Port**.
    *   **For File-based**: Set **Paths** to the file pattern matching the location where ProxySG uploads logs on the remote server (e.g., `/var/log/proxysg-log.log`).
6.  **Important**: In **Advanced options**, select the **Access Log Format** that matches the format configured on your ProxySG appliance.
7.  Click **Save and Continue**.

### Validation

1.  **Generate Test Traffic**: Access various websites through the ProxySG appliance to generate log entries.
2.  **Check Data in Kibana**:
    *   Navigate to **Discover**.
    *   Select the `logs-*` data view.
    *   Search for `event.dataset: "proxysg.log"` to filter for ProxySG events.
    *   Verify that logs are appearing with the correct timestamps and fields (e.g., `client.ip`, `url.domain`, `http.request.method`).
3.  **Review Dashboards**:
    *   Navigate to **Dashboards** in Kibana.
    *   Search for "ProxySG" to verify that the dashboard visualizations are populating with data.

## Troubleshooting

For help with Elastic ingest tools, check [Common problems](https://www.elastic.co/docs/troubleshoot/ingest/fleet/common-problems).

### Common Configuration Issues

-   **No logs appearing in Kibana**:
    *   **Check Connectivity**: Ensure the ProxySG appliance can reach the Elastic Agent on the configured IP and Port. Check firewalls and routing.
    *   **Verify Agent Status**: Ensure the Elastic Agent is healthy and the integration policy is applied.
    *   **Check Listen Interface**: If sending from a remote appliance, ensure Listen Address is set to `0.0.0.0` to bind to all interfaces, not `localhost`.
    *   **Verify ProxySG Configuration**: In the ProxySG Management Console, check **Configuration > Access Logging > Logs** and ensure the "Log Hosts" settings match your Elastic Agent configuration.

-   **Parsing Errors / Incorrect Fields**:
    *   **Format Mismatch**: The most common cause is a mismatch between the **Access Log Format** selected in the integration and the actual format configured on the ProxySG appliance. Verify both are set to the same standard format (e.g., both set to `main`).
    *   **Custom Formats**: This integration supports the standard vendor formats (`main`, `bcreportermain_v1`, `bcreporterssl_v1`, `ssl`). If you have customized the log string on the ProxySG, parsing may fail. Revert to a standard format or use the `preserve_original_event` option to debug.

-   **Missing SSL Fields**:
    *   If SSL-related fields are empty, ensure you are using a log format that supports them, such as `ssl` or `bcreporterssl_v1`.

### Ingestion Errors

-   **Timestamp Issues**:
    *   Ensure the ProxySG appliance and the Elastic Agent host are synchronized with a reliable NTP source to prevent timestamp skews. Verify timezone settings on both systems.

### Vendor Resources

-   [Broadcom ProxySG Log Formats](https://techdocs.broadcom.com/us/en/symantec-security-software/web-and-network-security/edge-swg/7-3/getting-started/page-help-administration/page-help-logging/log-formats/default-formats.html)
-   [Blue Coat Systems Product Use Guide](https://docs.broadcom.com/doc/blue-coat-systems-product-use-guide-en)
-   [SGOS Administration Guide](https://techdocs.broadcom.com/us/en/symantec-security-software/web-and-network-security/edge-swg.html)

## Performance and scaling

Consider the following recommendations for optimal performance and scalability when collecting data from Broadcom ProxySG:

- *Log Volume*: ProxySG appliances can generate high volumes of logs. Ensure your network bandwidth and the Elastic Agent host resources (CPU/RAM) are sufficient to handle the load.
- *Load Balancing*: For high availability and scaling, you can place a load balancer in front of multiple Elastic Agents and configure the ProxySG to send logs to the load balancer VIP.

For more information on architectures that can be used for scaling this integration, check the [Ingest Architectures](https://www.elastic.co/docs/manage-data/ingest/ingest-reference-architectures) documentation.

## Reference

### log

The `log` data stream collects access logs from the ProxySG appliance. It supports the following log formats: `main`, `bcreportermain_v1`, `bcreporterssl_v1`, and `ssl`.

### Inputs used

These inputs can be used with this integration:
<details>
<summary>filestream</summary>

## Setup

For more details about the Filestream input settings, check the [Filebeat documentation](https://www.elastic.co/docs/reference/beats/filebeat/filebeat-input-filestream).


### Collecting logs from Filestream

To collect logs via Filestream, select **Collect logs via Filestream** and configure the following parameters:

- Filestream paths: The full path to the related log file.
</details>
<details>
<summary>tcp</summary>

## Setup

For more details about the TCP input settings, check the [Filebeat documentation](https://www.elastic.co/docs/reference/beats/filebeat/filebeat-input-tcp).

### Collecting logs from TCP

To collect logs via TCP, select **Collect logs via TCP** and configure the following parameters:

**Required Settings:**
- Host
- Port

**Common Optional Settings:**
- Max Message Size - Maximum size of incoming messages
- Max Connections - Maximum number of concurrent connections
- Timeout - How long to wait for data before closing idle connections
- Line Delimiter - Character(s) that separate log messages

## SSL/TLS Configuration

To enable encrypted connections, configure the following SSL settings:

**SSL Settings:**
- Enable SSL*- Toggle to enable SSL/TLS encryption
- Certificate - Path to the SSL certificate file (`.crt` or `.pem`)
- Certificate Key - Path to the private key file (`.key`)
- Certificate Authorities - Path to CA certificate file for client certificate validation (optional)
- Client Authentication - Require client certificates (`none`, `optional`, or `required`)
- Supported Protocols - TLS versions to support (e.g., `TLSv1.2`, `TLSv1.3`)

**Example SSL Configuration:**
```yaml
ssl.enabled: true
ssl.certificate: "/path/to/server.crt"
ssl.key: "/path/to/server.key"
ssl.certificate_authorities: ["/path/to/ca.crt"]
ssl.client_authentication: "optional"
```
</details>
<details>
<summary>udp</summary>

## Setup

For more details about the UDP input settings, check the [Filebeat documentation](https://www.elastic.co/docs/reference/beats/filebeat/filebeat-input-udp).

### Collecting logs from UDP

To collect logs via UDP, select **Collect logs via UDP** and configure the following parameters:

**Required Settings:**
- Host
- Port

**Common Optional Settings:**
- Max Message Size - Maximum size of UDP packets to accept (default: 10KB, max: 64KB)
- Read Buffer - UDP socket read buffer size for handling bursts of messages
- Read Timeout - How long to wait for incoming packets before checking for shutdown
</details>

