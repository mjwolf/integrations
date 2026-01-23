# Service Info

The Cisco Secure Email Gateway (formerly known as Cisco Email Security Appliance or ESA) integration allows you to ingest logs related to email traffic, security threats, and system operations into the Elastic Stack. This integration provides visibility into your email security posture by parsing diverse data types including anti-spam, antivirus, and Advanced Malware Protection (AMP) events.

## Common use cases

The Cisco Secure Email Gateway integration provides deep visibility into email traffic, security threats, and system performance by ingesting logs directly from the appliance into the Elastic Stack.
- **Threat Detection and Analysis:** Monitor Advanced Malware Protection (AMP) and Antivirus logs to identify malicious attachments and blocked threats, allowing security teams to respond to emerging phishing or malware campaigns.
- **Mail Delivery Troubleshooting:** Analyze Text Mail and Bounce logs to track the lifecycle of an email message, diagnose delivery failures, and investigate why specific messages were delayed or rejected.
- **Administrative Auditing and Compliance:** Track user logins, logouts, and configuration changes through Authentication and System logs to maintain an audit trail for regulatory compliance and internal security policies.
- **Appliance Health Monitoring:** Use Status logs to observe real-time resource utilization, including CPU usage, RAM utilization, and queue depths, ensuring the gateway is operating within optimal performance parameters.

## Data types collected

This integration collects the following data types through specific data streams. Each stream is designed to handle different ingestion methods depending on your network architecture:

- **Security Logs:** Detailed events from security engines including AMP (Advanced Malware Protection), Anti-Spam (CASE), and Antivirus (Sophos/McAfee).
- **Traffic and Transaction Logs:** Information on incoming and outgoing mail flows, including sender and recipient details, message IDs (MID), and injection/delivery status.
- **System and Management Logs:** Administrative activity, system status updates, service restarts, and configuration changes.
- **Authentication and Web Logs:** GUI access events, SSH login attempts, and HTTP requests to the management interface.
- **Specific Log Formats:** Supports standard Syslog formats, Common Event Format (CEF) for consolidated logs, and raw text logs for file-based ingestion.

The following data streams are available:
- **Cisco Secure Email Gateway logs (TCP):** Collect Cisco Secure Email Gateway logs via TCP input. This stream provides reliable delivery for system and security events.
- **Cisco Secure Email Gateway logs (UDP):** Collect Cisco Secure Email Gateway logs via UDP input. This stream is optimized for high-performance transmission of event data.
- **Cisco Secure Email Gateway logs (Logfile):** Collect Cisco Secure Email Gateway logs via logfile. This stream is used for local file monitoring or logs pushed to a central server via FTP/SCP.

## Compatibility

This module has been tested against **Cisco Secure Email Gateway** (formerly Cisco Email Security Appliance) version **14.0.0**. Specifically, it has been verified using the **Virtual Gateway C100V** model. While newer sub-versions of 14.x are expected to work, users should verify log patterns if using significantly older legacy versions.

This integration requires **Kibana version ^8.11.0 or ^9.0.0**.

## Scaling and Performance

To ensure optimal performance in high-volume email environments, consider the following:
- **Transport/Collection Considerations:** This integration supports TCP and UDP for network-based ingestion. While UDP offers lower overhead for high-speed transmission, TCP is recommended for production environments to ensure reliable delivery and prevent data loss during network congestion. TCP is particularly preferred for large Consolidated Event Logs (CEF).
- **Data Volume Management:** Cisco ESA can generate a massive volume of data, particularly through `mail_logs`. Configure the vendor appliance to forward only necessary events and set the log level to `Information` or lower. Avoid `Debug` levels as they can significantly increase the ingest load and storage requirements.
- **Elastic Agent Scaling:** For high-throughput environments, deploy multiple Elastic Agents behind a network load balancer to distribute Syslog traffic. Ensure host machines have adequate CPU resources to handle the regex-based parsing required for unstructured Cisco text logs, and increase the number of workers for the ingest pipeline if processing delays are observed.

# Set Up Instructions

## Vendor prerequisites

1. **Administrative Access:** You must have an administrator account on the Cisco Secure Email Gateway web interface.
2. **Log Subscriptions Enabled:** The ability to create and modify Log Subscriptions must be available in the system license.
3. **Network Path:** Open firewall ports between the Cisco ESA management/data interface and the Elastic Agent (default ports are `514` for Syslog or custom ports as configured).
4. **Storage for File Collection:** If using the FTP Push method for Authentication or Bounce logs, a functional FTP/SCP server must be accessible by both the ESA and the Elastic Agent.
5. **Time Synchronization:** Ensure NTP is configured on both the Cisco ESA and the Elastic Agent host to maintain accurate event timestamps.

## Elastic prerequisites

- **Kibana Version:** This integration requires Kibana version **^8.11.0** or **^9.0.0**.
- **Elastic Agent Installation:** An Elastic Agent must be installed and enrolled in a policy via Fleet.
- **Connectivity:** The Elastic Agent must be reachable by the Cisco Secure Email Gateway over the configured Syslog ports or have access to the log paths where FTP files are pushed.
- **Integration Asset Installation:** Ensure the Cisco Secure Email Gateway integration assets are installed in Kibana before sending data.

## Vendor set up steps

Logging is configured within the Cisco Secure Email Gateway administrator portal. You must create individual log subscriptions for different data types.

### For Syslog Push (Recommended for most logs):
1. Log in to the Cisco Secure Email Gateway web interface.
2. Navigate to **System Administration** > **Log Subscriptions**.
3. Click **Add Log Subscription**.
4. Select the **Log Type** from the dropdown menu (e.g., `Text Mail Logs`, `Anti-Spam Logs`, `AMP Engine Logs`).
5. Enter a **Log Name**. **CRITICAL**: To ensure correct parsing, use the following names for their respective categories:
    - AMP Engine Logs -> `amp`
    - Anti-Spam Logs -> `antispam`
    - Antivirus Logs -> `antivirus`
    - Consolidated Event Logs -> `consolidated_event`
    - Text Mail Logs -> `mail_logs`
    - Status Logs -> `status`
    - System Logs -> `system`
6. Set the **Log Level** to `Information`.
7. Under **Retrieval Method**, select **Syslog Push**.
8. In the **Hostname** field, enter the IP address of your Elastic Agent.
9. Select the **Protocol** (TCP is recommended) and the **Port** (default `514`).
10. Click **Submit**. Repeat these steps for every log type you want to collect.
11. Click **Commit Changes** at the top of the page to apply the configuration.

### For FTP Push (Required for Authentication/Bounce logs):
1. Navigate to **System Administration** > **Log Subscriptions**.
2. Click **Add Log Subscription**.
3. Select **Authentication Logs** or **Bounce Logs** (these do not support Syslog Push).
4. For **Log Name**, use `authentication` or `bounces`.
5. Set the **Log Level** to `Information`.
6. Under **Retrieval Method**, select **FTP Push** or **SCP Push**.
7. Provide the credentials and directory path for your remote server.
8. Configure the Elastic Agent "Logfile" input to monitor the directory where these files are pushed.
9. Click **Submit** and then **Commit Changes**.

## Kibana set up steps

### Collecting Cisco Secure Email Gateway logs via TCP input.
1. In Kibana, navigate to **Integrations** > **Cisco Secure Email Gateway**.
2. Click **Add Cisco Secure Email Gateway**.
3. Provide a name for the integration and select an Agent Policy.
4. Under the **Collect Cisco Secure Email Gateway logs via TCP input** section, configure the following:
    - **Listen Address** (`listen_address`): The bind address to listen for TCP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
    - **Listen Port** (`listen_port`): The TCP port number to listen on. This must match the port configured on the Cisco ESA. Default: `514`.
    - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
5. Click **Save and continue**.

### Collecting Cisco Secure Email Gateway logs via UDP input.
1. In the integration configuration, locate the **Collect Cisco Secure Email Gateway logs via UDP input** section.
2. Configure the following fields:
    - **Listen Address** (`listen_address`): The bind address to listen for UDP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
    - **Listen Port** (`listen_port`): The UDP port number to listen on. This must match the port configured on the Cisco ESA. Default: `514`.
    - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
3. Click **Save and continue**.

### Collecting Cisco Secure Email Gateway logs.
1. In the integration configuration, locate the **Collecting Cisco Secure Email Gateway logs** section (logfile input).
2. Configure the following settings:
    - **Paths** (`paths`): Provide a list of absolute paths to the log files to be monitored (e.g., `/var/log/cisco/*.log`).
    - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
3. Click **Save and continue**.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on Cisco Secure Email Gateway:
- **Authentication event:** Log out of the Cisco ESA web interface and log back in with an administrator account to generate an authentication log.
- **Configuration event:** Navigate to **System Administration > Log Subscriptions**, update a subscription description, and click **Commit Changes** to trigger a system log.
- **Mail traffic:** Send a test email through the gateway to generate entries in the `mail_logs` category.
- **Status heartbeat:** Wait up to 60 seconds for the appliance to generate a periodic `status` log update.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "cisco_secure_email_gateway.log"`
4. Verify logs appear. Expand a log entry and confirm these fields:
   - `event.dataset` (should match `cisco_secure_email_gateway.log`)
   - `source.ip` and/or `host.ip`
   - `event.action` or `event.outcome`
   - `message` (the raw log payload)
5. Navigate to **Analytics > Dashboards** and search for "Cisco Secure Email Gateway" to verify visualizations are populating.

# Troubleshooting

## Common Configuration Issues

- **Uncommitted Changes**: Cisco ESA requires an explicit "Commit Changes" action. If logs are not appearing, ensure the yellow banner at the top of the ESA UI has been cleared by committing.
- **Retrieval Method Incompatibility**: Remember that **Authentication** and **Bounce** logs do not support Syslog Push. These must be collected via FTP Push/Logfile.
- **Log Name Mismatch**: If log names in the ESA configuration do not match the expected strings (e.g., using `mail` instead of `mail_logs`), parsing logic may fail to identify the log source correctly.
- **Port Conflicts**: Ensure no other service is using Port 514 on the Elastic Agent host. Use `netstat -ano | grep 514` to check for existing listeners.

## Ingestion Errors

- **Parsing Failures:** If logs appear in Discover but contain the tag `_grokparsefailure`, check if the ESA version is compatible. You can inspect the `error.message` field in the log document to see why the grok pattern failed to match the incoming string.
- **Timestamp Mismatch:** If logs do not appear in the current time range, check for timezone offsets between the ESA and the Elastic Stack. ESA logs often use a local timestamp that may lack timezone information, causing them to be indexed at the wrong time.
- **Truncated Logs:** Very long Consolidated Event Logs (CEF) might be truncated by some Syslog implementations over UDP. If you see incomplete JSON or CEF data, switch to the TCP protocol for more reliable delivery of large payloads.

## Vendor Resources

- [Cisco Email Security Appliance Product Page](https://www.cisco.com/site/us/en/products/security/secure-email/index.html)
- [Cisco ESA Administration Guide (Log Subscriptions)](https://www.cisco.com/c/en/us/td/docs/security/ces/user_guide/esa_user_guide_14-0/b_ESA_Admin_Guide_ces_14-0/b_ESA_Admin_Guide_12_1_chapter_0100111.html)
- [User Guide for AsyncOS 16.0.2 for Cisco Secure Email Gateway - MD - Logging](https://www.cisco.com/c/en/us/td/docs/security/esa/esa16-0-2/user_guide/b_ESA_Admin_Guide_16-0-2/b_ESA_Admin_Guide_12_1_chapter_0100111.html)

# Documentation sites

- [Cisco ESA Log Samples and Descriptions](https://www.cisco.com/c/en/us/td/docs/security/ces/user_guide/esa_user_guide_14-0/b_ESA_Admin_Guide_ces_14-0/b_ESA_Admin_Guide_12_1_chapter_0100111.html)
- [Cisco Secure Email Gateway Product Page](https://www.cisco.com/site/us/en/products/security/secure-email/index.html)
