# Service Info

## Common use cases

- **Network Access Auditing:** Monitor and report on "Passed Authentications" and "Failed Attempts" to track who is accessing the network and identify potential unauthorized access attempts.
- **Administrative Compliance:** Track "Administrative and Operational Audit" logs to maintain a record of configuration changes and system administration activities for compliance requirements.
- **Security Incident Investigation:** Correlate ISE authentication data with other security events in Elastic to investigate credential-based attacks or lateral movement within the network.

## Data types collected

The Cisco ISE integration collects **Logs** via syslog. This includes authentication events, guest access logs, profiling data, and administrative audit trails.

## Compatibility

This integration is compatible with Cisco Identity Services Engine (ISE) versions **1.x, 2.x, and 3.x**. It relies on the standard BSD and IETF syslog formats supported across most ISE releases.

## Scaling and Performance

Cisco ISE can generate a high volume of syslog messages, particularly in environments with frequent authentications. It is recommended to set the **Maximum Length** to **8192** to prevent message truncation. For high-traffic environments, consider using **TCP Syslog** for reliable delivery or distributing the load across multiple Elastic Agents.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have Super Admin or System Admin privileges in the Cisco ISE Administration Interface.
- **Network Connectivity:** Ensure that the Cisco ISE nodes can reach the Elastic Agent host over the network on the configured UDP/TCP port (typically 514 or 5514).

## Elastic prerequisites

- **Elastic Agent:** An active Elastic Agent must be installed and enrolled in a policy.
- **Integration Configuration:** The Cisco ISE integration must be added to the agent policy with a configured Syslog host and port that matches the "Remote Logging Target" settings in ISE.

## Vendor set up steps

1.  **Log in to your Cisco ISE Administration Interface.**
2.  **Navigate to Remote Logging Targets:** Click the menu icon and go to **Administration > System > Logging > Remote Logging Targets**.
3.  **Create a New Logging Target:** Click **Add** and configure the following:
    *   **Name**: e.g., `elastic-agent-syslog`.
    *   **Target Type**: Select `UDP Syslog` or `TCP Syslog` (must match your Elastic Agent configuration).
    *   **Host / IP Address**: Enter the IP address of the Elastic Agent server.
    *   **Port**: Enter the port number (e.g., `514`).
    *   **Facility Code**: Select an appropriate facility such as `LOCAL6` or `LOCAL7`.
    *   **Maximum Length**: Set to `8192`.
4.  **Save the Target:** Click **Save** and confirm any security warnings regarding unencrypted syslog.
5.  **Assign Categories:** Navigate to **Administration > System > Logging > Logging Categories**.
6.  **Edit Categories:** Select categories like `Passed Authentications` or `Failed Attempts`, move your new target to the **Selected** list, and click **Save**.

## Kibana set up steps

1.  In Kibana, go to **Management > Integrations**.
2.  Search for and select **Cisco ISE**.
3.  Click **Add Cisco ISE**.
4.  Configure the **Syslog host** (e.g., `0.0.0.0`) and **Syslog port** (e.g., `514`) to match the ISE Remote Logging Target settings.
5.  Select the **Existing policy** where your Elastic Agent is enrolled.
6.  Click **Save and continue**.

# Validation Steps

1.  **Trigger an Event:** Perform a test authentication on a device managed by Cisco ISE or log in/out of the ISE admin console.
2.  **Verify Ingestion:** In Kibana, navigate to **Analytics > Discover**.
3.  **Filter Data:** Use the filter `data_stream.dataset : "cisco_ise.log"` to view incoming logs.
4.  **Check Dashboards:** Open the **[Logs Cisco ISE] Overview** dashboard to verify that charts and visualizations are populating with data.

# Troubleshooting

## Common Configuration Issues

- **Port Conflicts:** Ensure the port configured (e.g., 514) is not being used by another service on the Elastic Agent host.
- **Firewall Blocking:** Verify that intermediate firewalls allow traffic from the Cisco ISE IP address to the Elastic Agent IP on the specified UDP/TCP port.
- **Facility Mismatch:** If logs are missing, ensure the Facility Code selected in ISE matches the expectations of your logging policy, though Elastic usually parses all facilities.

## Ingestion Errors

- **Truncated Messages:** If `error.message` indicates parsing failures due to incomplete logs, verify that the **Maximum Length** in the ISE Remote Logging Target is set to `8192`.
- **Mapping Failures:** If fields are not correctly mapped, ensure the ISE syslog format is set to the default "Cisco ISE" or "Default" message template, as custom templates may break the Elastic parser.

## API Authentication Errors

Not specified (This integration primarily uses Syslog; no API authentication is required for log ingestion).

## Vendor Resources

- [Cisco ISE Troubleshooting Guide](https://www.cisco.com/c/en/us/support/security/identity-services-engine/products-troubleshooting-guides-list.html)
- [Cisco Community - ISE Logging Discussion](https://community.cisco.com/t5/security-knowledge-base/tkb-p/4461-kb-security-ise)

# Documentation sites

- [Configure External Syslog Server on ISE - Cisco Official Guide](https://www.cisco.com/c/en/us/support/docs/security/identity-services-engine/222223-configure-external-syslog-server-on-ise.html)
- [Cisco ISE Logging User Guide](https://www.cisco.com/en/US/docs/security/ise/1.0/user_guide/ise10_logging.html)
- [Cisco ISE Product Support Overview](https://www.cisco.com/c/en/us/support/security/identity-services-engine/series.html)