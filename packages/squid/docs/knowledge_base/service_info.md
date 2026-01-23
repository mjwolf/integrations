# Service Info

The Squid Proxy integration allows for the seamless ingestion and analysis of access logs from Squid, a popular caching and forwarding HTTP web proxy. This integration provides real-time visibility into web traffic patterns, cache efficiency, and potential security threats originating from or passing through your network proxy layer. By centralizing these logs in the Elastic Stack, administrators can perform deep-dive analysis on user behavior and system performance.

## Common use cases

- **Security Monitoring and Threat Hunting:** Analyze proxy logs to identify connections to known malicious domains, unusual outbound traffic patterns, or unauthorized access attempts by internal users.
- **Bandwidth Management and Performance Tuning:** Monitor cache hit and miss ratios to optimize Squid configuration, ensuring that frequently accessed content is served efficiently and reducing overall external bandwidth consumption.
- **Compliance and Auditing:** Maintain comprehensive records of user web activity to meet regulatory requirements and internal policy audits, providing a searchable history of all proxied requests and responses.
- **Network Troubleshooting:** Quickly diagnose connectivity issues, DNS resolution failures, or application-level errors (4xx/5xx status codes) by inspecting the real-time flow of requests through the proxy.

## Data types collected

This integration can collect the following types of data:

- **Squid logs:** Collect Squid logs using the UDP input. This stream ingests proxy access events over the User Datagram Protocol.
- **Squid logs:** Collect Squid logs using the TCP input. This stream ingests proxy access events over the Transmission Control Protocol for reliable delivery.
- **Squid logs (filestream):** Collect Squid logs using the filestream input. This stream monitors local log files on the host where the Elastic Agent is installed.
- **Proxy Access Logs:** Detailed records of every request processed by the Squid proxy, including client IPs, requested URLs, HTTP methods, and status codes.
- **Data Formats:** The integration is specifically designed to parse the **Native log file** format (also known as the `squid` format).
- **Log Locations:** Logs can be collected from specific file paths such as `/var/log/squid/access.log` or `/var/log/squid-log.log`.

## Compatibility

The **Squid Proxy** integration is compatible with any version of Squid that supports the native `squid` log format and log modules for file, TCP, or UDP output.
- **Kibana Version Requirement:** A Kibana version of `^8.14.1` or `^9.0.0` is required for full compatibility with the integration assets.
- **Elastic Agent:** Compatible with all modern versions of the Elastic Agent that support the `udp`, `tcp`, and `filestream` inputs.
- **Log Format:** Requires the default native Squid log format for successful field parsing.

## Scaling and Performance

To ensure optimal performance in high-volume proxy environments, consider the following:

- **Transport/Collection Considerations:** For high-volume environments, the **Filestream** input is preferred if the Elastic Agent is co-located with Squid, as it handles log rotation and backpressure gracefully. If sending logs over the network, **TCP** is recommended over UDP to ensure delivery guarantees and prevent data loss during network congestion. UDP should only be used in low-latency environments where minor data loss is acceptable.
- **Data Volume Management:** Configure Squid's `access_log` directives to filter out high-volume, low-value events (such as internal health checks or requests for specific static assets) to reduce the ingest load. Regularly monitor and manage log rotation on the Squid server when using the Filestream input to prevent individual log files from becoming excessively large, which can impact Agent performance.
- **Elastic Agent Scaling:** For large-scale proxy farms, deploy one Elastic Agent per Squid instance to distribute the parsing and processing load. If using a centralized syslog collector, ensure the Agent host is provisioned with sufficient CPU and memory to handle the aggregate throughput. Consider placing multiple Agents behind a network load balancer to provide high availability and horizontal scaling for syslog ingestion.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** Root or sudo access on the Squid server is required to modify `squid.conf` and restart services.
- **Network Connectivity:** Ensure firewall rules allow traffic on the configured ports (default `9537`) between the Squid server and the Elastic Agent if using network inputs.
- **Logging Configuration:** Squid must be functional and configured to generate logs in the native `squid` format.
- **Log Permissions:** If using the filestream input, the user running the Elastic Agent must have read permissions for the Squid log directory and files (e.g., `/var/log/squid/`).
- **Disk Space:** Sufficient disk space for local log storage if using the filestream (file-based) collection method.

## Elastic prerequisites

1. **Elastic Agent:** An Elastic Agent must be installed on the Squid host (for filestream) or a central log collector host (for TCP/UDP).
2. **Fleet Management:** The Agent should be enrolled in Fleet and assigned to a policy that includes the Squid Proxy integration.
3. **Kibana Access:** Access to Kibana (8.14.1+) with permissions to add and configure integrations.

## Vendor set up steps

### For UDP/TCP (Network) Collection:
1.  Navigate to the Squid configuration directory (e.g., `/etc/squid/`).
2.  Open the `squid.conf` file in a text editor with administrative privileges.
3.  Add or modify the `access_log` directive to point to your Elastic Agent's listener.
    - For **UDP**: Add `access_log udp://<elastic_agent_ip>:9537 squid`
    - For **TCP**: Add `access_log tcp://<elastic_agent_ip>:9537 squid`
4.  Ensure the line ends with the `squid` keyword to ensure logs are sent in the native format.
5.  Save the file and exit the editor.
6.  Validate the configuration by running `squid -k parse`.
7.  Apply the changes by running `squid -k reconfigure`.

### For Filestream (Local Log File) Collection:
1.  Open the `squid.conf` file (typically located at `/etc/squid/squid.conf`).
2.  Verify the path where Squid is writing its access logs. A standard configuration looks like: `access_log daemon:/var/log/squid/access.log squid`.
3.  Ensure the logging format is set to `squid`.
4.  Verify that the Elastic Agent user (usually `root` or `elastic-agent`) has read permissions for the log directory and the specific log file (e.g., `/var/log/squid/access.log`).
5.  Save any changes and reload the service using `squid -k reconfigure`.
6.  Record the absolute file path for the Kibana configuration step.

## Kibana set up steps

1. In Kibana, navigate to **Integrations** and search for **Squid Proxy**.
2. Click **Add Squid Proxy**.
3. Follow the prompts to add the integration to an existing Elastic Agent policy or create a new one.
4. Configure the desired input types based on your vendor setup by filling in the following fields:

### Collecting syslog from Squid via UDP
Use this input to collect Squid logs using the UDP protocol.
- **UDP host to listen on** (`udp_host`): The interface address the Elastic Agent will use to listen for UDP traffic. Default: `localhost`.
- **UDP port to listen on** (`udp_port`): The port number the Elastic Agent will use to listen for incoming UDP logs. Default: `9537`.
- **Preserve original event** (`preserve_original_event`): If enabled, preserves a raw copy of the original event in the `event.original` field. Default: `False`.

### Collecting syslog from Squid via TCP
Use this input to collect Squid logs using the TCP protocol.
- **TCP host to listen on** (`tcp_host`): The interface address the Elastic Agent will use to listen for TCP connections. Default: `localhost`.
- **TCP port to listen on** (`tcp_port`): The port number the Elastic Agent will use to listen for incoming TCP logs. Default: `9537`.
- **Preserve original event** (`preserve_original_event`): If enabled, preserves a raw copy of the original event in the `event.original` field. Default: `False`.

### Collecting syslog from Squid via filestream
Use this input to collect Squid logs using the filestream input by monitoring local files.
- **Paths** (`paths`): A list of file paths to the Squid access logs. Default: `['/var/log/squid-log.log']`.
- **Preserve original event** (`preserve_original_event`): If enabled, preserves a raw copy of the original event in the `event.original` field. Default: `False`.

5. Click **Save and continue** to deploy the configuration to the Elastic Agent.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on Squid Proxy:
- **Generate Web Traffic:** From a client machine configured to use the proxy, execute a request: `curl -x http://<squid_ip>:3128 http://www.google.com`
- **Authentication/Access Event:** Attempt to access a blocked URL or use invalid proxy credentials to generate `403 Forbidden` or `407 Proxy Authentication Required` events.
- **Service Event:** Restart the Squid service to generate fresh log entries: `sudo systemctl restart squid`

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "squid.log"`
4. Verify logs appear in the results table. Expand a log entry and confirm the presence of these fields:
   - `event.dataset` (should match `squid.log`)
   - `source.ip` (the IP of the client making the request)
   - `event.outcome` (the result of the proxy request)
   - `message` (the raw log payload from Squid)
5. Navigate to **Analytics > Dashboards** and search for "Squid Proxy" to view the pre-built visualizations.

# Troubleshooting

## Common Configuration Issues

- **Port Binding Conflicts**: If the Elastic Agent fails to start the TCP or UDP input, check if another process is already using port `9537` using `netstat -tuln | grep 9537`.
- **File Access Permissions**: If the filestream input is not ingesting data, ensure the Elastic Agent user has execute permissions on the `/var/log/squid/` directory and read permissions on `access.log`.
- **Incorrect Log Format**: If logs are appearing in Kibana but fields are not being parsed, verify that the `access_log` directive in `squid.conf` specifies the `squid` native format and not a custom format.
- **Firewall Blockage**: If using network inputs and no data is arriving, check the host firewall (e.g., `iptables` or `ufw`) on the Elastic Agent machine to ensure port `9537` is open.

## Ingestion Errors

- **Unparsed Fields**: If logs appear in Discover but fields like `source.ip` are missing, check the `error.message` field in the document. This often indicates that the log format does not match the expected native Squid format.
- **Timezone Mismatches**: Squid logs use Unix timestamps. If your logs appear with incorrect timestamps, ensure the system clock on the Squid server is synchronized via NTP and that the Elastic Agent's host time is accurate.
- **Message Truncation**: For very long URLs, UDP packets might be truncated. If you notice incomplete `message` fields, consider switching to the `tcp` input or the `filestream` method.

## Vendor Resources

- [Squid Wiki: Log Modules](https://wiki.squid-cache.org/Features/LogModules#Module:_System_Log)
- [Squid Native Log Format Details](https://wiki.squid-cache.org/Features/LogFormat#squid-native-accesslog-format-in-detail)
- [General Squid Log Information](https://wiki.squid-cache.org/SquidFaq/SquidLogs#accesslog)
- [Squid Configuration Directive: access_log](https://www.squid-cache.org/Doc/config/access_log/)

## Documentation sites

- [Squid Wiki: Log Modules Configuration](https://wiki.squid-cache.org/Features/LogModules#Module:_System_Log)
- [Squid access.log Details](https://wiki.squid-cache.org/SquidFaq/SquidLogs#accesslog)
- [Squid Native Log Format](https://wiki.squid-cache.org/Features/LogFormat#squid)
- Refer to the official vendor website for additional resources.
