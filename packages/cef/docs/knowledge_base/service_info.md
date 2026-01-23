# Service Info

## Common use cases

The Common Event Format (CEF) integration is a versatile tool designed to ingest and parse standardized log data from a wide array of security appliances and network devices. By utilizing the Elastic Agent's `decode_cef` processor, it transforms raw CEF-encoded strings into structured, ECS-compliant events for deep analysis.
- **Centralized Security Monitoring:** Aggregate logs from diverse vendors like Forcepoint and Check Point into a single dashboard to identify multi-vector threats across the infrastructure.
- **Compliance and Auditing:** Maintain a tamper-evident record of administrative actions and system changes by collecting CEF-formatted audit logs, satisfying requirements for PCI-DSS, HIPAA, or SOC2.
- **Network Traffic Analysis:** Monitor ingress and egress traffic patterns by ingesting firewall logs, allowing security teams to detect anomalous connections or data exfiltration attempts.
- **Threat Hunting:** Utilize normalized fields like `source.ip`, `destination.ip`, and `vulnerability.id` to perform cross-platform hunting for known indicators of compromise (IoCs) across different security layers.

## Data types collected

This integration collects CEF-formatted logs via multiple input methods. Each method corresponds to a specific data stream:

- **CEF logs (logs/logfile):** Collect CEF logs using log input. This is designed for systems that write CEF events directly to local disks or shared volumes.
- **CEF logs (logs/udp):** Collect CEF logs using udp input. This is ideal for high-speed, best-effort delivery from network appliances over the network.
- **CEF logs (logs/tcp):** Collect CEF logs using tcp input. This provides reliable, connection-oriented delivery for critical security events where log integrity is a priority.

General data categories include:
- **Firewall and Network Logs:** Traffic events, including source/destination details, ports, protocols, and interface names.
- **Security Alerts:** Intrusion detection events, malware alerts, and threat prevention logs containing severity levels and risk scores.
- **Audit and Authentication Logs:** Administrative login attempts, configuration changes, and system-level events formatted in CEF.

## Compatibility

This integration is designed to work with any CEF-compliant source. Specific vendors and versions tested include:
- **Forcepoint NGFW Security Management Center (SMC)** version 6.6.1 and newer.
- **Check Point devices** (R81 and newer) using the Log Exporter configured for CEF.
- Any standard CEF-compliant log source using Syslog or file-based delivery.

**Elastic requirements:**
- **Kibana version:** `^8.15.1` or `^9.0.0`
- **Elastic Agent:** Version 8.15.1 or newer is required for optimal compatibility with this package version (2.22.0).

## Scaling and Performance

To ensure optimal performance in high-volume environments, consider the following:

- **Transport/Collection Considerations:** For network-based collection, while UDP offers lower overhead, TCP is recommended for environments where delivery guarantees are required to prevent data loss during network spikes. When using the **logfile** input, ensure the host system has sufficient disk I/O to support the read volume of the log files. High-throughput TCP streams may benefit from increasing the `queue.mem.events` setting in the agent policy.
- **Data Volume Management:** Recommend filtering or limiting data at the source vendor appliance. For example, configure the device to forward only necessary events (e.g., severity level 4 or higher) or use source-side filtering to exclude high-volume/low-value "allow" logs that can overwhelm the ingest pipeline.
- **Elastic Agent Scaling:** For high-throughput environments exceeding 10,000 events per second (EPS), deploy multiple Elastic Agents behind a network load balancer to distribute traffic evenly. This ensures the `decode_cef` processor has enough CPU cycles to parse complex key-value pairs efficiently across multiple Agent instances.

# Set Up Instructions

## Vendor prerequisites

1. **Administrative Access:** Ensure you have root or expert-level CLI access for Check Point devices, or Superuser GUI access for Forcepoint SMC.
2. **Network Connectivity:** Configure firewall rules to allow traffic from the source devices to the Elastic Agent host on the configured ports (typically UDP `9003` or TCP `9004`).
3. **Format Requirements:** Confirm the source device supports log exporting in the **CEF** or **ArcSight** format.
4. **Resource Information:** Identify the IP address of the Elastic Agent host and the desired port number before beginning vendor configuration.
5. **Feature Activation:** For Check Point, ensure the **Log Exporter** feature is installed or enabled on the Management or Log Server.

## Elastic prerequisites

- **Elastic Agent Installation:** An Elastic Agent must be installed on a host reachable by the vendor device and enrolled in a policy via Fleet.
- **Kibana Version:** Ensure your Elastic Stack is running version **8.15.1** or later.
- **Integration Installation:** The CEF integration must be added to the Elastic Agent policy in Kibana.
- **Network Visibility:** The host machine running the Elastic Agent must have its local firewall configured to listen on the selected Syslog ports.

## Vendor set up steps

### For Forcepoint NGFW Security Management Center (SMC):
1. Log in to the Forcepoint Security Management Center (SMC).
2. Ensure that logging is enabled in your policy rules; the **Log Level** must not be set to `None`.
3. Navigate to **Home**, then browse to **Others > Log Server**.
4. Right-click the Log Server you want to configure and select **Properties**.
5. Select the **Log Forwarding** tab and click **Add** to create a new rule (or **Audit Forwarding** for audit logs).
6. Configure the rule:
   - **Target Host**: Select or create a Host element for the Elastic Agent's IP address.
   - **Service**: Select `UDP` or `TCP` (matching your Kibana input configuration).
   - **Port**: Enter the port (e.g., `9003` or `9004`).
   - **Format**: Select **CEF** from the drop-down.
7. Click **OK** to save and activate the rule immediately.

### For Check Point (R81 and newer):
1. Log in to Check Point SmartConsole.
2. Navigate to **Objects > More object types > Server > Log Exporter / SIEM**.
3. Create a new object named `elastic-agent-cef`.
4. On the **General** page:
   - **Export Configuration**: Enabled.
   - **Target Server**: IP or FQDN of the Elastic Agent.
   - **Target Port**: Port number (e.g., `9003`).
   - **Protocol**: UDP or TCP.
5. On the **Data Manipulation** page, select **Common Event Format (CEF)** as the Format.
6. Navigate to **Gateways & Servers**, open the properties for your Management/Log Server, and select **Logs > Export**.
7. Click `+`, add the `elastic-agent-cef` object, and click **OK**.
8. Select **Install database** from the top menu to apply changes.

## Kibana set up steps

1. In Kibana, navigate to **Management > Integrations**.
2. Search for **Common Event Format (CEF)** and select it.
3. Click **Add Common Event Format (CEF)**.
4. Choose the appropriate input type(s) based on how your vendor device is sending logs.
5. Configure the specific variables for each enabled input as detailed below.
6. Select an **Elastic Agent policy** to apply this integration to.
7. Click **Save and continue**.

### Collecting application logs from CEF instances (input: logfile)
Use this input to read CEF logs from a local file on the Elastic Agent host.

*   **Paths** (`paths`): The list of paths to look for log files. Default: `['/var/log/cef.log']`.
*   **Ignore Empty Values** (`ignore_empty_values`): Ignore CEF fields that are empty. The alternative behavior is to treat an empty field as an error. Default: `False`.
*   **Dataset name** (`data_stream.dataset`): Dataset to write data to. Changing the dataset will send the data to a different index. Default: `cef.log`.
*   **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.

### Collecting application logs from CEF instances (input: udp)
Use this input to listen for logs sent over the network via UDP.

*   **Syslog Host** (`syslog_host`): The interface to listen to UDP based syslog traffic. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
*   **Syslog Port** (`syslog_port`): The UDP port to listen for syslog traffic. Default: `9003`.
*   **Dataset name** (`data_stream.dataset`): Dataset to write data to. Changing the dataset will send the data to a different index. Default: `cef.log`.
*   **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
*   **Ignore Empty Values** (`ignore_empty_values`): Ignore CEF fields that are empty. The alternative behavior is to treat an empty field as an error. Default: `False`.

### Collecting application logs from CEF instances (input: tcp)
Use this input to listen for logs sent over the network via TCP.

*   **Syslog Host** (`syslog_host`): The interface to listen to TCP based syslog traffic. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
*   **Syslog Port** (`syslog_port`): The TCP port to listen for syslog traffic. Default: `9004`.
*   **Dataset name** (`data_stream.dataset`): Dataset to write data to. Changing the dataset will send the data to a different index. Default: `cef.log`.
*   **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
*   **Ignore Empty Values** (`ignore_empty_values`): Ignore CEF fields that are empty. The alternative behavior is to treat an empty field as an error. Default: `False`.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on CEF Source:
- **Authentication event:** Log out and log back into the administration interface (e.g., Forcepoint SMC or Check Point Management).
- **Configuration change:** Enter and exit config mode or modify a test network object description.
- **Traffic event:** Generate a network request (e.g., `ping` or `curl`) that matches a logged firewall rule.
- **Manual log generation (logfile only):** If using file input, run: `echo "CEF:0|Elastic|CEF-Test|2.0|100|Test|1|msg=Validation test" >> /var/log/cef.log`

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "cef.log"`
4. Verify logs appear. Expand a log entry and confirm these fields:
   - `event.dataset` (should match `cef.log`)
   - `source.ip` and/or `destination.ip`
   - `event.action` or `event.outcome`
   - `message` (the raw log payload)
5. Navigate to **Analytics > Dashboards** and search for "CEF" to view the overview dashboard.

# Troubleshooting

## Common Configuration Issues

- **Port Conflict**: If the Elastic Agent fails to start the listener, check if another process is already using the configured Syslog port (9003/9004) using `netstat -ano` or `ss -tuln`.
- **Firewall Blockage**: If the vendor device shows logs are being sent but none arrive in Kibana, ensure the host's OS firewall (iptables/firewalld/Windows Firewall) is explicitly allowing traffic on the listener port.
- **Incorrect Protocol**: Ensure the protocol configured on the vendor device (UDP vs TCP) exactly matches the input type selected in the Elastic Agent policy.
- **Policy Deployment Delay**: Changes to the Elastic Agent policy in Fleet can take a few minutes to propagate. Verify the Agent status is "Healthy" and the updated configuration is applied.

## Ingestion Errors

- **Non-Compliant CEF Format:** Some vendors send CEF messages that deviate from the specification (e.g., missing headers). Use the **Pre-Processors** option in the Agent configuration to modify the `message` field before it reaches the `decode_cef` processor.
- **Empty Field Errors:** If logs are dropped or show errors due to empty extensions, enable the **Ignore Empty Values** (`ignore_empty_values`) setting in the integration configuration.
- **Parsing Failures:** Check the `error.message` field in Kibana Discover. If the CEF header is malformed, the `decode_cef` processor will fail. The original message will still be available in `event.original` if **Preserve original event** was enabled.
- **Timestamp Mismatches:** If events appear in the past or future, verify the time synchronization (NTP) on both the source device and the Elastic Agent host.

## Vendor Resources

- [Forcepoint SMC Configuration (KB 15002)](https://support.forcepoint.com/s/article/000015002)
- [Check Point Log Exporter - sk122323](https://support.checkpoint.com/results/sk/sk122323)
- [Check Point Deployment of Log Exporter in CLI](https://sc1.checkpoint.com/documents/Log_Exporter/EN/Content/Topics/Deployment-CLI.htm)
- [Configuring syslog service on Forcepoint devices - ManageEngine](https://www.manageengine.com/log-management/help/data-source-configuration/network-devices/forcepoint.html)

## Documentation sites

- [Check Point Log Exporter CEF Field Mappings](https://community.checkpoint.com:443/t5/Management/Log-Exporter-CEF-Field-Mappings/td-p/41060)
- Forcepoint SMC KB Article 000015002
- Refer to the official vendor website for additional resources.
