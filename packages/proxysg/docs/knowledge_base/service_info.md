# Service Info

## Common use cases

Broadcom ProxySG (Edge SWG) is a comprehensive secure web gateway solution designed to provide advanced web security and wide-area network optimization. This integration allows organizations to ingest access logs into the Elastic Stack for deep visibility into web traffic and security posture.

- **Security Monitoring and Threat Detection:** Analyze ProxySG access logs to identify malicious web requests, SSL inspection results, and potential advanced persistent threats (APTs) targeting the internal network. Detailed logging allows for the identification of suspicious domains and anomalous traffic patterns.
- **User Activity Auditing:** Monitor web usage patterns and user authentication events to ensure compliance with corporate internet usage policies and regulatory requirements. This is critical for maintaining an audit trail of user access to sensitive or restricted web resources.
- **Bandwidth and Performance Optimization:** Review caching efficiency and bandwidth usage trends to optimize network performance. By analyzing byte counts and cache hit/miss ratios, administrators can reduce costs associated with external web traffic and improve user experience.
- **Incident Response and Forensics:** Utilize detailed access logs to reconstruct the timeline of a security incident. The integration provides the necessary context to trace the source, destination, and outcome of suspicious web connections during a post-mortem analysis.

## Data types collected

This integration can collect the following types of data as defined by its data streams:

- **ProxySG Access logs**: Collect ProxySG access logs from file. This stream uses the `filestream` input and is ideal for environments where ProxySG uploads logs to a central logging server or staging directory via FTP or SCP.
- **ProxySG logs (via UDP)**: Collect ProxySG logs (via UDP). This stream uses the `udp` input to receive real-time, high-speed log transmissions from the appliance without the overhead of connection management.
- **ProxySG logs (via TCP)**: Collect ProxySG logs (via TCP). This stream uses the `tcp` input for reliable, connection-oriented streaming of access logs, ensuring no data loss during network transit.

Supported formats and methods include:
- **Web Access Logs:** Detailed records of every HTTP, HTTPS, and FTP request processed by the appliance, including client information, request URIs, and server responses.
- **Security Event Data:** Logs indicating URL filtering actions, threat protection triggers, and SSL inspection outcomes.
- **Data Formats:** The integration specifically supports the Broadcom **main** log format, which includes standardized fields for web traffic analysis.

## Compatibility

The Broadcom ProxySG integration is compatible with the following versions:
- **Broadcom ProxySG (Edge SWG)** version **7.3** and higher.
- **Elastic Stack** version **8.13.0** or higher is required to support the configuration options and data streams provided in this package.
- The integration is specifically tested and documented for the **main** access log format. Other custom log formats provided by ProxySG may require additional parsing adjustments.

## Scaling and Performance

To ensure optimal performance in high-volume environments, consider the following:

- **Transport/Collection Considerations:** For network-based ingestion, **TCP** is recommended for reliability as it ensures log delivery through flow control. **UDP** offers lower overhead but may result in data loss during network congestion. If the environment produces extremely high log volumes, using the **filestream** input (via log file upload) is the most performant method as it allows the Elastic Agent to read from disk and handle backpressure more effectively.
- **Data Volume Management:** Configure the ProxySG appliance to forward only necessary events. Use the ProxySG configuration to filter out high-volume, low-value logs such as internal health checks or specific noisy categories. Ensure the `main` log format is used to prevent parsing overhead associated with unknown custom formats.
- **Elastic Agent Scaling:** In high-throughput environments, deploy the Elastic Agent on a dedicated host with sufficient resources. For very large deployments, use multiple Elastic Agents behind a network load balancer to distribute the incoming syslog (TCP/UDP) traffic across multiple processing nodes.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** Credentials for the ProxySG Management Console and the CLI (Enable/Configure mode).
- **Network Connectivity:** Unrestricted network path between the ProxySG appliance and the Elastic Agent on the configured port (e.g., `514` for UDP, `601` for TCP).
- **Log Format Configuration:** The ProxySG must be configured to use the `main` access log format; other custom formats will result in parsing failures.
- **Feature License:** Ensure the Access Logging feature is enabled and licensed on the ProxySG appliance.
- **External Storage (Optional):** If using the File Upload method, an FTP server accessible by both the ProxySG and the Elastic Agent is required.

## Elastic prerequisites

- **Elastic Agent:** An active Elastic Agent must be installed and enrolled in Fleet.
- **Stack Version:** Ensure the Elastic Stack (Elasticsearch and Kibana) is on version **8.13.0** or higher.
- **Network Configuration:** Configure firewalls between the ProxySG appliance and the Elastic Agent to allow inbound traffic on the specific ports selected (e.g., `514`, `601`).

## Vendor set up steps

### For Syslog (TCP/UDP) Collection:
1. Log in to the ProxySG management console and navigate to **Configuration > Access Logging > Logs**.
2. Click **New** to create a new access log. Set the **Log Name** (e.g., `elastic-syslog`) and select the `main` format from the **Log Format** dropdown. Set **Save the log file as** to `Text file`.
3. Navigate to the **Upload Client** tab and set the **Client type** to `Custom Client`.
4. In the **Settings** section, enter the IP address and port of your Elastic Agent (e.g., `192.168.1.50:601` for TCP or `192.168.1.50:514` for UDP).
5. In the **Upload Schedule** tab, set the **Upload type** to `continuously`.
6. Open the **Visual Policy Manager (VPM)**, add a rule to a **Web Access Layer**, and set the **Action** to **Modify Access Logging**. Enable logging for your new `elastic-syslog` log.
7. Click **Install Policy** to apply the changes.

### For File Upload (FTP) Collection:
1. In the ProxySG console, navigate to **Configuration > Access Logging > Logs**.
2. Create a new log named `elastic-ftp`, selecting the `main` format and `Text file` type.
3. In the **Upload Client** tab, select `FTP Client` and provide the credentials and directory path for your FTP server.
4. Set the **Upload Schedule** to `continuously`.
5. Apply the configuration and use the VPM to enable logging to the `elastic-ftp` object as described in the Syslog steps.
6. (Optional) Use the ProxySG CLI to optimize upload frequency:
   ```bash
   enable
   configure terminal
   access-log
   edit log elastic-ftp
   connect-wait-time 5
   continuous-upload rotate-remote hourly 1 0
   exit
   ```

## Kibana set up steps

1. In Kibana, navigate to **Management > Integrations**.
2. Search for **Broadcom ProxySG** and click on the integration tile.
3. Click **Add Broadcom ProxySG**.
4. Configure the integration with an **Integration name** and optional **Description**.
5. Configure the desired input types by enabling the toggles and filling in the specific configuration variables:

### Collecting access logs from ProxySG via logging server file
- **Paths** (`paths`): Provide the list of paths to the log files. Default: `['/var/log/proxysg-log.log']`.
- **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
- **Access Log Format** (`config`): Log configuration type for input. Use `main` for standard deployments. Default: `main`.

### Collecting logs from ProxySG via UDP
- **Listen Address** (`udp_host`): The bind address to listen for UDP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
- **Listen Port** (`udp_port`): The UDP port number to listen on. Default: `514`.
- **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
- **Access Log Format** (`config`): Log configuration type for input. Default: `main`.

### Collecting logs from ProxySG via TCP
- **Listen Address** (`tcp_host`): The bind address to listen for TCP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
- **Listen Port** (`tcp_port`): The TCP port number to listen on. Default: `601`.
- **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
- **Access Log Format** (`config`): Log configuration type for input. Default: `main`.

6. Choose the **Existing host policy** or create a **New host policy** for the Elastic Agent.
7. Click **Save and continue**, then **Add Elastic Agent to your hosts** if necessary.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on ProxySG:
- **Generate Web Traffic:** From a workstation configured to use the ProxySG as a proxy, browse to several external websites to generate access log entries.
- **Generate Management Events:** Log out and log back into the ProxySG Management Console to trigger administrative access events.
- **Verify Log Generation:** In the ProxySG console, navigate to **Statistics > Access Logging > Log Tab**, select your log name, and click **Show Log Tail** to ensure logs are being generated locally.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "proxysg.log"`
4. Verify logs appear. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `proxysg.log`)
   - `source.ip` (the client IP address)
   - `destination.ip` (the target server IP address)
   - `event.outcome` (the result of the connection attempt)
   - `message` (the raw log payload)
5. Navigate to **Analytics > Dashboards** and search for "ProxySG" to view the pre-built Overview dashboard.

# Troubleshooting

## Common Configuration Issues

- **Incorrect Log Format**: The integration only supports the `main` format. If `main-w3c` or a custom format is selected in the ProxySG "Logs" configuration, the integration will fail to parse the fields correctly.
- **Policy Not Applied**: Logs will not be generated unless the "Modify Access Logging" action is applied to a rule within the Visual Policy Manager and the policy is successfully installed.
- **Port Conflicts**: Ensure the port configured in the Elastic Agent (e.g., `514` or `601`) is not being used by another process on the Agent host and that the host firewall allows traffic on that port.
- **Upload Client Misconfiguration**: In the ProxySG "Upload Client" tab, ensure the "Primary Server" string includes both the IP and the port (e.g., `10.0.0.1:601`).

## Ingestion Errors

- **Parsing Failures**: Check the `error.message` field in Kibana. If logs show "provided log does not match the expected format," verify that no extra fields have been added to the `main` format in the ProxySG advanced settings.
- **Timestamp Mismatches**: Ensure the ProxySG appliance clock is synchronized via NTP, as timestamp discrepancies can cause logs to appear outside the current time range in Discover.
- **Original Event Missing**: If you need to debug raw log strings, ensure **Preserve original event** is checked in the Kibana integration settings to populate `event.original`.

## Vendor Resources

- [Configure an Access Log - Broadcom](https://techdocs.broadcom.com/us/en/symantec-security-software/web-and-network-security/edge-swg/7-3/getting-started/page-help-administration/page-help-logging/page-help-access-logging-log.html)
- [Configure access logging on ProxySG or ASG to an FTP server - Broadcom](https://knowledge.broadcom.com/external/article/165586/configure-access-logging-on-proxysg-or-a.html)
- [Broadcom ProxySG (Edge SWG) 7.3 Log Formats](https://techdocs.broadcom.com/us/en/symantec-security-software/web-and-network-security/edge-swg/7-3/getting-started/page-help-administration/page-help-logging/log-formats/default-formats.html)

## Documentation sites

- [Broadcom ProxySG Default Log Formats Guide](https://techdocs.broadcom.com/us/en/symantec-security-software/web-and-network-security/edge-swg/7-3/getting-started/page-help-administration/page-help-logging/log-formats/default-formats.html)
- [Elastic Integration Reference for ProxySG](https://techdocs.broadcom.com/us/en/symantec-security-software/web-and-network-security/edge-swg/7-3/getting-started/page-help-administration/page-help-logging/log-formats/default-formats.html)
