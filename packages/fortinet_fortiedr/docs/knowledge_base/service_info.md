# Service Info

## Common use cases

The Fortinet FortiEDR integration allows organizations to ingest high-fidelity endpoint security telemetry and system events directly into the Elastic Stack for centralized visibility and advanced threat hunting. By collecting logs from the FortiEDR Central Manager, security teams can correlate endpoint behavior with other network data sources.
- **Threat Detection and Incident Response:** Monitor real-time security events across all endpoints to identify malicious behavior, such as fileless attacks or ransomware execution, and respond using Elastic's alerting framework.
- **Compliance and Auditing:** Maintain a tamper-proof record of Audit Trail logs to track administrative changes, policy updates, and user activities within the FortiEDR management console for regulatory requirements.
- **System Health and Monitoring:** Collect System Events to monitor the operational status of FortiEDR components, ensuring that collectors are active and the Central Manager is functioning correctly across the environment.
- **Unified Security Analytics:** Aggregate FortiEDR data with firewall, identity, and cloud logs in Kibana to gain a holistic view of the attack surface and perform cross-platform correlation.

## Data types collected

This integration can collect the following types of data from the FortiEDR Central Manager via syslog:
- **Security Events:** Detailed logs regarding prevented threats, detected anomalies, and policy violations on monitored endpoints.
- **System Events:** Internal logs from the FortiEDR Central Manager and Aggregators regarding service status and system health.
- **Audit Trails:** Logs documenting changes to configurations, user logins, and modifications to security policies.
- **Data Formats:** The integration supports data forwarded in Common Event Format (CEF), Log Event Extended Format (LEEF), or Semicolon-separated (CSV) formats.
- **Transport Protocols:** Data is received via Syslog over UDP or TCP (with optional TLS encryption).

## Compatibility

This integration is compatible with **Fortinet FortiEDR** (and FortiXDR) versions 7.x and higher. The configuration steps provided are specifically validated for **FortiEDR Central Manager** version 7.2.0. Users should ensure their FortiEDR license includes support for external syslog forwarding.

## Scaling and Performance

- **Transport/Collection Considerations:** When configuring the connection, users must choose between TCP and UDP. TCP is recommended for high-reliability environments to ensure no security events are dropped during transit, especially when using TLS for encrypted communication. UDP may be used in high-throughput environments where low latency is prioritized over guaranteed delivery, though packet loss may result in missing security telemetry.
- **Data Volume Management:** To manage the load on the Elastic Stack, administrators should use the **Notifications** pane in the FortiEDR Central Manager to selectively enable only necessary event types (e.g., enabling "Security Events" while disabling "System Events"). Additionally, syslog forwarding is controlled at the policy level via Playbook policies; filtering data at this source ensures that only logs from critical endpoint groups are ingested.
- **Elastic Agent Scaling:** For high-volume environments managing thousands of endpoints, a single Elastic Agent may become a bottleneck. It is recommended to deploy multiple Elastic Agents behind a load balancer to distribute the incoming syslog traffic. Ensure that the Agent's host system has sufficient CPU and memory to handle the parsing of CEF or CSV formatted strings at scale.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have Super User or Administrator permissions in the FortiEDR Central Manager console to configure syslog destinations and modify Playbook policies.
- **Network Connectivity:** The FortiEDR Central Manager must have network line-of-sight to the Elastic Agent's IP address on the designated syslog port (e.g., TCP/UDP 9514).
- **Firewall Rules:** Ensure any intermediate firewalls allow traffic from the Central Manager to the Elastic Agent on the configured port.
- **License Requirements:** Ensure your FortiEDR license includes the ability to forward logs to external SIEM solutions.
- **Target Information:** Have the IP address of the Elastic Agent and the intended protocol (TCP or UDP) ready before beginning configuration.

## Elastic prerequisites

- **Elastic Agent:** An Elastic Agent must be installed and enrolled in a policy via Fleet.
- **Integration Installed:** The Fortinet FortiEDR integration must be added to the Agent policy.
- **Connectivity:** The Elastic Agent must be reachable by the FortiEDR Central Manager. If using TLS, the appropriate certificates must be available on the Agent host.

## Vendor set up steps

### For Syslog Collection:
1. Log in to your **FortiEDR Central Manager** console with administrative credentials.
2. Navigate to the **Administration** menu in the left-hand sidebar and select **Syslog**.
3. Click the **Add** button (represented by a `+` icon) to create a new syslog destination for the Elastic Agent.
4. In the configuration window, define the following settings:
    *   **Syslog Name**: Provide a unique identifier (e.g., `Elastic_Agent_Primary`).
    *   **Host**: Enter the IP address or FQDN of the server hosting the Elastic Agent.
    *   **Port**: Enter the port number configured in the Elastic Agent (e.g., `9514`).
    *   **Protocol**: Select either `TCP` or `UDP` to match your Elastic Agent configuration.
    *   **TLS**: Toggle this to enabled if you are using TCP and require encrypted transport (requires certificate configuration on the Agent).
    *   **Format**: Select `CEF` or `LEEF` for structured data. If these are not available, use the default `Semicolon` format.
5. Click the **Test** button to send a sample message to the Elastic Agent and verify the connection.
6. Once the test is successful, click **Save**.
7. In the list of syslog destinations, select the newly created row. A **Notifications** pane will appear on the right side.
8. Use the sliders in the Notifications pane to enable the specific data streams you wish to forward: **Security Events**, **System Events**, and **Audit Trail**.
9. **Important:** To ensure security events are actually forwarded, navigate to the **Playbook policies** section in the main menu.
10. Edit each relevant Playbook policy and ensure the **Send Syslog Notification** option is enabled. Without this step, security events generated by endpoints under those policies will not be sent to the syslog destination.

## Kibana set up steps

### For FortiEDR Log Collection:

1.  In Kibana, navigate to **Management > Integrations**.
2.  Search for **Fortinet FortiEDR** and select the integration tile.
3.  Click **Add Fortinet FortiEDR**.
4.  Under the **FortiEDR Logs** section, configure the following fields:
    *   **Syslog Host**: Set this to `0.0.0.0` to listen on all available interfaces, or a specific IP address of the Agent host.
    *   **Syslog Port**: Enter the same port number used in the FortiEDR Central Manager configuration (e.g., `9001`).
    *   **Protocol**: Select the protocol (TCP or UDP) to match the vendor configuration.
5.  (Optional) Expand the **Advanced options** to adjust internal queue sizes or to add custom tags to the incoming data.
6.  Select the **Existing host policy** where your Elastic Agent is enrolled.
7.  Click **Save and continue**, then click **Add agent policy** to deploy the configuration to the Agent.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from Fortinet FortiEDR to the Elastic Stack.

### 1. Trigger Data Flow on Fortinet FortiEDR:
- **Generate Audit Event:** Log out of the FortiEDR Central Manager and log back in, or modify a non-critical setting in the administration panel to trigger an audit log.
- **Generate System Event:** Restart a FortiEDR Collector service on a test endpoint or perform a connectivity test from the Central Manager to an endpoint.
- **Generate Security Event:** Use a benign test file or EICAR string on an endpoint managed by a Playbook policy with "Send Syslog Notification" enabled to trigger a security alert.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view or the specific integration data view.
3. Enter the following KQL filter: `data_stream.dataset : "fortinet_fortiedr.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `fortinet_fortiedr.log`)
   - `source.ip` and/or `observer.ip`
   - `event.action` or `event.category`
   - `fortinet_fortiedr.event_type` (e.g., Security Event, Audit)
   - `message` (containing the raw CEF/Syslog payload)
5. Navigate to **Analytics > Dashboards** and search for "Fortinet FortiEDR" to view pre-built visualizations and confirm data is being parsed correctly.

# Troubleshooting

## Common Configuration Issues

- **Playbook Notification Not Enabled**: Even if Syslog destinations are configured, logs will not be sent unless the "Send Syslog Notification" option is checked in the active Playbook policy under **Security Settings > Playbook**. Ensure the policy is correctly assigned to the Collector Groups you are monitoring.
- **Connection Test Failure**: If the "Test" button in Syslog Settings fails, check for firewall rules blocking the chosen port (TCP/UDP 514 or custom) between the Central Manager and the Elastic Agent. Verify that the Elastic Agent is actually running and listening on that port using `netstat` or `ss` commands on the host.
- **Incorrect Protocol/Port Mismatch**: Ensure the protocol (TCP vs UDP) and the port number configured in the FortiEDR Syslog Destination settings exactly match the values entered in the Kibana integration settings.
- **Notification Type Not Selected**: Ensure that the specific sliders for Security, System, and Audit events are toggled "On" in the Notifications pane of the Syslog Settings.

## Ingestion Errors

- **Parsing Failures**: If logs appear in Kibana but contain a `tags: ["_grokparsefailure"]` or similar error, verify that the "Format" selected in FortiEDR (CEF, LEEF, or Semicolon) matches what the integration expects.
- **Format Mismatch**: If you selected "Semicolon" in FortiEDR but the integration is expecting CEF, fields will not map correctly. Check the `error.message` field in Discover to see specific parsing error details.
- **Timezone Discrepancies**: If logs appear to be missing, check for timezone offsets. Ensure the FortiEDR Central Manager and the Elastic Agent host are synchronized via NTP, as timestamp mismatches can cause logs to appear in the "future" or "past" in Discover.

## Vendor Resources

- [Syslog | FortiEDR/XDR 7.0.0 | Fortinet Document Library](https://docs.fortinet.com/document/fortiedr/7.0.0/administration-guide/109591/syslog)
- [Syslog | FortiEDR/XDR 5.2.0 | Fortinet Document Library](https://docs.fortinet.com/document/fortiedr/5.2.0/administration-guide/109591/syslog)

# Documentation sites

- [Syslog | FortiEDR/XDR 7.0.0 | Fortinet Document Library](https://docs.fortinet.com/document/fortiedr/7.0.0/administration-guide/109591/syslog)
- [Syslog | FortiEDR/XDR 5.2.0 | Fortinet Document Library](https://docs.fortinet.com/document/fortiedr/5.2.0/administration-guide/109591/syslog)
- Refer to the official vendor website for additional resources.
