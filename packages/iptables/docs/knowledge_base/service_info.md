# Service Info

## Common use cases

*   **Security Auditing:** Monitor and log all dropped incoming packets to identify potential scanning activities or unauthorized access attempts from specific IP ranges.
*   **Compliance Logging:** Maintain a verifiable record of network traffic patterns and policy enforcement for regulatory requirements (e.g., PCI-DSS, HIPAA).
*   **Network Troubleshooting:** Debug connectivity issues by logging accepted or rejected packets on specific ports to verify that firewall rules are behaving as expected.

## Data types collected

This integration primarily collects **Logs**. These logs contain network packet metadata extracted by the Linux kernel, including source/destination IP addresses, MAC addresses, protocols, ports, and packet flags.

## Compatibility

This integration is compatible with most Linux distributions using `iptables` (v1.4.x and higher) and `rsyslog` (v8.x and higher). It has been commonly tested on Ubuntu, Debian, RHEL, and CentOS systems.

## Scaling and Performance

To maintain performance, always use the `-m limit` module in `iptables` rules to prevent the logging mechanism from consuming excessive CPU or disk I/O during a flood of traffic. Using the `& stop` directive in `rsyslog` is recommended to prevent logs from being written to both the local disk and the network, reducing redundant I/O operations.

# Set Up Instructions

## Vendor prerequisites

*   Linux operating system with `iptables` installed and active.
*   Root or `sudo` administrative privileges to modify firewall rules and system configurations.
*   `rsyslog` service installed and running as the primary system logger.

## Elastic prerequisites

*   Elastic Agent must be installed on a reachable host (either locally or on a central log collector).
*   The **Cisco/iptables** or generic **Custom Logs/UDP/TCP Syslog** integration must be added to an Elastic Agent policy.
*   The Elastic Agent must be configured to listen on a specific port (e.g., 514 or 9514) using either TCP or UDP.

## Vendor set up steps

1.  **Create a Logging Rule:** Add a rule to your desired `iptables` chain using the `LOG` target. For example, to log dropped packets:
    `sudo iptables -I INPUT -m limit --limit 5/min -j LOG --log-level 4 --log-prefix "IPTABLES_DROPPED: "`
2.  **Configure Rsyslog Filter:** Create a configuration file at `/etc/rsyslog.d/60-iptables-forward.conf` to intercept these logs.
3.  **Define Forwarding Target:** Add the following logic to the file, replacing placeholders with your Elastic Agent details:
    ```
    if $msg contains 'IPTABLES_DROPPED: ' then @@<ELASTIC_AGENT_IP>:<PORT>
    & stop
    ```
4.  **Restart Service:** Apply the changes by restarting the rsyslog daemon:
    `sudo systemctl restart rsyslog`

## Kibana set up steps

1.  In Kibana, navigate to **Management > Integrations** and search for **iptables**.
2.  Click **Add iptables** and configure the integration name and description.
3.  Under **Log file path**, if not using syslog forwarding, specify the path to your log files (e.g., `/var/log/iptables.log`).
4.  If using syslog forwarding (recommended), ensure the **Syslog host** and **Port** match the values used in your `rsyslog` configuration.
5.  Save the integration and deploy the updated policy to your Elastic Agent.

# Validation Steps

1.  **Generate Test Traffic:** Trigger an `iptables` rule by sending traffic that matches your logging criteria (e.g., try to connect to a blocked port).
2.  **Check Local Logs:** Verify the kernel is generating logs locally by running `dmesg | grep "IPTABLES_DROPPED:"` or checking `/var/log/kern.log`.
3.  **Verify Forwarding:** Use `tcpdump -i any port <PORT>` on the Elastic Agent host to confirm packets are arriving from the source server.
4.  **Confirm in Kibana:** Open **Discover**, select the `logs-*` index pattern, and search for `event.dataset: iptables.log` or the specific log prefix used.

# Troubleshooting

## Common Configuration Issues

*   **Port Blocked:** Ensure that the firewall on the Elastic Agent host allows incoming traffic on the configured syslog port (TCP or UDP).
*   **Incorrect Prefix:** If the `$msg contains` string in `rsyslog` does not exactly match the `--log-prefix` in `iptables`, logs will not be forwarded.
*   **Rsyslog Permissions:** Ensure the `rsyslog` user has the necessary permissions to read kernel logs and send data over the network.

## Ingestion Errors

*   **Parsing Failures:** If logs appear in Kibana but fields are not correctly mapped, verify that the `iptables` log format hasn't been modified by custom kernel patches or non-standard `rsyslog` templates.
*   **Truncated Messages:** For very large packet headers, ensure the `rsyslog` maximum message size is large enough to prevent truncation before sending to the agent.

## API Authentication Errors

Not specified

## Vendor Resources

*   [Netfilter/iptables Project Homepage](https://www.netfilter.org/)
*   [Rsyslog Documentation - Documentation Home](https://www.rsyslog.com/doc/)
*   [How to Log IPTables - Putorius Guide](https://www.putorius.net/how-to-log-iptables-send-messages-to.html)

# Documentation sites

*   [Iptables Man Page](https://linux.die.net/man/8/iptables)
*   [Elastic Integration for iptables](https://docs.elastic.co/en/integrations/iptables)
*   [Netfilter Documentation Reference](https://www.netfilter.org/documentation/)