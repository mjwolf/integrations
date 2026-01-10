# Service Info

## Common use cases

*   **Security Fabric Auditing:** Monitor and audit configuration changes across your Fortinet Security Fabric to ensure compliance and track administrative actions.
*   **Centralized Management Monitoring:** Collect and analyze system event logs from FortiManager to identify management-plane performance issues or unauthorized access attempts.
*   **Policy Change Correlation:** Correlate FortiManager policy deployments with network traffic patterns in Elastic to validate the impact of security rule updates.

## Data types collected

*   **System Event Logs:** Information regarding system restarts, firmware updates, and hardware status.
*   **Administrator Activity Logs:** Records of logins, logouts, and specific configuration changes made by users.
*   **Task Logs:** Details on background processes, scheduled tasks, and device synchronization events.
*   **Audit Logs:** Specific logs related to policy changes and security object modifications.

## Compatibility

*   **FortiManager Versions:** This integration is compatible with FortiManager versions 6.4, 7.0, 7.2, and 7.4.
*   **Elastic Agent:** Version 8.x or later is recommended for full feature support.

## Scaling and Performance

*   **Throughput:** FortiManager logs are typically administrative in nature and produce lower volume compared to security gateways; however, spikes may occur during bulk policy deployments.
*   **Aggregation:** To handle high-volume management environments, ensure the Elastic Agent is hosted on a machine with sufficient CPU to handle concurrent syslog streams if multiple Fortinet devices are forwarding data.

# Set Up Instructions

## Vendor prerequisites

*   **Administrative Access:** A FortiManager account with Super_User or equivalent permissions to modify System Settings.
*   **Network Connectivity:** Firewall rules must allow UDP or TCP traffic on the designated syslog port (typically 514) from the FortiManager IP to the Elastic Agent IP.

## Elastic prerequisites

*   **Elastic Agent:** Must be installed and enrolled in a policy.
*   **Integration Installation:** The Fortinet FortiManager integration must be added to the Elastic Agent policy via Fleet.
*   **Port Configuration:** The integration policy must be configured with a listening port that matches the vendor setup (e.g., 514).

## Vendor set up steps

1.  Log in to the FortiManager UI with administrator privileges.
2.  Navigate to **System Settings > Advanced > Syslog Server**.
3.  Click **Create New**.
4.  Configure the following settings:
    *   **Name**: `elastic-agent`
    *   **IP Address (or FQDN)**: Enter the IP of your Elastic Agent.
    *   **Syslog Server Port**: Enter the port configured in your Elastic Agent integration (e.g., `514`).
    *   **Reliable Connection**: Uncheck for UDP; check for TCP (must match Elastic Agent settings).
5.  Click **OK** to save and begin forwarding logs.
6.  (Optional) Use the CLI to verify settings: `config system syslog` then `show`.

## Kibana set up steps

1.  In Kibana, go to **Management > Integrations**.
2.  Search for and select **Fortinet FortiManager**.
3.  Click **Add Fortinet FortiManager**.
4.  Configure the integration name and select the desired Agent Policy.
5.  Under **Syslog Host**, enter the interface address (e.g., `0.0.0.0`) and the **Syslog Port** chosen during vendor setup.
6.  Click **Save and continue**.

# Validation Steps

1.  **Generate Test Log:** Perform a minor configuration change or log out and back in to FortiManager to generate an event.
2.  **Check Task Monitor:** Navigate to **System Settings > Task Monitor** in FortiManager to ensure system tasks are running.
3.  **Kibana Discover:** Open **Analytics > Discover** and filter for `event.module : "fortinet"` and `event.dataset : "fortinet.fortimanager"`.
4.  **Verify Data Flow:** Ensure that timestamped log entries appear in the results, indicating successful ingestion and parsing.

# Troubleshooting

## Common Configuration Issues

*   **Network Reachability:** Ensure there are no intermediate firewalls or Access Control Lists (ACLs) blocking the syslog port between FortiManager and the Elastic Agent.
*   **Protocol Mismatch:** If "Reliable Connection" is enabled in FortiManager, the Elastic Agent must be configured for TCP; otherwise, use UDP for both.
*   **Source IP Binding:** If FortiManager has multiple interfaces, ensure the management interface used for syslog is the one permitted by your security rules.

## Ingestion Errors

*   **Malformed Headers:** If `error.message` indicates parsing failures, verify that FortiManager is sending logs in the standard syslog format and that no custom log headers are appended.
*   **Time Synchronization:** Inaccurate clocks on either FortiManager or the Elastic Agent host can cause data to appear outside of the expected time range in Kibana.

## API Authentication Errors

*   Not specified (this integration primarily uses Syslog forwarding rather than API polling).

## Vendor Resources

*   [Fortinet Support Portal](https://support.fortinet.com)
*   [Fortinet Community Troubleshooting Tips](https://community.fortinet.com/t5/FortiManager/Troubleshooting-Tip-How-to-troubleshoot-issue-related-to/ta-p/409139)
*   [FortiManager CLI Reference Guide](https://docs.fortinet.com/product/fortimanager/)

# Documentation sites

*   [Fortinet FortiManager Administration Guide](https://docs.fortinet.com/document/fortimanager/7.4.0/administration-guide/)
*   [Fortinet Syslog Configuration Guide](https://docs.fortinet.com/document/fortimanager/7.4.0/administration-guide/155798/syslog-server)
*   [Elastic Fortinet Integration Documentation](https://docs.elastic.co/integrations/fortinet)