# Service Info

The **Custom UDP Logs** integration is designed to ingest data from any source capable of sending log messages over the User Datagram Protocol (UDP). It functions by initializing a listening socket on the Elastic Agent host, allowing for high-performance, low-overhead data collection from a wide variety of network equipment, custom applications, and legacy systems.

## Common use cases
The Custom UDP Logs integration is designed to provide a flexible ingestion point for any network-based log data transmitted via the User Datagram Protocol (UDP). This is particularly useful for legacy systems and network hardware that do not support modern transport protocols or encrypted tunnels.
- **Network Device Monitoring:** Capture logs from routers, switches, and firewalls (such as MikroTik or Cisco devices) that export system events and traffic logs via UDP syslog.
- **Legacy Application Integration:** Collect diagnostic information from older applications that are hardcoded to broadcast status messages or errors to a specific UDP network socket.
- **IoT and Sensor Data Ingestion:** Aggregate telemetry from lightweight IoT devices or sensors that utilize UDP for low-overhead communication of status updates.
- **Centralized Syslog Aggregation:** Act as a high-performance syslog receiver for RFC3164 or RFC5424 formatted messages, enabling automatic parsing and normalization of security events across the infrastructure.

## Data types collected

This integration can collect the following types of data:

- **Generic UDP Traffic:** Any raw data sent to the configured UDP port is captured and stored as a document in Elasticsearch.
- **Standardized Syslog Data:** When enabled, the integration automatically parses logs adhering to **RFC3164** (BSD syslog) and **RFC5424** (IETF syslog) formats.
- **Structured JSON Logs:** While raw by default, incoming JSON payloads can be processed via custom ingest pipelines for structured analysis.
- **Security and Audit Events:** Captures authentication logs, configuration changes, and system alerts forwarded from remote network devices.

The integration utilizes the following data stream:
- **udp.generic (logs):** The default data stream used to store all ingested UDP traffic as individual log documents.

## Compatibility

The **Custom UDP Logs** integration is compatible with the following versions and requirements:
- **Elastic Stack:** Requires Kibana version **9.2.0** or higher.
- **Elasticsearch:** Version **9.2.0** or later is required if utilizing the `use_logs_stream` feature.
- **Operating Systems:** Compatible with all major Linux distributions (RHEL, Ubuntu, Debian) and Windows environments supported by the Elastic Agent.

## Scaling and Performance

- **Transport/Collection Considerations:** UDP is a connectionless, "fire-and-forget" protocol. This makes it significantly faster than TCP as it lacks the overhead of three-way handshakes and acknowledgement packets. However, this comes at the cost of reliability; in congested networks, UDP packets may be dropped without notification. For high-reliability requirements, ensure the network path between the source and the Elastic Agent is stable and has sufficient bandwidth.
- **Data Volume Management:** To prevent overwhelming the Elastic Stack or the host running the Elastic Agent, it is recommended to filter data at the source. Configure your vendor devices to send only relevant log facilities (e.g., auth, security, system) and severity levels (e.g., Error, Critical, Warning) rather than "Informational" or "Debug" levels unless troubleshooting. High data volumes can impact CPU utilization on the Agent host during syslog parsing.
- **Elastic Agent Scaling:** A single Elastic Agent can handle thousands of UDP events per second, but performance depends on the complexity of parsing (e.g., enabling `syslog` parsing or custom pipelines). In high-volume environments, deploy multiple Elastic Agents behind a network load balancer to distribute the incoming UDP traffic. Ensure the host system has adequate CPU resources and that the OS-level UDP buffer sizes are tuned to prevent packet loss at the kernel level.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have administrative or configuration-level access to the source device (firewall, switch, or application) to modify remote logging settings.
- **Network Connectivity:** The source device must be able to reach the Elastic Agent host over the network.
- **Open UDP Port:** The specific UDP port chosen for the integration (default: **8080**) must be open on the host's local firewall (e.g., iptables, ufw, or Windows Firewall).
- **Protocol Knowledge:** You must know the IP address of the Elastic Agent and decide whether to use a standard syslog format or raw text.
- **Licensing:** Ensure any required "Remote Logging" or "Syslog Forwarding" licenses are active on your vendor hardware if applicable.

## Elastic prerequisites

- **Elastic Agent:** An active Elastic Agent must be installed and enrolled in Fleet.
- **Policy Configuration:** Access to the Kibana Fleet UI to add the UDP integration to an existing or new Agent Policy.
- **Network Security:** Host-based firewalls (like `ufw` or `firewalld`) on the Agent machine must be configured to allow inbound UDP traffic on the specified port.

## Vendor set up steps

### For General UDP/Syslog Forwarding:

1. **Log in to the management interface** of your source device (e.g., via Web UI, SSH, or Console).
2. **Navigate to the Logging Configuration** section. This is typically located under **System**, **Settings**, or **Monitoring**.
3. **Add a new Remote Syslog Server** or Logging Target.
4. **Configure the Destination IP**: Enter the IP address of the host where the Elastic Agent is running.
5. **Configure the Port**: Enter the port number configured in Kibana (default is **8080**).
6. **Select the Protocol**: Choose **UDP**.
7. **Choose the Format**: If the integration's `syslog` option is enabled, select **RFC3164 (BSD)** or **RFC5424 (IETF)**.
8. **Select Log Content**: Choose which facilities (e.g., System, Kernel, Auth) and severity levels you wish to forward.
9. **Save and Apply** the settings. Some devices may require a service restart or a "Commit" action.

### For Cisco ISE Collection:

1. Log in to the Cisco ISE administration portal.
2. Navigate to **Administration > System > Logging > Remote Logging Targets**.
3. Click **Add** and create a target pointing to the Elastic Agent's IP and UDP port.
4. Go to **Logging Categories** and map the desired categories to the new Remote Logging Target.
5. Save the configuration to begin forwarding logs.

### For Palo Alto Networks Collection:

1. Log in to the PAN-OS web interface.
2. Navigate to **Device > Server Profiles > Syslog**.
3. Click **Add** and enter a name for the profile.
4. Click **Add** in the Server section and enter the Elastic Agent IP, select **UDP**, and the configured port.
5. Go to **Objects > Log Forwarding** to create a profile that uses this syslog server for specific traffic or threat logs.
6. **Commit** the changes.

## Kibana set up steps

### Custom UDP Logs
1. Open Kibana and navigate to **Management > Integrations**.
2. Search for **Custom UDP Logs** and select it.
3. Click **Add Custom UDP Logs**.
4. Configure the following fields in the integration settings:
   - **Integration name**: Provide a unique name for this integration instance, such as `network-udp-listener`.
   - **Listen Address** (`listen_address`): The address the agent binds to. Use `0.0.0.0` to listen on all interfaces. Default: `localhost`.
   - **Listen Port** (`listen_port`): The UDP port to listen on. Default: `8080`.
   - **Dataset Name** (`data_stream.dataset`): The name of the target dataset. This cannot contain hyphens. Default: `udp.generic`.
   - **Max Message Size** (`max_message_size`): The maximum size for a single UDP message. Default: `10KiB`.
   - **Syslog Parsing** (`syslog`): Enable this checkbox to automatically parse RFC3164 and RFC5424 syslog data.
   - **Custom Pipeline** (`pipeline`): (Optional) Enter the ID of a custom ingest pipeline to process the logs.
5. Select the **Agent Policy** where you want to apply this integration.
6. Click **Save and continue**, then **Save and deploy changes** to push the configuration to your Elastic Agents.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from the log source to the Elastic Stack.

### 1. Trigger Data Flow on Custom UDP Logs:

- **Generate a manual test event:** From a machine that can reach the Elastic Agent, use a tool like `netcat` (nc) to send a test string: `echo "Test UDP Log Message" | nc -u -w1 <AGENT_IP> 8080`.
- **Generate a syslog event:** Use the `logger` command on a Linux system to simulate a syslog message: `logger -n <AGENT_IP> -P 8080 "This is a test syslog message over UDP"`.
- **Trigger a system event:** If configured on a network device, log out and log back into the device's administration interface to generate an authentication event.
- **Change Configuration:** Enter and exit the configuration mode on your network device to trigger a "Config Changed" syslog message.

### 2. Check Data in Kibana:

1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the following KQL filter: `data_stream.dataset : "udp.generic"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `udp.generic`)
   - `log.syslog.priority` (if syslog parsing is enabled)
   - `message` (containing the raw log payload or the test string sent)
   - `input.type` (should be `udp`)
   - `agent.id` and `host.name` (identifying the collecting Agent)
5. Navigate to **Analytics > Dashboards** and search for "UDP" to view any pre-built visualizations or data summaries.

# Troubleshooting

## Common Configuration Issues

- **Port Binding Conflicts**: If the Elastic Agent fails to start the UDP listener, check if another process is already using the configured port (default 8080). Use `netstat -ano | grep 8080` (Linux) or `Get-NetTCPConnection` (Windows) to identify conflicting services.
- **Firewall Restrictions**: If the Agent is running but no logs appear, ensure the host firewall and any intermediate network firewalls allow UDP traffic on the specified port.
- **Binding to Localhost**: If `listen_address` is set to `localhost` or `127.0.0.1`, the Agent will only accept logs from its own host. Change this to `0.0.0.0` to accept logs from remote network devices.
- **Privileged Ports**: On Linux, the Elastic Agent cannot bind to ports below 1024 (like the standard syslog port 514) unless it is running as root or has specific capabilities (`CAP_NET_BIND_SERVICE`) granted to the binary.

## Ingestion Errors
- **Truncated Messages**: If log messages appear cut off in Kibana, the source is sending packets larger than the `max_message_size` setting. Increase this value in the integration configuration to accommodate larger datagrams.
- **Parsing Failures**: If the `syslog` option is enabled but fields like `log.syslog.severity` are missing, the incoming data may not strictly adhere to RFC3164 or RFC5424. Check the `error.message` field in the log document to identify parsing issues.
- **Missing Custom Pipeline**: If you specified a `pipeline` ID but the data is not being processed as expected, verify that the pipeline exists in **Stack Management > Ingest Pipelines** and that there are no syntax errors in its processors.

## Vendor Resources

- [Configure External Syslog Server on ISE - Cisco](https://www.cisco.com/c/en/us/support/docs/security/identity-services-engine/222223-configure-external-syslog-server-on-ise.html)
- [Configure Syslog Monitoring - Palo Alto Networks](https://docs.paloaltonetworks.com/pan-os/11-1/pan-os-admin/monitoring/use-syslog-for-monitoring/configure-syslog-monitoring)

## Documentation sites

- [Configure External Syslog Server on ISE - Cisco](https://www.cisco.com/c/en/us/support/docs/security/identity-services-engine/222223-configure-external-syslog-server-on-ise.html)
- [Configure Syslog Monitoring - Palo Alto Networks](https://docs.paloaltonetworks.com/pan-os/11-1/pan-os-admin/monitoring/use-syslog-for-monitoring/configure-syslog-monitoring)
- Refer to the official vendor website for additional resources.
