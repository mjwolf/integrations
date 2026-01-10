# Service Info

## Common use cases

*   **Threat Detection and Prevention:** Monitor real-time logs from Palo Alto Networks firewalls to identify and respond to malicious activities, such as known vulnerabilities, spyware, and command-and-control traffic.
*   **Network Traffic Analysis:** Analyze traffic patterns and bandwidth usage across different security zones to optimize network performance and identify unauthorized access attempts.
*   **Compliance and Auditing:** Maintain a historical record of system configuration changes, administrator logins, and security policy modifications to satisfy regulatory requirements like PCI-DSS or HIPAA.

## Data types collected

This integration collects various log types via Syslog, including Traffic logs (session start/end), Threat logs (vulnerabilities and viruses), URL Filtering, Data Filtering, WildFire Submissions, System logs, Configuration logs, and User-ID events.

## Compatibility

This integration is compatible with Palo Alto Networks PAN-OS versions 10.2, 11.0, 11.1, and 11.2. It supports standard syslog formats including BSD (RFC 3164) and IETF (RFC 5424).

## Scaling and Performance

Performance is primarily determined by the syslog throughput of the Palo Alto firewall and the ingestion capacity of the Elastic Agent. For high-volume environments, it is recommended to use the TCP transport protocol with SSL/TLS for reliability and to ensure the Elastic Agent is deployed on hardware that meets the EPS (Events Per Second) requirements of the network edge.

# Set Set Up Instructions

## Vendor prerequisites

*   Administrative access to the Palo Alto Networks PAN-OS web interface.
*   A valid license for logging features (Traffic, Threat, etc.) on the firewall.
*   Network connectivity between the firewall management or data interface and the Elastic Agent on the configured syslog port.

## Elastic prerequisites

*   Elastic Agent must be installed and enrolled in Fleet.
*   The Palo Alto Networks integration must be added to the Agent policy.
*   The specified listening port (e.g., 9001) must be open on the host where Elastic Agent is running.

## Vendor set up steps

1.  **Create a Syslog Server Profile**: Navigate to **Device > Server Profiles > Syslog** and click **Add**. Define a profile name, then add a server entry with the IP address of your Elastic Agent, the desired transport (TCP/UDP/SSL), and the matching port number.
2.  **Create a Log Forwarding Profile**: Navigate to **Objects > Log Forwarding** and click **Add**. Create match lists for different log types (Traffic, Threat, etc.) and assign the Syslog Server Profile created in step 1 to each match list.
3.  **Apply Profile to Security Policies**: Navigate to **Policies > Security**, select relevant rules, and under the **Actions** tab, select your Log Forwarding Profile from the dropdown. Ensure "Log at Session End" is enabled.
4.  **Configure System Log Settings**: Navigate to **Device > Log Settings**. For System, Config, and User-ID logs, click the log type, add the desired severity levels, and select the Syslog Server Profile.
5.  **Commit Changes**: Click the **Commit** button in the top-right corner of the PAN-OS interface to apply and activate the logging configuration.

## Kibana set up steps

1.  In Kibana, go to **Management > Integrations** and search for "Palo Alto Networks".
2.  Click **Add Palo Alto Networks**.
3.  Configure the integration settings by selecting the correct **Syslog Host** (usually `0.0.0.0`) and **Syslog Port** to match the firewall's server profile.
4.  Select the **Protocol** (TCP or UDP) used by the firewall.
5.  Save the integration to apply it to your Elastic Agent policy.

# Validation Steps

1.  **Generate Traffic**: Trigger a test event by visiting a website or generating network traffic that matches a monitored security policy on the firewall.
2.  **Check Discover**: In Kibana, navigate to **Analytics > Discover** and filter for `event.dataset : "paloalto_networks.*"` to verify logs are appearing.
3.  **Review Dashboards**: Navigate to **Analytics > Dashboards** and search for "Palo Alto" to view pre-built visualizations for Traffic and Threat data.

# Troubleshooting

## Common Configuration Issues

*   **Firewall Changes Not Applied**: Ensure the **Commit** operation was successful in the PAN-OS interface; configuration changes do not take effect until committed.
*   **Connectivity Blocked**: Verify that intermediate firewalls or host-based firewalls (like `iptables` or `firewalld`) allow traffic on the configured syslog port (e.g., 9001).
*   **Protocol Mismatch**: Ensure the Transport protocol (TCP vs UDP) matches exactly between the PAN-OS Syslog Server Profile and the Elastic Agent integration settings.

## Ingestion Errors

*   **Parsing Failures**: If `error.message` indicates parsing issues, verify the **Format** in the Syslog Server Profile. Most Elastic integrations prefer **IETF (RFC 5424)**, though BSD is often supported.
*   **Field Mapping Issues**: Ensure the PAN-OS version is supported, as changes in log formats between major PAN-OS releases can occasionally cause mapping failures in older integration versions.

## API Authentication Errors

*   Not specified. Palo Alto log forwarding primarily uses Syslog (UDP/TCP/TLS) rather than a polling API for log ingestion.

## Vendor Resources

*   [Palo Alto Networks Troubleshooting Log Forwarding](https://knowledgebase.paloaltonetworks.com/KCSArticleDetail?id=kA10g000000ClGkCAK)
*   [Palo Alto Networks Documentation Portal](https://docs.paloaltonetworks.com/)
*   [PAN-OS Administrator Guide: Monitoring and Reporting](https://docs.paloaltonetworks.com/pan-os/11-0/pan-os-admin/monitoring)

# Documentation sites

*   [Elastic Integration for Palo Alto Networks](https://docs.elastic.co/integrations/panw)
*   [Configure Log Forwarding - Palo Alto Networks Official Guide](https://docs.paloaltonetworks.com/pan-os/11-0/pan-os-admin/monitoring/configure-log-forwarding)
*   [Palo Alto Networks XML API Reference](https://docs.paloaltonetworks.com/pan-os/11-0/pan-os-pan-view-api)