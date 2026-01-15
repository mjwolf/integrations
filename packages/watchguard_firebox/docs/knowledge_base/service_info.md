# Service Info

## Common use cases

The WatchGuard Firebox integration for Elastic Agent allows organizations to ingest and normalize security event data from Fireware-powered appliances into the Elastic Stack. This provides a centralized view of network security posture and operational health.
- **Threat Detection and Security Monitoring:** Monitor firewall logs for blocked connections, intrusion prevention system (IPS) alerts, and malware detections to identify potential security breaches in real-time.
- **Network Traffic Analysis:** Analyze traffic patterns and bandwidth usage across different interfaces and policies to optimize network performance and identify unauthorized service usage.
- **Regulatory Compliance and Auditing:** Maintain long-term storage of firewall logs to meet compliance requirements such as PCI DSS, HIPAA, or GDPR, specifically focusing on user authentication and policy changes.
- **Operational Troubleshooting:** Use diagnostic and system event logs to troubleshoot connectivity issues, VPN tunnel failures, and hardware performance bottlenecks across the Firebox infrastructure.

## Data types collected

This integration can collect the following types of data from WatchGuard Firebox appliances via the Syslog protocol:
- **Traffic Logs:** Details on allowed and denied connections, including source/destination IP addresses, ports, protocols, and policy names.
- **Alarm Logs:** Critical security events triggered by security services like IPS, Gateway AntiVirus, and Application Control.
- **Event Logs:** System-level information including administrator logins, configuration changes, and VPN session establishments.
- **Diagnostic Logs:** High-verbosity logs used for deep-dive troubleshooting of specific system processes or protocols.
- **Performance Logs:** Metrics related to CPU usage, memory consumption, and active connection counts (when mapped to specific syslog facilities).
- **Data Format:** Standard Syslog format (RFC 3164/5424) containing the Firebox serial number and timestamp for unique device identification.

## Compatibility

This integration is compatible with **WatchGuard Firebox** appliances running **Fireware OS** version 11.x or higher. It supports all physical Firebox models (T-series, M-series) and virtual/cloud-based Firebox instances that support syslog forwarding via the Fireware Web UI or Policy Manager.

## Scaling and Performance

- **Transport/Collection Considerations:** This integration primarily uses Syslog over UDP or TCP. While UDP (port 514) offers lower overhead and higher throughput for high-volume traffic logs, it does not guarantee delivery. For environments where log integrity is critical, TCP is recommended to ensure every event is acknowledged, though this may introduce slight performance overhead on the Firebox during peak traffic.
- **Data Volume Management:** To prevent overwhelming the Elastic Stack, administrators should use the Firebox "Logging" settings to filter specific event categories. It is highly recommended to enable "Traffic" logging only on relevant policies and to set "Diagnostic" levels to "Information" or "Error" rather than "Debug" unless actively troubleshooting.
- **Elastic Agent Scaling:** A single Elastic Agent can typically handle thousands of events per second (EPS) from multiple Fireboxes. However, in high-throughput environments with extensive "Traffic" logging, consider deploying multiple Elastic Agents behind a load balancer or dedicated high-performance collectors to distribute the processing load and provide high availability.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** Credentials for the Fireware Web UI or WatchGuard Policy Manager with "Device Administrator" permissions.
- **Network Connectivity:** The Firebox must be able to reach the Elastic Agent host over the configured Syslog port (default is UDP 514). Ensure any internal firewalls permit this traffic.
- **Logging Enabled on Policies:** Ensure that logging is enabled on the specific Firewall Policies from which you wish to collect data (e.g., "Send a log message" must be checked in the policy settings).
- **Static IP/DNS:** It is recommended that the Elastic Agent host has a static IP address to prevent log forwarding interruptions.
- **Fireware OS Version:** The appliance should be running a supported version of Fireware (v11.x+) to ensure compatible syslog formatting.

## Elastic prerequisites
- **Elastic Agent:** An active Elastic Agent must be installed and enrolled in a policy via Fleet.
- **Integration Installed:** The WatchGuard Firebox integration must be added to the agent's policy in Kibana.
- **Port Availability:** The host machine running the Elastic Agent must have the configured syslog port (e.g., 514) open and not in use by other services (like rsyslog or syslog-ng).

## Vendor set up steps

### For Syslog Collection via Fireware Web UI:

1.  Log in to the **Fireware Web UI** of your WatchGuard appliance using an administrator account.
2.  Navigate to the **System** menu in the left-hand sidebar and select **Logging**.
3.  In the main pane, select the **Syslog Server** tab.
4.  Check the box labeled **Send log messages to these syslog servers**.
5.  Click the **Add** button to open the Syslog Server configuration dialog.
6.  Configure the destination settings:
    *   **IP Address**: Enter the IP address of the server where your Elastic Agent is currently running.
    *   **Port**: Enter `514` (or the custom port you have specified in your Elastic Agent policy).
    *   **Log Format**: Select **Syslog** from the dropdown menu. (Note: Do not use IBM LEEF unless specifically required for legacy workflows).
7.  In the **Syslog format settings** section, check both **The time stamp** and **The serial number of the device** to ensure accurate event correlation in Kibana.
8.  Under the **Syslog Settings** table, review the facility mappings. The default mappings are typically:
    *   **Alarm**: Local0
    *   **Traffic**: Local1
    *   **Event**: Local2
    *   **Diagnostic**: Local3
    *   **Performance**: Local4
9.  Click **Save** to close the server dialog, then click the **Save** button at the bottom of the Logging page to apply the changes to the appliance.
10. Navigate to your **Firewall Policies** and ensure that the "Send a log message" option is enabled on any policy you wish to monitor.

## Kibana set up steps

1.  Log in to your Kibana instance and navigate to **Management > Integrations**.
2.  Search for **WatchGuard Firebox** and select the integration tile.
3.  Click **Add WatchGuard Firebox** to begin the configuration.
4.  Select the **Agent Policy** where you want to apply this integration.
5.  In the integration configuration, locate the **Syslog** input section:
    *   **Syslog Host**: Set this to `0.0.0.0` to listen on all interfaces, or the specific IP address of the Elastic Agent host.
    *   **Syslog Port**: Ensure this matches the port configured on the Firebox (e.g., `514`).
    *   **Protocol**: Select `udp` (recommended) or `tcp` to match your vendor settings.
6.  Under **Advanced options**, ensure the internal dataset is set to the default unless you are performing custom routing.
7.  Click **Save and continue** to deploy the updated policy to your Elastic Agents.

# Validation Steps

### 1. Trigger Data Flow on WatchGuard Firebox:
- **Generate authentication event:** Log out of the Fireware Web UI and log back in. This will generate an "Event" log entry.
- **Generate configuration event:** Make a minor change to a description field in a policy, save the change, and then revert it. This triggers administrative audit logs.
- **Generate traffic event:** From a machine behind the Firebox, attempt to access a website or perform a ping to an external IP address. Ensure the relevant firewall policy has logging enabled.
- **Generate security event:** If a test environment is available, perform a port scan against an interface protected by IPS to trigger an "Alarm" log.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the following KQL filter: `data_stream.dataset : "watchguard_firebox.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `watchguard_firebox.log`)
   - `source.ip` (the IP address of the traffic initiator)
   - `destination.ip` (the destination IP address of the traffic)
   - `event.action` (e.g., `allow`, `deny`, or `strip`)
   - `observer.serial_number` (should match the serial number of your Firebox)
   - `message` (containing the raw syslog payload from the Firebox)
5. Navigate to **Analytics > Dashboards** and search for "WatchGuard" to view the pre-built "WatchGuard Firebox Overview" dashboard.

# Troubleshooting

## Common Configuration Issues

- **Port Conflict**: If logs are not appearing, check if another service (like the default system syslog) is already listening on UDP port 514. You can check this using `netstat -lnup | grep 514`. If a conflict exists, change the port in both the Firebox UI and the Kibana integration settings to something like `10514`.
- **Missing Serial Number**: If logs are ingested but not correctly parsed or attributed, verify that the **The serial number of the device** checkbox is enabled in the Firebox Syslog Server settings. The parser often relies on this for device identification.
- **Firewall Blockage**: Ensure the host OS running the Elastic Agent (e.g., Linux iptables or Windows Firewall) is configured to allow inbound traffic on the selected Syslog port.
- **Policy Logging Disabled**: If only system events (logins) appear but no traffic data, verify that individual firewall policies have the **Send a log message** option checked in their respective "Logging" settings.

## Ingestion Errors

- **Parsing Failures**: If logs appear in Discover but have a `tags: ["_grokparsefailure"]`, verify that the "Log Format" on the Firebox is set to **Syslog** and NOT **IBM LEEF**. The Elastic integration is optimized for the standard WatchGuard Syslog format.
- **Timestamp Mismatches**: If logs appear with the wrong time, ensure the Firebox has NTP configured correctly and that the **The time stamp** option is enabled in the Syslog Server settings.
- **Identifying Issues**: In Kibana Discover, look for the `error.message` field. This field will contain specific details if the ingest pipeline encountered an issue during the transformation of the raw syslog string into ECS fields.

## Vendor Resources

- [Configure Syslog Server Settings - WatchGuard Technologies](https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/logging/send_logs_to_syslog_c.html)
- [Syslog - Watchguard Firebox Firewall – Huntress Support](https://support.huntress.io/hc/en-us/articles/38858658525843-Syslog-Watchguard-Firebox-Firewall)
- [Configuring WatchGuard firewall - Kaseya](https://help.rocketcyber.kaseya.com/help/Content/network-device-firewall/configure-network-device-watchguard-firewall.html)

# Documentation sites

- [Configure Syslog Server Settings - WatchGuard Technologies](https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/logging/send_logs_to_syslog_c.html)
- [Syslog - Watchguard Firebox Firewall – Huntress Support](https://support.huntress.io/hc/en-us/articles/38858658525843-Syslog-Watchguard-Firebox-Firewall)
- [Configuring WatchGuard firewall - Kaseya](https://help.rocketcyber.kaseya.com/help/Content/network-device-firewall/configure-network-device-watchguard-firewall.html)
