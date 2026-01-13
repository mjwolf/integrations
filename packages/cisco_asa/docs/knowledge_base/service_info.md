# Service Info

The Cisco ASA (Adaptive Security Appliance) integration allows you to monitor network security events, traffic patterns, and administrative actions. By ingesting ASA logs into the Elastic Stack, you gain centralized visibility into your network perimeter, making it easier to identify threats, troubleshoot connectivity, and ensure regulatory compliance.

## Common use cases

*   **Security Threat Detection:** Monitor firewall permit and deny logs in real-time to identify potential reconnaissance activity, brute-force attacks, or unauthorized access attempts across the network perimeter.
*   **Compliance and Auditing:** Maintain detailed logs of administrative access (AAA) and VPN session activity (logins, logouts, and durations) to satisfy regulatory requirements such as PCI-DSS, HIPAA, and GDPR.
*   **Network Troubleshooting:** Analyze connection establishment and teardown events to diagnose connectivity issues, asymmetric routing, or application-specific communication failures.
*   **VPN Monitoring:** Track remote access patterns, including user session information, assigned IP addresses, and data volume for AnyConnect and Site-to-Site VPN connections.
*   **Policy Optimization:** Use log data to identify unused firewall rules or shadowed policies by analyzing hit counts and traffic flows.
*   **Incident Response:** Correlate ASA events with other security data sources in Elastic Security to perform root cause analysis during security incidents.

## Data types collected

This integration collects several categories of data through the **`cisco_asa.log`** data stream. The collected data is mapped to the Elastic Common Schema (ECS) to ensure consistency across different data sources.

*   **Firewall Logs:** Standard syslog messages documenting traffic filtering events, including:
    *   **Permit/Deny Actions:** Records of traffic allowed or blocked by Access Control Lists (ACLs).
    *   **Connection Events:** Information on TCP/UDP session establishment and teardown.
*   **Authentication and Authorization (AAA) Logs:** Detailed events for administrative access and VPN user sessions, including:
    *   User login and logout events.
    *   Command execution history for administrative sessions.
*   **VPN Logs:** Comprehensive data for IPsec and SSL (AnyConnect) VPNs:
    *   Tunnel establishment and termination.
    *   Encryption algorithm negotiations.
    *   Assigned virtual IP addresses for remote clients.
*   **System Health and Operational Logs:**
    *   **Resource Utilization:** CPU and memory threshold alerts.
    *   **High Availability:** Failover status changes and health check failures.
    *   **Interface Status:** Link up/down events.

**Key Fields Captured:**
*   `cisco.asa.message_id`: The specific Cisco message identifier (e.g., `%ASA-6-302013`).
*   `source.ip` / `destination.ip`: The network addresses involved in the communication.
*   `source.port` / `destination.port`: The transport layer ports.
*   `event.action`: The action taken (e.g., `built`, `teardown`, `denied`).
*   `network.protocol`: The protocol used (e.g., `tcp`, `udp`, `icmp`).
*   `device.id`: The identifier for the ASA device generating the log.

## Compatibility

*   **Cisco ASA Software:** Versions 9.x and higher are supported.
*   **Cisco ASAv:** Virtual Appliance versions 9.x and higher.
*   **Cisco Firepower:** Devices running in ASA compatibility mode (ASA software image).
*   **Elastic Stack:** Version **8.0** or higher is required.
*   **Elastic Agent:** Version **8.0** or higher is required.

## Scaling and Performance

*   **Protocol Choice:** For high-volume environments (exceeding 10,000 events per second), use the **TCP** protocol. TCP provides reliable delivery and congestion control, whereas UDP may drop packets during traffic spikes.
*   **Resource Allocation:** Ensure the host running the Elastic Agent has sufficient CPU (at least 2-4 dedicated cores) and memory to handle the parsing overhead of high-velocity syslog streams.
*   **Load Balancing:** Deploy a network load balancer (NLB) in front of multiple Elastic Agent instances to distribute the syslog load and provide high availability.
*   **Log Rate Limiting:** If the ASA generates an overwhelming volume of logs, use the ASA's `logging rate-limit` command to throttle less critical message IDs.

# Set Up Instructions

## Vendor prerequisites

1.  **Administrative Access:** Ensure you have **Privileged EXEC mode (Level 15)** access via CLI or administrative rights in Cisco ASDM.
2.  **Network Connectivity:** Confirm the ASA can reach the Elastic Agent's IP address. If an intermediate firewall exists, ensure the configured syslog port (e.g., `514` or `9001`) is open.
3.  **Logging Enabled:** The internal logging processes must be globally active.
4.  **Time Synchronization:** Configure NTP on the Cisco ASA to ensure accurate timestamps, which are critical for log correlation.
5.  **Interface Selection:** Determine the interface (e.g., `management` or `inside`) through which the ASA will send logs to the Agent.

## Elastic prerequisites

*   **Elastic Stack Version:** Use version **8.0** or higher for full support of ECS mapping.
*   **Elastic Agent:** An Elastic Agent must be enrolled in Fleet and running on a host that can receive network traffic from the ASA.
*   **Port Availability:** The host running the Elastic Agent must have the designated syslog port (default `514`) available and not occupied by other services like `rsyslog`.

## Vendor set up steps

### For Configuration via ASDM (UI):

1.  Log in to the Cisco ASDM interface.
2.  Navigate to **Configuration > Device Management > Logging > Logging Setup**.
3.  Check the **Enable logging** checkbox.
4.  Go to **Configuration > Device Management > Logging > Syslog Servers**.
5.  Click **Add** and configure the following:
    *   **Interface**: Choose the interface with a route to the Agent (e.g., `inside`).
    *   **IP Address**: Enter the IP address of the Elastic Agent.
    *   **Protocol**: Select **TCP** (recommended) or **UDP**.
    *   **Port**: Enter the port number (e.g., `514`).
    *   Click **OK**.
6.  Navigate to **Configuration > Device Management > Logging > Logging Filters**.
7.  Select **Syslog Servers** and click **Edit**.
8.  Set the **Filter on severity** to **Informational** (level 6).
9.  Click **OK** and then **Apply**.

### For Configuration via CLI:

1.  Access the ASA CLI and enter configuration mode:
    ```bash
    enable
    configure terminal
    ```
2.  Enable logging and enable timestamps for all messages:
    ```bash
    logging enable
    logging timestamp
    ```
3.  Specify the Elastic Agent as the logging host. Replace `<interface>`, `<agent_ip>`, and `<port>` with your values:
    ```bash
    logging host <interface> <agent_ip> tcp/<port>
    ```
4.  Set the logging severity level to informational:
    ```bash
    logging trap informational
    ```
5.  (Optional) Set the syslog facility (default is 20):
    ```bash
    logging facility 20
    ```
6.  Save the configuration:
    ```bash
    write memory
    ```

## Kibana set up steps

1.  In Kibana, navigate to **Management** > **Integrations**.
2.  Search for **Cisco ASA** and click on the integration tile.
3.  Click **Add Cisco ASA**.
4.  Configure the following integration variables:
    *   **Syslog Host**: Set to `0.0.0.0` to listen on all available network interfaces, or specify the specific IP of the Agent host.
    *   **Syslog Port**: Enter the port number configured on the ASA (e.g., `514`).
    *   **Listen Protocol**: Select `tcp` or `udp` to match your ASA configuration.
    *   **Internal Zones**: Enter a comma-separated list of internal network CIDRs (e.g., `10.0.0.0/8, 172.16.0.0/12`). This helps populate the `network.direction` field.
    *   **External Zones**: Enter a comma-separated list of external network CIDRs (e.g., `0.0.0.0/0`).
    *   **Keep Raw Fields**: Toggle this on if you want to preserve the original, unparsed log message in the `event.original` field.
5.  (Optional) Expand **Advanced options** to add:
    *   **Tags**: Add custom strings (e.g., `datacenter-west`) to tag every incoming event.
    *   **Processors**: Add YAML-based processors for custom filtering or field enrichment at the Agent level.
6.  Click **Save and continue**.
7.  Select the **Agent Policy** where you want to deploy the integration and click **Save and deploy**.

# Validation Steps

## 1. Trigger Data Flow on Cisco ASA:
*   **Generate Traffic:** Initiate a connection (e.g., HTTPS request or Ping) that passes through the ASA to generate "Built" and "Teardown" session logs.
*   **Simulate Admin Activity:** Log out and log back into the ASA CLI or ASDM to generate AAA authentication logs.
*   **Check ASA Stats:** Run the command `show logging` on the ASA to verify that the "Logging Host" is active and messages are being sent.

## 2. Check Data in Kibana:
1.  Navigate to **Analytics > Discover**.
2.  Select the **Logs** data view.
3.  Filter for the Cisco ASA dataset by entering the following in the search bar:
    `data_stream.dataset : "cisco_asa.log"`
4.  Confirm that events are appearing and that the following fields are populated:
    *   `event.dataset`: Should be `cisco_asa.log`.
    *   `source.ip`: The originating IP address.
    *   `cisco.asa.message_id`: Should match Cisco's standard ID format (e.g., `%ASA-6-302013`).
    *   `event.outcome`: Should indicate `success` or `failure`.
5.  Navigate to **Analytics > Dashboards** and search for **[Logs Cisco ASA] Overview** to view pre-built visualizations of your firewall data.

# Troubleshooting

## Common Configuration Issues

*   **Port Conflict**: If the Elastic Agent fails to start the syslog listener, another service (like `rsyslog` or `syslog-ng`) may be using the port. Stop the conflicting service or change the port in both the ASA and Kibana settings.
*   **Network Firewalls**: Ensure that host-level firewalls (e.g., `Windows Firewall`, `iptables`, `ufw`) on the Elastic Agent machine are configured to allow inbound traffic on the configured TCP/UDP port.
*   **Routing**: Verify the ASA can reach the Agent by using the `ping <interface_name> <agent_ip>` command from the ASA CLI.

## Ingestion Errors

*   **Timezone Mismatches**: If logs appear with the wrong timestamp in Kibana, ensure the ASA has `logging timestamp` enabled and that NTP is correctly configured on both the ASA and the Elastic Agent host.
*   **Parsing Failures**: If logs appear in the `logs-cisco_asa.log` data stream but contain `error.message`, verify that you are not using a custom `logging message` format on the ASA that deviates from the standard syslog header.
*   **Truncated Messages**: If using UDP and seeing truncated logs, increase the MTU size along the network path or switch to **TCP**.

## Vendor Resources

*   [Cisco ASA Series Syslog Messages Reference Guide](https://www.cisco.com/c/en/us/td/docs/security/asa/syslog/asa-syslog.html)
*   [Cisco ASA 9.18 General Operations CLI Configuration Guide - Monitoring](https://www.cisco.com/c/en/us/td/docs/security/asa/asa918/configuration/general/asa-918-general-config/monitor-syslog.html)

# Documentation sites

*   [Elastic Integration Documentation: Cisco ASA](https://www.elastic.co/docs/reference/integrations/cisco_asa)
*   [Cisco ASA Support Documentation Home](https://www.cisco.com/c/en/us/support/security/adaptive-security-appliance-asa-software/series.html)