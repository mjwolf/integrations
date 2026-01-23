# Service Info

## Common use cases

The Cisco Adaptive Security Appliance (ASA) integration provides a robust solution for centralizing network security data within the Elastic Stack. This integration allows administrators and security analysts to gain deep visibility into their network traffic, security posture, and system health by ingesting and parsing complex ASA syslog data.

*   **Security Incident Monitoring:** Detect and investigate potential threats by analyzing access control list (ACL) hits, denied connection attempts, and unusual traffic patterns across firewall interfaces.
*   **VPN and Remote Access Auditing:** Track user connectivity for AnyConnect or site-to-site VPNs, including login/logout timestamps, assigned IP addresses, and session duration for compliance and troubleshooting.
*   **Network Performance Troubleshooting:** Identify connectivity bottlenecks or routing issues by monitoring system logs related to interface status changes, failover events, and resource exhaustion.
*   **Administrative Auditing:** Monitor configuration changes and command-line activity to maintain an audit trail of administrative actions performed on the ASA hardware or virtual appliance.

## Data types collected
This integration collects various log types through specialized data streams. Each stream is designed to handle different ingest methods from your Cisco ASA environment:

- **Cisco ASA logs (UDP/TCP):**
  - **Data Stream:** `log`
  - **Description:** Collect Cisco ASA logs. This includes connection events, system messages, and security events sent over the network via syslog.
- **Cisco ASA logs (File):**
  - **Data Stream:** `log`
  - **Description:** Collect Cisco ASA logs from file. This is used when logs are written to a disk by an intermediate syslog daemon or local buffer export.

Collected data typically includes:
- **Firewall Logs:** Connection setup and teardown events, denied traffic logs, and NAT translation details.
- **Authentication and VPN Logs:** User login/logout events, AnyConnect VPN session details, and AAA (Authentication, Authorization, and Accounting) messages.
- **System and Diagnostic Logs:** Hardware status, software process events, and configuration change notifications.

## Compatibility

*   **Cisco ASA:** Compatible with Cisco Adaptive Security Appliances supporting standard syslog output. Verified for **Cisco ASA version 8.4 and later**, including physical and virtual (ASAv) deployments.
*   **Kibana:** This integration requires Kibana version **^8.11.0** or **^9.0.0**.

## Scaling and Performance

To ensure optimal performance in high-volume environments, consider the following:

*   **Transport/Collection Considerations:** Users can choose between UDP and TCP for log delivery. UDP (default port `9001`) offers lower overhead and prevents the ASA from experiencing backpressure, though it lacks delivery guarantees. TCP provides reliable delivery but requires careful configuration (using `logging permit-hostdown`) to ensure the ASA does not block production traffic if the Elastic Agent becomes unreachable.
*   **Data Volume Management:** Configure the ASA logging level to `informational` (level 6) or lower to manage ingest volume. Avoid using the `debugging` (level 7) level in production environments as it can generate excessive data volume. Specific high-volume messages can be disabled using the `no logging message <ID>` command on the ASA to reduce noise.
*   **Elastic Agent Scaling:** For high-throughput environments processing thousands of events per second (EPS), deploy the Elastic Agent on dedicated hardware. In large-scale architectures, consider deploying multiple Elastic Agents behind a network load balancer to distribute the syslog ingestion load and ensure high availability.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have `enable` level or administrative privileges to the Cisco ASA Command Line Interface (CLI).
- **Network Reachability:** Ensure the ASA device can reach the Elastic Agent host on the configured port (default **9001**).
- **Interface Selection:** Identify the ASA interface (e.g., `inside`, `management`) used for egress syslog traffic.
- **System Clock:** Synchronize the ASA system clock via NTP to ensure log timestamps align with the Elastic Stack.
- **Resource Availability:** Verify sufficient CPU headroom before increasing logging severity levels.

## Elastic prerequisites
1. **Elastic Agent:** An active Elastic Agent must be installed and enrolled in Fleet on a host reachable by the ASA. The stack version must be `^8.11.0` or `^9.0.0`.
2. **Connectivity:** Firewalls between the ASA and the Elastic Agent must allow traffic on the chosen syslog port (**9001** by default).
3. **Permissions:** The user configuring the integration in Kibana must have the `all` privilege for the `Integrations` and `Fleet` features.

## Vendor set up steps

### For Syslog (UDP/TCP) Collection:

1.  **Enter Configuration Mode:**
    Connect to your ASA CLI:
    ```shell
    enable
    configure terminal
    ```

2.  **Enable Logging:**
    Activate the logging process:
    ```shell
    logging enable
    ```

3.  **Define the Syslog Destination:**
    Replace `<interface_name>` with your interface (e.g., `inside`) and `<agent_ip>` with the Agent address.
    
    **For UDP:**
    ```shell
    logging host <interface_name> <agent_ip> udp/9001
    ```
    
    **For TCP:**
    ```shell
    logging host <interface_name> <agent_ip> tcp/9001
    ```

4.  **Set Severity Level:**
    Set the trap level (Level 6 is recommended):
    ```shell
    logging trap informational
    ```

5.  **Configure Facility and Timestamps:**
    ```shell
    logging facility 20
    logging timestamp
    ```

6.  **(Recommended for TCP) Prevent Traffic Drops:**
    ```shell
    logging permit-hostdown
    ```

7.  **Save and Verify:**
    ```shell
    exit
    write memory
    show logging
    ```

### For Logfile Collection:

1.  **Configure Local Logging:**
    ```shell
    logging buffered informational
    ```
2.  **External Management:**
    Ensure the log file on the host (e.g., `/var/log/cisco-asa.log`) has read permissions for the Elastic Agent.

## Kibana set up steps

### Collecting logs from Cisco ASA via UDP
1. In Kibana, navigate to **Integrations** > **Cisco ASA**.
2. Click **Add Cisco ASA**.
3. Under the **Cisco ASA logs** section, locate the **Collecting logs from Cisco ASA via UDP** input.
4. Configure the following variables:
   - **Listen Address** (`udp_host`): The bind address to listen for UDP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
   - **Listen Port** (`udp_port`): The UDP port number to listen on. Default: `9001`.
   - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
   - **Preserve searchable message text.** (`keep_message`): Preserves the log message in a searchable field, `cisco.asa.full_message`. Default: `False`.
5. Click **Save and continue**.

### Collecting logs from Cisco ASA via TCP
1. In Kibana, navigate to **Integrations** > **Cisco ASA**.
2. Click **Add Cisco ASA**.
3. Under the **Cisco ASA logs** section, locate the **Collecting logs from Cisco ASA via TCP** input.
4. Configure the following variables:
   - **Listen Address** (`tcp_host`): The bind address to listen for TCP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
   - **Listen Port** (`tcp_port`): The TCP port number to listen on. Default: `9001`.
   - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
   - **Preserve searchable message text.** (`keep_message`): Preserves the log message in a searchable field, `cisco.asa.full_message`. Default: `False`.
5. Click **Save and continue**.

### Collecting logs from Cisco ASA via file
1. In Kibana, navigate to **Integrations** > **Cisco ASA**.
2. Click **Add Cisco ASA**.
3. Under the **Cisco ASA logs** section, locate the **Collecting logs from Cisco ASA via file** input.
4. Configure the following variables:
   - **Paths** (`paths`): List of paths to the log files. Default: `['/var/log/cisco-asa.log']`.
   - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
   - **Preserve searchable message text.** (`keep_message`): Preserves the log message in a searchable field, `cisco.asa.full_message`. Default: `False`.
5. Click **Save and continue**.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on Cisco ASA:
- **Configuration change:** Enter and exit config mode to generate an audit log: `configure terminal` then `exit`.
- **Interface event:** Toggle a test interface status: `shutdown` then `no shutdown`.
- **Authentication event:** Log out and log back into the CLI or ASDM.
- **Test Syslog:** If supported, use `send log 6 testmessage` to manually trigger a log.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "cisco_asa.log"`
4. Verify logs appear. Expand a log entry and confirm these fields:
   - `event.dataset` (should match `cisco_asa.log`)
   - `source.ip` and/or `destination.ip`
   - `event.action` or `event.outcome`
   - `message` (the raw log payload)
5. Navigate to **Analytics > Dashboards** and search for "Cisco ASA" to view visualizations.

# Troubleshooting

## Common Configuration Issues

- **Inbound Port Blocked**: Ensure the host OS firewall (iptables/firewalld) allows traffic on the specified port (default `9001`).
- **Interface Reachability**: Use `ping <interface_name> <agent_ip>` from the ASA to verify the routing path to the Agent.
- **Incorrect Severity Level**: Ensure `logging trap` is set to `informational` (6). If set to `errors` (3), most events will not be sent.
- **TCP Blocking Mode**: If using TCP, ensure `logging permit-hostdown` is configured to prevent traffic disruption if the Agent is unreachable.

## Ingestion Errors

- **Timestamp Parsing**: Ensure `logging timestamp` is enabled on the ASA CLI.
- **Missing Device ID**: Enable `logging device-id hostname` to help distinguish between multiple ASA units.
- **Truncated Logs**: Check for MTU mismatches or syslog maximum length settings if logs are incomplete.
- **Field Mapping**: Inspect `error.message` in Discover to see if non-standard syslog headers are causing grok failures.

## Vendor Resources

- [Cisco: Configure Adaptive Security Appliance (ASA) Syslog](https://www.cisco.com/c/en/us/support/docs/security/pix-500-series-security-appliances/63884-config-asa-00.html)
- [CLI Book 1: Cisco Secure Firewall ASA Series General Operations CLI Configuration Guide - Monitor Syslog](https://www.cisco.com/c/en/us/td/docs/security/asa/asa923/configuration/general/asa-923-general-config/monitor-syslog.html)

## Documentation sites
- [Configure Adaptive Security Appliance (ASA) Syslog - Cisco](https://www.cisco.com/c/en/us/support/docs/security/pix-500-series-security-appliances/63884-config-asa-00.html)
- [CLI Book 1: Cisco ASA Series General Operations CLI Configuration Guide, 9.16 - Logging](https://www.cisco.com/c/en/us/td/docs/security/asa/asa916/configuration/general/asa-916-general-config/monitor-syslog.html)
