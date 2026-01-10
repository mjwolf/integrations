# Service Info

## Common use cases

- **Security Threat Monitoring:** Monitor firewall events and blocked traffic in real-time to identify potential intrusion attempts or unauthorized access patterns within the network.
- **Network Troubleshooting:** Audit DHCP and DNS logs to diagnose connectivity issues, IP address conflicts, and resolution failures across client devices.
- **Compliance and Auditing:** Maintain long-term records of VPN connections and administrative authentication events to satisfy regulatory requirements and internal security audits.

## Data types collected

This integration collects **Logs** via syslog. Specific event types include Firewall events, System events, DNS events, DHCP events, VPN (IPsec/OpenVPN) events, Authentication events, and Gateway Monitor events.

## Compatibility

This integration is compatible with **pfSense 2.4.x, 2.5.x, 2.6.x, and 2.7.x (Community Edition)** as well as **pfSense Plus**. It supports the standard **BSD (RFC 3164)** syslog format.

## Scaling and Performance

Performance is dependent on the pfSense hardware and the volume of network traffic. For high-throughput environments (e.g., 1Gbps+ with heavy logging), ensure the Elastic Agent is deployed on a host with sufficient CPU and memory to handle rapid UDP/TCP syslog bursts. Using a load balancer or multiple Elastic Agents is recommended for massive firewall deployments.

# Set Up Instructions

## Vendor prerequisites

- Administrative access to the pfSense WebGUI.
- Network connectivity between the pfSense appliance and the Elastic Agent (ensure no intermediate firewalls block the configured syslog port).
- pfSense must have the "System Logs" feature enabled (default).

## Elastic prerequisites

- Elastic Agent must be installed and enrolled in a policy.
- The pfSense integration must be added to the agent policy.
- A dedicated UDP or TCP port (e.g., 514 or 5514) must be open on the Elastic Agent host to receive incoming syslog traffic.

## Vendor set up steps

1.  Log in to the **pfSense web interface**.
2.  Navigate to **Status > System Logs** and click on the **Settings** tab.
3.  Locate the **General Logging Options** section.
4.  Check the box for **"Send log messages to remote syslog server"**.
5.  In the **Remote log servers** section, configure the destination:
    *   **Remote syslog server 1**: Enter the IP address of the Elastic Agent.
    *   **Port**: Enter the port number configured in the Elastic integration (e.g., 514).
6.  In the **Remote Syslog Contents** section, select the logs to forward. It is recommended to check:
    *   **Everything** (or manually select Firewall, System, DNS, DHCP, VPN, and Auth events).
7.  Set the **Log Message Format** to **BSD (RFC 3164)**.
8.  Click **Save** at the bottom of the page to apply changes.

## Kibana set up steps

1.  In Kibana, go to **Management > Integrations**.
2.  Search for **pfSense** and select the integration.
3.  Click **Add pfSense**.
4.  Under **Integration settings**, configure the **Syslog Host** (usually `0.0.0.0`) and the **Syslog Port** to match the port specified in the pfSense WebGUI.
5.  Select the **Protocol** (UDP is standard for pfSense syslog).
6.  Choose the existing **Agent Policy** where your Elastic Agent is enrolled.
7.  Click **Save and continue**.

# Validation Steps

1.  **Check Data Stream:** In Kibana, navigate to **Analytics > Discover** and filter for `event.dataset : "pfsense.log"`. Verify that logs are appearing.
2.  **Verify Dashboards:** Navigate to **Analytics > Dashboards** and search for "[Logs pfSense] Overview" to ensure visualizations are populating with firewall and system data.
3.  **Trigger Test Event:** Create a temporary firewall rule or attempt a failed login to the pfSense WebGUI to verify that specific events are immediately forwarded and indexed.

# Troubleshooting

## Common Configuration Issues

- **Firewall Blocks:** If no logs appear, ensure the host OS running Elastic Agent allows inbound traffic on the configured syslog port (e.g., `ufw allow 514/udp` or Windows Firewall rules).
- **Incorrect IP/Port:** Verify that the "Remote syslog server" IP in pfSense matches the Elastic Agent's reachable internal IP address.
- **Port Conflict:** Ensure no other service is already using the chosen syslog port on the Elastic Agent host.

## Ingestion Errors

- **Parsing Failures:** If `error.message` contains parsing errors, ensure the **Log Message Format** in pfSense is strictly set to **BSD (RFC 3164)**. Other formats like Syslog (RFC 5424) may cause field mapping issues.
- **Timestamp Mismatch:** Ensure the time settings on the pfSense appliance and the Elastic Agent host are synchronized via NTP to avoid indexing delays or "future" data.

## API Authentication Errors

- Not specified (pfSense syslog forwarding does not use API-based authentication for standard log delivery).

## Vendor Resources

- [Netgate pfSense Documentation: Remote Logging](https://docs.netgate.com/pfsense/en/latest/monitoring/logs/remote.html)
- [Netgate Troubleshooting Firewall Logs](https://docs.netgate.com/pfsense/en/latest/monitoring/logs/firewall.html)
- [Netgate Forum - Logging & Monitoring](https://forum.netgate.com/category/50/logging-and-monitoring)

# Documentation sites

- [pfSense Product Overview](https://www.pfsense.org/)
- [Netgate Official Documentation](https://docs.netgate.com/pfsense/en/latest/index.html)
- [Elastic pfSense Integration Reference](https://docs.elastic.co/integrations/pfsense)