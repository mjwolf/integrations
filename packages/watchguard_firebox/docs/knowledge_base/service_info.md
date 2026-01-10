# Service Info

## Common use cases

- **Security Threat Monitoring**: Analyze logs from the Intrusion Prevention Service (IPS), Gateway AntiVirus (GAV), and Botnet Detection to identify and respond to active threats within the network.
- **Network Traffic Analysis**: Monitor firewall connection logs to visualize traffic patterns, identify bandwidth-intensive applications, and troubleshoot connectivity issues between internal and external networks.
- **Audit and Compliance**: Track administrative access, configuration changes, and system events to maintain a secure audit trail for regulatory compliance requirements such as PCI-DSS or HIPAA.

## Data types collected

- **Logs**: The integration primarily collects Syslog data, including traffic logs (allowed/denied connections), event logs (system status, administrative actions), and alarm logs (security service triggers).

## Compatibility

- **WatchGuard Firebox**: Compatible with all models running Fireware OS v11.x and v12.x.
- **WatchGuard XTM**: Compatible with legacy XTM appliances that support standard syslog output.

## Scaling and Performance

- **Throughput**: Log volume scales with the number of active security services and the total network traffic processed by the Firebox. For high-traffic environments, ensure the Elastic Agent host has sufficient CPU and memory to handle incoming UDP syslog streams without packet loss.
- **Network Latency**: Syslog uses UDP by default, which does not guarantee delivery; ensure low-latency connectivity between the Firebox and the Elastic Agent to minimize data gaps.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access**: Admin credentials for the WatchGuard Fireware Web UI or Policy Manager.
- **Logging Policy**: Individual firewall policies must have logging enabled (e.g., "Send a log message" checked in policy properties) to generate traffic data.
- **Network Connectivity**: The Firebox must be able to reach the Elastic Agent host over the network on the designated syslog port (typically UDP 514).

## Elastic prerequisites

- **Elastic Agent**: An active Elastic Agent must be installed on a host within the network.
- **Integration Policy**: The "WatchGuard Firebox" integration must be added to the Agent's policy via Fleet in Kibana.
- **Port Configuration**: Ensure the host's OS firewall allows incoming traffic on the port specified in the integration settings (default is 514).

## Vendor set up steps

1. Log in to the WatchGuard Fireware Web UI with administrator credentials.
2. Navigate to **System** > **Logging**.
3. Select the **Syslog Server** tab.
4. Check the box for **Send log messages to these syslog servers**.
5. Click **Add** and configure the destination:
   - **IP Address**: The IP of the host running the Elastic Agent.
   - **Port**: `514` (or your custom syslog port).
   - **Log Format**: Select **Syslog**.
   - **Description**: Enter `Elastic-Agent`.
6. Enable the **The syslog header** and **Time stamp** checkboxes to ensure proper log parsing in Elastic.
7. (Optional) Select **The serial number of the device** to distinguish logs in multi-device environments.
8. In **Syslog Settings**, assign facilities (e.g., **Local0** for alarms, **Local1** for traffic).
9. Click **Save** to apply changes.

## Kibana set up steps

1. In Kibana, go to **Management** > **Integrations**.
2. Search for and select **WatchGuard Firebox**.
3. Click **Add WatchGuard Firebox**.
4. Configure the integration settings:
   - **Syslog Host**: Set to `0.0.0.0` to listen on all interfaces or the specific IP of the host.
   - **Syslog Port**: Set to `514` (must match the Firebox configuration).
5. Assign the integration to an existing **Agent Policy** or create a new one.
6. Click **Save and continue**.

# Validation Steps

1. **Verify Log Generation**: On the Firebox, navigate to **Diagnostics** > **Log Messages** to ensure logs are being generated locally.
2. **Check Data Stream**: In Kibana, navigate to **Management** > **Dev Tools** and run:
   `GET _cat/indices/logs-watchguard_firebox.log-*?v`
3. **Inspect Dashboards**: Open the **[Logs WatchGuard Firebox] Overview** dashboard in Kibana to verify that traffic and security data are populating the visualizations.

# Troubleshooting

## Common Configuration Issues

- **Logs not appearing**: Verify that the "Send a log message" option is enabled on your specific Firewall Policies (e.g., the "Outgoing" or "External" policies).
- **Port Conflict**: If the Elastic Agent fails to start the syslog listener, ensure no other service is using UDP port 514 on the host.
- **Firewall Blocking**: Check that the Firebox's own "WatchGuard" policy allows outbound traffic to the Elastic Agent IP on the syslog port.

## Ingestion Errors

- **Parsing Failures**: If logs appear in `error.message`, ensure the "The syslog header" option is enabled in the Firebox Syslog Server settings, as the integration expects a standard RFC-compliant header.
- **Timestamp Mismatches**: Ensure the Firebox and the Elastic Agent host are synced via NTP; large time drifts can cause data to appear outside the current time range in Kibana.

## API Authentication Errors

- **Not applicable**: This integration uses push-based Syslog over UDP/TCP and does not require API authentication with the WatchGuard appliance.

## Vendor Resources

- [WatchGuard Help Center - Configure Syslog Settings](https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/logging/logging_syslog_external_c.html)
- [WatchGuard Knowledge Base](https://techsearch.watchguard.com/)
- [Fireware Integration Guides](https://www.watchguard.com/help/docs/help-center/en-US/Content/Integration-Guides/_intro/fireware-integrations.html)

# Documentation sites

- [Elastic WatchGuard Firebox Integration Docs](https://www.elastic.co/docs/reference/integrations/watchguard_firebox)
- [WatchGuard Fireware Product Documentation](https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/fireware_home.html)
- [Arctic Wolf Configuration Guide (Reference)](https://docs.arcticwolf.com/bundle/m_syslog/page/configure_watchguard_log_forwarding_using_fireware_web_ui.html)