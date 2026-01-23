# Service Info

## Common use cases

The Fortinet FortiManager integration is designed to provide centralized visibility into the security fabric and network operations managed by Fortinet devices. By ingesting logs into the Elastic Stack, users can achieve the following:

-   **Centralized Security Monitoring:** Aggregate security events from the Security Console and FortiGuard services to detect and respond to threats across the entire Security Fabric from a single dashboard.
-   **Operational Auditing:** Monitor system-level events, including firmware management, script execution, and object configuration changes, to maintain a detailed audit trail of administrative actions.
-   **Network Health and Status:** Track the health of managed devices through the FortiGate-FortiManager (FGFM) protocol and Device Manager logs to ensure optimal network performance and connectivity.
-   **Compliance and Reporting:** Utilize log data from FortiAnalyzer components, such as report generation and database status, to meet regulatory requirements and generate comprehensive compliance reports.
-   **Incident Response:** Correlate FortiManager audit logs with other security telemetry in Elasticsearch to identify the scope of unauthorized configuration changes or administrative breaches.

## Data types collected

This integration collects the following data streams:

- **Fortinet FortiManager logs (filestream)**: Collect Fortinet FortiManager logs via Filestream input. This data stream is used when the Elastic Agent has direct access to the local filesystem where FortiManager logs are stored.
- **Fortinet FortiManager logs (tcp)**: Collect Fortinet FortiManager logs via TCP input. This data stream is used to receive logs forwarded from FortiManager instances over a reliable TCP connection.
- **Fortinet FortiManager logs (udp)**: Collect Fortinet FortiManager logs via UDP input. This data stream is used to receive logs forwarded from FortiManager instances over a high-performance UDP connection.

These data streams capture comprehensive event data including:
- **System Events:** Audit logs covering administrative actions, system status, and process events.
- **Service Logs:** Specific events related to FortiGuard services, firmware management, and deployment tasks.
- **Communication Logs:** Logs detailing the interactions between FortiManager and managed FortiGate devices via the **FGFM** protocol.
- **Subtypes Supported:** Includes event subtypes for FortiManager (system, fgd, scply, fmwmgr, logd, iolog, fgfm, devmgr, dm, objcfg, scrmgr) and FortiAnalyzer (logfile, logging, logdev, logdb, fazsys, report).

## Compatibility

- **Elastic Requirements:** This integration requires Elastic Stack version **8.11.0** or higher.
- **Vendor Compatibility:** The Fortinet FortiManager integration is officially compatible with versions **7.2.2 and above**. It has been specifically tested against FortiManager and FortiAnalyzer version **7.2.2**. While versions newer than 7.2.2 are expected to be compatible due to standardized log formats, they have not been officially validated in the current test cycle.

## Scaling and Performance

To ensure optimal performance in high-volume environments, consider the following:

-   **Transport/Collection Considerations:** When choosing between TCP and UDP for syslog collection, prioritize TCP for guaranteed delivery and reliability, especially in high-compliance environments. Use UDP for higher performance and lower overhead when occasional log loss is acceptable. For local collection on the same host where logs are written to disk, use the Filestream input to ensure logs are tracked via inode to handle log rotation without data loss.
-   **Data Volume Management:** To manage data volume and reduce load on the Elastic Stack, use the FortiManager CLI to set the log severity level. For example, using `set severity information` provides a balance of detail, while `set severity warning` can significantly reduce volume by only capturing critical events. Filtering specific subtypes (e.g., excluding debug IO logs) at the source is recommended for high-throughput environments.
-   **Elastic Agent Scaling:** For high-throughput environments, deploy multiple Elastic Agents behind a network load balancer to distribute traffic evenly. Place Agents close to the data source to minimize latency. A single Elastic Agent can handle significant throughput, but in environments managing thousands of FortiGate devices, horizontal scaling is recommended.

# Set Up Instructions

## Vendor prerequisites

1. **Administrative Access:** You must have an administrative account on the FortiManager instance with permissions to modify **System Settings** and access the CLI.
2. **Network Connectivity:** Ensure that the FortiManager device can reach the Elastic Agent host over the network. By default, **TCP** or **UDP** port `9022` must be open on any intermediary firewalls.
3. **Licensing:** Ensure your FortiManager license is active and the logging features are enabled.
4. **Version Requirements:** Verify the FortiManager is running version **7.2.2** or later to ensure compatibility with the log schemas provided by this integration.

## Elastic prerequisites

1. **Kibana Version:** Ensure your Elastic Stack is version **8.11.0** or newer.
2. **Elastic Agent Enrollment:** An Elastic Agent must be installed and enrolled in Fleet. The agent must be in a **Healthy** state.
3. **Network Visibility:** The Elastic Agent host must be configured to listen on an interface reachable by the FortiManager (e.g., set the **Listen Address** to `0.0.0.0` or a specific internal IP).

## Vendor set up steps

### For TCP/UDP (Syslog) Collection:

Fortinet FortiManager requires a two-step process to send its local event logs to an external syslog server.

**Step 1: Add the Syslog Server via the Web UI**
1. Log in to the FortiManager web UI with administrative credentials.
2. Navigate to **System Settings > Advanced > Syslog Server**.
3. In the Syslog Server list, click **Create New** in the toolbar.
4. Configure the following settings:
   - **Name**: Provide a unique name, such as `elastic-agent-syslog`.
   - **IP address (or FQDN)**: Enter the IP address of the server where the Elastic Agent is installed.
   - **Syslog Server Port**: Enter the port number configured in your Elastic Agent input (default is `9022`).
   - **Reliable Connection**: Disable this for **UDP** transport; enable this for **TCP** transport.
5. Click **OK** to save the configuration.

**Step 2: Enable Log Forwarding via CLI**
1. Open an SSH session or use the console to access the FortiManager CLI.
2. Enter the configuration mode for local logging:
   ```shell
   config system locallog syslogd setting
   ```
3. Enable the service and link it to the server created in Step 1:
   ```shell
   set status enable
   set syslog-name "elastic-agent-syslog"
   ```
4. Configure the logging severity to ensure all necessary data is captured:
   ```shell
   set severity information
   ```
5. Save and apply the changes:
   ```shell
   end
   ```
6. (Optional) Verify the settings are applied correctly:
   ```shell
   get system locallog syslogd setting
   ```

### For Filestream (Local Log File) Collection:

1. Ensure the Elastic Agent is installed directly on the host that has access to the FortiManager log files.
2. Verify that the Elastic Agent user has read permissions for the log directory (typically `/var/log/fortinet/`).
3. If logs are stored in a non-standard location, note the absolute path for the Kibana configuration.

## Kibana set up steps

### Collecting logs from Fortinet FortiManager instances via filestream input.
1. In Kibana, navigate to **Integrations** and search for **Fortinet FortiManager**.
2. Click **Add Fortinet FortiManager**.
3. Under **Fortinet FortiManager logs**, ensure the **filestream** input is enabled.
4. Configure the following variables:
   - **Paths** (`paths`): A list of glob-based paths that will be crawled and fetched. Default: `['/var/log/fortinet/fortimanager.log*']`.
   - **Timezone Offset** (`tz_offset`): By default, datetimes in the logs will be interpreted as relative to the timezone configured in the host where the agent is running. If ingesting logs from a host on a different timezone, use this field to set the timezone offset so that datetimes are correctly parsed. Acceptable timezone formats are: a canonical ID (e.g. "Europe/Amsterdam"), abbreviated (e.g. "EST") or an HH:mm differential (e.g. "-05:00") from UCT. Default: `local`.
   - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
5. Click **Save and continue**.

### Collecting logs from Fortinet FortiManager instances via tcp input.
1. In Kibana, navigate to **Integrations** and search for **Fortinet FortiManager**.
2. Click **Add Fortinet FortiManager**.
3. Under **Fortinet FortiManager logs**, ensure the **tcp** input is enabled.
4. Configure the following variables:
   - **Listen Address** (`listen_address`): The bind address to listen for TCP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
   - **Listen Port** (`listen_port`): The TCP port number to listen on. Default: `9022`.
   - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
5. Click **Save and continue**.

### Collecting logs from Fortinet FortiManager instances via udp input.
1. In Kibana, navigate to **Integrations** and search for **Fortinet FortiManager**.
2. Click **Add Fortinet FortiManager**.
3. Under **Fortinet FortiManager logs**, ensure the **udp** input is enabled.
4. Configure the following variables:
   - **Listen Address** (`listen_address`): The bind address to listen for UDP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
   - **Listen Port** (`listen_port`): The UDP port number to listen on. Default: `9022`.
   - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
5. Click **Save and continue**.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on FortiManager:
- **Generate authentication event:** Log out of the FortiManager GUI and log back in as an administrator.
- **Generate configuration event:** Navigate to **Device Manager**, enter a configuration menu for a managed device, make a non-destructive change (like a description), and save it.
- **Generate system event:** Enter the CLI and run a simple command like `get system status` or enter and exit configuration mode using `config system global` followed by `end`.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "fortinet_fortimanager.log"`
4. Verify logs appear. Expand a log entry and confirm these fields:
   - `event.dataset` (should be `fortinet_fortimanager.log`)
   - `source.ip` (the IP of the FortiManager instance)
   - `event.action` or `event.outcome`
   - `message` (containing the raw log payload)
5. Navigate to **Analytics > Dashboards** and search for "Fortinet FortiManager" to view pre-built visualizations.

# Troubleshooting

## Common Configuration Issues

- **Port Conflict**: If the Elastic Agent fails to start the TCP/UDP listener, verify that the configured port (default `9022`) is not being used by another process using `netstat -tulpn | grep 9022`.
- **Bind Address Misconfiguration**: If the Elastic Agent is set to `localhost`, it will not accept connections from external FortiManager devices. Ensure the **Listen Address** is set to `0.0.0.0` or the specific IP of the Agent's network interface.
- **Firewall Obstruction**: If logs are not arriving, check the host firewall (e.g., `iptables` or `firewalld`) and network firewalls to ensure the Syslog port is permitted.
- **CLI Configuration Missing**: Ensure the `set status enable` command was run in the FortiManager CLI; defining the server in the GUI is insufficient to start the log flow.

## Ingestion Errors

- **Parsing Failures**: Check the `error.message` field in Kibana Discover. This often indicates that the log format from a specific FortiManager version has changed or contains unexpected characters.
- **Timezone Mismatch**: If logs appear to be "from the future" or delayed, ensure the `tz_offset` variable in the Kibana configuration matches the timezone configured on the FortiManager device.
- **Original Event Missing**: If you need to debug parsing issues but `event.original` is empty, ensure **Preserve original event** is set to `True` in the integration settings.

## Vendor Resources

- [Log Forwarding | FortiManager 7.2.1 | Fortinet Document Library](https://docs.fortinet.com/document/fortimanager/7.2.1/fortiaiops-1-1-0-user-guide/46437/log-forwarding)
- [How to send FortiManager local event logs ... - Fortinet Community](https://community.fortinet.com/t5/FortiManager/Technical-Tip-How-to-send-FortiManager-local-event-logs-to/ta-p/206352)
- [FortiManager & FortiAnalyzer Log Reference](https://fortinetweb.s3.amazonaws.com/docs.fortinet.com/v2/attachments/5a0d548a-12b0-11ed-9eba-fa163e15d75b/FortiManager_%26_FortiAnalyzer_7.2.1_Log_Reference.pdf)

## Documentation sites

- https://docs.fortinet.com/document/fortimanager/7.2.2/administration-guide/374190/syslog-server
- https://fortinetweb.s3.amazonaws.com/docs.fortinet.com/v2/attachments/5a0d548a-12b0-11ed-9eba-fa163e15d75b/FortiManager_%26_FortiAnalyzer_7.2.1_Log_Reference.pdf
- https://help.fortinet.com/fmgr/vm-install/56/Resources/HTML/0000_OnlineHelp%20Cover.htm
- Refer to the official Fortinet Documentation Library for additional administration guides.
