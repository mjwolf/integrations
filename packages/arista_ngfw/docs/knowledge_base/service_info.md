# Service Info

## Common use cases

The Arista NG Firewall (formerly Untangle) integration allows organizations to ingest high-fidelity security and network event data into the Elastic Stack for centralized monitoring and threat hunting. By collecting data from various security modules, administrators can gain deep visibility into network traffic patterns and security posture.
- **Threat Detection and Response:** Monitor logs from the Intrusion Prevention System (IPS) to identify and block malicious activity or exploit attempts in real-time.
- **Content Filtering Compliance:** Analyze Web Filter events to ensure employees are adhering to corporate browsing policies and to identify potential data exfiltration attempts through unauthorized web categories.
- **Network Traffic Auditing:** Utilize Firewall and Application Control events to audit session data, providing a clear trail of who accessed what resources and when.
- **System Health and Audit:** Track administrative changes and system-level events to maintain an audit trail for compliance and to troubleshoot configuration issues within the firewall appliance.

## Data types collected

This integration can collect the following types of data:
- **Firewall Logs:** Connection details, blocked/allowed traffic events, and session information forwarded via syslog.
- **Intrusion Prevention Events:** Detailed alerts regarding detected exploits, signature matches, and blocked attack vectors.
- **Authentication Logs:** User login and logout events (uvm.LoginEvent) including ingress authentication attempts.
- **Web Filtering Logs:** Metadata regarding blocked or flagged URLs, categories, and HTTP request details (https.HttpRequestEvent).
- **Format:** Data is collected in Syslog format and parsed into ECS-compatible (Elastic Common Schema) events.
- **Data Streams:** Log data is typically routed to the `logs-arista_ngfw.log` data stream.

## Compatibility

This integration is compatible with **Arista NGFW** (formerly known as Untangle NG Firewall). It supports versions that include the remote syslog export feature, typically version 16.x and higher. The integration leverages standard Syslog protocols (UDP/TCP) for broad compatibility across physical and virtual appliance deployments.

## Scaling and Performance

- **Transport/Collection Considerations:** The integration supports both UDP and TCP for syslog transport. UDP is recommended for high-volume environments where low latency is prioritized and occasional packet loss is acceptable. TCP is recommended for environments requiring guaranteed delivery of security events, though it may introduce overhead on the firewall appliance during periods of network congestion.
- **Data Volume Management:** To prevent performance degradation on the Arista NG Firewall, it is critical to use "Event Rules" rather than the default "Forward All" rule. Users should filter data at the source by selecting only essential log classes (e.g., `IntrusionPreventionEvents`) and applying specific field filters to exclude high-volume, low-value events like DNS lookups or repetitive session updates.
- **Elastic Agent Scaling:** A single Elastic Agent can typically handle several thousand events per second (EPS) from Arista NGFW. For high-throughput environments or large-scale distributed branch deployments, it is recommended to deploy multiple Elastic Agents behind a load balancer or use dedicated Agent instances for different log classes to distribute the processing load and ensure high availability.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have "Full Administration" privileges on the Arista NG Firewall interface to modify Event and Syslog settings.
- **Network Connectivity:** The firewall must be able to reach the Elastic Agent host over the network. Ensure that any intermediate firewalls allow traffic on the configured syslog port (typically UDP/TCP 514 or 5514).
- **License Requirements:** Remote syslog capabilities are generally included in the platform, but specific application logs (like Web Filter or IPS) require those specific apps to be installed and licensed on the Arista device.
- **Infrastructure Info:** You will need the static IP address or FQDN of the machine hosting the Elastic Agent.

## Elastic prerequisites

- **Elastic Agent:** An Elastic Agent must be installed and enrolled in a policy via Fleet.
- **Integration Package:** The Arista NGFW integration must be added to the Agent policy.
- **Connectivity:** The Elastic Agent host must be configured to listen on the specific protocol (TCP/UDP) and port defined in the Kibana integration settings.

## Vendor set up steps

### For Syslog Collection:

1.  Log in to the **Arista NG Firewall** (Untangle) administration web interface using your administrator credentials.
2.  In the top navigation menu, go to **Config** and select **Events** from the sidebar, then click on the **Syslog** tab.
3.  Locate the **Enable Remote Syslog** checkbox and ensure it is checked to activate the syslog forwarding engine.
4.  In the **Connection Settings** section, configure the destination for your logs:
    *   **IP Address**: Enter the internal IP address of the server where your Elastic Agent is currently running.
    *   **Port**: Enter the port number you have configured (or plan to configure) in the Elastic Agent integration settings (Default: `514`).
    *   **Protocol**: Select either **UDP** or **TCP**. Ensure this matches the protocol selected in the Kibana configuration later.
5.  **CRITICAL PERFORMANCE STEP**: Under the "Syslog Rules" section, the default configuration often includes a rule that sends all system events. It is strongly recommended to **uncheck** or **Delete** the default rule to prevent performance degradation and excessive data ingestion.
6.  Click the **Add** button to create a refined rule for Elastic.
7.  In the rule creation dialog, enter a **Name** (e.g., `Elastic_Agent_Forwarding`).
8.  Use the **Class** dropdown to select the specific event types required for your security monitoring. You should add a separate rule or multiple selections for:
    *   `firewall.FirewallEvent`
    *   `uvm.LoginEvent`
    *   `https.HttpRequestEvent`
    *   `web-filter.WebFilterEvent`
    *   `intrusion-prevention.IntrusionPreventionEvent`
9.  Leave the **Field** and **Condition** sections empty if you wish to forward all events within that class, or refine them further if you only want specific sub-events.
10. Click **Done** to close the rule dialog, then click the **Save** button at the bottom of the main Syslog page to commit and apply the changes to the system.

## Kibana set up steps

### For Syslog Input:

1.  In Kibana, navigate to **Management > Integrations**.
2.  Search for **Arista NG Firewall** and select it.
3.  Click **Add Arista NG Firewall**.
4.  Configure the integration settings:
    *   **Syslog Host**: Set this to `0.0.0.0` to listen on all available interfaces or specify the internal IP of the Agent host.
    *   **Syslog Port**: Enter the port number you configured in the Arista interface (e.g., `514`).
    *   **Protocol**: Select the protocol (TCP or UDP) that matches your vendor configuration.
5.  Under **Advanced options**, ensure the internal timezone is set correctly if your firewall does not provide UTC timestamps.
6.  Click **Save and continue**.
7.  Select the **Agent Policy** that is applied to your Elastic Agent and click **Save and deploy changes**.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from Arista NG Firewall to the Elastic Stack.

### 1. Trigger Data Flow on Arista NG Firewall:
- **Generate Firewall Traffic:** Use a client device behind the firewall to browse to a website or initiate a connection that matches a firewall rule, creating `FirewallEvents`.
- **Generate Security Event:** If the Intrusion Prevention module is active, perform a safe network scan (like `nmap`) against the firewall's internal interface to trigger an `IntrusionPreventionEvents` log.
- **Trigger Configuration Log:** Log out of the Arista administrative interface and log back in, or change a minor setting (like a rule description) and click **Save** to generate system audit events.
- **Web Filter Event:** Attempt to access a URL that falls under a categorized block list (if configured) to generate a `WebFilterEvents` entry.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view or the specific integration data view.
3. Enter the following KQL filter: `data_stream.dataset : "arista_ngfw.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `arista_ngfw.log`)
   - `source.ip` and/or `destination.ip` (populated for firewall/IPS events)
   - `event.action` (e.g., `allowed`, `blocked`, or `detected`)
   - `arista_ngfw.class` (confirming the specific log class like `WebFilterEvents`)
   - `message` (containing the raw log payload from the Arista device)
5. Navigate to **Analytics > Dashboards** and search for "Arista NGFW" to view pre-built visualizations and confirm that charts are populating with real-time data.

# Troubleshooting

## Common Configuration Issues

- **Logs Not Arriving at Agent**: Verify that the Arista NGFW can reach the Elastic Agent host over the network. Use `ping` or `nc -v [Agent_IP] 514` from a troubleshooting tool on the same network to check connectivity. Ensure the "Enable Remote Syslog" checkbox is active in the Arista UI.
- **Protocol Mismatch**: If Arista is set to UDP but the Elastic Agent is listening on TCP (or vice versa), no data will be processed. Ensure both sides are configured with the identical protocol and port.
- **Local Firewall Blocking Port**: Check the local firewall on the Elastic Agent host (e.g., `ufw` or Windows Firewall). It must explicitly allow inbound traffic on the configured Syslog port (default 514).
- **Syslog Rules Too Restrictive**: If specific rules were created in the Arista "Syslog Rules" section, ensure they are not accidentally filtering out the events you are trying to collect. Try creating a "catch-all" rule for testing purposes.

## Ingestion Errors

- **Parsing Failures**: If logs appear in Kibana but contain the tag `_grokparsefailure`, the log format coming from the Arista device may differ from the expected syslog format. Check the `error.message` field in Discover to see specific regex failures.
- **Timestamp Mismatches**: If logs appear to be missing, check the "Time Picker" in Kibana. A mismatch between the Arista appliance's system clock and the Elastic Stack's UTC time can cause logs to appear in the "future" or "past." Ensure NTP is configured on the Arista appliance.
- **Truncated Messages**: If log messages appear cut off, verify the maximum message length settings in the Elastic Agent syslog input configuration. Large Web Filter or IPS logs may exceed the default buffer size.

## Vendor Resources

- [How to create syslog event rules - Arista Networks](https://support.edge.arista.com/hc/en-us/articles/115012950828-How-to-create-syslog-event-rules)
- [Arista Next Generation Firewall | SIEM Documentation - Rapid7](https://docs.rapid7.com/insightidr/arista/)

# Documentation sites

- [How to create syslog event rules - Arista Networks](https://support.edge.arista.com/hc/en-us/articles/115012950828-How-to-create-syslog-event-rules)
- [Arista Next Generation Firewall | SIEM Documentation - Rapid7](https://docs.rapid7.com/insightidr/arista/)
