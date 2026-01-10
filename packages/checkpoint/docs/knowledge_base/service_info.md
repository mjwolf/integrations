# Service Info

## Common use cases

*   **Threat Detection and Monitoring:** Centralize security gateway traffic logs to identify malicious patterns, unauthorized access attempts, and potential security breaches across the network.
*   **Compliance Auditing:** Meet regulatory requirements (such as PCI-DSS, GDPR, or HIPAA) by maintaining a long-term, searchable archive of audit and security events in the Elastic Stack.
*   **Operational Visibility:** Monitor VPN connection trends, application usage, and system health across Check Point Management and Log Servers to troubleshoot network performance issues.

## Data types collected

The integration collects logs including Security Traffic logs, Audit logs, Threat Prevention events (IPS, Anti-Bot, Anti-Virus), and System logs (Management and Gateway events).

## Compatibility

This integration is compatible with Check Point versions R81 and higher. It has been specifically validated against R81.10 and R81.20 using the Log Exporter feature in SmartConsole.

## Scaling and Performance

The Check Point Log Exporter is a multi-threaded daemon designed for high throughput. Performance depends on the Log Server hardware and log volume; users can optimize performance by applying filters within the Log Exporter configuration to exclude noisy logs or by using multiple exporter instances for different log types.

# Set Up Instructions

## Vendor prerequisites

*   Check Point Management Server or Log Server running version R81 or higher.
*   Administrative access to Check Point SmartConsole.
*   Network connectivity between the Check Point Log Server and the Elastic Agent over the configured Syslog port (TCP or UDP).

## Elastic prerequisites

*   Elastic Agent installed and enrolled in a policy via Fleet.
*   The Check Point integration package added to the Elastic Agent policy.
*   Syslog input settings in the integration must match the port and protocol defined in the Check Point Log Exporter.

## Vendor set up steps

1.  **Create Log Exporter Object:** In SmartConsole, go to **Objects > More object types > Server > Log Exporter /SIEM**.
2.  **Configure General Settings:** Name the object (e.g., `elastic-agent-exporter`), set **Export Configuration** to **Enabled**, and enter the Elastic Agent's **Target Server** IP and **Target Port**.
3.  **Define Format:** On the **Data Manipulation** page, select `Syslog` or `CEF` as the format.
4.  **Assign to Server:** Navigate to **Gateways & Servers**, open your Management/Log Server properties, and under **Logs > Export**, add the new exporter object.
5.  **Apply Configuration:** Navigate to **Menu > Install database**, select all relevant objects, and click **Install** to push the changes to the Log Server.

## Kibana set up steps

1.  In Kibana, navigate to **Management > Integrations** and search for **Check Point**.
2.  Click **Add Check Point** and select the desired data streams (e.g., Firewall, Audit, IPS).
3.  Configure the **Listen Port** and **Listen Address** to match the values provided in the Check Point SmartConsole setup.
4.  Click **Save and continue** to deploy the configuration to your Elastic Agent.

# Validation Steps

*   **Check Daemon Status:** Log into the Check Point Management Server CLI and run `cp_log_export status` to ensure the exporter is "Running".
*   **Trigger Test Event:** Generate a log entry (e.g., a test login or a policy change) and verify it appears in Kibana **Discover** under `logs-checkpoint.*`.
*   **Verify Dashboards:** Open the **[Logs Check Point] Overview** dashboard in Kibana to ensure visualizations are populating with real-time data.

# Troubleshooting

## Common Configuration Issues

*   **Log Database Not Installed:** Changes to Log Exporter objects do not take effect until the "Install Database" operation is successfully completed in SmartConsole.
*   **Network Connectivity:** Ensure that intermediate firewalls allow traffic from the Log Server to the Elastic Agent on the specified TCP/UDP port.
*   **Incorrect Target IP:** Verify that the "Target Server" IP in the Log Exporter object matches the host IP where the Elastic Agent is running, not the Fleet Server IP.

## Ingestion Errors

*   **Format Mismatch:** If logs appear in Kibana but are not parsed correctly (e.g., remaining in the `message` field), ensure the format in Check Point is set to a standard `Syslog` or `CEF` mapping that matches the integration's expected input.
*   **Timezone Offsets:** If logs appear with the wrong timestamp, verify the system time on the Check Point Management Server and ensure the Elastic Agent integration is configured to handle the correct timezone.

## API Authentication Errors

Not specified (This integration primarily uses Syslog/Log Exporter which does not rely on API-based authentication for data ingestion).

## Vendor Resources

*   [Check Point Log Exporter Troubleshooting Guide](https://sc1.checkpoint.com/documents/Log_Exporter/EN/Content/Topics/Troubleshooting.htm)
*   [Check Point CheckMates Community - Logging & Reporting](https://community.checkpoint.com/t5/Logging-and-Reporting/bd-p/logging-and-reporting)

# Documentation sites

*   [Check Point Log Exporter Administration Guide](https://sc1.checkpoint.com/documents/Log_Exporter/EN/Content/Topics/Deployment-SmartConsole.htm)
*   [Check Point R81.20 Logging And Monitoring Admin Guide](https://sc1.checkpoint.com/documents/R81.20/WebAdminGuides/EN/CP_R81.20_LoggingAndMonitoring_AdminGuide/Content/Topics-LMG/Log-Exporter.htm)
*   [Elastic Check Point Integration Reference](https://docs.elastic.co/integrations/checkpoint)