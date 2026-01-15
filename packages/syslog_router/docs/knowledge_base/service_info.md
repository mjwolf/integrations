# Service Info

## Common use cases

The Syslog Router integration is designed to ingest, parse, and visualize system log messages from network routing hardware, providing centralized visibility into network health and security. This integration allows administrators to monitor distributed infrastructure from a single pane of glass.
- **Security Auditing and Access Monitoring:** Track user logins, logout events, and failed authentication attempts on network hardware to identify potential brute-force attacks or unauthorized access.
- **Network Stability and Performance Troubleshooting:** Monitor interface status changes (up/down events), routing protocol flaps (OSPF, BGP), and hardware alerts such as power supply failures or overheating.
- **Configuration Change Tracking:** Capture events related to configuration mode entry and exit, providing a clear audit trail of who changed what on the network infrastructure and when.
- **Compliance and Long-term Retention:** Automatically archive router logs to the Elastic Stack to meet regulatory requirements for log retention that often exceed the local buffer capacity of physical routers.

## Data types collected

This integration can collect the following types of data:
- **System Event Logs:** General system messages including boot sequences, process restarts, and hardware diagnostic alerts.
- **Authentication and Authorization Logs:** Records of SSH/Telnet sessions, console access, and AAA (Authentication, Authorization, and Accounting) events.
- **Interface and Link State Logs:** Transitions in physical and logical interface states, including bandwidth threshold alerts.
- **Protocol Logs:** Events from routing protocols like BGP, OSPF, and EIGRP, including neighbor adjacency changes.
- **Configuration Audit Logs:** Detailed records of commands entered in configuration mode and global configuration changes.
- **Data Formats:** Supports standard Syslog formats (RFC 3164 and RFC 5424) over UDP or TCP transports.

## Compatibility

This integration is compatible with **Cisco IOS**, **Cisco IOS-XE**, and other networking platforms that support standard Syslog forwarding. It is tested and validated for devices running **Cisco IOS version 12.x and higher** and **Junos OS version 15.1 and higher**. Most industry-standard routers following RFC 3164 formatting are supported.

## Scaling and Performance

- **Transport/Collection Considerations:** This integration supports both UDP and TCP. UDP is recommended for high-performance environments where low overhead is critical, though it does not guarantee delivery. TCP is recommended for environments requiring reliable log delivery (e.g., security auditing) where network congestion might otherwise cause packet loss.
- **Data Volume Management:** To prevent overwhelming the Elastic Agent and the network, use the `logging trap` command on the router to filter logs at the source. It is recommended to set the severity level to `informational` (6) or `notifications` (5) to capture essential operational data while excluding high-volume `debugging` (7) logs unless performing active troubleshooting.
- **Elastic Agent Scaling:** A single Elastic Agent can typically handle thousands of events per second from multiple routers. For high-volume environments with hundreds of edge devices, consider deploying multiple Elastic Agents behind a network load balancer to distribute the syslog traffic and provide high availability for log ingestion.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have privileged EXEC level access (enable mode) to the router's command-line interface (CLI) via SSH, Telnet, or Console.
- **Network Connectivity:** The router must have IP reachability to the Elastic Agent. Any firewalls or Access Control Lists (ACLs) between the router and the Agent must allow traffic on the configured port (e.g., UDP 514 or TCP 1468).
- **NTP Synchronization:** Network Time Protocol (NTP) must be configured on the router to ensure log timestamps are accurate and synchronized with the Elastic Stack for effective correlation.
- **Interface Configuration:** A stable interface, such as a Loopback interface, should be configured to serve as the consistent source IP for all outbound syslog traffic.

## Elastic prerequisites
- **Elastic Agent:** An active Elastic Agent must be installed and enrolled in Fleet.
- **Agent Policy:** The Agent must be assigned a policy that includes the Syslog Router integration.
- **Open Ports:** The host machine running the Elastic Agent must have its local firewall configured to allow inbound traffic on the designated syslog port.

## Vendor set up steps

### For Cisco IOS/XE Syslog Collection:

1.  **Access the CLI:** Log in to your router and enter global configuration mode.
    ```bash
    enable
    configure terminal
    ```
2.  **Enable System Logging:** Ensure the logging process is active on the device.
    ```bash
    logging on
    ```
3.  **Define the Elastic Agent Target:** Configure the destination IP address of your Elastic Agent and specify the transport protocol and port.
    ```bash
    logging host <agent-ip> transport <protocol> port <port>
    ```
    *Example:* `logging host 192.168.1.50 transport udp port 514`
4.  **Configure Log Severity:** Set the logging level to filter which messages are sent to the Agent.
    ```bash
    logging trap informational
    ```
5.  **Set the Source Interface:** (Highly Recommended) Use a loopback interface to ensure the source IP of the logs remains constant even if physical interfaces go down.
    ```bash
    logging source-interface Loopback0
    ```
6.  **Enable Enhanced Timestamps:** Configure the router to include millisecond precision and timezone information in the log messages for better analysis in Kibana.
    ```bash
    service timestamps log datetime show-timezone msec
    ```
7.  **Set Logging Facility:** (Optional) Set the facility to `local7` (default) or another local facility to help categorize these logs in the Elastic Stack.
    ```bash
    logging facility local7
    ```
8.  **Save Configuration:** Exit to privileged EXEC mode and save the settings to the startup configuration.
    ```bash
    end
    copy running-config startup-config
    ```

## Kibana set up steps

1.  **Navigate to Integrations:** In Kibana, go to **Management > Integrations**.
2.  **Find the Integration:** Search for **Syslog Router** and select it.
3.  **Add to Policy:** Click **Add Syslog Router**.
4.  **Configure Input:** Under the **Syslog Router Log** data stream settings, perform the following:
    - Set **Listen Address** to `0.0.0.0` to listen on all interfaces or provide the specific IP of the Agent host.
    - Set **Listen Port** to match the port configured on the router (e.g., `514`).
    - Select the **Protocol** (UDP or TCP) to match the router configuration.
5.  **Advanced Settings:** Ensure the **Internal Encoding** is set to `UTF-8` unless your specific device uses a different character set.
6.  **Save and Deploy:** Click **Save and continue** and then **Add agent integration to policy**. This will automatically deploy the configuration to the Elastic Agents enrolled in that policy.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from the router to the Elastic Stack.

### 1. Trigger Data Flow on Router:
- **Generate Configuration Event:** Enter and exit configuration mode to trigger a `SYS-5-CONFIG_I` event:
  ```shell
  conf t
  exit
  ```
- **Trigger Interface Event:** Toggle a non-critical interface to generate `LINK-3-UPDOWN` and `LINEPROTO-5-UPDOWN` messages:
  ```shell
  interface GigabitEthernet0/1
  shutdown
  no shutdown
  ```
- **Generate Authentication Event:** Log out of your current SSH session and log back in to trigger login success messages.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the following KQL filter: `data_stream.dataset : "syslog_router.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `syslog_router.log`)
   - `source.address` (should reflect the Router's IP)
   - `log.syslog.priority` or `log.level`
   - `message` (containing the raw router log, e.g., `%SYS-5-CONFIG_I: Configured from console by admin`)
   - `event.timezone` (confirming it matches your router's local time settings)
5. Navigate to **Analytics > Dashboards** and search for "Syslog Router" to view pre-built visualizations.

# Troubleshooting

## Common Configuration Issues

- **Clock Skew and Timestamp Mismatches**: If logs appear in the "future" or "past" in Discover, verify that NTP is synchronized on the router and that `service timestamps log datetime` is enabled. Without synchronized clocks, log correlation becomes impossible.
- **Network Path Blocking**: If no logs arrive, check for ACLs on the router itself or intermediate firewalls blocking the configured UDP/TCP port. Use `tcpdump` or `Wireshark` on the Elastic Agent host to verify if packets are reaching the interface.
- **Incorrect Source Interface**: If logs are reaching the agent but are being filtered or categorized incorrectly, ensure `logging source-interface` is set. If not, the router may use the IP of the exit interface, which can change due to routing changes.
- **UDP Packet Loss**: In high-traffic environments, UDP packets may be dropped by the OS buffer. Monitor the Agent host's network statistics for dropped packets and consider switching to TCP or increasing the Agent's socket buffer size.

## Ingestion Errors

- **Parsing Failures**: If logs appear in Kibana with a `_grokparsefailure` tag, check if the router is using a non-standard syslog format. Ensure `service timestamps` is enabled on the router to provide the expected date format.
- **Time Synchronization Issues**: If logs appear in the future or the past in Discover, verify that both the Router and the Elastic Agent host are synchronized to the same time source via NTP.
- **Empty Message Fields**: If the message field is populated but specific fields like `syslog_router.mnemonic` are missing, verify that the log message follows the standard Cisco `%FACILITY-SEVERITY-MNEMONIC: description` format.

## Vendor Resources
- [System Monitoring Configuration Guide for Cisco NCS 540 Series Routers](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5xx/system-monitoring/24xx/b-system-monitoring-cg-25xx-ncs540/implementing-system-logging.html)
- [Configure Syslog Forwarding to External Destinations - Palo Alto Networks](https://docs.paloaltonetworks.com/panorama/10-1/panorama-admin/manage-log-collection/configure-syslog-forwarding-to-external-destinations)

# Documentation sites

- https://networkproguide.com/configure-syslog-cisco-ios-switch-router/
- https://ipcisco.com/lesson/syslog-configuration-cisco/
