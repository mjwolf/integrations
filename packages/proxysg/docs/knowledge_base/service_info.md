# Service Info

## Common use cases

*   **Security Monitoring and Threat Detection:** Analyze web traffic logs to identify communication with malicious domains, command-and-control (C2) servers, or unauthorized data exfiltration.
*   **Compliance and Auditing:** Maintain a comprehensive audit trail of system events and user web activity to meet regulatory requirements such as GDPR, HIPAA, or PCI-DSS.
*   **Operational Troubleshooting:** Monitor ProxySG system health and performance through event logs to diagnose connectivity issues or configuration errors in the proxy environment.

## Data types collected

The Bluecoat ProxySG integration primarily collects **logs** in Syslog format. These include system event logs (startup, configuration changes, and hardware status) and web access logs containing details on client requests and server responses.

## Compatibility

This integration is compatible with Bluecoat ProxySG and Symantec Edge SWG appliances. Specific version support is not restricted, provided the appliance supports standard Syslog forwarding and the W3C Extended Log File Format (ELFF) for access logs.

## Scaling and Performance

Scaling and throughput are dependent on the hardware specifications of the ProxySG appliance and the volume of web traffic. For high-traffic environments, ensure the Elastic Agent is hosted on a machine with sufficient resources to handle incoming UDP/TCP syslog bursts.

# Set Up Instructions

## Vendor prerequisites

*   Administrative access to the Bluecoat ProxySG (Symantec Edge SWG) Management Console.
*   An active license for the ProxySG appliance.
*   Network connectivity between the ProxySG appliance and the Elastic Agent (typically via port 514 or a custom syslog port).

## Elastic prerequisites

*   Elastic Agent must be installed and enrolled in a policy.
*   The "Bluecoat ProxySG" integration must be added to the Agent policy.
*   UDP or TCP port listener configured in the integration settings to match the ProxySG output.

## Vendor set up steps

1.  **Log in to the ProxySG Management Console:** Open a web browser and navigate to the IP address or hostname of your Bluecoat ProxySG appliance.
2.  **Navigate to Event Logging:** Click on the **Maintenance** tab. From the left-hand menu, select **Event Logging**, then click the **Syslog** tab.
3.  **Configure the Syslog Destination:** In the **Loghost** text box, enter the IP address of the Elastic Agent server.
4.  **Enable Syslog Forwarding:** Check the box next to **Enable syslog** to activate forwarding.
5.  **Set the Logging Level:** Click the **Level** tab within Event Logging and select **Informational** to ensure a comprehensive level of detail is transmitted.
6.  **Apply the Changes:** Click the **Apply** button to save the configuration and start data transmission.

## Kibana set up steps

1.  Log in to Kibana and navigate to **Management > Integrations**.
2.  Search for **Bluecoat ProxySG** and select it.
3.  Click **Add Bluecoat ProxySG**.
4.  Configure the **Syslog Host** (typically `0.0.0.0`) and **Syslog Port** (e.g., `514` or `9514`) to match your environment.
5.  Select the **Agent Policy** where you want to add the integration.
6.  Click **Save and continue** and then **Add agent integration**.

# Validation Steps

1.  **Generate Test Traffic:** Access several websites through the ProxySG to generate access log entries.
2.  **Verify Data Stream:** Navigate to **Kibana > Discover**. Filter for `event.dataset : "bluecoat.proxysg"` to confirm logs are arriving.
3.  **Check Dashboards:** Open the **[Logs Bluecoat ProxySG] Overview** dashboard in Kibana to ensure visualizations are populating with data.
4.  **Check Agent Status:** In **Fleet**, verify the Elastic Agent status is "Healthy."

# Troubleshooting

## Common Configuration Issues

*   **Connectivity Issues:** Ensure that firewalls between the ProxySG appliance and the Elastic Agent allow traffic on the configured syslog port (UDP/TCP).
*   **Logs Not Appearing:** Verify that the "Enable syslog" checkbox is checked in the ProxySG Management Console and that the "Apply" button was clicked.
*   **Incorrect Log Level:** If system events are missing, ensure the logging level is set to "Informational" rather than "Error" or "Warning."

## Ingestion Errors

*   **Parsing Failures:** If fields are not correctly extracted, check `error.message` in Kibana. This often occurs if the ProxySG is using a non-standard log format.
*   **Timestamp Mismatch:** Ensure the ProxySG and Elastic Agent servers are synchronized via NTP to avoid indexing delays or incorrect time-sorting.

## API Authentication Errors

Not specified (Integration relies on Syslog forwarding, which typically does not use API-based authentication for data ingestion).

## Vendor Resources

*   [Broadcom Knowledge Base: Required ports and protocols](https://knowledge.broadcom.com/external/article/150987/required-ports-protocols-and-services-fo.html)
*   [Symantec ProxySG Security Troubleshooting Guide](https://knowledge.broadcom.com/external/article/165985/proxysg-troubleshooting-and-diagnostic-r.html)
*   [Broadcom Support Portal](https://support.broadcom.com/)

# Documentation sites

*   [Broadcom Edge SWG (ProxySG) Official Documentation](https://techdocs.broadcom.com/us/en/symantec-security-software/web-and-network-security/proxysg.html)
*   [Elastic Integration Reference](https://docs.elastic.co/integrations/bluecoat_proxysg)
*   [Trellix/Blue Coat Configuration Reference](https://docs.trellix.com/bundle/enterprise-security-manager-data-source-configuration-guide/page/UUID-3ab59958-f128-68ee-3d8f-cd8050d875bc.html)