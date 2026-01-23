# Service Info

## Common use cases
The Palo Alto Networks (PANW) integration for Elastic transforms raw firewall logs into actionable intelligence, providing comprehensive visibility into network security and operational health.
- **Threat Detection and Hunting:** Utilize Elastic SIEM to analyze Threat, WildFire, and Correlation logs for real-time identification of malicious activity and advanced persistent threats.
- **Network Traffic Analysis:** Leverage intuitive Kibana dashboards to visualize traffic patterns, identify bandwidth bottlenecks, and monitor application-level activity across the network.
- **Security Posture Auditing:** Review Configuration and System logs to track administrative changes, monitor device health, and ensure compliance with internal security policies.
- **Automated Incident Response:** Centralize PANW data to build a foundation for Zero Trust architectures and integrate with SOAR platforms like Cortex XSOAR for automated remediation.

## Data types collected
This integration collects **Palo Alto Networks PAN-OS firewall logs**, which provide detailed security and operational telemetry across several specialized data streams. 

The following data streams are collected:
- **Collect logs via syslog over TCP**: Collecting firewall logs from PAN-OS instances (input: tcp). This stream provides high-reliability ingestion of firewall logs (Traffic, Threat, etc.) using the TCP protocol to ensure delivery.
- **Collect logs via syslog over UDP**: Collecting firewall logs from PAN-OS instances (input: udp). This stream provides high-throughput ingestion of firewall logs using the UDP protocol for high-performance environments.
- **Log files**: Collect logs via log file. This stream enables direct monitoring of log files (input: logfile) stored on the local host where the Elastic Agent is running, providing a flexible collection method for exported logs.

The types of information captured within these streams include:
*   **Firewall Logs:** Includes Traffic, Threat, WildFire Submissions, and URL Filtering logs.
*   **System & Management Logs:** Includes System, Configuration (Config), Authentication, and User-ID logs.
*   **Specialized Network Logs:** Includes GlobalProtect, HIP Match, Decryption, GTP, SCTP, IP-Tag, and Tunnel Inspection logs.
*   **Data Formats:** Primarily Syslog (IETF RFC 5424 or BSD) over TCP/UDP, or direct CSV-based log files.

## Compatibility
This integration requires **Kibana version ^8.11.0 or ^9.0.0**. It is compatible with **Palo Alto Next-Gen Firewall (PAN-OS)** versions **10.2**, **11.1**, and **11.2**.
- **GlobalProtect logs:** Supported from PAN-OS version 9.1.3 and later.
- **User-ID logs:** Supported for PAN-OS version 8.1 and higher.
- **Tunnel Inspection logs:** Supported for version 9.1 and later.

## Scaling and Performance

To ensure optimal performance in high-volume environments, consider the following:
- **Transport/Collection Considerations:** While UDP provides lower overhead for high-throughput syslog transmission, TCP is recommended for environments where delivery guarantees are required. When using TCP, ensure the network infrastructure can handle the connection overhead. If logs are truncated, increase the **Max message size** in the advanced settings to accommodate larger security events.
- **Data Volume Management:** Use Log Forwarding Profiles on the PAN-OS appliance to filter events at the source. Configure the vendor device to forward only necessary events (e.g., severity level "High" or "Critical") for specific log types. Avoiding the ingestion of "Informational" traffic logs can significantly reduce the load on the ingest pipeline and storage.
- **Elastic Agent Scaling:** For high-throughput environments with multiple firewall clusters, deploy multiple Elastic Agents behind a network load balancer to distribute the syslog traffic. Place Agents close to the data source to minimize network latency and ensure the host has sufficient CPU and memory to handle parsing, especially if **Preserve original event** is enabled.

# Set Up Instructions

## Vendor prerequisites
1. **Administrative Access:** Ensure you have Superuser or equivalent permissions on the Palo Alto Networks management interface.
2. **Network Connectivity:** Verify an unrestricted network path exists between the PAN-OS device and the Elastic Agent on the configured Syslog Port (default `9001`).
3. **Log Forwarding Licenses:** Ensure active security subscriptions (e.g., Threat Prevention) are in place to generate relevant security log types.
4. **Time Synchronization:** Both the PAN-OS device and the Elastic Agent host should be synchronized via NTP to ensure correct event timestamping.

## Elastic prerequisites
- **Elastic Agent:** Must be installed and enrolled in Fleet on a host capable of reaching the PAN-OS management or data plane.
- **Connectivity:** The agent must be able to bind to the specified **Syslog Host** address and **Syslog Port** to listen for incoming traffic.
- **Kibana Version:** Ensure your Elastic Stack is running version 8.11.0 or higher.

## Vendor set up steps

### For Syslog (TCP or UDP) Collection:
1. Log in to your PAN-OS web interface.
2. Navigate to **Device > Server Profiles > Syslog** and click **Add**.
3. In the **Syslog Server Profile**, set the **Name** (e.g., `ElasticAgent-Syslog`) and add a server entry with the Elastic Agent's IP address.
4. Set the **Transport** to `TCP` (recommended) or `UDP` and the **Port** to match your integration settings (default is `9001`).
5. Select **Format** as `IETF` (RFC 5424) for TCP or `BSD` for UDP.
6. (Recommended) Go to the **Custom Log Format** tab, select **Config**, and paste the following to ensure correct parsing:
   `1,$receive_time,$serial,$type,$subtype,2561,$time_generated,$host,$vsys,$cmd,$admin,$client,$result,$path,$before-change-detail,$after-change-detail,$seqno,$actionflags,$dg_hier_level_1,$dg_hier_level_2,$dg_hier_level_3,$dg_hier_level_4,$vsys_name,$device_name,$dg_id,$comment,0,$high_res_timestamp`
7. Navigate to **Objects > Log Forwarding**, click **Add**, and create a profile (e.g., `Forward-to-Elastic`). Add match list entries for all required log types (Traffic, Threat, etc.) and link them to the Syslog Server Profile.
8. Navigate to **Policies > Security**, edit your rules, and in the **Actions** tab, select the Log Forwarding Profile created in the previous step.
9. Navigate to **Device > Log Settings** and assign the Syslog Server Profile to **System**, **Config**, and **User-ID** logs.
10. Click **Commit** in the top-right corner to apply the configuration.

### For Log File Collection:
1. Configure the PAN-OS device or an intermediate syslog server to export logs to a local directory on the Elastic Agent host.
2. Ensure the logs are formatted as CSV or standard syslog before being written to the file (e.g., `/var/log/pan-os.log`).
3. Verify that the Elastic Agent service account has read permissions for the log directory and the specific files.
4. Configure the file rotation policy on the source system to prevent disk exhaustion while ensuring the Agent has time to read the files before they are purged.

## Kibana set up steps

1. In Kibana, navigate to **Integrations** and search for **Palo Alto Next-Gen Firewall**.
2. Click **Add Palo Alto Next-Gen Firewall**.
3. Follow the prompts to add the integration to an Elastic Agent policy.
4. Configure the following input sections as needed:

### Collecting firewall logs from PAN-OS instances (input: tcp).
- **Syslog Host** (`syslog_host`): The listen address that will be used to receive syslog messages. Default: `localhost`.
- **Syslog Port** (`syslog_port`): The listen port that will receive syslog messages. Default: `9001`.
- **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
- **Preserve duplicate custom fields** (`preserve_duplicate_custom_fields`): Preserve custom fields for all ECS mappings. Default: `False`.
- **Time Zone Offset** (`tz_offset`): Set the time zone offset so that datetimes are correctly parsed. Accepts "Local", canonical IDs (e.g., "Europe/Amsterdam"), abbreviations (e.g., "EST"), or HH:mm differentials. Default: `Local`.

### Collecting firewall logs from PAN-OS instances (input: udp).
- **Syslog Host** (`syslog_host`): The listen address that will be used to receive syslog messages. Default: `localhost`.
- **Syslog Port** (`syslog_port`): The listen port that will receive syslog messages. Default: `9001`.
- **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
- **Preserve duplicate custom fields** (`preserve_duplicate_custom_fields`): Preserve custom fields for all ECS mappings. Default: `False`.
- **Time Zone Offset** (`tz_offset`): Set the time zone offset so that datetimes are correctly parsed. Default: `Local`.

### Collecting logs via log file.
- **Paths** (`paths`): The list of file paths to monitor. Provide the absolute path to your PAN-OS log files. Default: `[/var/log/pan-os.log]`.
- **Time Zone Offset** (`tz_offset`): By default, datetimes without a time zone will be interpreted as relative to the agent host's time zone. Set this field if ingesting logs from a different time zone. Default: `local`.
- **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event in `event.original`. Default: `False`.
- **Preserve duplicate custom fields** (`preserve_duplicate_custom_fields`): Preserve custom fields for all ECS mappings. Default: `False`.

5. Save the integration. The Elastic Agent will automatically update its configuration.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on Palo Alto Next-Gen Firewall:
- **Authentication event:** Log out of the PAN-OS web interface and log back in to trigger System/Authentication logs.
- **Configuration event:** Enter configuration mode, change a rule description, and click **Commit**.
- **Traffic event:** From a client behind the firewall, browse to several different websites to generate Traffic and URL Filtering logs.
- **Security event:** Trigger a non-malicious "Threat" signature (such as the EICAR test file) to generate Threat logs.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "panw.panos"`
4. Verify logs appear. Expand a log entry and confirm these fields:
   - `event.dataset` (should match `panw.panos`)
   - `source.ip` and/or `destination.ip`
   - `event.action` or `event.outcome`
   - `message` (the raw log payload)
5. Navigate to **Analytics > Dashboards** and search for "PANW" to view the **[Logs PANW] Overview** dashboard.

# Troubleshooting

## Common Configuration Issues
- **Port Conflicts**: Ensure that no other service is using the configured Syslog Port (e.g., 9001) on the Elastic Agent host. Use `netstat -tuln` to verify port availability.
- **Uncommitted Changes**: Configuration changes in PAN-OS do not take effect until the **Commit** operation is successfully completed.
- **Firewall Blocking**: Check local host firewalls (iptables, firewalld, or Windows Firewall) on the Elastic Agent machine to ensure they allow incoming traffic on the specified TCP/UDP port.
- **Incorrect Log Forwarding Profile**: Verify that the Log Forwarding Profile is actually applied to the specific Security Policies that are active for your traffic.

## Ingestion Errors
- **Truncated Events**: If log messages exceed the default size, fields may be missing. Increase the `max_message_size` in the Advanced Options of the integration.
- **Parsing Failures for Config Logs**: If configuration details like `before-change-detail` are not appearing, ensure the custom log format string provided in the setup steps is correctly pasted into the PAN-OS Syslog Server Profile.
- **Timezone Mismatches**: If logs appear with incorrect timestamps, verify the `tz_offset` variable in the integration settings matches the timezone of the PAN-OS device.
- **Framing Issues**: When using TCP, ensure that the PAN-OS device and Elastic Agent are aligned on RFC 6587 (Octet Counting), which is enabled by default in the integration.

## Vendor Resources
- [Configure Syslog Monitoring](https://docs.paloaltonetworks.com/ngfw/administration/monitoring/use-syslog-for-monitoring/configure-syslog-monitoring)
- [Configure Log Forwarding](https://docs.paloaltonetworks.com/ngfw/administration/monitoring/configure-log-forwarding)

## Documentation sites
- [Custom Log Event Format Configuration](https://docs.paloaltonetworks.com/ngfw/administration/monitoring/use-syslog-for-monitoring/syslog-field-descriptions/custom-logevent-format)
- [Config Log Fields Reference](https://docs.paloaltonetworks.com/ngfw/administration/monitoring/use-syslog-for-monitoring/syslog-field-descriptions/config-log-fields)
- [Palo Alto Networks Integration Reference](https://www.elastic.co/docs/reference/integrations/panw)
