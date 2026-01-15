# Service Info

## Common use cases

The NetFlow integration for Elastic allows organizations to gain deep visibility into their network traffic patterns by collecting and analyzing flow records exported from routers, switches, and firewalls.
- **Network Traffic Monitoring:** Gain real-time visibility into top talkers, popular applications, and protocol distribution across the enterprise network.
- **Capacity Planning:** Identify bandwidth bottlenecks and analyze historical trends to make informed decisions about infrastructure upgrades and network optimizations.
- **Security Analytics:** Detect anomalies such as DDoS attacks, unauthorized data exfiltration, or lateral movement by monitoring flow patterns that deviate from established baselines.
- **Compliance and Auditing:** Maintain long-term records of network connections to satisfy regulatory requirements for data retention and forensic investigation after security incidents.

## Data types collected

This integration can collect the following types of data:
- **NetFlow Flow Records:** Captures detailed information about IP traffic flows, including source and destination IP addresses, source and destination ports, and the transport protocol used.
- **Traffic Metrics:** Collects quantitative data such as total bytes transferred, total packets sent, and flow duration for each identified session.
- **Network Metadata:** Includes information such as Input/Output SNMP interface indices, TCP flags, and Type of Service (ToS) bytes.
- **Data Formats:** Supports standard NetFlow formats including NetFlow version 5, NetFlow version 9, and IPFIX (NetFlow version 10).
- **Transport Protocol:** All records are typically received via UDP on a configurable port (defaulting to 2055).

## Compatibility

The NetFlow integration is compatible with **Cisco IOS** (Release 12.x and higher), **Cisco ASA**, **Juniper Junos**, **HP ProCurve**, and any other vendor hardware or software that supports standard NetFlow v5, v9, or IPFIX (v10) export protocols. It has been specifically tested with **Cisco IOS Release 15.M&T** for Version 9 export.

## Scaling and Performance

- **Transport/Collection Considerations:** NetFlow uses UDP for data transport, which is connectionless and has lower overhead but provides no guarantee of delivery. In high-traffic environments, network congestion can lead to dropped UDP packets. To mitigate this, ensure the Elastic Agent host has a large enough UDP receive buffer and that the network path between the exporter and collector is uncongested.
- **Data Volume Management:** For high-throughput core routers, it is highly recommended to use **Sampled NetFlow** (e.g., 1:100 or 1:1000 packets) at the source device. This significantly reduces the volume of flow records sent to the Elastic Agent while still providing a statistically accurate representation of network traffic. Additionally, only enable NetFlow on critical edge or core interfaces rather than every port on a switch.
- **Elastic Agent Scaling:** A single Elastic Agent can handle thousands of flows per second, but performance depends on the number of fields being parsed. For large-scale deployments, consider deploying multiple Elastic Agents behind a load balancer (using a UDP-capable load balancer) to distribute the ingestion load and provide high availability.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have administrative or "enable" level access to the command-line interface (CLI) of the network device to configure export settings.
- **Network Connectivity:** Ensure that the network device has a clear path to reach the Elastic Agent. Firewalls must be configured to allow UDP traffic on the designated NetFlow port (default is 2055).
- **Resource Availability:** Verify that the network device has sufficient CPU and memory resources to handle flow caching and exporting, especially if you are enabling ingress and egress monitoring on multiple high-speed interfaces.
- **Static IP/Hostname:** The Elastic Agent should ideally have a static IP address to prevent the network device from losing the export destination during a DHCP lease change.
- **Supported Version Knowledge:** Determine which NetFlow version (v5, v9, or IPFIX) your specific hardware supports to ensure the configuration matches the device capabilities.

## Elastic prerequisites

- **Elastic Agent:** A running Elastic Agent enrolled in Fleet or configured standalone.
- **Connectivity:** The Elastic Agent host must have an open firewall port to receive inbound UDP traffic from the network devices.
- **Version Requirements:** Elastic Stack version 7.10 or higher is recommended for full compatibility with NetFlow data streams.

## Vendor set up steps

### For Cisco IOS Devices:
1. Log in to your Cisco device via SSH or Console and enter global configuration mode:
   ```shell
   enable
   configure terminal
   ```
2. Set the NetFlow export version to 9 (recommended) or 5:
   ```shell
   ip flow-export version 9
   ```
3. Configure the destination IP address where the Elastic Agent is running and the UDP port (default is 2055):
   ```shell
   ip flow-export destination <elastic-agent-ip> 2055
   ```
4. Enter the configuration mode for the specific interface you want to monitor:
   ```shell
   interface GigabitEthernet0/1
   ```
5. Enable NetFlow for ingress (incoming) traffic on that interface:
   ```shell
   ip flow ingress
   ```
6. (Optional) Enable NetFlow for egress (outgoing) traffic:
   ```shell
   ip flow egress
   ```
7. Exit and save the configuration to permanent memory:
   ```shell
   exit
   exit
   write memory
   ```

### For Palo Alto Networks (PAN-OS):
1. Navigate to **Network > NetFlow** in the Palo Alto web interface.
2. Click **Add** to create a new NetFlow Server Profile.
3. Name the profile and add a server entry with the Elastic Agent's IP address and port 2055.
4. Select the NetFlow version (IPFIX is recommended for Palo Alto).
5. Navigate to **Network > Interfaces** and select the interface you wish to monitor.
6. In the interface settings, go to the **NetFlow Profile** dropdown and select the profile you created.
7. Click **Commit** to apply the changes to the device.

## Kibana set up steps

1. **Find the Integration:**
   Log in to Kibana and navigate to **Management > Integrations**. Search for "NetFlow" and select the official NetFlow integration.

2. **Add the Integration:**
   Click **Add NetFlow Records**. Choose a name for the integration policy and a description.

3. **Configure the Input:**
   In the integration settings, locate the **NetFlow Record** input section.
   - **UDP Host:** Set this to `0.0.0.0` to listen on all available interfaces or a specific IP address of the host.
   - **UDP Port:** Ensure this matches the port configured on your router (e.g., `2055`).
   - **Internal Networks:** Define your internal IP ranges (e.g., `10.0.0.0/8`, `192.168.0.0/16`) to enable correct source/destination classification.

4. **Select Agent Policy:**
   Choose the Agent policy that contains the Elastic Agent(s) responsible for receiving the NetFlow data.

5. **Deploy:**
   Click **Save and Continue**. The policy will be deployed to the Agents, and they will begin listening for incoming UDP packets.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from the network device to the Elastic Stack.

### 1. Trigger Data Flow on Cisco IOS:
- **Generate Traffic:** Perform a large file transfer (e.g., SCP or HTTP) through the monitored interface or ping a remote host repeatedly.
- **Verify Cache:** On the Cisco device, run `show ip cache flow` to confirm the device is identifying flows locally before exporting.
- **Check Statistics:** Run `show ip flow export` and look for the "Number of export packets" to increment, which confirms packets are being sent to the Elastic Agent.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the following KQL filter: `data_stream.dataset : "netflow.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `netflow.log`)
   - `netflow.source_ipv4_address` or `source.ip`
   - `netflow.destination_ipv4_address` or `destination.ip`
   - `netflow.protocol_identifier`
   - `netflow.octet_delta_count`
   - `message` (containing the raw flow metadata)
5. Navigate to **Analytics > Dashboards** and search for "NetFlow" to view the pre-built traffic overview and top talkers visualizations.

# Troubleshooting

## Common Configuration Issues

- **UDP Port Conflict**: If the Elastic Agent fails to start the listener, another process might be using port 2055. Use `netstat -tulpn | grep 2055` on Linux to identify conflicting services.
- **Firewall Blockage**: The host OS firewall (iptables/nftables/Windows Firewall) may be dropping incoming UDP 2055 traffic. Ensure an explicit "allow" rule is created for the source IP of the network device.
- **Access Control Lists (ACLs)**: On the router itself, ensure there are no egress ACLs preventing the device from sending UDP traffic to the Elastic Agent's management IP.
- **Mismatched Version**: If data is appearing but looks corrupted or missing fields, verify that the version configured in the router (`ip flow-export version`) matches the expectations of the integration templates.

## Ingestion Errors

- **Missing NetFlow V9 Templates**: NetFlow v9 and IPFIX require templates to be sent by the router before the Agent can decode the binary data. If you see "Flow sets without template" errors in the Agent logs, reduce the `template options timeout-rate` on the router to 1 minute to ensure templates are sent frequently.
- **Parsing Failures**: If the `error.message` field in Kibana indicates parsing issues, verify that the router is not using a proprietary flow format and is strictly adhering to NetFlow v5/v9 or IPFIX standards.
- **Timestamp Mismatches**: If flows appear in the past or future, ensure that both the network device and the Elastic Agent host are synchronized using NTP (Network Time Protocol).

## Vendor Resources

- [NetFlow Configuration Guide, Cisco IOS Release 15S](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_pi/configuration/15-s/nf-15-s-book/get-start-cfg-nflow.html)
- [Configure NetFlow Exports - Palo Alto Networks](https://docs.paloaltonetworks.com/pan-os/11-0/pan-os-admin/monitoring/netflow-monitoring/configure-netflow-exports)

# Documentation sites

- [NetFlow Configuration Guide, Cisco IOS Release 15M&T](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/netflow/configuration/15-mt/nf-15-mt-book/get-start-cfg-nflow.html)
- Refer to the official vendor website for additional resources.
