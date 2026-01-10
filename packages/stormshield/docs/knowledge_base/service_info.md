# Service Info

## Common use cases

- **Security Threat Monitoring:** Monitor real-time security alerts and firewall alarms to detect and respond to unauthorized access or malicious network activity.
- **Compliance and Auditing:** Maintain long-term records of web traffic and connection logs to satisfy regulatory requirements and internal security audits.
- **Network Traffic Analysis:** Analyze filter logs and connection patterns to optimize firewall rules and troubleshoot network connectivity issues.

## Data types collected

- **Logs:** Collects a variety of system and security logs including Alarms, Connections, Web traffic, Filter events, and System activity.

## Compatibility

- **Stormshield Network Security (SNS):** Compatible with SNS version 4.x and higher.
- **Formats:** Supports RFC5424 (recommended) and LEGACY-LONG syslog formats.

## Scaling and Performance

- Performance is dependent on the hardware model of the Stormshield appliance and the configured syslog throughput. 
- For high-volume environments, it is recommended to use **TCP** or **TLS** to ensure delivery reliability, and to distribute log processing across multiple Elastic Agents if necessary.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** Administrator credentials for the Stormshield SNS web interface.
- **Network Objects:** A defined network host object in SNS corresponding to the IP address of the Elastic Agent.
- **Syslog Slots:** At least one available syslog profile slot (SNS supports up to four concurrent profiles).

## Elastic prerequisites

- **Elastic Agent:** An active Elastic Agent installed on a host reachable by the Stormshield appliance.
- **Integration Policy:** The Stormshield integration must be added to an Elastic Agent policy with the Syslog UDP/TCP port configured to match the appliance settings.

## Vendor set up steps

1.  **Log in** to the Stormshield SNS administration interface.
2.  **Navigate** to the Syslog Configuration Menu via **LOGS - SYSLOG - IPFIX** and select the **Syslog** tab.
3.  **Create/Modify a Profile:** Select an available profile slot. Double-click the **Status** field to enable it.
4.  **Name the Profile:** Provide a descriptive name, such as `elastic-agent`.
5.  **Configure Server Details:**
    *   **Syslog server:** Select the network host object for your Elastic Agent.
    *   **Protocol:** Select `UDP`, `TCP`, or `TLS` (must match the Elastic Agent policy).
    *   **Port:** Enter the destination port (e.g., `514` or `1514`) that the Elastic Agent is listening on.
6.  **Define Log Format:** In the **Format** dropdown, select `RFC5424` for standard structured logging or `LEGACY-LONG` to prevent message truncation.
7.  **Select Log Types:** In the **Logs enabled** table, double-click the **Status** for each desired category (Alarm, Connection, Web, Filter, etc.) to enable forwarding.
8.  **Save:** Apply and save the configuration to begin data transmission.

## Kibana set up steps

1. In Kibana, go to **Management > Integrations**.
2. Search for and select the **Stormshield** integration.
3. Click **Add Stormshield**.
4. Configure the integration settings:
    *   **Listen Address:** Set to `0.0.0.0` or the specific IP of the Agent host.
    *   **Listen Port:** Set the port to match the one configured in the SNS appliance.
    *   **Protocol:** Match the protocol (UDP/TCP) selected in the SNS appliance.
5. Click **Save and continue** to add the integration to your Agent policy.

# Validation Steps

1. **Check SNS Status:** In the SNS interface, verify that the Syslog profile status is active and shows no connectivity errors.
2. **Verify Data Flow:** In Kibana, navigate to **Analytics > Discover**.
3. **Filter Logs:** Search for `event.dataset : "stormshield.*"` to confirm logs are being ingested.
4. **Check Dashboards:** Open the **[Logs Stormshield] Overview** dashboard in Kibana to view visualized data.

# Troubleshooting

## Common Configuration Issues

- **Protocol Mismatch:** If logs are not appearing, ensure the protocol (e.g., UDP) matches exactly between the SNS profile and the Elastic Agent policy.
- **Firewall Blockage:** Verify that intermediate firewalls or host-based firewalls (like iptables or Windows Firewall) allow traffic on the configured syslog port.
- **Host Object Errors:** Ensure the "Syslog server" object in SNS is configured with the correct, static IP address of the Elastic Agent.

## Ingestion Errors

- **Parsing Failures:** If `error.message` indicates parsing issues, ensure the SNS format is set to `RFC5424`. Standardizing on this format ensures compatibility with the Elastic integration's built-in grok patterns.
- **Truncated Messages:** If log lines appear cut off, switch the SNS log format to `LEGACY-LONG`.

## API Authentication Errors

- **Not specified:** This integration relies on syslog (push mechanism) rather than API polling, so API authentication errors are generally not applicable for data ingestion.

## Vendor Resources

- [Stormshield SNS Syslog Configuration Guide](https://documentation.stormshield.eu/SNS/v4/en/Content/User_Configuration_Manual_SNS_v4/Logs-syslog/Syslog_tab.htm)
- [Stormshield Troubleshooting Support](https://www.stormshield.com/technical-documentation/)

# Documentation sites

- [Elastic Stormshield Integration Documentation](https://docs.elastic.co/integrations/stormshield)
- [Official Stormshield SNS Documentation Portal](https://documentation.stormshield.eu/)
- [ManageEngine SNS Syslog Reference](https://www.manageengine.com/products/eventlog/help/StandaloneManagedServer-UserGuide/ConfiguringSyslogService/configuring-the-syslog-service-on-stormshield-devices.html)