# Service Info

## Common use cases

*   **Centralized Linux Logging:** Forwarding system and application logs from multiple Linux distributions to Elastic using `rsyslog` or `syslog-ng` over a reliable TCP connection.
*   **Custom Application Integration:** Ingesting real-time event data from proprietary applications that write logs directly to a network socket instead of a local file.
*   **Legacy Network Device Monitoring:** Collecting log data from routers, firewalls, and switches that support remote logging via TCP for improved reliability over UDP.

## Data types collected

*   **Logs:** Generic log data in plain text or JSON format sent via the TCP stream.
*   **Events:** Structured or unstructured event notifications from network services and devices.

## Compatibility

*   **Protocol:** Transmission Control Protocol (TCP) as defined in RFC 793.
*   **Clients:** Compatible with any system capable of establishing a TCP connection and streaming text (e.g., `rsyslog`, `syslog-ng`, Logstash, custom Python/Java/Go loggers).
*   **Elastic Agent:** Supported on all platforms where Elastic Agent is available.

## Scaling and Performance

*   **Reliability:** TCP ensures delivery through acknowledgment packets, making it more reliable than UDP for high-importance log data.
*   **Throughput:** Performance is primarily limited by network bandwidth and the Elastic Agent's processing power; however, TCP can experience head-of-line blocking if the network is congested.
*   **Framing:** Performance is optimized when using newline-delimited framing, allowing the Agent to efficiently split and process events in parallel.

# Set Up Instructions

## Vendor prerequisites

*   **Network Access:** The source device must have network connectivity to the Elastic Agent on the specified TCP port.
*   **Logging Permissions:** Administrative access to the source system to modify logging configuration files (e.g., `/etc/rsyslog.conf`) or management interfaces.
*   **Formatting Capability:** Ability to configure the client to append a newline character (`\n`) to each log message to ensure proper event separation.

## Elastic prerequisites

*   **Elastic Agent:** Must be installed and enrolled in Fleet or configured as a standalone agent.
*   **Integration Policy:** An Elastic Agent policy must include the "TCP" integration.
*   **Firewall Configuration:** Ensure the host machine running the Elastic Agent allows inbound traffic on the configured TCP port (e.g., port 9000).

## Vendor set up steps

1.  **Access Logging Configuration:** Log in to the source application or server. For Linux systems, open the logging configuration file (e.g., `/etc/rsyslog.conf` or `/etc/syslog-ng/syslog-ng.conf`).
2.  **Define Remote Target:** Configure the application to use the TCP protocol for remote logging.
3.  **Specify Destination:** Enter the IP address or Hostname of the Elastic Agent and the specific Port number configured in the integration.
4.  **Configure Framing:** Ensure the output is set to "newline-delimited." If using `rsyslog`, use the `@@` prefix to specify TCP:
    ```bash
    *.* @@[Elastic_Agent_IP]:[Port]
    ```
5.  **Restart Service:** Save the configuration and restart the logging service (e.g., `sudo systemctl restart rsyslog`) to apply changes.

## Kibana set up steps

1.  **Add Integration:** In Kibana, go to **Management > Integrations** and search for **TCP**.
2.  **Configure Input:** Click **Add TCP**, then define the **Listen Host** (default is `0.0.0.0` to listen on all interfaces) and the **Listen Port**.
3.  **Select Policy:** Choose the existing Agent policy or create a new one to apply this integration.
4.  **Save and Deploy:** Click **Save and continue** to push the updated configuration to the managed Elastic Agents.

# Validation Steps

1.  **Check Port Status:** On the Elastic Agent host, verify the port is listening using `netstat -an | grep <PORT_NUMBER>`.
2.  **Test Connection:** From the source device, use a tool like `telnet` or `nc` to verify connectivity:
    ```bash
    echo "Test log message" | nc [Elastic_Agent_IP] [Port]
    ```
3.  **Monitor Logs Stream:** In Kibana, navigate to **Observability > Logs > Stream** and filter by `event.dataset: "tcp"` or the configured service name to see incoming data.

# Troubleshooting

## Common Configuration Issues

*   **Connection Refused:** Ensure the Elastic Agent is running and the TCP port configured in the policy matches the port the client is sending to.
*   **Firewall Blocking:** Check both the source and destination host firewalls to ensure traffic is permitted on the chosen TCP port.
*   **Binding Errors:** If the Agent fails to start the TCP input, ensure no other service is already using the specified port on that host.

## Ingestion Errors

*   **Message Merging:** If multiple logs appear as a single entry in Kibana, the client is likely not sending a newline character (`\n`) at the end of each message.
*   **Encoding Issues:** Ensure the source is sending data in UTF-8 encoding. Non-standard encodings may result in `error.message` fields or garbled text.
*   **Truncated Messages:** For very large log lines, check the "Max Message Size" setting in the integration configuration; messages exceeding this limit may be dropped or truncated.

## API Authentication Errors

*   **Not specified:** The TCP integration generally uses socket-level transmission and does not involve API authentication at the protocol level. For Fleet-related authentication issues, check the Elastic Agent logs.

## Vendor Resources

*   **rsyslog Documentation:** [https://www.rsyslog.com/doc/](https://www.rsyslog.com/doc/)
*   **syslog-ng Documentation:** [https://www.syslog-ng.com/technical-documents](https://www.syslog-ng.com/technical-documents)
*   **Elastic Community Forums:** [https://discuss.elastic.co/](https://discuss.elastic.co/)

# Documentation sites

*   **TCP Input Reference:** [https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-input-tcp.html](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-input-tcp.html)
*   **RFC 793 (TCP Specification):** [https://datatracker.ietf.org/doc/html/rfc793](https://datatracker.ietf.org/doc/html/rfc793)
*   **Elastic Agent Integration Guide:** [https://www.elastic.co/docs/current/integrations/tcp](https://www.elastic.co/docs/current/integrations/tcp)