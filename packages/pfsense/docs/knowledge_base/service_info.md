# Service Info

The pfSense integration for Elastic allows users to seamlessly ingest, parse, and visualize logs from pfSense and OPNsense firewall distributions. By centralizing firewall, VPN, and system activity data, security teams can gain deep visibility into network traffic patterns and potential security threats across their perimeter.

## Common use cases

- **Firewall Event Monitoring:** Analyze blocked and allowed traffic patterns to identify potential scanning activity or misconfigured firewall rules across the network.
- **VPN and Remote Access Auditing:** Track OpenVPN and IPsec connection attempts, session durations, and user authentication events to ensure secure remote access compliance.
- **Network Service Troubleshooting:** Monitor DHCP lease assignments and DNS resolution logs (Unbound) to diagnose connectivity issues or identify malicious domain requests.
- **Web Proxy and Load Balancer Analysis:** Gain visibility into web traffic and service availability by collecting logs from integrated packages like Squid and HAProxy.
- **Security Compliance:** Maintain long-term records of administrative access and configuration changes for auditing and regulatory requirements.

## Data types collected

This integration collects logs from pfSense and OPNsense systems using the following data streams:

- **pfSense syslog logs (input: udp):** Collect pfsense logs using udp input. This is a logs-type data stream (`pfsense.log`) that captures events transmitted via the User Datagram Protocol.
- **pfSense syslog logs (input: tcp):** Collect pfsense logs using tcp input. This is a logs-type data stream (`pfsense.log`) that captures events transmitted via the Transmission Control Protocol, supporting optional SSL/TLS encryption.

These data streams include:
- **Firewall Logs:** Detailed records of packet filtering events, including source/destination IPs, ports, and rule actions (allow/block).
- **DNS Resolver Logs:** Information from the Unbound DNS daemon regarding query resolution and system activity.
- **DHCP Logs:** Records of IP address leases, requests, and releases managed by the DHCP daemon.
- **VPN Logs:** Connection and status information for both OpenVPN and IPsec tunnels.
- **Package-Specific Logs:** Data from popular add-ons including HAProxy (load balancing), Squid (caching proxy), and others when forwarded.
- **Authentication Logs:** Web interface login and session management events captured via PHP-FPM.

## Compatibility

The pfSense integration is compatible with **pfSense** and **OPNsense** firewalls running recent stable versions. 

**Elastic prerequisites:**
- **Elastic Stack version:** 8.11.0 or higher (compatible with versions ^8.11.0 or ^9.0.0).
- **Elastic Agent:** An Elastic Agent must be installed and successfully enrolled in Fleet.

## Scaling and Performance

To ensure optimal performance in high-volume environments, consider the following:

- **Transport/Collection Considerations:** This integration supports both **UDP** and **TCP** transport. While UDP is faster and places less overhead on the firewall, TCP is recommended for environments where log delivery guarantees are required to prevent data loss during network spikes. For encrypted transport, use the TCP input with SSL enabled.
- **Data Volume Management:** Configure the firewall appliance to forward only necessary events. Recommend filtering or limiting data at the source by avoiding the "Everything" checkbox in pfSense remote logging unless package logs (like Squid or HAProxy) are specifically required. This prevents overwhelming the ingest pipeline with debug-level information.
- **Elastic Agent Scaling:** For high-throughput environments exceeding 10,000 events per second (EPS), deploy the Elastic Agent on a host with sufficient CPU and memory resources. Consider deploying multiple Elastic Agents behind a network load balancer to distribute the syslog traffic and ensure high availability of the collection point.

# Set Up Instructions

## Vendor prerequisites

1. **Administrative Access:** Ensure you have full administrative privileges to the pfSense or OPNsense web-based GUI to configure remote logging settings.
2. **Network Connectivity:** The firewall must have a clear network path to the Elastic Agent host on the configured port (default is `9001`).
3. **Port Availability:** Confirm that the selected port (`9001`) is not being used by other services on the Elastic Agent host and is allowed through any host-based firewalls (e.g., `ufw` or `firewalld`).
4. **Log Format Selection:** pfSense should be configured to use `syslog (RFC 5424)` for the most reliable parsing results.

## Elastic prerequisites

1. **Elastic Agent:** An Elastic Agent must be installed and successfully enrolled in Fleet.
2. **Agent Policy:** A dedicated or shared agent policy must be created to hold the pfSense integration configuration.
3. **Connectivity:** The host running the Elastic Agent must be reachable by the firewall over the network via the chosen protocol (UDP or TCP).

## Vendor set up steps

### For pfSense Firewalls:
1. Log in to the pfSense web interface and navigate to **Status > System Logs**.
2. Click on the **Settings** tab to access log configuration.
3. In the **General Logging Options**, set the **Log Message Format** to `syslog (RFC 5424, with RFC 3339 microsecond-precision timestamps)`.
4. Scroll down to the **Remote Logging Options** section and check the **Enable Remote Logging** box.
5. For **Source Address**, select the interface that will communicate with the agent (e.g., `LAN` or `Management`).
6. In the **Remote log servers** field, enter the IP address of the Elastic Agent followed by the port, for example: `192.168.1.50:9001`.
7. Under **Remote Syslog Contents**, select the specific log types you wish to forward (e.g., Firewall Events, DNS Events).
8. Click **Save** at the bottom of the page to apply the configuration.

### For OPNsense Firewalls:
1. Log in to the OPNsense web interface and navigate to **System > Settings > Logging / Targets**.
2. Click the **+** (Add) button to create a new logging target.
3. Set the **Transport** to either **UDP (4)** or **TCP (4)** to match your integration input configuration.
4. In the **Hostname** field, enter the IP address of the host running the Elastic Agent.
5. In the **Port** field, enter the destination port (default `9001`).
6. For **Applications**, leave blank to send all logs or select specific services like `filter`, `dhcp`, or `unbound`.
7. Set the **Description** to `Elastic Agent Syslog` and click **Save**.
8. Click **Apply** on the main Logging Targets page to activate the new settings.

## Kibana set up steps

1. In Kibana, navigate to **Management > Integrations**.
2. Search for **pfSense** and select the integration.
3. Click **Add pfSense**.
4. Configure the desired input types:

### Collecting logs from pfSense systems (input: udp)
This input type collects pfSense logs using the UDP protocol.
- **Syslog Host** (`syslog_host`): The interface to listen to UDP based syslog traffic. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
- **Syslog Port** (`syslog_port`): The UDP port to listen for syslog traffic. Ports below 1024 require Filebeat to run as root. Default: `9001`.
- **Internal Networks** (`internal_networks`): The internal IP subnet(s) of the network. Used for identifying internal vs external traffic. Default: `['private']`.
- **Timezone Offset** (`tz_offset`): Used to correctly parse datetimes from logs that lack timezone information. Accepts canonical IDs (e.g., `Europe/Amsterdam`), abbreviations (e.g., `EST`), or differentials (e.g., `-05:00`). Default: `local`.
- **Preserve original event** (`preserve_original_event`): If enabled, preserves a raw copy of the original event in the field `event.original`. Default: `False`.

### Collecting logs from pfSense systems (input: tcp)
This input type collects pfSense logs using the TCP protocol, allowing for encrypted transport.
- **Syslog Host** (`syslog_host`): The interface to listen to TCP based syslog traffic. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
- **Syslog Port** (`syslog_port`): The TCP port to listen for syslog traffic. Ports below 1024 require Filebeat to run as root. Default: `9001`.
- **Internal Networks** (`internal_networks`): The internal IP subnet(s) of the network. Default: `['private']`.
- **Timezone Offset** (`tz_offset`): Used to set the timezone offset for correct datetime parsing relative to the agent host. Default: `local`.
- **SSL Configuration** (`ssl`): SSL configuration options. Use this to configure certificate and key paths for secure TCP transport.
- **Preserve original event** (`preserve_original_event`): If enabled, preserves a raw copy of the original event in the field `event.original`. Default: `False`.

5. Click **Save and continue** to save the integration and apply the policy to your Elastic Agents.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on pfSense:
- **Authentication event:** Log out of the pfSense/OPNsense web interface and log back in to trigger a web authentication event.
- **Firewall event:** Create a temporary firewall rule to block a specific IP and attempt to ping that IP from a device behind the firewall.
- **Service event:** Restart a service such as the DNS Resolver (Unbound) or DHCP daemon via the **Status > Services** menu.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "pfsense.log"`
4. Verify logs appear. Expand a log entry and confirm these fields:
   - `event.dataset` (should match `pfsense.log`)
   - `source.ip` and/or `destination.ip`
   - `event.outcome` (verify it captures `success`, `failure`, `allow`, or `deny`)
   - `message` (the raw log payload)
5. Navigate to **Analytics > Dashboards** and search for "pfSense" to verify visualizations are populating.

# Troubleshooting

## Common Configuration Issues

- **Port Binding Conflicts**: If the Elastic Agent fails to start the listener, check if another service is already using port 9001. Use `ss -tulpn | grep 9001` (Linux) to identify conflicting processes.
- **Firewall Blocking**: Ensure that the firewall on the Elastic Agent host itself is configured to allow inbound traffic on port `9001` (UDP or TCP).
- **Incorrect Log Format**: If logs are appearing in Kibana but fields are not being parsed correctly, verify that pfSense is set to "syslog (RFC 5424)" format.
- **Host Binding**: If the **Syslog Host** is set to `localhost` in Kibana, the agent will only accept logs from the local machine. Change this to `0.0.0.0` to accept logs from your network firewall.

## Ingestion Errors

- **Timestamp/Timezone Mismatch**: If logs appear in the past or future, ensure the **Timezone Offset** in Kibana matches the firewall's timezone. Alternatively, switch the pfSense Log Message Format to **Syslog (RFC 5424)** to include explicit timezone metadata.
- **Parsing Failures**: Logs from unsupported packages or customized log formats may result in `tags: ["_grokparsefailure"]`. Check the `error.message` field in Discover to identify which part of the ingest pipeline is failing.
- **Message Truncation**: Large log messages from services like HAProxy may be truncated if using UDP. If you see incomplete JSON or cut-off messages, switch the transport protocol to TCP.

## Vendor Resources

- [Official pfSense Remote Logging Documentation](https://docs.netgate.com/pfsense/en/latest/monitoring/logs/settings.html)
- [pfSense Remote Syslog Configuration Guide](https://docs.netgate.com/pfsense/en/latest/monitoring/logs/remote.html)
- [Huntress pfSense Syslog Configuration Reference](https://support.huntress.io/hc/en-us/articles/34546803133971-Syslog-pfSense)

## Documentation sites

- [pfSense Official Documentation - System Logs](https://docs.netgate.com/pfsense/en/latest/monitoring/logs/settings.html)
- [Elastic pfSense Integration Reference](https://www.elastic.co/docs/reference/integrations/pfsense)
- OPNsense Documentation - Logging Configuration
