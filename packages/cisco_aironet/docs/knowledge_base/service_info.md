# Service Info

## Common use cases

*   **Wireless Connectivity Troubleshooting**: Monitor and analyze client association, authentication, and roaming events across Cisco Aironet Access Points to identify coverage gaps or connection failures.
*   **Security Auditing**: Track administrative changes on the Cisco Catalyst 9800 WLC and monitor for unauthorized access attempts or suspicious wireless activity.
*   **Operational Health Monitoring**: Centralize system logs from multiple distributed Access Points to proactively identify hardware failures, radio interference, or firmware issues.

## Data types collected

*   **Logs**: System logs (Syslog) from both the Cisco Catalyst 9800 Series Wireless LAN Controller (WLC) and managed Cisco Aironet Access Points.
*   **Events**: Security events, client state changes, and system status notifications.

## Compatibility

*   **Cisco Catalyst 9800 Series Wireless LAN Controllers**: Versions 16.x, 17.x (tested up to 17.17).
*   **Cisco Aironet Access Points**: All models managed by a Catalyst 9800 Series WLC via AP Join Profiles.

## Scaling and Performance

The integration performance depends on the syslog severity level configured on the WLC and APs. Using **Informational** (Level 6) provides a high volume of data; for high-density environments with thousands of APs, ensure the Elastic Agent host is provisioned with sufficient network throughput and CPU to handle the UDP/TCP syslog stream.

# Set Up Instructions

## Vendor prerequisites

*   Administrative access to the Cisco Catalyst 9800 Series WLC web interface.
*   Network connectivity between the WLC/APs and the Elastic Agent (typically UDP port 514 or a custom configured port).
*   A managed AP Join Profile already created or the use of the `default-ap-profile`.

## Elastic prerequisites

*   Elastic Agent installed and enrolled in a policy.
*   The Cisco Aironet or Cisco WLC integration added to the Elastic Agent policy.
*   Syslog input listener configured on the Elastic Agent (matching the port and protocol used in the vendor setup).

## Vendor set up steps

### Part 1: Configure the Controller to Send Logs
1.  Log in to the **Cisco Catalyst 9800 Series WLC** web interface.
2.  Navigate to **Troubleshooting > Logs**.
3.  Click **Manage Syslog Servers**.
4.  Under **IP Configuration**, click **Add** and enter the Elastic Agent server IP address or FQDN.
5.  Under **Log Level Settings**, set the **Syslog** severity level to **Informational** (Level 6).
6.  Click **Apply to Device**.

### Part 2: Configure Access Points (APs) to Send Logs
1.  Navigate to **Configuration > Tags & Profiles > AP Join**.
2.  Select the relevant **AP Join Profile** (e.g., `default-ap-profile`).
3.  Go to the **Management** tab, then the **Device** sub-tab.
4.  In the **System Log** section, enter the **Host IPv4/IPv6 Address** of the Elastic Agent.
5.  Select **local7** for the **Facility Value** and set **Log Trap Value** to **Informational**.
6.  Click **Update & Apply to Device**.

## Kibana set up steps

1.  In Kibana, navigate to **Management > Integrations**.
2.  Search for and select the **Cisco Aironet** (or relevant Cisco Wireless) integration.
3.  Click **Add Cisco Aironet**.
4.  Configure the integration settings, ensuring the **Syslog Host** and **Port** match the WLC configuration.
5.  Select the **Existing Host Policy** where your Elastic Agent is assigned.
6.  Click **Save and continue**.

# Validation Steps

1.  In Kibana, go to **Analytics > Discover**.
2.  Filter for `event.dataset : "cisco_aironet.log"` or search for the WLC's IP address.
3.  Trigger a test event by logging in/out of the WLC or toggling an AP's administrative status.
4.  Verify that fields such as `message`, `log.level`, and `cisco.wlc.facility` are correctly populated.
5.  If available, open the **[Logs Cisco Aironet] Overview** dashboard to view visualized data.

# Troubleshooting

## Common Configuration Issues

*   **Connectivity**: Ensure that firewalls between the WLC/APs and the Elastic Agent allow traffic on the configured syslog port (usually UDP 514).
*   **Profile Application**: If AP logs are missing, verify that the modified AP Join Profile is actually assigned to the active APs under **Configuration > Tags & Profiles > Tags**.
*   **Facility Mismatch**: Ensure the syslog facility (e.g., local7) matches between the vendor device and the integration settings if specific filtering is used.

## Ingestion Errors

*   **Parsing Failures**: If `error.message` appears, it often indicates a non-standard syslog format. Verify that the WLC is not using a custom log format that deviates from standard RFC 3164 or RFC 5424.
*   **Timestamp Issues**: Ensure NTP is configured on both the Cisco WLC and the Elastic Agent host to prevent time drift and indexing delays.

## API Authentication Errors

*   **Not specified**: This integration primarily uses Syslog (push mechanism) and does not typically require API authentication for standard log ingestion.

## Vendor Resources

*   [Cisco Catalyst 9800 Troubleshooting - Quick Start Guide](https://www.cisco.com/c/en/us/support/docs/wireless/catalyst-9800-series-wireless-controllers/215523-quick-start-guide-on-what-logs-and-debug.html)
*   [Cisco Wireless Support Community](https://community.cisco.com/t5/wireless/ct-p/4441-wireless)

# Documentation sites

*   [Cisco Catalyst 9800 Series Configuration Guide - Enabling Syslog](https://www.cisco.com/c/en/us/td/docs/wireless/controller/9800/17-17/config-guide/b_wl_17_17_cg/m_syslog_server.html)
*   [Cisco Aironet AP Software Guide - Appendix: Event Log Messages](https://www.cisco.com/en/US/docs/wireless/access_point/350/configuration/guide/ap350axc_ps458_TSD_Products_Configuration_Guide_Chapter.html)
*   [Elastic Integration Documentation for Cisco](https://docs.elastic.co/integrations/cisco_aironet)