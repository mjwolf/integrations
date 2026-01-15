# Service Info

The UDP integration allows for the seamless ingestion of log data and events sent over the User Datagram Protocol (UDP). This is a foundational integration used to collect data from a wide variety of sources that support network logging but may not have a dedicated, product-specific Elastic integration available.

## Common use cases

The UDP integration for Elastic Agent provides a high-performance, connectionless mechanism for ingesting log data from a wide variety of network-attached devices and applications. It acts as a versatile listener that can be configured to capture and normalize data streams in environments where low latency is a priority.
- **Network Infrastructure Monitoring:** Efficiently capturing high-velocity logs from routers, switches, and firewalls that utilize Syslog over UDP to track traffic patterns and security events.
- **IoT and Telemetry Collection:** Collecting lightweight status updates and telemetry from Internet of Things (IoT) devices that use UDP to minimize communication overhead and power consumption.
- **Legacy Application Integration:** Bridging the gap for older software systems or proprietary applications that are hardcoded to broadcast diagnostic information or audit trails via UDP datagrams.
- **High-Throughput Security Logging:** Ingesting security events from Intrusion Detection Systems (IDS) or endpoint security tools where immediate visibility of events is required.

## Data types collected

This integration can collect the following types of data:
- **Syslog Messages:** Standardized system logs including facility, severity, and message content (RFC 3164 or RFC 5424).
- **Network Events:** Connection logs, interface status changes, and security alerts from network infrastructure.
- **Application Logs:** Plain text or structured (JSON) log data emitted by services over network sockets.
- **Security Formats:** Common Event Format (CEF) or Log Event Extended Format (LEEF) messages often used by security appliances.
- **Raw Payloads:** The integration captures the raw content of the UDP packet, typically mapped to the `message` field for further processing via ingest pipelines.

## Compatibility

The **UDP Integration** is compatible with any source capable of sending data over the UDP protocol. This includes:
- **Unix/Linux Distributions:** Any system running **rsyslog** or **syslog-ng**.
- **Network Hardware:** Devices from vendors like Cisco, Juniper, and Arista.
- **Elastic Agent:** Requires Elastic Agent version **7.14.0** or higher for stable UDP input support.

## Scaling and Performance

- **Transport/Collection Considerations:** UDP is a connectionless "fire and forget" protocol, which offers significantly lower latency and overhead compared to TCP. However, it provides no delivery guarantees; in congested networks or high-load scenarios, packets may be dropped without notification to the sender. To mitigate this, ensure the Elastic Agent host has sufficient network bandwidth and that the OS-level UDP receive buffers are tuned to handle bursts of incoming traffic.
- **Data Volume Management:** Because UDP does not support backpressure, high-volume sources can easily overwhelm a single receiver. Users should implement filtering at the source whenever possible, such as adjusting log levels to "Warning" or "Error" instead of "Debug." Additionally, consider utilizing internal load balancers to distribute UDP traffic across multiple Elastic Agent instances to prevent packet loss during peak periods.
- **Elastic Agent Scaling:** A single Elastic Agent can handle substantial UDP throughput depending on the complexity of the ingest pipeline. For high-volume environments (exceeding 10,000 events per second), it is recommended to deploy multiple Agents in a Fleet policy and use a Round Robin DNS or a dedicated hardware load balancer. Ensure that the host machine is provisioned with multiple CPU cores to allow the Agent to process concurrent datagram streams effectively.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have administrative or root-level access to the source device or application to modify its logging and network configuration.
- **Network Routing:** Ensure that the source device has a clear network path to the Elastic Agent's IP address and that no intermediary firewalls are blocking UDP traffic on the selected port.
- **Source Configuration Capability:** The vendor system must support remote log forwarding, specifically allowing the definition of a remote host IP and a destination UDP port.
- **Documentation Access:** Access to the vendor's technical manual to identify the specific log format (e.g., CEF, LEEF, or Standard Syslog) being transmitted.
- **Port Availability:** Identify an available UDP port on the Elastic Agent host that is not currently bound to another service (standard ports are often 514 or 9000-9500).

## Elastic prerequisites

- **Elastic Agent Status:** An Elastic Agent must be installed and successfully enrolled in Fleet.
- **Policy Management:** You must have permissions to edit Agent Policies in Kibana to add and configure the "Custom Logs" integration.
- **Connectivity:** The Elastic Agent host must be reachable from the data source on the specific UDP port configured in the integration settings.

## Vendor set up steps

### For Log Collection via rsyslog:

Follow these steps to configure a standard Linux server to forward logs to the Elastic Agent.

1.  **Access the Source Server:** Log in to the server from which you wish to collect logs using an SSH client.
2.  **Create Configuration File:** Create a dedicated configuration file for Elastic forwarding to keep the main configuration clean.
    ```bash
    sudo nano /etc/rsyslog.d/90-elastic-udp.conf
    ```
3.  **Define the Forwarding Rule:** Add the following configuration block. This rule instructs rsyslog to forward all system logs to the Agent.
    ```text
    # Forward all messages to a remote server using UDP.
    action(
        type="omfwd"
        protocol="udp"
        target="<elastic-agent-ip>"
        port="<port>"
    )
    ```
    *Replace `<elastic-agent-ip>` with your Agent's IP and `<port>` with your configured port.*
4.  **Save the Changes:** Press `Ctrl+O` to write the file and `Ctrl+X` to exit the editor.
5.  **Test Configuration Syntax:** Before restarting the service, verify that there are no syntax errors in your new file.
    ```bash
    sudo rsyslogd -N1
    ```
6.  **Apply Configuration:** Restart the rsyslog daemon to begin forwarding logs.
    ```bash
    sudo systemctl restart rsyslog
    ```
7.  **Confirm Service Status:** Ensure rsyslog is running correctly after the restart.
    ```bash
    sudo systemctl status rsyslog
    ```

## Kibana set up steps

1. In Kibana, navigate to **Management > Integrations**.
2. Search for the **UDP** integration and select it.
3. Click on **Add UDP**.
4. Configure the integration settings:
    - **Listen Address:** Enter `0.0.0.0` to listen on all available network interfaces or specify a specific local IP.
    - **Listen Port:** Enter the port number that matches your vendor configuration (e.g., `9000`).
    - **Dataset Name:** Provide a descriptive name for the data stream (e.g., `generic_udp_logs`).
5. Expand the **Advanced options** to adjust internal buffer sizes if you expect high-volume traffic.
6. Scroll down and select the **Existing host policy** where your Elastic Agent is enrolled.
7. Click **Save and continue**, then click **Add agent integration** to deploy the configuration to your Agents.
8. Wait a few moments for the Agent to receive the updated policy and start the UDP listener.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from the source to the Elastic Stack.

### 1. Trigger Data Flow on Vendor:
- **Generate Test Log Entry:** Use a tool like `netcat` from a remote system to simulate a log: `echo "Test UDP Message from Vendor" | nc -u -w1 <AGENT_IP> 9000`.
- **System Action:** Perform a configuration change on the network hardware, such as entering and exiting the configuration terminal (`config t` followed by `exit`).
- **Restart Service:** Restart a monitored application service to trigger "Service Started" or "Initialization" logs.
- **Toggle Interface:** If testing a network switch, perform a `shutdown` and `no shutdown` on a test interface to generate status change logs.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view or create one specifically for the integration.
3. Enter the following KQL filter: `data_stream.dataset : "udp.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `udp.log`)
   - `source.ip` (should match the IP of your vendor device or test machine)
   - `udp.port` (should match the port configured in the integration)
   - `message` (containing the raw log payload or the test string sent)
   - `event.ingested` (confirming the timestamp of arrival)
5. Navigate to **Analytics > Dashboards** and search for "UDP" to view any pre-built visualizations or use the "Lens" tool to create a quick chart of `source.ip` counts.

# Troubleshooting

## Common Configuration Issues

- **Firewall Blocking Traffic:** If no logs appear, check the local firewall on the Elastic Agent host (e.g., `ufw` or `firewalld`). Ensure the specific UDP port is allowed. Run `sudo tcpdump -i any udp port <port>` on the Agent host to see if packets are reaching the interface.
- **Incorrect Binding Address:** If the Elastic Agent is configured to listen on `127.0.0.1`, it will only accept logs from itself. Ensure the **Listen Host** is set to `0.0.0.0` or the specific LAN IP of the Agent machine.
- **Port Conflicts:** Ensure no other service (like a native rsyslog listener on the Agent host) is already using the configured UDP port. Use `sudo netstat -ulnp | grep <port>` to check for existing bindings.

## Ingestion Errors

- **Message Truncation**: If log messages appear cut off or incomplete in Kibana, the `max_message_size` may be too small for the incoming packets. Increase this value in the YAML configuration (e.g., to `20KiB` or `50KiB`) and redeploy the policy.
- **Parsing Failures**: If logs appear in the `message` field but are not being parsed into structured fields, ensure that the format sent by the vendor matches the expected structure. You may need to add custom ingest pipelines in Elasticsearch to handle specific vendor formats.
- **High Packet Loss**: If significant data gaps are observed, check the operating system's UDP receive buffer size. You may need to increase the kernel's `net.core.rmem_max` and `net.core.rmem_default` settings to handle high-burst traffic.
- **Identifying Issues in Kibana**: Check for the presence of the `error.message` or `tags` field (e.g., `_grokparsefailure`) in Discover to identify logs that failed to process correctly through the ingest pipeline.

## Vendor Resources

- [https://www.rsyslog.com/doc/getting_started/forwarding_logs.html](https://www.rsyslog.com/doc/getting_started/forwarding_logs.html)
- [https://rsyslog.readthedocs.io/en/latest/getting_started/beginner_tutorials/06-remote-server.html](https://rsyslog.readthedocs.io/en/latest/getting_started/beginner_tutorials/06-remote-server.html)

## Documentation sites

- [https://www.rsyslog.com/doc/getting_started/forwarding_logs.html](https://www.rsyslog.com/doc/getting_started/forwarding_logs.html)
- [https://rsyslog.readthedocs.io/en/latest/getting_started/beginner_tutorials/06-remote-server.html](https://rsyslog.readthedocs.io/en/latest/getting_started/beginner_tutorials/06-remote-server.html)
- Refer to the official vendor website for additional resources.
