# Service Info

## Common use cases

The Cisco Secure Email Gateway (formerly Cisco Email Security Appliance or ESA) integration is designed to provide comprehensive visibility into email traffic and security events across your organization. By ingesting these logs into the Elastic Stack, security teams can monitor mail flow and detect potential threats in real-time.
- **Threat Detection and Analysis:** Identify and investigate malicious email activity, including phishing attempts, malware attachments, and spam campaigns targeting internal users.
- **Mail Flow Monitoring:** Track the delivery status of emails, identify bottlenecks in mail processing, and monitor sender reputation to ensure reliable communication.
- **Compliance and Auditing:** Maintain long-term records of email metadata and administrative actions to satisfy regulatory requirements and support forensic investigations.
- **Operational Troubleshooting:** Diagnose issues related to mail filters, content scanning rules, and delivery failures by analyzing detailed gateway logs.

## Data types collected

This integration can collect the following types of data from Cisco Secure Email Gateway devices:
- **Consolidated Event Logs:** High-level summary logs formatted in Common Event Format (CEF) that provide a unified view of email processing events.
- **Text Mail Logs:** Detailed, text-based logs containing comprehensive information about every message processed by the gateway, including sender, recipient, and filter results.
- **System Logs:** Operational logs detailing the health and status of the gateway appliance itself, including service restarts and hardware alerts.
- **Authentication Logs:** Records of administrative logins and access attempts to the Cisco Secure Email Gateway management interface.
- **Format Support:** Data is typically received via Syslog in either RFC 3164/5424 or CEF (Common Event Format) over UDP or TCP protocols.

## Compatibility

The **Cisco Secure Email Gateway** (ESA) integration is designed to support a wide range of appliance versions.
- Tested and supported with **AsyncOS versions 12.x through 15.x and higher**.
- Compatible with both physical hardware appliances and virtual instances (vESA).
- Requires the appliance to support **Consolidated Event Logs** and **Syslog Push** retrieval methods.

## Scaling and Performance

To ensure the Elastic Stack efficiently processes high volumes of email log data, consider the following performance guidelines:

- **Transport/Collection Considerations:** Use **TCP** for log delivery whenever possible to ensure reliable transmission. While UDP is faster and has lower overhead, it lacks delivery confirmation, which can lead to data loss during periods of network congestion or high email volume. If encrypted transport is required, enable TLS on both the Cisco gateway and the Elastic Agent listener.
- **Data Volume Management:** Utilize **Consolidated Event Logs** instead of standard mail logs where possible. Consolidated logs summarize an entire email transaction into a single log line, significantly reducing the total number of events processed without losing critical security metadata. Additionally, apply log level filtering on the Cisco appliance to exclude debug or informational events that do not provide security value.
- **Elastic Agent Scaling:** For high-traffic environments processing thousands of emails per minute, deploy multiple Elastic Agents behind a load balancer. A single Agent on a standard 4-CPU virtual machine can typically handle several thousand events per second (EPS), but distributed collection ensures high availability and horizontal scalability during traffic spikes.

# Set Up Instructions

## Vendor prerequisites

Before configuring the integration, ensure the following requirements are met on the Cisco Secure Email Gateway appliance:
- **Administrative Access:** You must have an account with "Cloud Administrator" or "System Administrator" privileges to modify log subscriptions.
- **Network Connectivity:** The gateway must have an outbound network path to the Elastic Agent's IP address on the designated syslog port (e.g., 514, 5514).
- **Feature Licensing:** Ensure that the necessary security licenses (e.g., Anti-Spam, Sophos/McAfee Anti-Virus, DLP) are active, as these provide the data populated in the logs.
- **IP Information:** Have the IP address or fully qualified domain name (FQDN) of the Elastic Agent host ready for configuration.
- **Certificate Management:** If using TLS for syslog, ensure a valid CA-signed certificate is available for the appliance.

## Elastic prerequisites

- **Elastic Agent:** An Elastic Agent must be installed and enrolled in a policy via Fleet.
- **Integration Installed:** The "Cisco Secure Email Gateway" integration must be added to the Agent's policy.
- **Network Access:** Firewall rules must allow inbound traffic to the Elastic Agent on the configured Syslog port and protocol from the Cisco gateway's IP address.

## Vendor set up steps

### For Syslog Push (CEF or Text Mail) Collection:
1. Log in to the Cisco Secure Email Gateway GUI using your administrative credentials.
2. Navigate to the **System Administration** menu and select **Log Subscriptions**.
3. Click the **Add Log Subscription** button to start the configuration wizard.
4. Select the **Log Type** you wish to export. For the best compatibility with Elastic's SIEM features, select **Consolidated Event Logs**. Alternatively, you may select **Text Mail Logs**.
5. Provide a descriptive **Subscription Name**, such as `Elastic_Syslog_Export`.
6. Under the **Retrieval Method** section, select the radio button for **Syslog Push**.
7. Configure the syslog handler parameters:
    - **Hostname**: Enter the IP address or FQDN of the server hosting the Elastic Agent.
    - **Protocol**: Select `TCP` (recommended) or `UDP` to match your Elastic Agent configuration.
    - **Port**: Enter the port number the Elastic Agent is listening on (e.g., `514` or `5140`).
    - **Facility**: Choose a syslog facility code, such as `Local1`.
    - **Maximum Message Size**: Set this value appropriately (e.g., `65535` for TCP or `9216` for UDP).
8. If using TCP, optionally enable **TLS** and select the appropriate client certificate if encryption is required for your compliance standards.
9. Click **Submit** to save the log subscription settings.
10. Click the **Commit Changes** button in the upper-right corner of the GUI and enter a comment to finalize the deployment.

## Kibana set up steps

### For Consolidated Event Logs:
1. In Kibana, navigate to **Management > Integrations**.
2. Search for **Cisco Secure Email Gateway** and select the integration tile.
3. Click **Add Cisco Secure Email Gateway**.
4. Select the **Agent Policy** where you want to add this integration.
5. Under the **Consolidated Logs** section (or the relevant Syslog input), configure the following:
    *   **Syslog Host:** Set this to `0.0.0.0` to listen on all interfaces or the specific IP of the Agent host.
    *   **Syslog Port:** Enter the port number configured on the Cisco appliance (e.g., `514` or `1514`).
    *   **Protocol:** Select the protocol (`udp` or `tcp`) that matches your vendor configuration.
6. Expand **Advanced options** if you need to adjust internal queues or set a specific internal timezone if the appliance logs do not include UTC offsets.
7. Click **Save and continue**.
8. Click **Add Elastic Agent to your hosts** if you haven't already, or **Save and deploy changes** to update the existing policy.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from Cisco Secure Email Gateway to the Elastic Stack.

### 1. Trigger Data Flow on Cisco Secure Email Gateway:
- **Generate Message Event:** Send a test email through the gateway from an external address to an internal recipient to trigger a "Consolidated Event" entry.
- **Trigger Configuration Event:** Navigate to **System Administration > Configuration File** and perform a "Display" or "Download" action, or simply log out and log back in to generate an authentication/audit event.
- **Test Syslog Connection:** Use the CLI on the Cisco ESA and run the command `logconfig` -> `test` (if available in your version) to verify connectivity to the remote Syslog server.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view or the specific integration data view.
3. Enter the following KQL filter: `data_stream.dataset : "cisco_secure_email_gateway.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `cisco_secure_email_gateway.log`)
   - `source.ip` (the IP address of the email sender)
   - `event.action` (the action taken by the gateway, e.g., "DELIVERED" or "QUARANTINED")
   - `cisco.esa.mid` (the Message ID assigned by the Cisco gateway)
   - `message` (containing the raw CEF payload from the appliance)
5. Navigate to **Analytics > Dashboards** and search for "Cisco Secure Email Gateway" to view pre-built visualizations for email traffic and threat telemetry.

# Troubleshooting

## Common Configuration Issues

- **Uncommitted Changes**: In the Cisco GUI, changes to log subscriptions do not take effect until the **Commit Changes** button is clicked and the configuration is saved to the appliance.
- **Port Conflicts**: Ensure that no other service is using the designated syslog port on the Elastic Agent host. You can use `netstat -tulpn | grep <port>` on Linux to verify port availability.
- **Firewall Blocking**: Check intermediate firewalls or host-based firewalls (like `iptables` or `firewalld`) to ensure they allow traffic from the Cisco Gateway IP to the Elastic Agent IP on the specified port/protocol.
- **Protocol Mismatch**: A common error is configuring the Cisco Gateway for TCP while the Elastic Agent is set to UDP, or vice versa. Ensure these values match exactly in both configurations.

## Ingestion Errors

- **Parsing Failures**: If logs appear in Kibana but contain a `log.flags: [ "dissect_parsing_error" ]` or similar tag, the log format sent by the gateway (e.g., Text Mail) might not match the expected CEF format. Ensure the Cisco Log Subscription is set to the correct Log Type.
- **Message Truncation**: If logs are incomplete, check the **Maximum Message Size** in the Cisco Log Subscription. For complex CEF logs, this may need to be increased to the maximum allowable value (65535 for TCP).
- **Timezone Inconsistency**: If logs appear with the wrong timestamp, verify the time and timezone settings on both the Cisco Secure Email Gateway and the Elastic Agent host.

## Vendor Resources

- [User Guide for AsyncOS 16.0.2 for Cisco Secure Email Cloud Gateway - Logging](https://www.cisco.com/c/en/us/td/docs/security/ces/ces_16-0-2/user_guide/b_ESA_Admin_Guide_ces_16-0-2/b_ESA_Admin_Guide_12_1_chapter_0100111.html)
- [User Guide for AsyncOS 16.0 for Cisco Secure Email Gateway - GD (General Deployment)](https://www.cisco.com/c/en/us/td/docs/security/esa/esa16-0/user_guide/b_ESA_Admin_Guide_16-0.html)

# Documentation sites

- [User Guide for AsyncOS 16.0.2 for Cisco Secure Email Cloud Gateway - Logging](https://www.cisco.com/c/en/us/td/docs/security/ces/ces_16-0-2/user_guide/b_ESA_Admin_Guide_ces_16-0-2/b_ESA_Admin_Guide_12_1_chapter_0100111.html)
- [User Guide for AsyncOS 16.0 for Cisco Secure Email Gateway - GD (General Deployment)](https://www.cisco.com/c/en/us/td/docs/security/esa/esa16-0/user_guide/b_ESA_Admin_Guide_16-0.html)
- Refer to the official vendor website for additional resources.
