# Service Info

## Common use cases
The HPE Aruba CX integration is designed to provide comprehensive visibility into the health and security of HPE Aruba Networking CX Switches. By collecting and parsing AOS-CX event logs, administrators can maintain high availability and security posture across their network infrastructure.
- **Security Audit and Compliance:** Monitor AAA (Authentication, Authorization, and Accounting) events to track user access, command execution, and unauthorized login attempts across the network fabric.
- **Routing and Protocol Troubleshooting:** Analyze BGP, OSPF (v2/v3), and PIM events in real-time to identify route flapping, adjacency changes, or multicast distribution issues that could impact network performance.
- **Hardware Reliability Monitoring:** Proactively detect hardware failures by monitoring fan speeds, temperature sensors, power supply status, and ASIC resource utilization metrics.
- **Network Topology and Connectivity:** Use LLDP, CDP, and interface state logs to maintain an accurate view of network physical connections and identify cable faults or port-security violations immediately.

## Data types collected

This integration collects logs from HPE Aruba CX devices via multiple transport methods. Each method provides the following data stream:

- **HPE Aruba CX logs (log):** A unified data stream that collects and parses all supported event types from AOS-CX switches.
    - **TCP Input Description:** Collect Aruba CX logs using tcp input.
    - **UDP Input Description:** Collect Aruba CX logs.
    - **Filestream Input Description:** Collect Aruba logs using the filestream input.

The types of information captured within these streams include:
- **System and Audit Logs:** Administrative actions, configuration changes (via REST or CLI), and system-level events.
- **Security Events:** AAA logs, accounting data, ACL logs, ARP security, and IP source lockdown events.
- **Routing and Protocol Logs:** Detailed events for BFD, BGP, OSPFv2/v3, PIM, VRRP, and VXLAN.
- **Layer 2 and Connectivity Logs:** Interface transitions, LACP/LAG status, LLDP discovery, and Spanning Tree updates.
- **Hardware Telemetry:** Environmental data including temperature, fan speeds, power supply status, and ASIC resource utilization.
- **Service Logs:** DHCP (Server, Relay, Snooping), DNS client, NTP, and SSH/Telnet server events.

## Compatibility

- **AOS-CX Versions:** This integration is specifically designed for and tested against **HPE Aruba Networking CX Switches** (models **6000, 6300, and 8360**) running **AOS-CX version 10.15**.
- **Kibana Version:** Requires Kibana version **8.15.0** or higher.
- **Language:** This integration exclusively supports logs formatted in **English**.

## Scaling and Performance

To ensure optimal performance in high-volume network environments, consider the following:

- **Transport/Collection Considerations:** When choosing between TCP and UDP for syslog collection, prioritize **TCP** (port `1470`) for guaranteed delivery if your network environment supports it. If minimizing switch overhead is critical, use **UDP** (port `1024`), but be aware that packets may be dropped during periods of high network congestion. For environments where logs are already centralized in a file system, the **filestream** input provides robust rotation handling and ensures no data is lost during agent restarts.
- **Data Volume Management:** AOS-CX switches can generate significant log volume during protocol instability or high-traffic periods. Configure the vendor appliance to forward only necessary events by using the `severity` filter on the switch CLI (e.g., setting it to `informational` or `warning`). Avoid forwarding debug-level logs as they can overwhelm the ingest pipeline and increase CPU load on both the source system and the Elastic Agent.
- **Elastic Agent Scaling:** For high-throughput environments, such as large-scale campus or data center deployments, deploy multiple Elastic Agents behind a network load balancer to distribute traffic evenly. Place Agents close to the data source to minimize latency and ensure the Agent host has sufficient CPU and memory resources to handle the regex-based parsing required for AOS-CX syslog formats.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have `admin` or `operator` level access to the Aruba CX Command Line Interface (CLI) to configure logging parameters.
- **Network Connectivity:** The switch must have IP connectivity to the Elastic Agent. Ensure that any firewalls between the switch and the Agent allow traffic on the configured syslog ports (**TCP 1470** or **UDP 1024**).
- **AOS-CX Version:** Ensure the switch is running a compatible version of AOS-CX (10.15 or later recommended).
- **Management VRF Knowledge:** Identify if the switch uses a specific Virtual Routing and Forwarding (VRF) instance for management traffic (commonly named `mgmt`).
- **English Language Setting:** The switch must be configured to output logs in English, as internationalization is not supported by the parser.

## Elastic prerequisites

- **Kibana Version:** Ensure your Elastic Stack is running version **8.15.0** or higher.
- **Elastic Agent Installation:** An Elastic Agent must be installed and enrolled in a policy via Fleet.
- **Integration Installation:** The HPE Aruba CX integration must be added to the Agent policy.
- **Connectivity:** The Elastic Agent must be reachable by the switch on the configured TCP or UDP ports.

## Vendor set up steps

The HPE Aruba CX Switch must be configured to forward logs to the Elastic Agent via the CLI.

### For Syslog (TCP/UDP) Collection:
1. Connect to your HPE Aruba CX switch using an SSH client or a direct console connection.
2. Enter privileged EXEC mode by typing `enable`.
3. Enter global configuration mode by typing `configure terminal`.
4. Configure the remote logging server using the IP address of your Elastic Agent. For **UDP** (Default Port `1024`):
   ```sh
   logging <AGENT_IP_ADDRESS> port 1024 udp
   ```
5. For **TCP** (Default Port `1470`):
   ```sh
   logging <AGENT_IP_ADDRESS> port 1470 tcp
   ```
6. Set the logging severity to ensure all relevant events are captured (recommend `informational`):
   ```sh
   logging <AGENT_IP_ADDRESS> severity informational
   ```
7. (Optional) If your switch management is in a specific VRF (e.g., `mgmt`), specify it:
   ```sh
   logging <AGENT_IP_ADDRESS> vrf mgmt
   ```
8. Exit configuration mode and save the settings:
   ```sh
   end
   write memory
   ```
9. Verify the configuration with the command: `show running-config | include logging`

### For Logfile Collection:
1. Ensure your log aggregator is writing Aruba CX logs to a local disk accessible by the Elastic Agent.
2. Grant the Elastic Agent read permissions for the directory where logs are stored.
3. Identify the absolute path to the log files, such as `/var/log/audit/*.log`.

## Kibana set up steps

### Collecting logs from HPE Aruba CX via TCP
1. In Kibana, navigate to **Integrations** and search for **HPE Aruba CX**.
2. Click **Add HPE Aruba CX**.
3. Select the **Collecting logs from HPE Aruba CX via TCP** input type.
4. Configure the following fields:
   - **Listen Address** (`listen_address`): The bind address to listen for TCP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
   - **Listen Port** (`listen_port`): The TCP port number to listen on. Default: `1470`.
   - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
5. Save and deploy the integration to your Elastic Agent policy.

### Collecting logs from HPE Aruba CX via UDP
1. In Kibana, navigate to **Integrations** and search for **HPE Aruba CX**.
2. Click **Add HPE Aruba CX**.
3. Select the **Collecting logs from HPE Aruba CX via UDP** input type.
4. Configure the following fields:
   - **Listen Address** (`listen_address`): The bind address to listen for UDP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
   - **Listen Port** (`listen_port`): The UDP port number to listen on. Default: `1024`.
   - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
5. Save and deploy the integration to your Elastic Agent policy.

### Collect logs from HPE Aruba CX instances using filestream input.
1. In Kibana, navigate to **Integrations** and search for **HPE Aruba CX**.
2. Click **Add HPE Aruba CX**.
3. Select the **Collect logs from HPE Aruba CX instances using filestream input.** input type.
4. Configure the following fields:
   - **Paths** (`paths`): List of paths to monitor for log files. Default: `['/var/log/audit/*.log']`.
   - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
5. Save and deploy the integration to your Elastic Agent policy.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on HPE Aruba CX:
- **Generate a Configuration Change Event:** Enter global configuration mode and exit to trigger a management log: `configure terminal` then `exit`.
- **Trigger an Interface State Change:** Select a non-production interface and toggle its state: `interface 1/1/1`, `shutdown`, followed by `no shutdown`.
- **Generate an Authentication Event:** Log out of your current SSH session and log back in to generate AAA/Accounting events.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "hpe_aruba_cx.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields:
   - `event.dataset` (should match `hpe_aruba_cx.log`)
   - `source.ip` (the IP address of the switch)
   - `event.action` (e.g., `link_state_change` or `config_change`)
   - `message` (the raw log payload)
5. Navigate to **Analytics > Dashboards** and search for "HPE Aruba CX" to view pre-built visualizations.

# Troubleshooting

## Common Configuration Issues

- **Incorrect VRF Routing:** If the switch is configured to use the `mgmt` VRF but the `logging` command does not specify it, logs will fail to route to the Elastic Agent. Ensure the `vrf <name>` parameter is included in the CLI command.
- **Port and Protocol Mismatch:** If the switch is sending via UDP on port 514 but the Agent is listening on TCP port 1470, no data will be ingested. Verify both sides match exactly in the CLI and Kibana UI.
- **Severity Level Filtering:** If the switch severity is set too high (e.g., `emergency`), standard operational logs like interface changes (`info`) will not be sent. Set `logging severity debug` for initial testing.
- **Firewall Blockage:** Network firewalls or local server firewalls (iptables/firewalld) may block the incoming TCP/UDP ports. Verify connectivity using a tool like `netcat` or `tcpdump` on the Agent host to see if packets are arriving.

## Ingestion Errors

- **Parsing Failures:** If logs appear in Kibana with a `tags: ["_grokparsefailure"]`, it usually indicates the log format has changed in a new AOS-CX version or the language is not set to English.
- **Field Mapping Mismatches:** Review the `error.message` field in Discover to identify if specific fields are failing to map to ECS due to unexpected data types or schema conflicts.
- **Time Sync Issues:** If logs appear with incorrect timestamps or in the "future/past," verify that NTP is configured and synchronized on both the Aruba switch and the Elastic Agent host.

## Vendor Resources

- Official HPE Aruba CX 10.15 Event Log Message Reference Guide
- Refer to the official vendor website for additional resources.

## Documentation sites
- AOS-CX 10.15 Event Log Message Reference Guide
- HPE Aruba CX AAA Event Reference
- HPE Aruba CX ACL Event Reference
- HPE Aruba CX BGP Event Reference
- Refer to the official vendor website for additional configuration guides.
