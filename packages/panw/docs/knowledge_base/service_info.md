# Service Info

## Common use cases
The Palo Alto Networks PAN-OS integration provides comprehensive visibility into your network security posture by centralizing firewall logs within the Elastic Stack. This integration is essential for security operations teams who need to monitor high-traffic environments and respond to threats in real-time.
- **Threat Detection and Response:** Monitor "threat" logs to identify malicious activity such as command-and-control (C2) traffic, spyware, and vulnerability exploits targeting your network.
- **Compliance and Audit Logging:** Capture "system" and "config" logs to track administrative changes, user logins, and policy updates to ensure adherence to regulatory standards like PCI DSS, HIPAA, or SOC2.
- **Network Traffic Analysis:** Analyze "traffic" logs to visualize bandwidth usage, identify top talkers, and troubleshoot connectivity issues across different zones and interfaces.
- **Security Policy Validation:** Review "URL filtering" and "WildFire" logs to ensure that web access policies and automated sandbox analysis are effectively blocking malicious content as intended.

## Data types collected

This integration can collect the following types of data:
- **Traffic Logs:** Information on network flows, including source/destination IP addresses, ports, protocols, application type, and session duration.
- **Threat Logs:** Security events triggered by Content-ID, covering viruses, spyware, vulnerabilities, and file blocking.
- **System Logs:** Operational events from the firewall, such as service restarts, HA status changes, and hardware alerts.
- **Configuration Logs:** Audit trails of all administrative changes made to the firewall settings and security policies.
- **Data Filtering Logs:** Records of specific data patterns (e.g., credit card numbers, SSNs) identified within files or traffic.
- **Data Formats:** Logs are ingested via Syslog in BSD format or Common Event Format (CEF) over TCP or UDP protocols.

## Compatibility

This integration is compatible with **Palo Alto Networks PAN-OS** devices. It supports versions 8.1, 9.x, 10.x, and 11.x. For the best parsing results, ensure the firewall is configured to send logs in the standard BSD Syslog format or CEF.

## Scaling and Performance

- **Transport/Collection Considerations:** For high-volume environments, TCP is the recommended transport protocol as it provides reliable delivery and congestion control. While UDP has lower overhead, it may result in data loss during network congestion. If using TCP, ensure the Elastic Agent and firewall can handle the persistent connection overhead.
- **Data Volume Management:** Traffic logs are typically the highest volume data source. To manage ingestion costs and processing load, use the "Match List" in the Log Forwarding Profile on the Palo Alto device to filter out low-value logs (e.g., specific trusted internal traffic) or log only at session end rather than both start and end.
- **Elastic Agent Scaling:** A single Elastic Agent can handle thousands of events per second (EPS), but resource consumption increases with throughput. For environments exceeding 10,000 EPS, consider deploying multiple Agents behind a load balancer or increasing the Agent's CPU and memory allocation to prevent ingestion bottlenecks.

# Set Up Instructions

## Vendor prerequisites
- **Administrative Access:** You must have superuser or device-administrator permissions on the PAN-OS web interface.
- **Network Connectivity:** Ensure the firewall can reach the Elastic Agent over the configured syslog port (e.g., TCP/UDP 9001).
- **Security Policies:** You must have existing security policies to which you can attach log forwarding profiles.
- **Licenses:** Ensure licenses for Threat Prevention or URL Filtering are active if you intend to collect those specific log types.
- **Device Information:** You will need the IP address or Fully Qualified Domain Name (FQDN) of the host where Elastic Agent is running.

## Elastic prerequisites
- **Elastic Agent:** An Elastic Agent must be installed and enrolled in a Fleet policy.
- **Integration Package:** The "Palo Alto Networks" integration must be added to the Elastic Agent's policy via Kibana.
- **Network Listener:** The Elastic Agent must be reachable on the specified Syslog port and configured to listen on all interfaces (`0.0.0.0`) or a specific management IP.

## Vendor set up steps

### For Syslog Collection:

1. **Create a Syslog Server Profile:**
   - Log in to your Palo Alto Networks firewall web interface (PAN-OS).
   - Navigate to **Device > Server Profiles > Syslog**.
   - Click **Add** at the bottom of the screen.
   - **Name**: Enter `Elastic-Agent-Syslog`.
   - In the **Servers** tab, click **Add**.
   - **Name**: `Elastic-Agent-Host`.
   - **Syslog Server**: Enter the IP address of the Elastic Agent.
   - **Transport**: Select `TCP` (recommended) or `UDP`.
   - **Port**: Enter `9001` (or your chosen port).
   - **Format**: Select `BSD`.
   - **Facility**: Select `LOG_USER`. Click **OK**.

2. **Define a Custom Log Format (Optional):**
   - If using CEF, go to the **Custom Log Format** tab within the same Syslog Server Profile.
   - Select the log type (e.g., Traffic) and paste the standard CEF format string.
   - Click **OK** twice to save the profile.

3. **Create a Log Forwarding Profile:**
   - Navigate to **Objects > Log Forwarding**.
   - Click **Add**.
   - **Name**: `Forward-to-Elastic`.
   - Under **Match List**, click **Add**.
   - **Name**: `Traffic-Events`.
   - **Log Type**: Select `traffic`.
   - Under **Syslog**, click **Add** and select your `Elastic-Agent-Syslog` profile.
   - Repeat for `threat`, `system`, and `config` log types as needed.
   - Click **OK**.

4. **Apply the Profile to Security Policies:**
   - Navigate to **Policies > Security**.
   - Select a rule and go to the **Actions** tab.
   - In the **Log Forwarding** dropdown, select `Forward-to-Elastic`.
   - Ensure **Log at Session End** is checked.
   - Click **OK**.

5. **Configure Service Routes (if management port isn't used):**
   - Navigate to **Device > Setup > Services**.
   - Click **Service Route Configuration**.
   - Select **Syslog** and choose the interface that can reach the Elastic Agent.

6. **Commit Changes:**
   - Click **Commit** in the top-right corner to activate the configuration.

## Kibana set up steps

### For PAN-OS Log Ingestion:

1. **Navigate to Integrations:**
   - In Kibana, go to **Management > Integrations**.
   - Search for and select **Palo Alto Networks**.
2. **Add Integration to Policy:**
   - Click **Add Palo Alto Networks**.
   - Select the **Palo Alto Networks PAN-OS** data stream.
3. **Configure Input Settings:**
   - Set the **Syslog Host** to `0.0.0.0` to listen on all interfaces or the specific Agent IP.
   - Set the **Syslog Port** to `9001` (must match the vendor setup).
   - Select the **Protocol** (TCP or UDP) used in the firewall configuration.
4. **Advanced Settings:**
   - Under **Internal Tags**, add `palo-alto` to help with filtering.
   - Ensure the **Format** matches what you set on the firewall (BSD or CEF).
5. **Save and Deploy:**
   - Click **Save and Continue** and then **Save and Deploy changes** to push the configuration to the Agents.

# Validation Steps

### 1. Trigger Data Flow on Palo Alto Networks:
- **Generate configuration event:** Log in to the firewall CLI or Web UI, enter configuration mode, make a minor change (like a description), and perform a **Commit**.
- **Generate authentication event:** Log out of the Palo Alto administration interface and log back in to generate a System/Auth log.
- **Trigger security event:** From a client device protected by the firewall, attempt to access a known test URL that triggers a URL filtering category you have set to "alert" or "block".
- **Generate traffic logs:** Ping an external IP address (e.g., `8.8.8.8`) from a zone behind the firewall to generate session traffic logs.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the following KQL filter: `data_stream.dataset : "palo_alto_networks.panos"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `palo_alto_networks.panos`)
   - `source.ip` and `destination.ip` (for traffic/threat logs)
   - `event.action` (e.g., `allowed`, `denied`, or `reset`)
   - `palo_alto_networks.panos.type` (e.g., `traffic`, `threat`, `system`)
   - `message` (containing the raw syslog payload)
5. Navigate to **Analytics > Dashboards** and search for "Palo Alto" to view the pre-built **[Logs Palo Alto Networks] Overview** dashboard.

# Troubleshooting

## Common Configuration Issues
- **Commit Not Performed**: Changes in Palo Alto Networks do not take effect until the **Commit** operation is successfully completed. If logs are not flowing, verify the commit status in the Tasks manager.
- **Network Path Blocked**: Use `tcpdump` on the Elastic Agent host to verify if packets are reaching the server: `tcpdump -ni any port 9514`. If no packets arrive, check intermediate firewalls or local `iptables`/`ufw` settings.
- **Log Forwarding Profile Assignment**: Logs will not be sent if the Log Forwarding Profile is created but not assigned to a specific **Security Policy Rule**. Ensure the profile is applied to the rules that govern your traffic.
- **Protocol Mismatch**: If the firewall is sending TCP and the Agent is listening on UDP, the connection will fail. Double-check that both ends are using the exact same transport protocol.

## Ingestion Errors
- **Parsing Failures**: If logs appear in Kibana but are not parsed into fields (remaining as raw `message`), check for the `tags` field containing `_grokparsefailure`. This often happens if the Syslog format on the firewall is set to something other than the default BSD or IETF formats.
- **Timestamp Mismatches**: Ensure the firewall and the Elastic Agent host have synchronized time via NTP. Significant clock skew can cause logs to appear in the "past" or "future" in Discover.
- **Field Mapping Issues**: Check the `error.message` field in Kibana Discover to see if specific fields are failing to map due to unexpected data types or length.

## Vendor Resources

- [Configure Syslog Monitoring - Palo Alto Networks](https://docs.paloaltonetworks.com/pan-os/11-1/pan-os-admin/monitoring/use-syslog-for-monitoring/configure-syslog-monitoring)
- [Configure Log Forwarding - Palo Alto Networks](https://docs.paloaltonetworks.com/pan-os/11-0/pan-os-admin/monitoring/configure-log-forwarding)

## Documentation sites
- [Configure Log Forwarding - Palo Alto Networks](https://docs.paloaltonetworks.com/pan-os/11-0/pan-os-admin/monitoring/configure-log-forwarding)
- [NGFW Administration: Configure Log Forwarding - Palo Alto Networks](https://docs.paloaltonetworks.com/ngfw/administration/monitoring/configure-log-forwarding)
- Refer to the official vendor website for additional resources.
