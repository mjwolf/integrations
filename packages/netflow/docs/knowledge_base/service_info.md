# Service Info

## Common use cases
The NetFlow integration provides a comprehensive solution for monitoring network traffic by collecting flow records from routers, switches, and firewalls. By ingesting these records into the Elastic Stack, administrators gain deep visibility into network utilization and security patterns without requiring full packet capture.
- **Network Traffic Analysis:** Identify top talkers and high-bandwidth applications across your infrastructure to optimize capacity and detect bottlenecks.
- **Security Threat Detection:** Monitor for unusual traffic patterns, such as data exfiltration, port scanning, or Command & Control (C2) communication, by analyzing source/destination IP pairs and port usage.
- **Capacity Planning:** Use historical flow data to trend bandwidth growth and make informed decisions about infrastructure upgrades or traffic shaping policies.
- **Incident Response and Forensics:** Investigate historical network events to determine the scope of a breach, identifying which internal hosts communicated with malicious external IPs during a specific timeframe.

## Data types collected
This integration can collect the following types of data:
- **Network Flow Logs:** Detailed records describing IP traffic flows, including metadata about the connection rather than the packet payload.
- **Supported Formats:** The integration natively handles NetFlow versions 1, 5, 6, 7, 8, and 9, as well as the IPFIX (IP Flow Information Export) protocol.
- **Automatic Mapping:** For legacy NetFlow versions (v1-v8), fields are automatically normalized and mapped to NetFlow v9 compatible fields within the Elastic Common Schema (ECS).
- **Collect NetFlow logs:** 
    - **Stream name:** `log`
    - **Type:** `logs`
    - **Description:** Collect NetFlow logs using the netflow input. This data stream decodes binary UDP templates and flow sets into structured events.

## Compatibility
The **NetFlow** integration is compatible with a wide range of network devices that support standard flow export protocols. 
- **Protocol Support:** Supports **NetFlow versions 1, 5, 6, 7, 8, and 9** along with **IPFIX** (standardized version of NetFlow v10).
- **Vendor Validation:** Validated against major vendors including Cisco, Juniper, and various open-source flow exporters.
- **Kibana Requirements:** This integration requires Kibana version **^8.14.0** or **^9.0.0**.

## Scaling and Performance

To ensure optimal performance in high-volume network environments, consider the following:
- **Transport/Collection Considerations:** NetFlow and IPFIX are typically exported over UDP, which is a connectionless protocol. While this minimizes overhead on the network device, it can lead to data loss during periods of high network congestion or packet drops at the Agent host. Ensure the network path has sufficient bandwidth to handle the bursty nature of flow exports and that the host's UDP receive buffer is appropriately tuned.
- **Data Volume Management:** In high-traffic environments, exporting every flow (1:1 sampling) can generate massive amounts of data. It is highly recommended to configure "Sampled NetFlow" on the vendor device (e.g., 1 out of every 100 packets) to reduce volume while maintaining statistical accuracy. Additionally, use filters on the exporter to only send data for specific critical interfaces or traffic directions.
- **Elastic Agent Scaling:** For high-throughput environments, deploy multiple Elastic Agents behind a network load balancer to distribute the incoming UDP traffic evenly. Place Agents as close to the data source as possible to minimize latency. Resource consumption (CPU and Memory) scales with the volume of unique flow templates and the total number of flows per second; horizontal scaling of Agents is the preferred method for managing increased load.

# Set Up Instructions

## Vendor prerequisites
- **Administrative Access:** You must have privileged access (e.g., `enable` mode on Cisco devices) to the command-line interface or management console of the network exporter.
- **Network Connectivity:** The network device must be able to reach the Elastic Agent host via UDP on the configured port (default is 2055). Ensure no intermediary firewalls or Access Control Lists (ACLs) are blocking this traffic.
- **Protocol Support:** Verify that your network hardware supports NetFlow (v5/v9) or IPFIX export. Some older devices or basic switches may not support flow export.
- **License Requirements:** On certain enterprise-grade routers and firewalls, NetFlow/IPFIX features may require specific software licenses or feature sets (e.g., Cisco's "Data" or "Security" packages).
- **Configuration Knowledge:** You will need the IP address of the Elastic Agent and the specific interfaces on your device that you wish to monitor.

## Elastic prerequisites
- **Elastic Agent Installed:** An Elastic Agent must be installed and enrolled in Fleet on a host that is network-accessible to the network exporters.
- **Kibana Version:** Ensure your Elastic Stack is running version **8.14.0** or higher.
- **Healthy Status:** Ensure the Agent status is "Healthy" in the Fleet management UI before configuring the integration.
- **Port Availability:** The UDP port designated for NetFlow (typically 2055) must be free and not occupied by other services (like `nfdump` or other flow collectors) on the Agent host.

## Vendor set up steps

The specific configuration depends on your hardware vendor (e.g., Cisco, Juniper, Fortinet). Below are the steps for a standard NetFlow v9 configuration.

### For Generic NetFlow/IPFIX Collection:
1. **Define a Flow Exporter:** Create a configuration block that specifies the destination (the Elastic Agent IP) and the UDP port (e.g., 2055).
2. **Define a Flow Monitor:** Create a monitor that references a record format (e.g., `netflow-original` or a custom IPFIX record).
3. **Configure Cache Timeouts:** Set the active timeout (typically 60 seconds) and inactive timeout (typically 15 seconds) to ensure flows are exported in a timely manner.
4. **Apply to Interfaces:** Navigate to each interface you wish to monitor and apply the flow monitor in the `input` (ingress) or `output` (egress) direction.
5. **Save Configuration:** Commit or save the changes to the device's running configuration.

### For Cisco IOS (Conceptual Example):
1. **Enter Configuration Mode:**
   ```bash
   configure terminal
   ```
2. **Configure the Flow Exporter:**
   ```bash
   flow exporter ELASTIC_AGENT_EXPORTER
    destination <IP_OF_ELASTIC_AGENT>
    source <SOURCE_INTERFACE>
    transport udp 2055
    export-protocol netflow-v9
   ```
3. **Configure the Flow Monitor:**
   ```bash
   flow monitor ELASTIC_AGENT_MONITOR
    exporter ELASTIC_AGENT_EXPORTER
    cache timeout active 60
    record netflow-original
   ```
4. **Apply the Monitor to an Interface:**
   ```bash
   interface GigabitEthernet0/1
    ip flow monitor ELASTIC_AGENT_MONITOR input
    ip flow monitor ELASTIC_AGENT_MONITOR output
   ```
5. **Verify the Configuration:** Use the command `show flow exporter statistics` to confirm packets are being sent.

## Kibana set up steps

### Collecting NetFlow logs using the netflow input
1. In Kibana, navigate to **Integrations** > **NetFlow Records**.
2. Click **Add NetFlow Records**.
3. Follow the prompts to add the integration to an existing Elastic Agent policy or create a new one.
4. Configure the following variables for the **Collect NetFlow logs** input:
    - **UDP host to listen on** (`host`): Specify the interface address the agent should bind to. Default: `localhost`. Use `0.0.0.0` to listen on all interfaces.
    - **UDP port to listen on** (`port`): Specify the port the agent will listen on for NetFlow/IPFIX packets. Default: `2055`.
    - **Internal Networks** (`internal_networks`): Provide a list of CIDR ranges describing the IP addresses that are considered internal. This is used in determining the values of `source.locality`, `destination.locality`, and `flow.locality`. The values can be either a CIDR value or one of the named ranges supported by the network condition. Default: `[private]`, which classifies RFC 1918 (IPv4) and RFC 4193 (IPv6) addresses as internal.
5. Save and deploy the integration. The Elastic Agent will automatically update its configuration and begin listening on the specified UDP port.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on NetFlow:
- **Generate Network Traffic:** From a host behind the configured network device, perform a large file transfer (e.g., using SCP or HTTP) or run a ping sweep to generate multiple flow records.
- **Authentication event:** Log out and log back into the administration interface of the network device to generate management-related flows.
- **Toggle Interface:** Perform a `shutdown` followed by a `no shutdown` on a non-production interface to trigger interface-state change events that result in updated flow data.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "netflow.log"`
4. Verify logs appear in the results table. Expand a log entry and confirm the presence and accuracy of these fields:
   - `event.dataset` (should be `netflow.log`)
   - `source.ip` and `destination.ip` (should match the hosts involved in the traffic)
   - `network.transport` (e.g., `udp`, `tcp`, or `icmp`)
   - `network.bytes` (the total byte count of the flow)
   - `message` (the raw log payload)
5. Navigate to **Analytics > Dashboards** and search for "Netflow" to view pre-built visualizations and confirm traffic distribution.

# Troubleshooting

## Common Configuration Issues
- **UDP Port Conflict:** If the Elastic Agent fails to start the input, check if another service is already using the configured UDP port (e.g., 2055). Use `netstat -ulnp | grep 2055` on Linux to identify conflicting processes.
- **Firewall Obstruction:** Data may not reach the Agent if the host-based firewall (iptables/firewalld/Windows Firewall) is blocking incoming UDP traffic. Ensure the port is explicitly allowed.
- **Internal Network Misconfiguration:** If `source.locality` or `destination.locality` fields are incorrect, verify the `internal_networks` CIDR list in the Kibana configuration matches your actual network architecture.
- **Sampling Rate Mismatch:** If traffic volumes in Kibana look significantly lower than expected, check the sampling rate on the network device. The integration does not automatically scale values based on vendor-specific sampling rates unless configured in the exporter.

## Ingestion Errors

- **Missing Templates (NetFlow v9/IPFIX)**: NetFlow v9 and IPFIX use templates to describe the data format. These templates are sent periodically. If the Agent hasn't received a template yet, it cannot parse the flows. Solution: Wait several minutes for the next template transmission or decrease the template transmit interval on the network device.
- **Parsing Failures**: Check the `error.message` field in Kibana. If you see "failed to parse field," it may indicate a non-standard implementation of a NetFlow field by the vendor.
- **Time Synchronization**: If flows appear with incorrect timestamps or are in the "future/past," ensure both the network device and the Elastic Agent host are synchronized via NTP.

## Vendor Resources
- [RFC 3954: Cisco Systems NetFlow Services Export Version 9](https://www.ietf.org/rfc/rfc3954.txt)
- [RFC 7011: IP Flow Information Export (IPFIX) Protocol](https://www.ietf.org/rfc/rfc7011.txt)
- [NetFlow Configuration Guide, Cisco IOS Release 15S](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_pi/configuration/15-s/nf-15-s-book/cfg-nflow-data-expt.html)
- [Getting Started with Configuring Cisco IOS NetFlow and NetFlow Data Export](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_pi/configuration/15-s/nf-15-s-book/get-start-cfg-nflow.html)

## Documentation sites
- [NetFlow Configuration Guide, Cisco IOS Release 15S](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_pi/configuration/15-s/nf-15-s-book/get-start-cfg-nflow.html)
- [RFC 3954: Cisco Systems NetFlow Services Export Version 9](https://www.ietf.org/rfc/rfc3954.txt)
- [RFC 7011: IP Flow Information Export (IPFIX) Protocol](https://www.ietf.org/rfc/rfc7011.txt)
