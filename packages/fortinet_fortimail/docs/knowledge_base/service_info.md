# Service Info

## Common use cases

*   **Threat Detection and Prevention:** Monitor AntiVirus and AntiSpam logs to detect malicious email activity and block phishing or malware attempts in real-time.
*   **Mail Flow Auditing:** Track SMTP history and delivery status to ensure compliance with data retention policies and troubleshoot delivery failures.
*   **System Health Monitoring:** Analyze system events and administrative logs to monitor the health of the FortiMail appliance and detect unauthorized configuration changes.

## Data types collected

*   **Logs:** System events, SMTP history, AntiVirus detections, and AntiSpam filtering events formatted as Syslog data.

## Compatibility

*   **FortiMail:** Version 7.6.1 and later are officially supported. This integration is generally compatible with any FortiMail version capable of exporting standard Syslog data.

## Scaling and Performance

*   Not specified. See vendor documentation for hardware-specific logging throughput and appliance performance limits.

# Set Up Instructions

## Vendor prerequisites

*   Administrative access to the FortiMail web UI.
*   An active FortiMail license with security features enabled to generate AntiSpam and AntiVirus logs.
*   Network connectivity allowing the FortiMail appliance to reach the Elastic Agent host over the configured Syslog port.

## Elastic prerequisites

*   Elastic Agent must be installed and enrolled in an active policy.
*   The Fortinet FortiMail integration must be added to the Elastic Agent policy.
*   The host running the Elastic Agent must have the specified Syslog port (e.g., 514) open to receive traffic from the FortiMail IP address.

## Vendor set up steps

1.  Log in to the FortiMail web UI.
2.  Navigate to **Log & Report > Log Setting > Remote**.
3.  Click **New** to create a new remote logging profile.
4.  Configure the following settings in the dialog:
    *   **Status**: Select the checkbox to enable the profile.
    *   **Name**: Enter a descriptive name (e.g., `ElasticAgentSyslog`).
    *   **Server name/IP**: Enter the IP address or FQDN of the Elastic Agent host.
    *   **Server port**: Enter the port number configured in the Elastic integration (e.g., 514).
    *   **Protocol**: Select **Syslog**.
    *   **Mode**: Select **UDP** or **TCP**. (TCP or TCP over TLS is recommended for reliability).
    *   **Level**: Select **Information** or **Notification** to ensure comprehensive data collection.
    *   **Facility**: Select a facility identifier (e.g., `local7`).
    *   **CSV format**: Ensure this is **disabled** to maintain standard Syslog format.
5.  In the **Logging Policy Configuration** section, select the log categories to export: **History**, **AntiVirus**, **AntiSpam**, and **System Event**.
6.  Click **Create** to save and activate the profile.

## Kibana set up steps

1.  In Kibana, navigate to **Management > Integrations**.
2.  Search for **Fortinet FortiMail** and select the integration.
3.  Click **Add Fortinet FortiMail**.
4.  Configure the integration settings:
    *   **Syslog Host**: Set to `0.0.0.0` to listen on all interfaces or provide a specific IP.
    *   **Syslog Port**: Ensure this matches the port configured on the FortiMail appliance (default is `9528` for the integration, change if using `514`).
    *   **Protocol**: Select the protocol (UDP or TCP) matching the FortiMail configuration.
5.  Under **Advanced options**, ensure the internal timezone settings match your FortiMail appliance.
6.  Click **Save and continue** to deploy the changes to the Elastic Agent.

# Validation Steps

1.  Generate a log event on the FortiMail appliance, such as sending a test email or logging out/in to create a system event.
2.  In Kibana, go to **Discover**.
3.  Filter the data using `data_stream.dataset : "fortinet_fortimail.log"`.
4.  Verify that events are appearing and that fields such as `fortinet.fortimail.action`, `source.ip`, and `event.category` are correctly populated.

# Troubleshooting

## Common Configuration Issues

*   **Connectivity Blocked:** Ensure that intermediate firewalls or the Elastic Agent host's local firewall are not blocking the configured Syslog port (UDP/TCP).
*   **Protocol Mismatch:** If logs are not appearing, verify that the **Mode** in FortiMail (UDP/TCP) exactly matches the **Protocol** selected in the Kibana integration settings.
*   **Port Conflicts:** Ensure no other service is bound to the Syslog port on the Elastic Agent host.

## Ingestion Errors

*   **Parsing Failures:** If `error.message` indicates a parsing failure, verify that **CSV format** is disabled in the FortiMail remote logging profile.
*   **Timestamp Mismatch:** If logs appear with incorrect timestamps, check the timezone settings in both the FortiMail appliance and the integration configuration.

## API Authentication Errors

*   Not specified. This integration primarily uses Syslog and does not typically require API credentials for standard log ingestion.

## Vendor Resources

*   [FortiMail Troubleshooting Guide](https://docs.fortinet.com/document/fortimail/7.6.1/administration-guide/332364/configuring-logging)
*   [Fortinet Support Portal](https://support.fortinet.com/)

# Documentation sites

*   [FortiMail 7.6.1 Administration Guide](https://docs.fortinet.com/document/fortimail/7.6.1/administration-guide/332364/configuring-logging)
*   [Elastic Fortinet FortiMail Integration Docs](https://docs.elastic.co/en/integrations/fortinet_fortimail)
*   [Trellix FortiMail Configuration Reference](https://docs.trellix.com/bundle/enterprise-security-manager-data-source-configuration-guide/page/UUID-327ad1e3-dbde-eae9-81bd-467b9443718b.html)