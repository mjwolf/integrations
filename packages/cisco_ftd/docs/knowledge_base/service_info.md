# Service Info

## Common use cases

*   **Security Threat Detection:** Monitor firewall intrusion events, malware detections, and Security Intelligence hits to identify and respond to active network threats.
*   **VPN and Access Monitoring:** Audit Remote Access and Site-to-Site VPN connections to track user activity, session durations, and authentication attempts.
*   **Network Compliance:** Maintain long-term logs of connection events and administrative changes for regulatory compliance and forensic auditing.

## Data types collected

This integration primarily collects **logs** in syslog format from the Cisco FTD appliance. These include connection events, intrusion (IPS) events, file and malware events, security intelligence events, and system health status messages.

## Compatibility

This integration is compatible with Cisco Firepower Threat Defense (FTD) and Cisco Firepower Management Center (FMC) versions 6.x and 7.x. It supports logs sent via standard Syslog protocols (UDP or TCP).

## Scaling and Performance

For high-traffic environments, it is recommended to use **Rate Limiting** within the FMC platform settings to prevent syslog flooding. Performance is dependent on the configured logging severity; setting logging to "Informational" or "Debug" on high-throughput interfaces can impact the FTD's management plane performance.

# Set Up Instructions

## Vendor prerequisites

*   Administrative access to the Cisco Firepower Management Center (FMC) web interface.
*   Cisco FTD appliances must be successfully registered and "Online" within the FMC.
*   Network connectivity between the FTD management or data interfaces and the Elastic Agent (port 514 by default).

## Elastic prerequisites

*   Elastic Agent must be installed and enrolled in a policy.
*   The **Cisco FTD integration** must be added to the agent policy with the correct listener port and protocol (UDP/TCP) configured.
*   Ensure the host firewall on the Elastic Agent machine allows incoming traffic on the configured syslog port.

## Vendor set up steps

1.  Log in to your **Firepower Management Center (FMC)** web interface.
2.  Navigate to **Devices > Platform Settings**.
3.  Click **New Policy** and select **Threat Defense Settings** to create a new policy, or edit an existing one, and assign it to your target FTD appliance.
4.  In the policy editor, select the **Syslog** tab from the side menu and click **Logging Setup**. Check the **Enable Logging** box.
5.  Select **Syslog Servers** from the side menu and click **Add**.
6.  Configure the syslog server details:
    *   **IP Address**: Select/create a network object for the Elastic Agent host IP.
    *   **Protocol**: Select **TCP** or **UDP** (must match Elastic Agent settings).
    *   **Port**: Enter the listener port (e.g., 514).
    *   **Available Zones**: Select the security zone(s) from which the FTD reaches the Elastic Agent.
7.  Click **OK** to save the server configuration, then click **Save** for the platform policy.
8.  Click **Deploy**, select the target FTD appliance, and click **Deploy** again to push the configuration changes.

## Kibana set up steps

1.  In Kibana, navigate to **Management > Integrations**.
2.  Search for **Cisco FTD** and select the integration.
3.  Click **Add Cisco FTD**.
4.  Configure the integration name and select the **Agent Policy**.
5.  Under **Syslog Host**, enter the IP address the agent should listen on (e.g., `0.0.0.0`) and the **Syslog Port** (e.g., `514`).
6.  Click **Save and continue** to deploy the integration to your agents.

# Validation Steps

1.  **Check Data Stream:** In Kibana, go to **Analytics > Discover** and filter for `event.dataset : "cisco_ftd.log"`.
2.  **Verify Syslog Traffic:** Run `tcpdump -i <interface> port 514` on the Elastic Agent host to confirm packets are arriving from the FTD IP.
3.  **Inspect Dashboards:** Open the **[Logs Cisco FTD] Overview** dashboard in Kibana to ensure visualizations for connection events and threats are populating.
4.  **Test Event:** Trigger a sample event (e.g., an ICMP ping that matches a logged access rule) and verify it appears in the logs within 1-2 minutes.

# Troubleshooting

## Common Configuration Issues

*   **No Data Received:** Ensure the "Enable Logging" checkbox is checked in the FMC Logging Setup. Verify that the correct security zones are selected for the syslog server destination.
*   **Policy Not Deployed:** Changes in FMC do not take effect until a **Deployment** is successfully completed. Check the "Tasks" status in FMC.
*   **Network Path:** Ensure no intermediate firewalls or Access Control Lists (ACLs) are blocking the syslog port (UDP/TCP 514) between the FTD and Elastic Agent.

## Ingestion Errors

*   **Parsing Failures:** If `error.message` indicates parsing issues, verify that the FTD syslog format is set to the default. Custom syslog formats or headers may break the Elastic integration's grok patterns.
*   **Incorrect Timestamp:** Ensure NTP is synchronized across the FTD, FMC, and Elastic Agent host to prevent time-skew issues in the logs.

## API Authentication Errors

*   **Not specified:** This integration uses push-based syslog; no API authentication is required for standard log ingestion. See vendor documentation for FMC API troubleshooting if using automation scripts.

## Vendor Resources

*   [Cisco FTD Logging Configuration Guide](https://www.cisco.com/c/en/us/support/docs/security/firepower-ngfw/200479-Configure-Logging-on-FTD-via-FMC.html)
*   [Cisco Secure Firewall Support Community](https://community.cisco.com/t5/security-firewalls/bd-p/4661-discussions-security-firewalls)
*   [Cisco Syslog Message Reference Guide](https://www.cisco.com/c/en/us/td/docs/security/firepower/Syslogs/fptd_syslog_guide.html)

# Documentation sites

*   [Elastic Cisco FTD Integration Reference](https://www.elastic.co/docs/reference/integrations/cisco_ftd)
*   [Cisco Firepower Management Center Configuration Guide](https://www.cisco.com/c/en/us/support/security/defense-center/products-installation-and-configuration-guides-list.html)
*   [Official Cisco FTD Product Overview](https://www.cisco.com/c/en/us/products/security/firewalls/index.html)