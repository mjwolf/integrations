# Service Info

## Common use cases

The Common Event Format (CEF) integration is designed to ingest standardized security logs from a wide variety of third-party network and security appliances into the Elastic Stack. By normalizing data into the Elastic Common Schema (ECS), it enables centralized monitoring and advanced security analytics.

*   **Security Information and Event Management (SIEM):** Centralize logs from disparate security vendors (e.g., Palo Alto, Check Point, Fortinet) into a single dashboard for unified threat hunting and incident response. Use standardized field names like `source.ip` and `event.action` to build vendor-agnostic security alerts.
*   **Regulatory Compliance Auditing:** Maintain long-term, searchable archives of firewall access logs, administrative audit trails, and authentication events to meet compliance requirements such as PCI-DSS, HIPAA, or GDPR.
*   **Real-time Threat Detection:** Use standardized CEF fields to trigger Elastic Security alerts when high-severity events (e.g., `event.severity` > 7) are detected by edge security devices, regardless of the underlying hardware vendor.
*   **Infrastructure Troubleshooting:** Correlation of network traffic patterns and security blocks across multiple hop points in a network to identify misconfigurations or performance bottlenecks.
*   **Asset Management:** Use source and destination information to discover and categorize assets within the corporate network automatically by tracking `source.mac` and `host.hostname` fields where provided by the CEF source.

## Data types collected

This integration collects log data via the following datastreams:

*   **`cef.log` (Logs):** This is the primary datastream for all incoming CEF data. It handles the ingestion of various security events including:
    *   **Firewall and Traffic Logs:** Records of permitted and denied network connections, including source/destination IPs, ports, and protocols.
    *   **Intrusion Detection/Prevention (IDS/IPS) Alerts:** Notifications of malicious activity signatures, attack names, and severity levels.
    *   **Web Application Firewall (WAF) Events:** Details on blocked web requests, SQL injection attempts, and cross-site scripting (XSS) violations.
    *   **Administrative Audit Logs:** Tracking of configuration changes, user logins, and privilege escalations on security appliances.
*   **Data Format:** All data is received in the Common Event Format (CEF) via Syslog transport (UDP or TCP). The integration utilizes an ingest pipeline to map these fields directly to ECS.
*   **Key Field Mapping:** The integration parses the CEF header and extension dictionary, mapping fields such as:
    *   `src` -> `source.ip`
    *   `dst` -> `destination.ip`
    *   `spt` -> `source.port`
    *   `dpt` -> `destination.port`
    *   `proto` -> `network.transport`
    *   `deviceReceiptTime` -> `event.ingested`
    *   `msg` -> `message`

## Compatibility

The **Common Event Format (CEF)** integration is compatible with any device or software capable of exporting logs in the ArcSight-style CEF format. 

*   **Tested Vendors:** Palo Alto Networks PAN-OS (v8.x, v9.x, v10.x, v11.x), Check Point R80.x and R81.x, Fortinet FortiOS, Cisco ASA, Imperva WAF, and MikroTik RouterOS. 
*   **Transport Protocols:** Supports transport via **Syslog (RFC 3164 or RFC 5424)**.
*   **Elastic Stack Requirements:** Requires Elastic Stack version **8.0.0** or higher for full ECS 8.x compatibility and optimal pipeline performance.

## Scaling and Performance

*   **High Volume Ingestion:** For environments generating more than 10,000 events per second (EPS), it is recommended to use the **UDP protocol** to minimize transport overhead, though this may result in data loss during network congestion.
*   **Horizontal Scaling:** Deploy multiple Elastic Agents across different subnets or behind a network load balancer (e.g., HAProxy or F5) to distribute the ingestion load and provide high availability for log collection.
*   **Resource Allocation:** Ensure the host running the Elastic Agent has sufficient CPU (at least 2-4 cores) and memory (4GB+), as parsing complex CEF extensions can be CPU-intensive at high volumes.
*   **Queue Management:** Adjust the internal memory queue within the integration settings to handle spikes in traffic without dropping packets.

# Set Up Instructions

## Vendor prerequisites

1.  **Administrative Privileges:** You must have administrative access to the management interface (Web UI or CLI) of the security device sending the logs.
2.  **Network Routing:** Ensure the security device has a clear network path to the Elastic Agent's IP address on the configured port (default is `9003`).
3.  **Firewall Permissions:** Any intermediate firewalls or Access Control Lists (ACLs) must allow traffic on the selected port (`9003`) and protocol (`UDP`/`TCP`).
4.  **Log Formatting Capabilities:** The device must support "CEF" or "ArcSight" as a log export format.
5.  **Source Identification:** Identify the source IP of the security appliance to allow filtering or access control on the Elastic Agent host if necessary.

## Elastic prerequisites

1.  **Elastic Agent Enrollment:** An Elastic Agent must be installed on a host and successfully enrolled in **Fleet Management**.
2.  **Agent Policy:** A dedicated or existing Agent Policy must be available to apply the CEF integration settings.
3.  **Elastic Stack Version:** Ensure the Elastic Stack is at version **8.0.0** or higher.
4.  **Internet Connectivity:** The Elastic Agent must be able to reach the Elastic Package Registry (EPR) to download the CEF integration package, or have access to an air-gapped package registry.

## Vendor set up steps

### For General Syslog/CEF Forwarding:
1.  Log in to the management console of your security appliance.
2.  Navigate to the **System Settings**, **Logging**, or **Remote Log Servers** section.
3.  Create a new **Remote Syslog Server** or **Log Forwarding Profile**.
4.  Enter the **IP Address** of the server where the Elastic Agent is installed.
5.  Set the **Destination Port** to `9003` (or the custom port you intend to use).
6.  Select the **Protocol** as `UDP` or `TCP` (UDP is standard for CEF).
7.  Set the **Log Format** specifically to `CEF` or `ArcSight CEF`.
8.  Choose the **Facility** (e.g., `Local7` or `Syslog`) and the **Severity Level** (e.g., `Informational` or `Warning`) you wish to export.
9.  Save the configuration and, if necessary, **Commit** or **Install Policy** to activate the logging.

### For Palo Alto Networks (PAN-OS):
1.  Navigate to **Device > Log Settings**.
2.  Under the **Syslog** section, click **Add** to create a new profile.
3.  Name the profile (e.g., `Elastic-CEF`) and click **Add** again to define a server.
4.  Enter the Elastic Agent IP and port `9003`, and select `UDP`.
5.  Go to the **Custom Log Format** tab.
6.  Select the log type (e.g., Traffic, Threat) and paste the specific **CEF Template** provided by Palo Alto for ArcSight integration.
7.  Navigate to **Objects > Log Forwarding** and create a profile that uses this Syslog server.
8.  Assign this Log Forwarding Profile to your **Security Rules**.
9.  Click **Commit** to apply the changes.

### For Check Point Firewalls:
1.  Open **SmartConsole** and navigate to **Logs & Monitor**.
2.  In the **Manage Catalog**, search for "Log Exporter".
3.  Run the following CLI command on the Security Management Server or Log Server:
    `cp_log_export add name elastic-cef target-server <Agent_IP> target-port 9003 protocol udp format cef`
4.  Verify the exporter status: `cp_log_export status`.
5.  Start the exporter: `cp_log_export restart name elastic-cef`.

## Kibana set up steps

1.  In Kibana, navigate to **Management** > **Fleet** > **Agent policies**.
2.  Click on the policy assigned to your data collection agent.
3.  Click **Add integration** and search for **Common Event Format (CEF)**.
4.  Click the **Common Event Format (CEF)** tile and select **Add Common Event Format (CEF)**.
5.  In the **Configure integration** screen, configure the following fields:
    *   **Listen Address (`syslog_host`)**: Set to `0.0.0.0` (to listen on all available network interfaces) or the specific IP of the agent's interface.
    *   **Listen Port (`syslog_port`)**: Set to `9003` (this must match your vendor-side configuration).
    *   **Protocol**: Select `udp` or `tcp` to match your vendor settings.
6.  Expand **Advanced options** to configure:
    *   **Internal Queue Size**: Default is `1000`. Increase to `5000` or higher for high-traffic environments.
    *   **Max Message Size**: Default is `20KiB`. Increase if you expect very large CEF messages containing long URL paths or detailed payload analysis.
    *   **Keep Raw Fields**: Enable this if you need to retain the original unparsed message in the `event.original` field for auditing purposes.
    *   **Tags**: Add custom tags (e.g., `cef-firewall`, `datacenter-1`) to help filter data in Kibana Discover.
7.  Click **Save and continue**, then click **Add agent integration to policy** to deploy the configuration to your Elastic Agents.

# Validation Steps

Once the configuration is complete, follow these steps to ensure data is flowing correctly.

### 1. Trigger Data Flow on Vendor:
1.  **Firewall Logs:** Initiate a network request from a machine behind the firewall to a blocked or allowed external IP.
2.  **IPS/IDS Logs:** Use a testing tool to simulate a non-malicious attack signature if safe (e.g., triggering a "User-Agent" signature alert by using a custom string with `curl -A "TestIDS" http://example.com`).
3.  **Administrative Logs:** Log out and log back into the security appliance's management console to generate an authentication audit event.

### 2. Check Data in Kibana:
1.  Navigate to **Analytics** > **Discover**.
2.  Select the **Logs Explorer** or the `logs-*` index pattern.
3.  Enter the following filter query in the KQL search bar to isolate CEF data: 
    `data_stream.dataset : "cef.log"`
4.  Verify that the following fields are populated and correctly parsed:
    *   `cef.vendor`: Displays the appliance vendor (e.g., `Palo Alto Networks`).
    *   `cef.device_product`: Displays the product name (e.g., `PAN-OS`).
    *   `source.ip`: Should contain the source IP of the network event.
    *   `destination.ip`: Should contain the destination IP of the network event.
    *   `event.outcome`: Should show `success`, `failure`, or `deny`.
    *   `event.severity`: Should be mapped to a numeric value representing the event importance.
5.  Navigate to **Dashboard** and search for the **[Logs CEF] Overview** dashboard to visualize incoming CEF events and verify data distribution.

# Troubleshooting

## Common Configuration Issues

*   **Port Conflict**: If the Elastic Agent fails to start the CEF listener, check if another service is using port 9003. Use `netstat -ano | grep 9003` (Linux) or `Get-NetTCPConnection -LocalPort 9003` (Windows) to identify conflicting processes.
*   **Network Connectivity**: If no logs appear, verify connectivity using `tcpdump` or `Wireshark` on the Elastic Agent host: `tcpdump -i any port 9003`. If no packets arrive, check the vendor's routing and local firewall (iptables/ufw/Windows Firewall).
*   **Protocol Mismatch**: Ensure the protocol (UDP vs TCP) selected in the Elastic integration UI exactly matches the protocol configured on the vendor device. TCP often fails if the agent is not listening when the device tries to establish a connection.
*   **Incorrect Listen Address**: If the **Listen Address** is set to `127.0.0.1`, the agent will only accept logs from its own host. Change this to `0.0.0.0` to accept external logs.

## Ingestion Errors

*   **Parsing Failures**: If logs appear but fields are not parsed (remaining as a raw message), check the `error.message` field in Kibana. It may indicate that the Syslog header does not conform to RFC 3164 or RFC 5424.
*   **Timestamp Mismatches**: If logs appear delayed, verify that the time and timezone settings on the vendor appliance match the UTC expectation of the Elastic Stack.
*   **Truncated Messages**: If CEF extensions are missing, the **Max Message Size** in the integration settings might be too low. Increase this value (e.g., to `8192` or higher) to accommodate large CEF packets.

## Vendor Resources

*   [MikroTik: CEF with Elasticsearch Configuration Guide](https://help.mikrotik.com/docs/spaces/ROS/pages/319782960/CEF+with+Elasticsearch)
*   [Palo Alto Networks: Configure Syslog Forwarding (PAN-OS 11.1)](https://docs.paloaltonetworks.com/pan-os/11-1/pan-os-admin/monitoring/use-syslog-for-monitoring)
*   [Check Point: Log Exporter Configuration Guide](https://support.checkpoint.com/results/sk/sk122323)
*   [Imperva WAF: CEF Configuration via ArcSight Integration](https://docs.imperva.com/bundle/v14.7-waf-administration-guide/page/6355.htm)

# Documentation sites

*   [Palo Alto Networks: Syslog Field Descriptions (PAN-OS 11.1)](https://docs.paloaltonetworks.com/pan-os/11-1/pan-os-admin/monitoring/use-syslog-for-monitoring/syslog-field-descriptions)
*   [Fortinet: FortiGate CEF Log Format Reference](https://docs.fortinet.com/document/fortigate/7.4.0/administration-guide/543168/cef-log-format)
*   OpenText: ArcSight Common Event Format (CEF) Implementation Guide