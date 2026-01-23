# Service Info

## Common use cases

The Syslog Router integration acts as a centralized traffic controller for syslog data, enabling organizations to ingest multiple disparate log streams through a single port and automatically route them to the correct vendor-specific integration. This eliminates the need to manage dozens of different listeners for various firewall and network device vendors.

- **Multi-Vendor Log Consolidation:** Use this integration to collect logs from a mixed environment containing Cisco, Fortinet, and Palo Alto devices via a single TCP or UDP port, where the router identifies the source and applies the correct ingest pipeline.
- **Dynamic Data Stream Routing:** Automatically forward identified events to specialized data streams such as `cisco_asa.log` or `fortinet_fortigate.log` by matching unique regex patterns within the syslog message header or payload.
- **Custom Log Logic Implementation:** Define custom `if/then` logic blocks to handle unique or proprietary log formats that are not supported by default, ensuring that every event is either correctly categorized or sent to a generic log stream for further analysis.
- **Policy-Based Event Processing:** Prioritize high-traffic integrations or more specific matches by reordering configuration blocks, ensuring that critical security events are processed efficiently without being shadowed by generic patterns.

## Data types collected

This integration is designed to route logs to Elastic integrations, providing a centralized ingestion point for diverse syslog sources. It supports the collection of the following data types:

- **Network Security Logs:** Traffic, threat, and system logs from firewalls including Cisco ASA/FTD, FortiGate, Palo Alto, and Sonicwall.
- **System and Audit Logs:** Administrative actions, authentication events, and system health status from QNAP NAS and Arista NG Firewall.
- **Application Security Logs:** Web application firewall (WAF) alerts and protection events from Citrix WAF and Imperva SecureSphere.
- **Intrusion Detection System (IDS) Events:** Packet alerts and security signatures from Snort and Stormshield systems.
- **Data Formats Supported:** Standard Syslog (RFC3164/RFC5424), Common Event Format (CEF) for Citrix/Imperva, and various vendor-proprietary key-value formats.

This integration includes the following data streams:

- **Syslog events (TCP):** `log` (logs) - Collect Syslog events using the TCP input. This stream provides reliable delivery for high-integrity logging requirements.
- **Syslog events (UDP):** `log` (logs) - Collect Syslog events using the UDP input. This stream is optimized for high-performance, low-latency ingestion of high-volume network events.
- **Syslog events (Filestream):** `log` (logs) - Collect Syslog events using the Filestream input. This stream monitors local log files on disk, supporting efficient rotation and state tracking.

## Compatibility

- **Kibana Version:** This integration requires Kibana version **^8.14.3** or **^9.0.0**.
- **Elastic Subscription:** A **Basic** subscription or higher is required.
- **Vendor Support:** Supports routing to a wide array of third-party integrations including **Arista**, **Check Point**, **Cisco (ASA, FTD, ISE, SEG, IOS)**, **Fortinet**, **Palo Alto**, **Snort**, **Sophos**, and **Juniper**.

## Scaling and Performance

To ensure optimal performance in high-volume environments, consider the following:
- **Transport/Collection Considerations:** While UDP offers higher performance and lower latency for high-speed network logging, TCP is recommended for environments where delivery guarantees are required. TCP ensures no log messages are lost due to network congestion, though it may introduce backpressure on the source device during peak loads.
- **Data Volume Management:** Configure the source vendor appliances to forward only necessary events. To optimize CPU performance on the Agent, administrators should disable unused routing patterns by removing their `if/then` blocks from the configuration. Ensure high-volume traffic patterns (e.g., Cisco ASA or Fortinet) are placed at the top of the reroute configuration list to reduce regex evaluation overhead.
- **Elastic Agent Scaling:** For high-throughput environments, deploy multiple Elastic Agents behind a network load balancer to distribute traffic evenly. Place Agents close to the data source to minimize latency. Ensure the host system has sufficient CPU cores, as the router's intensive regular expression matching is a CPU-bound process.

# Set Up Instructions

## Vendor prerequisites

1. **Administrative Access:** Ensure you have access to the management console or CLI of the network devices (e.g., Cisco, Fortinet, Palo Alto) to configure remote syslog destinations.
2. **Network Visibility:** Verify a clear network path between the log-emitting devices and the host running the Elastic Agent. Firewalls must allow traffic on the configured syslog port (default **9514**).
3. **Target Integration Assets:** Before the router can successfully forward data, you must install the specific assets for target integrations (e.g., the **Cisco ASA** or **Check Point** integration) via the Kibana Integrations UI.
4. **Log Format Knowledge:** Identify the syslog format emitted by the vendor (e.g., BSD, RFC5424, or CEF) to ensure the **Reroute configuration** patterns match the incoming `message` correctly.

## Elastic prerequisites

- **Elastic Agent Enrollment:** The Elastic Agent must be installed and enrolled in a policy via Fleet.
- **Network Visibility:** The Elastic Agent must be hosted on a system that can reach the Elasticsearch cluster and is reachable by the vendor devices sending syslog data.
- **Kibana Version:** Ensure your Elastic Stack is running version 8.14.3 or higher.

## Vendor set up steps

### For Network-based (TCP/UDP) Collection:
1. Access the management interface (CLI or Web UI) of your network device (e.g., Cisco ASA, FortiGate).
2. Navigate to the **Logging** or **Syslog** configuration section.
3. Define a new remote syslog server or "Syslog Server" entry.
4. Set the **IP Address/Hostname** to the address of the Elastic Agent.
5. Set the **Port** to `9514` (or your custom configured port).
6. Select the **Protocol** (TCP or UDP) to match your Kibana input selection.
7. Set the **Log Format** to the vendor's standard format (e.g., for Citrix WAF or Imperva, ensure CEF format is enabled).
8. Save the configuration and apply or deploy the changes.

### For File-based (Filestream) Collection:
1. Configure your local syslog daemon (rsyslog or syslog-ng) to write incoming logs to a specific file.
2. Ensure the log file is located at `/var/log/syslog.log` or update the integration `paths` variable to match your custom location.
3. Verify that the user running the Elastic Agent has read permissions for the specified log file.
4. Configure log rotation for the file to prevent disk exhaustion, ensuring the Agent is capable of tracking the rotated files.

## Kibana set up steps

### Route logs using the TCP input
1. In Kibana, navigate to **Integrations > Syslog Router**.
2. Click **Add Syslog Router**.
3. Under the **Collect syslog events** policy template, locate the **Route logs using the TCP input** section.
4. Configure the following variables:
    - **Listen Address** (`listen_address`): The bind address to listen for TCP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
    - **Listen Port** (`listen_port`): The TCP port number to listen on. Default: `9514`.
    - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
    - **Reroute configuration** (`reroute_config`): A list of YAML-based `if/then` rules that use `regexp.message` to identify logs and `add_fields` to set the target `_conf.dataset` (e.g., `cisco_asa.log`, `checkpoint.firewall`, `fortinet_fortigate.log`).
5. Save the integration.

### Route logs using the UDP input
1. In Kibana, navigate to **Integrations > Syslog Router**.
2. Click **Add Syslog Router**.
3. Under the **Collect syslog events** policy template, locate the **Route logs using the UDP input** section.
4. Configure the following variables:
    - **Listen Address** (`listen_host`): The bind address to listen for UDP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
    - **Listen Port** (`listen_port`): The UDP port number to listen on. Default: `9514`.
    - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
    - **Reroute configuration** (`reroute_config`): Define the list of configurations for rerouting logs based on regex matching of the message payload.
5. Save the integration.

### Route logs using the filestream input
1. In Kibana, navigate to **Integrations > Syslog Router**.
2. Click **Add Syslog Router**.
3. Under the **Collect syslog events** policy template, locate the **Route logs using the filestream input** section.
4. Configure the following variables:
    - **Paths** (`paths`): List of absolute paths to the log files to monitor for syslog events. Default: `['/var/log/syslog.log']`.
    - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
    - **Reroute configuration** (`reroute_config`): Define the list of configurations for rerouting file-based logs to their respective integration datasets.
5. Save the integration.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on [Vendor]:
- **Authentication event:** Log out and log back into the administration interface of your firewall or network device to create an audit log.
- **Configuration change:** Enter configuration mode on the device (e.g., `config t` on Cisco or similar) and make a minor change then exit to trigger a syslog event.
- **Traffic generation:** Generate network traffic through the device (e.g., ping a remote host or browse a website) to trigger traffic session logs.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "syslog_router.log"`
4. Verify logs appear. Expand a log entry and confirm these fields:
   - `event.dataset` (should be `syslog_router.log`)
   - `source.ip` and/or `destination.ip` (should reflect the traffic source)
   - `event.action` or `event.outcome`
   - `message` (the raw log payload)
5. Navigate to **Analytics > Dashboards** and search for "Syslog Router" or the specific vendor dashboard (e.g., "Cisco ASA").

# Troubleshooting

## Common Configuration Issues

- **Incorrect Pattern Order**: If logs are being routed to the wrong integration or the generic `log` stream, check the YAML `reroute_config`. More generic regex patterns (like those for Cisco IOS) can "shadow" specific ones (like Cisco ASA) if placed higher in the list.
- **Downstream Assets Not Installed**: If logs are correctly identified (the `_conf.dataset` field is set) but not parsed or visualized, verify that you have manually installed the assets for the target integration (e.g., "Install Cisco ASA assets") via the Integrations Settings tab.
- **Port Bind Failure**: If the Elastic Agent fails to start the input, check for port conflicts. Ensure no other service is using port 9514 and that the Agent has permissions to bind to privileged ports if configured below 1024.
- **Listen Address Mismatch**: If the Agent is listening on `localhost` but the vendor devices are on a remote network, the connection will be refused. Ensure `Listen Address` is set to `0.0.0.0`.

## Ingestion Errors

- **Regex Mismatch**: Vendor software updates can sometimes change syslog headers. If logs appear in the generic `log` stream, compare the `message` field against the regex patterns in the configuration. You may need to update the `regexp.message` value to accommodate the new format.
- **CEF Parsing Failures**: For Citrix or Imperva, ensure the `decode_cef` processor is included in the configuration block. If the message is not valid CEF, the processor will fail and the event may not be correctly indexed.
- **Missing Required Fields**: Ensure every `then` block in your custom YAML includes the `add_fields` processor setting the `_conf.dataset` field. Without this field, the routing mechanism will default to the generic log stream.
- **Identify via error.message**: Check for an `error.message` field in Discover. Common errors include `provided condition is invalid` which indicates a syntax error in your YAML configuration.

## Vendor Resources

- Refer to the official vendor website for additional resources.

# Documentation sites

- Refer to the official vendor website for additional resources.
