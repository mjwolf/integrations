# Service Info

The Cisco IOS integration for Elastic provides a comprehensive observability and security monitoring solution for network infrastructure running Cisco IOS and IOS-XE operating systems. By centralizing logs from routers, switches, and wireless controllers, organizations can gain real-time visibility into network traffic patterns, hardware performance, and administrative activities across their entire enterprise network. This integration is designed to ingest, parse, and visualize syslog data, converting raw network events into actionable insights within the Elastic Stack.

## Common use cases

-   **Security Threat Detection:**
    *   **Unauthorized Access:** Monitor for SSH, Telnet, or console login failures and brute-force attempts.
    *   **ACL Monitoring:** Track configuration changes or matches in access control lists (ACLs) to identify denied traffic or potential scanning activities at the network edge.
    *   **AAA Auditing:** Monitor Authentication, Authorization, and Accounting events to ensure only authorized users are making changes.
-   **Operational Health Monitoring:**
    *   **Hardware Status:** Track critical hardware events such as fan failures, power supply issues, and environmental sensor alerts.
    *   **Resource Utilization:** Monitor logs related to high CPU utilization and memory exhaustion to prevent network downtime and service degradation.
-   **Network Troubleshooting:**
    *   **Interface Stability:** Analyze interface "flaps" (rapid up/down transitions) and physical layer errors.
    *   **Routing Protocol Stability:** Monitor OSPF, BGP, and EIGRP state changes to identify routing instability.
    *   **Spanning Tree Events:** Identify STP topology changes that may indicate loops or suboptimal pathing.
-   **Audit and Compliance:**
    *   **Configuration Logging:** Maintain a permanent record of all administrative commands executed in privileged EXEC mode (`archive config logging`).
    *   **Regulatory Alignment:** Use centralized logs to meet requirements for PCI-DSS, HIPAA, or SOX regarding infrastructure change management and accountability.

## Data types collected

This integration collects logs via the `cisco_ios.log` data stream. Data is gathered through standard syslog protocols and parsed into the Elastic Common Schema (ECS).

-   **Log Data (`cisco_ios.log`):**
    *   **Operational Messages:** Messages regarding the state of the device, including boot sequences, system restarts, and environmental monitors.
    *   **Security Events:** AAA logs, including successful logins, failed attempts, and command execution logs.
    *   **Network Events:** Interface status transitions (UP/DOWN), routing adjacency changes, and protocol-specific warnings.
-   **Field Mapping:** Raw syslog messages are parsed into key Elastic Common Schema (ECS) fields for easier analysis:
    *   **`event.action`**: The action performed (e.g., `login`, `configuration-change`).
    *   **`log.level`**: The severity of the event mapped from Cisco severity levels (e.g., `emergency`, `alert`, `critical`, `error`, `warning`, `notice`, `informational`, `debug`).
    *   **`source.ip`**: The IP address of the device generating the log.
    *   **`cisco.ios.facility`**: The Cisco-specific facility name (e.g., `SYS`, `LINK`, `SEC_LOGIN`).
    *   **`cisco.ios.mnemonic`**: The specific device message code used for identification (e.g., `UPDOWN`, `CONFIG_I`).
    *   **`cisco.ios.sequence`**: The sequence number assigned by the Cisco device if sequence numbers are enabled.
    *   **`message`**: The full original syslog message content.

## Compatibility

The **Cisco IOS** integration is compatible with the following versions and requirements:

-   **Cisco IOS:** Versions 12.x and 15.x that support standard remote syslog commands.
-   **Cisco IOS-XE:** All versions (16.x, 17.x) including Catalyst 9000, ISR 4000, and ASR 1000 series.
-   **Cisco IOS-XR:** Basic support via standard syslog formatting (though XR-specific features may require a different integration).
-   **Elastic Stack:** Version **8.0** or later is required for full support of all ECS field mappings, data stream structures, and pre-built dashboards.
-   **Elastic Agent:** Version **8.0** or later managed via Fleet.

## Scaling and Performance

To ensure reliable log delivery and high performance in large-scale environments:

-   **High Availability:** Deploy multiple Elastic Agents behind a network load balancer (e.g., F5 or Nginx). Configure the load balancer to distribute UDP or TCP syslog traffic across multiple agents to prevent any single point of failure.
-   **Throughput Management:** 
    *   A single dedicated Elastic Agent can typically handle several thousand events per second (EPS) depending on hardware.
    *   Monitor the `system.cpu.utilization` and `system.memory.usage` of the agent host to ensure it is not saturated.
-   **Buffer Optimization:** On high-traffic routers, increase the internal logging buffer to prevent log loss during spikes. Use the command `logging buffered <size>` (e.g., `logging buffered 16384`) on the Cisco device.
-   **Transport Protocol:** Use **TCP** for mission-critical logging where delivery guarantees are required, provided the Cisco device and network latency support it. UDP is faster but may drop packets during network congestion.

# Set Up Instructions

## Vendor prerequisites

-   **Administrative Access:** You must have `level 15` (privileged EXEC) access to the Cisco device via SSH, Telnet, or Console.
-   **Network Path:** Ensure that UDP port `514` (standard) or your custom TCP/UDP port is allowed through all firewalls between the Cisco device's source interface and the Elastic Agent host.
-   **NTP Synchronization:** Devices must have synchronized clocks for accurate event correlation. Configure NTP using `ntp server <IP_ADDRESS>`.
-   **Timestamp Configuration:** `service timestamps` must be enabled to provide millisecond precision required for Elastic parsing.
-   **Source Interface:** Identify the specific interface (e.g., `Loopback0` or `Vlan10`) that will be used to send syslog traffic to ensure consistency in the source IP address.

## Elastic prerequisites

-   **Elastic Agent:** An active Elastic Agent (version 8.0 or later recommended) must be installed and enrolled in Fleet.
-   **Integration Policy:** The Cisco IOS integration must be added to an Agent Policy.
-   **Connectivity:** The agent host must be listening on a network interface reachable by the Cisco devices.
-   **Kibana Access:** Users must have the `editor` or `admin` role in Kibana to configure integrations.

## Vendor set up steps

1.  Log in to your Cisco IOS device and enter privileged EXEC mode:
    ```bash
    enable
    ```
2.  Enter global configuration mode:
    ```bash
    configure terminal
    ```
3.  Enable high-precision timestamping. This allows Elastic to correctly parse and order logs:
    ```bash
    service timestamps log datetime msec show-timezone
    ```
4.  Configure the Elastic Agent as the remote syslog destination. Replace `<ELASTIC_AGENT_IP>` with your agent's IP and `<PORT>` with your chosen port:
    ```bash
    logging host <ELASTIC_AGENT_IP> transport udp port <PORT>
    ```
5.  Set the severity level for logs to `informational` (level 6) to capture significant operational data:
    ```bash
    logging trap informational
    ```
6.  (Optional) Specify the source interface for syslog packets to ensure the source IP remains constant:
    ```bash
    logging source-interface <INTERFACE_NAME>
    ```
7.  Set the syslog facility (default is `local7`):
    ```bash
    logging facility local7
    ```
8.  Ensure global logging is enabled:
    ```bash
    logging on
    ```
9.  Save the configuration to non-volatile memory (NVRAM):
    ```bash
    end
    write memory
    ```
10. Verify the logging status and destination:
    ```bash
    show logging
    ```

## Kibana set up steps

1.  In Kibana, navigate to **Management** > **Integrations**.
2.  Search for **Cisco IOS** and click the integration tile.
3.  Click **Add Cisco IOS**.
4.  Configure the following input settings:
    *   **Syslog Host**: Set this to `0.0.0.0` to listen on all available interfaces, or provide the specific IP of the Elastic Agent host.
    *   **Syslog Port**: Enter the port number (e.g., `514` or `9001`) that matches your Cisco `logging host` configuration.
    *   **Protocol**: Select `udp` or `tcp` based on your vendor setup.
    *   **Internal Zones**: (Optional) Specify internal network ranges in CIDR format to help the integration identify internal vs. external traffic.
    *   **External Zones**: (Optional) Specify external network ranges for traffic classification.
5.  Under **Advanced options**, you can configure additional parameters:
    *   **Tags**: Add custom metadata to incoming events (e.g., `location:ny`, `env:prod`).
    *   **Processors**: Add custom script processors or field renamings if necessary.
6.  Review the **Dataset Name**, which defaults to `cisco_ios.log`.
7.  Click **Save and continue**.
8.  Select the **Agent Policy** where you want to deploy this configuration.
9.  Click **Save and deploy changes**. The Elastic Agent will automatically download the configuration and start the syslog listener on the specified port.

# Validation Steps

After completing the setup, verify that data is flowing correctly into the Elastic Stack.

1.  **Generate Test Events on the Cisco Device:**
    *   Trigger an authentication event by logging out and back in via SSH.
    *   Create a configuration log by entering and exiting configuration mode:
        ```bash
        conf t
        interface GigabitEthernet0/0
        description Monitoring_Test
        exit
        exit
        ```
    *   Alternatively, use the `send log` command if your version supports it: `send log 6 "Elastic validation test"`

2.  **Verify Data Ingestion in Kibana Discover:**
    *   Navigate to **Analytics** > **Discover**.
    *   Select the **logs-*** data view.
    *   Enter the following filter in the KQL search bar: 
        `data_stream.dataset : "cisco_ios.log"`
    *   Verify that logs appear with the following fields populated:
        *   `message`: The original syslog string.
        *   `log.level`: Should show levels like `info`, `notice`, or `warning`.
        *   `event.code`: Should contain the Cisco mnemonic (e.g., `CONFIG_I`, `UPDOWN`).
        *   `source.address`: The IP address of your Cisco device.
        *   `event.dataset`: Should be `cisco_ios.log`.

3.  **Inspect the Pre-built Dashboard:**
    *   Navigate to **Analytics** > **Dashboard**.
    *   Search for and open the **[Logs Cisco IOS] Overview** dashboard.
    *   Confirm that the "Log Level" breakdown and "Event Codes" visualizations are populating with real-time data from your network devices.

# Troubleshooting

## Common Configuration Issues

-   **Port Conflicts:** If the Elastic Agent fails to start the listener, ensure no other service (like `rsyslog` or `syslog-ng`) is using the same port. Use `netstat -tunlp | grep <PORT>` on Linux or `Get-NetTCPConnection -LocalPort <PORT>` on Windows to check.
-   **ACL/Firewall Blocks:** If no data reaches Kibana, check the Cisco device's outbound ACLs and intermediate firewalls. Run `show ip access-lists` on the router to see if packets are being dropped on egress.
-   **Incorrect Source IP:** If the device has multiple interfaces, the syslog packet might originate from an IP not allowed by your firewall. Use the `logging source-interface` command on the Cisco device to standardize the source IP.
-   **Timestamp Mismatch:** If logs appear in the past or future, ensure `service timestamps log datetime msec` is configured and NTP is synchronized across the infrastructure.

## Ingestion Errors

-   **Parsing Failures:** If you see `_grokparsefailure` tags in the `tags` field, ensure the Cisco device is using the standard syslog format. Disable non-standard logging options such as `logging simplified-syslog-iso8601` or custom header formats.
-   **Truncated Messages:** Large messages (e.g., long ACL logs) may be cut off over UDP. Switch both the Cisco device and the Kibana integration configuration to **TCP** to handle larger payloads reliably.
-   **Log Level Filtering:** If expected logs (like config changes) aren't appearing, check the `logging trap` level on the Cisco device. Ensure it is set to `informational (6)` or `debug (7)`.

## Vendor Resources

-   [Configuring System Message Logs - Cisco Catalyst 9300 Series (IOS XE)](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9300/software/release/17-17/configuration_guide/sys_mgmt/b_1717_sys_mgmt_9300_cg/configuring_system_message_logs.html)
-   [Cisco IOS Network Management Configuration Guide: Troubleshooting and Fault Management](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/bsm/configuration/15-2mt/bsm-troubleshooting.html)

# Documentation sites

-   [Cisco IOS XE 17.x System Management Configuration Guide](https://www.cisco.com/c/en/us/td/docs/routers/ios/config/17-x/syst-mgmt/b-system-management/m_reliable-del-filter-0.html)
-   [Cisco IOS 15.x Network Management Guide - Logging Configuration](https://www.cisco.com/c/en/us/td/docs/ios/netmgmt/configuration/guide/15_0s/nm_15_0s_book/nm_troubleshooting.html)
-   [Cisco Community: How to Configure Logging in Cisco IOS](https://community.cisco.com/t5/networking-knowledge-base/how-to-configure-logging-in-cisco-ios/ta-p/3132434)