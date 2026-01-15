# Service Info

## Common use cases

The Cisco Firepower Threat Defense (FTD) integration allows organizations to ingest high-fidelity security and network event data into the Elastic Stack for centralized visibility and advanced analytics. By collecting logs from FTD appliances, security teams can monitor their network perimeter and internal segments with precision.
- **Threat Detection and Incident Response:** Monitor intrusion events (IPS/IDS) and malware detections in real-time to identify and mitigate active threats across the network infrastructure.
- **Compliance Auditing and Reporting:** Maintain a comprehensive audit trail of administrative changes, user logins, and policy modifications to satisfy regulatory requirements like PCI-DSS, HIPAA, and GDPR.
- **Network Traffic Analysis:** Analyze connection logs to identify top talkers, unusual traffic patterns, and potential data exfiltration attempts through granular protocol and port-level visibility.
- **Operational Troubleshooting:** Use system and diagnostic logs to identify hardware failures, interface flapping, or configuration errors that may be impacting network performance and availability.

## Data types collected

This integration can collect the following types of data:
- **Connection Logs:** Detailed records of network sessions including source/destination IP addresses, ports, protocols, and data volumes.
- **Intrusion Events:** Logs from the integrated IPS engine identifying signature-based attacks and exploit attempts.
- **Security Intelligence Logs:** Events triggered by IP, URL, or DNS blacklists and reputation feeds.
- **System and Audit Logs:** Administrative events such as user logins, configuration deployments, and health status changes.
- **Malware and File Events:** Information regarding file transfers and sandbox analysis results for potentially malicious attachments.
- **Format:** Data is primarily collected via Syslog, supporting standard Cisco message formats and potentially Common Event Format (CEF).

## Compatibility

This integration is compatible with **Cisco Firepower Threat Defense (FTD)** appliances managed via the **Cisco Firepower Management Center (FMC)**. It supports FTD software version 6.x and version 7.x, as these versions utilize the FMC for centralized policy and logging configuration.

## Scaling and Performance

- **Transport/Collection Considerations:** The Cisco FTD integration primarily uses network-based collection (Syslog). While UDP (port 514) offers lower latency and less overhead on the firewall, TCP (port 1468) is recommended for production environments to ensure reliable delivery of security events and to support TLS encryption for data in transit.
- **Data Volume Management:** Firewalls can generate an immense volume of logs. To optimize performance, it is recommended to enable "Log at End of Connection" rather than "At Start and End" to halve connection log volume. Additionally, users should configure severity levels to only send relevant informational or warning events, filtering out high-frequency debugging logs at the FTD source.
- **Elastic Agent Scaling:** For high-throughput environments processing tens of thousands of events per second, a dedicated Elastic Agent should be deployed. In extremely large deployments, multiple Agents should be placed behind a network load balancer to distribute the incoming syslog stream and provide high availability.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have administrative credentials to log in to the Firepower Device Manager (FDM) web interface or Firepower Management Center (FMC).
- **Network Connectivity:** The FTD device must have a clear network path to the Elastic Agent. Ensure that the management or data interface used for logging can reach the Agent's IP address.
- **Firewall Rules:** Any intermediate firewalls must allow traffic on the chosen syslog port (e.g., UDP 514 or TCP 1468).
- **License Requirements:** Ensure the FTD device has active licenses for Threat, Malware, and URL Filtering if you wish to collect those specific event types.
- **IP Information:** Knowledge of the Elastic Agent's static IP address and the intended protocol (TCP or UDP).

## Elastic prerequisites

- **Elastic Agent Installation:** The Elastic Agent must be installed on a host reachable by the Cisco FTD and must be enrolled in Fleet with an active policy.
- **Version Requirements:** It is recommended to use Elastic Stack version 8.x for the best compatibility with the Cisco FTD integration features.
- **Network Configuration:** The host running the Elastic Agent must be configured to allow inbound traffic on the specific syslog port (default `514`) from the FTD management IP or data interface IP.

## Vendor set up steps

### For Syslog Collection via Firepower Management Center (FMC):

1.  Log in to your **Firepower Management Center (FMC)** web interface with administrative privileges.
2.  Navigate to **Devices > Platform Settings** from the top navigation bar.
3.  Create a new policy by clicking **New Policy** and selecting **Threat Defense Settings**, or click the edit icon (pencil) next to an existing policy already assigned to your FTD appliances.
4.  Within the policy editor, select the **Syslog** tab from the left-hand sidebar.
5.  On the **Logging Setup** page, locate the **Enable Logging** checkbox and ensure it is checked. This is a mandatory global setting.
6.  Select the **Syslog Servers** tab from the left-hand menu to define the Elastic Agent destination.
7.  Click the **Add** button to configure the connection to the Elastic Agent.
8.  In the **Add Syslog Server** dialog, fill in the following details:
    *   **IP Address**: Select an existing network object or create a new one containing the IP address of your Elastic Agent host.
    *   **Protocol**: Select either **TCP** or **UDP**. Ensure this matches your configuration in Kibana.
    *   **Port**: Enter the port number the Elastic Agent is listening on (default is `514`).
    *   **Available Zones**: Select the security zones or interfaces through which the FTD can reach the Elastic Agent and move them to the **Selected Zones/Interfaces** column.
9.  **Note on TCP**: If you selected **TCP**, it is highly recommended to check the box **Allow user traffic to pass when TCP syslog server is down** to prevent a network outage if the Elastic Agent host is rebooted.
10. Navigate to **Logging Destinations** on the left menu to define what events are sent.
11. Click **Add** to create a new logging filter:
    *   **Logging Destination**: Select **Syslog Server**.
    *   **Event Class**: Choose specific classes (e.g., `app`, `auth`, `system`) or use a custom **Event List**.
    *   **Logging Level**: Select the minimum severity level (e.g., `Informational`) to forward.
12. Click **Save** at the bottom of the page to save the policy configuration.
13. Deploy the changes by navigating to **Deploy > Deployment**, selecting the target FTD appliances, and clicking **Deploy**.

## Kibana set up steps

### For Cisco FTD Log Ingestion:

1. **Navigate to Integrations:** Log in to Kibana and go to **Management > Integrations**. Search for "Cisco FTD" and select the integration.
2. **Add Integration:** Click **Add Cisco FTD**.
3. **Configure Integration Inputs:**
   - Locate the **Syslog** input section.
   - **Syslog Host:** Set this to `0.0.0.0` to listen on all available interfaces or the specific IP of the Agent host.
   - **Syslog Port:** Set this to the port you configured in the FMC (e.g., `514`).
   - **Protocol:** Select the protocol (TCP or UDP) that matches your FMC configuration.
4. **Advanced Settings:** Expand the **Advanced options** if you need to adjust internal timeouts or custom tags for specific FTD clusters.
5. **Assign to Policy:** Choose the **Existing hosts** tab and select the Agent Policy that is currently applied to your Elastic Agent.
6. **Save and Deploy:** Click **Save and Continue**, then click **Add integration to policy**. The configuration will be automatically pushed to your Elastic Agent.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from Cisco FTD to the Elastic Stack.

### 1. Trigger Data Flow on Cisco FTD:
- **Generate authentication event:** Log out of the FMC or the FTD CLI and log back in to trigger an authentication syslog message.
- **Generate configuration event:** Access the FTD CLI and enter configuration mode (e.g., `configure terminal` if applicable, or make a minor change in FMC) and then exit or deploy to generate an audit log.
- **Generate network traffic:** From a workstation protected by the FTD, browse to several external websites to generate connection events.
- **Trigger security event:** If in a lab environment, use a tool like `curl` to simulate a non-harmful but signature-matching string to trigger an IPS event.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the following KQL filter: `data_stream.dataset : "cisco_ftd.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `cisco_ftd.log`)
   - `source.ip` and/or `destination.ip` (verifying network traffic parsing)
   - `event.action` (e.g., connection-built, connection-teardown, or blocked)
   - `cisco_ftd.mnemonic` (e.g., `1-106023` or similar Cisco-specific identifiers)
   - `message` (containing the raw syslog payload from the FTD)
5. Navigate to **Analytics > Dashboards** and search for "Cisco FTD" to view pre-built visualizations for traffic and threats.

# Troubleshooting

## Common Configuration Issues

- **Pending Deployment**: Changes made in the FDM interface (like adding a syslog server) do not take effect immediately. You must click the **Deploy** icon and wait for the process to complete before logs start flowing.
- **Interface Routing Mismatch**: If you select the "Management" interface for syslog but the Agent is on a network reachable only via a "Data" interface, the logs will fail to send. Verify the routing table on the FTD.
- **Rule-Level Logging Disabled**: If you see system logs but no traffic/connection logs, check the Access Control Policy. Each rule must have logging explicitly enabled in its properties.
- **Port Conflict**: Ensure no other service or another Elastic Agent integration is already listening on the configured syslog port (e.g., 514) on the same host.

## Ingestion Errors

- **Grok Parse Failures**: If logs appear with the tag `_grokparsefailure`, the FTD might be sending logs in an unexpected format. Ensure the FTD is not configured to use a custom "External Logging" format that deviates from standard Cisco timestamps.
- **Timezone Mismatches**: If logs appear to be "missing," check the time filter in Discover. Mismatched timezones between the FTD device and the Elastic Stack can cause logs to be indexed with timestamps in the future or the past.
- **Truncated Messages**: If log messages appear cut off, verify the MTU settings on the network path and the maximum message length settings in the Elastic Agent's UDP/TCP input configuration.
- **Identifying Failures**: Check the `error.message` field in Kibana Discover for specific details regarding mapping or parsing exceptions.

## Vendor Resources

- [Configure Logging on FTD via FMC - Cisco](https://www.cisco.com/c/en/us/support/docs/security/firepower-ngfw/200479-Configure-Logging-on-FTD-via-FMC.html)
- [Cisco Secure Firewall Threat Defense Syslog Messages](https://www.cisco.com/c/en/us/td/docs/security/firepower/Syslogs/fptd_syslog_guide/about.html)

# Documentation sites

- [Configure and Verify Syslog in Firepower Device Manager - Cisco](https://www.cisco.com/c/en/us/support/docs/security/firepower-2130-security-appliance/220231-configure-and-verify-syslog-in-firepower.html)
- Refer to the official vendor website for additional resources.
