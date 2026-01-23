# Service Info

## Common use cases

The Custom TCP Logs integration is designed to ingest raw or formatted log data transmitted over the Transmission Control Protocol (TCP). It allows the Elastic Agent to act as a listener for network-based log streams from various sources, providing a reliable and persistent connection for high-integrity log collection.

*   **Legacy System Integration:** Centralize logs from older applications, mainframe environments, or hardware appliances that do not support modern API-based logging or file-based export but can output text streams over a dedicated TCP socket.
*   **Reliable Syslog Forwarding:** Ingest system logs from Linux servers or network devices using rsyslog or syslog-ng over TCP. This provides significantly better reliability and delivery guarantees compared to standard UDP-based syslog, ensuring that no logs are dropped during periods of network congestion.
*   **Custom Application Logging:** Directly stream application-level events, detailed stack traces, or JSON-formatted business logs from custom-built software directly to the Elastic Stack. This eliminates the need for middleman log collectors or writing sensitive data to local disk files.
*   **Cross-Network Log Aggregation:** Collect data from remote or isolated environments where direct file-read access is not possible. By using TCP, you can tunnel log data through firewalls and corporate proxies to a centralized Elastic Agent.
*   **Encrypted Log Transport:** When configured with SSL/TLS, the integration provides a secure, encrypted pathway for sensitive log data across untrusted or public networks, ensuring confidentiality and integrity of the telemetry.

## Data types collected

This integration can collect the following types of data:

- **Application Logs:** Raw text or structured logs emitted by custom software services, including stack traces, debugging info, and transaction records. These are ingested as individual events based on the configured framing.
- **Syslog Data:** Standardized system messages following RFC3164 (BSD syslog) and RFC5424 (IETF syslog) formats, which include headers such as Priority, Timestamp, Hostname, and App-Name.
- **JSON Structured Logs:** If configured with a custom ingest pipeline, the integration can accept and process structured JSON data sent as single-line TCP messages.
- **Network Device Events:** Audit logs and configuration change events from networking hardware that supports TCP-based log streaming.

This integration supports the following data streams:

- **tcp.generic (logs):** This is the default data stream for all incoming TCP traffic. It captures the raw payload of the TCP message. Each line or framed message received on the TCP socket is indexed as an individual document. The specific dataset name is controlled by the **Dataset name** variable.
- **logs (logs):** If the **Use the "logs" data stream** option is enabled, data is routed to the generic logs data stream. This is intended for use with Elasticsearch 9.2.0+ to simplify data management across different integration types.

## Compatibility

The **Custom TCP Logs** integration is designed for modern Elastic Stack deployments:
- **Kibana Requirements:** This integration requires Kibana version **9.2.0** or later for full UI compatibility and integration management.
- **Elasticsearch Requirements:** Requires Elasticsearch version **9.2.0** or later to utilize advanced features such as the "Write to logs streams" capability.
- **Elastic Agent Compatibility:** The Elastic Agent must be running a version compatible with the Kibana integration manager (typically matching the Kibana version) and must be enrolled via Fleet.

## Scaling and Performance

To ensure optimal performance in high-volume environments, consider the following:

- **Transport/Collection Considerations:** This integration utilizes TCP, a connection-oriented protocol providing guaranteed delivery and packet ordering. While more reliable than UDP, TCP introduces overhead due to acknowledgments and flow control. Ensure the **Framing** method (`delimiter` or `rfc6587`) matches the source capability to prevent event fragmentation. Monitor the **Timeout** (default `300s`) to ensure inactive connections are closed promptly, freeing up system file descriptors and memory.
- **Data Volume Management:** Configure the **Max Message Size** (default `20MiB`) to prevent memory exhaustion from exceptionally large payloads or runaway processes. It is highly recommended to filter data at the source—for example, by configuring rsyslog to only forward specific facilities—to reduce the ingest load. Use the **Keep Null Values** setting (default `false`) to minimize storage requirements by excluding empty fields from the indexed documents.
- **Elastic Agent Scaling:** For high-throughput environments receiving logs from hundreds of sources, a single Elastic Agent may reach CPU or memory limits. Deploy multiple Elastic Agents behind a network load balancer to distribute the TCP connection load evenly. Monitor the Agent's resource usage and increase **Max Connections** if your environment requires a high number of concurrent, long-lived TCP sessions from many endpoints.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have administrative or root-level access to the server hosting the application or log forwarder to modify network configuration and logging settings.
- **Network Connectivity:** The source system must be able to reach the Elastic Agent's host IP address on the configured TCP port (default is `8080`).
- **Firewall Exceptions:** Ensure any local or network firewalls (e.g., iptables, firewalld, or cloud security groups) are configured to allow inbound traffic on the specific **Listen port**.
- **Knowledge of Data Format:** You must know whether your source is sending raw text, RFC3164 syslog, or RFC5424 syslog to properly configure the **Framing** and **Syslog Parsing** options.
- **TLS Certificates (Optional):** If secure transmission is required, you must have valid SSL/TLS certificates and keys ready for both the Elastic Agent and the client application for the SSL Configuration.

## Elastic prerequisites

- **Elastic Agent Enrollment:** The Elastic Agent must be installed and successfully enrolled in a Fleet policy.
- **Kibana Version:** Ensure Kibana is running version **9.2.0** or later to support the integration's configuration schema.
- **Connectivity:** The Elastic Agent must be reachable over the network by the log-producing vendors or servers.

## Vendor set up steps

### For rsyslog (Linux Forwarding):

1. **Access the log source server**: Log in via SSH to the Linux server from which you want to send logs.
2. **Create a new rsyslog configuration file**: Create a dedicated file to maintain a clean configuration.
   ```bash
   sudo nano /etc/rsyslog.d/90-elastic-tcp.conf
   ```
3. **Add the TCP forwarding rule**: Copy and paste the following configuration. This uses the `omfwd` module for reliable forwarding.
   ```text
   # Forward all logs to Elastic Agent over TCP
   action(
       type="omfwd"
       protocol="tcp"
       target="<YOUR_ELASTIC_AGENT_IP>"
       port="<YOUR_TCP_PORT>"
       queue.type="linkedList"
       queue.saveonshutdown="on"
       action.resumeRetryCount="-1"
   )
   ```
4. **Update parameters**: Replace `<YOUR_ELASTIC_AGENT_IP>` with your Elastic Agent's IP and `<YOUR_TCP_PORT>` with your configured port (e.g., `8080`).
5. **Save and exit**: Press `Ctrl+X`, then `Y`, then `Enter`.
6. **Restart rsyslog**: Apply the configuration by restarting the daemon.
   ```bash
   sudo systemctl restart rsyslog
   ```
7. **Verify transmission**: Use the `logger` utility to send a test message.
   ```bash
   logger "Testing TCP connectivity to Elastic Agent"
   ```

## Kibana set up steps

### Custom TCP Logs
1. In Kibana, navigate to **Management > Integrations** and search for **Custom TCP Logs**.
2. Click **Add Custom TCP Logs**.
3. Follow the prompts to add the integration to an Elastic Agent policy.
4. Configure the following **Custom TCP Logs** settings:
    - **Listen Address** (`listen_address`): Bind address for the listener. Use `0.0.0.0` to listen on all interfaces. Default: `localhost`.
    - **Listen port** (`listen_port`): Bind port for the listener. Default: `8080`.
    - **Use the "logs" data stream** (`use_logs_stream`): Enabling this will send all ingested data to the "logs" data stream. Requires Elasticsearch 9.2.0 or later. Default: `false`.
    - **Dataset name** (`data_stream.dataset`): Dataset to write data to. This determines the destination index (e.g., `logs-tcp.generic-default`). Default: `tcp.generic`.
    - **Ingest Pipeline** (`pipeline`): The Ingest Node pipeline ID to be used by the integration for processing at the Elasticsearch level.
    - **Tags** (`tags`): Custom tags to include in the published event for easier filtering.
    - **Syslog Parsing** (`syslog`): Enable the syslog parser to automatically parse RFC3164 and RFC5424 syslog formatted data.
    - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event in the `event.original` field. Default: `false`.
5. For advanced configuration, click **Advanced options** to view additional settings:
    - **Max Message Size** (`max_message_size`): The maximum size of the message received over TCP. Default: `20MiB`.
    - **Framing** (`framing`): Specify the framing used to split incoming events (`delimiter` or `rfc6587`). Default: `delimiter`.
    - **Line Delimiter** (`line_delimiter`): Specify the characters used to split incoming events. Default: `\n`.
    - **Max Connections** (`max_connections`): The maximum number of simultaneous connections to accept.
    - **Timeout** (`timeout`): The duration of inactivity before a connection is closed (e.g., `300s`).
    - **Keep Null Values** (`keep_null`): Publish fields with null values in the output document. Default: `false`.
    - **Processors** (`processors`): Add YAML configuration for Elastic Agent processors to filter or enhance data before ingest.
    - **Syslog Configuration** (`syslog_options`): Provide custom YAML for syslog settings like `field`, `format`, or `timezone`.
    - **SSL Configuration** (`ssl`): Configure certificate and key details for encrypted transport using standard Filebeat SSL options.
    - **Custom configurations** (`custom`): Add additional YAML configuration options directly to the input.
6. Click **Save and continue**.

# Validation Steps

### 1. Trigger Data Flow on [Vendor]:
To verify the integration is listening and receiving data, perform the following actions from the source machine or a machine with network access to the Agent:
- **Send a test string via Netcat:** Run the command `echo "Test log message from source" | nc <AGENT_IP> 8080`. 
- **Generate a Syslog event (if enabled):** Use the logger command on a Linux source: `logger --server <AGENT_IP> --port 8080 --tcp "Test syslog message"`.
- **Verify Port Binding:** On the Elastic Agent host, run `netstat -ano | grep 8080` (Linux/Windows) to confirm the agent is actively listening on the port.

### 2. Check Data in Kibana:
1.  Navigate to **Analytics > Discover**.
2.  Select the `logs-*` data view.
3.  Enter the following KQL filter: `data_stream.dataset : "tcp.generic"` (or your custom dataset name).
4.  Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
    - `event.dataset` (should match `tcp.generic`)
    - `log.source.address` or `source.ip` (showing the sender's IP)
    - `message` (containing the "Test log message" string)
    - `event.ingested` (timestamp of when Elastic received the log)
5.  Navigate to **Analytics > Dashboards** and search for "TCP" to view any custom visualizations created for this data stream.

# Troubleshooting

## Common Configuration Issues

- **Port Conflict**: If the Elastic Agent fails to start the listener, check if the `listen_port` (default 8080) is already being used by another process on the host. Use `netstat -tulpn | grep 8080` or `ss -tulpn` to identify conflicting services.
- **Firewall Blocking**: If the agent is running but no logs are arriving, check the firewall settings. Run `telnet <AGENT_IP> 8080` from the source machine. If it fails to connect, the port is blocked by a firewall or security group.
- **Binding to Localhost**: If `listen_address` is set to `localhost` or `127.0.0.1`, the agent will only accept connections from the local machine. Change this to `0.0.0.0` in the integration settings to accept remote traffic.
- **Framing Mismatch**: If multiple log lines appear as a single document in Kibana, or if logs are being truncated, ensure the `framing` and `line_delimiter` settings in Kibana match the output format of the source application.

## Ingestion Errors

- **Message Truncation**: If logs appear incomplete, the `max_message_size` may be smaller than the incoming log lines. Increase this value in the integration configuration.
- **Framing Mismatch**: If multiple log lines are merged into a single event, or one line is split into many, verify that the `framing` setting matches the source's method (e.g., using `rfc6587` when the source sends length-prefixed messages).
- **Parsing Failures**: If `syslog` parsing is enabled but fields are not populating correctly, verify that the source is strictly adhering to RFC 3164 or RFC 5424. Check the `error.message` field in Discover for specific parsing error details.
- **Dataset Naming**: If data is not being indexed, ensure the `data_stream.dataset` does not contain hyphens, as this is a known limitation.

## Vendor Resources

- Rebex Syslog library for .NET - Rebex.NET
- [Using Rsyslog to send application logs to syslog server - Server Fault](https://serverfault.com/questions/561664/using-rsyslog-to-send-application-logs-to-syslog-server)
- Refer to the official vendor website for additional resources.

## Documentation sites

- [Filebeat Processors Documentation](https://www.elastic.co/docs/reference/beats/filebeat/filtering-enhancing-data)
- [Syslog Configuration Options](https://www.elastic.co/docs/reference/beats/filebeat/syslog)
- [SSL Configuration](https://www.elastic.co/docs/reference/beats/filebeat/configuration-ssl#ssl-common-config)
