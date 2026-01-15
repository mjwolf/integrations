# Service Info

## Common use cases

The Imperva integration for Elastic allows organizations to ingest security and system logs from both Imperva Cloud WAF (Application Security) and Imperva SecureSphere (On-premise) environments. This integration provides centralized visibility into web application attacks, system health, and administrative activity.
- **Threat Detection and Analysis:** Monitor web traffic for security violations such as SQL injection, Cross-Site Scripting (XSS), and bot activity to identify and mitigate active threats in real-time.
- **Regulatory Compliance:** Maintain a detailed audit trail of security alerts and system events to meet compliance requirements such as PCI-DSS, HIPAA, and GDPR by centralizing logs in the Elastic Stack.
- **Operational Health Monitoring:** Track the health and status of Imperva Gateways and Management Servers (MX) to ensure continuous protection and identify performance bottlenecks.
- **Incident Response:** Provide security analysts with granular details of security alerts, including source IP addresses, attack signatures, and destination URLs, to accelerate investigation and remediation.

## Data types collected

This integration can collect the following types of data:
- **Imperva Cloud WAF Logs:** Detailed logs of web requests, security violations, and traffic statistics processed by the Imperva Cloud Security Console.
- **SecureSphere System Events:** Internal appliance logs including audit logs, system health status, and administrative actions performed on the SecureSphere gateway or manager.
- **Security Alerts:** Specific events triggered by security policies, including the severity, source IP, destination URL, and the type of attack detected.
- **Data Formats:** Events are primarily collected via Syslog in standard formats or specialized formats like LEEF (Log Event Extended Format) depending on the configuration.
- **Communication Protocol:** Transmission occurs over UDP or TCP via the Syslog protocol to the Elastic Agent's listening port.

## Compatibility

The integration is compatible with the following Imperva products:
- **Imperva Cloud WAF** (via the log integration script and configuration file).
- **Imperva SecureSphere** (version 12.x and higher, utilizing Action Sets).

## Scaling and Performance

- **Transport/Collection Considerations:** When configuring syslog collection for Imperva, choose between TCP and UDP based on your requirements. TCP is recommended for reliable delivery to ensure no security events are dropped during network congestion, whereas UDP may be used for higher throughput environments where minor data loss is acceptable.
- **Data Volume Management:** To optimize performance, use Imperva SecureSphere Action Sets to filter events at the source. Instead of sending all system events, configure policies to only forward high-severity alerts or specific violation types (e.g., `security violation–all`) to reduce the processing load on the Elastic Agent and the indexing load on the Elastic Stack.
- **Elastic Agent Scaling:** For high-volume environments processing thousands of events per second (EPS) from multiple Imperva Gateways, deploy multiple Elastic Agents behind a load balancer. Ensure the host machine running the Elastic Agent has sufficient CPU and memory resources to handle the parsing of CEF-formatted logs, which can be computationally intensive.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** High-level administrative credentials for the Imperva Cloud Security Console or the Imperva SecureSphere Management Server (MX) are required to configure log forwarding.
- **Network Connectivity:** Ensure that the Imperva appliances (Gateway/MX) or the Cloud WAF log integration service can reach the Elastic Agent host on the configured Syslog port (e.g., UDP/TCP 514 or 5514).
- **Log Integration Feature:** For Cloud WAF, the Log Integration feature must be enabled within your Imperva subscription to access the log configuration files.
- **IP Information:** Knowledge of the internal/external IP address of the Elastic Agent server to be used as the destination for log forwarding.
- **Format Requirements:** Ensure the Imperva appliance is capable of exporting logs in the CEF (Common Event Format) for optimal parsing by the Elastic integration.

## Elastic prerequisites

- **Elastic Agent:** An Elastic Agent must be installed and enrolled in a policy via Fleet.
- **Integration Package:** The Imperva integration must be added to the Agent's policy in Kibana.
- **Network Access:** Ensure that any firewalls between the Imperva source and the Elastic Agent are configured to allow traffic on the specific port chosen for Syslog ingestion.

## Vendor set up steps

### For Imperva Cloud WAF (Cloud Application Security):
1. Log in to your **Imperva Cloud Security Console**.
2. Navigate to the **Log Setup** page.
3. Locate the link to download the log configuration file, `Settings.Config`, and save it to your local system.
4. Open the `Settings.Config` file in a text editor (e.g., Notepad++, VS Code).
5. Locate the parameter `SYSLOG_ENABLE` and change its value to `YES`.
6. Update the `SYSLOG_ADDRESS` field with the IP address of your Elastic Agent server.
7. Configure the `SYSLOG_PORT` with the port number your Elastic Agent is listening on (default is often 514 or 5514). Note: Verify if your configuration file uses the spelling `SYSLO_PORT` as documented in some versions.
8. Ensure the `APIID` and `APIKEY` parameters are correctly populated with the credentials provided on the Cloud Security Console Log Setup page.
9. Save the changes to `Settings.Config` and ensure the log integration process is running to begin forwarding data.

### For Imperva SecureSphere (On-Premise):
1. Log in to the **Imperva SecureSphere Management (MX)** interface.
2. Navigate to **Policies > Action Sets**.
3. **Configure Security Violations:**
    - Click **New** to create a custom Action Set. Name it (e.g., "Elastic-Security-Violations").
    - Select `security violation–all` for the event type.
    - Under the Action list, add `gateway security system log > gateway log security event to system log (syslog)`.
    - Set the log format to **CEF standard**.
    - Assign this Action Set as a "followed action" to your active Security Policy rules.
4. **Configure Security Alerts (Aggregated Events):**
    - Create another Action Set named "Elastic-Security-Alerts".
    - Select `any event type`.
    - Add the action `server system log > log security event to system log (syslog)`.
    - Ensure the format is set to **CEF standard**.
    - Assign this to the relevant security policy rules.
5. **Configure System Events:**
    - Create a third Action Set named "Elastic-System-Events".
    - Add the action `server system log > log system event to system log (syslog)` using the **CEF standard**.
    - Navigate to **Policies > System Events Policy**, create a new policy, and assign this Action Set to it.
6. Apply changes and ensure the MX/Gateway has connectivity to the Elastic Agent destination.

## Kibana set up steps

### For Imperva Log Input:
1. In Kibana, navigate to **Management > Integrations**.
2. Search for **Imperva** and select the integration.
3. Click **Add Imperva**.
4. Configure the integration settings based on your vendor setup:
    * Select the input type (e.g., **Syslog**).
    * Set the **Syslog Host** to `0.0.0.0` to listen on all interfaces or provide a specific IP.
    * Set the **Syslog Port** to the same port defined in the Imperva DRA console (e.g., `514`).
    * Ensure the **Protocol** (UDP or TCP) matches the vendor configuration.
5. Scroll down to the **Advanced options** if you need to adjust internal timeouts or buffer sizes for high-volume traffic.
6. Select the **Existing host policy** where your Elastic Agent is enrolled.
7. Click **Save and continue**, then click **Add agent to your hosts** if you have not already enrolled one, or simply deploy the updated policy.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from Imperva to the Elastic Stack.

### 1. Trigger Data Flow on Imperva:
- **Send Test Message:** Use the "Send Test Syslog Message" button located in the **Configuration > SIEM Integration** section of the Imperva DRA console.
- **Generate Security Event:** Trigger a manual incident or change the status of an existing incident (e.g., close a low-severity alert) to generate a notification event.
- **Update Configuration:** Modify a minor setting in the DRA console to trigger a configuration audit event, if audit logging is enabled.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view or the specific integration data view.
3. Enter the following KQL filter: `data_stream.dataset : "imperva.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `imperva.log`)
   - `log.syslog.facility.code` (matching the facility selected in the vendor console)
   - `event.action` (e.g., incident-opened, incident-closed)
   - `event.severity` (should match the DRA severity level)
   - `message` (containing the raw JSON payload from Imperva)
5. Navigate to **Analytics > Dashboards** and search for "Imperva" to view pre-built visualizations and confirm data is being aggregated correctly.

# Troubleshooting

## Common Configuration Issues

- **Port Spelling Discrepancy (Cloud WAF)**: In the `Settings.Config` file for Cloud WAF, the port parameter may be listed as `SYSLO_PORT` instead of `SYSLOG_PORT`. If logs are not sending, verify the exact spelling required by your version of the configuration file.
- **Mismatched Protocol**: If the Imperva appliance is configured to send logs via TCP but the Elastic Agent is listening for UDP (or vice versa), logs will not be ingested. Ensure the protocol matches on both ends.
- **Firewall Blockage**: Check if the host firewall on the Elastic Agent server is blocking the configured syslog port. Use `tcpdump -i any port <port_number>` on the Elastic Agent host to verify if packets are arriving from the Imperva appliance.
- **Action Set Assignment**: In SecureSphere, logs may not flow if the Action Set is created but not assigned as a "followed action" to an active policy. Re-verify the policy assignments in the MX interface.

## Ingestion Errors

- **Parsing Failures**: Check the `error.message` field in Kibana. If you see "Provided Grok expressions do not match", it may indicate that the Imperva log format has changed or is not sending valid JSON.
- **Timestamp Mismatches**: Ensure that both the Imperva appliance and the Elastic Agent host are synchronized via NTP. Large clock skews can cause logs to appear in the "future" or "past" in Discover.
- **Field Mapping Conflicts**: If specific Imperva fields are not being indexed, check the `elasticsearch.index.logs.imperva` index templates for mapping errors or type mismatches (e.g., a field expected as a keyword arriving as an integer).

## Vendor Resources

- [Log Configuration File - Imperva](https://docs-cybersec.thalesgroup.com/bundle/cloud-application-security/page/more/log-configuration.htm)
- [Configuring a system event action for Imperva SecureSphere - IBM](https://www.ibm.com/docs/en/dsm?topic=securesphere-configuring-system-event-action-imperva)

## Documentation sites

- [Log Configuration File - Imperva](https://docs-cybersec.thalesgroup.com/bundle/cloud-application-security/page/more/log-configuration.htm)
- [Configuring a system event action for Imperva SecureSphere - IBM](https://www.ibm.com/docs/en/dsm?topic=securesphere-configuring-system-event-action-imperva)
- Refer to the official vendor website for additional resources.
