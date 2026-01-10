# Service Info

## Common use cases

- **Security Auditing:** Monitor unauthorized access attempts, SSH logins, and administrative changes to ensure device security.
- **File Integrity & Access Monitoring:** Track file operations (create, delete, modify) across SMB, FTP, and File Station to maintain data governance.
- **Hardware Health & Performance:** Proactively identify disk failures, RAID rebuilds, and power issues through system event logs to prevent downtime.

## Data types collected

- **System Logs:** Operational events including startup/shutdown, hardware status, and application errors.
- **Access Logs:** Connection data for services like HTTP/HTTPS, SMB, AFP, FTP, and Telnet/SSH.
- **Event Logs:** Security-related events and system configuration changes.

## Compatibility

- **QNAP QTS 4.5.2 and later:** Compatible via the QuLog Center application.
- **QNAP QTS 4.5.1 and earlier:** Compatible via the legacy System Logs/Syslog Client Management menu.

## Scaling and Performance

Performance is dependent on the volume of activity on the QNAP NAS and the throughput capacity of the network and Elastic Agent. For high-traffic environments (1000+ events/sec), it is recommended to use the UDP protocol for lower overhead or ensure the Elastic Agent is scaled horizontally.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** Log in credentials for the QTS web interface with administrator privileges.
- **Network Connectivity:** Ensure the QNAP device can reach the Elastic Agent host over the network on the designated syslog port (typically UDP/TCP 514).
- **Firmware:** Ensure the device is running a supported version of QTS or QuTS hero.

## Elastic prerequisites

- **Elastic Agent:** An active Elastic Agent must be installed and running on a host accessible by the QNAP device.
- **Integration Policy:** The Syslog or QNAP integration must be added to the Agent policy.
- **Listener Configuration:** The integration must be configured to listen on a specific port and protocol (e.g., UDP 514) that matches the QNAP settings.

## Vendor set up steps

### For QNAP QTS 4.5.2 and later (using QuLog Center)
1. Log in to the QNAP QTS web interface and open **Control Panel**.
2. Navigate to **System > QuLog Center**.
3. Select **Log Sender** from the left-hand menu and click the **Send to Syslog Server** tab.
4. Enable the checkbox for **Send logs to a remote syslog server**.
5. Click **+ Add Destination** and enter the **IP Address** of the Elastic Agent.
6. Set the **Port** (e.g., 514) and **Protocol** (UDP or TCP) to match your Elastic Agent configuration.
7. Select **IETF (RFC 5424)** for the Log Format for optimal parsing.
8. Under **Log Type**, select **Event and Access Logs** and click **Apply**.

### For QNAP QTS 4.5.1 and earlier (using System Logs)
1. Log in to the QNAP QTS web interface and open **Control Panel**.
2. Navigate to **System > System Logs** and click on **Syslog Client Management**.
3. Check the box to **Enable Syslog**.
4. Enter the **Syslog server** IP address and the **UDP Port** matching your Elastic Agent.
5. In the "Select the logs to record" section, check **System Event Logs** and **System Connection Logs**.
6. Click **Apply** to save the changes.

## Kibana set up steps

1. In Kibana, go to **Management > Integrations**.
2. Search for and select the **QNAP** integration.
3. Click **Add QNAP**.
4. Configure the integration settings by specifying the **Syslog Host** (e.g., `0.0.0.0`) and **Syslog Port** (e.g., `514`) where the agent will listen.
5. Select the **Existing host policy** where the Elastic Agent is enrolled.
6. Click **Save and continue** to deploy the configuration.

# Validation Steps

1. **Trigger a Test Event:** Perform a logout/login on the QNAP NAS or change a non-critical system setting to generate a log entry.
2. **Check Data Stream:** In Kibana, navigate to **Observability > Logs** or **Discover**.
3. **Filter Data:** Filter by `event.dataset : "qnap.log"` or search for the QNAP device's IP address.
4. **Verify Dashboards:** If the integration includes pre-built dashboards, open the **[Logs QNAP] Overview** dashboard to verify visual data population.

# Troubleshooting

## Common Configuration Issues

- **Network/Firewall Blocks:** Ensure that port 514 (or your custom port) is open on the host firewall where Elastic Agent is running.
- **Protocol Mismatch:** If the QNAP is set to UDP but the Elastic Agent is listening on TCP (or vice versa), logs will not be received.
- **Incorrect IP Address:** Verify that the "Destination" IP in QuLog Center matches the actual IP of the Elastic Agent host.

## Ingestion Errors

- **Parsing Failures:** If logs appear in `logs-syslog.dataset` but contain `error.message`, ensure the Log Format in QNAP is set to **RFC 5424**.
- **Timezone Offsets:** If logs appear in the past or future, ensure both the QNAP device and the Elastic Agent host are synchronized with an NTP server.

## API Authentication Errors

Not specified (The integration primarily uses Syslog over UDP/TCP, which does not require API-based authentication).

## Vendor Resources

- [QNAP Official FAQ](https://www.qnap.com/en/how-to/faq)
- [QNAP QuLog Center User Guide](https://docs.qnap.com/operating-system/qts/5.0.x/en-us/qulog-center-A7E77980.html)
- [QNAP Support Portal](https://www.qnap.com/en/support)

# Documentation sites

- [QNAP Product Documentation](https://docs.qnap.com/)
- [Logmanager QNAP Configuration Guide](https://doc.logmanager.com/latest/log-source-devices/qnap/)
- [Logsign QNAP Syslog Setup](https://support.logsign.net/hc/en-us/articles/5579355649426-Adding-Qnap-Nas-Server-via-Syslog)