# Service Info

## Common use cases

*   **Security Threat Detection:** Monitor firewall traffic, intrusion prevention system (IPS) alerts, and antivirus logs to identify and mitigate cyber threats in real-time.
*   **Compliance Auditing:** Maintain long-term audit trails of user activity, administrative changes, and system events to meet regulatory requirements such as PCI-DSS, HIPAA, or GDPR.
*   **Network Troubleshooting:** Analyze traffic patterns and session data to diagnose connectivity issues, optimize bandwidth usage, and verify VPN performance.

## Data types collected

This integration collects several categories of **Logs**, including:
*   **Traffic Logs:** Information on allowed and denied sessions, including source/destination IPs and protocols.
*   **Security Logs:** Events from specialized engines like Antivirus, Web Filter, Application Control, and IPS.
*   **System Event Logs:** Administrative activities, hardware status, and HA (High Availability) events.
*   **VPN Logs:** Detailed records of IPsec and SSL-VPN authentication and session activity.

## Compatibility

*   **FortiOS:** Tested and compatible with FortiOS versions 6.x, 7.0, 7.2, and 7.4.
*   **Elastic Agent:** Requires Elastic Agent version 7.14.0 or higher.
*   **Deployment:** Supports physical appliances, virtual machines (FortiGate-VM), and cloud-native instances (AWS, Azure, GCP).

## Scaling and Performance

*   **Throughput:** For high-traffic environments (e.g., >10,000 EPS), use **TCP** or **Reliable** (TCP with confirmation) mode to prevent data loss associated with UDP packet drops.
*   **Hardware Impact:** Logging to external syslog servers generally has a lower impact on FortiGate CPU than local disk logging.
*   **Optimization:** Use severity filters (e.g., `Information` vs `Debug`) to control the volume of data sent to the Elastic Agent and reduce network overhead.

# Set Up Instructions

## Vendor prerequisites

*   **Administrative Access:** Read/Write permissions to the FortiGate "Log & Report" and "System" settings.
*   **Network Access:** Ensure the FortiGate appliance can reach the Elastic Agent host over the network via the designated syslog port (typically UDP/TCP 514).
*   **License:** A valid FortiGuard license is required to generate logs for advanced security features (IPS, Antivirus, etc.).

## Elastic prerequisites

*   **Elastic Agent:** Must be installed and enrolled in a policy via Fleet.
*   **Integration Policy:** The "Fortinet FortiGate" integration must be added to the agent's policy.
*   **Input Configuration:** The integration must be configured to listen on the same protocol (UDP/TCP) and port as specified in the FortiGate settings.

## Vendor set up steps

### Method 1: Configuration via GUI
1.  Log in to the FortiGate administration interface.
2.  Navigate to **Log & Report > Log Settings**.
3.  In the **Remote Logging and Archiving** section, enable **Send Logs to Syslog**.
4.  Enter the IP address of the server where the Elastic Agent is running.
5.  Specify the **Port** (default `514`). This must match your Elastic Agent policy.
6.  Select the **Syslog Format**. CEF (Common Event Format) is recommended for best compatibility.
7.  Choose the **Log Types** to send (e.g., **All**).
8.  Set the minimum **Log Level** to `Information` to capture most relevant events.
9.  Click **Apply**.

### Method 2: Configuration via CLI
1.  Log in to the FortiGate CLI (SSH or Console).
2.  Execute the following commands:
    ```bash
    config log syslogd setting
        set status enable
        set server "ELASTIC_AGENT_IP"
        set mode udp
        set port 514
        set format default
        set facility local7
    end
    ```
3.  (Optional) Specify a source IP if the FortiGate has multiple interfaces:
    ```bash
    config log syslogd setting
        set source-ip "FORTIGATE_INTERFACE_IP"
    end
    ```

## Kibana set up steps

1.  In Kibana, go to **Management > Integrations**.
2.  Search for **Fortinet FortiGate** and select it.
3.  Click **Add Fortinet FortiGate**.
4.  Select an existing agent policy or create a new one.
5.  In the integration settings, specify the **Listen Port** and **Listen Address** (e.g., `0.0.0.0` or a specific interface IP).
6.  Choose the **Protocol** (UDP or TCP) to match your FortiGate configuration.
7.  Click **Save and continue** and then **Add agent to your hosts** if the agent is not yet deployed.

# Validation Steps

1.  **Generate Test Logs:** In the FortiGate CLI, run the command `diagnose log test` to generate a series of dummy logs.
2.  **Verify Data Flow:** Go to Kibana **Observability > Logs > Stream** or **Discover**. 
3.  **Check for Dataset:** Filter logs by `data_stream.dataset : "fortinet.fortigate"` to ensure logs are arriving.
4.  **Review Dashboard:** Open the **[Logs Fortinet FortiGate] Overview** dashboard in Kibana to view pre-built visualizations.

# Troubleshooting

## Common Configuration Issues

*   **Port Conflict:** Ensure no other service on the Elastic Agent host is using the configured syslog port (e.g., a native `rsyslog` service).
*   **Firewall Blocking:** Verify that intermediate firewalls or host-based firewalls (iptables/firewalld) are allowing traffic on the syslog port.
*   **Source IP Mismatch:** If the FortiGate is routing traffic through a specific interface, use `set source-ip` in the CLI to ensure the traffic originates from an expected IP.

## Ingestion Errors

*   **Parsing Failures:** If `error.message` appears in the logs, verify that the `format` on the FortiGate (default/CEF) matches the expected format in the integration settings.
*   **Timestamp Mismatches:** Ensure the FortiGate and Elastic Agent host have synchronized clocks via NTP to prevent indexing delays or "future" logs.

## API Authentication Errors

Not specified (Syslog-based integration does not typically utilize API authentication for log forwarding).

## Vendor Resources

*   [Fortinet Technical Tip: How to configure syslog](https://community.fortinet.com/t5/FortiGate/Technical-Tip-How-to-configure-syslog-on-FortiGate/ta-p/331959)
*   [Fortinet Documentation: Log Settings Reference](https://docs.fortinet.com/document/fortigate/latest/administration-guide/464816/log-settings)
*   [Fortinet Community Support](https://community.fortinet.com/)

# Documentation sites

*   [Elastic Fortinet Integration Guide](https://www.elastic.co/docs/reference/integrations/fortinet_fortigate)
*   [FortiGate Product Documentation](https://docs.fortinet.com/product/fortigate)
*   [FortiOS CLI Reference](https://docs.fortinet.com/document/fortigate/latest/cli-reference)