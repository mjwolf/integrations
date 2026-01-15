# Service Info

The Cisco Aironet integration allows for the seamless ingestion of log data from Cisco Aironet Access Points, which are typically managed via a Cisco Wireless LAN Controller (WLC). By centralizing these logs within the Elastic Stack, administrators gain deep visibility into wireless network performance, security events, and client connectivity patterns across the enterprise.

## Common use cases

The Cisco Aironet integration for Elastic Agent allows you to collect and analyze system logs from Cisco Aironet wireless access points (APs) and controllers running Cisco IOS. This integration centralizes wireless network data to help network administrators maintain uptime and security.
- **Client Connectivity Monitoring:** Track when wireless clients associate, disassociate, or roam between access points to identify coverage gaps or client-side connection issues.
- **Security Auditing and Access Control:** Monitor for failed authentication attempts, unauthorized access configurations, and potential rogue access point detections within the wireless environment.
- **System Health and Performance:** Capture hardware alerts, interface status changes, and software exceptions to proactively address performance bottlenecks or equipment failures.
- **Troubleshooting and Root Cause Analysis:** Utilize detailed system messages and error codes (mnemonics) to correlate wireless events with broader network issues in the Elastic Stack.

## Data types collected

This integration can collect the following types of data:
- **System Logs:** Operational status messages, system initialization events, and hardware-specific alerts formatted as standard Cisco Syslog.
- **Security Logs:** Authentication success/failure messages, authorization events, and encryption-related alerts (e.g., WPA/WPA2/WPA3 handshakes).
- **Client Connectivity Logs:** Detailed information regarding client associations, disassociations, and roaming events between different access points.
- **Configuration Logs:** Audit trails of changes made via the Command Line Interface (CLI) or the Wireless LAN Controller (WLC) management interface.
- **Radio and Interface Logs:** Status updates for 2.4GHz and 5GHz radio modules, including interference detections and channel changes.
- **Data Formats:** Standard Syslog (RFC 3164/5424) and Cisco-specific log formats (e.g., `%FACILITY-SEVERITY-MNEMONIC: Message`).

## Compatibility

The integration is compatible with **Cisco Wireless LAN Controllers (WLC)** managing **Cisco Aironet** Series Access Points. It is generally supported for WLC software versions 7.x, 8.x, and the newer Cisco IOS-XE based controllers (such as the 9800 series), provided they support standard Syslog forwarding.

## Scaling and Performance

- **Transport/Collection Considerations:** For Cisco Aironet integrations, UDP is the most common transport protocol for syslog. While UDP offers lower overhead on the access point, it does not guarantee delivery. In high-latency or congested wireless backhaul environments, consider using TCP (if supported by the specific IOS version) to ensure log reliability, though this may increase the load on the AP's control plane.
- **Data Volume Management:** Cisco devices can generate a significant volume of logs, especially when debugging is enabled. It is highly recommended to set the `logging trap` level to `informational` (level 6) or `notifications` (level 5) for production environments. Avoid using `debugging` (level 7) unless actively troubleshooting, as it can overwhelm the Elastic Agent and negatively impact the performance of the access point.
- **Elastic Agent Scaling:** A single Elastic Agent can typically handle syslog traffic from hundreds of Cisco Aironet access points. However, in large campus deployments, it is recommended to deploy multiple Elastic Agents behind a load balancer to distribute the incoming UDP/TCP traffic and provide high availability. Ensure the host running the Agent has sufficient CPU to process the regex-heavy parsing required for Cisco mnemonics.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** High-level administrative credentials (read/write) for the Cisco WLC Web Interface or CLI are required.
- **Network Connectivity:** The Cisco WLC must have network reachability to the Elastic Agent's IP address on the configured Syslog port (default is UDP 514).
- **Firewall Rules:** Ensure that intermediate firewalls allow traffic from the WLC management IP to the Elastic Agent on the chosen Syslog port.
- **DNS Configuration:** While not strictly required, ensuring the WLC can resolve the Elastic Agent's hostname can simplify configuration in dynamic environments.
- **Logging License:** Ensure the WLC has the appropriate features enabled to export logs to external Syslog servers.

## Elastic prerequisites

- **Elastic Agent:** An Elastic Agent must be installed and enrolled in a policy via Fleet.
- **Integration Policy:** The Cisco Aironet integration must be added to the agent policy.
- **Connectivity:** The agent host must have its local firewall configured to allow inbound traffic on the designated syslog port (e.g., port 514/UDP).

## Vendor set up steps

### Method 1: For Aironet APs in Lightweight Mode (Managed by a WLC)

In this mode, syslog is configured centrally on the Wireless LAN Controller (WLC), which forwards logs for itself and connected APs.

1.  Log in to the Cisco WLC web interface using administrative credentials.
2.  Navigate to **Management** > **Logs** > **Config**.
3.  In the **Syslog Server IP Address** field, enter the IP address of the Elastic Agent.
4.  Click the **Add** button. You can add up to three syslog servers for redundancy.
5.  Set the **Syslog Level** to `Informational` (Level 6). This is the recommended setting for comprehensive monitoring without excessive volume.
6.  Set the **Syslog Facility** to a value that matches your Elastic Agent input configuration (standard defaults are `Syslog` or `Local7`).
7.  Click **Apply** to save the configuration.
8.  To configure AP-specific logging, log in to the WLC CLI via SSH and execute: `config ap syslog host global <elastic_agent_ip>`.
9.  Set the global AP logging level: `config ap logging syslog level informational`.

### Method 2: For Aironet APs in Standalone (Autonomous) Mode

Autonomous APs must be configured individually via the Cisco IOS command-line interface.

1.  Log in to the Access Point CLI via SSH or console and enter privileged EXEC mode: `enable`.
2.  Enter global configuration mode: `configure terminal`.
3.  Enable the logging service by entering: `logging on`.
4.  Specify the Elastic Agent as the syslog host: `logging host <elastic_agent_ip>`.
5.  Set the logging severity to informational: `logging trap informational`.
6.  (Optional) Specify a source interface to ensure consistent IP addressing in logs: `logging source-interface <interface_name>` (e.g., `logging source-interface Vlan1`).
7.  Enable timestamps for accurate event correlation: `service timestamps log datetime msec show-timezone`.
8.  Exit configuration mode and save the settings: `end` followed by `write memory`.

## Kibana set up steps

1. Log in to Kibana and navigate to **Management > Integrations**.
2. In the search bar, type **Cisco Aironet** and select the integration.
3. Click **Add Cisco Aironet**.
4. Configure the integration settings:
   - **Integration Name:** Provide a descriptive name (e.g., `cisco-aironet-wlc-01`).
   - **Syslog Host:** Set this to `0.0.0.0` to listen on all interfaces, or the specific IP of the Elastic Agent host.
   - **Syslog Port:** Set this to `514` (the default) or your preferred custom port.
   - **Protocol:** Select `udp` or `tcp` to match your WLC configuration.
5. Expand the **Advanced options** if you need to modify internal field mappings or add custom tags to the data.
6. Click **Save and continue**.
7. Select the **Agent Policy** that corresponds to the Elastic Agent you previously installed.
8. Click **Save and deploy** to push the configuration to the Agent.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from the Cisco WLC to the Elastic Stack.

### 1. Trigger Data Flow on Cisco WLC:
- **Generate Configuration Event:** Enter and exit configuration mode on the WLC CLI, or change a minor setting in the GUI (such as an AP description) and click "Apply".
- **Trigger Administrative Event:** Log out of the WLC administration interface and log back in to generate an authentication/session event.
- **Generate Client Event:** Disconnect a wireless client from an Aironet AP and reconnect it to trigger association and authentication logs.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view or the specific integration data view if created.
3. Enter the following KQL filter: `data_stream.dataset : "cisco_aironet.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `cisco_aironet.log`)
   - `source.ip` (should reflect the WLC management IP)
   - `log.syslog.priority` or `log.level`
   - `cisco.aironet.mnemonic` (e.g., `AP_DISASSOCIATED` or `AUTH_SUCCESS`)
   - `message` (containing the raw log payload from the WLC)
5. Navigate to **Analytics > Dashboards** and search for "Cisco Aironet" to view pre-built visualizations and confirm data is populating the widgets.

# Troubleshooting

## Common Configuration Issues

- **Syslog Server Not Reached**: Ensure that the WLC can ping the Elastic Agent's IP address. Check for intermediate firewalls or Access Control Lists (ACLs) on the WLC itself that might be blocking outbound traffic on UDP/TCP port 514.
- **Port Conflict**: If the Elastic Agent fails to start the Syslog listener, verify that no other service (like `rsyslog` or `syslog-ng`) is already bound to port 514 on the Agent host. You may need to stop the local syslog service or change the integration port to a value like 1514.
- **Incorrect Severity Level**: If only critical errors are appearing in Kibana, check the WLC configuration under **Management > Logs > Config**. If the level is set to `Alerts (1)` or `Critical (2)`, most operational logs will be filtered out at the source. Change this to `Informational (6)`.
- **Facility Mismatch**: If you are using multiple syslog integrations on the same Agent, ensure the `facility` set on the WLC does not conflict with other logic you have implemented for log routing, though the Elastic integration usually handles this via the listener port.

## Ingestion Errors

- **Parsing Failures**: If logs appear in Discover but contain the `_grokparsefailure` tag, the syslog format sent by the WLC may be non-standard. Check if the WLC is using a custom message format and ensure it is set to the default Cisco format.
- **Field Mapping Issues**: Check the `error.message` field in Kibana for clues. If specific fields like `source.ip` are missing, it may be due to the WLC sending logs without a standard header; verify the "Timestamp" and "Hostname" settings in the WLC logging configuration.
- **Time Sync Issues**: If logs appear to be from the future or the past, ensure both the Cisco WLC and the Elastic Agent host are synchronized to the same NTP server.

## Vendor Resources

- [Configure Syslog Server on Wireless LAN Controllers - Cisco](https://www.cisco.com/c/en/us/support/docs/wireless/4100-series-wireless-lan-controllers/107252-WLC-Syslog-Server.html)
- [How to Configure Syslog on a Cisco IOS Switch or Router ...](https://networkproguide.com/configure-syslog-cisco-ios-switch-router/)

# Documentation sites

- [Syslog configuration on Cisco WLC - SGBox](https://www.sgbox.eu/en/knowledge-base/syslog-configuration-on-cisco-wlc/)
- Refer to the official vendor website for additional documentation.
