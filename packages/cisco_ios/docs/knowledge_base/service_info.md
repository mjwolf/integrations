# Service Info

## Common use cases

*   **Network Health Monitoring:** Monitor real-time status changes of physical interfaces and routing protocol neighbor adjacencies (e.g., OSPF/BGP flaps) to ensure network availability.
*   **Security Auditing:** Track administrative activities by logging configuration changes, login attempts (successful and failed), and command execution across the device.
*   **Performance Troubleshooting:** Identify hardware-related issues such as high CPU utilization, memory exhaustion, or environment alarms (temperature/fan) through system event notifications.

## Data types collected

This integration primarily collects **logs** in the form of Syslog messages. These messages include critical metadata such as timestamps, facility codes, severity levels, and specific mnemonics (event identifiers) along with the descriptive log message.

## Compatibility

This integration is compatible with a wide range of Cisco hardware running **Cisco IOS** and **Cisco IOS-XE**. It has been specifically tested and documented for Cisco IOS-XE version 17.x and older legacy IOS branches.

## Scaling and Performance

Performance is largely dependent on the configured `logging trap` severity level. Setting the level to `debugging` (level 7) on high-traffic backbone routers can impact the device's CPU and generate significant log volume; for production environments, `informational` (level 6) or lower is recommended to maintain optimal performance.

# Set Up Instructions

## Vendor prerequisites

*   **CLI Access:** Administrative access to the Cisco IOS device via Console, SSH, or Telnet.
*   **Permissions:** Privileged EXEC mode access (`enable` password) is required to enter global configuration mode.
*   **Network Connectivity:** The device must have IP connectivity to the Elastic Agent host on the configured UDP/TCP port.

## Elastic prerequisites

*   **Elastic Agent:** Must be installed and enrolled in a Fleet policy.
*   **Integration Policy:** The "Cisco IOS" integration must be added to the agent's policy.
*   **Listener Configuration:** Ensure the Elastic Agent is configured with a UDP or TCP input that matches the port and host specified in the Cisco configuration.

## Vendor set up steps

1.  **Enter Configuration Mode:** Connect to the CLI and enter global configuration.
    ```shell
    enable
    configure terminal
    ```
2.  **Enable Accurate Timestamps:** Configure high-precision timestamps including milliseconds and timezone information for accurate correlation.
    ```shell
    service timestamps log datetime msec show-timezone
    ```
3.  **Configure Syslog Destination:** Specify the Elastic Agent IP and the transport settings. Replace `<elastic_agent_ip>` and `<port>` with your actual values.
    ```shell
    logging host <elastic_agent_ip> transport tcp port 9002
    ```
4.  **Set Logging Severity:** Define the minimum level of logs to be forwarded.
    ```shell
    logging trap informational
    ```
5.  **Set Source Interface (Optional):** Use a loopback interface for a consistent source IP address across the network.
    ```shell
    logging source-interface Loopback0
    ```
6.  **Activate Logging:** Ensure the logging process is globally enabled and save the configuration.
    ```shell
    logging on
    end
    write memory
    ```

## Kibana set up steps

1.  In Kibana, navigate to **Management** > **Integrations**.
2.  Search for and select **Cisco IOS**.
3.  Click **Add Cisco IOS**.
4.  In the configuration settings, enter the **Syslog Host** (usually `0.0.0.0`) and the **Syslog Port** (e.g., `9002`) to match your Cisco CLI settings.
5.  Select the **Protocol** (UDP or TCP) used in the Cisco device configuration.
6.  Click **Save and continue** to deploy the policy to your Elastic Agents.

# Validation Steps

1.  **Generate a Test Event:** Enter and exit configuration mode on the Cisco device; this typically generates a "Configured from console" log message.
    ```shell
    configure terminal
    exit
    ```
2.  **Verify Device Output:** Run the command `show logging` on the Cisco device to confirm the syslog host is active and messages are being sent.
3.  **Check Kibana Discover:** Navigate to **Logs Explorer** or **Discover** in Kibana and filter by `event.dataset: cisco_ios.log` to see incoming logs.
4.  **Dashboard Check:** Open the [Cisco IOS] Overview dashboard to verify that data is populating the visualizations.

# Troubleshooting

## Common Configuration Issues

*   **Network Blockage:** Ensure that any firewalls or Access Control Lists (ACLs) between the Cisco device and the Elastic Agent permit traffic on the selected port (e.g., UDP 514 or TCP 9002).
*   **Timestamp Mismatch:** If logs appear with incorrect times, ensure `service timestamps log datetime` is enabled on the Cisco device to provide the full date/time instead of simplified "uptime" counters.
*   **Interface Flapping:** If the source IP in Elastic keeps changing, ensure the `logging source-interface` is set to a stable interface like a Loopback.

## Ingestion Errors

*   **Parsing Failures:** If messages contain `_grokparsefailure`, verify that the Cisco device is not using a custom `logging message-counter` or unconventional logging formats that deviate from standard RFC formats.
*   **Truncated Messages:** For very large log messages, ensure the MTU size on the network path supports the packet size, or switch from UDP to TCP for reliable delivery of fragmented packets.

## API Authentication Errors

*   **CLI Access Denied:** Authentication failures generally occur at the Cisco CLI level; ensure the user has sufficient privilege levels (usually level 15) to execute logging commands.
*   **Agent Connection:** Not specified for this integration as it uses standard Syslog push mechanisms rather than API pulling.

## Vendor Resources

*   [Cisco System Management Configuration Guide](https://www.cisco.com/c/en/us/td/docs/routers/ios/config/17-x/syst-mgmt/b-system-management/m_reliable-del-filter-0.html)
*   [Auvik Guide: How to Configure Syslog on Cisco IOS](https://www.auvik.com/franklyit/blog/configure-syslog-cisco/)
*   [Cisco Community - Troubleshooting Syslog](https://community.cisco.com/)

# Documentation sites

*   [Elastic Cisco IOS Integration Documentation](https://www.elastic.co/docs/reference/integrations/cisco_ios)
*   [Cisco IOS XE System Messages Reference](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sys-msg/configuration/15-mt/sys-msg-15-mt-book.html)
*   [Official Cisco Documentation Portal](https://www.cisco.com/c/en/us/support/index.html)