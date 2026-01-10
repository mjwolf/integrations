# Service Info

## Common use cases

*   **Security Information and Event Management (SIEM):** Aggregating and normalizing security events from diverse network hardware like firewalls, IDS/IPS, and load balancers into a single format for threat detection.
*   **Centralized Compliance Logging:** Meeting regulatory requirements by collecting standardized audit trails from heterogeneous systems using the industry-standard CEF protocol.
*   **Infrastructure Monitoring:** Correlating operational events from different vendors (e.g., Fortinet, Cisco, Check Point) to identify network performance issues or security breaches.

## Data types collected

This integration primarily collects **logs** and **events** formatted according to the Micro Focus (formerly ArcSight) Common Event Format (CEF). These messages are typically delivered via the Syslog protocol and contain metadata such as severity, device vendor, device product, and specific event IDs.

## Compatibility

The CEF integration is compatible with any hardware or software source capable of exporting logs in the CEF format. This includes, but is not limited to:
*   **Fortinet:** FortiGate/FortiOS 6.x and 7.x.
*   **Cisco:** Cyber Vision and various security appliances.
*   **Trend Micro:** Deep Security and Apex Central.
*   **Check Point:** Quantum Security Gateways.
*   **MikroTik:** RouterOS devices.

## Scaling and Performance

Performance is largely dependent on the Elastic Agent's host resources and the transport protocol used. Using **TCP** or **TLS** is recommended for high-throughput environments to ensure delivery reliability, while **UDP** may experience packet loss under extreme load. For high-volume log streams, consider deploying multiple Elastic Agents behind a load balancer to distribute the Syslog processing load.

# Set Up Instructions

## Vendor prerequisites

*   **Administrative Access:** You must have permissions to modify logging or syslog settings on the source device.
*   **CEF Support:** Verify that your device or application natively supports "Common Event Format" or "CEF" as a logging output.
*   **Network Connectivity:** The source device must have network access to the Elastic Agent host on the configured Syslog port (standard ports are 514 or 6514).

## Elastic Prerequisites

*   **Elastic Agent:** Must be installed and enrolled in a Fleet policy.
*   **Policy Configuration:** The agent policy must include the "Custom UDP/TCP Logs" or specific "CEF" integration (depending on the package version).
*   **Input Listener:** The Elastic Agent must be configured to listen on a specific IP and Port that matches the destination configured on the vendor device.

## Vendor set up steps

### General Configuration (Applicable to most devices)
1.  **Access Management Interface:** Log in to the device's web UI or CLI.
2.  **Navigate to Logging/Syslog:** Locate the remote logging or syslog server configuration section.
3.  **Add Remote Target:** Create a new syslog destination.
4.  **Configure Elastic Agent Destination:** 
    *   **IP Address:** Enter the IP of the host running Elastic Agent.
    *   **Port:** Use the port configured in the Elastic Agent integration (e.g., `514`).
    *   **Protocol:** Select TCP or TLS for reliability, or UDP if performance is a priority.
5.  **Set Format to CEF:** Select **CEF** or **Common Event Format** from the log format options.
6.  **Save and Enable:** Apply the changes to begin transmitting logs.

### Specific Example: Fortinet FortiGate (via CLI)
1.  **Access CLI:** Connect via SSH or console.
2.  **Enter Syslog Mode:** 
    ```shell
    config log syslogd setting
    ```
3.  **Configure Server:**
    ```shell
    set status enable
    set format cef
    set server <elastic_agent_ip>
    set port <port_number>
    end
    ```
4.  **Define Filter:**
    ```shell
    config log syslogd filter
    set severity information
    set forward-traffic enable
    end
    ```

## Kibana set up steps

1.  **Navigate to Integrations:** In Kibana, go to **Management > Integrations**.
2.  **Search for CEF:** Select the **CEF** (or Custom Syslog) integration.
3.  **Add to Policy:** Click **Add CEF** and choose an existing or new Agent Policy.
4.  **Configure Input:** 
    *   Set the **Listen Address** (e.g., `0.0.0.0`).
    *   Set the **Listen Port** to match your vendor configuration (e.g., `514`).
    *   Select the appropriate **Protocol** (UDP or TCP).
5.  **Save and Deploy:** Click **Save and Continue** to push the configuration to your agents.

# Validation Steps

1.  **Check Agent Logs:** Verify the Elastic Agent has started the syslog listener without errors in the **Fleet > Agents** logs.
2.  **Generate Traffic:** Trigger an event on the source device (e.g., a login attempt or a firewall rule match).
3.  **Inspect Discover:** In Kibana, go to **Discover** and search for `data_stream.dataset : "cef.log"`.
4.  **Verify Fields:** Ensure fields like `deviceVendor`, `deviceProduct`, and `cef.extensions` are correctly populated and not just raw text.

# Troubleshooting

## Common Configuration Issues

*   **Connectivity Interruption:** Check if local firewalls (iptables, firewalld, or Windows Firewall) on the Elastic Agent host are blocking the configured Syslog port.
*   **Incorrect Port/Protocol:** Ensure the protocol (UDP vs TCP) matches exactly between the vendor device and the Kibana integration settings.
*   **Binding Errors:** If the Elastic Agent cannot start the listener, ensure no other service is already using the configured port (e.g., a native rsyslog service).

## Ingestion Errors

*   **Parsing Failures:** If data appears in the `message` field but not in structured fields, the source device might not be sending valid CEF headers. Check for `error.message` fields in the logs.
*   **Timezone Mismatch:** CEF logs often lack timezone offsets; ensure the Elastic Agent or the source device is configured with a consistent time reference (UTC is recommended).

## API Authentication Errors

*   **Not specified:** This integration typically uses Syslog (push-based) and does not require API-level authentication between the vendor and Elastic.

## Vendor Resources

*   [Fortinet CEF Support Reference](https://docs.fortinet.com/document/fortigate/7.4.1/fortios-log-message-reference/604144/cef-support)
*   [Micro Focus CEF Integration Guide](https://www.microfocus.com/documentation/arcsight/arcsight-smartconnectors/PDF_Guides/CommonEventFormat.pdf)
*   [MikroTik CEF Configuration](https://help.mikrotik.com/docs/spaces/ROS/pages/319782960/CEF+with+Elasticsearch)

# Documentation sites

*   [Elastic CEF Integration Overview](https://docs.elastic.co/en/integrations/cef)
*   [Microsoft Sentinel CEF Troubleshooting (General Reference)](https://learn.microsoft.com/en-us/azure/sentinel/connect-cef-syslog-ama)
*   [Cisco Cyber Vision CEF Guide](https://www.cisco.com/c/en/us/td/docs/security/cyber_vision/3-0/configuration/guide/Cisco_Cyber_Vision_Configuration_Guide_3_0/Cisco_Cyber_Vision_Configuration_Guide_3_0_chapter_0110.html)