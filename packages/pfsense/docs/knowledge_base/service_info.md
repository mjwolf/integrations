# Service Info

## Common use cases

The pfSense integration for Elastic allows users to ingest and visualize critical network data from their pfSense firewalls and routers, providing deep visibility into network security and performance.
- **Security Monitoring and Threat Detection:** Analyze firewall pass and block logs to identify malicious traffic patterns, port scanning attempts, or unauthorized access from known bad actors.
- **Network Troubleshooting:** Inspect DNS resolver (Unbound), DHCP, and gateway logs to diagnose connectivity issues, latency spikes, or IP assignment conflicts within the environment.
- **Compliance and Auditing:** Maintain a tamper-proof record of system events, administrative logins, and configuration changes to satisfy regulatory requirements like PCI-DSS and SOC2.
- **VPN and Remote Access Visibility:** Monitor OpenVPN or IPsec logs to track remote user activity, session durations, and authentication successes or failures.

## Data types collected

This integration can collect the following types of data from the pfSense environment, primarily via the Syslog protocol:
- **Firewall Filter Logs:** Detailed information about packets allowed or blocked by the pfSense packet filter (pf), including source/destination IPs, ports, and protocols.
- **System Logs:** Operating system events, kernel messages, and process-specific logs from the underlying FreeBSD platform.
- **DHCP Logs:** Information regarding IP address assignments, renewals, and client requests from the DHCP server.
- **DNS Resolver Logs:** Query logs from the Unbound DNS service, including requested hostnames and response statuses.
- **VPN Logs:** Connection events and authentication logs from IPsec or OpenVPN services configured on the firewall.
- **Gateway and Routing Logs:** Events related to gateway monitoring (dpinger) and routing table changes.
- **Data Format:** Logs are collected in Syslog format (BSD or RFC 5424) and parsed into Elastic Common Schema (ECS) fields.

## Compatibility

The **pfSense** integration is compatible with the following versions:
- **pfSense CE (Community Edition):** Version 2.4.x and 2.5.x, and 2.6.x+ are supported.
- **pfSense Plus:** Version 21.02, 22.01, and 23.01+ are supported.
- The integration assumes logs are being sent in a format compatible with standard BSD syslog or RFC 5424.

## Scaling and Performance

- **Transport/Collection Considerations:** pfSense uses UDP for its default remote syslog implementation. While UDP offers high performance with low overhead, it does not guarantee delivery. For high-volume environments or where log integrity is critical, ensure the network path between pfSense and the Elastic Agent is low-latency and non-congested to minimize packet loss.
- **Data Volume Management:** To prevent overwhelming the Elastic Stack, avoid selecting the "Everything" checkbox in pfSense Remote Logging Options unless necessary. Instead, select only the specific log categories required for your use case (e.g., Firewall, VPN, and DHCP). Filtering out noisy firewall rules at the source (pfSense rule level) can significantly reduce the volume of "Default Deny" logs.
- **Elastic Agent Scaling:** A single Elastic Agent can typically handle several thousand events per second (EPS). However, for environments with multiple pfSense high-availability clusters or high-throughput 10Gbps+ interfaces, consider deploying multiple Elastic Agents behind a load balancer to distribute the syslog ingestion load and provide high availability for the log collection layer.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have an account with administrative privileges to the pfSense WebGUI to modify system logging settings.
- **Network Connectivity:** The pfSense device must be able to reach the Elastic Agent host over the network on the designated syslog port (e.g., UDP 514 or a custom port like 9514).
- **Firewall Rules:** Ensure that local pfSense firewall rules allow outgoing traffic from the firewall itself to the Elastic Agent's IP address and port.
- **NTP Synchronization:** It is critical that pfSense has NTP configured and synchronized to ensure that log timestamps align correctly with the Elastic Stack.
- **License Requirement:** No special licenses are required; both pfSense CE and Plus versions support remote syslog natively.

## Elastic prerequisites

To successfully ingest pfSense logs, ensure the following Elastic environment settings:
- **Elastic Agent:** An Elastic Agent must be installed and successfully enrolled in Fleet.
- **Integration Policy:** The pfSense integration must be added to the Agent policy associated with your deployed agent.
- **Connectivity:** The host running the Elastic Agent must have its local firewall configured to allow inbound traffic on the port designated for pfSense syslog (e.g., port 5144).

## Vendor set up steps

To configure pfSense to forward logs to Elastic Agent, follow these detailed steps within the pfSense web interface:

1.  Log in to your **pfSense web interface** using an account with administrative rights.
2.  Navigate to the top menu and select **Status > System Logs**.
3.  Click on the **Settings** tab located in the sub-menu.
4.  In the **General Logging Options** section, find the **Log Message Format** dropdown. It is highly recommended to select `syslog (RFC 5424, with RFC 3339 microsecond precision timestamps)` as this provides the most structured data for the Elastic parser.
5.  Scroll down to the **Remote Logging Options** section.
6.  Check the box labeled **Send log messages to remote syslog server**.
7.  Configure the specific remote logging parameters:
    *   **Source Address**: Select `Default (any)` or choose a specific interface IP if you have a complex routing setup.
    *   **IP Protocol**: Select `IPv4` or `IPv6` based on your network environment.
    *   **Remote log servers**: Enter the IP address and port of your Elastic Agent. Format the entry as `IP:PORT` (e.g., `192.168.1.50:514`). You may specify multiple servers separated by commas if using a load-balanced setup.
8.  Under **Remote Syslog Contents**, you can customize what data is sent. For a comprehensive overview, check the **Everything** box. Alternatively, specifically check boxes for **System Events**, **Firewall Events**, **DNS Resolver**, and **DHCP Events**.
9.  Scroll to the bottom of the page and click the **Save** button.
10. Navigate to **Status > System Logs > Remote** to verify that the firewall shows logs being queued for transmission.

## Kibana set up steps

### For Syslog Collection:
1.  Open Kibana and navigate to **Management > Integrations**.
2.  Search for **pfSense** and select the integration.
3.  Click **Add pfSense**.
4.  In the integration configuration, locate the **Syslog** input section.
5.  Set the **Syslog Host** to `0.0.0.0` to listen on all interfaces or enter the specific IP address of the host where Elastic Agent is running.
6.  Set the **Syslog Port** to the same port configured in pfSense (default is often `514` or `9514`).
7.  Select the **Protocol** (usually `UDP`, matching your pfSense configuration).
8.  Under **Internal interfaces**, enter any internal interface names (e.g., `lan`, `opt1`) used by pfSense to help the integration correctly categorize traffic direction.
9.  Click **Save and continue**, then select the appropriate **Agent Policy** to deploy the configuration to your agents.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from pfSense to the Elastic Stack.

### 1. Trigger Data Flow on pfSense:
- **Generate Firewall Event:** Attempt to access a blocked port or service from a device on the network to trigger a "block" event in the firewall logs.
- **Generate Web Traffic:** Browse several websites from a client located behind the pfSense firewall to generate "pass" traffic logs.
- **Generate Authentication Event:** Log out of the pfSense WebGUI and then log back in to generate system authentication and session events.
- **Force DNS Query:** Run a command like `nslookup elastic.co` from a machine using the pfSense device as its DNS resolver to trigger DNS logs.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view or the specific `pfsense` data view if created.
3. Enter the following KQL filter: `data_stream.dataset : "pfsense.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `pfsense.log`)
   - `source.ip` (should show the source of the network traffic)
   - `destination.ip` (should show the destination of the network traffic)
   - `pfsense.firewall.action` (should show `pass` or `block`)
   - `message` (containing the raw RFC 5424 or 3164 log payload)
5. Navigate to **Analytics > Dashboards** and search for **[Logs pfSense] Overview** to view the pre-built dashboards and confirm visualizations are populating with data.

# Troubleshooting

## Common Configuration Issues

- **Log Format Mismatch**: If logs appear in Discover but fields are not correctly parsed (showing as tags like `_grokparsefailure`), ensure that the **Log Message Format** in pfSense (Status > System Logs > Settings) is set to `RFC 5424`. The integration is optimized for this format.
- **Network Connectivity and Firewalls**: If no logs appear, verify that the pfSense device can reach the Elastic Agent host. Check for any intermediate firewalls blocking the syslog port (UDP/514). Use `tcpdump` on the Elastic Agent host to confirm packets are arriving: `tcpdump -ni any port 514`.
- **Port Conflicts**: Ensure that no other service on the Elastic Agent host is already bound to the syslog port. If the default port 514 is in use by the system's local syslog daemon (like rsyslog or syslog-ng), change the port in both pfSense and the Elastic integration to a high port like `9514`.
- **Interface Naming**: If traffic direction is not being identified correctly, ensure that the interface names in the integration settings match the internal interface names used in pfSense (e.g., `em1`, `igb0`).

## Ingestion Errors

- **Parsing Failures**: If logs appear but fields are not correctly parsed (indicated by `tags: ["_grokparsefailure"]`), ensure that pfSense is not using a custom log format. The integration expects standard BSD/RFC 3164 format.
- **Timestamp Mismatches**: If logs appear in the past or future, verify that the Time Zone settings on the pfSense appliance (**System > General Setup**) match the time synchronization on the Elastic Agent host.
- **Field Mapping Issues**: Check the `error.message` field in Kibana Discover. If you see "mapping conflict" errors, ensure you have the latest version of the pfSense integration and that the package assets have been correctly installed.

## Vendor Resources

- [Remote Logging with Syslog | pfSense Documentation](https://docs.netgate.com/pfsense/en/latest/monitoring/logs/remote.html)

## Documentation sites

- [How to configure pfSense to forward syslogs to LogCentral](https://docs.logcentral.io/en/articles/10310276-how-to-configure-pfsense-to-foward-syslogs-to-logcentral)
- Refer to the official vendor website for additional product administration guides.
