# Service Info

## Common use cases

The Arista Edge Threat Management (NGFW) integration allows organizations to centralize security telemetry from their perimeter defenses into the Elastic Stack. By ingesting these logs, security teams can gain deep visibility into network traffic patterns and potential threats across their infrastructure.

-   **Threat Detection and Response:** Monitor `ThreatEvent` and `IntrusionPreventionEvent` classes to identify and block malicious activities, such as SQL injection, cross-site scripting, or malware communication. Use Elastic Security's pre-built detection rules to alert on these events in real-time.
-   **Network Traffic Analysis:** Use `SessionEvent` data to analyze bandwidth usage across the network. Identify "top talkers," monitor application-layer traffic (Layer 7), and ensure compliance with corporate usage policies by visualizing traffic by category.
-   **User Activity Auditing:** Track user authentication events and web filtering logs to audit internet access. This is critical for identifying unauthorized access attempts and generating reports for regulatory compliance such as HIPAA, GDPR, or PCI-DSS.
-   **Operational Health Monitoring:** Collect system-level events and `SystemEvent` classes to monitor the health of the Arista NGFW appliance. Track CPU utilization, memory usage, and service status to ensure that security services are running optimally and hardware resources are not over-provisioned.
-   **Application Control Visibility:** Monitor `ApplicationControlEvent` logs to see which applications (e.g., social media, P2P, streaming) are being used within the network and apply granular policies based on actual usage data.
-   **Policy Validation:** Review `FirewallEvent` data to verify that access control lists (ACLs) are functioning as intended and to identify overly permissive rules that may need tightening.

## Data types collected

This integration collects logs via the **Syslog** input. The data is primarily stored in the `arista_ngfw.log` datastream, which provides comprehensive coverage of the following data types:

-   **Firewall and Session Logs (`SessionEvent`):** Detailed records of allowed and blocked connections, including:
    -   `source.ip` and `destination.ip`: IP addresses involved in the communication.
    -   `source.port` and `destination.port`: Port numbers for the session.
    -   `network.transport`: Protocol information (e.g., `tcp`, `udp`).
    -   `arista_ngfw.event_class`: Categorization of the specific event type.
-   **Security Events (`ThreatEvent`, `IntrusionPreventionEvent`):** Logs from security modules such as Virus Blocker, Intrusion Prevention (IPS), and Threat Prevention. These include:
    -   `event.action`: Action taken (e.g., `blocked`, `flagged`).
    -   `rule.name`: The specific security rule or signature ID triggered.
    -   `threat.indicator.name`: Names for identified malware or network attacks.
-   **Web and Content Filtering (`WebFilterEvent`):** Records of URLs visited and blocked content based on policy, including:
    -   `url.full` and `url.domain`: Complete web address and destination domain.
    -   `http.response.status_code`: The HTTP response from the web server.
    -   `arista_ngfw.category`: Content category (e.g., "Social Networking", "Malware").
-   **Application Control (`ApplicationControlEvent`):** Visibility into Layer 7 application usage, mapping traffic to specific applications like BitTorrent, Facebook, or Office 365.
-   **Administrative Audit Logs:** Records of configuration changes, administrator logins, and system updates to maintain an audit trail of appliance management.
-   **Data Format:** All data is collected via the **Syslog** protocol (UDP or TCP). The integration parses these raw strings into **Elastic Common Schema (ECS)** compliant fields, ensuring compatibility with the Elastic Security and Observability apps.

## Compatibility

This integration is compatible with **Arista Edge Threat Management** (formerly **Untangle NG Firewall**) versions **15.x, 16.x, and 17.x**.

-   **Deployment Models:** Supports hardware appliances (z-Series), virtual machines (ESXi, Hyper-V, Proxmox), and public cloud instances (AWS, Azure).
-   **Elastic Requirements:** This integration requires **Elastic Stack version 8.12.0** or higher for full support of the latest ECS mappings and dashboard visualizations.

## Scaling and Performance

-   **Event Filtering at Source:** To maintain high performance, use the **Syslog Rules** in Arista NGFW to forward only essential event classes (e.g., `ThreatEvent`, `SessionEvent`) rather than selecting `All`. Forwarding all events can cause significant CPU overhead on the appliance during peak traffic.
-   **Load Balancing:** In high-volume environments (over 5,000 events per second), point the Arista NGFW syslog output to a load balancer (e.g., HAProxy or F5) that distributes traffic across multiple Elastic Agents.
-   **Protocol Choice:**
    -   **TCP:** Recommended for reliable delivery and environments where log integrity is critical. Ensure the Elastic Agent is configured to handle the expected number of concurrent connections.
    -   **UDP:** Recommended for lower latency and reduced overhead on the firewall appliance, though packets may be dropped during network congestion.
-   **Resource Allocation:** Ensure the host running the Elastic Agent has sufficient CPU and memory to handle the parsing load. A typical rule of thumb is 1 CPU core per 5,000 events per second (EPS) for parsing.

# Set Up Instructions

## Vendor prerequisites

-   **Administrative Access:** You must have an account with **Full Administrator** privileges to access the **Config > Events** menu.
-   **Network Connectivity:** The Arista appliance must have a clear network path to the Elastic Agent's IP address.
-   **Required Licenses:** Ensure that the specific apps (e.g., Threat Prevention, Web Filter) are licensed and enabled; otherwise, they will not generate the logs required by this integration.
-   **Firewall Access Rules:** If the Elastic Agent is located in a different zone (e.g., a Management VLAN), ensure an **Access Rule** exists in Arista NGFW to allow traffic from the "Local" or "Internal" interface to the destination port on the Elastic Agent.

## Elastic prerequisites

-   **Elastic Agent:** An Elastic Agent must be installed and enrolled in a policy via Fleet.
-   **Elastic Stack Version:** Version **8.12.0** or higher is required.
-   **Connectivity:** Ensure the host firewall (e.g., `iptables` or Windows Firewall) on the Elastic Agent machine allows inbound traffic on the selected Syslog port.

## Vendor set up steps

1.  Log in to the **Arista Edge Threat Management** (Untangle) administration interface.
2.  Navigate to **Config** > **Events** and click on the **Syslog** tab.
3.  In the **Syslog Server** section, configure the destination for the logs:
    -   **Host**: Enter the **IP address** of the server hosting the Elastic Agent.
    -   **Port**: Enter the port number (e.g., `514` or a custom port like `9514`).
    -   **Protocol**: Select **UDP** or **TCP** (this must match the setting in Kibana).
4.  Navigate to the **Syslog Rules** section and click **Add**.
5.  Define the rules for forwarding data:
    -   **Enable**: Ensure the checkbox is checked.
    -   **Class**: Select a class from the dropdown (e.g., `SessionEvent`).
    -   **Remote Syslog**: **This checkbox must be checked** to send the data to the Elastic Agent.
    -   **Description**: Enter a name (e.g., "Forward Sessions to Elastic").
6.  Click **Done** to close the rule editor.
7.  Repeat steps 4-6 for other important classes: `ThreatEvent`, `IntrusionPreventionEvent`, `WebFilterEvent`, and `ApplicationControlEvent`.
8.  Click **Save** in the bottom right corner of the page to apply the changes.

## Kibana set up steps

1.  In Kibana, navigate to **Management** > **Integrations**.
2.  Search for **Arista NGFW** and select it.
3.  Click **Add Arista NGFW**.
4.  Configure the following configuration variables:
    -   **Syslog Host**: Specify the interface for the agent to listen on. Use `0.0.0.0` to listen on all available network interfaces, or provide a specific IP address of the Elastic Agent host.
    -   **Syslog Port**: Enter the port number configured in the Arista UI (e.g., `9514`).
    -   **Protocol**: Select **udp** or **tcp** to match the Arista appliance configuration.
    -   **Internal Zones**: List the internal network zones (e.g., `Internal`, `VLAN10`). This helps the integration correctly identify traffic direction.
    -   **External Zones**: List the external or WAN zones (e.g., `WAN`, `External`).
5.  Under **Advanced options**, you may optionally:
    -   Adjust **Internal Queue Size** if you expect high bursts of traffic to prevent packet loss.
    -   Add custom **Tags** to identify logs from specific branch offices or hardware models.
6.  Select the **Existing policy** where your Elastic Agent is enrolled.
7.  Click **Save and continue**, then **Save and deploy**.

# Validation Steps

### 1. Trigger Data Flow on Arista NGFW
1.  **Generate Traffic:** From a device behind the Arista NGFW, visit several websites to generate `WebFilterEvent` and `SessionEvent` logs.
2.  **Test Security Logging:** Trigger a security event by downloading a non-malicious test string. Use the following direct link for the [EICAR test file](https://www.eicar.org/download/eicar.com.txt) to trigger a `ThreatEvent` in the Virus Blocker or Threat Prevention app.
3.  **System Event:** Perform a "Test Connectivity" action or a "Check for Updates" in the Arista UI under **Config > Support** to generate system-level logs.

### 2. Check Data in Kibana
1.  Navigate to **Analytics** > **Discover**.
2.  Select the `logs-*` index pattern.
3.  Filter for the specific Arista dataset using the following query:
    `data_stream.dataset : "arista_ngfw.log"`
4.  Verify the presence of the following fields to ensure successful parsing:
    -   `event.dataset`: Verify it is set to `arista_ngfw.log`.
    -   `source.ip`: Check that it contains valid internal or external client IPs.
    -   `arista_ngfw.event_class`: Ensure this shows values like `SessionEvent`, `ThreatEvent`, or `WebFilterEvent`.
    -   `message`: Confirm the raw syslog string is being captured.
5.  Navigate to **Analytics** > **Dashboard** and open the **[Logs Arista NGFW] Overview** dashboard to see the visualized data.

# Troubleshooting

## Common Configuration Issues

-   **Network Path Blocked**: Use the Arista **Reports > Network > Sessions** to see if traffic to the syslog port is being dropped by the firewall itself. Ensure an Access Rule allows traffic to the agent.
-   **Service Binding Conflict**: If the Elastic Agent log shows `bind: address already in use`, another service (like `rsyslog` or `syslog-ng`) is using the port. Stop the conflicting service or change the **Syslog Port** in both Arista and Kibana to an unused port (e.g., `9514`).
-   **Protocol Mismatch**: If you see "connection reset" or no data arriving, ensure that if Arista is set to **TCP**, the Kibana integration is not set to **UDP**. Both must be identical.

## Ingestion Errors

-   **_grokparsefailure**: This occurs if the Arista syslog format has been customized. Ensure Arista is using the default syslog output format as specified in the vendor documentation.
-   **Time Synchronization**: If logs are not appearing in the current time window, check **Config > Support** in Arista to ensure NTP is synchronized. Mismatched clocks will cause logs to appear in the "past" or "future" in Kibana.

## Vendor Resources

-   [Arista Edge Threat Management Wiki - Syslog Events](https://wiki.edge.arista.com/index.php/Events#Syslog)
-   [Arista Wiki - Event Definitions and Classes](https://wiki.edge.arista.com/index.php/Event_Definitions)
-   [Arista Wiki - Troubleshooting Guide](https://wiki.edge.arista.com/index.php/Troubleshooting)

# Documentation sites

-   [Arista Edge Threat Management Official Wiki](https://wiki.edge.arista.com/index.php/Main_Page)
-   [Elastic Common Schema (ECS) Reference](https://www.elastic.co/docs/reference/ecs)