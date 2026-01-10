# Service Info

## Common use cases
*   **Network Security Monitoring:** Identify suspicious traffic patterns, potential data exfiltration, or unauthorized access attempts by analyzing flow records.
*   **Capacity Planning:** Monitor bandwidth utilization across network interfaces to identify bottlenecks and plan for infrastructure upgrades.
*   **Troubleshooting Network Performance:** Diagnose latency issues and "top talker" applications by analyzing protocol distribution and traffic volume between specific source and destination IPs.

## Data types collected
*   **Network Flow Records (Events):** Collects IP traffic metadata including source/destination addresses, protocols, port numbers, byte/packet counts, and interface identifiers.
*   **Metrics:** Derived flow data such as traffic volume (bytes/packets) and duration of network sessions.

## Compatibility
*   **Cisco IOS/IOS-XE:** Supports Flexible NetFlow (v9) and Traditional NetFlow (v5).
*   **NetFlow Protocols:** Compatible with NetFlow version 5 and version 9.
*   **Elastic Agent:** Compatible with current versions of the Elastic Agent and Fleet.

## Scaling and Performance
Performance is highly dependent on the device's cache timeout settings. Setting an `active timeout` of 60 seconds ensures a steady stream of data for long-lived flows, while an `inactive timeout` of 15 seconds helps clear short-lived entries from the device's memory. For high-volume environments, ensure the Elastic Agent host has sufficient UDP buffer sizing to prevent packet loss.

# Set Up Instructions

## Vendor prerequisites
*   Administrative access (privileged EXEC mode) to the Cisco CLI.
*   Network connectivity between the Cisco device's management or source interface and the Elastic Agent's listening IP address.
*   License support for NetFlow or Flexible NetFlow on the specific Cisco hardware/software version.

## Elastic prerequisites
*   Elastic Agent must be installed and enrolled in Fleet.
*   The NetFlow integration must be added to the Agent policy.
*   A dedicated UDP port (e.g., 2055) must be open on the Agent host's firewall to receive inbound traffic.

## Vendor set up steps
1.  **Enter Configuration Mode:**
    ```bash
    enable
    configure terminal
    ```
2.  **Configure Flow Record (v9):** Define fields to capture, including `ipv4 protocol`, `transport source-port`, and `counter bytes`.
3.  **Configure Flow Exporter:** Point the device to the Elastic Agent IP and UDP port:
    ```bash
    flow exporter ELASTIC_NETFLOW_EXPORTER
     destination <agent-ip-address>
     transport udp <agent-port>
     export-protocol netflow-v9
     template data timeout 60
    ```
4.  **Configure Flow Monitor:** Link the record and exporter, and set `cache timeout active 60`.
5.  **Apply to Interface:** Enable the monitor on specific interfaces using the `ip flow monitor ELASTIC_NETFLOW_MONITOR input` command.
6.  **Traditional NetFlow (v5) Alternative:** Use `ip flow-export destination <agent-ip-address> <agent-port>` and `ip flow-export version 5` for legacy hardware.

## Kibana set up steps
1.  In Kibana, navigate to **Management > Integrations** and search for **NetFlow**.
2.  Select **Add NetFlow** and configure the integration settings.
3.  Set the **UDP listener host** (typically `0.0.0.0`) and the **UDP listener port** to match your Cisco exporter configuration (e.g., `2055`).
4.  Select the **Existing hosts** policy where your Elastic Agent is enrolled.
5.  Click **Save and continue** to deploy the configuration to the Agent.

# Validation Steps
1.  **Verify Device Export:** On the Cisco device, run `show flow exporter` and `show flow monitor` to confirm flows are being captured and exported.
2.  **Check Agent Logs:** Review Elastic Agent logs for any "address already in use" or "permission denied" errors related to the UDP port.
3.  **Confirm Ingestion:** Go to **Kibana > Discover** and filter for `data_stream.dataset : "netflow.log"` to view incoming flow records.
4.  **Dashboard Check:** Open the **[Metrics NetFlow] Overview** dashboard in Kibana to ensure visualizations are populating with data.

# Troubleshooting

## Common Configuration Issues
*   **Firewall Blocks:** Ensure the Elastic Agent host's firewall allows inbound UDP traffic on the configured port.
*   **Duplicate Data:** If monitoring both input and output on the same interface, flows may be counted twice. Only apply `input` monitoring across all interfaces to avoid this.
*   **Template Mismatch:** If using v9, data may not appear until the template is received. The `template data timeout 60` command ensures the template is sent frequently.

## Ingestion Errors
*   **Parsing Failures:** Occur if the Cisco flow record fields do not match the expected Elastic schema. Ensure all mandatory fields (source/dest IP, ports, protocol) are included in the flow record.
*   **Version Mismatch:** Ensure the integration's version setting (v5 vs v9) matches the version configured in the Cisco `ip flow-export version` or `export-protocol` command.

## API Authentication Errors
*   **Not specified:** This integration uses the UDP protocol for data transport and does not rely on vendor API authentication.

## Vendor Resources
*   [Cisco NetFlow Configuration Guide](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_pi/configuration/15-s/nf-15-s-book/cfg-nflow-data-expt.html)
*   [SolarWinds: Configure NetFlow for Cisco Routers](https://solarwindscore.my.site.com/SuccessCenter/s/article/How-to-configure-NetFlow-for-Cisco-routers-and-switches-running-IOS-video)

# Documentation sites
*   [Elastic NetFlow Integration Documentation](https://docs.elastic.co/en/integrations/netflow)
*   [Cisco Flexible NetFlow Reference](https://www.cisco.com/c/en/us/products/ios-nx-os-software/flexible-netflow/index.html)
*   [Official NetFlow v9 Export Format (RFC 3954)](https://datatracker.ietf.org/doc/html/rfc3954)