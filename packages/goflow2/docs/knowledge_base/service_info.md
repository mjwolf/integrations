# Service Info

## Common use cases

The goflow2 integration for Elastic allows organizations to ingest and analyze high-volume network flow data, including NetFlow v5/v9, IPFIX, and sFlow. By converting these binary protocols into structured JSON, goflow2 enables the Elastic Stack to provide deep visibility into network traffic patterns and security anomalies.
- **Network Traffic Monitoring:** Gain real-time visibility into bandwidth consumption across the infrastructure to identify top talkers and congested links.
- **Security Incident Response:** Detect lateral movement and unauthorized data exfiltration by analyzing source and destination IP patterns and traffic volumes.
- **Capacity Planning:** Utilize historical flow data to trend network growth and make informed decisions regarding hardware upgrades and peering arrangements.
- **Network Troubleshooting:** Quickly isolate connectivity issues by verifying if specific traffic flows are reaching the collector and analyzing the associated metadata.

## Data types collected

This integration can collect the following types of data:
- **Network Flow Logs:** Comprehensive details of network sessions including NetFlow v5, NetFlow v9, IPFIX, and sFlow.
- **Data Formats:** All ingested flow data is converted and output as structured JSON objects for easy parsing.
- **Log File Paths:** Typically collected from user-defined paths such as `/var/log/goflow2.json` or `/opt/goflow2/logs/flow.log`.
- **Flow Metrics:** Includes specific data points such as source/destination IP addresses, source/destination ports, protocol numbers, byte counts, and packet counts.

## Compatibility

The **goflow2** integration is compatible with **goflow2 version 1.2** and higher. It supports network devices capable of exporting standard NetFlow v5/v9, IPFIX, or sFlow protocols. The Elastic Agent must be running version 7.14 or later to support the required custom log collection features.

## Scaling and Performance

- **Transport/Collection Considerations:** Since goflow2 primarily receives data via UDP (standard for NetFlow and sFlow), it is susceptible to packet loss during high-traffic bursts. To mitigate this, ensure the host's UDP buffer sizes are tuned appropriately. When forwarding data from goflow2 to the Elastic Agent via stdout, ensure the local processing overhead is minimal to prevent backpressure.
- **Data Volume Management:** To reduce the load on the Elastic Stack, it is highly recommended to implement sampling at the network device level (e.g., configure sFlow or NetFlow with a sampling rate like 1:1000). Additionally, use goflow2's internal filtering capabilities or use Elastic Agent processors to drop irrelevant flows, such as internal heartbeat traffic or DNS noise, before the data is indexed.
- **Elastic Agent Scaling:** For high-throughput environments processing thousands of flows per second, deploy multiple Elastic Agents behind a load balancer. Ensure each Agent has sufficient CPU resources to handle the high volume of JSON parsing required when goflow2 outputs records to stdout. Resource sizing should start with at least 2 CPUs and 4GB of RAM for dedicated collector nodes.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** Root or sudo permissions on the Linux host where goflow2 will be installed to manage services and log file permissions.
- **Network Connectivity:** Open UDP ports (typically 2055 for NetFlow/IPFIX and 6343 for sFlow) on the host firewall to allow inbound traffic from network devices.
- **Source Configuration:** Access to the management interface of routers or switches to configure flow export targets (pointing to the goflow2 host IP).
- **Storage Requirements:** Sufficient disk space for flow logs, considering that flow data can grow rapidly; implement log rotation (e.g., logrotate) to prevent disk exhaustion.
- **Binary Installation:** Ability to download and execute the goflow2 binary or run the official Docker container.

## Elastic prerequisites

- **Elastic Stack Version:** An active Elastic Stack deployment (version 8.x recommended) is required.
- **Elastic Agent:** The Elastic Agent must be installed on the host where goflow2 is running and successfully enrolled in Fleet.
- **Connectivity:** The host running the Elastic Agent must have outbound connectivity to Fleet Server and Elasticsearch.

## Vendor set up steps

### For goflow2 Binary Deployment:
1. **Download the Binary:** Obtain the latest `goflow2` release for your architecture from the vendor's release page.
2. **Assign Permissions:** Grant execution permissions to the binary using `chmod +x goflow2`.
3. **Identify Listening Ports:** Determine which ports your network devices are sending data to. By default, use 2055 for NetFlow and 6343 for sFlow.
4. **Execute with JSON Output:** Run goflow2 using the `-format=json` flag to ensure the output is compatible with the Elastic integration.
   ```bash
   ./goflow2 -listen 'sflow://:6343,netflow://:2055' -format=json
   ```
5. **Redirect Output (Optional):** If not capturing directly from stdout via a container, redirect the output to a log file that the Elastic Agent can monitor:
   ```bash
   ./goflow2 -format=json > /var/log/goflow2/flows.log 2>&1 &
   ```
6. **Verify Process:** Use `ps aux | grep goflow2` to confirm the collector is running in the background.

### For goflow2 Docker Deployment:
1. **Pull the Image:** Execute `docker pull netsampler/goflow2:latest` to retrieve the most recent image.
2. **Create Network Rules:** Ensure the Docker host allows traffic on UDP ports 2055 and 6343.
3. **Launch the Container:** Run the container with the ports mapped and the format set to JSON.
   ```bash
   sudo docker run -d \
     --name goflow2-collector \
     -p 6343:6343/udp -p 2055:2055/udp \
     netsampler/goflow2:latest \
     -listen 'sflow://:6343,netflow://:2055' -format=json
   ```
4. **Inspect Logs:** Confirm data is being generated in JSON format by checking the container logs:
   ```bash
   docker logs -f goflow2-collector
   ```

## Kibana set up steps

1. **Navigate to Integrations:** Log into Kibana and go to **Management > Integrations**.
2. **Find Goflow2:** Search for "goflow2" in the integration search bar and select it.
3. **Add Integration:** Click on **Add goflow2**.
4. **Configure Input:**
   - Select the **Log file** input type if you redirected goflow2 output to a file, or the **Custom Logs** input if capturing from a specific path.
   - **Log file path:** Enter the path where the goflow2 JSON output is being written (e.g., `/var/log/goflow2/*.log`).
   - If using Docker, ensure the Elastic Agent is configured to collect logs from the Docker container stdout.
5. **Set Dataset Name:** Ensure the dataset name is set to `goflow2.log` to match the expected data stream.
6. **Save and Deploy:** Click **Save and continue**, then select the appropriate **Agent Policy** to deploy the configuration to your Agents.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from your network devices to the Elastic Stack.

### 1. Trigger Data Flow on Network Devices:
- **Generate Web Traffic:** From a client machine whose traffic passes through the sFlow-enabled switch, browse several high-bandwidth websites or perform a large file download to ensure flow samples are captured.
- **Ping Test:** Perform a ping test from the network device itself or a connected host to a known destination (e.g., `ping 8.8.8.8`) to generate simple ICMP flow records.
- **Interface Toggle:** Briefly disable and re-enable a non-critical network interface on the switch to trigger interface state change notifications and associated flow metadata.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view or the specific `goflow2` data view.
3. Enter the following KQL filter: `data_stream.dataset : "goflow2.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `goflow2.log`)
   - `source.ip` and `destination.ip` (representing the flow endpoints)
   - `network.transport` (e.g., `tcp`, `udp`, or `icmp`)
   - `goflow2.sampler_address` (the IP of the network device sending the flows)
   - `message` (containing the raw JSON payload from the goflow2 collector)
5. Navigate to **Analytics > Dashboards** and search for "goflow2" to view pre-built visualizations for traffic volume and top talkers.

# Troubleshooting

## Common Configuration Issues

- **Listener Port Conflicts**: If goflow2 fails to start, verify that no other service is using ports 2055 or 6343. Use `netstat -tulpn | grep 2055` to check for existing listeners.
- **Firewall Blocking**: If logs are not appearing in the file despite traffic being sent, ensure the host firewall allows UDP traffic on the configured ports. Use `sudo ufw allow 2055/udp` or equivalent commands.
- **Permission Denied**: If the Elastic Agent reports errors reading the log file, ensure the agent user (usually `elastic-agent` or `root`) has read permissions for both the directory and the `.json` file.
- **Incorrect Transport Flag**: Ensure `-transport=file` is explicitly set. If omitted, goflow2 may default to stdout, and data will not be written to the log file for the agent to harvest.

## Ingestion Errors

- **JSON Parsing Failures**: If the `error.message` field in Kibana indicates a parsing failure, verify that the `goflow2` command included the `-format json` flag. Without this, the collector might be writing in a format the integration doesn't recognize.
- **Missing Mapping Fields**: If specific fields like `src_addr` or `dst_addr` are missing in Kibana, double-check your `mapping.yaml` file to ensure those fields are explicitly listed under the `formatter.fields` section.
- **Log Rotation Delays**: If data appears to "stop" suddenly, check if log rotation moved the file. Ensure the Elastic Agent is configured to watch the active log file and that rotation happens frequently enough to keep file sizes manageable but not so frequently that the Agent misses data.

## Vendor Resources

- [goflow2 Installation and Setup Guide](https://deepwiki.com/netsampler/goflow2/1.2-installation-and-setup)
- [goflow2 GitHub repository](https://github.com/netsampler/goflow2/releases)

## Documentation sites

- [GoFlow2 | Elastic integrations](https://www.elastic.co/docs/reference/integrations/goflow2)
- [goflow2 GitHub Documentation](https://github.com/netsampler/goflow2)
