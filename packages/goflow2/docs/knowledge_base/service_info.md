# Service Info

## Common use cases

The GoFlow2 integration is designed to ingest and normalize sFlow network flow data collected by the GoFlow2 collector, providing deep visibility into network traffic patterns and security events.

*   **Network Traffic Visibility:** Monitor real-time bandwidth usage and traffic distribution across various network interfaces and autonomous systems to identify bottlenecks and optimize network routing.
*   **Security Monitoring and Threat Detection:** Analyze flow records to detect suspicious communication patterns, such as internal scanning, data exfiltration attempts, or communication with known malicious IP addresses.
*   **Capacity Planning and Performance Analysis:** Utilize historical flow data to understand long-term growth trends, allowing network administrators to make informed decisions about infrastructure upgrades.
*   **Compliance and Audit Logging:** Maintain a granular record of all network connections, including source/destination IPs, ports, and protocols, to satisfy regulatory requirements for network activity auditing.

## Data types collected

This integration collects network telemetry via the following data streams:

- **sFlow logs (sflow):** This data stream collects sFlow logs from the GoFlow2 collector. It is a `logs` type data stream that processes JSON-formatted flow records and normalizes them into ECS fields for consistent analysis across the Elastic Stack.
- **Data Format:** The integration expects JSON-formatted logs produced by GoFlow2.
- **Log File Paths:** By default, the integration monitors files located at `/var/log/sflow/goflow2/*.log`.
- **Stream Description:** Collect sFlow logs form GoFlow2.

## Compatibility

The **GoFlow2** integration is compatible with the following versions and environments:

*   **Kibana Versions:** This integration requires Kibana version **^8.11.0** or **^9.0.0**.
*   **Elasticsearch Versions:** This integration requires an Elasticsearch cluster with a **Basic** subscription or higher.
*   **GoFlow2 Version:** The collector must support the `-format json` and `-mapping` flags for proper normalization into the expected Elastic Common Schema (ECS) fields.
*   **Protocol Support:** This integration exclusively supports the **sFlow** protocol. IPFIX and NetFlow normalization are currently not supported through this specific integration policy.

## Scaling and Performance

To ensure optimal performance in high-volume environments, consider the following:

*   **Transport/Collection Considerations:** This integration utilizes the `filestream` input to monitor JSON log files. Ensure the host system has sufficient disk I/O performance to handle concurrent writes from the GoFlow2 collector and reads from the Elastic Agent. Implement robust log rotation (e.g., using `logrotate`) to manage disk space while providing enough retention for the Agent to process all events.
*   **Data Volume Management:** Recommended practice is to filter data at the source. Use the GoFlow2 `mapping.yaml` configuration to export only the fields required for your analysis. Additionally, adjust the **sampling_rate** on your network hardware (routers/switches) to balance the level of visibility against the volume of flow records generated.
*   **Elastic Agent Scaling:** In high-throughput environments processing significant flows per second, deploy the Elastic Agent on a dedicated host or use multiple Agents to distribute the load. For environments with high event rates, ensure the Agent has adequate CPU and memory resources to handle JSON parsing and ingestion overhead.

# Set Up Instructions

## Vendor prerequisites

1. **Administrative Access:** Ensure you have root or sudo permissions on the host where GoFlow2 and the Elastic Agent will be installed to manage services and directories.
2. **Network Connectivity:** The host must be reachable by network devices (switches/routers) via UDP port `6343`, which is the industry default sFlow port.
3. **GoFlow2 Binary:** The `goflow2` binary must be downloaded and installed from the official repository.
4. **Storage Space:** Allocate sufficient disk space in `/var/log/sflow/goflow2/` to store the generated JSON log files; high-volume environments may generate several gigabytes of logs per hour.
5. **Firewall Configuration:** Ensure that local firewalls (e.g., `iptables`, `ufw`, or `firewalld`) allow inbound UDP traffic on port `6343`.

## Elastic prerequisites

1. **Kibana Version:** Ensure Kibana is running version `^8.11.0` or `^9.0.0`.
2. **Elastic Agent:** An Elastic Agent must be installed on the same host as the GoFlow2 collector and enrolled in a policy via Fleet.
3. **Connectivity:** The Elastic Agent must have a stable network connection to Elasticsearch and Kibana (typically over port 443 or 9200/5601).
4. **Integration Installation:** Navigate to **Management > Integrations** and ensure the GoFlow2 integration package is installed before configuring the Agent policy.

## Vendor set up steps

### For Logfile Collection via GoFlow2:

1. **Download the Binary:** Obtain the latest version of GoFlow2 from the GitHub releases page.
   ```shell
   # Example for Linux AMD64
   wget https://github.com/netsampler/goflow2/releases/download/v2.0.4/goflow2_2.0.4_linux_amd64.tar.gz
   tar -xvzf goflow2_2.0.4_linux_amd64.tar.gz
   sudo mv goflow2 /usr/local/bin/goflow2
   sudo chmod +x /usr/local/bin/goflow2
   ```

2. **Prepare Directories:** Create the necessary directories for configuration and logging.
   ```shell
   sudo mkdir -p /etc/goflow2/
   sudo mkdir -p /var/log/sflow/goflow2/
   ```

3. **Configure Field Mapping:** Create the file `/etc/goflow2/mapping.yaml`. This file defines the JSON structure that the Elastic integration parses.
   ```yaml
   formatter:
       fields:
           - type
           - time_flow_start_ns
           - sampler_address
           - sequence_num
           - in_if
           - out_if
           - src_addr
           - dst_addr
           - etype
           - proto
           - src_port
           - dst_port
           - src_vlan
           - dst_vlan
           - sampling_rate
           - bytes
   ```

4. **Initialize the Collector:** Run GoFlow2 manually to verify it starts correctly and begins writing to the log file.
   ```shell
   /usr/local/bin/goflow2 -format json -listen "sflow://:6343" -mapping /etc/goflow2/mapping.yaml -transport.file /var/log/sflow/goflow2/goflow2.log
   ```

5. **Establish as a Persistent Service:** Create a systemd unit file at `/etc/systemd/system/goflow2.service` to ensure the collector runs on boot.
   ```ini
   [Unit]
   Description=GoFlow2 sFlow Collector
   After=network.target

   [Service]
   Type=simple
   User=root
   ExecStart=/usr/local/bin/goflow2 -format json -listen "sflow://:6343" -mapping /etc/goflow2/mapping.yaml -transport.file /var/log/sflow/goflow2/goflow2.log
   Restart=always

   [Install]
   WantedBy=multi-user.target
   ```

6. **Enable and Start Service:**
   ```shell
   sudo systemctl daemon-reload
   sudo systemctl enable goflow2.service
   sudo systemctl start goflow2.service
   ```

## Kibana set up steps

### Collecting logs via log file
1. In Kibana, navigate to **Management > Integrations** and search for **GoFlow2**.
2. Click **Add GoFlow2 logs**.
3. Follow the prompts to add the integration to an existing Elastic Agent policy or create a new one.
4. Configure the following variables under the **Collect logs via log file** input:
    - **Paths** (`paths`): The list of paths to the GoFlow2 JSON log files where the collector is writing data. Default: `['/var/log/sflow/goflow2/*.log']`.
    - **Preserve original event** (`preserve_original_event`): If enabled, this preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
5. Review the **sFlow logs** stream settings to ensure it is configured for the `sflow` dataset.
6. Click **Save and continue** to deploy the configuration to your Elastic Agent.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on GoFlow2:
- **Initiate Network Traffic:** Generate traffic from a network device (e.g., a switch or router) configured to send sFlow samples to the GoFlow2 collector on port **UDP 6343**.
- **Simulate Activity:** If hardware is unavailable, use a tool like `sflowtool` to replay known sFlow samples toward the GoFlow2 listener address.
- **Verify Log Write:** Confirm the collector is successfully writing JSON records to disk: `tail -f /var/log/sflow/goflow2/goflow2.log`.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "goflow2.sflow"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `goflow2.sflow`)
   - `source.ip` (the source of the flow)
   - `destination.ip` (the destination of the flow)
   - `network.transport` (the protocol used)
   - `message` (the raw JSON flow record)
5. Navigate to **Analytics > Dashboards** and search for "GoFlow2" to confirm that visualizations are populating with data.

# Troubleshooting

## Common Configuration Issues

- **Service Start Failure**: If the `goflow2` service fails to start, check `journalctl -u goflow2.service`. This is often caused by the binary not being located at `/usr/bin/goflow2` or port 6343 being occupied by another process.
- **Permission Denied for Log Directory**: If the log file is not being created, ensure the user specified in the systemd service (e.g., `nobody`) has write permissions to `/var/log/sflow/goflow2/`.
- **Elastic Agent Permission Issues**: If logs are present on disk but not appearing in Kibana, ensure the Elastic Agent user has read permissions for the log files. You may need to add the Agent user to the group that owns the logs.
- **Port 6343 Connectivity**: If GoFlow2 is running but no logs are generated, verify that firewall rules (e.g., `iptables` or `ufw`) allow UDP traffic on port 6343 from your network devices.

## Ingestion Errors

- **JSON Parsing Failures**: If the `error.message` field in Discover indicates parsing issues, verify that the `goflow2` command includes the `-format json` flag and that the `/etc/goflow2/mapping.yaml` file is correctly formatted.
- **Mapping Mismatch**: If specific fields like `src_addr` are missing from the logs, confirm that the `mapping.yaml` file explicitly lists these fields under the `formatter.fields` section.
- **Empty Logs**: If GoFlow2 is receiving data but the logs are empty, check if the mapping file path in the `goflow2` command matches the actual location on disk.

## Vendor Resources

- [GoFlow2 GitHub Repository](https://github.com/netsampler/goflow2)
- [netsampler/goflow2 Releases](https://github.com/netsampler/goflow2/releases)
- [goflow2 Go Package Documentation](https://pkg.go.dev/github.com/netsampler/goflow2/v2)

## Documentation sites

- [GoFlow2 | Elastic integrations](https://www.elastic.co/docs/reference/integrations/goflow2)
- [GoFlow2 GitHub Repository](https://github.com/netsampler/goflow2)
