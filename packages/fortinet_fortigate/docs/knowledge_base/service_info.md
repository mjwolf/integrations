# Service Info

## Common use cases

The Fortinet FortiGate Firewall Logs integration for Elastic enables the collection and analysis of security events, traffic logs, and system activities from Fortinet devices. This integration helps security and network teams gain deep visibility into their network perimeter and internal segments.

*   **Real-time Threat Detection:** Leverage Elastic SIEM to detect and respond to threats identified in UTM logs, such as antivirus matches, IPS triggers, and malicious DNS requests.
*   **Network Traffic Analysis:** Use pre-built Kibana dashboards to visualize traffic patterns, identify bandwidth-intensive applications, and optimize network performance based on source and destination trends.
*   **Compliance and Auditing:** Maintain long-term, searchable archives of firewall policy decisions and configuration changes to meet regulatory requirements and internal security audits.
*   **VPN and Remote Access Monitoring:** Monitor VPN connection attempts, session durations, and authentication results to troubleshoot remote access issues and detect unauthorized access attempts.
*   **Operational Health Monitoring:** Track system-level events such as High Availability (HA) failovers, resource exhaustion (CPU/Memory), and administrative changes to ensure firewall uptime.

## Data types collected

This integration collects several categories of security and network telemetry. Based on the data stream configuration, the following streams are available:

- **Fortinet firewall logs (tcp)**: Collect Fortinet firewall logs using tcp input. This stream processes logs received over TCP connections, typically used for reliable log delivery. (Data type: `logs`)
- **Fortinet firewall logs (udp)**: Collect Fortinet firewall logs using udp input. This stream processes logs received over UDP, suitable for high-volume environments where minimal overhead is required. (Data type: `logs`)
- **Fortinet FortiGate logs (log)**: Collect Fortinet FortiGate logs using log input. This stream monitors and reads logs directly from local files on the system where the Elastic Agent is running. (Data type: `logs`)

The specific log content includes:
- **Traffic logs**: Records of firewall decisions to allow or deny traffic, including source/destination IPs, ports, and protocols.
- **UTM logs**: Specialized security events from Antivirus, Web Filter, Application Control, IPS, and DNS Filter modules.
- **Event logs**: System-level logs covering high-availability status, configuration changes, and general system health.
- **Authentication logs**: Comprehensive records of administrator logins, VPN authentication attempts, and user-level access events.

## Compatibility

This integration is compatible with **Fortinet FortiGate** firewalls running **FortiOS versions 6.x and 7.x**. It has been specifically tested against versions up to **FortiOS 7.4.1**. 

The following Elastic requirements must be met:
*   **Elastic Stack version:** 8.11.0 or higher.
*   **Elastic Agent:** Version 8.11.0 or higher.

## Scaling and Performance

To ensure optimal performance in high-volume environments, consider the following:

*   **Transport/Collection Considerations:** When high reliability is required, TCP is the preferred protocol. FortiGate supports "reliable mode" for TCP syslog, which should be used with `rfc6587` framing to ensure log integrity. For high-throughput environments where some data loss is acceptable, UDP can be used to reduce overhead on the firewall and the Elastic Agent.
*   **Data Volume Management:** To prevent overwhelming the ingest pipeline, use the FortiGate CLI `config log syslogd filter` to exclude high-volume, low-value logs such as `local-traffic`. Setting the severity level to `information` or higher is recommended to capture necessary security data while filtering out debugging noise at the source.
*   **Elastic Agent Scaling:** For high-throughput environments (e.g., multi-gigabit traffic with tens of thousands of events per second), deploy multiple Elastic Agents behind a network load balancer to distribute traffic evenly. Place Agents geographically close to the firewalls to minimize network latency and potential packet loss for UDP.

# Set Up Instructions

## Vendor prerequisites

1.  **Administrative Access:** You must have `super_admin` or equivalent permissions on the FortiGate device to modify log settings via the GUI or CLI.
2.  **Network Connectivity:** Ensure the FortiGate device can reach the Elastic Agent host on the configured port (default **9004**) and that no intermediate firewalls or Access Control Lists (ACLs) block this traffic.
3.  **Logging Enabled:** Logging must be globally enabled on the FortiGate device. Navigate to **Log & Report > Log Settings** to verify.
4.  **Policy Configuration:** Specific security profiles (Antivirus, IPS, Web Filter) must have logging enabled within their respective firewall policies to generate data.

## Elastic prerequisites

- **Elastic Stack Version**: Ensure your environment is running version **8.11.0** or higher.
- **Elastic Agent**: An Elastic Agent must be installed and enrolled in Fleet on a host capable of receiving syslog traffic or accessing the FortiGate log files.
- **Integration Asset Installation**: The Fortinet FortiGate integration must be installed in Kibana via the Integrations app before the Agent can begin processing data.

## Vendor set up steps

### For Syslog (UDP/TCP) Collection:

You can configure FortiGate to send logs using either the web-based GUI or the command-line interface (CLI).

**Method 1: GUI Configuration**
1. Log in to the FortiGate web-based manager.
2. Navigate to **Log & Report > Log Settings**.
3. Locate the **Remote Logging and Archiving** section and enable **Send Logs to Syslog**.
4. In the **IP Address/FQDN** field, enter the IP address of the Elastic Agent host.
5. In the **Port** field, enter the port configured in your Elastic integration (e.g., `9004`).
6. Set the **Log Format** to **CEF** for optimal parsing with the Elastic integration.
7. Under **Log Settings**, ensure that **Event Logging** and **Traffic Logging** (including Forward and Local Traffic) are selected.
8. Click **Apply** to save the changes.

**Method 2: CLI Configuration (Recommended for TCP/Reliable Syslog)**
1. Log in to the FortiGate CLI via SSH or the console.
2. Enter the following commands to configure the syslog destination:
   ```sh
   config log syslogd setting
       set status enable
       set server "<ELASTIC_AGENT_IP>"
       set port <PORT_NUMBER>
       set mode reliable
       set format rfc6587
       set facility local7
   end
   ```
3. To refine which logs are sent, configure the syslog filter:
   ```sh
   config log syslogd filter
       set severity information
       set forward-traffic enable
       set local-traffic enable
       set utm-event enable
       set event enable
   end
   ```
4. Verify the configuration by running `get log syslogd setting`.

### For Logfile Collection:

1. Log in to the host where the Elastic Agent is installed.
2. Ensure the FortiGate logs are being written to a local directory (e.g., via a local syslog daemon like rsyslog or a mounted network share).
3. Verify that the Elastic Agent has read permissions for the log directory and files (e.g., `/var/log/fortinet-firewall.log`).
4. Note the absolute path of the log file for use in the Kibana configuration.

## Kibana set up steps

In Kibana, navigate to **Integrations** > **Fortinet FortiGate** and click **Add Fortinet FortiGate**. Configure the following inputs as needed for your environment:

### Collecting logs from Fortinet FortiGate instances (input: tcp)
This input type allows the agent to listen for logs sent over TCP.
1. Enable the **Collect Fortinet FortiGate logs (input: tcp)** section.
2. **Listen Address** (`syslog_host`): Specify the bind address to listen for TCP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
3. **Listen Port** (`syslog_port`): Specify the TCP port number to listen on. Default: `9004`.
4. **Preserve original event** (`preserve_original_event`): If checked, this preserves a raw copy of the original event in the `event.original` field. Default: `False`.
5. Under **Advanced Options**, ensure **Framing** is set to match the vendor configuration (e.g., `rfc6587`).

### Collecting logs from Fortinet FortiGate instances (input: udp)
This input type allows the agent to listen for logs sent over UDP.
1. Enable the **Collect Fortinet FortiGate logs (input: udp)** section.
2. **Listen Address** (`syslog_host`): Specify the bind address to listen for UDP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
3. **Listen Port** (`syslog_port`): Specify the UDP port number to listen on. Default: `9004`.
4. **Preserve original event** (`preserve_original_event`): If checked, this preserves a raw copy of the original event in the `event.original` field. Default: `False`.

### Collecting logs from Fortinet FortiGate instances (input: logfile)
This input type allows the agent to read logs directly from a file.
1. Enable the **Collect Fortinet FortiGate logs (input: logfile)** section.
2. **Paths** (`paths`): Provide a list of absolute file paths to monitor. Example: `/var/log/fortinet-firewall.log`.
3. **Preserve original event** (`preserve_original_event`): If checked, this preserves a raw copy of the original event in the `event.original` field. Default: `False`.

Save the integration to an Elastic Agent policy to deploy the configuration.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly.

### 1. Trigger Data Flow on Fortinet FortiGate:
- **Generate configuration event:** Log in to the CLI and enter `config system global`, then `set hostname [current_name]`, and `end` to trigger a configuration change log.
- **Generate web traffic:** From a device behind the FortiGate, browse to several different websites to generate traffic and web filter logs.
- **Generate authentication event:** Log out of the FortiGate GUI and log back in to generate system authentication logs.
- **Trigger security event:** If safe, browse to a known blocked category (e.g., a test site categorized as "Gambling" if your policy blocks it) to trigger a UTM/Web Filter event.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "fortinet_fortigate.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields:
   - `event.dataset` (should be `fortinet_fortigate.log`)
   - `source.ip` and/or `destination.ip`
   - `event.action` or `event.outcome`
   - `fortinet.fortigate.subtype`
   - `message` (should contain the raw log payload starting with `date=... time=...`)
5. Navigate to **Analytics > Dashboards** and search for "Fortinet FortiGate Overview" to verify visualizations are populating.

# Troubleshooting

## Common Configuration Issues

- **TCP Framing Mismatch**: If you see malformed logs or parsing errors in Kibana, ensure that the FortiGate `format` is set to `rfc6587` and the Elastic Integration `framing` setting (under Advanced Options) is also set to `rfc6587`.
- **Policy Logging Disabled**: If system logs arrive but traffic logs are missing, navigate to **Policy & Objects > Firewall Policy** and ensure **Log Allowed Traffic** is set to **All Sessions** for each active policy.
- **Port Conflicts**: If the Elastic Agent fails to start the listener, check if another service is using the configured port (e.g., 514). Use `netstat -tuln` on the Agent host to verify port availability.
- **Network Unreachable**: Use `ping` or `nc -zv <agent_ip> <port>` from the FortiGate CLI to confirm the firewall can reach the Agent host over the network.

## Ingestion Errors

- **Parsing Failures**: Look for the `error.message` or `tags` field containing `_grokparsefailure` or `_jsonparsefailure` in Kibana Discover. This often indicates the log format on FortiGate is not set to standard Key-Value or CEF.
- **Timestamp Mismatches**: If logs appear "in the future" or "in the past," verify the timezone settings on both the FortiGate and in the Elastic integration advanced options.
- **Logfile Permissions**: For the logfile input, ensure the `elastic-agent` user has read access to the directory and the log files. Use `ls -l` to verify permissions.

## Vendor Resources

- [FortiGate CLI Reference - Syslog Settings](document/fortigate/7.4.0/cli-reference/405620/config-log-syslogd-setting)
- Fortinet Documentation Library
- [Technical Tip: How to send only selected logs to Syslog server from FortiGate](https://community.fortinet.com/t5/FortiGate/Technical-Tip-How-to-send-only-selected-logs-to-Syslog-server/ta-p/411370)

## Documentation sites

- [FortiGate CLI Reference - Syslog Settings](document/fortigate/7.4.0/cli-reference/405620/config-log-syslogd-setting)
- [FortiGate Administration Guide](product/fortigate)
- [fortinet.fortios.fortios_log_syslogd_setting module – Ansible Documentation](https://docs.ansible.com/projects/ansible/latest/collections/fortinet/fortios/fortios_log_syslogd_setting_module.html)
