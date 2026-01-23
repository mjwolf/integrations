# Service Info

The SonicWall Firewall integration for Elastic allows organizations to centralize, visualize, and analyze log data from SonicOS devices. By ingesting enhanced syslog data, security teams can gain deep visibility into network traffic, security threats, and administrative actions across their infrastructure.

## Common use cases

- **Security Threat Detection:** Monitor logs from security services such as Anti-Virus, Anti-Spyware, Intrusion Prevention (IPS), and Botnet Filtering to identify and respond to active threats in real-time.
- **Network Traffic Auditing:** Track access rule triggers, NAT translations, and website access logs to audit network usage and identify unauthorized communication patterns.
- **User Activity Monitoring:** Analyze authentication events, including RADIUS and SSO agent interactions, to track user logins, logouts, and potential credential abuse.
- **Operational Health and Troubleshooting:** Monitor system status, HA cluster events, and interface changes to maintain firewall uptime and diagnose configuration issues quickly.

## Data types collected

This integration collects multiple categories of security and operational data from SonicWall appliances. The following data streams are supported:

- **Syslog logs (`log`):** Collect logs via syslog. This stream ingests SonicWall logs transmitted over the network via UDP using the Enhanced Syslog format from SonicOS 6.5 and 7.0. It covers firewall events, security services, and system status.
- **Log files (`log`):** Collect logs from file. This stream ingests SonicWall logs from local log files where syslog data may be redirected or stored on the host system.

Specific data types within these streams include:
- **Firewall Logs:** Access rule triggers, application firewall events, and advanced settings logs.
- **Network Logs:** ARP, DNS, ICMP, IP, NAT, TCP, and interface status logs.
- **Security Services Logs:** Anti-Virus, Anti-Spyware, Intrusion Prevention (IPS), Botnet filtering, and Geo-IP filtering logs.
- **User Activity Logs:** Authentication attempts (Local, RADIUS, SSO), administrator logins, and session management.
- **System Events:** Boot sequences, firmware updates, high availability (HA) cluster status, and hardware status.
- **Wireless Logs:** WLAN IDS events and RF monitoring data from SonicWall wireless access points.

## Compatibility

The SonicWall Firewall integration is compatible with the following:
- **Elastic Stack:** Requires Kibana version **8.11.0** or higher (including **9.0.0**).
- **SonicOS Versions:** Compatible with **SonicOS 6.5** (including 6.5.x firmware) and **SonicOS 7.0** (including 7.0.x firmware).
- **Format:** This integration requires the **Enhanced Syslog** format for proper field extraction and parsing.

## Scaling and Performance

To ensure optimal performance in high-volume environments, consider the following:
- **Transport/Collection Considerations:** This integration primarily uses UDP for syslog collection. While UDP is faster for syslog transmission, it does not provide delivery guarantees. Ensure the network path between the SonicWall appliance and the Elastic Agent has sufficient bandwidth and low congestion to minimize packet loss. For log file collection, ensure the disk I/O performance of the host can handle the log writing and reading operations simultaneously.
- **Data Volume Management:** SonicWall firewalls can generate a massive volume of data. Use the "Log Settings" matrix in SonicOS to selectively enable syslog forwarding only for critical categories (severity level "Informational" or lower). Avoid forwarding "Debug" level logs in production environments as they can overwhelm the ingest pipeline and increase CPU load on both the source and the Elastic Agent.
- **Elastic Agent Scaling:** For high-throughput environments, deploy multiple Elastic Agents behind a network load balancer to distribute traffic evenly. Place Agents close to the data source to minimize latency. Ensure the Agent host has sufficient CPU and memory resources to handle the expected log volume and parsing overhead.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** High-level administrative credentials (admin or equivalent) are required to access the SonicOS management interface and modify log settings.
- **Licensing:** Ensure the firewall has active licenses for the security services (IPS, GAV, etc.) you wish to monitor.
- **Connectivity:** Port connectivity must be open between the SonicWall firewall (LAN/DMZ/WAN) and the Elastic Agent host on the configured UDP port (default is `9514`).
- **Format Support:** The firewall must support the **Enhanced Syslog** format.
- **NTP Configuration:** For accurate event correlation, the firewall and the Elastic Agent host should both be synchronized to a reliable NTP time source.

## Elastic prerequisites

- **Elastic Agent:** An Elastic Agent must be installed and successfully enrolled in Fleet using a version compatible with the integration (8.11.0+).
- **Integration Policy:** A SonicWall Firewall integration policy must be created and assigned to the Agent.
- **Network Ingress:** The host running the Elastic Agent must allow inbound UDP traffic on the port specified in the integration settings (e.g., `9514`).

## Vendor set up steps

### For UDP (Syslog) Collection:

1. Log in to your SonicWall management interface.
2. Navigate to the Syslog configuration page:
   - For **SonicOS 7.0**: Go to **Device > Log > Syslog**.
   - For **SonicOS 6.5**: Go to **Manage > Log Settings > Syslog**.
3. Click the **Add** button to define a new Syslog Server.
4. Enter the following configuration details in the Add Syslog Server window:
   - **Name or IP Address**: The IP address of the host running the Elastic Agent.
   - **Port**: The UDP port number configured in the Kibana integration (default: `9514`).
   - **Server Type**: Set this to **Syslog Server**.
   - **Syslog Format**: Select **Enhanced Syslog** (Required).
   - **Syslog ID**: Set a unique identifier (default: `firewall`). This populates the `observer.name` field in Elastic.
5. Click **OK** or **Add** to save the server settings.
6. Configure Log Categories:
   - For **SonicOS 7.0**: Go to **Device > Log > Settings**.
   - For **SonicOS 6.5**: Go to **Manage > Log Settings > Base Setup**.
7. Ensure the **Logging Level** is set to **Informational** (or your desired level) and that the categories you wish to monitor (e.g., Firewall, Users, Network) have the **Syslog** checkbox selected.
8. Click **Accept** or **Save** to apply the logging changes.
9. (Highly Recommended) Configure Time Settings:
   - Navigate to **Device > Settings > Time** (SonicOS 7.0) or **Manage > Appliance > System Time** (SonicOS 6.5).
   - Check the box for **Display UTC in logs (instead of local time)**. This ensures timestamps are parsed correctly without complex offset configurations.
   - Click **Accept**.

### For Logfile Collection:

1. Configure your SonicWall firewall or an intermediate syslog relay to write the Enhanced Syslog output to a flat file on the Elastic Agent host.
2. Ensure the file is saved in a location accessible by the Elastic Agent (e.g., `/var/log/sonicwall-firewall.log`).
3. Verify that the file permissions allow the Elastic Agent user to read the file.

## Kibana set up steps

1. In Kibana, navigate to **Integrations** and search for **SonicWall Firewall**.
2. Click **Add SonicWall Firewall**.
3. Configure the following global variables:
    - **Timezone Offset** (`tz_offset`): By default, datetimes in the logs will be interpreted as relative to the timezone configured in the host where the agent is running. If ingesting logs from a host on a different timezone, use this field to set the timezone offset so that datetimes are correctly parsed. Acceptable timezone formats are: a canonical ID (e.g. `Europe/Amsterdam`), abbreviated (e.g. `EST`) or an HH:mm differential (e.g. `-05:00`) from UTC. Default: `local`.
    - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `false`.
4. Configure the input types based on your data source:

### Collecting logs via syslog
This input collects logs via the UDP syslog protocol.
- **Listen address** (`syslog_host`): Address where the agent will accept syslog messages. Use `0.0.0.0` to receive syslog on all interfaces. Default: `0.0.0.0`.
- **Listen Port** (`syslog_port`): UDP Port where the Agent will receive syslog messages. Default: `9514`.

### Collecting logs from file
This input collects logs from specified log files on the host.
- **Paths** (`paths`): A list of paths to the desired log files. Default: `['/var/log/sonicwall-firewall.log']`.

5. Click **Save and continue** to deploy the integration to an Elastic Agent policy.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on SonicWall Firewall:
- **Authentication event:** Log out of the SonicWall management interface and log back in to generate an administrative login event.
- **Web traffic:** From a workstation behind the firewall, browse to several websites to generate "Website Accessed" logs.
- **Interface event:** Log into the CLI or UI and toggle a test interface or VPN tunnel to generate interface status logs.
- **Configuration change:** Modify a description field in **Device > Settings** and click **Accept** to generate a configuration audit log.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "sonicwall_firewall.log"`
4. Verify logs appear. Expand a log entry and confirm these fields:
   - `event.dataset` (should match `sonicwall_firewall.log`)
   - `source.ip` and/or `destination.ip`
   - `event.action` or `event.outcome`
   - `message` (the raw Enhanced Syslog payload)
5. Navigate to **Analytics > Dashboards** and search for "SonicWall Firewall" to verify visualizations are populating.

# Troubleshooting

## Common Configuration Issues

- **Timezone Mismatch**: If log timestamps in Kibana appear several hours in the past or future, ensure the **Display UTC in logs** setting is enabled on the firewall. If it cannot be enabled, ensure the **Timezone Offset** variable in the Kibana integration configuration matches the firewall's local timezone.
- **Enhanced Syslog Not Enabled**: If the logs appear in Kibana but are not being parsed into ECS fields (e.g., missing `source.ip`), verify that the **Syslog Format** is set to **Enhanced Syslog** on the SonicWall appliance. Standard Syslog format is not fully supported for detailed parsing.
- **UDP Port Blocking**: If no data is appearing in Discover, verify that the host firewall (e.g., iptables or Windows Firewall) on the Elastic Agent server is allowing traffic on UDP port 9514. Use a tool like `tcpdump` or `Wireshark` on the Agent host to verify packets are arriving.
- **Syslog Matrix Not Configured**: If only certain types of logs are missing, check the **Device > Log > Settings** matrix on the firewall to ensure the "Syslog" checkbox is checked for that specific category and that the "Logging Level" is set appropriately.

## Ingestion Errors

- **Parsing Failures**: Check the `error.message` field in Kibana. This often indicates that the syslog message received does not follow the expected Enhanced Syslog format or that a specific field type has changed in a new firmware version.
- **Truncated Logs**: Syslog via UDP has a maximum packet size. If your firewall is generating extremely large log strings (e.g., with long URL strings), they may be truncated, leading to parsing errors.
- **Mapping Issues**: Verify that the `observer.name` field is populated with your **Syslog ID**. If it is missing, the global syslog ID may not be set on the firewall.

## Vendor Resources

- [Configuring Syslog Server with custom event profile on SonicWall](https://www.sonicwall.com/support/knowledge-base/configuring-syslog-server-with-custom-event-profile-on-sonicwall/kA1VN0000000G850AE)
- [Syslog - SonicWall Firewall – Huntress Support](https://support.huntress.io/hc/en-us/articles/33940022013843-Syslog-SonicWall-Firewall)

## Documentation sites

- [SonicOS 6.5.4 Log Events Reference Guide](https://www.sonicwall.com/techdocs/pdf/sonicos-6-5-4-log-events-reference-guide.pdf)
- Refer to the official vendor website for additional resources.
