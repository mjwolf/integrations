# Service Info

## Common use cases

- **Security Compliance and Auditing:** Track administrative login attempts, configuration changes, and unauthorized access attempts across Aruba CX network infrastructure.
- **Network Performance Monitoring:** Monitor system events and hardware alerts to proactively identify and resolve performance bottlenecks or interface flaps.
- **Operational Troubleshooting:** Centralize logs to diagnose routing issues, VLAN membership changes, and Virtual Switching Extension (VSX) synchronization events.

## Data types collected

- **Logs:** System logs (syslog) including audit logs, configuration changes, and hardware events.
- **Events:** Critical system state changes, performance alerts, and service availability notifications.

## Compatibility

- **ArubaOS-CX (AOS-CX):** Versions 10.x and higher.
- **Hardware Platforms:** Compatible with CX 6000, 6100, 6200, 6300, 6400, 8300, and 8400 series switches.

## Scaling and Performance

- **Throughput:** Aruba CX switches support high-velocity logging; however, rate limiting may occur during extreme event bursts to protect switch CPU.
- **Payload Limits:** Starting with AOS-CX 10.11, the maximum REST payload sent to a syslog server is 3500 characters.
- **Management Traffic:** Use of a dedicated Management VRF (Virtual Routing and Forwarding) is recommended to isolate logging traffic from data plane traffic.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** SSH or console access to the Aruba CX switch Command Line Interface (CLI).
- **Network Connectivity:** Unrestricted network path between the switch (management or data port) and the Elastic Agent on the configured syslog port (usually UDP 514 or 5514).

## Elastic prerequisites

- **Elastic Agent:** An active Elastic Agent must be installed and enrolled in Fleet.
- **Integration Policy:** The "Custom Logs" or "UDP/TCP Syslog" integration must be added to the agent policy with a matching port and protocol.

## Vendor set up steps

1.  Access the Aruba CX switch CLI via SSH or a console connection.
2.  Enter the global configuration context:
    ```bash
    configure
    ```
3.  Add the Elastic Agent as a syslog destination:
    ```bash
    logging <elastic-agent-ip-or-fqdn>
    ```
4.  (Optional) Set the minimum severity level for logs (e.g., warning, error, critical):
    ```bash
    logging <elastic-agent-ip-or-fqdn> severity warning
    ```
5.  (Optional) Specify the VRF if using a dedicated management network:
    ```bash
    logging <elastic-agent-ip-or-fqdn> vrf mgmt
    ```
6.  Verify the logging configuration:
    ```bash
    show logging
    ```
7.  Save the configuration to persistent memory:
    ```bash
    write memory
    ```

## Kibana set up steps

1.  Log in to Kibana and navigate to **Management > Integrations**.
2.  Search for and select the **Aruba CX** integration (or use the Syslog integration if a dedicated one is not present).
3.  Click **Add Aruba CX**.
4.  Configure the **Syslog Host** (usually `0.0.0.0`) and **Port** to match the target of your switch's logging command.
5.  Assign the integration to an **Agent Policy** and click **Save and Continue**.

# Validation Steps

1.  On the Aruba CX switch, trigger a test log entry by entering and exiting configuration mode:
    ```bash
    configure
    exit
    ```
2.  Run the command `show logging -r` to verify the entry was generated locally.
3.  In Kibana, navigate to **Analytics > Discover**.
4.  Filter by `event.dataset : "aruba_cx.log"` or search for the switch IP address to confirm data ingestion.

# Troubleshooting

## Common Configuration Issues

- **VRF Routing:** If the Elastic Agent is unreachable, ensure the correct VRF is specified in the `logging` command. If no VRF is specified, the switch uses the `default` VRF.
- **UDP Port Mismatch:** Ensure the port configured in the Elastic Agent policy matches the default syslog port (514) unless otherwise specified.
- **Security Groups/ACLs:** Verify that any intermediary firewalls or switch ACLs allow traffic from the switch to the Elastic Agent IP.

## Ingestion Errors

- **Timestamp Parsing:** If logs appear with the wrong timestamp, ensure NTP is configured on the switch using `ntp server <ip-address>`.
- **Field Mapping:** If `message` fields are not being parsed correctly, ensure the switch is using the standard syslog format (enabled by default).

## API Authentication Errors

- **Not specified:** This integration primarily uses Syslog over UDP/TCP; API authentication is not typically required for standard log ingestion.

## Vendor Resources

- [ArubaOS-CX Documentation Portal](https://arubanetworking.hpe.com/techdocs/AOS-CX/help_portal/Content/home.htm)
- [Aruba Support Knowledge Base](https://asp.arubanetworks.com/)
- [AOS-CX CLI Reference Guide](https://cadinc.com/wp-content/uploads/2024/08/CLI-Ref-Guide-2019v1-HPE-Aruba-Cisco-CX-from-Carolina-Advanced-Digtial.pdf)

# Documentation sites

- [ArubaOS-CX Switching Platform - Configuration Guide](https://docs.fortinet.com/document/fortisiem/7.1.4/external-systems-configuration-guide/259298/arubaos-cx-switching-platform)
- [HPE Aruba Networking Product Documentation](https://www.hpe.com/us/en/networking.html)
- [Elastic Integration Guide](https://docs.elastic.co/en/integrations)