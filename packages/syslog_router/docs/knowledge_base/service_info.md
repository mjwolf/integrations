# Service Info

## Common use cases

- **Infrastructure Monitoring:** Detect and alert on network health issues such as interface flaps, hardware failures, or routing protocol changes.
- **Security Auditing:** Monitor administrative logins, unauthorized configuration attempts, and security-related events across the network fabric.
- **Traffic Analysis:** Investigate network performance bottlenecks by centralizing logs from various routers and switches for correlation.

## Data types collected

- **Logs:** System messages, configuration change alerts, and operational events in standard Syslog format (RFC 3164 or RFC 5424).

## Compatibility

- **Cisco Devices:** Compatible with Cisco IOS, IOS-XE, and IOS-XR versions.
- **Generic Routers:** Supports any network device capable of forwarding messages via UDP or TCP Syslog protocols.

## Scaling and Performance

- **Transport Protocols:** UDP is recommended for high-volume environments to minimize router overhead, though TCP is preferred if delivery guarantees are required.
- **Throughput:** Performance is largely determined by the router's logging severity levels; setting levels to `debugging (7)` can significantly increase log volume and impact router CPU performance.

# Set Up Instructions

## Vendor prerequisites

- Administrative access to the router CLI (Console, SSH, or Telnet).
- Network connectivity between the router's management or source interface and the Elastic Agent.
- Correct Network Time Protocol (NTP) configuration to ensure event timestamp accuracy.

## Elastic prerequisites

- Elastic Agent must be installed and enrolled via Fleet.
- An Agent Policy must be configured with the **Syslog** or **Syslog Router** integration.
- The host firewall must permit inbound traffic on the designated Syslog port (default is UDP 514).

## Vendor set up steps

1.  **Enter Configuration Mode:** Access the global configuration terminal.
    ```shell
    enable
    configure terminal
    ```
2.  **Enable Timestamps:** Configure high-resolution timestamps for log messages.
    ```shell
    service timestamps log datetime msec localtime show-timezone
    ```
3.  **Configure Logging Destination:** Set the IP address of the Elastic Agent host.
    ```shell
    # For standard UDP 514
    logging host <Elastic-Agent-IP>
    # For custom TCP 9002
    logging host <Elastic-Agent-IP> transport tcp port 9002
    ```
4.  **Set Severity Level:** Define the minimum severity of messages to be forwarded.
    ```shell
    logging trap informational
    ```
5.  **Define Source Interface:** (Optional) Set a stable source IP for the logs, such as a Loopback interface.
    ```shell
    logging source-interface Loopback0
    ```
6.  **Activate and Save:** Enable logging and save the running configuration.
    ```shell
    logging on
    end
    write memory
    ```

## Kibana set up steps

1. In Kibana, go to **Management > Integrations**.
2. Search for the **Syslog** integration and click **Add Syslog**.
3. Configure the **Listen Port** and **Listen Address** (e.g., `0.0.0.0`) to match your router settings.
4. Select the appropriate **Agent Policy** and click **Save and continue**.

# Validation Steps

1. **Check Router Status:** Execute `show logging` on the router to verify that the logging host is active and sending packets.
2. **Generate Test Event:** Enter and exit configuration mode on the router to trigger a `SYS-5-CONFIG_I` message.
3. **Verify in Kibana:** Open **Discover**, select the `logs-syslog.*` data stream, and verify that events are appearing with the router's source IP.

# Troubleshooting

## Common Configuration Issues

- **Connectivity Blocked:** Ensure that intermediate firewalls or Router ACLs are not dropping traffic on the configured Syslog port.
- **Timestamp Mismatches:** If logs appear with incorrect times in Kibana, verify that both the router and the Elastic Agent host are synced to the same NTP source.
- **Port Conflict:** Ensure no other process on the Elastic Agent host is already bound to the Syslog port (use `netstat -ano` or `ss -tulpn`).

## Ingestion Errors

- **Parsing Errors:** If logs contain an `error.message` field, the router may be sending a non-standard header. Use the **Syslog Router** integration to define custom patterns for non-RFC compliant logs.
- **Empty Fields:** If `host.hostname` is missing, ensure the router has a configured `hostname` and that the Elastic Agent is correctly parsing the Syslog header.

## API Authentication Errors

- **Not specified:** This integration uses direct network protocols (UDP/TCP) rather than API-based ingestion.

## Vendor Resources

- [Cisco: Configuring System Message Logging](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9500/software/release/17-17/configuration_guide/sys_mgmt/b_1717_sys_mgmt_9500_cg/configuring_system_message_logs.html)
- [Cisco Support Community](https://community.cisco.com/)

# Documentation sites

- [Elastic Syslog Router Integration Guide](https://www.elastic.co/docs/reference/integrations/syslog_router)
- [IP Cisco: Syslog Configuration on Cisco Devices](https://ipcisco.com/lesson/syslog-configuration-cisco/)
- [Auvik: How to Configure Syslog for Cisco](https://www.auvik.com/franklyit/blog/configure-syslog-cisco/)