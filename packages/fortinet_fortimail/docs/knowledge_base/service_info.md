# Service Info

The Fortinet FortiMail integration allows for the seamless ingestion of logs from FortiMail secure email gateway appliances into the Elastic Stack. This integration provides critical visibility into email-based threats, mail flow performance, and administrative actions, enabling security teams to detect and respond to phishing, malware, and spam campaigns in real-time.

## Common use cases

The Fortinet FortiMail integration for Elastic Agent allows organizations to ingest and analyze comprehensive email security logs to improve threat detection and operational visibility.
- **Security Monitoring and Threat Detection:** Identify and respond to email-based threats such as phishing, malware, and business email compromise (BEC) by analyzing AntiVirus and AntiSpam logs in real-time.
- **Compliance and Audit Logging:** Maintain a detailed record of system events and administrative actions to satisfy regulatory requirements (e.g., GDPR, HIPAA, or PCI-DSS) and internal security policies.
- **Email Delivery Troubleshooting:** Utilize "History" logs to track the lifecycle of email messages, helping administrators diagnose delivery failures, latency issues, or routing errors across the mail gateway.
- **Operational Performance Analysis:** Monitor system health events to ensure the FortiMail appliance is operating within normal parameters and to proactively identify hardware or software issues.

## Data types collected

This integration can collect the following types of data from Fortinet FortiMail appliances:
- **Event Logs:** System-level events, including administrator logins, configuration changes, and system updates.
- **AntiSpam Logs:** Details on messages identified as spam, including the sender, recipient, and the specific antispam rule triggered.
- **AntiVirus Logs:** Information regarding detected viruses and malware in email attachments, including the virus name and the action taken (e.g., quarantined or deleted).
- **History/Content Logs:** Metadata regarding mail sessions and content filtering results for both inbound and outbound traffic.
- **Data Format:** All logs are collected via standard Syslog format.
- **Transport Protocols:** Supports ingestion via UDP or TCP protocols.

## Compatibility

The **Fortinet FortiMail** integration is designed to work with:
- **FortiMail Appliance and VM** versions 7.0, 7.2, 7.4, and 7.6 (tested specifically with version **7.6.3**).
- Earlier versions (6.x) may be compatible if they support standard remote syslog forwarding, though specific field mappings may vary.

## Scaling and Performance

- **Transport/Collection Considerations:** When configuring the connection between FortiMail and the Elastic Agent, users can choose between UDP, TCP, and TCP over TLS. UDP is optimized for high-speed transmission with lower overhead but lacks delivery guarantees. TCP is recommended for critical security environments to ensure no logs are dropped during transit. For environments requiring high security, TCP over TLS should be utilized to encrypt log data in transit between the FortiMail appliance and the Elastic Agent.
- **Data Volume Management:** To prevent overwhelming the Elastic Stack or the network, it is recommended to use the "Level" setting on the FortiMail appliance to filter out low-priority logs. Setting the logging level to "notification" or "warning" can significantly reduce volume by excluding routine "information" or "debug" events. Additionally, administrators should only enable specific logging categories (e.g., Mail Event, AntiSpam) that are necessary for their specific monitoring use cases.
- **Elastic Agent Scaling:** For high-volume environments processing millions of emails per day, a single Elastic Agent may become a bottleneck. In such scenarios, it is recommended to deploy multiple Elastic Agents behind a load balancer. This allows the Syslog traffic from FortiMail to be distributed across several agents, ensuring high availability and horizontal scalability of the ingestion pipeline.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have an administrator account with "Super User" or "Read-Write" permissions to the FortiMail web-based manager.
- **Network Connectivity:** Ensure that the FortiMail appliance can reach the Elastic Agent over the network. If a firewall exists between them, port 514 (or your custom configured port) must be open for UDP or TCP traffic.
- **Logging License:** Ensure the FortiMail appliance has the necessary feature set enabled to support remote syslog logging.
- **IP Information:** You will need the static IP address of the server or container where the Elastic Agent is currently running.
- **Facility Knowledge:** Identify an unused syslog facility (e.g., `local7`) to prevent log interleaving with other devices on the same agent.

## Elastic prerequisites

- **Elastic Agent Enrollment:** An Elastic Agent must be installed and successfully enrolled in a Fleet-managed environment or configured as a standalone agent.
- **Integration Installation:** The Fortinet FortiMail integration package must be installed within the Elastic environment via the Integrations app in Kibana.
- **Connectivity:** The Elastic Agent must be reachable from the FortiMail appliance and have an active connection to the Elastic Stack (Elasticsearch and Kibana).

## Vendor set up steps

### For Syslog (Remote Logging) Collection:

1.  Log in to the **Fortinet FortiMail** administration web interface using your administrator credentials.
2.  In the navigation menu, go to **Log and Report > Log Settings > Remote Log Settings**.
3.  Click the **New** button in the toolbar to create a new remote logging profile.
4.  In the **New Remote Log Setting** dialog, configure the following parameters:
    *   **Enabled:** Ensure the checkbox is **Selected** to activate the logging profile.
    *   **Profile name:** Enter a unique name for this target, such as `ElasticAgent_Syslog`.
    *   **IP:** Enter the IPv4 or IPv6 address of the host where your **Elastic Agent** is installed.
    *   **Port:** Enter the port number that the Elastic Agent is listening on (default is `514`).
    *   **Level:** Set the log level (e.g., `Information` or `Notice`) to determine the granularity of the logs forwarded.
    *   **Facility:** Select a facility identifier from the dropdown menu (e.g., `local7`). 
    *   **CSV format:** Ensure this checkbox is **Deselected**. The Elastic integration expects the standard syslog key-value format.
5.  Locate the **Logging Policy Configuration** section within the same window.
6.  Select the specific log types you wish to forward to Elastic by checking the boxes for:
    *   **Event** (System events)
    *   **AntiVirus** (Malware detection)
    *   **AntiSpam** (Spam filtering)
    *   **Content** (Content filtering)
7.  Click **Create** or **OK** to save the profile.
8.  Verify that the profile appears in the list and shows an active status.

## Kibana set up steps

### For Syslog Input:
1. Open Kibana and navigate to **Management > Integrations**.
2. Search for **Fortinet FortiMail** and select the integration.
3. Click **Add Fortinet FortiMail**.
4. Configure the integration settings:
   - Select **Syslog** as the input type if prompted.
   - Set the **Syslog Host** to `0.0.0.0` (to listen on all interfaces) or the specific IP of the Agent host.
   - Set the **Syslog Port** to match the value used in the FortiMail setup (e.g., `9514`).
   - Choose the **Protocol** (TCP or UDP) to match your FortiMail settings.
5. (Optional) Under **Advanced options**, you can adjust internal queue sizes or add custom tags to the incoming data.
6. Scroll down and click **Save and continue**.
7. Assign the integration to the appropriate **Agent Policy** and click **Save and deploy changes**.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from FortiMail to the Elastic Stack.

### 1. Trigger Data Flow on Fortinet FortiMail:
- **Generate Administrative Logs:** Log out of the FortiMail GUI and log back in. This will generate an "Admin activity" log entry for the login event.
- **Trigger Configuration Event:** Navigate to a non-critical setting (like a comment field in a policy) and make a minor change, then save it to generate a configuration change event.
- **Generate Mail Event:** If possible in a lab environment, send a test email through the FortiMail gateway to trigger a "Mail History" event.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view or the specific integration data view.
3. Enter the following KQL filter: `data_stream.dataset : "fortinet_fortimail.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `fortinet_fortimail.log`)
   - `log.syslog.facility.code` (should match the facility set in FortiMail, e.g., `23` for `local7`)
   - `event.action` (e.g., `login`, `update`, or `relay`)
   - `fortinet_fortimail.log.type` (should indicate `event`, `history`, `virus`, or `spam`)
   - `message` (containing the raw log payload starting with the FortiMail header)
5. Navigate to **Analytics > Dashboards** and search for "Fortinet FortiMail" to view pre-built visualizations and confirm data is populating the charts.

# Troubleshooting

## Common Configuration Issues

- **Incorrect Port or Protocol**: If no logs appear in Kibana, verify that the `Server port` and `Mode` in the FortiMail Remote Log settings exactly match the `Syslog Port` and `Protocol` configured in the Elastic integration.
- **Firewall Blockage**: Ensure that the network path between the FortiMail appliance and the Elastic Agent is open. Test connectivity using tools like `tcpdump` or `nc` (netcat) on the Elastic Agent host to see if packets are arriving on the specified port.
- **Status Not Enabled**: In the FortiMail GUI under **Log & Report > Log Setting > Remote**, ensure the **Status** checkbox for your remote logging profile is checked.
- **Log Level Too High**: If only critical errors are appearing, check the **Level** setting in the FortiMail configuration. If it is set to "Emergency" or "Alert," lower-priority logs like "Information" or "Notice" will be suppressed at the source.

## Ingestion Errors

- **Parsing Failures**: If logs appear in Kibana but contain the tag `_grokparsefailure` or `_jsonparsefailure`, the log format might have changed or contain unexpected characters. **Solution**: Inspect the `error.message` field in Discover to identify the specific part of the log that failed to parse and ensure the FortiMail firmware version is compatible.
- **Field Mapping Issues**: If data appears in the `message` field but not in specific fields like `source.ip`, the ECS mapping may not be applying. **Solution**: Ensure the integration is correctly installed and that the `data_stream.dataset` is exactly `fortinet_fortimail.log`.
- **Timezone Mismatch**: Logs may appear to be missing if the timestamps are mapped to the wrong timezone. **Solution**: Check the FortiMail system time settings and ensure the Elastic Agent host is synchronized with a reliable NTP source.

## Vendor Resources

- [Configuring logging | FortiMail Appliance and VM 7.6.3 | Fortinet Document Library](https://docs.fortinet.com/document/fortimail/7.6.3/administration-guide/332364/configuring-logging)

## Documentation sites

- [FortiMail Remote Logging Guide (Trellix Resource)](https://docs.trellix.com/bundle/enterprise-security-manager-data-source-configuration-guide/page/UUID-327ad1e3-dbde-eae9-81bd-467b9443718b.html)
- Refer to the official vendor website for additional product administration guides.
