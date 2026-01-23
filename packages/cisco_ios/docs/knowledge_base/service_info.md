# Service Info

## Common use cases

The Cisco IOS integration for Elastic provides a centralized platform for ingesting, parsing, and visualizing logs from Cisco network devices, including routers and switches. This visibility is essential for maintaining network health, security posture, and operational efficiency.

-   **Security Auditing and Compliance:** Monitor logs for unauthorized access attempts, failed login events, and administrative configuration changes. This helps security teams identify potential brute-force attacks on the VTY lines or console and ensures adherence to security policies and regulatory requirements.
-   **Network Troubleshooting and Performance Monitoring:** Quickly identify and diagnose network issues such as interface flapping, protocol errors (OSPF, BGP neighbors changing state), and hardware failures by analyzing system messages in real-time.
-   **Change Management Tracking:** Maintain a detailed audit trail of when configuration mode was entered and exited. By tracking **SYS-5-CONFIG_I** events, teams can correlate network performance shifts with specific configuration updates performed by administrators.
-   **Operational Health Visibility:** Collect and visualize system-level events such as high CPU usage alerts, memory warnings, and environmental sensor data (temperature, power supply status) across the entire Cisco IOS infrastructure.
-   **Asset Management and Inventory:** Track hardware insertions and removals, such as SFP modules or line cards, to maintain an accurate view of the physical network state.

## Data types collected

This integration collects several variants of logs based on the transmission method. According to the data stream summaries, the following data streams are available:

-   **Cisco IOS logs (logs):** 
    -   **Description:** Collect Cisco IOS logs.
    -   **UDP/TCP Input:** Collects Cisco IOS logs via standard syslog protocols. These include system messages, interface status changes, and routing updates.
    -   **Data Content:** Captures the full spectrum of Cisco system messages including severity levels (0-7), facility codes, and mnemonics (e.g., `LINK-3-UPDOWN`).
    -   **Security Events:** Captures authentication events, firewall/ACL hits (if logged to system log), and configuration changes.

-   **Cisco IOS logs (logs):**
    -   **Description:** Collect Cisco IOS logs from file.
    -   **Logfile Input:** Collects Cisco IOS logs from a local file. This is used when logs are written to a disk or a management server before ingestion.
    -   **Data Content:** Monitors local files specified in the **Paths** configuration variable, such as `/var/log/cisco-ios.log`, providing the same level of detail as the network-based inputs.

## Compatibility

This integration is designed for **Cisco IOS** and **Cisco IOS XE** network devices. Compatibility is generally based on the standard Cisco syslog format rather than a specific version of the operating system. It has been referenced for use with platforms such as **Catalyst 9300 Switches** running **IOS XE 17.x** and various **Integrated Services Routers (ISR)**.

The integration requires Elastic Stack version **8.11.0** or higher.

## Scaling and Performance

To ensure optimal performance in high-volume environments, consider the following:

-   **Transport/Collection Considerations:** Users can choose between **UDP**, **TCP**, and **Logfile** for log delivery. While UDP is faster for high-volume syslog transmission due to lower overhead, TCP is recommended for environments where delivery guarantees are required to prevent log loss during network congestion. If using the **logfile** input, ensure the disk I/O on the host is sufficient to handle the write/read throughput.
-   **Data Volume Management:** Configure the Cisco device to forward only necessary events using the `logging trap` command. Recommend setting this to `informational` (level 6) or lower. Avoid forwarding `debugging` (level 7) logs in production environments, as they can significantly overwhelm the ingest pipeline and increase CPU load on both the source device and the Elastic Agent.
-   **Elastic Agent Scaling:** For high-throughput environments with thousands of network devices, deploy multiple Elastic Agents behind a network load balancer to distribute traffic evenly across the configured **syslog_port**. Place Agents close to the data source to minimize latency and ensure the Agent host has sufficient resources (CPU/RAM) to handle the concurrent parsing of incoming streams.

# Set Up Instructions

## Vendor prerequisites

1. **Administrative Access:** You must have `enable` level access (privilege level 15) to the Cisco IOS Command-Line Interface (CLI) to configure logging.
2. **Network Connectivity:** The Cisco device must have a clear network path to the Elastic Agent on the configured port (default `9002`) and protocol (TCP or UDP).
3. **Timestamp Configuration:** Detailed timestamps must be enabled on the device for logs to be parsed correctly by the Elastic integration using `service timestamps log datetime msec`.
4. **Logging Enabled:** The internal logging process must be active on the Cisco device using the `logging on` command.
5. **Feature Support:** Ensure the device supports the `logging host` command with the required transport protocols (TCP/UDP).

## Elastic prerequisites

-   **Elastic Stack Version:** Ensure your Kibana version is at least **8.11.0**.
-   **Elastic Agent:** An active Elastic Agent must be installed and enrolled in Fleet.
-   **Integration Policy:** The Cisco IOS integration must be added to the Agent policy associated with the Agent receiving the data.
-   **Network Reachability:** The host running the Elastic Agent must be reachable by the Cisco device's source IP and must be listening on the specified port (**syslog_port** `9002` by default).

## Vendor set up steps

### For Syslog (TCP/UDP) Collection:

1.  **Enter Global Configuration Mode:**
    Access the device CLI via SSH or Console and enter configuration mode:
    ```bash
    enable
    configure terminal
    ```

2.  **Enable Detailed Timestamps:**
    Cisco IOS does not enable datetime-based timestamps by default. You must enable them for the integration to parse events:
    ```bash
    service timestamps log datetime msec show-timezone
    ```

3.  **Enable Sequence Numbers:**
    This allows the integration to populate the `event.sequence` field correctly, helping to identify missing logs:
    ```bash
    service sequence-numbers
    ```

4.  **Configure the Remote Logging Host:**
    Specify the Elastic Agent IP and the chosen protocol/port. Replace `<agent-ip>` and `<port>` with your specific values:
    ```bash
    logging host <agent-ip> transport tcp port 9002
    ```
    *(Alternatively, use `transport udp` if that protocol was selected in the integration settings.)*

5.  **Set Severity Level:**
    Define the amount of detail to send. Level 6 (informational) is standard for most environments:
    ```bash
    logging trap 6
    ```

6.  **Specify Source Interface:**
    Ensures logs are sent from a consistent IP address (e.g., Loopback0) to maintain consistent `observer.ip` mapping:
    ```bash
    logging source-interface Loopback0
    ```

7.  **Apply and Save:**
    Finalize the configuration and save to non-volatile memory:
    ```bash
    end
    write memory
    ```

### For Logfile Collection:

1.  **Direct Logs to Local File:** Ensure the Cisco device (or a syslog relay) is writing logs to a local disk or a mounted filesystem accessible by the Elastic Agent.
2.  **Verify File Permissions:** The user running the Elastic Agent service must have read permissions for the log file (e.g., `/var/log/cisco-ios.log`).
3.  **Configure Rotation:** Set up log rotation (e.g., via `logrotate`) to prevent the file from consuming all disk space. The Elastic Agent will automatically track rotated files if configured with wildcards or appropriate path settings.

## Kibana set up steps

### Collecting logs from Cisco IOS via UDP
1. In Kibana, navigate to **Integrations** > **Cisco IOS**.
2. Click **Add Cisco IOS**.
3. Select the **Collecting logs from Cisco IOS via UDP** input type.
4. Configure the following variables:
   - **Host to listen on** (`syslog_host`): The interface address the agent should bind to. Default: `localhost`.
   - **Syslog Port** (`syslog_port`): The port number the agent listens on for Cisco logs. Default: `9002`.
   - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
5. Save and deploy the integration.

### Collecting logs from Cisco IOS via TCP
1. In Kibana, navigate to **Integrations** > **Cisco IOS**.
2. Click **Add Cisco IOS**.
3. Select the **Collecting logs from Cisco IOS via TCP** input type.
4. Configure the following variables:
   - **Host to listen on** (`syslog_host`): The interface address the agent should bind to. Default: `localhost`.
   - **Syslog Port** (`syslog_port`): The port number the agent listens on for Cisco logs. Default: `9002`.
   - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
5. Save and deploy the integration.

### Collecting logs from Cisco IOS via file
1. In Kibana, navigate to **Integrations** > **Cisco IOS**.
2. Click **Add Cisco IOS**.
3. Select the **Collecting logs from Cisco IOS via file** input type.
4. Configure the following variables:
   - **Paths** (`paths`): The list of absolute paths to the log files. Default: `['/var/log/cisco-ios.log']`.
   - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
5. Ensure the Agent has the necessary OS-level permissions to read the specified paths.
6. Save and deploy the integration.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on Cisco IOS:
- **Generate configuration event:** Enter and exit global configuration mode by typing `configure terminal` followed by `exit`.
- **Trigger interface event:** Administratively shut down and then re-enable a non-critical interface (e.g., `interface Loopback99`, `shutdown`, `no shutdown`).
- **Generate authentication event:** Log out of the current SSH session and log back in to trigger a login success event.
- **Verify device status:** Run the command `show logging` on the Cisco device CLI to confirm the "Logging to host" counter is incrementing for the configured agent IP.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "cisco_ios.log"`
4. Verify logs appear. Expand a log entry and confirm these fields:
   - `event.dataset` (should match `cisco_ios.log`)
   - `source.ip` (the IP of the Cisco device)
   - `event.sequence` (the sequence number from the Cisco device)
   - `event.outcome` (the result of the event)
   - `message` (the raw log payload)
5. Navigate to **Analytics > Dashboards** and search for "Cisco IOS" to view the pre-built overview dashboard.

# Troubleshooting

## Common Configuration Issues
- **Missing Timestamps**: If logs are not parsing correctly, ensure `service timestamps log datetime` is configured on the device. Without this, the integration cannot determine the event time.
- **Port Conflicts**: If the Agent fails to start the syslog listener, check if another process is already using port `9002`. Use `netstat -ano | grep 9002` to verify.
- **Relay Header Prefixing**: If logs are sent through a syslog relay, extra headers may be added. This causes parsing failures. Use a Beats processor to strip the extra text before it reaches the Cisco IOS ingest pipeline.
- **Firewall Blocking**: Ensure the Cisco device's source IP is allowed to communicate with the Elastic Agent on the specified port. Use `tcpdump` or `Wireshark` on the Agent host to verify packets are arriving.

## Ingestion Errors

- **Timestamp Parsing Failure**: If logs appear in Kibana but the timestamp is incorrect or the `error.message` field indicates a parsing issue, ensure `service timestamps log datetime` is configured on the device. Without this, Cisco logs may use a format the integration does not recognize.
- **Relay Header Interference**: If you are using a syslog relay (like rsyslog), it might prepend its own headers. This integration expects raw Cisco format. Use a Beats processor to strip additional headers before the data reaches the Cisco IOS pipeline.
- **Timezone Mismatch**: If logs are appearing with the wrong time in Kibana, check the `Timezone` and `Timezone Map` settings in the integration configuration to ensure they match the device's clock settings.

## Vendor Resources

- [Cisco Syslog Message Logging Documentation](https://www.cisco.com/c/en/us/td/docs/routers/access/wireless/software/guide/SysMsgLogging.html)
- [Cisco Timestamp Configuration Guide](https://www.cisco.com/c/en/us/td/docs/routers/access/wireless/software/guide/SysMsgLogging.html#wp1054710)
- [System Management Configuration Guide, Cisco IOS XE 17.17.x (Catalyst 9300 Switches) - Configuring System Message Logs](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9300/software/release/17-17/configuration_guide/sys_mgmt/b_1717_sys_mgmt_9300_cg/configuring_system_message_logs.html)
- [System Management Configuration Guide, Cisco IOS XE 17.x - Reliable Delivery and Filtering for Syslog](https://www.cisco.com/c/en/us/td/docs/routers/ios/config/17-x/syst-mgmt/b-system-management/m_reliable-del-filter-0.html)

# Documentation sites

- Cisco Developer Documentation
- [Cisco Syslog Message Logging Guide](https://www.cisco.com/c/en/us/td/docs/routers/access/wireless/software/guide/SysMsgLogging.html)
- [Cisco IOS XE System Management Guide](https://www.cisco.com/c/en/us/td/docs/routers/ios/config/17-x/syst-mgmt/b-system-management/m_reliable-del-filter-0.html)
