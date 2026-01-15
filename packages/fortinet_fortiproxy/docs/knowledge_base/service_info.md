# Service Info

The Fortinet FortiProxy integration allows users to ingest logs from their Secure Web Gateway (SWG) devices into the Elastic Stack. This provides centralized visibility into web traffic, security events, and administrative actions performed on the appliance, enabling advanced threat hunting and compliance reporting.

## Common use cases

The Fortinet FortiProxy integration allows organizations to centralize security and network traffic data from their high-performance secure web proxy. By ingesting these logs into the Elastic Stack, administrators can perform deep analysis on web traffic patterns and security threats.
- **Threat Detection and Hunting:** Analyze proxy logs to identify communication with known malicious domains, command-and-control (C2) servers, or unusual outbound traffic patterns that indicate a compromised internal host.
- **Regulatory Compliance:** Maintain long-term storage of web access logs and administrative actions to satisfy requirements for data retention and auditability as mandated by frameworks like PCI-DSS, HIPAA, or GDPR.
- **Network Performance Monitoring:** Identify bottlenecks or high-latency web requests by monitoring proxy response times and throughput metrics across different user groups or destinations.
- **User Activity Auditing:** Monitor and report on employee web usage, ensuring adherence to corporate acceptable use policies and identifying unauthorized attempts to access restricted content categories.

## Data types collected
This integration can collect the following types of data:
- **Traffic Logs:** Detailed information about web requests, including source/destination IP addresses, URLs, protocols, and data volume.
- **Web Filter Logs:** Events related to URL filtering, category-based blocking, and security profile actions taken on web traffic.
- **System Logs:** Administrative events, configuration changes, system health status, and daemon-level notifications.
- **Authentication Logs:** Records of user login attempts, logout events, and authentication method results for proxy access.
- **Format:** Data is primarily collected via Syslog in a key-value pair format (default Fortinet format), which is then parsed into the Elastic Common Schema (ECS).

## Compatibility
This integration is compatible with **Fortinet FortiProxy** devices.
- It is designed to work with **FortiProxy version 7.0 and higher**.
- Supports standard Syslog output using both UDP and Reliable (TCP) transport protocols with RFC6587 framing.

## Scaling and Performance

- **Transport/Collection Considerations:** Users can choose between UDP and TCP (referred to as "reliable" in Fortinet CLI) for log transmission. UDP offers lower overhead and higher speed but lacks delivery guarantees, making it suitable for high-volume traffic logs where occasional loss is acceptable. TCP/Reliable mode should be used when data integrity and delivery confirmation are paramount, though it may introduce latency if the network path is congested.
- **Data Volume Management:** To prevent overwhelming the Elastic Agent or the network, it is recommended to use the FortiProxy "Log Settings" to filter events at the source. Administrators should select only necessary log levels (e.g., Information or Warning and above) and disable logging for high-volume, low-value traffic types if they exceed the ingestion capacity of the environment.
- **Elastic Agent Scaling:** A single Elastic Agent can handle thousands of events per second (EPS), but in high-throughput enterprise environments, it is advisable to deploy multiple Agents behind a hardware or software load balancer. Ensure the Agent host has sufficient CPU and memory resources to handle the overhead of parsing high-volume syslog streams before forwarding to Elasticsearch.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have a super-administrator account or an account with "Log & Report" read/write permissions on the FortiProxy device.
- **Network Connectivity:** The FortiProxy appliance must have an unobstructed network path to the Elastic Agent host. Ensure any intermediate firewalls allow traffic on the configured syslog port (e.g., UDP/514 or TCP/514).
- **License Requirements:** Ensure the FortiProxy device has a valid license, as some logging features for security modules (like Web Filter or AntiVirus) require active FortiGuard subscriptions.
- **Source IP Identification:** Note the management IP or the specific interface IP of the FortiProxy that will be sending the syslog packets for configuration within the Elastic Agent's allowed hosts (if applicable).

## Elastic prerequisites

- **Elastic Agent:** An Elastic Agent must be installed and enrolled in a policy via Fleet.
- **Integration Installed:** The Fortinet FortiProxy integration must be added to the Agent policy in Kibana.
- **Connectivity:** The Elastic Agent must be configured to listen on a network interface accessible by the FortiProxy device (e.g., `0.0.0.0` to listen on all interfaces).

## Vendor set up steps

### For Syslog Collection (GUI Method):
1. **Log in** to the FortiProxy web-based manager (GUI).
2. Navigate to the sidebar and select **Log & Report > Log Settings**.
3. Locate the "Remote Logging and Archiving" section and toggle on **Send Logs to Syslog**.
4. In the **IP Address/FQDN** field, enter the IP address of the server where your Elastic Agent is running.
5. In the **Port** field, enter the port number the Elastic Agent is listening on (e.g., `514` or `5514`).
6. Configure the **Mode**: Select `UDP` or `Reliable` (for TCP). Ensure this matches your Elastic Agent configuration.
7. Set the **Format** to `Default`.
8. If you selected `Reliable` (TCP), ensure the **Framing** option is set to `rfc6587`.
9. Click **Apply** at the bottom of the page to save and activate the logging configuration.

### For Syslog Collection (CLI Method):
1. **Connect** to the FortiProxy CLI via SSH or the web-based console.
2. Enter the configuration mode for syslog settings:
   ```sh
   config log syslogd setting
       set status enable
       set server <elastic_agent_ip>
       set port <listening_port>
       set format default
       set mode reliable
   end
   ```
3. If you are using TCP (Reliable mode), you must specifically set the framing property to match the Elastic integration requirements:
   ```sh
   config log syslogd setting
       set framing rfc6587
   end
   ```
4. To verify the configuration, run the command `show log syslogd setting` to ensure all parameters are correctly applied.
5. (Optional) Run `diagnose log test` to generate a sample log event to test connectivity.

## Kibana set up steps

### Configure the FortiProxy Integration:
1. In Kibana, navigate to **Management > Integrations**.
2. Search for and select **Fortinet FortiProxy**.
3. Click **Add Fortinet FortiProxy**.
4. Configure the Integration settings:
   - **Syslog Host:** Set this to `0.0.0.0` to listen on all interfaces or provide a specific IP.
   - **Syslog Port:** Enter the port number (e.g., `514` or `5514`) that matches your FortiProxy configuration.
   - **Protocol:** Select `udp` or `tcp` to match the "Mode" set on the FortiProxy.
5. Under **Advanced options**, ensure the internal timezone settings match your device's log output if necessary.
6. Click **Save and continue**.
7. Select the **Agent Policy** where your Elastic Agent is enrolled and click **Save and deploy**.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from FortiProxy to the Elastic Stack.

### 1. Trigger Data Flow on Fortinet FortiProxy:
- **Generate Administrative Event:** Log out of the FortiProxy GUI and log back in, or enter and exit configuration mode in the CLI (e.g., `config system global` then `end`).
- **Trigger Traffic Event:** From a workstation that uses the FortiProxy as its gateway, browse to a few public websites to generate traffic logs.
- **Generate Web Filter Event:** Attempt to access a URL that is explicitly blocked by your FortiProxy security policy to trigger a "Deny" or "Block" log entry.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view or the specific integration data view.
3. Enter the following KQL filter: `data_stream.dataset : "fortinet_fortiproxy.log"`
4. Verify logs appear in the results table. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `fortinet_fortiproxy.log`)
   - `source.ip` and/or `destination.ip` (reflecting the traffic generated in step 1)
   - `event.action` or `event.outcome` (e.g., `allowed`, `denied`, or `blocked`)
   - `fortinet_fortiproxy.log.subtype` (verifying specific Fortinet data fields are parsed)
   - `message` (containing the raw syslog payload from the FortiProxy)
5. Navigate to **Analytics > Dashboards** and search for "Fortinet FortiProxy" to view the pre-built dashboards and confirm visualizations are populating with data.

# Troubleshooting

## Common Configuration Issues

- **Incorrect Source IP Binding**: By default, the FortiProxy may use the IP of the interface closest to the Elastic Agent. If there is an ACL or firewall rule expecting a specific Management IP, logs will be dropped. Use the CLI command `set source-ip` under `config log syslogd setting` to force the correct IP.
- **Syslog Port Conflict**: If the Elastic Agent fails to start the listener, another process might be using port 514. Check the Agent logs for "address already in use" errors and either stop the competing process or change the syslog port in both Kibana and the FortiProxy.
- **Policy Not Applied**: Changes in the FortiProxy GUI sometimes require an "Apply" click on multiple pages if you are also modifying firewall policies that allow the log traffic. Ensure the underlying firewall policy permits traffic from the proxy to the Elastic Agent on port 514.

## Ingestion Errors

- **Parsing Failures**: If logs appear in Kibana but contain the tag `_grokparsefailure` or `_jsonparsefailure`, check the `error.message` field. This often happens if the FortiProxy is using a non-standard log format or an unsupported version of the firmware.
- **Field Mapping Issues**: If specific fields like `source.ip` are missing, verify the raw `message` field to ensure the FortiProxy is sending logs in the expected key-value pair format (e.g., `srcip=192.168.1.1`).
- **Timezone Discrepancies**: If logs appear to be delayed or in the future, check the system time on both the FortiProxy and the Elastic Agent host to ensure they are synchronized via NTP.

## Vendor Resources

- [Configuring syslog service on Fortinet devices - ManageEngine](https://www.manageengine.com/log-management/help/data-source-configuration/network-devices/fortinet-devices.html)
- [Configure Fortinet FortiGate using the command line interface - Trellix](https://docs.trellix.com/bundle/enterprise-security-manager-data-source-configuration-guide/page/UUID-8a124f01-77ad-6f79-2aca-4879a6301fb3.html)

## Documentation sites

- [Configuring syslog service on Fortinet devices - ManageEngine](https://www.manageengine.com/log-management/help/data-source-configuration/network-devices/fortinet-devices.html)
- [Configure Fortinet FortiGate using the command line interface - Trellix](https://docs.trellix.com/bundle/enterprise-security-manager-data-source-configuration-guide/page/UUID-8a124f01-77ad-6f79-2aca-4879a6301fb3.html)
- Refer to the official vendor website for additional FortiProxy administration guides.
