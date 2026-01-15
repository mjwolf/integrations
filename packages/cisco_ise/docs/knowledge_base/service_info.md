# Service Info

## Common use cases

The Cisco ISE integration for Elastic provides comprehensive visibility into network access control, identity management, and security policy enforcement across your enterprise.
- **Security Monitoring and Auditing:** Track all administrative actions within the Cisco ISE console to identify unauthorized configuration changes or potential insider threats.
- **Authentication Analysis:** Monitor successful and failed authentication attempts in real-time to detect brute-force attacks or identify misconfigured network devices and client supplicants.
- **Compliance Reporting:** Collect and retain long-term logs of network access events, which are essential for meeting regulatory requirements such as PCI-DSS, HIPAA, and SOX.
- **Troubleshooting Network Access:** Rapidly diagnose connectivity issues by analyzing detailed logs from Policy Service Nodes (PSNs) to see exactly why a specific user or device was denied access.

## Data types collected

This integration can collect the following types of data:
- **Authentication Logs:** Detailed records of user and device authentication attempts, including protocol used (802.1X, MAB, WebAuth) and the result (Pass/Fail).
- **Authorization Logs:** Information regarding the specific permissions and profiles granted to a user or device after successful authentication.
- **Administrative Audit Logs:** Captures changes made to ISE system configuration, policy sets, and identity stores by administrators.
- **System Health and Operational Logs:** Monitoring data related to the health of ISE nodes, including service restarts, database status, and synchronization events.
- **Profiler Logs:** Data regarding the identification and categorization of endpoints connected to the network.
- **Data Format:** All logs are typically collected via Syslog in RFC 3164 or RFC 5424 formats, or encapsulated as CEF (Common Event Format) messages.

## Compatibility

The **Cisco ISE** integration is compatible with:
- **Cisco ISE versions 2.x and 3.x** (including 3.0, 3.1, and 3.2+).
- Supports logs exported via standard Syslog protocols (UDP and TCP).
- Requires **Elastic Agent** version 7.14.0 or higher for full data stream support.

## Scaling and Performance

- **Transport/Collection Considerations:** When configuring Cisco ISE to export logs, the choice between UDP and TCP is critical. UDP (port 514) offers lower overhead and is suitable for high-volume environments where occasional packet loss is acceptable. TCP is recommended for environments requiring guaranteed delivery (e.g., compliance auditing), though it introduces additional overhead and potential head-of-line blocking if the network or Elastic Agent is congested.
- **Data Volume Management:** Cisco ISE generates significant log volume, especially in large-scale deployments. To manage this, administrators should use the **Logging Categories** feature in ISE to only forward essential logs (e.g., "Passed Authentications" and "Failed Attempts") and exclude high-verbosity debug logs unless active troubleshooting is required. Filtering at the source reduces the processing load on both the ISE Policy Service Nodes and the Elastic Stack.
- **Elastic Agent Scaling:** For high-throughput Cisco ISE deployments (exceeding 5,000 events per second), it is recommended to deploy multiple Elastic Agents behind a network load balancer. Ensure the host system running the Elastic Agent has sufficient CPU and memory resources (typically 4 vCPUs and 8GB RAM per high-traffic instance) to handle peak authentication windows, such as the start of a business day.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have Super Admin or System Admin permissions within the Cisco ISE Administration Interface.
- **Network Connectivity:** Ensure the Cisco ISE nodes can reach the Elastic Agent host on the configured Syslog port (e.g., UDP/TCP 514, 5514, or 9514).
- **Firewall Rules:** Update any intermediary firewalls to allow traffic from the IP addresses of all ISE nodes (PAN, MNT, and PSNs) to the Elastic Agent IP on the designated port.
- **License Requirements:** Ensure your Cisco ISE deployment has the appropriate licensing (Essentials, Advantage, or Premier) enabled to support external logging exports.
- **Inventory of Nodes:** Prepare a list of all ISE node IP addresses that will be sending logs to ensure they are properly recognized by the Elastic Agent.

## Elastic prerequisites

- **Elastic Agent:** An active Elastic Agent must be installed and enrolled in Fleet.
- **Integration Policy:** The Cisco ISE integration must be added to the agent's policy in Kibana.
- **Network Accessibility:** The Elastic Agent host must be configured to listen on all interfaces (`0.0.0.0`) or a specific IP reachable by Cisco ISE.

## Vendor set up steps

### For Syslog Collection:

1.  Log in to your Cisco ISE Administration Interface with administrative credentials.
2.  Navigate to **Administration > System > Logging > Remote Logging Targets**.
3.  Click **Add** to create a new remote logging target for the Elastic Stack.
4.  Configure the target with the following specific parameters:
    *   **Name**: Provide a descriptive name such as `Elastic_Agent_ISE_Target`.
    *   **Target Type**: Select `UDP Syslog` or `TCP Syslog`. This must strictly match the protocol you plan to configure in the Kibana integration settings.
    *   **IP Address**: Enter the static IP address of the server where the Elastic Agent is running.
    *   **Port**: Enter the port number (e.g., `514` or `5514`) on which the Elastic Agent is listening.
    *   **Facility Code**: Select a syslog facility code such as `Local6` or `Local7`.
    *   **Maximum Length**: Set this value to `8192` bytes to prevent the truncation of large Cisco ISE log messages.
5.  Click **Save** to finalize the creation of the logging target. If a warning appears regarding unsecure connections, acknowledge it to proceed.
6.  Assign the logging categories to the new target by navigating to **Administration > System > Logging > Logging Categories**.
7.  Locate the categories you wish to export (e.g., `Passed Authentications`, `Failed Attempts`, `Radius Accounting`, `Administrative and Operational Audit`).
8.  For each desired category, click to edit and move your newly created target (e.g., `Elastic_Agent_ISE_Target`) from the **Available** column to the **Selected** column.
9.  Click **Save** at the bottom of the page for each category modified.
10. Ensure the **Log Collector** is active on the ISE nodes by checking the deployment status under **Administration > System > Deployment**.

## Kibana set up steps

### For UDP/TCP Syslog Input:

1.  In Kibana, navigate to **Management > Integrations**.
2.  Search for **Cisco ISE** and select the integration.
3.  Click **Add Cisco ISE** to add it to an Agent policy.
4.  Under the **Logs (Cisco ISE)** section, configure the following:
    *   **Syslog Host**: Set this to `0.0.0.0` to listen on all available network interfaces, or specify the specific IP of the Agent host.
    *   **Syslog Port**: Enter the port number you configured in the ISE Remote Logging Target (e.g., `514`).
    *   **Protocol**: Select either `udp` or `tcp` to match the ISE configuration.
5.  Scroll down and click **Save and Continue**.
6.  When prompted, click **Add agent to your hosts** or, if the agent is already enrolled, click **Save and deploy changes**.
7.  Ensure the policy is successfully deployed to the Agent by checking the **Fleet > Agents** tab.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from Cisco ISE to the Elastic Stack.

### 1. Trigger Data Flow on Cisco ISE:
- **Generate authentication event:** Log out of the Cisco ISE Administration Interface and log back in. This will generate an administrative audit log.
- **Generate configuration event:** Navigate to any policy page, make a minor descriptive change (like a description update), and click **Save**.
- **Trigger endpoint event:** Use a test device to authenticate against the network via RADIUS (e.g., connecting a laptop to a 802.1X enabled switch port).
- **Manual Log Test:** If available in your ISE version, use the "Log Test" feature under the Remote Logging Target configuration to send a test packet.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view or the specific integration data view.
3. Enter the following KQL filter: `data_stream.dataset : "cisco_ise.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `cisco_ise.log`)
   - `source.ip` (should reflect the IP of your ISE node)
   - `event.outcome` (e.g., `success` or `failure`)
   - `cisco_ise.mnemonic` (e.g., `Authentication-Succeeded` or `Config-Change-Detected`)
   - `message` (containing the raw syslog payload from ISE)
5. Navigate to **Analytics > Dashboards** and search for "Cisco ISE" to view pre-built visualizations for authentication trends and system health.

# Troubleshooting

## Common Configuration Issues

- **Firewall Blocking Traffic**: If no logs appear in Kibana, verify that the Elastic Agent host's OS-level firewall (e.g., `iptables` or Windows Firewall) and any physical firewalls are allowing traffic on the configured UDP/TCP port from the ISE node IPs.
- **Truncated Log Messages**: If logs appear garbled or fields are missing, check the **Maximum Length** setting in the Cisco ISE Remote Logging Target. If it is set to the default (1024), long ISE messages will be cut off. Increase this to `8192`.
- **Target Status Down**: In the Cisco ISE UI, check the status of the Remote Logging Target. If ISE cannot reach the Elastic Agent, it may mark the target as "Down." Use `ping` or `telnet` (for TCP) from the ISE CLI to test connectivity to the Agent.
- **Incorrect Port/Protocol**: Ensure the Port and Protocol (TCP/UDP) configured in the Kibana integration policy exactly match the settings in the ISE "Remote Logging Target" UI.

## Ingestion Errors

- **Parsing Failures**: If logs appear in Discover but have a `_grokparsefailure` or `_jsonparsefailure` tag, the ISE log format may not be using the expected header. Verify that ISE is using a standard logging template and not a custom one that modifies the Syslog header.
- **Timestamp Mismatches**: If logs appear to be "from the future" or are missing from the current time range, check for timezone discrepancies between the Cisco ISE nodes and the Elastic Agent host. Configure the `Timezone` setting in the Kibana integration UI if necessary.
- **Field Mapping Issues**: If specific fields like `source.ip` are missing, check the `error.message` field in the log record in Kibana to see if the ingest pipeline encountered an error during processing.

## Vendor Resources

- [Configure External Syslog Server on ISE - Cisco](https://www.cisco.com/c/en/us/support/docs/security/identity-services-engine/222223-configure-external-syslog-server-on-ise.html)
- [Syslogs in Cisco ISE - Netenrich](https://support.netenrich.com/hc/en-us/articles/16299079340061-Syslogs-in-Cisco-ISE)

# Documentation sites

- [Configure External Syslog Server on ISE - Cisco](https://www.cisco.com/c/en/us/support/docs/security/identity-services-engine/222223-configure-external-syslog-server-on-ise.html)
- Refer to the official vendor website for additional resources.
