# Service Info

The Citrix Web App Firewall (WAF) integration allows you to ingest security event logs and violation data from Citrix ADC (formerly NetScaler) appliances into the Elastic Stack. This provides centralized visibility into web-based threats, such as SQL injection, cross-site scripting (XSS), and buffer overflows, enabling security teams to monitor and respond to attacks targeting their web applications in real-time.

## Common use cases

The Citrix WAF (NetScaler AppFirewall) integration is designed to provide deep visibility into web application security events by centralizing security logs within the Elastic Stack.
- **Threat Detection and Mitigation:** Identify and respond to common web attacks such as SQL injection, Cross-Site Scripting (XSS), and buffer overflows by analyzing structured CEF logs in real-time.
- **Compliance and Auditing:** Maintain a comprehensive audit trail of all security violations and policy enforcement actions to meet regulatory requirements like PCI-DSS, HIPAA, or GDPR.
- **Security Policy Fine-tuning:** Analyze frequent false positives or blocked legitimate traffic to refine AppFirewall signatures and relaxation rules, ensuring optimal security without impacting user experience.
- **Security Posture Monitoring:** Visualize global attack trends, targeted URLs, and attacker geographic distributions through pre-built Kibana dashboards to assess the overall security health of protected applications.

## Data types collected

This integration can collect the following types of data from Citrix NetScaler/WAF devices:
- **Audit Logs:** Records of administrative actions, command executions, and configuration changes within the NetScaler environment.
- **WAF Security Logs:** Detailed events regarding blocked or flagged web requests, including the violation type, target URL, and client information.
- **System Events:** High-level system logs including service restarts, hardware status updates, and high availability (HA) failover events.
- **Authentication Logs:** Events related to user logins and sessions via the NetScaler Gateway or management interface.
- **Data Formats:** Logs are primarily collected via the Syslog protocol in standard BSD or IETF formats, depending on the NetScaler version and configuration.

## Compatibility

This integration is compatible with **Citrix NetScaler ADC (formerly Citrix ADC/Citrix Gateway)**.
- Supported on **NetScaler 12.1, 13.0, 13.1, and 14.x** versions or higher.
- Requires the **Web App Firewall** feature license to be active on the NetScaler appliance.
- Optimized for logs formatted in **Common Event Format (CEF)**.

## Scaling and Performance

- **Transport/Collection Considerations:** For high-volume environments, TCP is recommended over UDP to ensure delivery reliability, as UDP may drop packets during network congestion. If using UDP, ensure the network path between the Citrix appliance and the Elastic Agent has low latency and high bandwidth to minimize data loss.
- **Data Volume Management:** To reduce the volume of logs sent to the Elastic Stack, configure the WAF security profiles to log only "Block" or "Alert" actions rather than all traffic. You can also use the `Expression` field in the Citrix Syslog Policy to filter out specific non-security events or low-priority informational logs before they leave the appliance.
- **Elastic Agent Scaling:** A single Elastic Agent can handle significant syslog throughput, but for large-scale deployments with multiple Citrix appliances, consider deploying a dedicated fleet of Agents behind a load balancer. Ensure the Agent host has sufficient CPU and memory to handle the parsing overhead of the CEF format at high EPS (events per second) rates.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have `superuser` or `nsroot` credentials to the NetScaler ADC management GUI or CLI.
- **Network Connectivity:** Ensure the NetScaler appliance can reach the Elastic Agent host over the network. Port `514` (or your custom configured port) must be open on any intermediary firewalls.
- **Feature Licensing:** The Web Application Firewall feature must be licensed and enabled on the NetScaler appliance to generate WAF-specific logs.
- **Log Source Information:** Identify the IP address or FQDN of the Elastic Agent that will act as the Syslog collector.
- **Time Synchronization:** Ensure that NTP is configured on the NetScaler appliance to ensure accurate timestamps in the security logs.

## Elastic prerequisites

- **Elastic Agent:** An active Elastic Agent must be installed and enrolled in Fleet.
- **Policy Assignment:** The agent must be assigned to a policy that includes the Citrix WAF integration.
- **Inbound Access:** The host running the Elastic Agent must allow inbound traffic on the Syslog port (e.g., TCP/UDP 514).

## Vendor set up steps

### Part 1: Configure the Syslog Server and Policy
1. Log in to your Citrix ADC / NetScaler management interface (GUI).
2. Navigate to **Configuration > System > Auditing > Syslog**.
3. Select the **Servers** tab and click **Add**.
4. Configure the server details:
   - **Name**: `elastic-agent-syslog`
   - **IP Address**: Enter the IP address of the Elastic Agent.
   - **Port**: `514` (or your chosen port).
   - **Transport**: Select `TCP` or `UDP` (matching your Agent configuration).
   - **Log Level**: Select all levels (EMERGENCY to INFORMATIONAL).
5. Click **Create**.
6. Select the **Policies** tab and click **Add**.
7. Configure the policy settings:
   - **Name**: `elastic-agent-policy`
   - **Server**: Select `elastic-agent-syslog`.
   - **Expression**: Enter `true`.
8. Click **Create**.
9. In the Syslog section, click on **Advanced Policy Global Bindings**.
10. Click **Add Binding**, select your policy, set a priority (e.g., `100`), and set the **Bind Point** to `APPFW_GLOBAL`.

### Part 2: Enable CEF Logging Format
1. Open an SSH session to the Citrix ADC CLI.
2. Enter the following command to enable CEF formatted logs for WAF:
   ```shell
   set appfw settings -CEFLogging ON
   ```
3. To verify the configuration, run:
   ```shell
   sh appfw settings | grep CEFLogging
   ```
4. Ensure the output displays `CEFLogging: ON`.

### Part 3: Configure WAF Profiles for Logging
1. In the Citrix ADC GUI, navigate to **Security > Citrix Web App Firewall > Profiles**.
2. Select the profile applied to your application and click **Edit**.
3. In the **Advanced Settings** pane, click on **Security Checks**.
4. Review the list of checks (e.g., HTML SQL Injection, Buffer Overflow).
5. For every check that should be monitored, ensure the **Log** checkbox is enabled.
6. Click **OK** and then **Done** to save the profile changes.
7. Save the overall configuration by clicking the disk icon in the top right of the GUI.

## Kibana set up steps

1. Navigate to **Management > Integrations** in Kibana.
2. Search for **Citrix WAF** and select it.
3. Click **Add Citrix WAF**.
4. Under the **Syslog** configuration section:
   - Set **Syslog Host** to `0.0.0.0` to listen on all interfaces or provide the specific IP of the Agent host.
   - Set **Syslog Port** to `514` (ensure this matches the port configured on the Citrix appliance).
   - Select the **Protocol** (TCP or UDP) that you chose during the vendor setup.
5. Expand **Advanced options** if you need to adjust internal timeouts or queue sizes for high-volume traffic.
6. Scroll down and click **Save and continue**.
7. Select the Agent Policy where you want to deploy the integration.
8. Click **Save and deploy** to push the configuration to your Elastic Agents.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from Citrix WAF to the Elastic Stack.

### 1. Trigger Data Flow on Citrix WAF:
- **Trigger Security Violation:** Attempt to access a protected application URL and append a common attack string such as `' OR 1=1--` to a query parameter. This should trigger a SQL injection violation log.
- **Generate Audit Log:** Log out of the NetScaler management interface and log back in to generate an authentication audit event.
- **Toggle Feature:** Enter configuration mode and run `disable appfw feature` followed by `enable appfw feature` to generate configuration state change logs.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view or the specific integration data view.
3. Enter the following KQL filter: `data_stream.dataset : "citrix_waf.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `citrix_waf.log`)
   - `source.ip` (representing the client IP that triggered the WAF event)
   - `event.action` or `event.outcome` (e.g., `blocked` or `denied`)
   - `citrix_waf.appfw.violation_type` or `citrix_waf.mnemonic`
   - `message` (containing the raw CEF payload from NetScaler)
5. Navigate to **Analytics > Dashboards** and search for "Citrix WAF" to view pre-built visualizations and confirm data is being aggregated.

# Troubleshooting

## Common Configuration Issues

- **CEF Logging Disabled**: If logs appear as unstructured text and are not being parsed into specific fields, ensure the command `set appfw settings -CEFLogging ON` was executed and saved on the Citrix ADC.
- **Incorrect Bind Point**: If no WAF logs are arriving but general system logs are, verify that the Syslog Policy is bound to `APPFW_GLOBAL` specifically. Binding to `SYSTEM_GLOBAL` may not capture all granular WAF violation details in some firmware versions.
- **Network Firewall Blocking Traffic**: If no logs arrive at the Agent, check for firewalls or Network Security Groups (NSGs) between the Citrix ADC SNIP and the Agent. Test connectivity using `nc -zv <Agent_IP> 514`.
- **Port Conflict**: If the Elastic Agent fails to start the integration, check the Agent logs (`elastic-agent.log`) for "address already in use" errors. This indicates another process is already listening on port 514.

## Ingestion Errors

- **Parsing Failures**: Check for the `error.message` field in Kibana. If CEF logs are malformed due to non-standard characters in the URL, the parser may fail. Ensure the NetScaler is using a recent firmware version that adheres to standard CEF formatting.
- **Time Sync Issues**: If logs appear with the wrong timestamp, verify that NTP is configured and synchronized on both the NetScaler appliance and the Elastic Agent host.
- **Truncated Logs**: Syslog over UDP has a maximum packet size. If WAF logs are very large (due to long URLs or cookie headers), they may be truncated. Switch the `syslogAction` and Elastic Agent to use **TCP** to handle larger payloads.

## Vendor Resources

- [Send logs to external syslog server - Citrix Customer Support](https://support.citrix.com/external/article/CTX483235/send-logs-to-external-syslog-server.html)
- [Web App Firewall logs | Web App Firewall - NetScaler Docs](https://docs.netscaler.com/en-us/citrix-adc/current-release/application-firewall/logs.html)
- [How to create message action to log to syslog in Citrix NetScaler](https://support.citrix.com/external/article/CTX200908/how-to-create-message-action-to-log-to-s.html)

## Documentation sites

- [NetScaler Application Firewall Logging Configuration Guide](https://docs.netscaler.com/en-us/citrix-adc/current-release/application-firewall/logs.html)
- [Citrix Support Article CTX483235: Configuring External Syslog](https://support.citrix.com/external/article/CTX483235/send-logs-to-external-syslog-server.html)
- [Citrix Support Article CTX200908: Message Action Log configuration](https://support.citrix.com/external/article/CTX200908/how-to-create-message-action-to-log-to-s.html)
