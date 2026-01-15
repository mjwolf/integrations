# Service Info

The Check Point integration allows for the seamless ingestion of security logs from Check Point Management Servers and Log Servers into the Elastic Stack. By leveraging the Check Point Log Exporter utility, users can gain comprehensive visibility into their network security posture, facilitating advanced threat hunting, compliance auditing, and real-time operational monitoring across the entire enterprise infrastructure.

## Common use cases

The Check Point integration for Elastic Agent allows organizations to ingest, normalize, and visualize security events from Check Point Management Servers and Log Servers. By centralizing these logs in the Elastic Stack, security teams can perform advanced analysis and correlation across their network security infrastructure.

- **Threat Detection and Response:** Monitor firewall logs, IPS events, and Anti-Bot detections in real-time to identify and mitigate malicious activity within the network.
- **Compliance Reporting:** Maintain a comprehensive audit trail of administrative changes and network traffic to satisfy regulatory requirements such as PCI-DSS, HIPAA, and GDPR.
- **Network Visibility and Optimization:** Analyze traffic patterns and application usage to optimize firewall rules and identify bandwidth bottlenecks.
- **Security Posture Auditing:** Track administrative logins, policy installations, and configuration changes to ensure internal security policies are being followed.

## Data types collected

This integration can collect the following types of data from Check Point devices:
- **Firewall Logs:** Connection details including source/destination IP, ports, protocols, and policy actions (accept, drop, reject).
- **IPS (Intrusion Prevention System) Logs:** Detailed alerts regarding signature-based threats and protocol anomalies.
- **Audit Logs:** Administrative changes made via SmartConsole, including policy installations and object modifications.
- **VPN Logs:** Endpoint connectivity data, user authentication events, and tunnel establishment details.
- **Threat Prevention Logs:** Events from Anti-Bot, Anti-Virus, SandBlast (Threat Emulation), and URL Filtering blades.
- **Data Formats:** Logs are typically collected in **CEF (Common Event Format)**, but the integration also supports standard Syslog and JSON formats exported over network protocols.

## Compatibility

The Check Point integration is compatible with the following:

- **Check Point R80.x** and **R81.x** versions (including R81.10 and R81.20) using the built-in Log Exporter utility.
- Older versions (e.g., R77.30) may require the manual installation of the Log Exporter Jumbo Hotfix.
- Supports both Gaia and SecurePlatform operating systems.

## Scaling and Performance

- **Transport/Collection Considerations:** The integration supports both TCP and UDP protocols. For production environments, TCP is recommended to ensure reliable delivery of security events, especially when using the CEF format which can result in large packet sizes that may be fragmented over UDP.
- **Data Volume Management:** To reduce the load on the Elastic Stack and the Check Point Management Server, use the `cp_log_export` filtering capabilities to exclude high-volume, low-value logs (such as "Accept" logs for DNS or NTP) at the source before they are transmitted over the network.
- **Elastic Agent Scaling:** A single Elastic Agent can handle several thousand events per second (EPS) depending on the hardware; however, for high-throughput environments exceeding 10,000 EPS, it is recommended to deploy multiple Elastic Agents behind a network load balancer to distribute the ingestion load and provide high availability.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have SSH access to the Check Point Management Server or Log Server with "Expert" mode privileges.
- **Log Exporter Utility:** Ensure the Log Exporter is installed (included by default in R80.20+; available as a jumbo hotfix for earlier R80.x versions).
- **Network Connectivity:** Firewall rules must allow communication from the Check Point server to the Elastic Agent IP on the configured Syslog/CEF port (e.g., TCP/UDP 514 or 5514).
- **Licensing:** Ensure the appropriate logging and management licenses are active on the Check Point environment to allow log generation and export.
- **Resource Availability:** Verify that the Management Server has sufficient CPU and memory to run the additional `exporter` process without impacting management performance.

## Elastic prerequisites

- **Elastic Agent:** An active Elastic Agent must be installed and enrolled in a policy via Fleet.
- **Integration Policy:** The Check Point integration must be added to the Agent policy.
- **Network Listener:** The Elastic Agent must be reachable over the network on the port specified in the integration settings (default is often 514 or 9001).

## Vendor set up steps

### For Log Exporter (CEF) Collection:

1.  **Log in to the Check Point Server:** Connect via SSH to the Management Server or Log Server that handles the logs you wish to export.
2.  **Enter Expert Mode:** Run the command `expert` and provide the expert password to access the advanced shell.
3.  **Domain Context (Multi-Domain only):** If you are operating in a Multi-Domain Security Management (MDS) environment, switch to the context of the specific domain:
    ```shell
    mdsenv <IP Address or Name of Domain Management Server>
    ```
4.  **Create the Exporter Instance:** Execute the `cp_log_export add` command to define the connection to the Elastic Agent. Use the following syntax:
    ```shell
    cp_log_export add name elastic-agent-exporter target-server <ELASTIC_AGENT_IP> target-port <PORT> protocol <tcp|udp> format cef
    ```
    *   Replace `<ELASTIC_AGENT_IP>` with the IP of your Elastic Agent.
    *   Replace `<PORT>` with the port configured in your Elastic Agent policy (e.g., 5514).
5.  **Start the Exporter:** Initiate the log forwarding process by running:
    ```shell
    cp_log_export start name elastic-agent-exporter
    ```
6.  **Verify Status:** Confirm that the exporter is running and successfully connected:
    ```shell
    cp_log_export status
    ```
7.  **Verify Configuration Details:** To double-check the settings for your specific exporter instance, use:
    ```shell
    cp_log_export show name elastic-agent-exporter
    ```
8.  **Automatic Startup:** The exporter is designed to survive reboots, but you should verify it appears in the status list after any maintenance window.

## Kibana set up steps

### For Syslog/CEF Input:

1.  **Navigate to Integrations:** In Kibana, go to **Management > Integrations** and search for **Check Point**.
2.  **Add Integration:** Click **Add Check Point** and select an existing Agent Policy or create a new one.
3.  **Configure Input:** Under the **Logs (Check Point)** section, select the input type (e.g., **TCP** or **UDP**) that matches your Check Point `cp_log_export` command.
4.  **Set Listen Settings:** 
    *   **Listen Address:** Set this to `0.0.0.0` to listen on all interfaces or the specific IP of the Agent host.
    *   **Listen Port:** Enter the port number you used in the vendor setup (e.g., `5514`).
5.  **Set Log Format:** Ensure the **Syslog Format** or mapping matches the `cef` format exported by Check Point.
6.  **Save and Deploy:** Click **Save and continue** and then **Save and deploy changes** to push the configuration to your Elastic Agents.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from Check Point to the Elastic Stack.

### 1. Trigger Data Flow on Check Point:
- **Generate Traffic Event:** From a machine protected by the Check Point gateway, browse to a website or initiate a connection that will trigger a firewall rule.
- **Generate Management Event:** Log into SmartConsole, make a minor change (like a comment on a network object), and click "Publish". This generates an audit log.
- **Generate System Event:** In the Check Point CLI (Expert mode), run `cp_log_export restart name elastic-agent`. This action is logged and should be exported immediately.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the following KQL filter: `data_stream.dataset : "checkpoint.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `checkpoint.log`)
   - `source.ip` and/or `destination.ip` (representing the traffic seen by the firewall)
   - `event.action` (e.g., `accept`, `drop`, or `reject`)
   - `checkpoint.rule_name` or `checkpoint.rule_number`
   - `message` (containing the raw syslog payload from the Log Exporter)
5. Navigate to **Analytics > Dashboards** and search for "Check Point" to view the pre-built **[Logs Check Point] Overview** dashboard.

# Troubleshooting

## Common Configuration Issues

- **Log Exporter Not Running**: If logs are not appearing, run `cp_log_export show` in the Check Point CLI. If the status is "Not Running", use `cp_log_export restart name <name>` to initiate the process.
- **Network Connectivity/Firewall Blocks**: Ensure that no intermediate firewalls are blocking the traffic on the configured TCP/UDP port. You can test connectivity from the Check Point server using `nc -zv <agent_ip> <port>`.
- **Incorrect Protocol Mapping**: A common error is configuring the Check Point Log Exporter to use UDP while the Elastic Agent is set to listen for TCP (or vice versa). Ensure the `protocol` parameter matches exactly in both locations.
- **Port Conflict**: If the Elastic Agent fails to start the integration, check the agent logs for "address already in use" errors. This occurs if another process is already listening on the chosen port (e.g., a system-wide rsyslog service).

## Ingestion Errors

- **Parsing Failures:** If logs appear in Kibana but contain the tag `_grokparsefailure` or `_jsonparsefailure`, verify that the Check Point Log Exporter is set to `format cef`. Non-standard formats may not align with the integration's built-in parsers.
- **Timezone Mismatches:** If logs appear to be delayed or from the future, verify that the system time on the Check Point server is synchronized via NTP and that the Elastic Agent is correctly interpreting the timestamp format.
- **Field Mapping Issues:** Check the `error.message` field in Discover to identify if specific CEF fields are causing mapping conflicts in Elasticsearch.

## Vendor Resources

- [sk122323 - Log Exporter - Check Point Log Export](https://support.checkpoint.com/results/sk/sk122323)
- [Log Exporter Basic Configuration in CLI - Check Point R81 Administration Guide](https://sc1.checkpoint.com/documents/R81/WebAdminGuides/EN/CP_R81_LoggingAndMonitoring_AdminGuide/Topics-LMG/Log-Exporter-Configuration-in-CLI-Basic.htm)

## Documentation sites

- [cp_log_export - Check Point R81.20 Security Management Administration Guide](https://sc1.checkpoint.com/documents/R81.20/WebAdminGuides/EN/CP_R81.20_SecurityManagement_AdminGuide/Content/Topics-SECMG/CLI/cp_log_export.htm)
- [sk122323 - Log Exporter - Check Point Log Export](https://support.checkpoint.com/results/sk/sk122323)
- Refer to the official vendor website for additional resources regarding SmartConsole and Gateway logging.
