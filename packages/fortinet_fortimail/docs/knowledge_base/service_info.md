# Service Info

## Common use cases

The Fortinet FortiMail integration provides comprehensive visibility into email security and traffic patterns, enabling security teams to monitor and respond to various email-borne threats.

-   **Threat Detection and Response:** Monitor Antivirus and Antispam logs to identify phishing attempts, malware distribution, and Business Email Compromise (BEC) attacks targeting the organization. By analyzing these logs, security operations centers (SOC) can correlate email threats with endpoint or network alerts.
-   **Audit and Compliance:** Track System logs to maintain a record of administrative activities, configuration changes, and user login/logout events. This is essential for meeting regulatory compliance requirements such as GDPR, HIPAA, or PCI-DSS which mandate strict auditing of security appliance access.
-   **Email Traffic Analysis:** Utilize History and Mail logs to visualize email traffic flow, identify high-volume senders, and monitor for unusual spikes in mail activity. This data helps in identifying compromised internal accounts being used for outbound spam or data exfiltration.
-   **Secure Communication Monitoring:** Monitor Encryption logs to track Identity-Based Encryption (IBE) events and ensure that sensitive communications are being handled according to organizational security policies. This ensures that encrypted mail delivery is functioning as expected for sensitive business units.

## Data types collected

This integration collects several categories of security and operational data from Fortinet FortiMail instances using the following data streams:

*   **Fortinet FortiMail logs (filestream):** Collect Fortinet FortiMail logs via Filestream input. This data stream is designed for environments where the Elastic Agent has direct access to log files stored on a local disk or a mounted network share.
*   **Fortinet FortiMail logs (tcp):** Collect Fortinet FortiMail logs via TCP input. This provides a reliable, connection-oriented stream of events over the network, ensuring data integrity during transmission.
*   **Fortinet FortiMail logs (udp):** Collect Fortinet FortiMail logs via UDP input. This offers a low-overhead, connectionless method for receiving log data over the network, suitable for high-performance environments where minor packet loss is acceptable.

Each data stream processes the following event types:
- **Email Traffic Logs:** Collected via the History logs, providing details on every email transaction processed by the FortiMail unit.
- **Administrative Audit Logs:** Collected via the System logs, recording configuration changes and administrative access.
- **Security Event Logs:** Specific logs for Antispam, Antivirus, and Encryption detections, providing granular detail on blocked or flagged content.
- **CSV Formatted Syslog:** All data is received as CSV-formatted strings and parsed into standardized Elastic Common Schema (ECS) fields.

## Compatibility

- **Fortinet FortiMail:** This integration has been specifically tested against and supports **version 7.2.2**. It is likely compatible with newer versions within the 7.x branch, provided the CSV logging format remains consistent.
- **Elastic Stack:** Requires Kibana/Elasticsearch version **8.11.0** or higher.

## Scaling and Performance

To ensure optimal performance in high-volume environments, consider the following:

- **Transport/Collection Considerations:** For high-volume email environments, **TCP** is recommended over **UDP** to ensure delivery reliability and prevent data loss during network congestion. If using UDP, ensure the network path between the FortiMail appliance and the Elastic Agent is low-latency to minimize packet drops. The default port is **9024**.
- **Data Volume Management:** To reduce the processing load on the Elastic Agent, configure the FortiMail "Level" setting to `Information` or higher, avoiding `Debug` logs unless troubleshooting. Users should selectively enable only the required log types (e.g., History, System) in the FortiMail Logging Policy to filter out unnecessary events at the source before they reach the Elastic Agent.
- **Elastic Agent Scaling:** For enterprises processing millions of emails daily, consider deploying multiple Elastic Agents behind a network load balancer to distribute traffic evenly. Place Agents geographically or logically close to the FortiMail instances to minimize latency and ensure the host running the Agent has sufficient CPU resources to handle the regex-based parsing of the CSV formatted logs.

# Set Up Instructions

## Vendor prerequisites

1.  **Administrative Access:** You must have super-user or equivalent administrative permissions on the FortiMail appliance to modify Log Settings and create Remote Log profiles.
2.  **CSV Logging Requirement:** The FortiMail unit must be configured to output logs in **CSV format**; the integration will fail to parse standard syslog formats.
3.  **Network Connectivity:** Ensure the FortiMail appliance can reach the Elastic Agent host over the configured Syslog port (default **TCP/UDP 9024**). Verify no intermediate firewalls are blocking this traffic.
4.  **Firmware Version:** Confirm the appliance is running version **7.2.2** or a compatible release (v7.x or higher recommended) to ensure field mapping accuracy.

## Elastic prerequisites

- **Elastic Agent:** An Elastic Agent must be installed on a host reachable by the FortiMail appliance and enrolled in Fleet.
- **Stack Version:** The Elastic Stack (Elasticsearch and Kibana) must be on version **8.11.0** or higher as required by the integration manifest.
- **Connectivity:** The host running the Elastic Agent must have the specified TCP or UDP ports (default **9024**) open in its local firewall (e.g., `ufw` or `firewalld`) to receive incoming traffic.

## Vendor set up steps

### For Syslog (TCP/UDP) Collection:
1. Log in to the FortiMail web administration interface.
2. Navigate to **Log & Report > Log Setting > Remote**.
3. Click **New** to create a new remote logging configuration.
4. In the configuration dialog, check the **Enable** box.
5. Set the **Name** to a descriptive value such as `ElasticAgent_Syslog`.
6. Enter the **Server name/IP** of the host where your Elastic Agent is running.
7. For **Log protocol**, select **Syslog**.
8. Set the **Port** to match your Kibana input configuration (default is `9024`).
9. Set the **Level** to `Information` or `Notification` to capture relevant security and system events.
10. Set the **Facility** to `local7` (or another preferred facility).
11. **IMPORTANT:** Check the box for **Enable CSV format**. This is mandatory for the integration to work.
12. In the **Logging Policy Configuration** section, select the checkboxes for: **History**, **System**, **Mail**, **Antispam**, **Antivirus**, and **Encryption**.
13. Click **Create** to save the profile.

### For Logfile (Filestream) Collection:
1. Configure FortiMail to store logs locally or on a mounted network share that the Elastic Agent can access.
2. Ensure the log files are written in **CSV format** by navigating to **Log & Report > Log Setting > Local** and verifying the format settings.
3. Identify the absolute directory path where FortiMail writes its `.log` files (e.g., `/var/log/fortimail/`).
4. Ensure the Elastic Agent service account has sufficient read permissions for this directory and all files within it.

## Kibana set up steps

1. In Kibana, navigate to **Management > Integrations**.
2. Search for **Fortinet FortiMail** and select the integration.
3. Click **Add Fortinet FortiMail**.
4. Configure the integration name and select an Agent Policy.
5. Select and configure one or more of the following input types based on your environment:

### Collecting logs from Fortinet FortiMail instances via filestream input.
Use this input to monitor log files local to the agent.
- **Paths** (`paths`): A list of glob-based paths that will be crawled and fetched. (e.g., `/var/log/fortimail/*.log`).
- **Timezone Offset** (`tz_offset`): By default, datetimes in the logs will be interpreted as relative to the timezone configured in the host where the agent is running. If ingesting logs from a host on a different timezone, use this field to set the timezone offset so that datetimes are correctly parsed. Acceptable timezone formats are: a canonical ID (e.g. `Europe/Amsterdam`) or an HH:mm differential (e.g. `-05:00`). Default: `local`.
- **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.

### Collecting logs from Fortinet FortiMail instances via tcp input.
Use this input to receive logs via TCP syslog.
- **Listen Address** (`listen_address`): The bind address to listen for TCP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
- **Listen Port** (`listen_port`): The TCP port number to listen on. Default: `9024`.
- **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.

### Collecting logs from Fortinet FortiMail instances via udp input.
Use this input to receive logs via UDP syslog.
- **Listen Address** (`listen_address`): The bind address to listen for UDP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
- **Listen Port** (`listen_port`): The UDP port number to listen on. Default: `9024`.
- **Timezone Offset** (`tz_offset`): By default, datetimes in the logs will be interpreted as relative to the timezone configured in the host where the agent is running. If ingesting logs from a host on a different timezone, use this field to set the timezone offset so that datetimes are correctly parsed. Acceptable timezone formats are: a canonical ID (e.g. `Europe/Amsterdam`) or an HH:mm differential (e.g. `-05:00`). Default: `local`.
- **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.

6. Click **Save and continue** to deploy the configuration to the Elastic Agent.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on Fortinet FortiMail:
- **Authentication event:** Log out of the FortiMail administrator UI and log back in to generate a System log entry.
- **Traffic event:** Send a test email through the FortiMail gateway to trigger History and Mail events.
- **Configuration change:** Modify a non-critical description field in the FortiMail settings and click **Apply** to generate an audit log.
- **Security event:** If in a test environment, send a harmless EICAR test string through the mail system to trigger an Antivirus detection log.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "fortinet_fortimail.log"`
4. Verify logs appear. Expand a log entry and confirm these fields:
   - `event.dataset` (should be `fortinet_fortimail.log`)
   - `source.ip` (the IP of the FortiMail appliance or the email sender)
   - `event.action` or `event.outcome`
   - `message` (the raw CSV log payload)
5. Navigate to **Analytics > Dashboards** and search for "Fortinet FortiMail" to view the pre-built Overview dashboard.

# Troubleshooting

## Common Configuration Issues

- **CSV Format Not Enabled**: If logs appear in Kibana but are not being parsed into specific fields (remaining as raw text in the `message` field), verify that **Enable CSV format** is checked in the FortiMail Remote Log settings.
- **Port Conflict**: If the Elastic Agent fails to start the Syslog input, check if another service is already using the configured port (e.g., port 514 is often used by the system syslog daemon). Use `netstat -tuln` to check for listening ports.
- **Incorrect Listen Address**: If the Elastic Agent is running in a container or on a multi-homed host, ensure the **Listen Address** is set to `0.0.0.0` to accept connections from the FortiMail appliance's IP.
- **Firewall Blocking Traffic**: Ensure that any intermediate firewalls and the local firewall on the Elastic Agent host allow traffic on the configured TCP/UDP port (e.g., 9024).

## Ingestion Errors

- **Parsing Failures**: If logs contain the tag `_grokparsefailure` or `_csvparsefailure`, check the `error.message` field in Kibana. This often happens if the FortiMail version is different from 7.2.2 and uses a different CSV column order.
- **Timezone Mismatch**: If logs appear to be arriving in the future or the past, adjust the **Timezone Offset** variable in the Kibana input configuration to match the FortiMail appliance's system time.
- **Incomplete Log Policy**: If certain event types (like Antivirus) are missing, revisit the **Logging Policy Configuration** in the FortiMail UI to ensure those specific checkboxes are enabled.

## Vendor Resources

- [Configuring logging | FortiMail Appliance and VM 7.2.2](https://docs.fortinet.com/document/fortimail/7.2.2/administration-guide/332364/configuring-logging)
- [About FortiMail logging | FortiMail Appliance and VM 7.2.2](https://docs.fortinet.com/document/fortimail/7.2.2/administration-guide/435158/about-fortimail-logging)

# Documentation sites

- [Fortinet FortiMail Administration Guide: Configuring Logging](https://docs.fortinet.com/document/fortimail/7.2.2/administration-guide/332364/configuring-logging)
- [About FortiMail Logging Reference](https://docs.fortinet.com/document/fortimail/7.2.2/administration-guide/435158/about-fortimail-logging)
- Refer to the official vendor website for additional resources.
