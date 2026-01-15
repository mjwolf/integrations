# Service Info

## Common use cases
The SonicWall integration for Elastic Agent allows organizations to ingest, parse, and visualize security and network events from SonicWall firewalls. This visibility is essential for maintaining a secure network perimeter and identifying potential threats in real-time.
- **Security Threat Monitoring:** Detect and alert on intrusion attempts, malware detections, and unauthorized access attempts identified by SonicWall’s security services (GAV, IPS, and App Control).
- **Network Traffic Analysis:** Analyze bandwidth usage, identify top talkers, and monitor traffic patterns across different interfaces and VPN tunnels to optimize network performance.
- **Compliance and Auditing:** Maintain long-term logs of administrative changes, user authentication events, and firewall policy hits to satisfy regulatory requirements like PCI-DSS, HIPAA, and GDPR.
- **VPN and Remote Access Tracking:** Monitor SSL-VPN and Global VPN Client (GVC) connections to track user activity, session durations, and potential brute-force login attempts from external sources.

## Data types collected

This integration can collect the following types of data:
- **Firewall Traffic Logs:** Detailed records of inbound and outbound traffic, including source/destination IP addresses, ports, protocols, and the firewall action taken (allow, drop, or reject).
- **Authentication and User Activity:** Logs related to local user logins, LDAP/RADIUS authentication results, and administrative access events.
- **Security Service Events:** Data from advanced security features including Gateway Anti-Virus, Anti-Spyware, and Intrusion Prevention Service (IPS).
- **System Health and Audit Logs:** Hardware status updates, firmware management events, and detailed records of configuration changes made via the web interface or CLI.
- **Format:** All data is collected via Syslog, specifically utilizing the SonicWall **Enhanced Syslog** format for structured key-value pair parsing.

## Compatibility
The SonicWall integration is compatible with **SonicWall Next-Generation Firewalls (NGFW)** running the following firmware:
- **SonicOS 7.x**: Fully supported with enhanced logging features.
- **SonicOS 6.5**: Supported using the standard syslog configuration paths.
- **SonicOSv**: Virtual appliance versions compatible with the aforementioned firmware releases.

## Scaling and Performance

- **Transport/Collection Considerations:** This integration primarily uses Syslog over UDP or TCP. While UDP offers lower overhead and higher speed, TCP is recommended for environments where log delivery reliability is critical. Ensure the network path between the SonicWall appliance and the Elastic Agent can handle the expected packets-per-second (PPS) during peak traffic hours.
- **Data Volume Management:** To reduce the load on both the firewall and the Elastic Stack, use the SonicWall "Log Settings" to filter out high-volume, low-value events. It is recommended to prioritize "Warning" level events and critical security categories like Security Policy and Authentication while limiting verbose debugging or informational connection logs.
- **Elastic Agent Scaling:** For high-throughput environments with multiple firewall clusters, a single Elastic Agent may become a bottleneck. Deploy multiple Elastic Agents behind a load balancer to distribute the incoming Syslog stream, and ensure the host server has sufficient CPU resources for parsing the "Enhanced" syslog strings.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have full administrator credentials for the SonicWall management interface (SonicOS).
- **Network Connectivity:** The SonicWall appliance must be able to reach the Elastic Agent host over the configured Syslog port (default is usually UDP 514 or TCP 1468).
- **Address Objects:** You must be able to create new Address Objects within the SonicWall configuration to define the syslog destination.
- **License Requirements:** No specific license is required for Syslog forwarding, though certain log categories (like IPS or GAV) require their respective security services to be licensed and active.
- **Logging Configuration:** The firewall must be configured to use the **Enhanced Syslog** format to ensure compatibility with Elastic's ECS mapping.

## Elastic prerequisites

- **Elastic Agent:** An Elastic Agent must be installed and enrolled in a policy via Fleet.
- **Integration Policy:** The SonicWall integration must be added to the Agent policy.
- **Network Port:** The host running the Elastic Agent must have its local firewall configured to allow inbound traffic on the port specified in the integration settings.

## Vendor set up steps

### For SonicOS 7.X:
1. Log in to your SonicWall firewall as an administrator.
2. Navigate to **Object > Address Objects**. Click **Add** to create a new Address Object for the Elastic Agent.
   - **Name**: `Elastic_Agent_Host`
   - **Zone Assignment**: Select the zone where the agent resides (e.g., LAN).
   - **Type**: `Host`
   - **IP Address**: Enter the IP address of your Elastic Agent.
3. Navigate to **Device > Log > Syslog**.
4. Under the **Syslog Servers** section, click the **Add** button.
5. In the "Add Syslog Server" window, configure the following:
   - **Name or IP address**: Select the `Elastic_Agent_Host` object created in step 2.
   - **Port**: Enter `514` (or your configured port).
   - **Syslog Format**: Select **Enhanced** from the dropdown menu (this is critical for parsing).
   - **Syslog ID**: Enter `firewall`.
6. Click **OK** to save the server.
7. Navigate to **Device > Log > Settings**.
8. Expand the categories and ensure the **Syslog** column is checked for the events you wish to forward.
9. Click **Accept** at the bottom of the screen to apply changes.

### For SonicOS 6.5:
1. Log in to the SonicWall management interface.
2. Navigate to **Manage > Objects > Address Objects**.
3. Click **Add** and create an entry for the Elastic Agent IP address (e.g., Name: `ElasticAgent`, Type: `Host`).
4. Navigate to **Manage > Log Settings > SYSLOG**.
5. In the **Syslog Servers** table, click **Add**.
6. Set the following parameters:
   - **Name or IP address**: Select the `ElasticAgent` address object.
   - **Port**: Set to `514`.
   - **Syslog Format**: Select **Enhanced**.
   - **Syslog ID**: Set to `firewall`.
7. Click **OK**.
8. Navigate to **Manage > Log Settings > Base Setup**.
9. Set the **Logging Level** (e.g., "Informational" or "Notice").
10. Ensure the **Syslog** checkbox is enabled for the desired log categories in the table below.
11. Click **Accept** to save configuration.

## Kibana set up steps

1. In Kibana, navigate to **Management > Integrations**.
2. Search for and select **SonicWall**.
3. Click **Add SonicWall**.
4. Configure the integration settings:
    * **Syslog Host**: Set to `0.0.0.0` to listen on all interfaces or the specific IP of the Agent host.
    * **Syslog Port**: Enter the port number (e.g., `514` for UDP or `1468` for TCP).
    * **Protocol**: Choose either `udp` or `tcp` to match your SonicWall configuration.
5. Under the **Advanced options**, ensure the internal data stream is enabled (usually `logs-sonicwall.log`).
6. Click **Save and continue**.
7. Select the Agent Policy that contains your active Elastic Agent.
8. Click **Save and deploy changes**.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from SonicWall to the Elastic Stack.

### 1. Trigger Data Flow on SonicWall:
- **Generate authentication event:** Log out of the SonicWall web management interface and log back in. This should trigger an authentication event.
- **Generate configuration event:** Navigate to any setting (e.g., a comment on an address object), modify it, and click "Save" to trigger a configuration audit event.
- **Generate VPN event:** If configured, connect and disconnect a user via the NetExtender or Mobile Connect VPN client to trigger SSL VPN logs.
- **Generate web traffic:** From a machine behind the SonicWall firewall, attempt to access a website that is logged or filtered by the Security Policy.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the following KQL filter: `data_stream.dataset : "sonicwall.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `sonicwall.log`)
   - `source.ip` and `destination.ip` (should reflect the traffic passing through the firewall)
   - `event.action` (e.g., `login`, `built`, `denied`)
   - `sonicwall.mnemonic` or `sonicwall.event_id` (e.g., `1080` for VPN login)
   - `message` (containing the raw Enhanced Syslog payload)
5. Navigate to **Analytics > Dashboards** and search for "SonicWall" to view the pre-built Overview dashboard.

# Troubleshooting

## Common Configuration Issues

- **Incorrect Syslog Format**: If the SonicWall is set to "Standard" or "Default" syslog format instead of **Enhanced**, the Elastic Agent may fail to parse specific fields correctly. Ensure the "Syslog Format" is explicitly set to **Enhanced** in the Syslog Server settings.
- **Network Routing/ACLs**: If logs are not arriving, check that the SonicWall zone where the Elastic Agent resides has a rule allowing traffic on the Syslog port. Also, verify that the host machine's OS firewall (iptables/Windows Firewall) is not blocking the inbound port.
- **Address Object Mismatch**: If the Address Object IP address does not exactly match the Elastic Agent's IP, the firewall will send logs to a non-existent destination. Double-check the Address Object configuration under **Object > Match Objects > Addresses**.
- **Log Category Disablement**: If you see traffic logs but no VPN or Admin logs, verify that the "Syslog" column is checked for those specific categories under **Device > Log > Settings**.

## Ingestion Errors

- **Parsing Failures**: If logs appear in Discover but have a `_grokparsefailure` or `_jsonparsefailure` tag, check the `error.message` field. This often happens if the "Enhanced" syslog format has been customized with non-standard delimiters.
- **Timestamp Mismatches**: Ensure the SonicWall appliance and the Elastic Agent host are synchronized using NTP. Significant clock drift can cause logs to appear as "in the future" or "in the past" in Kibana Discover.
- **Field Mapping Issues**: If specific SonicWall fields are missing, verify that the version of SonicOS is supported. Newer firmware versions may introduce new fields not yet covered by the integration's ingest pipeline.

## Vendor Resources

- [How can I configure a syslog server on a SonicWall firewall?](https://www.sonicwall.com/support/knowledge-base/how-can-i-configure-a-syslog-server-on-a-sonicwall-firewall/kA1VN0000000TWl0AM)
- [Configure a SonicWall firewall with SonicOS 7.x to send logs - Arctic Wolf](https://docs.arcticwolf.com/bundle/m_syslog/page/configure_a_sonicwall_firewall_with_sonicos_7x_to_send_logs.html)

## Documentation sites

- [How can I configure a syslog server on a SonicWall firewall?](https://www.sonicwall.com/support/knowledge-base/how-can-i-configure-a-syslog-server-on-a-sonicwall-firewall/kA1VN0000000TWl0AM)
- [Configure a SonicWall firewall with SonicOS 7.x to send logs - Arctic Wolf](https://docs.arcticwolf.com/bundle/m_syslog/page/configure_a_sonicwall_firewall_with_sonicos_7x_to_send_logs.html)
