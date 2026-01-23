# Service Info

## Common use cases

The Check Point integration for Elastic is designed to provide comprehensive visibility into the security posture and network activity of Check Point Security Gateways and Management Servers. By centralizing logs within the Elastic Stack, organizations can effectively monitor firewall traffic, administrative changes, and system health.
- **Real-time Threat Detection:** Leverage Elastic Security to monitor firewall connection logs (accept, drop, reject) in real-time, allowing security teams to identify and respond to malicious traffic patterns or unauthorized access attempts.
- **Network Traffic Analysis:** Utilize pre-built Kibana dashboards to visualize network traffic distributions, identify top talkers, and analyze VPN activity to optimize network performance and bandwidth allocation.
- **Compliance Auditing and Reporting:** Maintain a long-term, searchable archive of SmartConsole audit logs and firewall events to satisfy regulatory requirements and facilitate forensic investigations during security audits.
- **Infrastructure Health Monitoring:** Collect Gaia OS system logs (e.g., messages, secure, dmesg) to track appliance performance, monitor for hardware failures, and audit administrator SSH sessions.

## Data types collected

This integration collects several categories of logs from Check Point environments. Each data type is mapped to the `firewall` dataset:

- **Check Point firewall logs (syslog over UDP):** Collect Check Point firewall logs using udp input. This stream ingests firewall connection events, VPN logs, and system events sent via the UDP protocol.
- **Check Point firewall logs (syslog over TCP):** Collect Check Point firewall logs using tcp input. This provides reliable delivery of security events, audit logs, and management server activity using the TCP protocol.
- **Check Point firewall logs (log):** Collect Check Point firewall logs using log input. This method ingests plain-text system logs such as Gaia OS logs and Management Server audit files directly from the filesystem.
- **Data Formats:** The integration primarily processes data in **Syslog** format for network-based ingestion and standard text formats for file-based ingestion.
- **Specific Log Paths:** For file collection, common paths include `/var/log/messages` for system events, `/var/log/secure` for authentication, and `$FWDIR/log/cpm.elg` for SmartConsole audit events.

## Compatibility

This integration is compatible with **Check Point Security Gateways and Management Servers** running version **R81.x**. It requires the Elastic Stack version **8.11.0 or higher** (including 9.0.0+) for full functionality of ingest pipelines and dashboards.

## Scaling and Performance

To ensure optimal performance in high-volume environments, consider the following:
- **Transport/Collection Considerations:** While UDP is faster for syslog transmission, TCP is recommended for environments where delivery guarantees are required. TCP ensures no log messages are lost due to network congestion, though it introduces slightly higher overhead. When using the Log Exporter, ensure the network path between the Check Point device and the Elastic Agent supports the chosen protocol and port without fragmentation.
- **Data Volume Management:** Configure the Check Point appliance to forward only necessary events. Use the Log Exporter's filtering capabilities in SmartConsole to exclude high-volume, low-value logs (e.g., frequent "accept" events for trusted internal traffic). Recommend enabling **semi-unified logging** in Check Point to prevent duplicate ingestion of events with identical timestamps and unique IDs.
- **Elastic Agent Scaling:** For high-throughput environments processing thousands of events per second, deploy multiple Elastic Agents behind a network load balancer to distribute the Syslog ingestion load evenly. Ensure the host running the Elastic Agent has sufficient CPU resources to handle the regex-heavy parsing required for complex Check Point log formats.

# Set Up Instructions

## Vendor prerequisites

1. **Administrative Access:** You must have full administrative permissions within Check Point SmartConsole to create Log Exporter objects and install database configurations.
2. **Network Connectivity:** Ensure that firewalls between the Check Point devices and the Elastic Agent allow traffic on the configured syslog port (typically UDP/TCP `514` or a custom port like `9001`).
3. **SSH Access:** Required for configuring logfile collection or performing advanced troubleshooting on the Check Point appliance command line.
4. **Licensing:** Ensure the Logging and Monitoring blade is enabled on your Check Point Management Server or Log Server.
5. **Configuration Knowledge:** Have the IP address of the Elastic Agent host and the intended listening port ready before beginning setup.

## Elastic prerequisites

- **Elastic Agent:** An Elastic Agent must be installed and enrolled in Fleet on a host capable of communicating with the Check Point infrastructure.
- **Stack Version:** Ensure your Elasticsearch and Kibana instances are at version **8.11.0** or higher.
- **Network Access:** The Elastic Agent host must be reachable by the Check Point device's IP addresses on the syslog port defined in the integration settings (default is **9001**).

## Vendor set up steps

### For UDP/TCP (Syslog) Collection:
1. **Log in to Check Point SmartConsole:** Open the SmartConsole application and authenticate with administrative credentials.
2. **Create Log Exporter Object:** Navigate to **Objects > More object types > Server > Log Exporter/SIEM**.
3. **Configure General Settings:** Provide a name (e.g., `Elastic_Agent_Syslog`), enter the IP address of the Elastic Agent in the **Target Server** field, and set the **Target Port** (e.g., `9001`).
4. **Select Protocol:** Choose **TCP** (recommended for reliability) or **UDP** to match your Elastic Agent input configuration.
5. **Set Format:** In the **Data Manipulation** or **Format** section, ensure the format is explicitly set to **Syslog**.
6. **Save Object:** Click **OK** to save the Log Exporter configuration.
7. **Link to Log Server:** Go to **Gateways & Servers**, open the properties of your Management or Log Server, navigate to **Logs > Export**, and add the newly created Log Exporter object.
8. **Install Database:** From the top menu, select **Menu > Install database**, select all relevant objects, and click **Install** to apply the changes.

### For Logfile Collection:
1. **Access the Appliance:** Connect to the Check Point Security Gateway or Management Server via SSH using administrative credentials.
2. **Verify Log Paths:** Confirm the existence and location of the desired logs, such as `/var/log/messages` (system logs) or `$FWDIR/log/cpm.elg` (management audit logs).
3. **Ensure Permissions:** Verify that the user account associated with the Elastic Agent (if running locally on the appliance) or the collection method has read access to these files.
4. **Note Proprietary Logs:** Note that binary security logs like `fw.log` cannot be collected via the logfile method and must be handled via the Syslog/Log Exporter method.

## Kibana set up steps

### Collecting firewall logs from Check Point instances (input: udp)
1. In Kibana, navigate to **Management > Integrations** and search for **Check Point**.
2. Click **Add Check Point**.
3. Under the **Check Point logs** policy template, locate the **Collect Check Point firewall logs (input: udp)** section.
4. Configure the following variables:
   - **Syslog Host**: Enter the IP address or hostname for the Elastic Agent to listen on. Use `0.0.0.0` to listen on all available network interfaces. Default: `localhost`.
   - **Syslog Port**: Enter the UDP port to listen on. Default: `9001`.
   - **Preserve original event**: Check this box to keep a raw copy of the original event in the `event.original` field. Default: `False`.
5. Save the integration to deploy the configuration to the Elastic Agent.

### Collecting firewall logs from Check Point instances (input: tcp)
1. In Kibana, navigate to **Management > Integrations** and search for **Check Point**.
2. Click **Add Check Point**.
3. Under the **Check Point logs** policy template, locate the **Collect Check Point firewall logs (input: tcp)** section.
4. Configure the following variables:
   - **Syslog Host**: Enter the IP address or hostname for the Elastic Agent to listen on. Use `0.0.0.0` to listen on all available network interfaces. Default: `localhost`.
   - **Syslog Port**: Enter the TCP port to listen on. Default: `9001`.
   - **Preserve original event**: Check this box to keep a raw copy of the original event in the `event.original` field. Default: `False`.
5. Save the integration to deploy the configuration to the Elastic Agent.

### Collecting firewall logs from Check Point instances (input: logfile)
1. In Kibana, navigate to **Management > Integrations** and search for **Check Point**.
2. Click **Add Check Point**.
3. Under the **Check Point logs** policy template, locate the **Collect Check Point firewall logs (input: logfile)** section.
4. Configure the following variables:
   - **Paths**: Provide a list of absolute paths to the log files you wish to monitor. Use a new line for each path (e.g., `/var/log/messages`).
   - **Preserve original event**: Check this box to keep a raw copy of the original event in the `event.original` field. Default: `False`.
5. Save the integration to deploy the configuration to the Elastic Agent.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on Check Point:
- **Generate firewall traffic:** From a client protected by the Check Point Gateway, attempt to browse several websites or trigger a specific firewall rule to ensure "Accept" or "Drop" events are generated.
- **Generate audit logs:** Log into the Check Point SmartConsole and modify an object description or create a dummy object to trigger a Management Server audit event.
- **Generate system events:** Log in and out of the Check Point appliance via SSH to generate authentication entries in `/var/log/secure` or `/var/log/messages`.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "checkpoint.firewall"`
4. Verify logs appear. Expand a log entry and confirm these fields:
   - `event.dataset` (should be `checkpoint.firewall`)
   - `source.ip` and/or `destination.ip`
   - `event.action` or `event.outcome`
   - `checkpoint.loguid` (specific to Check Point events)
   - `message` (the raw log payload)
5. Navigate to **Analytics > Dashboards** and search for "Check Point" to confirm data is populating the pre-built visualizations.

# Troubleshooting

## Common Configuration Issues

- **Policy Not Installed**: Changes to the Log Exporter object in SmartConsole do not take effect until a policy installation is successfully completed on the management/log server.
- **Port Conflict**: Ensure that no other service on the Elastic Agent host is using the configured syslog port (e.g., 514 or 9001). Use `netstat -tuln` on the agent host to verify port availability.
- **Network Security Groups**: In cloud environments or segmented networks, verify that Security Groups or ACLs allow traffic from the Check Point Gateway IP to the Elastic Agent IP on the specified protocol and port.
- **Incorrect Log Format**: If logs are reaching Elastic but not parsing, verify that the **Format** in the Check Point Log Exporter object is explicitly set to `Syslog`.

## Ingestion Errors

- **Fingerprint Collisions**: If multiple events arrive with the same `loguid` and timestamp, they may be dropped. Enable "semi-unified logging" in the Check Point Log Exporter settings to ensure unique event identification.
- **Parsing Failures**: Check for the `error.message` field in Kibana Discover. This often indicates that the raw syslog header does not match expected RFC formats.
- **Permission Denied**: For logfile collection, if the Elastic Agent logs show "Permission denied" errors, verify that the agent process has read access to the files in `/var/log/` or `$FWDIR/log/`.
- **Binary Log Collection**: If you see garbage characters or unreadable data, you may be attempting to read binary files (like `fw.log`) directly via the logfile input. Switch to the Syslog/Log Exporter method for these data types.

## Vendor Resources

- [Check Point R81 Log Exporter Configuration Guide](https://sc1.checkpoint.com/documents/R81/WebAdminGuides/EN/CP_R81_LoggingAndMonitoring_AdminGuide/Topics-LMG/Log-Exporter-Configuration-in-SmartConsole.htm?tocpath=Log%20Exporter%7C%5F%5F%5F%5F%5F2)
- [Check Point Log Exporter Command Line Utility (sk122323)](https://support.checkpoint.com/results/sk/sk122323)
- [Log Exporter Appendix - Semi-unified Logging](https://sc1.checkpoint.com/documents/R81/WebAdminGuides/EN/CP_R81_LoggingAndMonitoring_AdminGuide/Topics-LMG/Log-Exporter-Appendix.htm?TocPath=Log%20Exporter%7C_____9)

## Documentation sites

- [Deployment of Log Exporter in SmartConsole](https://sc1.checkpoint.com/documents/Log_Exporter/EN/Content/Topics/Deployment-SmartConsole.htm)
- [Configuring Log Exporter in SmartConsole - R81.20](https://sc1.checkpoint.com/documents/R81.20/WebAdminGuides/EN/CP_R81.20_LoggingAndMonitoring_AdminGuide/Content/Topics-LMG/Log-Exporter-Configuration-in-SmartConsole.htm)
- [Check Point R81 Logging and Monitoring Administration Guide](https://sc1.checkpoint.com/documents/R81/WebAdminGuides/EN/CP_R81_LoggingAndMonitoring_AdminGuide/Topics-LMG/Log-Exporter-Configuration-in-SmartConsole.htm?tocpath=Log%20Exporter%7C%5F%5F%5F%5F%5F2)
