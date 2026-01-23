# Service Info

## Common use cases

The Cisco ISE integration allows organizations to centralize network access control data and security events into the Elastic Stack for real-time monitoring and analysis.
- **Security Auditing and Compliance:** Track every authentication request across the network to maintain an audit trail for regulatory compliance such as PCI-DSS or HIPAA.
- **Threat Detection and Response:** Identify failed authentication attempts and potential brute-force attacks by monitoring "Failed Attempts" and "Radius Accounting" logs.
- **Network Visibility and Troubleshooting:** Monitor user session activity and device health to quickly diagnose connectivity issues for specific users or endpoints.
- **Policy Verification:** Ensure that Cisco ISE policies are being applied correctly by analyzing "Passed Authentications" and checking if users are assigned to the correct VLANs or SGTs.

## Data types collected

This integration can collect the following types of data from Cisco ISE via Syslog or direct file ingestion:
- **Authentication Logs:** Detailed records of passed and failed authentication requests including username, endpoint MAC, and policy results.
- **Accounting Logs:** RADIUS accounting data providing information on session duration, data usage, and session start/stop events.
- **System Health and Audit Logs:** Logs relating to the Cisco ISE appliance performance, configuration changes made by administrators, and process status.
- **Profiling Logs:** Information about the types of devices connecting to the network based on profiling probes.

According to the data stream summaries, the following streams are available:
- **Cisco_ISE logs (logs):** Collect Cisco ISE logs via TCP input. This is the primary stream for network-based ingestion requiring reliable delivery.
- **Cisco_ISE logs (logs):** Collect Cisco ISE logs via UDP input. This stream is used for high-throughput, non-guaranteed delivery scenarios.
- **Cisco_ISE logs (logs):** Collect Cisco ISE logs via file input. This stream is used when the Elastic Agent has direct access to the filesystem where logs are stored.

## Compatibility

This integration is compatible with **Cisco Identity Services Engine (ISE)**. It has been specifically tested and validated against **version 3.1.0.518**. For optimal parsing and field mapping, ensure the ISE syslog format matches the standards expected by the version 3.x series.

**Elastic Requirements:**
- **Kibana version:** ^8.11.0 || ^9.0.0
- **Elastic Agent:** Must be enrolled in Fleet.

## Scaling and Performance

To ensure optimal performance in high-volume Cisco ISE environments, consider the following:
- **Transport/Collection Considerations:** For most deployments, TCP is recommended over UDP to ensure delivery reliability and to prevent log loss during periods of high network congestion. When using the TCP input, ensure that the network path between Cisco ISE and the Elastic Agent supports persistent connections. If low latency is prioritized over reliability, the UDP input can be used, though packet loss may occur under heavy load.
- **Data Volume Management:** To manage high event volumes, administrators should navigate to **Logging Categories** in Cisco ISE and select only the essential log categories (e.g., Passed Authentications, AAA Audit) for export. Filtering data at the source reduces the processing load on the Elastic Agent and minimizes storage costs in Elasticsearch by excluding verbose or repetitive system messages.
- **Elastic Agent Scaling:** For high-throughput environments (thousands of authentications per second), deploy multiple Elastic Agents behind a network load balancer to distribute traffic evenly. Place Agents close to the Cisco ISE Policy Service Nodes (PSNs) to minimize latency. Ensure the host running the Agent has sufficient CPU and memory resources to handle the regex-heavy parsing required for complex Cisco ISE syslog messages.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have Super Admin or System Admin privileges in the Cisco ISE Portal to configure Remote Logging Targets and Logging Categories.
- **Network Connectivity:** The Cisco ISE nodes must be able to reach the Elastic Agent host over the network via the configured ports (default `9025` for TCP or `9026` for UDP).
- **Log Formatting Knowledge:** Understanding of Cisco ISE facility codes (e.g., Local6, Local7) is required to ensure logs do not conflict with existing syslog infrastructure.
- **Firewall Rules:** Internal firewalls must be configured to allow traffic from the Cisco ISE Policy Service Nodes (PSNs) to the Elastic Agent's listening IP and port.

## Elastic prerequisites

- **Kibana Version:** This integration requires Kibana version `8.11.0` or higher.
- **Elastic Agent Status:** An Elastic Agent must be installed and enrolled in a policy via Fleet.
- **Input Configuration:** The specific Cisco ISE integration must be added to the Agent policy with the correct input type (TCP, UDP, or Filestream) enabled.
- **Connectivity:** The Elastic Agent must be listening on the specified network interface and port to receive incoming syslog traffic from the ISE nodes.

## Vendor set up steps

### For Syslog (TCP/UDP) Collection:

To configure Cisco ISE to send logs to the Elastic Agent, follow these steps to define the remote destination and assign data categories.

1. Log in to your **Cisco ISE Administration Portal**.
2. Navigate to **Administration > System > Logging > Remote Logging Targets**.
3. Click **Add** to create a new remote logging destination.
4. Configure the target with the following parameters:
    *   **Name**: Enter a descriptive name, such as `Elastic_Agent_Target`.
    *   **Target Type**: Select `UDP Syslog` or `TCP Syslog`. This **must** match the protocol you select in the Kibana integration settings.
    *   **IP Address**: Enter the IP address of the server where the Elastic Agent is installed.
    *   **Port**: Enter the port number. By default, use `9025` for TCP or `9026` for UDP.
    *   **Facility Code**: Choose an appropriate syslog facility (e.g., `Local6`).
    *   **Maximum Length**: **Crucially, set this value to 8192**. Values lower than this will cause log segmentation, which breaks the integration's ability to parse fields correctly.
5. Click **Submit** to save the target configuration and verify it appears in the Remote Logging Targets list.
6. Navigate to **Administration > System > Logging > Logging Categories**.
7. For each category you wish to monitor (e.g., `AAA Audit`, `Passed Authentications`, `Failed Attempts`), click the radio button next to the category and click **Edit**.
8. In the **Targets** section, move your newly created target (`Elastic_Agent_Target`) from the **Available** column to the **Selected** column.
9. Click **Save** for each category. Repeat this for all relevant log types to begin the data flow.

## Kibana set up steps

### Collecting Cisco ISE logs via TCP input.
1. In Kibana, navigate to **Integrations** and search for **Cisco ISE**.
2. Click **Add Cisco ISE** and select the **Collecting Cisco ISE logs via TCP input** input type.
3. Configure the following variables:
   - **Listen Address**: The bind address to listen for TCP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
   - **Listen Port**: The TCP port number to listen on. Default: `9025`.
   - **Preserve original event**: Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
   - **Timezone Offset**: When interpreting syslog timestamps without a time zone, use this timezone offset (e.g., "Europe/Amsterdam" or "-05:00").
   - **SSL Configuration**: Provide YAML configuration for SSL if using encrypted transport.
4. Save the integration to the desired Elastic Agent policy.

### Collecting Cisco ISE logs via UDP input.
1. In Kibana, navigate to **Integrations** and search for **Cisco ISE**.
2. Click **Add Cisco ISE** and select the **Collecting Cisco ISE logs via UDP input** input type.
3. Configure the following variables:
   - **Listen Address**: The bind address to listen for UDP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
   - **Listen Port**: The UDP port number to listen on. Default: `9026`.
   - **Preserve original event**: Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
   - **Timezone Offset**: Use this parameter to adjust the timezone offset when importing logs from a host in a different timezone.
4. Save the integration to the desired Elastic Agent policy.

### Collecting Cisco ISE logs using filestream input.
1. In Kibana, navigate to **Integrations** and search for **Cisco ISE**.
2. Click **Add Cisco ISE** and select the **Collecting Cisco ISE logs using filestream input** input type.
3. Configure the following variables:
   - **Paths**: The list of file paths to monitor for Cisco ISE logs. Default: `['/var/log/cisco_ise*']`.
   - **Preserve original event**: Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
   - **Timezone Offset**: Use this parameter to adjust the timezone offset for logs without explicit timezone data.
4. Save the integration to the desired Elastic Agent policy.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from Cisco ISE to the Elastic Stack.

### 1. Trigger Data Flow on Cisco ISE:
- **Generate authentication event:** Attempt to log into the Cisco ISE Administrator Portal with an existing account to generate a "Passed Authentication" or "Administrative Audit" log.
- **Generate failed login event:** Attempt to log into the Cisco ISE portal with a non-existent username to trigger a "Failed Attempt" event.
- **Generate configuration event:** Navigate to any system setting, change a minor description or toggle a non-critical setting, and click **Save** to generate an operational audit log.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "cisco_ise.log"`
4. Verify logs appear. Expand a log entry and confirm these fields:
   - `event.dataset` (should be `cisco_ise.log`)
   - `source.ip` (the IP of the ISE node or the client)
   - `event.outcome` (e.g., `success` or `failure`)
   - `cisco_ise.mnemonic` (e.g., `Passed-Authentications` or `Failed-Attempt`)
   - `message` (the raw log payload)
5. Navigate to **Analytics > Dashboards** and search for "Cisco ISE" to visualize the incoming data.

# Troubleshooting

## Common Configuration Issues

- **Log Message Truncation**: If fields are missing or the `message` field appears cut off, check that the **Maximum Length** in the Cisco ISE Remote Logging Target is set to **8192**. Cisco ISE defaults to 1024, which is insufficient for many RADIUS payloads.
- **Network Connectivity/Firewalls**: If no logs are appearing, verify that the Cisco ISE node can reach the Elastic Agent IP on the specified port. Use `tcpdump` or `nc -l -u -p 9026` on the Agent host to verify packets are arriving.
- **Logging Category Not Assigned**: A common mistake is creating the Remote Logging Target but forgetting to assign it to specific **Logging Categories**. Navigate to **Administration > System > Logging > Logging Categories** and ensure your target is in the "Selected" column for the desired logs.
- **Port Conflict**: If the Elastic Agent fails to start the input, check the Agent logs for "address already in use" errors. Ensure no other service or another integration is using the same port.

## Ingestion Errors

- **Parsing Failures**: If logs appear in Kibana but contain the `_grokparsefailure` or `_jsonparsefailure` tags, check if the syslog header format has been modified in Cisco ISE. The integration expects the standard Cisco ISE syslog format.
- **Timezone Mismatches**: If logs appear to be from the future or the past, ensure the `tz_offset` variable in the Kibana configuration matches the timezone configured on your Cisco ISE nodes.
- **Field Mapping Issues**: If `event.original` is present but specific fields like `user.name` are missing, verify that the log message hasn't been segmented by an intermediate syslog relay or load balancer.

## Vendor Resources

- [Configure External Syslog Server on ISE - Cisco](https://www.cisco.com/c/en/us/support/docs/security/identity-services-engine/222223-configure-external-syslog-server-on-ise.html)
- [Cisco ISE Syslogs - Introduction to Cisco ISE Syslogs](https://www.cisco.com/c/en/us/td/docs/security/ise/syslog/Cisco_ISE_Syslogs/m_IntrotoSyslogs.html)
- [Official Cisco ISE Syslog Documentation](https://www.cisco.com/c/en/us/td/docs/security/ise/syslog/Cisco_ISE_Syslogs/m_SyslogsList.html)

# Documentation sites

- [Cisco ISE Product Page](https://www.cisco.com/site/us/en/products/security/identity-services-engine/index.html)
- [Cisco ISE Syslog Messages List](https://www.cisco.com/c/en/us/td/docs/security/ise/syslog/Cisco_ISE_Syslogs/m_SyslogsList.html)
- Refer to the official vendor website for additional configuration and administrative guides.
