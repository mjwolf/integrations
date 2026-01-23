# Service Info

## Common use cases

The QNAP NAS integration allows organizations to centralize logs from their Network Attached Storage devices into the Elastic Stack for enhanced visibility, security auditing, and system health monitoring.
- **Security Auditing and Compliance:** Monitor user access logs to track who is accessing specific files or folders, and identify failed login attempts that might indicate brute-force attacks.
- **System Health Monitoring:** Collect system event logs to stay informed about hardware status, RAID rebuilds, firmware updates, and power events that could impact data availability.
- **Network Troubleshooting:** Analyze connection logs to diagnose issues with network protocols like SMB, AFP, or FTP, ensuring consistent performance for connected clients.
- **Regulatory Retention:** Forward logs to a centralized Elastic instance to satisfy long-term log retention requirements that might exceed the local storage capacity or retention policy of the NAS itself.

## Data types collected
This integration collects log data from QNAP devices via the Syslog protocol. Based on the data stream configuration, this integration includes the following:

- **QNAP NAS logs (TCP):** Collect QNAP NAS event and access logs using the TCP input type for reliable, connection-oriented log delivery. This data stream is defined as `qnap_nas.log` and is used to collect QNAP NAS logs using TCP input.
- **QNAP NAS logs (UDP):** Collect QNAP NAS event and access logs using the UDP input type for low-overhead log transmission. This data stream is defined as `qnap_nas.log` and is used to collect QNAP NAS logs using UDP input.

Specific data categories include:
- **System Event Logs:** Captures information, warnings, and errors related to the operating system, hardware components (disks, fans, power supplies), and installed applications.
- **System Access Logs:** Records user authentication events (SSH, HTTP, SAMBA, AFP, FTP) and file-level access activities across supported protocols.
- **Data Formats:** Logs are collected in the standard RFC-3164 Syslog format, ensuring compatibility with the legacy syslog protocol and proper field mapping.

## Compatibility
The QNAP NAS integration is compatible with **QNAP NAS** devices and the Elastic Stack under the following conditions:

- **Elastic Stack:** Requires Kibana/Elasticsearch version **8.11.0** or higher.
- **QNAP Firmware:** Compatible with **QTS version 4.5.4** (tested) and newer versions.
- **Software Requirements:** The integration requires the **QuLog Center** application to be installed on the QNAP device to manage and forward logs to a remote destination.

## Scaling and Performance

To ensure optimal performance in high-volume environments, consider the following:
- **Transport/Collection Considerations:** While UDP is faster for syslog transmission and has lower overhead, TCP is recommended for environments where log delivery guarantees are required. TCP ensures no log messages are lost due to network congestion or packet drops. For encrypted transmission, ensure the Agent and NAS are configured for TLS over TCP.
- **Data Volume Management:** Configure the QNAP QuLog Center to forward only necessary events (e.g., Warning and Error levels). For high-activity file servers, **Access Logs** can generate high volume; evaluate if file-level auditing is required for all shares or just sensitive directories to reduce ingest load and storage costs.
- **Elastic Agent Scaling:** For high-throughput environments with many NAS devices, deploy multiple Elastic Agents behind a network load balancer to distribute traffic evenly. Place Agents close to the data source to minimize latency and ensure the host has sufficient CPU resources to handle RFC-3164 parsing at high events-per-second (EPS) rates.

# Set Up Instructions

## Vendor prerequisites

1. **Administrative Access:** You must have an administrator account for the QNAP NAS to access the QTS desktop and QuLog Center settings.
2. **QuLog Center Installation:** Ensure the **QuLog Center** application is installed via the QNAP App Center. Most modern QTS versions include this by default.
3. **Network Connectivity:** The QNAP NAS must have network reachability to the Elastic Agent host. Ensure that any firewalls between the NAS and the Agent allow traffic on the configured syslog port (default is `9301`).
4. **Format Requirements:** The remote syslog server configuration on the QNAP must be set to use the **RFC-3164** format.
5. **License/Feature Check:** Confirm that your NAS model supports remote log forwarding (standard on most QNAP SMB and Enterprise models).

## Elastic prerequisites
- **Elastic Agent:** An active Elastic Agent must be installed and enrolled in a policy via Fleet.
- **Connectivity:** The Elastic Agent must be reachable from the QNAP NAS IP address. Ensure that the Agent's host firewall permits inbound traffic on the Syslog port (e.g., **9301**).
- **Minimum Version:** Ensure your Elastic Stack is running version **8.11.0** or higher.

## Vendor set up steps

### For QTS 4.5.2 and Newer (QuLog Center):
1. Log in to your QNAP QTS web interface as an administrator.
2. Open the **Control Panel** from the desktop or main menu.
3. Navigate to **System > QuLog Center**.
4. In the QuLog Center application, select **Log Sender** from the left-hand navigation menu.
5. Click on the **Send to Syslog Server** tab at the top of the interface.
6. Enable the checkbox labeled **Send logs to a remote Syslog Server**.
7. Click the **+Add Destination** button to open the configuration wizard.
8. Enter the following configuration details:
    *   **Destination Name**: A descriptive name (e.g., `Elastic-Agent`).
    *   **IP Address**: The IP address of the server running the Elastic Agent.
    *   **Port**: The port number configured in your Elastic integration (default is `9301`).
    *   **Protocol**: Select **UDP**, **TCP**, or **TLS** (ensure this matches your Kibana input configuration).
    *   **Log Type**: Select **Event & Access Logs** to ensure comprehensive coverage.
9. Click **Apply** and then **Test** to verify connectivity.

### For Older QTS Versions (Legacy System Logs):
1. Log in to your QNAP QTS web interface as an administrator.
2. Open the **Control Panel**.
3. Navigate to **System > System Logs**.
4. Click on the **Syslog Client Management** tab or button.
5. Enable the checkbox for **Enable Syslog**.
6. Configure the remote server settings:
    *   **Syslog server**: Enter the IP address of the Elastic Agent host.
    *   **Port**: Enter the port (e.g., `9301`).
    *   **Protocol**: Select **UDP** or **TCP**.
    *   **Select the logs to record**: Ensure **System Event Logs** and **System Connection Logs** are both checked.
7. Click **Apply** to begin forwarding logs.

## Kibana set up steps

### Collecting logs from QNAP NAS via TCP
1. In Kibana, navigate to **Integrations** > **QNAP NAS**.
2. Click **Add QNAP NAS**.
3. Select the **Collecting logs from QNAP NAS via TCP** input.
4. Configure the following variables:
   - **Syslog Host** (`syslog_host`): The interface address to listen on. Use `0.0.0.0` to listen on all interfaces. Default: `localhost`.
   - **Syslog Port** (`syslog_port`): The TCP port the agent will listen on. This must match the port configured in the QNAP Log Sender. Default: `9301`.
   - **Timezone Offset** (`tz_offset`): By default, datetimes in the logs will be interpreted as relative to the timezone configured in the host where the agent is running. If ingesting logs from a host on a different timezone, use this field to set the timezone offset so that datetimes are correctly parsed. Acceptable timezone formats are: a canonical ID (e.g. "Europe/Amsterdam"), abbreviated (e.g. "EST") or an HH:mm differential (e.g. "-05:00") from UCT. Default: `local`.
   - **SSL Configuration** (`ssl`): SSL configuration options. Provide paths to certificates and keys if using the TLS protocol in the QNAP settings.
   - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
5. Save and deploy the integration.

### Collecting logs from QNAP NAS via UDP
1. In Kibana, navigate to **Integrations** > **QNAP NAS**.
2. Click **Add QNAP NAS**.
3. Select the **Collecting logs from QNAP NAS via UDP** input.
4. Configure the following variables:
   - **Syslog Host** (`syslog_host`): The interface address to listen on for UDP packets. Default: `localhost`.
   - **Syslog Port** (`syslog_port`): The UDP port the agent will listen on. This must match the port configured in the QNAP Log Sender. Default: `9301`.
   - **Timezone Offset** (`tz_offset`): By default, datetimes in the logs will be interpreted as relative to the timezone configured in the host where the agent is running. If ingesting logs from a host on a different timezone, use this field to set the timezone offset so that datetimes are correctly parsed. Acceptable timezone formats are: a canonical ID (e.g. "Europe/Amsterdam"), abbreviated (e.g. "EST") or an HH:mm differential (e.g. "-05:00") from UCT. Default: `local`.
   - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
5. Save and deploy the integration.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on QNAP NAS:
- **Authentication event:** Log out of the QNAP QTS web interface and log back in, or intentionally enter an incorrect password to trigger a "Login Fail" event.
- **System event:** Navigate to **Control Panel > System > Hardware** and toggle a setting like the "Alert Buzzer" or perform a manual S.M.A.R.T. disk test.
- **Connection event:** Access a shared folder on the NAS via SMB or AFP from a client computer and create or delete a test file.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "qnap_nas.log"`
4. Verify logs appear. Expand a log entry and confirm these fields:
   - `event.dataset` (should be `qnap_nas.log`)
   - `source.ip` (the IP address of your QNAP NAS)
   - `event.action` (e.g., `login`, `connection`, or system-specific actions)
   - `message` (the raw RFC-3164 syslog payload)
5. Navigate to **Analytics > Dashboards** and search for "QNAP NAS" to verify pre-built visualizations are populating.

# Troubleshooting

## Common Configuration Issues

- **QuLog Center App Missing**: Older QTS versions or specific lite versions may not have QuLog Center installed. Go to the **App Center** and search for "QuLog Center" to install it.
- **Port Conflict**: If the Elastic Agent fails to start the syslog listener, ensure no other service on the host is using port **9301**. Use `netstat -ano | grep 9301` on Linux or `Get-NetTCPConnection -LocalPort 9301` on Windows to check.
- **Firewall Blocking**: If logs are not arriving, verify that the local firewall on the Elastic Agent host (e.g., `iptables`, `ufw`, or Windows Firewall) is configured to allow inbound traffic on the selected port and protocol.
- **Protocol Mismatch**: Ensure the **Transfer Protocol** selected in QNAP QuLog Center (TCP or UDP) matches the input type you enabled in the Kibana integration settings.

## Ingestion Errors

- **Parsing Failures**: If logs appear in Discover but have tags like `_grokparsefailure`, ensure the QNAP is set to **RFC-3164** format. Using "Common Log Format" or other proprietary formats will cause parsing errors.
- **Timezone Mismatch**: If logs appear to be in the "future" or "past" in Kibana, check the **Timezone Offset** variable in the integration settings. Ensure it matches the timezone configured on the QNAP NAS hardware.
- **Missing Fields**: If specific fields like `user.name` are missing from Access logs, ensure that "Access Logs" are specifically checked in the **Log Type** section of the QNAP Log Sender configuration.

## Vendor Resources

- QNAP Official Website
- [QuLog Center Quick Start Guide](how-to/tutorial/article/qulog-center-quick-start-guide)
- [Syslog Configuration on QNAP (SGBOX Knowledge Base)](https://www.sgbox.eu/en/knowledge-base/syslog-configuration-on-qnap/)

# Documentation sites

- [QNAP NAS Integration Reference](how-to/tutorial/article/qulog-center-quick-start-guide)
- [Elastic Common Schema (ECS) Documentation](https://www.elastic.co/docs/reference/ecs)
- Refer to the official QNAP support portal for model-specific QTS documentation.
