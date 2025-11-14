# Service Info

## Common use cases

- **Network Security Monitoring**: Zeek provides comprehensive, in-depth analysis of network traffic, enabling organizations to detect suspicious activities and potential security threats across their infrastructure.
- **Incident Response and Forensics**: By generating detailed logs of network events, Zeek aids in post-incident analysis, forensic investigations, and security event reconstruction.
- **Threat Hunting**: Security teams utilize Zeek to proactively search for indicators of compromise, anomalous behavior, and advanced persistent threats within network traffic.
- **Protocol Analysis**: Zeek supports analysis of various network protocols, facilitating the identification of protocol anomalies, misuse, and non-standard implementations.
- **Compliance Monitoring**: Organizations use Zeek logs to maintain audit trails and demonstrate compliance with security policies and regulatory requirements.

## Data types collected

Zeek collects comprehensive network traffic logs including:

- **Connection logs** (TCP, UDP, ICMP): Network connection metadata including source/destination IPs, ports, bytes transferred, and connection states
- **DNS logs**: DNS queries and responses with query types, answers, and response codes
- **HTTP/HTTPS logs**: HTTP requests, responses, headers, user agents, and methods
- **SSL/TLS logs**: SSL/TLS handshakes, certificate details, cipher suites, and JA3/JA3S fingerprints
- **File analysis logs**: Metadata and content hashes of files transferred over the network
- **SMTP logs**: Email transaction details including sender, recipient, subject, and message IDs
- **SSH logs**: SSH connection details, authentication attempts, and version information
- **SMB logs**: SMB commands, file operations, and mappings
- **FTP logs**: FTP commands, file transfers, and authentication
- **Authentication logs**: Kerberos, NTLM, and RADIUS authentication events
- **DHCP logs**: DHCP lease information and client details
- **NTP logs**: Network Time Protocol events and synchronization
- **Intel logs**: Threat intelligence indicator matches
- **Notice logs**: Zeek-generated alerts and anomaly notices
- **Weird logs**: Unexpected or unusual network-level activity
- **X.509 logs**: X.509 certificate details and validation information
- **Known hosts/services/certificates**: Tracking of observed network entities
- **Software logs**: Detected software versions on the network
- **RDP, VNC (RFB), SIP, Modbus, DNP3**: Industrial control system and remote access protocols

## Compatibility

- **Zeek Versions**: This integration has been developed against Zeek 2.6.1 and is expected to work with current versions of Zeek.
- **Operating Systems**: Zeek requires a Unix-like platform and currently supports Linux, FreeBSD, and macOS.
- **Elastic Stack**: Requires Kibana version 8.12.0 or higher, or 9.0.0 or higher.
- **Log Format**: Zeek must be configured to output logs in JSON format using the json-logs policy.

## Scaling and Performance

- **High-Performance Networks**: Zeek is designed to operate efficiently in high-speed network environments and can handle large volumes of network traffic.
- **Distributed Deployment**: Zeek supports distributed deployments across multiple sensors to distribute monitoring load across network segments.
- **Resource Requirements**: Proper hardware provisioning (CPU, memory, disk I/O) is essential to maintain optimal performance, especially in large-scale deployments with high traffic volumes.
- **Log Rotation**: Zeek automatically rotates logs, and the integration monitors the "current" directory for active logs.
- **Packet Filtering**: Utilize packet filtering techniques (e.g., BPF filters) to manage data throughput and focus on relevant traffic.

# Set Up Instructions

## Vendor prerequisites

- **JSON Logging Enabled**: Zeek must be configured to output logs in JSON format (see Vendor set up steps).
- **File System Access**: The system must have adequate disk space for log storage, as network traffic analysis can generate substantial log volumes.

## Elastic prerequisites

- **Elastic Agent**: Elastic Agent must be installed and configured to collect logs from the Zeek log directories.

## Vendor set up steps

### Configure Zeek for JSON Output

1. **Locate your Zeek configuration**: Find your `local.zeek` configuration file (typically in `/opt/zeek/share/zeek/site/` or `/usr/local/zeek/share/zeek/site/`).

2. **Enable JSON logging**: Add the following line to your `local.zeek` file to enable JSON log output:
   ```
   @load policy/tuning/json-logs.zeek
   ```

3. **Deploy the configuration**: Run `zeekctl deploy` or restart Zeek to apply the configuration changes.

4. **Verify log format**: Check that Zeek is now generating logs in JSON format in the logs directory (default locations: `/opt/zeek/logs/current/`, `/var/log/bro/current/`, or `/usr/local/var/spool/zeek/`).

### Verify Zeek is Running

1. If using ZeekControl, run `zeekctl status` to verify Zeek is running.
2. Confirm that log files are being generated in the logs directory.
3. Ensure logs are in JSON format by inspecting a log file (e.g., `cat conn.log` should show JSON objects).

## Kibana set up steps

1. **Navigate to Integrations**: In Kibana, go to Management → Integrations.

2. **Add Zeek Integration**: 
   - Search for "Zeek" in the integrations catalog
   - Click on the Zeek integration
   - Click "Add Zeek"

3. **Configure Integration**:
   - **Integration name**: Provide a descriptive name for this integration instance
   - **Base Path**: Specify the path(s) to Zeek log files. Default paths include:
     - `/var/log/bro/current`
     - `/opt/zeek/logs/current`
     - `/usr/local/var/spool/zeek`
   - Multiple paths can be specified if Zeek logs to different locations
   - Adjust the base path to match your Zeek installation's log directory
   - **Select log types**: Enable each log type you wish to capture, and ensure the filename matches the name used in your Zeek instance.


4. **Configure Advanced Options** (optional):
   - **Preserve original event**: Enable to keep a raw copy of the original log in `event.original`
   - **Processors**: Add custom processors if needed for additional data transformation
   - **Tags**: Add custom tags for filtering and organization

5. **Select Agent Policy**: Choose an existing Elastic Agent policy or create a new one.

6. **Save and Deploy**: Save the integration configuration. The Elastic Agent will begin collecting Zeek logs.

# Validation Steps

1. **Generate Network Traffic**: Generate some network activity that Zeek is configured to monitor (e.g., web browsing, DNS queries, file transfers).

2. **Check Zeek Logs**: 
   - Verify that Zeek is generating logs in the configured directory
   - Confirm logs are in JSON format
   - Check timestamps to ensure logs are current

3. **Verify Ingestion in Kibana**:
   - Navigate to Kibana → Discover
   - Select the `logs-zeek.*` index pattern
   - Confirm that recent Zeek events are visible
   - Verify that different log types are being ingested (conn, dns, http, etc.)

4. **Review Dashboards**:
   - Navigate to Kibana → Dashboards
   - Open the "[Zeek] Overview" dashboard or related Zeek dashboards
   - Confirm that visualizations are populated with data
   - Verify that data appears accurate and timely

5. **Test Specific Data Streams**:
   - Check that connection logs show network flows
   - Verify DNS logs contain query information
   - Confirm HTTP logs capture web traffic details

# Troubleshooting

## Common Configuration Issues

### Issue: Service failed to start
- **Solution**: Check Zeek logs for error messages. Common issues include:
  - Insufficient permissions to monitor network interfaces (requires root or CAP_NET_RAW capability)
  - Invalid configuration in `local.zeek` or other policy files
  - Missing dependencies or library conflicts
  - Network interface not available or specified incorrectly

### Issue: No data collected
- **Solution**: 
  - Verify Zeek is running: `zeekctl status`
  - Confirm Zeek is monitoring the correct network interfaces
  - Check that there is active network traffic on the monitored interfaces
  - Verify log files are being created in the expected directory
  - Confirm the Elastic Agent has read permissions to the Zeek log directory
  - Check that the base path in the integration configuration matches the actual log location

### Issue: Logs not in JSON format
- **Solution**:
  - Verify that `@load policy/tuning/json-logs.zeek` is present in `local.zeek`
  - Redeploy the configuration: `zeekctl deploy`
  - Check for conflicting logging policies in your Zeek configuration
  - Examine log files to confirm they contain JSON objects

### Issue: Integration shows "No data collected"
- **Solution**:
  - Verify the base path configuration matches your Zeek log directory
  - Ensure the Elastic Agent has file system permissions to read the logs
  - Check Elastic Agent logs for permission or file access errors
  - Verify that Zeek is actively generating logs (check file modification times)

## Ingestion Errors

### Issue: Parse failures or mapping errors
- **Solution**:
  - Verify Zeek is outputting valid JSON (test with `cat logfile.log | jq .`)
  - Check the Elastic Agent logs for specific parsing error messages
  - Ensure Zeek version compatibility (integration tested with Zeek 2.6.1+)
  - Review ingest pipeline errors in Kibana → Stack Management → Ingest Pipelines

### Issue: Missing fields or incomplete data
- **Solution**:
  - Verify that all required Zeek log files are being collected
  - Check that the Zeek configuration includes all necessary policies for the expected log types
  - Review field mappings to ensure custom Zeek configurations align with expected formats

### Issue: Timestamp parsing errors
- **Solution**:
  - Zeek JSON logs should include timestamps in standard format
  - Verify the `ts` field is present in JSON logs
  - Check for custom timestamp formats that may require additional ingest pipeline configuration

## Vendor Resources

- **Zeek Documentation - Troubleshooting**: https://docs.zeek.org/en/stable/troubleshooting/
- **Zeek Mailing List**: https://community.zeek.org/
- **Zeek Slack Community**: Available through the Zeek community website

# Documentation sites

- **Zeek Official Website**: https://zeek.org/
- **Zeek Documentation**: https://docs.zeek.org/
- **Zeek GitHub Repository**: https://github.com/zeek/zeek
- **Zeek Package Manager**: https://packages.zeek.org/
- **JSON Logs Policy Documentation**: https://docs.zeek.org/en/lts/scripts/policy/tuning/json-logs.zeek.html
- **Zeek Log Formats**: https://docs.zeek.org/en/stable/logs/index.html
- **Zeek Analysis Tools (ZAT)**: https://github.com/SuperCowPowers/zat
- **Elastic Zeek Integration**: https://www.elastic.co/docs/current/integrations/zeek
- **Zeek Community**: https://community.zeek.org/

