# Service Info

The Fortinet FortiGate integration allows organizations to ingest logs from FortiGate Next-Generation Firewalls (NGFW) into the Elastic Stack. This integration provides deep visibility into network traffic, security threats, and system performance, enabling security teams to monitor and respond to incidents in real-time.

## Common use cases

The Fortinet FortiGate integration is essential for several security and operational workflows:
- **Network Traffic Monitoring:** Analyze flow data to identify bandwidth bottlenecks, unusual traffic spikes, or unauthorized lateral movement within the network.
- **Threat Detection and Incident Response:** Monitor Intrusion Prevention System (IPS), Antivirus, and Web Filter logs to detect and respond to malicious activities and malware infections.
- **Compliance and Auditing:** Maintain long-term logs of administrative changes, user authentication events, and firewall policy hits to meet regulatory requirements like PCI-DSS or HIPAA.
- **VPN and User Activity Tracking:** Gain insights into remote access patterns, including login/logout times, assigned IP addresses, and tunnel duration for both SSL and IPsec VPNs.

## Data types collected

This integration can collect the following types of data:
- **Traffic Logs:** Detailed information about network sessions, including source/destination IPs, ports, protocols, and data volume (bytes sent/received).
- **Security Logs:** High-fidelity alerts from security profiles such as Antivirus, Web Filter, Application Control, and IPS (Intrusion Prevention System).
- **Event Logs:** System-level information including administrator logins, configuration changes, HA (High Availability) status, and hardware health.
- **Format:** Data is typically received in **Syslog** format, including key-value pairs or CEF-compliant strings depending on the device configuration.
- **Metrics:** While primarily log-based, the integration facilitates the extraction of metrics like connection counts and session rates from the log stream.

## Compatibility

This integration is compatible with **Fortinet FortiGate / FortiOS** devices.
- **Supported Versions:** Tested and supported for FortiOS version 7.6.3 and higher.
- **Legacy Support:** Generally compatible with earlier 6.x and 7.x versions that support standard syslog output.

## Scaling and Performance

- **Transport/Collection Considerations:** For high-volume environments, using TCP (set as `mode reliable` in FortiOS) is recommended over UDP to ensure delivery and prevent data loss during network congestion. However, UDP provides lower overhead and may be preferred in environments where performance is prioritized over absolute reliability.
- **Data Volume Management:** To reduce the load on the Elastic Stack and the network, use the FortiOS CLI to configure log filters (`config log syslogd filter`). By setting the severity level to `information` or `warning` and disabling unnecessary log categories (e.g., disabling `traffic` logs for specific low-risk policies), you can significantly reduce data volume without losing critical security insights.
- **Elastic Agent Scaling:** A single Elastic Agent can handle thousands of events per second (EPS), but for enterprise-scale deployments with multiple high-throughput firewalls, consider deploying a dedicated Elastic Agent for syslog collection. If the load exceeds the capacity of a single agent, use a network load balancer to distribute syslog traffic across a pool of multiple Elastic Agents.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have a super_admin or a custom admin profile with read/write permissions for "Log & Report" settings on the FortiGate device.
- **Network Connectivity:** The FortiGate appliance must be able to reach the Elastic Agent's IP address over the configured port (default is UDP/TCP 514).
- **Firewall Rules:** Ensure that any intermediate firewalls or local ACLs allow traffic from the FortiGate Management IP (or source IP) to the Elastic Agent.
- **Licensing:** Ensure the FortiGate has a valid license if specific security logs (like IPS or Sandbox) are required for ingestion.
- **System Time:** Ensure NTP is configured on the FortiGate to provide accurate timestamps for all generated logs.

## Elastic prerequisites

- **Elastic Agent Installation:** An Elastic Agent must be installed on a host reachable by the FortiGate appliance.
- **Fleet Management:** The agent should be enrolled in Fleet and assigned to a policy that includes the Fortinet FortiGate integration.
- **Connectivity:** The host running the agent must have port 514 (or your chosen custom port) open and not occupied by other syslog services (like rsyslog or syslog-ng).

## Vendor set up steps

You can configure your Fortinet FortiGate appliance to send syslog messages to the Elastic Agent through either the graphical user interface (GUI) or the command-line interface (CLI).

### Method 1: Configuration via GUI (Basic Setup)
This method is suitable for basic setups using default protocols.
1. Log in to your FortiGate admin interface.
2. Navigate to **Log & Report > Log Settings**.
3. Enable the **Send Logs to Syslog** toggle.
4. In the **IP Address/FQDN** field, enter the IP address of the server where the Elastic Agent is running.
5. Click **Apply** at the bottom of the page to save the changes. Note that the GUI method typically defaults to UDP on port 514.

### Method 2: Configuration via CLI (Recommended)
The CLI provides more granular control over the log format, protocol, and content.
1. Open a terminal session (SSH) to your FortiGate appliance.
2. Enter the following commands to access the syslog configuration:
   ```shell
   config log syslogd setting
   ```
3. Enable the syslog service:
   ```shell
   set status enable
   ```
4. Set the IP address of the Elastic Agent:
   ```shell
   set server "YOUR_ELASTIC_AGENT_IP"
   ```
5. Specify the communication protocol. Use `reliable` for TCP or `udp` for UDP. This must match the configuration in the Elastic Agent integration.
   ```shell
   set mode reliable
   ```
6. Set the port number that the Elastic Agent is listening on (default 514):
   ```shell
   set port 514
   ```
7. Ensure compatibility by disabling CSV format:
   ```shell
   set csv disable
   ```
8. Save the configuration:
   ```shell
   end
   ```

### Step 3: Configure Log Filtering (CLI)
Specify which log types and severity levels to send to the Elastic Agent.
1. Enter the syslog filter configuration:
   ```shell
   config log syslogd filter
   ```
2. Set the minimum severity level (e.g., `information`):
   ```shell
   set severity information
   ```
3. Enable forwarding for all desired log categories:
   ```shell
   set traffic enable
   set event enable
   set utm enable
   set web enable
   set app-ctrl enable
   set ips enable
   set antivirus enable
   ```
4. Save the filter configuration by typing `end`.

## Kibana set up steps

### For Syslog Collection:
1. Open Kibana and navigate to **Management > Integrations**.
2. Search for **Fortinet FortiGate** and select the integration.
3. Click **Add Fortinet FortiGate**.
4. Configure the integration settings:
   - **Syslog Host:** Set to `0.0.0.0` to listen on all interfaces, or provide the specific IP of the Agent host.
   - **Syslog Port:** Set to `514` (ensure this matches your FortiGate CLI/GUI configuration).
   - **Protocol:** Select `udp` or `tcp` to match the FortiGate setting.
5. Scroll down and click **Save and continue**.
6. Select the **Agent Policy** that contains your active Elastic Agent.
7. Click **Save and deploy changes** to push the configuration to the Agent.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from Fortinet to the Elastic Stack.

### 1. Trigger Data Flow on Fortinet FortiGate:
1. **Generate Web Traffic:** From a workstation located behind the FortiGate firewall, browse to several public websites. This will trigger Traffic and Web Filter logs.
2. **Generate Admin Event:** Log out of the FortiGate GUI/CLI and log back in. This generates an authentication and system event.
3. **Trigger Policy Event:** If possible, attempt to access a website that is explicitly blocked by your Web Filter profile to generate a "Deny" action log.
4. **Configuration Change:** Enter configuration mode in the CLI (`config system global`) and exit without making changes to trigger a configuration log.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view or create one for the specific integration.
3. Enter the following KQL filter: `data_stream.dataset : "fortinet_fortigate.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `fortinet_fortigate.log`)
   - `source.ip` (the client IP behind the firewall)
   - `destination.ip` (the external server IP)
   - `event.action` (e.g., `allowed`, `denied`, or `passthrough`)
   - `fortinet.fortigate.subtype` (e.g., `forward`, `webfilter`, or `system`)
   - `message` (containing the raw syslog payload)
5. Navigate to **Analytics > Dashboards** and search for **[Logs Fortinet FortiGate] Overview** to view the pre-built dashboards.

# Troubleshooting

## Common Configuration Issues

- **Logs Not Received (Network/Firewall)**: Ensure that UDP or TCP port 514 is allowed on the host OS firewall (e.g., iptables, firewalld, or Windows Firewall) where the Elastic Agent is running. Also, verify that no other service is already listening on port 514.
- **Incorrect Source IP**: If the FortiGate is sending logs through a VPN or NAT, ensure the `set server` IP in the CLI is the reachable address of the Elastic Agent from that specific routing context.
- **Partial Data/Missing Logs**: If only some logs (like system logs) are appearing but not traffic logs, verify the filtering settings in `config log syslogd filter`. Ensure `set traffic enable` is set to `enable`.
- **Policy Not Applied**: After changing log settings in the GUI, ensure you clicked **Apply**. In the CLI, always ensure you typed `end` to save and commit the configuration changes.

## Ingestion Errors

- **Parsing Failures**: If logs appear in Kibana but contain a `_grokparsefailure` or `_jsonparsefailure` tag, verify that the FortiGate is sending logs in the standard key-value format and not in a proprietary CEF or JSON format that might require a different integration template.
- **Timezone Mismatch**: If logs appear with the wrong timestamp, check the system time on the FortiGate appliance (`get system status`) and ensure it is synchronized via NTP.
- **Field Mapping Issues**: Check the `error.message` field in Kibana Discover. If you see "mapping conflict" errors, it may be due to an older version of the integration or a custom field being sent by the FortiGate that conflicts with ECS.

## Vendor Resources

- [How to configure syslog on FortiGate - Fortinet Community](https://community.fortinet.com/t5/FortiGate/Technical-Tip-How-to-configure-syslog-on-FortiGate/ta-p/331959)
- [Configuring multiple Syslog servers - Fortinet Community](https://community.fortinet.com/t5/FortiGate/Technical-Tip-Configuring-multiple-Syslog-servers/ta-p/194117)

# Documentation sites

- [Log settings and targets | FortiGate / FortiOS 7.6.3 Administration Guide](https://docs.fortinet.com/document/fortigate/7.6.3/administration-guide/250999/log-settings-and-targets)
- [How To Configure Fortinet Fortigate Logging and Reporting](https://www.webspy.com/getting-started/fortinet/)
- Refer to the official vendor website for additional resources.
