# Service Info

The Fortinet FortiEDR integration allows organizations to ingest endpoint security events, system logs, and audit trails directly into the Elastic Stack. By centralizing FortiEDR data, security operations center (SOC) teams can correlate endpoint telemetry with other network and cloud data sources to detect advanced persistent threats (APTs) and automate incident response workflows.

## Common use cases
The Fortinet FortiEDR integration allows organizations to ingest endpoint security events and system logs into the Elastic Stack for centralized monitoring and advanced threat hunting. By collecting data from FortiEDR playbooks and export settings, security teams can correlate endpoint activity with other network data.
- **Endpoint Threat Monitoring:** Monitor real-time security events detected by FortiEDR across all protected endpoints to identify malware, ransomware, or suspicious behavior.
- **Automated Incident Response Tracking:** Track the actions taken by automated playbooks to understand how threats are being mitigated or blocked in the environment.
- **Compliance and Auditing:** Collect audit trail and system logs to maintain a record of administrative changes and system health for regulatory compliance.
- **Unified Security Analytics:** Centralize FortiEDR logs alongside firewall, identity, and cloud logs in Kibana to build a comprehensive view of the organization's security posture.

## Data types collected

This integration can collect the following types of data:

- **Endpoint Detection and Response Logs:** Real-time security events including malicious file detection, process termination, and memory protection alerts.
- **System Events:** Internal health and status logs from the FortiEDR components, including collector status changes and communication errors.
- **Audit Trails:** Detailed records of administrative actions performed within the FortiEDR Management Console, such as policy changes or user logins.
- **Data Formats:** Supports multiple syslog formats including JSON, Common Event Format (CEF), and Log Event Extended Format (LEEF).

The following data streams are available:
- **Fortinet FortiEDR Endpoint Detection and Response logs:**
    - **UDP Input:** Collect Fortinet FortEDR Endpoint Detection and Response logs via network transmission.
    - **TCP Input:** Collect Fortinet FortEDR Endpoint Detection and Response logs via network transmission.
    - **Logfile Input:** Collect Fortinet FortEDR Endpoint Detection and Response logs from file exports.

## Compatibility

- **Elastic Requirements:** This integration requires Kibana version **8.11.0** or higher.
- **Vendor Compatibility:** This integration is compatible with **Fortinet FortiEDR** version 5.0.0 and higher. It supports standard syslog output formats as defined in the official Fortinet documentation. Performance and field mapping are optimized for the ECS (Elastic Common Schema).

## Scaling and Performance

To ensure optimal performance in high-volume environments, consider the following:
- **Transport/Collection Considerations:** For high-volume environments, TCP is recommended over UDP to ensure delivery reliability, as UDP may drop packets during network congestion. If using UDP, ensure the network path between FortiEDR and the Elastic Agent has minimal latency. When using the `logfile` input, ensure the disk I/O on the agent host is sufficient to handle the write/read throughput of the incoming logs.
- **Data Volume Management:** To optimize storage and processing, utilize the **NOTIFICATIONS** pane in the FortiEDR Export Settings to filter which categories (Security, System, or Audit Trail) are forwarded. Additionally, configure the security Playbook policies to only trigger syslog notifications for high-priority event types, reducing the noise from benign system activities.
- **Elastic Agent Scaling:** For high-throughput environments, deploy multiple Elastic Agents behind a network load balancer to distribute traffic evenly. Place Agents close to the data source to minimize latency. Ensure the host running the Elastic Agent has sufficient CPU and memory to handle the parsing load, typically starting with 2-4 vCPUs for dedicated log collection.

# Set Up Instructions

## Vendor prerequisites
1. **Administrative Access:** You must have administrative credentials for the Fortinet FortiEDR Central Manager console to modify **Export Settings** and **Playbooks**.
2. **Network Connectivity:** Ensure the FortiEDR Central Manager can communicate with the Elastic Agent over the configured port (default is `9509`) via **TCP** or **UDP**.
3. **Feature Licensing:** Ensure the FortiEDR license includes access to **Automated Incident Response Playbooks**, as syslog notifications are configured within these policies.
4. **Destination Details:** You must know the IP address or hostname of the machine hosting the Elastic Agent.
5. **Syslog Knowledge:** Familiarity with the facility codes (defaulting to 16/Custom App) and severity levels (Notice) used by FortiEDR.

## Elastic prerequisites
1. **Elastic Agent Status:** An Elastic Agent must be installed and enrolled in Fleet with a policy that includes the Fortinet FortiEDR integration.
2. **Network Access:** Firewalls between the FortiEDR Central Manager and the Elastic Agent must allow traffic on the specified syslog port (e.g., `9509`).
3. **Elastic Stack Version:** This integration requires Kibana version **8.11.0** or higher.

## Vendor set up steps

### For Syslog (TCP/UDP) Collection:

1. Log in to the **FortiEDR Central Manager** console.
2. Navigate to **Administration > Export Settings**.
3. Select the **Syslog** tab to view existing syslog targets.
4. Click the **+** icon (Add) to create a new syslog destination for the Elastic Agent.
5. In the configuration window, enter the following:
   - **Syslog Name:** Enter a unique name (e.g., `Elastic_Agent_Target`).
   - **IP Address / Hostname:** The IP address of the machine running the Elastic Agent.
   - **Port:** Set to `9509` (or the port defined in your Kibana configuration).
   - **Protocol:** Choose `TCP` or `UDP`.
   - **Format:** Select `JSON` or `CEF` for optimal parsing.
6. In the **Notifications** pane on the right side of the screen, toggle the sliders to enable `Security Events`, `System Events`, and `Audit Trail` as needed.
7. Click **Save** to apply the export settings.

### For Enabling Event Notifications:

1. Navigate to **Security Settings > Playbooks** in the FortiEDR console.
2. Locate the Playbook policy currently applied to your Collector Groups. Click the policy to expand its settings.
3. Review the list of security event categories (e.g., **Malicious File Detected**, **Suspicious Process Terminated**).
4. For every event type that you want to send to Elastic, ensure the **Send Syslog Notification** checkbox is enabled in the Playbook actions column.
5. If the Playbook is not already assigned, ensure it is linked to the relevant **Collector Groups** under the **Assignment** tab.
6. Click **Save** to finalize the Playbook configuration. FortiEDR will now begin streaming these events to your Elastic Agent.

## Kibana set up steps

### Collecting logs from Fortinet FortiEDR instances (input: udp)
1. In Kibana, navigate to **Integrations** and search for **Fortinet FortiEDR**.
2. Click **Add Fortinet FortiEDR**.
3. Select the **Collect Fortinet FortiEDR logs (input: udp)** input type.
4. Configure the following fields:
   - **Listen Address** (`udp_host`): The bind address to listen for UDP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
   - **Listen Port** (`udp_port`): The UDP port number to listen on. Default: `9509`.
   - **Timezone offset (+HH:mm format)** (`tz_offset`): The timezone offset to apply to the logs. Default: `local`.
   - **Add non-ECS fields** (`rsa_fields`): Whether to add fields that do not map to the Elastic Common Schema. Default: `True`.
   - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
5. Save and deploy the integration.

### Collecting logs from Fortinet FortiEDR instances (input: tcp)
1. In Kibana, navigate to **Integrations** and search for **Fortinet FortiEDR**.
2. Click **Add Fortinet FortiEDR**.
3. Select the **Collect Fortinet FortiEDR logs (input: tcp)** input type.
4. Configure the following fields:
   - **Listen Address** (`tcp_host`): The bind address to listen for TCP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
   - **Listen Port** (`tcp_port`): The TCP port number to listen on. Default: `9509`.
   - **Timezone offset (+HH:mm format)** (`tz_offset`): The timezone offset to apply to the logs. Default: `local`.
   - **Add non-ECS fields** (`rsa_fields`): Whether to add fields that do not map to the Elastic Common Schema. Default: `True`.
   - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
5. Save and deploy the integration.

### Collecting logs from Fortinet FortiEDR instances (input: logfile)
1. In Kibana, navigate to **Integrations** and search for **Fortinet FortiEDR**.
2. Click **Add Fortinet FortiEDR**.
3. Select the **Collect Fortinet FortiEDR logs (input: logfile)** input type.
4. Configure the following fields:
   - **Paths**: Specify the local paths to the log files (e.g., `/var/log/fortiedr/*.log`).
   - **Timezone offset (+HH:mm format)** (`tz_offset`): The timezone offset to apply to the logs. Default: `local`.
   - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
5. Save and deploy the integration.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from Fortinet FortiEDR to the Elastic Stack.

### 1. Trigger Data Flow on Fortinet FortiEDR:
- **Generate Audit Event:** Log out of the FortiEDR Central Manager console and log back in. This will trigger an audit trail event for the user login.
- **Generate Configuration Event:** Modify a non-critical setting in **Administration > Export Settings** (such as the name of a syslog target) and click **Save**.
- **Test Playbook Action:** If a test endpoint is available, run a known safe "grayware" tool or trigger a test alert via the FortiEDR simulator to verify the Playbook's `Send Syslog Notification` action is triggered.
- **Send Test Syslog:** Use the "Send Test" function (if available in your version) within the **Syslog** export settings tab to verify connectivity.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "fortinet_fortiedr.log"`
4. Verify logs appear. Expand a log entry and confirm these fields:
   - `event.dataset` (should be `fortinet_fortiedr.log`)
   - `source.ip` and/or `destination.ip`
   - `event.action` (e.g., `login`, `file_create`, or `process_start`)
   - `message` (the raw log payload)
5. Navigate to **Analytics > Dashboards** and search for "Fortinet FortiEDR" to view pre-built visualizations.

# Troubleshooting

## Common Configuration Issues

- **Playbook Notifications Disabled**: If syslog targets are defined but no data is received, check the Playbook settings in **Security Settings > Playbooks**. Ensure the **Send Syslog Notification** action is checked for the specific event types you are monitoring.
- **Syslog Target Not Saved**: Changes in the **Export Settings** page often require clicking the **Save** button at the bottom of the list. Verify the target appears in the table after refreshing the page.
- **Protocol Mismatch**: Ensure that the protocol selected in FortiEDR (TCP or UDP) matches the input type configured in the Elastic Agent integration settings. A mismatch will prevent the listener from acknowledging or processing packets.
- **Port Conflict**: If the Elastic Agent fails to start the syslog listener, check if another service is already using port `9509` on the host machine using `netstat -ano` or `ss -tuln`.

## Ingestion Errors

- **Parsing Failures**: If logs appear in Kibana but fields are not correctly mapped, check the **Format** setting in FortiEDR **Export Settings**. This integration is optimized for standard syslog formats; switching to JSON or CEF in the FortiEDR console usually resolves mapping issues.
- **Timestamp Mismatches**: If logs appear with the wrong time, verify the `tz_offset` variable in the Kibana configuration matches the timezone of the FortiEDR Central Manager.
- **Field Length Limits**: Extremely large event payloads may be truncated by syslog transport. Check `error.message` in Kibana to see if the Elastic Agent encountered malformed messages or unexpected end-of-line characters.

## Vendor Resources
- [FortiEDR Administration Guide: Syslog Configuration](https://docs.fortinet.com/document/fortiedr/5.0.0/administration-guide/109591/syslog)
- [FortiEDR Administration Guide: Automated Incident Response Playbooks](https://docs.fortinet.com/document/fortiedr/5.0.0/administration-guide/419440/automated-incident-response-playbooks-page)

# Documentation sites

- [FortiEDR Administration Guide: Syslog](https://docs.fortinet.com/document/fortiedr/5.0.0/administration-guide/109591/syslog)
- [FortiEDR Administration Guide: Playbooks](https://docs.fortinet.com/document/fortiedr/5.0.0/administration-guide/419440/automated-incident-response-playbooks-page)
- Refer to the official vendor website for additional resources.
