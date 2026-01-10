# Service Info

## Common use cases

- **Security Threat Monitoring:** Monitor real-time security events such as intrusion prevention system (IPS) alerts, gateway anti-virus detections, and botnet filtering to identify and respond to active threats.
- **Network Traffic Auditing:** Analyze firewall rule hits and network access logs to audit internal traffic patterns, ensure compliance, and optimize security policies.
- **System Health and Administration:** Track administrator logins, configuration changes, and system hardware status to maintain the operational integrity of the firewall cluster.

## Data types collected

- **Logs:** System logs, network traffic logs, security event logs, and administrator activity logs formatted as Syslog messages.
- **Events:** Discrete security and operational events including VPN tunnel status, session timeouts, and rule-based traffic permits/denies.

## Compatibility

- **SonicOS 7.X:** Fully supported via Enhanced Syslog format.
- **SonicOS 6.5:** Fully supported via Enhanced Syslog format.
- **Elastic Agent:** Compatible with the UDP/TCP Syslog input.

## Scaling and Performance

- **Throughput:** Performance is largely dependent on the firewall's logging level (e.g., "Informational" vs "Alert") and the hardware model's maximum packets-per-second capability.
- **Optimization:** To prevent performance degradation on high-traffic appliances, it is recommended to use "Enhanced" format to minimize payload size and to filter log categories at the source to include only necessary security events.

# Set Up Instructions

## Vendor prerequisites

- **Administrator Access:** Super User or Administrative permissions for the SonicWall management interface.
- **Network Connectivity:** Unrestricted network path between the SonicWall appliance and the Elastic Agent on the configured Syslog port (default 514).
- **Firmware:** A supported version of SonicOS (6.5 or 7.x recommended).

## Elastic prerequisites

- **Elastic Agent:** An active Elastic Agent enrolled in Fleet.
- **Integration Policy:** The "UDP/TCP Syslog" or "SonicWall" integration added to an agent policy.
- **Listening Port:** A dedicated port (typically 514 or 5514) configured in the integration to receive incoming UDP/TCP traffic.

## Vendor set up steps

### For SonicOS 7.X
1. Log in to the SonicWall administrator interface.
2. Navigate to **Device** > **Log** > **Syslog**.
3. Under the **Syslog Servers** section, click **Add**.
4. Configure the server settings:
    - **Name or IP Address**: IP address of the Elastic Agent.
    - **Port**: Match the port configured in Elastic Agent (default 514).
    - **Syslog Format**: Select **Enhanced** (Required for proper parsing).
    - **Syslog ID**: Set to `firewall`.
5. Click **OK**.
6. Navigate to **Log** > **Settings** and ensure desired event categories are enabled for logging.
7. Click **Accept** to apply.

### For SonicOS 6.5
1. Log in to the SonicWall administrator interface.
2. Navigate to **Manage** > **Log Settings** > **SYSLOG**.
3. Under **Syslog Servers**, click **Add**.
4. Configure the server settings:
    - **Name or IP Address**: IP address of the Elastic Agent.
    - **Port**: Match the port configured in Elastic Agent (default 514).
    - **Syslog Format**: Select **Enhanced**.
    - **Syslog ID**: Set to `firewall`.
5. Click **OK**.
6. Navigate to **Log** > **Settings** and select the desired **Logging Level**.
7. Check the boxes for the event types you wish to forward.
8. Click **Accept** to save changes.

## Kibana set up steps

1. In Kibana, go to **Management** > **Integrations**.
2. Search for and select the **SonicWall** integration.
3. Click **Add SonicWall**.
4. Configure the integration settings:
    - **Syslog Host**: Set to `0.0.0.0` or the specific IP the agent should listen on.
    - **Syslog Port**: Ensure this matches the port used in the SonicWall vendor setup (e.g., 514).
5. Select the **Existing integration policy** where your agent is enrolled.
6. Click **Save and continue**.

# Validation Steps

1. **Verify Connectivity:** Run `tcpdump -i any port 514` on the Elastic Agent host to confirm packets are arriving from the SonicWall IP.
2. **Trigger Test Event:** Perform a test action on the SonicWall (e.g., a failed login attempt) to generate a log entry.
3. **Check Data Stream:** In Kibana, navigate to **Observability** > **Logs** > **Stream** and filter by `event.dataset : "sonicwall.firewall"`.
4. **Dashboard Verification:** Open the [SonicWall] Overview dashboard to verify that widgets are populating with data.

# Troubleshooting

## Common Configuration Issues

- **No Data Received:** Verify that the firewall's Syslog ID is set to `firewall`. If it differs, the parser may fail to identify the source.
- **Incorrect Port:** Ensure the port configured in the SonicWall "Add Syslog Server" window matches the port the Elastic Agent is listening on. Check host-level firewalls (iptables/firewalld) to ensure the port is open.
- **Network Routing:** Ensure there is no NAT or upstream firewall blocking UDP 514 traffic between the SonicWall and the Elastic Agent.

## Ingestion Errors

- **Parsing Failures:** If `error.message` indicates a parsing error, verify that the **Syslog Format** on the SonicWall is set to **Enhanced**. "Default" or "Legacy" formats are not compatible with the standard Elastic integration.
- **Field Mapping Issues:** Ensure the SonicWall system clock is synchronized via NTP; mismatched timestamps can cause issues with time-based indexing and data visibility.

## API Authentication Errors

- Not specified (This integration primarily uses Syslog/UDP and does not require API authentication).

## Vendor Resources

- [How can I configure a syslog server on a SonicWall firewall?](https://www.sonicwall.com/support/knowledge-base/how-can-i-configure-a-syslog-server-on-a-sonicwall-firewall/kA1VN0000000TWl0AM)
- [SonicWall Technical Documentation Center](https://sonicwall.vanillacommunities.com/technology-and-support/categories/technical-documentation-center)
- [SonicWall Support Knowledge Base](https://www.sonicwall.com/support/knowledge-base/)

# Documentation sites

- [SonicOS 7.X Administration Guide](https://www.readkong.com/page/sonicos-7-device-settings-administration-guide-sonicwall-6297948)
- [Elastic Integration Documentation: SonicWall](https://docs.elastic.co/en/integrations/sonicwall)
- [SonicWall Official Product Overview](https://www.sonicwall.com/products/firewalls/)