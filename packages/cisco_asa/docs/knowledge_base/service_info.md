# Service Info

## Common use cases

The Cisco ASA (Adaptive Security Appliance) integration for Elastic provides deep visibility into your network security infrastructure by ingesting and normalizing firewall, VPN, and system logs.
- **Security Monitoring and Threat Detection:** Identify blocked connection attempts, unauthorized access attempts, and potential reconnaissance activity by analyzing denied traffic logs across all interfaces.
- **VPN Activity Auditing:** Monitor Remote Access VPN and Site-to-Site VPN sessions, including user logins, session duration, assigned IP addresses, and data transfer volumes for compliance and usage tracking.
- **Administrative Audit Logging:** Track configuration changes, login attempts to the ASA CLI or ASDM, and command execution to maintain a secure audit trail of administrative actions.
- **Network Troubleshooting:** Use detailed connection logs (NAT translations, ACL hits, and teardown reasons) to diagnose connectivity issues and optimize firewall policy performance.

## Data types collected

This integration can collect the following types of data from Cisco ASA devices:
- **Firewall Traffic Logs:** Detailed records of permitted and denied connections, including source/destination IPs, ports, and protocols.
- **System Event Logs:** High-level system messages regarding hardware status, failover events, and resource utilization.
- **VPN/Tunneling Logs:** Authentication events, session establishment, and teardown details for AnyConnect and IPsec tunnels.
- **Audit Logs:** Command-line execution history and configuration change notifications.
- **Data Formats:** Logs are primarily collected via the **Syslog** protocol in Cisco's proprietary message format or standard BSD/IETF formats.
- **Metrics:** While primarily log-based, the integration extracts data that can be used for metrics such as connection counts, throughput per interface, and active VPN session counts.

## Compatibility
The **Cisco ASA** integration is compatible with physical ASA 5500-X series appliances, Firepower hardware running ASA software, and the virtualized ASAv platform.
- **Supported Versions:** Cisco ASA software version 9.x and higher.
- **Firmware Requirements:** Works with standard ASA syslog formats; customization of the logging message format may require adjustments to the integration's ingest pipeline.

## Scaling and Performance

- **Transport/Collection Considerations:** This integration supports both UDP and TCP transport for syslog. While UDP (port 514) offers higher performance and lower overhead on the ASA, it does not guarantee delivery. For critical security environments, TCP is recommended to ensure no logs are lost during network congestion, though it requires the `permit-hostdown` setting on the ASA to prevent traffic interruption if the Agent is unreachable.
- **Data Volume Management:** To manage high log volumes, use the `logging trap` command on the ASA to filter messages by severity. It is recommended to use `informational` (level 6) for general monitoring. Avoid using `debugging` (level 7) in production environments as it can significantly increase CPU load on the ASA and generate excessive data volumes that may overwhelm the ingestion pipeline.
- **Elastic Agent Scaling:** For high-throughput environments processing thousands of events per second, it is recommended to deploy the Elastic Agent on a dedicated VM with at least 2-4 vCPUs. In large-scale deployments, use multiple Elastic Agents behind a network load balancer (NLB) to distribute the incoming UDP/TCP syslog traffic and provide high availability for the log collection layer.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** Must have `enable` level access to the CLI or administrative credentials for the ASDM interface.
- **Network Connectivity:** The ASA must be able to reach the Elastic Agent's IP address over the chosen protocol (UDP port 514 or a custom TCP port).
- **Interface Selection:** Identify the specific ASA interface (e.g., `inside`, `management`, or `dmz`) that has a network path to the Elastic Agent.
- **License Requirements:** No specific "Logging" license is required, but the ASA must have a valid Base or Security Plus license to operate and generate traffic logs.
- **Time Synchronization:** It is highly recommended to configure NTP on the Cisco ASA to ensure log timestamps align correctly with the Elastic Stack.

## Elastic prerequisites

- **Elastic Agent:** An active Elastic Agent must be installed and enrolled in a policy via Fleet.
- **Integration Package:** The **Cisco ASA** integration must be added to the Agent policy.
- **Network Visibility:** Ensure that the host running the Elastic Agent has firewall rules (iptables/firewalld) allowing inbound traffic on the configured syslog port (e.g., 514/UDP or 1514/TCP).

## Vendor set up steps

### For Command Line Interface (CLI) Collection:
1. **Access the ASA CLI:** Log in to the appliance via SSH or console and enter global configuration mode.
   ```bash
   enable
   configure terminal
   ```
2. **Enable Global Logging:** Activate the logging subsystem.
   ```bash
   logging enable
   ```
3. **Configure the Destination Host:** Point the ASA to the IP address of your Elastic Agent. Replace `[interface]` with your source interface name (e.g., inside), `[agent_ip]` with the Agent's IP, and `[port]` with your chosen port.
   *   **For UDP:** `logging host [interface] [agent_ip] udp/[port]`
   *   **For TCP:** `logging host [interface] [agent_ip] tcp/[port]`
4. **Set Severity Level:** Determine which logs are sent. Level 6 (informational) is standard for most security monitoring.
   ```bash
   logging trap informational
   ```
5. **Configure Timestamps:** Ensure logs include date and time information.
   ```bash
   logging timestamp
   ```
6. **Set Facility Code:** Configure a syslog facility (local7 is common, value 23).
   ```bash
   logging facility 23
   ```
7. **Handle TCP Failures (Critical for TCP):** If using TCP, ensure the ASA does not stop traffic if the Agent is down.
   ```bash
   logging permit-hostdown
   ```
8. **Save Changes:** Write the configuration to memory.
   ```bash
   write memory
   ```

### For ASDM (GUI) Collection:
1. Log in to the **Cisco ASDM** interface.
2. Navigate to **Configuration > Device Management > Logging > Syslog Servers**.
3. Click **Add** to create a new server entry.
4. Select the **Interface** used to reach the Elastic Agent, enter the **IP Address**, choose the **Protocol** (UDP/TCP), and specify the **Port**.
5. Click **OK**.
6. Navigate to **Logging > Logging Filters**.
7. Select **Syslog Servers** and click **Edit**.
8. Set the **Filter on severity** to **Informational** (or your preferred level) and click **OK**.
9. Click **Apply** to push the configuration to the appliance.

## Kibana set up steps

1.  Navigate to **Management > Integrations** in the Kibana main menu.
2.  Search for **Cisco ASA** and click on the integration tile.
3.  Click **Add Cisco ASA**.
4.  Configure the integration settings:
    - **Syslog Host:** Set this to `0.0.0.0` to listen on all interfaces or the specific IP of the Agent host.
    - **Syslog Port:** Enter the port configured on the ASA (e.g., `514` for UDP or `1514` for TCP).
    - **Protocol:** Select either `udp` or `tcp` to match the ASA configuration.
5.  Under **Advanced options**, ensure the **Dataset name** is set to `cisco_asa.log`.
6.  Select the **Existing host policy** where your Elastic Agent is enrolled.
7.  Click **Save and continue**, then click **Add agent to your hosts** or **Save and deploy** to apply the configuration.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from Cisco ASA to the Elastic Stack.

### 1. Trigger Data Flow on Cisco ASA:
- **Generate configuration event:** Log in to the CLI, enter `configure terminal`, then `exit`. This triggers a "User configured the device" log.
- **Trigger interface event:** Perform a `shutdown` followed by `no shutdown` on a spare or non-production interface to generate link-state events.
- **Generate authentication event:** Log out of the ASDM or SSH session and log back in to generate AAA (Authentication, Authorization, and Accounting) logs.
- **Generate traffic events:** From a host behind the ASA, attempt to ping or browse an address that is explicitly denied or allowed by an Access Control List (ACL).

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the following KQL filter: `data_stream.dataset : "cisco_asa.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `cisco_asa.log`)
   - `source.ip` and/or `destination.ip` (for traffic logs)
   - `cisco_asa.message_id` (e.g., `302013`, `106023`)
   - `cisco_asa.mnemonic` (e.g., `Built`, `Deny`)
   - `message` (containing the raw log payload starting with `%ASA-`)
5. Navigate to **Analytics > Dashboards** and search for "Cisco ASA" to view the pre-built **[Logs Cisco ASA] Overview** dashboard.

# Troubleshooting

## Common Configuration Issues

- **Incorrect Interface Selection**: If the ASA is configured to send logs out of the `inside` interface but the Elastic Agent is only reachable via the `management` interface, logs will never arrive. Use the `ping [interface] [agent_ip]` command on the ASA to verify reachability.
- **Port/Protocol Mismatch**: Ensure that the protocol (UDP vs TCP) and port number (514 vs custom) exactly match between the ASA configuration and the Kibana integration settings.
- **ASA Port Range Restrictions**: Remember that Cisco ASA requires custom syslog ports for TCP to be within the range of 1025-65535. Using a port like 514 for TCP may fail on some software versions.
- **Logging Hostdown Blocking**: If using TCP and the Elastic Agent goes offline, the ASA might stop passing new traffic by default. Use `logging permit-hostdown` to prevent service outages during maintenance of the Elastic Stack.

## Ingestion Errors

- **Parsing Failures**: If logs appear in Kibana but fields like `source.ip` are missing, check the `error.message` field. This often happens if the ASA is using a custom prefix or timestamp format that deviates from the standard `%ASA-` format.
- **Timezone Offsets**: If logs appear to be "from the future" or "hours ago," ensure the ASA has NTP configured and that the `logging timestamp` command is enabled. The integration expects UTC or a timestamp with a valid offset.
- **Missing Integration Fields**: If only the `message` field is populated, ensure that the "Cisco ASA" integration is being used and not a generic "UDP/TCP Syslog" input, as the ASA-specific parser is required to map the mnemonic codes.

## Vendor Resources

- [Configure Adaptive Security Appliance (ASA) Syslog - Cisco](https://www.cisco.com/c/en/us/support/docs/security/pix-500-series-security-appliances/63884-config-asa-00.html)
- [CLI Book 1: Cisco Secure Firewall ASA Series General Operations CLI Configuration Guide, 9.23 - Logging](https://www.cisco.com/c/en/us/td/docs/security/asa/asa923/configuration/general/asa-923-general-config/monitor-syslog.html)

## Documentation sites

- [Configure Adaptive Security Appliance (ASA) Syslog - Cisco](https://www.cisco.com/c/en/us/support/docs/security/pix-500-series-security-appliances/63884-config-asa-00.html)
- [CLI Book 1: Cisco Secure Firewall ASA Series General Operations CLI Configuration Guide, 9.23 - Logging](https://www.cisco.com/c/en/us/td/docs/security/asa/asa923/configuration/general/asa-923-general-config/monitor-syslog.html)
