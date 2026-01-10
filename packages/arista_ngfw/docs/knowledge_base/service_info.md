# Service Info

## Common use cases

*   **Security Threat Monitoring:** Track and alert on malicious activity identified by the Arista NG Firewall, such as blocked malware downloads or intrusion attempts.
*   **Web Traffic Analysis:** Monitor web usage and HTTP requests across the network to identify high-bandwidth users or access to restricted categories.
*   **Network Auditing and Compliance:** Maintain a detailed log of network sessions and administrative changes to satisfy regulatory requirements and forensic investigations.

## Data types collected

This integration primarily collects **Logs** and **Events** via the syslog protocol. This includes session details, firewall blocks, application-specific events (e.g., HTTP, DNS), and administrative alerts.

## Compatibility

This integration is compatible with Arista NG Firewall (formerly Untangle ETM) versions that support Remote Syslog functionality. It has been tested against recent releases of the ETM NG Firewall platform.

## Scaling and Performance

Performance is dependent on the volume of network traffic and the number of event classes selected for forwarding. For high-throughput environments (1Gbps+), it is recommended to use **UDP** for lower overhead or ensure the Elastic Agent host has sufficient resources to handle the incoming syslog stream. To optimize performance, use specific **Syslog Rules** to filter out high-volume, low-value events like repetitive session notifications.

# Set Up Instructions

## Vendor prerequisites

*   Administrative access to the Arista NG Firewall web interface.
*   A valid subscription or license that includes the "Events" and "Reports" functionality.
*   Network connectivity between the Arista NG Firewall and the Elastic Agent host over the chosen syslog port (default 514).

## Elastic prerequisites

*   Elastic Agent must be installed and enrolled in a policy.
*   The **Arista NG Firewall** integration must be added to the agent policy.
*   The syslog input must be configured to match the protocol (UDP/TCP) and port specified in the Arista settings.

## Vendor set up steps

1.  Log in to your Arista NG Firewall administration interface.
2.  Navigate to **Config > Events > Syslog**.
3.  Check the box to **Enable Remote Syslog**.
4.  Configure the syslog server settings:
    *   **Host**: The IP address of the server where the Elastic Agent is running.
    *   **Port**: The port the Elastic Agent is listening on (e.g., `514`).
    *   **Protocol**: Select `UDP` (default) or `TCP` based on your Elastic Agent configuration.
5.  Navigate to the **Syslog Rules** tab.
6.  Click **Add** to create a rule for data forwarding.
7.  To forward all firewall events, use the following:
    *   **Enable**: Checked.
    *   **Class**: `All`.
    *   **Conditions**: Leave blank to include all traffic.
8.  Click **Done** and then click **Save** at the bottom of the page to apply the changes.

## Kibana set up steps

1.  In Kibana, go to **Management > Integrations**.
2.  Search for **Arista NG Firewall** and select it.
3.  Click **Add Arista NG Firewall**.
4.  Configure the integration settings:
    *   **Syslog Host**: Set to `0.0.0.0` or the specific interface IP to listen on.
    *   **Syslog Port**: Ensure this matches the port set in the Arista UI (e.g., `9514` is often used to avoid conflicts with system syslog).
5.  Select the **Existing host policy** where your Elastic Agent is enrolled.
6.  Click **Save and continue**.

# Validation Steps

1.  **Generate Traffic:** Access a website or trigger a firewall rule on a device behind the Arista NG Firewall.
2.  **Verify Flow:** In Arista NG Firewall, go to **Reports > Events** to confirm events are being generated locally.
3.  **Check Kibana Discover:** In Kibana, navigate to **Discover** and filter for `event.dataset : "arista_ng_firewall.log"`.
4.  **Inspect Dashboards:** Open the **[Logs Arista NG Firewall] Overview** dashboard in Kibana to view visualized event data.

# Troubleshooting

## Common Configuration Issues

*   **Connectivity:** Ensure that any intermediate firewalls or the host OS firewall (e.g., iptables/firewalld) allow traffic on the configured syslog port.
*   **Port Conflicts:** If using port 514, ensure the native system syslog (rsyslog/syslog-ng) on the Agent host is not already binding to that port.
*   **Protocol Mismatch:** If the firewall is sending via UDP but the Agent is configured for TCP, no data will appear.

## Ingestion Errors

*   **Parsing Failures:** If `error.message` indicates a parsing issue, verify that the Arista event format has not been customized. This integration expects the standard JSON-formatted syslog output from Arista ETM.
*   **Timezone Mismatch:** If logs appear with the wrong timestamp, ensure the time settings on the Arista NG Firewall (Config > System > Time) match the expected UTC offset.

## API Authentication Errors

Not specified (This integration uses Syslog and does not require API credentials for data ingestion).

## Vendor Resources

*   [Arista ETM Wiki - Events Documentation](https://wiki.edge.arista.com/index.php/Events)
*   [Arista Edge Threat Management Support Reference](https://www.arista.com/en/ug-etm-ngf/etm-ngf-reference-material)
*   [Arista Support Portal](https://www.arista.com/en/support)

# Documentation sites

*   [Official Arista NG Firewall Product Page](https://www.arista.com/en/products/edge-threat-management-ng-firewall)
*   [Arista NG Firewall User Guide](https://www.arista.com/assets/data/pdf/user-manual/um-books/ETM-NG-Firewall-User-Guide.pdf)
*   [Elastic Integration Documentation](https://docs.elastic.co/en/integrations/arista_ng_firewall)