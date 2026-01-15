# Service Info

## Common use cases

The Zeek integration is designed to ingest and normalize network traffic data captured by the Zeek (formerly Bro) network security monitor. By converting Zeek's rich, protocol-aware logs into the Elastic Common Schema (ECS), security analysts can gain deep visibility into network activity without the overhead of full packet capture.
- **Network Threat Hunting:** Use detailed connection, DNS, and HTTP logs to identify lateral movement, data exfiltration, or communication with known malicious command-and-control (C2) infrastructure.
- **Incident Response and Forensics:** Pivot from high-level alerts to specific Zeek session IDs to reconstruct the sequence of events during a security breach, including file hashes and SSL certificate details.
- **Protocol Compliance and Auditing:** Monitor the use of insecure protocols (like Telnet or expired TLS versions) across the environment to ensure compliance with internal security policies.
- **Performance Monitoring:** Analyze network latency and connection durations through structured connection logs to identify bottlenecks in critical infrastructure.

## Data types collected

This integration can collect the following types of data from a Zeek deployment:
- **Network Connection Logs:** Comprehensive metadata for every TCP/UDP/ICMP session (e.g., `conn.log`).
- **Protocol-Specific Logs:** Detailed activity for application-layer protocols including DNS, HTTP, SSL/TLS, SMTP, FTP, and SSH (e.g., `dns.log`, `http.log`, `ssl.log`).
- **File Analysis Logs:** Metadata for files transferred over the network, including MD5/SHA1 hashes and file types (e.g., `files.log`).
- **Security Event Logs:** Detection of "weird" network behavior and potential signatures of interest (e.g., `weird.log`, `notice.log`).
- **Data Format:** All logs must be converted to **JSON** format for optimal ingestion and parsing by the Elastic Agent.
- **Typical File Paths:** Logs are generally collected from the Zeek log directory, such as `/opt/zeek/logs/current/*.log` or `/usr/local/zeek/logs/current/*.log`.

## Compatibility

The Zeek integration is compatible with **Zeek** version 3.0 and higher, including the latest LTS releases. It is specifically designed to work with the **Corelight JSON Streaming Logs** package for enhanced metadata. Support is also noted for **Security Onion 2.4** and later environments utilizing Zeek for network monitoring.

## Scaling and Performance

- **Transport/Collection Considerations:** Since Zeek writes logs directly to the local file system, the Elastic Agent uses the `filestream` input to monitor these files. To ensure high performance, users should use high-speed storage (SSDs) for the Zeek log directory and monitor disk I/O, as Zeek can generate several gigabytes of logs per hour on high-throughput links.
- **Data Volume Management:** Zeek can be extremely verbose. To manage data volume, users should filter unnecessary logs at the source by modifying `local.zeek`. It is recommended to only enable the log streams required for your specific security use cases and to use the `exclude_files` pattern in the Elastic Agent configuration to ignore high-volume, low-value files like `capture_loss.log`.
- **Elastic Agent Scaling:** For high-volume environments (10 Gbps+ traffic), it is recommended to deploy the Elastic Agent on the same physical or virtual machine where Zeek is running to minimize network overhead. If the CPU load of the Agent becomes a bottleneck, consider increasing the number of `harvester` workers or deploying multiple Zeek nodes in a cluster configuration, each with its own local Elastic Agent.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** Sudo or root-level access to the Linux server where Zeek is installed to modify configuration files and restart services.
- **Zeek Installation:** A working installation of Zeek (v3.0+) managed via `zeekctl` or as a standalone process.
- **JSON Support:** Zeek must be configured to output logs in JSON format rather than the default tab-separated value (TSV) format.
- **Storage Space:** Sufficient disk space to store local JSON logs before they are harvested by the Elastic Agent.
- **Network Configuration:** Ensure the Zeek instance is correctly receiving traffic from a SPAN port, TAP, or packet broker.

## Elastic prerequisites

- **Elastic Agent:** A running Elastic Agent enrolled in Fleet or configured as a standalone agent on the Zeek host.
- **Network Connectivity:** The host must have outbound connectivity to the Elastic Stack (Elasticsearch and Kibana) on port 443 or 9200.
- **Integration Package:** The Zeek integration must be installed in Kibana via the Integrations app.

## Vendor set up steps

### For JSON Syslog Collection:

1. **Stop the Zeek Service:** Before making changes to the script configurations, stop the running Zeek instance to prevent configuration conflicts.
   ```bash
   zeekctl stop
   ```

2. **Install the Corelight JSON Streaming Logs Package:** Use `zkg` to install the package that provides the `_path` field metadata required for the Elastic Agent to distinguish between different Zeek log types.
   ```bash
   zkg install corelight/json-streaming-logs
   ```

3. **Locate the Configuration File:** Identify your `local.zeek` file. This is typically found at `/opt/zeek/share/zeek/site/local.zeek` or `/usr/local/zeek/share/zeek/site/local.zeek`.

4. **Modify local.zeek:** Open the file in a text editor (e.g., `vi` or `nano`) and append the configuration for JSON output and syslog forwarding.
   ```bro
   # Load the necessary packages
   @load corelight/json-streaming-logs
   @load zeek/telemetry/syslog

   # Enable JSON logging format
   redef LogAscii::use_json = T;

   # Configure the syslog writer
   redef Telemetry::syslog_host = "ELASTIC_AGENT_IP";
   redef Telemetry::syslog_port = 1514;
   redef Telemetry::syslog_protocol = "tcp";

   # Set the default writer to Syslog
   redef Log::default_writer = Log::WRITER_SYSLOG;
   ```
   *Note: Replace `ELASTIC_AGENT_IP` with the IP of your Elastic Agent host.*

5. **Deploy the Changes:** Use `zeekctl` to check the configuration and deploy the updated scripts to the running environment.
   ```bash
   zeekctl deploy
   ```

6. **Verify Zeek Status:** Ensure all workers are running correctly and that no errors are reported in the initialization phase.
   ```bash
   zeekctl status
   ```

## Kibana set up steps

1.  Log in to Kibana and navigate to **Management > Integrations**.
2.  Search for **Zeek** and select the integration.
3.  Click **Add Zeek** to begin the configuration.
4.  Select the desired **Data Streams** (e.g., Connection, DNS, HTTP, SSL). It is recommended to keep the defaults enabled for full visibility.
5.  For each enabled data stream, configure the **Paths** field. This should point to the location of your JSON logs.
    *   Example path for Connection logs: `/opt/zeek/logs/current/conn.log`
    *   Using wildcards: `/opt/zeek/logs/current/*.log`
6.  Under the **Advanced options** for each stream, ensure that "Ignore older" is set appropriately (e.g., `72h`) to avoid ingesting historical data if not desired.
7.  Choose the **Existing policy** where your Zeek host's Elastic Agent is enrolled.
8.  Click **Save and continue**, then **Rollout changes** to deploy the configuration to the Agent.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from Zeek to the Elastic Stack.

### 1. Trigger Data Flow on Zeek:
- **Generate HTTP Traffic:** From a machine on the monitored network, browse to a non-encrypted website (e.g., `http://testphp.vulnweb.com`) to generate `http.log` entries.
- **Generate DNS Queries:** Run a command like `nslookup elastic.co` or `dig google.com` from a monitored host to create `dns.log` events.
- **Establish a Connection:** Initiate an SSH session or a simple `curl` request to any external IP to trigger a `conn.log` entry.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the following KQL filter: `data_stream.dataset : "zeek.*"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (e.g., `zeek.connection`, `zeek.dns`, or `zeek.http`)
   - `source.ip` and `destination.ip` (should contain the network addresses of the test traffic)
   - `zeek.session_id` (the unique identifier Zeek assigns to the flow)
   - `network.transport` (e.g., `tcp` or `udp`)
   - `message` (containing the raw JSON log payload from Zeek)
5. Navigate to **Analytics > Dashboards** and search for **[Logs Zeek] Overview** to view the pre-built dashboards and confirm visualizations are populating.

# Troubleshooting

## Common Configuration Issues

- **Zeek Package Missing**: If logs are appearing but not being parsed correctly, ensure `zkg install corelight/json-streaming-logs` was executed. Without this package, the `_path` field is missing, and the Elastic Agent cannot route the log to the correct dataset pipeline.
- **Port Bind Conflict**: If the Elastic Agent fails to start the syslog listener, check if another service (like rsyslog or another Agent input) is already using port 1514. Use `netstat -tulpn | grep 1514` to identify conflicting services.
- **Firewall Blocking**: If Zeek shows no errors in `reporter.log` but no data reaches Kibana, verify that the firewall on the Elastic Agent host allows incoming traffic on the configured port/protocol (e.g., `ufw allow 1514/tcp`).
- **Incorrect Protocol**: Ensure the protocol defined in `local.zeek` (`Telemetry::syslog_protocol = "tcp"`) matches the protocol selected in the Kibana integration settings. A mismatch will prevent the connection from establishing.

## Ingestion Errors

- **Parsing Failures**: If the `error.message` field in Kibana mentions parsing errors, verify that Zeek is not outputting any non-JSON content (like headers) into the log files. Setting `use_json = T` usually disables headers automatically.
- **Field Type Mismatch**: If some fields are not appearing in Discover, check for mapping conflicts in Elasticsearch. This can happen if a field was previously indexed as a different data type. Check the `logs-zeek.*` index template.
- **Missing Integration Fields**: If specific Zeek metadata is missing, ensure the Zeek scripts responsible for that protocol (e.g., `base/protocols/http`) are loaded in your `local.zeek` file.

## Vendor Resources

- [Zeek Log Formats and Inspection](https://docs.zeek.org/en/master/log-formats.html)
- [Ingest Zeek Logs - Sumo Logic Docs](https://www.sumologic.com/help/docs/cse/sensors/ingest-zeek-logs/)
- [Zeek — Security Onion Documentation](https://docs.securityonion.net/en/2.4/zeek.html)

## Documentation sites

- [Zeek Log Formats and Inspection](https://docs.zeek.org/en/master/log-formats.html)
- [Ingest Zeek Logs - Sumo Logic Docs](https://www.sumologic.com/help/docs/cse/sensors/ingest-zeek-logs/)
- [Zeek — Security Onion Documentation](https://docs.securityonion.net/en/2.4/zeek.html)
