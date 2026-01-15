# Service Info

## Common use cases

The Fortinet FortiManager integration allows organizations to centralize the monitoring and auditing of their Fortinet security fabric management platform. By ingesting local event logs into the Elastic Stack, administrators can gain deep visibility into the operational health and security posture of the management appliance.
- **Audit and Compliance Monitoring:** Track administrative changes to policies, objects, and device configurations to maintain a detailed audit trail for regulatory compliance (e.g., PCI-DSS, HIPAA).
- **Security Incident Detection:** Identify unauthorized access attempts, brute-force attacks against the management interface, and suspicious administrative logins from unusual IP addresses.
- **Operational Troubleshooting:** Monitor system-level events such as task failures, firmware upgrade statuses, and synchronization issues between FortiManager and managed FortiGate devices.
- **Resource Management:** Analyze system health logs to track CPU, memory, and disk utilization over time, ensuring the appliance remains performant during high-load periods.

## Data types collected

This integration can collect the following types of data:
- **Administrative Audit Logs:** Records of user logins, logouts, configuration updates, and script executions performed on the FortiManager platform.
- **System Logs:** Operational events related to FortiManager system services, resource utilization, and internal health checks.
- **Managed Device Security Logs:** Security events forwarded from managed FortiGate devices, including firewall traffic, IPS hits, and malware detections.
- **Managed Device System Logs:** Operational and diagnostic logs from the managed fleet, covering interface status, routing changes, and system reboots.
- **Data Formats:** Logs are primarily collected via Syslog in standard format or Common Event Format (CEF) for enhanced parsing accuracy.

## Compatibility

The integration is compatible with **Fortinet FortiManager** version 7.x and higher. Configuration steps provided are based on the FortiManager 7.4.2 administration guide, but the syslog logic remains consistent across most 6.x and 7.x releases. Ensure the Elastic Agent is running version 8.x or later for optimal parser performance.

## Scaling and Performance

- **Transport/Collection Considerations:** This integration supports both UDP and TCP (referred to as "Reliable Connection" in FortiManager) for log transport. For high-volume environments, UDP provides the lowest latency and overhead but does not guarantee delivery. If data integrity and delivery confirmation are required, especially for audit and compliance logs, TCP should be used. Ensure the Elastic Agent is configured to match the protocol selected in the FortiManager settings.
- **Data Volume Management:** To manage high log volumes, it is recommended to filter logs at the source. Within FortiManager and managed FortiGates, users can select specific event types (e.g., "Security Events" vs "All Sessions") and log levels (e.g., "Information" vs "Warning") to reduce the ingestion load. Over-logging high-frequency traffic sessions can impact both the performance of the FortiManager appliance and the network bandwidth.
- **Elastic Agent Scaling:** For large-scale deployments managing hundreds of FortiGate devices, a single Elastic Agent may become a bottleneck. In such scenarios, deploy multiple Elastic Agents behind a load balancer to distribute the incoming syslog traffic. Ensure that each Elastic Agent host is adequately sized with sufficient CPU and memory to handle the parsing of high-frequency CEF or Syslog messages.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have Super_User or equivalent administrative permissions to access the **System Settings** and the CLI on the FortiManager appliance.
- **Network Connectivity:** The FortiManager appliance must have network reachability to the Elastic Agent host on the configured Syslog port (default is `514`).
- **Firewall Rules:** Ensure that any intermediate firewalls allow traffic on UDP or TCP port 514 between the FortiManager management IP and the Elastic Agent.
- **Appliance License:** Logging features must be active; ensure the appliance is not in a restricted state due to licensing issues.
- **Hostname/IP Knowledge:** You will need the IP address or Fully Qualified Domain Name (FQDN) of the machine where the Elastic Agent is installed.

## Elastic prerequisites

- **Elastic Agent:** An Elastic Agent must be installed and enrolled in a policy via Fleet.
- **Integration Installed:** The Fortinet FortiManager integration must be added to the Elastic Agent policy.
- **Network Visibility:** The host running the Elastic Agent must be reachable by the FortiManager IP address on the configured syslog port.

## Vendor set up steps

### For Web UI Configuration:
1. Log in to the **FortiManager Web UI**.
2. Navigate to **System Settings > Advanced > Syslog Server**.
3. Click **Create New** to define a new destination.
4. In the **Name** field, enter a descriptive name such as `ElasticAgent`.
5. In the **IP Address** field, enter the IP address of the server running your Elastic Agent.
6. Set the **Port** to match your Elastic Agent configuration (e.g., `514`).
7. Configure **Reliable Connection**:
    - Leave **disabled** to use **UDP**.
    - **Enable** this option to use **TCP** (ensure this matches the Elastic Agent input setting).
8. Click **OK** to save the configuration.

### For CLI Configuration:
1. Open the CLI via the Web UI dashboard or an SSH session.
2. Enter the local log configuration context:
   ```sh
   config system locallog syslogd setting
   ```
3. Enable the status and link the syslog server created in the Web UI:
   ```sh
   set status enable
   set syslog-name "ElasticAgent"
   ```
4. Define the log detail level and facility:
   ```sh
   set severity information
   set facility local7
   ```
5. Apply the changes:
   ```sh
   end
   ```
6. Verify the connection by running:
   ```sh
   diagnose test connection syslog "ElasticAgent"
   ```

## Kibana set up steps

1.  In Kibana, navigate to **Management > Integrations**.
2.  Search for **Fortinet FortiManager** and select the integration.
3.  Click **Add Fortinet FortiManager** to add it to an Elastic Agent policy.
4.  In the integration configuration, locate the **Syslog** input type and configure the following:
    *   **Syslog Host**: Set to `0.0.0.0` to listen on all interfaces or provide the specific IP of the Agent host.
    *   **Syslog Port**: Enter the port number configured on the FortiManager (e.g., `514`).
    *   **Protocol**: Select `udp` or `tcp` to match the "Reliable Connection" setting on the appliance.
5.  Check the **Advanced options** if you need to adjust internal queue sizes or worker threads for high-volume environments.
6.  Click **Save and Continue** to apply the configuration to the Agent policy.
7.  Monitor the **Fleet > Agents** tab to ensure the policy is successfully deployed to the agent.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from Fortinet FortiManager to the Elastic Stack.

### 1. Trigger Data Flow on Fortinet FortiManager:
- **Generate authentication event:** Log out of the current FortiManager web session and log back in. This will trigger a `login` and `logout` event.
- **Generate configuration event:** Navigate to **System Settings > Network** and update a description field or setting, then save the changes.
- **Force a test event:** Use the CLI command `diagnose test connection syslog "Your-Server-Name"` to send a test packet to the Elastic Agent.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the following KQL filter: `data_stream.dataset : "fortinet_fortimanager.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset`: Should be exactly `fortinet_fortimanager.log`.
   - `source.ip`: Should show the IP of the FortiManager appliance.
   - `event.action`: Should show actions like `login`, `logout`, or `update`.
   - `fortinet_fortimanager.logid`: Verify the vendor-specific log ID is present.
   - `message`: Ensure the raw syslog payload is visible and correctly parsed.
5. Navigate to **Analytics > Dashboards** and search for "Fortinet FortiManager" to view the pre-built overview dashboard.

# Troubleshooting

## Common Configuration Issues

- **Syslog Name Mismatch**: If the `syslog-name` set in the CLI does not exactly match the `Name` field in the GUI Syslog Server settings, the appliance will fail to forward logs. Re-verify both settings for typos or case sensitivity.
- **Port Conflict**: If another service on the Elastic Agent host is already using port 514, the Agent will fail to start the syslog listener. Use `netstat -tuln | grep 514` on the host to check for conflicts.
- **Incorrect Severity Level**: If logs are not appearing, check the CLI setting `set severity`. If it is set to `emergency`, common events like logins will be ignored. Set it to `information` for broader visibility.
- **Firewall Obstruction**: The FortiManager might show a "Success" test connection, but the host OS firewall (iptables/firewalld) might still be dropping packets. Check the host firewall logs.

## Ingestion Errors

- **Parsing Failures**: If logs appear in Discover but have the `_grokparsefailure` tag, check if the FortiManager log format has been customized or if an unsupported version is being used.
- **Missing Fields**: If standard ECS fields like `source.ip` are missing, verify that the syslog header is in the expected RFC 3164 or RFC 5424 format.
- **Identifying Issues in Kibana**: Search for `tags : "_grokparsefailure"` or check the `error.message` field in the `logs-fortinet_fortimanager.log-*` data stream to see specific regex or mapping errors.

## Vendor Resources

- [Log Forwarding | FortiManager 7.4.0 | Fortinet Document Library](https://docs.fortinet.com/document/fortimanager/7.4.0/fortiaiops-1-1-1-user-guide/46437/log-forwarding)
- [Technical tip: Configure FortiManager to send logs to a syslog server - Fortinet Community](https://community.fortinet.com/t5/FortiManager/Technical-tip-Configure-FortiManager-to-send-logs-to-a-syslog/ta-p/191412)

## Documentation sites

- [Log Forwarding | FortiManager 7.4.0 | Fortinet Document Library](https://docs.fortinet.com/document/fortimanager/7.4.0/fortiaiops-1-1-1-user-guide/46437/log-forwarding)
- [Technical tip: Configure FortiManager to send logs to a syslog server - Fortinet Community](https://community.fortinet.com/t5/FortiManager/Technical-tip-Configure-FortiManager-to-send-logs-to-a-syslog/ta-p/191412)
