# Service Info

The Imperva integration allows for the seamless ingestion of security and system logs from Imperva SecureSphere into the Elastic Stack. This integration provides visibility into web application security, database activity, and management server health, enabling centralized monitoring and rapid incident response.

## Common use cases
The Imperva integration for Elastic allows organizations to ingest and analyze critical security data from Imperva SecureSphere Web Application Firewalls (WAF). This visibility is essential for monitoring web traffic, identifying malicious activity, and ensuring compliance across the application stack.

*   **Threat Detection and Monitoring:** Monitor real-time security violations and aggregated alerts to identify SQL injection, Cross-Site Scripting (XSS), and other OWASP Top 10 threats targeted at web applications. By centralizing these logs, security teams can correlate Imperva alerts with other infrastructure logs.
*   **Security Auditing and Compliance:** Collect detailed system events and security logs to maintain an audit trail for compliance frameworks such as PCI-DSS, HIPAA, and GDPR. This includes tracking administrative access and configuration changes on the Imperva MX.
*   **Incident Response and Forensics:** Utilize aggregated security alerts and detailed violation logs to conduct deep-dive investigations into security incidents. Analysts can trace the lifecycle of an attack from the initial probe to the blocked violation.
*   **Operational Health Monitoring:** Track SecureSphere system events to monitor the health and performance of the MX management server and gateways. This ensures that the security infrastructure itself is performing optimally and that logging services remain operational.

## Data types collected

This integration collects logs from Imperva SecureSphere using multiple input methods to accommodate different architectural requirements. Each method populates the **Imperva SecureSphere logs** (`securesphere`) data stream.

The following data types and streams are collected:
- **Security Violations:** Detailed logs of individual security policy breaches identified by the Imperva Gateway.
- **Security Alerts:** Aggregated views of related violations, providing a higher-level summary of security incidents.
- **System Events:** Internal logs related to the operation of the SecureSphere Management Server (MX) and its managed Gateways.
- **Data Formats:** All logs are collected in the Common Event Format (CEF) to ensure structured ingestion and field mapping.

According to the integration manifest, the following collection streams are available:
- **Imperva SecureSphere logs (TCP):** Collect Imperva SecureSphere logs via TCP input. This is the preferred method for reliable, connection-oriented log delivery.
- **Imperva SecureSphere logs (UDP):** Collect Imperva SecureSphere logs via UDP input. This provides a low-overhead method for high-volume environments where occasional packet loss is acceptable.
- **Imperva SecureSphere logs (Filestream):** Collect Imperva SecureSphere logs via Filestream input. This method is used when logs are written to local files on the Elastic Agent host or a mounted network share.

## Compatibility

The Imperva integration is compatible with the following versions:
- **Imperva SecureSphere**: Supported on versions **v14.7** and **v15.0**.
- **Elastic Stack**: Requires Kibana version **^8.11.0** or **^9.0.0**.
- **Subscription**: This integration is available with a **Basic** Elastic subscription.

## Scaling and Performance

To ensure optimal performance in high-volume environments, consider the following:
*   **Transport/Collection Considerations:** Users can choose between TCP, UDP, or Filestream based on their architecture. **TCP** (port `9507`) is recommended for reliable delivery of security events, ensuring no data loss during network congestion. **UDP** (port `9507`) provides lower overhead and higher throughput but does not guarantee delivery. **Filestream** should be utilized when logs are persisted to a local disk or an intermediate log aggregator before ingestion by the Elastic Agent.
*   **Data Volume Management:** To manage high volumes of log data, use Imperva's "Action Sets" to filter events at the source. Configure the vendor appliance to forward only necessary events, such as high-severity alerts. Avoid forwarding low-level debug logs unless troubleshooting, as they can significantly increase the ingest load and storage requirements on the Elastic Stack.
*   **Elastic Agent Scaling:** For high-throughput environments, deploy multiple Elastic Agents behind a network load balancer to distribute traffic evenly across multiple TCP/UDP listeners. Place Agents close to the data source to minimize network latency and use dedicated Agent policies for different data streams to improve processing efficiency.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have administrative credentials for the Imperva SecureSphere Management Server (MX).
- **Network Connectivity:** Ensure that the Imperva Gateway and Management Server can reach the Elastic Agent host on the configured port (default is `9507` for both TCP and UDP).
- **Licensing:** A valid license for SecureSphere WAF or Database Security that supports external log forwarding must be active.
- **Knowledge Requirements:** You will need the IP address of the Elastic Agent host and an understanding of your internal syslog facility requirements (e.g., `Local7`).

## Elastic prerequisites
1.  **Agent Installation:** Elastic Agent must be installed on a host reachable by the Imperva MX and Gateways.
2.  **Fleet Enrollment:** The Elastic Agent must be enrolled in Fleet and have a policy that includes the Imperva integration.
3.  **Host Firewall:** Ensure the firewall on the Elastic Agent host allows incoming traffic on the configured port (e.g., `9507`).
4.  **Version Check:** Ensure Kibana and Elasticsearch are at version `8.11.0` or higher.

## Vendor set up steps

### For Syslog (TCP/UDP) Collection:
1.  Log in to the **Imperva SecureSphere Management (MX)** interface.
2.  Navigate to **Policies > Action Sets**.
3.  In the **Action Sets** pane, click **Create** and select **Custom Action Set**.
4.  Define the Action Set (e.g., `Elastic-Syslog-Security-Alerts`):
    *   Select the **Gateway Security System Log** > **Gateway log security event to system log (syslog)** interface.
    *   Set **Syslog Host** to the IP of the Elastic Agent.
    *   Set **Syslog Port** to the port configured in Kibana (default `9507`).
    *   Set **Format** to `CEF`.
5.  For **Security Alerts (Violations)**, use the following CEF template in the **Message** field:
    `CEF:0|Imperva Inc.|SecureSphere|[Version]|${Alert.alertType}|${Alert.alertMetadata.alertName}|${Alert.severity}|act=${Alert.immediateAction} dst=${Event.destInfo.serverIp} dpt=${Event.destInfo.serverPort} duser=${Alert.username} src=${Event.sourceInfo.sourceIp} spt=${Event.sourceInfo.sourcePort} proto=${Event.sourceInfo.ipProtocol} rt=#arcsightDate (${Alert.createTime}) cat=Alert cs1=${Rule.parent.displayName} cs1Label=Policy cs2=${Alert.serverGroupName} cs2Label=ServerGroup cs3=${Alert.serviceName} cs3Label=ServiceName cs4=${Alert.applicationName} cs4Label=ApplicationName cs5=${Alert.description} cs5Label=Description`
6.  Navigate to **Policies > Security**, select your policy, and find the **Followed Actions** section.
7.  Add the Action Set created in step 4 to the **Followed Actions** list.
8.  For **System Events**, create a new Action Set using the **Server System Log** > **Log system event to system log (syslog)** interface.
9.  Apply the System Event Action Set to the **System Events Policy** under **Policies > Global**.

### For Logfile Collection:
1.  Configure Imperva to write logs to a local directory on the gateway or MX.
2.  Ensure the Elastic Agent has read permissions to the directory where log files are stored.
3.  Note the absolute path (e.g., `/var/log/imperva/securesphere.log`) for use in the Kibana configuration.

## Kibana set up steps

1. In Kibana, navigate to **Management > Integrations**.
2. Search for **Imperva** and select it.
3. Click **Add Imperva**.
4. Configure the desired inputs by expanding the relevant sections and filling in the configuration variables.
5. Select the **Existing Policy** or **New Policy** to add the integration to.
6. Click **Save and continue** to deploy the configuration to your Elastic Agents.

### Collecting logs from Imperva SecureSphere via TCP.
Select this input if you have configured a TCP Action Interface in Imperva. This input collects Imperva SecureSphere logs via TCP input.
- **Listen Address** (`listen_address`): The bind address to listen for TCP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
- **Listen Port** (`listen_port`): The TCP port number to listen on. Default: `9507`.
- **Timezone Offset** (`tz_offset`): When interpreting syslog timestamps without a time zone, use this timezone offset. Datetimes recorded in logs are by default interpreted in relation to the timezone set up on the host where the agent is operating. Both a canonical ID (such as "Europe/Amsterdam") and an HH:mm differential (such as "-05:00") are acceptable formats.
- **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.

### Collecting logs from Imperva SecureSphere via UDP.
Select this input if you have configured a UDP Action Interface in Imperva. This input collects Imperva SecureSphere logs via UDP input.
- **Listen Address** (`listen_address`): The bind address to listen for UDP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
- **Listen Port** (`listen_port`): The UDP port number to listen on. Default: `9507`.
- **Timezone Offset** (`tz_offset`): Use this parameter to adjust the timezone offset when importing logs from a host in a different timezone so that datetimes are appropriately interpreted.
- **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.

### Collecting logs from Imperva SecureSphere via File.
Select this input to crawl local log files using the filestream input. This input collects Imperva SecureSphere logs via Filestream input.
- **Paths** (`paths`): A list of glob-based paths that will be crawled and fetched. Example: `/var/log/imperva/*.log`.
- **Timezone Offset** (`tz_offset`): Use this parameter to appropriately interpret datetimes from hosts in different timezones.
- **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on Imperva:
- **Generate authentication event:** Log out and log back into the Imperva SecureSphere Management Server (MX) administration interface to trigger a system audit event.
- **Generate web traffic:** Browse several websites or applications protected by the Imperva Gateway to generate standard traffic logs.
- **Trigger security violation:** Attempt to access a restricted path or use a basic SQL injection string (e.g., `' OR 1=1 --`) on a protected non-production application to trigger a security violation and followed action.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "imperva.securesphere"`
4. Verify logs appear. Expand a log entry and confirm these fields:
   - `event.dataset` (should be `imperva.securesphere`)
   - `source.ip` and/or `destination.ip`
   - `event.action` or `event.outcome`
   - `message` (the raw log payload)
5. Navigate to **Analytics > Dashboards** and search for "Imperva" to view the pre-built visualizations.

# Troubleshooting

## Common Configuration Issues
- **Port Conflict**: If the Elastic Agent fails to start the Syslog listener, check if another service is already using port 9507 by running `netstat -tulpn | grep 9507`. Change the port in both Kibana and Imperva MX if a conflict exists.
- **Network Connectivity**: If logs are not arriving, verify that the Imperva Gateway can reach the Elastic Agent host. Use `tcpdump -i any port 9507` on the Agent host to check if packets are arriving at the network interface.
- **Policy Not Applied**: Logs will not be sent if the Action Set is created but not assigned as a "Followed Action" to an active Security Policy. Ensure the policy is applied and deployed to the relevant gateways.
- **CEF Format Errors**: If logs are arriving but not being parsed, ensure the "Message" template in the Imperva Action Set exactly matches the CEF structure required by the integration.

## Ingestion Errors
- **Parsing Failures**: If logs appear in Kibana with a `tags: ["_syslog_parse_failure"]` or `tags: ["_cef_parse_failure"]`, verify that the Imperva version in the CEF header is correctly formatted and that no trailing characters are being added by the syslog relay.
- **Timezone Mismatch**: If events appear in the "future" or "past," check the `tz_offset` variable in the Kibana integration settings to ensure it matches the timezone of the Imperva Management Server.
- **Mapping Issues**: Check the `error.message` field in Discover for any mapping conflicts. This can happen if a custom field in the CEF message conflicts with an existing Elastic Common Schema (ECS) field definition.

## Vendor Resources

- [Imperva v15.0: Working with Action Sets and Followed Actions](https://docs.imperva.com/bundle/v15.0-waf-management-server-manager-user-guide/page/Working_with_Action_Sets_and_Followed_Actions.htm)
- [Imperva v14.7: Alerts, Violations, and Events](https://docs-cybersec.thalesgroup.com/bundle/v14.7-waf-user-guide/page/1024.htm)
- [Imperva v14.7: Logging System Events for Auditing](https://docs-cybersec.thalesgroup.com/bundle/v14.7-waf-administration-guide/page/58987.htm)

# Documentation sites

- [Imperva v15.0: Working with Action Sets and Followed Actions](https://docs.imperva.com/bundle/v15.0-waf-management-server-manager-user-guide/page/Working_with_Action_Sets_and_Followed_Actions.htm)
- [Imperva v14.7: Alerts, Violations, and Events](https://docs-cybersec.thalesgroup.com/bundle/v14.7-waf-user-guide/page/1024.htm)
- [Imperva v14.7: Logging System Events for Auditing](https://docs-cybersec.thalesgroup.com/bundle/v14.7-waf-administration-guide/page/58987.htm)
- Refer to the official vendor website for additional resources.
