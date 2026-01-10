# Service Info

## Common use cases

- **Network Device Monitoring:** Collecting standard syslog data from firewalls, switches, and routers that use UDP to export event logs for security auditing.
- **Legacy Application Logging:** Ingesting logs from older or custom-built internal applications that transmit semi-structured data via UDP datagrams to minimize overhead.
- **IoT and Embedded Systems:** Receiving status updates or sensor data from resource-constrained devices where the overhead of TCP or HTTP is prohibitive for continuous transmission.

## Data types collected

This integration collects log data and event streams sent over the User Datagram Protocol (UDP), which can include raw text, JSON, or syslog-formatted messages.

## Compatibility

- **Elastic Agent:** Version 7.14.0 or higher is recommended for full Fleet management support.
- **Protocols:** Compatible with any system capable of sending data over standard UDP datagrams, including RFC 3164 and RFC 5424 syslog formats.

## Scaling and Performance

UDP is connectionless and does not support flow control, which may lead to packet loss during high-volume bursts if the host network stack or Elastic Agent cannot process data fast enough. To scale, increase the operating system's maximum UDP receive buffer (e.g., `sysctl -w net.core.rmem_max`) and ensure the Elastic Agent host has sufficient CPU resources for the ingestion volume.

# Set Up Instructions

## Vendor prerequisites

- **Network Access:** Ensure that the source systems sending data have network connectivity to the Elastic Agent host via the specified UDP port.
- **Firewall Permissions:** UDP traffic must be explicitly allowed through any intermediate firewalls, cloud security groups, or local host firewalls (e.g., `iptables` or `ufw`).

## Elastic prerequisites

- **Fleet Management:** An active Fleet environment with at least one Elastic Agent enrolled.
- **Agent Policy:** A policy must be created and assigned to the agent that will act as the UDP listener.
- **Permissions:** User must have `all` or `write` permissions for Fleet and Integrations in Kibana.

## Vendor set up steps

1. Access the configuration interface or configuration file (e.g., `/etc/rsyslog.conf` or `/etc/syslog-ng/syslog-ng.conf`) of the system sending the logs.
2. Locate the remote logging or log forwarding section of the vendor configuration.
3. Configure the destination IP address to match the host running the Elastic Agent.
4. Set the destination port to match the port configured in the Elastic "Custom UDP Logs" integration (default is often 9002).
5. Restart the logging service on the vendor system to initiate the UDP stream.

## Kibana set up steps

1. Log in to Kibana and navigate to **Management > Fleet > Agent policies**.
2. Select the policy for the agent that will receive the logs and click **Add integration**.
3. Search for and select **Custom UDP Logs**, then click **Add Custom UDP Logs**.
4. Configure the **Listen Address** (e.g., `0.0.0.0` for all interfaces) and the **Listen Port** (e.g., `9002`).
5. Specify the **Data Stream Namespace** (e.g., `default`) to organize your incoming data.
6. If applicable, enable **Syslog Options** to parse standard syslog fields automatically.
7. Click **Save and continue** to deploy the configuration to the agents.

# Validation Steps

1. **Verify Agent Status:** Check the **Fleet** UI to ensure the agent assigned to the policy is showing a "Healthy" status.
2. **Check Port Binding:** On the agent host, run `netstat -ulnp | grep <port>` to verify the Elastic Agent is successfully listening on the configured UDP port.
3. **Trigger Test Event:** Use a tool like `nc` (netcat) to send a test message: `echo "test message" | nc -u -w1 <Agent_IP> <Port>`.
4. **Inspect Data:** Navigate to **Discover** in Kibana and search the `logs-udp.*` data stream to confirm the test message appears.

# Troubleshooting

## Common Configuration Issues

- **Port Conflicts:** If the agent fails to start, check if another service (like a native syslog daemon) is already using the configured UDP port.
- **Binding Failures:** Ensure the **Listen Address** is set to `0.0.0.0` or a valid local IP address reachable by the source devices.
- **Firewall Blocks:** Verify that local firewalls or cloud security groups are not dropping incoming UDP traffic on the listener port.

## Ingestion Errors

- **Malformed Syslog:** If syslog messages are not being parsed correctly, verify the `Syslog Options` match the RFC version used by the vendor (RFC 3164 vs RFC 5424).
- **Truncated Messages:** Large UDP packets exceeding the default buffer size may be truncated; check if the vendor allows adjusting the packet size or if the agent needs custom `max_message_size` settings.

## API Authentication Errors

- **Policy Permissions:** If the integration fails to deploy, ensure the Fleet Server has the necessary API keys and permissions to push configuration updates to the agents.
- **Enrollment Issues:** Verify the agent is correctly enrolled in Fleet and has not lost its connection to the Elastic stack.

## Vendor Resources

- [Elastic Support Portal](https://www.elastic.co/support)
- [Elastic Community Forums](https://discuss.elastic.co/)
- [UDP Networking Troubleshooting (Linux)](https://www.redhat.com/sysadmin/benchmarking-udp)

# Documentation sites

- [Custom UDP Log integration | Elastic Documentation](https://www.elastic.co/docs/reference/integrations/udp)
- [Fleet and Elastic Agent Guide](https://www.elastic.co/guide/en/fleet/current/index.html)
- [Elastic Integration Reference](https://www.elastic.co/integrations)