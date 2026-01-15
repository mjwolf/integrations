# Service Info

The QNAP NAS integration allows organizations to centralize logs from QNAP Network Attached Storage devices into the Elastic Stack for real-time monitoring and security auditing. By collecting system events and connection logs, administrators can gain deep visibility into storage health, user activities, and potential security threats across their QTS-based infrastructure.

## Common use cases

The QNAP integration allows users to centralize and analyze logs from QNAP Network Attached Storage (NAS) devices running QTS, QuTS hero, or QNE Network operating systems. By forwarding logs to the Elastic Stack, administrators can gain deep visibility into the health, security, and operational status of their storage infrastructure.
- **Security Auditing and Compliance:** Monitor user login activities, failed authentication attempts, and SSH access to detect potential brute-force attacks or unauthorized access to sensitive data.
- **System Health Monitoring:** Track hardware-related events such as disk failures, RAID rebuilding status, fan malfunctions, and temperature alerts to prevent data loss and minimize downtime.
- **File Access Tracking:** Analyze access logs (Samba, FTP, AFP) to monitor file operations and ensure data governance policies are being followed across the organization's shared folders.
- **Administrative Oversight:** Capture and review configuration changes made within the QTS or QuTS hero web interface to maintain an audit trail of administrative actions.

## Data types collected
This integration can collect the following types of data from QNAP NAS devices:
- **System Event Logs:** Captured via Syslog, these include records of system startup/shutdown, firmware updates, hardware failures, and service status changes.
- **System Connection Logs (Access Logs):** Detailed records of user connections via protocols such as SMB/CIFS, AFP, FTP, HTTP/HTTPS, and SSH, including source IP and action performed.
- **Data Formats:** Logs are transmitted in standard Syslog format over UDP or TCP, depending on the QTS version and configuration.
- **Log Categories:** The integration categorizes events into "Information," "Warning," and "Error" levels to facilitate prioritized alerting and filtering within Kibana.

## Compatibility
The integration is compatible with **QNAP QTS** firmware versions. Specifically:
- **QNAP QTS 4.5.2 and later** using the **QuLog Center** application for advanced log management.
- **QNAP QTS 4.5.1 and earlier** using the legacy **System Logs** management interface.
- Support is generally applicable to all QNAP NAS models running these firmware versions.

## Scaling and Performance
- **Transport/Collection Considerations:** Users can choose between UDP and TCP protocols for log transmission. While UDP offers lower overhead and higher throughput, TCP is recommended for environments where log delivery reliability is critical, as it ensures packets are not dropped during transit.
- **Data Volume Management:** To reduce the volume of data sent to the Elastic Stack, administrators should selectively enable log types. For high-traffic NAS environments, it is recommended to enable "Event Logs" for security and health while limiting "Access Logs" to specific sensitive shares or protocols to prevent overwhelming the ingest pipeline.
- **Elastic Agent Scaling:** A single Elastic Agent can typically handle logs from dozens of QNAP NAS devices in standard environments. For high-volume enterprise deployments generating thousands of events per second, consider deploying multiple Elastic Agents behind a load balancer to distribute the ingestion load and provide high availability.

# Set Up Instructions

## Vendor prerequisites
- **Administrative Access:** You must have a user account with administrator privileges for the QNAP QTS web interface.
- **Network Connectivity:** The QNAP NAS must have network reachability to the Elastic Agent. Ensure that any intermediate firewalls allow traffic on the designated Syslog port (e.g., UDP/TCP 514 or 1514).
- **Firmware Check:** Verify the firmware version of your NAS to determine whether to use the QuLog Center (4.5.2+) or legacy System Logs (4.5.1-) configuration path.
- **App Installation:** For newer versions, ensure the **QuLog Center** application is installed and updated via the QNAP App Center.
- **Internal IP Knowledge:** Have the static IP address or FQDN of your Elastic Agent host ready for configuration.

## Elastic prerequisites
- **Elastic Agent:** An Elastic Agent must be installed and enrolled in a policy via Fleet.
- **Integration Asset:** The QNAP NAS integration must be added to the Agent policy.
- **Connectivity:** The host running the Elastic Agent must be configured to listen on a network interface accessible by the QNAP NAS and have the specified port open in the OS firewall.

## Vendor set up steps

### For QTS 4.5.2 and later (using QuLog Center):
1. Log in to your QNAP QTS web interface using your administrator credentials.
2. Open the **App Center** and ensure **QuLog Center** is installed, then open the application.
3. In the QuLog Center interface, navigate to the **Log Sender** tab located in the left-hand menu.
4. Locate the section labeled **Remote Syslog Server Settings** and check the box for **Send logs to a remote syslog server**.
5. Click the **+ Add Destination** button to open the configuration wizard.
6. In the **IP Address/Hostname** field, enter the IP address of your Elastic Agent.
7. Set the **Port** number to match your Elastic Agent configuration (e.g., `514` or `1514`).
8. Select the **Protocol** (UDP or TCP) that matches your Elastic Agent's input settings.
9. Under the **Log Type** section, select both **Event Logs** and **Access Logs** to ensure comprehensive coverage.
10. Click **Apply** and then **Test** (if available) to verify the connection, followed by **Apply** on the main screen to save the settings.

### For QTS 4.5.1 and earlier (using System Logs):
1. Log in to your QNAP QTS web interface as an administrator.
2. Open the **Control Panel** from the desktop or main menu.
3. Navigate to **System > System Logs**.
4. Select the **Syslog Client Management** tab at the top of the window.
5. Check the box to **Enable Syslog**.
6. In the **Syslog server** field, enter the IP address of the Elastic Agent.
7. Enter the **UDP Port** number your Elastic Agent is listening on (default is often `514`). Note that legacy versions typically only support UDP.
8. Under the **Select the logs to record** section, check the boxes for both **system event logs** and **system connection logs**.
9. Click the **Apply** button at the bottom of the screen to commit the configuration.
10. Verify the status indicator shows the service is active and sending data.

## Kibana set up steps

### For QNAP Syslog Input:
1. In Kibana, navigate to **Management > Integrations**.
2. Search for **QNAP** and select the integration.
3. Click **Add QNAP**.
4. Select the **Agent Policy** where you want to add this integration.
5. In the integration configuration, locate the **Syslog** input section.
6. Configure the following fields based on your QNAP settings:
    *   **Syslog Host**: Set this to `0.0.0.0` to listen on all interfaces, or the specific IP of the Agent host.
    *   **Syslog Port**: Enter the port you chose in the QNAP vendor setup (e.g., `9002`).
    *   **Protocol**: Select the protocol that matches your QNAP configuration (`tcp` or `udp`).
7. Expand the **Advanced options** if you need to adjust internal timeouts or specify a custom `tags` field for organizational purposes.
8. Click **Save and continue**.
9. Click **Add agent integration to policy** to deploy the changes to your Elastic Agents.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from QNAP to the Elastic Stack.

### 1. Trigger Data Flow on QNAP:
- **Generate authentication event:** Log out of the QNAP web interface and then log back in using your administrator credentials. This should trigger a "User login" event.
- **Generate configuration event:** Navigate to **Control Panel > General Settings** and toggle a non-critical setting (like the system time display format) and click apply to generate a configuration change log.
- **Trigger system test:** In the **QuLog Center > Log Sender** settings, use the **Send a Test Message** button to manually push a test syslog packet to the Elastic Agent.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the following KQL filter: `data_stream.dataset : "qnap.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `qnap.log`)
   - `log.syslog.priority` (identifies the severity of the QNAP event)
   - `message` (contains the actual text of the QNAP log, e.g., "User [admin] logged in from...")
   - `observer.ip` (should match the IP address of your QNAP NAS)
   - `event.module` (should be `qnap`)
5. Navigate to **Analytics > Dashboards** and search for "QNAP" to view the pre-built dashboards and visualize your storage event trends.

# Troubleshooting

## Common Configuration Issues

- **Port Binding Failure**: If the Elastic Agent fails to start the Syslog listener, verify that another service (like a local rsyslog or another Agent integration) is not already using the configured port. Check the Agent logs for "address already in use" errors.
- **Firewall Obstruction**: If no logs appear in Discover, ensure the host running the Elastic Agent allows inbound traffic on the Syslog port. Use `tcpdump` or `Wireshark` on the Agent host to verify that packets are arriving from the QNAP IP.
- **Protocol Mismatch**: Ensure the protocol (TCP or UDP) selected in the QNAP QuLog Center exactly matches the protocol selected in the Kibana integration settings. A mismatch will result in a connection failure or silent data loss.
- **Incorrect IP Configuration**: Double-check that the "Hostname/IP Address" field in QuLog Center points to the internal IP of the Agent, and not the loopback address (127.0.0.1).

## Ingestion Errors
- **Parsing Failures**: If logs appear in Kibana but contain a `_grokparsefailure` or `_jsonparsefailure` tag, the log format sent by the QNAP device may differ from the expected format. Check the `error.message` field in Discover for specific clues.
- **Timezone Mismatch**: If logs appear to be "missing" in Discover, they may be indexed with a future or past timestamp. Ensure the QNAP NAS time and timezone settings are synchronized with an NTP server and match the expectations of the Elastic Stack.
- **Truncated Logs**: If using UDP for very long log messages, packets may be truncated. Switch the configuration to TCP if your QTS version supports it to handle larger log payloads.

## Vendor Resources

- [Sending System Logs to a Syslog Server - QNAP Systems](https://docs.qnap.com/operating-system/qne-network/1.0.x/en-us/sending-system-logs-to-a-syslog-server-35997D85.html)
- [QuLog Center | QTS 5.0.x - QNAP Systems](https://docs.qnap.com/operating-system/qts/5.0.x/en-us/qulog-center-A7E77980.html)

# Documentation sites

- [QNAP | Logmanager documentation](https://doc.logmanager.com/latest/log-source-devices/qnap/)
- Refer to the official vendor website.
