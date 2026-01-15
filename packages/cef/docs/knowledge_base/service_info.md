# Service Info

## Common use cases

The Common Event Format (CEF) integration is a universal log collector designed to ingest, parse, and normalize security events from a wide range of third-party security appliances and applications.
- **Security Information and Event Management (SIEM):** Centralize security logs from firewalls, intrusion detection systems (IDS), and endpoint security software into the Elastic Stack for unified threat detection.
- **Regulatory Compliance:** Aggregate audit trails from multi-vendor environments to meet compliance requirements such as PCI-DSS, HIPAA, or GDPR by ensuring all logs are stored in a standard, searchable format.
- **Cross-Vendor Correlation:** Use the standardized CEF format to correlate events between different hardware vendors, such as matching a firewall block event with an endpoint malware alert.
- **Legacy System Integration:** Provide a path for older or proprietary systems that lack native Elastic integrations but support the industry-standard CEF syslog output to stream data into Elastic.

## Data types collected

This integration can collect the following types of data:
- **Security Event Logs:** High-priority alerts including malware detections, intrusion attempts, and policy violations.
- **Network Traffic Logs:** Connection details including source/destination IP addresses, ports, protocols, and bytes transferred.
- **Authentication Logs:** User login attempts, privilege escalations, and session management events.
- **System Audit Logs:** Configuration changes, service restarts, and administrative actions taken on the source device.
- **Data Formats:** Logs are ingested as Syslog messages containing a CEF-formatted payload (typically starting with `CEF:0`).
- **Transport Protocols:** Supports ingestion over both UDP and TCP protocols.

## Compatibility
The CEF integration is compatible with any device or software capable of outputting logs in the ArcSight-style CEF standard. Specific products tested or documented include:
- **Palo Alto Networks PAN-OS** (version 11.1 and earlier/later versions supporting custom syslog formats).
- **Linux rsyslog** (all versions supporting standard forwarding rules).
- Any vendor-neutral appliance following the CEF header standard: `CEF:Version|Device Vendor|Device Product|Device Version|Device Event Class ID|Name|Severity|Extension`.

## Scaling and Performance

- **Transport/Collection Considerations:** For high-reliability requirements, TCP is the recommended transport protocol as it supports acknowledgement of data receipt and allows the use of memory or disk-based queues (e.g., rsyslog's `linkedList` queue) to buffer events if the Elastic Agent is momentarily unreachable. UDP may be used for higher throughput with lower overhead, but it risks data loss during network congestion or buffer overflows on the receiving host.
- **Data Volume Management:** To prevent overwhelming the ingestion pipeline, it is recommended to apply filtering at the source or the rsyslog forwarder level. Users should configure their source devices to send only relevant security events (e.g., severity 'Warning' and above) and exclude high-volume informational logs like routine heartbeat messages or noisy connection-allowed logs unless required for forensic auditing.
- **Elastic Agent Scaling:** A single Elastic Agent can typically handle several thousand CEF events per second (EPS) depending on the available system resources. For environments exceeding 5,000 EPS, it is recommended to deploy multiple Elastic Agents behind a network load balancer (e.g., HAProxy or F5 BIG-IP) to distribute the load and provide high availability.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have administrative or root-level access to the Linux server acting as the syslog forwarder and administrative access to the source security appliance.
- **Network Connectivity:** Ensure the source device can reach the rsyslog forwarder on the configured syslog port (typically UDP/514 or TCP/514).
- **Firewall Rules:** Update any internal firewalls to allow traffic from the source device IP to the forwarder IP, and from the forwarder IP to the Elastic Agent host.
- **CEF Support:** Verify that the source device has the "CEF" or "ArcSight" log format option enabled in its logging configuration.
- **Resource Allocation:** The intermediate Linux forwarder should have sufficient CPU and memory to handle the expected log volume and manage rsyslog queues.

## Elastic prerequisites

- **Elastic Agent:** An active Elastic Agent must be installed and enrolled in Fleet.
- **Policy Assignment:** The Agent must be assigned to a policy that includes the CEF integration.
- **Connectivity:** The host running the Elastic Agent must have the specified syslog port open in its local OS firewall (e.g., `ufw` or `firewalld`).

## Vendor set up steps

### For General CEF Collection:
1. Log in to the management interface of your security appliance.
2. Navigate to the **Logging**, **Syslog**, or **Remote Access** settings section.
3. Add a new **Syslog Server** or **Remote Log Destination**.
4. Enter the **IP Address** of the host where the Elastic Agent is running.
5. Set the **Port** to match your Elastic Agent configuration (e.g., `514` or `5514`).
6. Select the **Format** or **Template** as `CEF` or `Common Event Format`.
7. Choose the **Transport Protocol** (UDP or TCP) as per your organizational requirements.
8. Save the configuration and, if necessary, apply the changes to the device's running policy.

### For Fortinet FortiGate (CLI Method):
1. Establish a CLI session (SSH or Console) with the FortiGate device.
2. Enter the syslog configuration context:
   ```sh
   config log syslogd setting
   ```
3. Enable the syslog destination and set the format to CEF:
   ```sh
   set status enable
   set format cef
   ```
4. Define the target Elastic Agent IP address and communication port:
   ```sh
   set server "192.168.1.50"
   set port 514
   ```
5. Specify the transport mode (udp, legacy-reliable for TCP, or rsyslog-reliable):
   ```sh
   set mode udp
   ```
6. Commit the configuration:
   ```sh
   end
   ```
7. Verify the output format by running:
   ```sh
   show full-configuration log syslogd setting | grep format
   ```

## Kibana set up steps

1.  **Navigate to Integrations:** In Kibana, go to **Management > Integrations**.
2.  **Find CEF:** Search for "CEF" and select the integration tile. Click **Add CEF**.
3.  **Configure Input:** Under the **CEF (Syslog)** input configuration, set the following:
    - **Syslog Host**: Set to `0.0.0.0` to listen on all interfaces, or a specific IP address of the Agent host.
    - **Syslog Port**: Enter the port number (e.g., `514` or `5514`) that your Rsyslog forwarder is sending data to.
    - **Protocol**: Select `tcp` or `udp` to match your forwarder configuration.
4.  **Select Policy:** Choose the existing Agent policy or create a new one where this integration will be deployed.
5.  **Save and Deploy:** Click **Save and continue** and then **Add agent integration**. The configuration will be pushed to all Agents enrolled in that policy.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from the Vendor to the Elastic Stack.

### 1. Trigger Data Flow on CEF Source:
- **Generate Authentication Event:** Attempt to log in to the vendor device's management console with an incorrect password to generate an "Auth Failure" log.
- **Trigger Configuration Change:** Enter the configuration CLI or Web UI, make a minor change (like a description), and save/commit it to trigger a "System" or "Config" event.
- **Test Traffic Flow:** If configuring a firewall, attempt to access a blocked resource or ping a monitored interface to generate "Traffic" or "Threat" logs.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the following KQL filter: `data_stream.dataset : "cef.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `cef.log`)
   - `observer.vendor` and/or `observer.product` (derived from the CEF header)
   - `event.severity` (mapped from the CEF severity field)
   - `cef.device_event_class_id` (the specific event ID from the source)
   - `message` (containing the raw CEF payload)
5. Navigate to **Analytics > Dashboards** and search for "CEF" to view the pre-built **[Logs CEF] Overview** dashboard.

# Troubleshooting

## Common Configuration Issues

- **Port Conflict**: If the Elastic Agent fails to start the syslog listener, check if another service (like rsyslog or syslog-ng) is already bound to the same port. Use `netstat -tunlp | grep <port>` to identify conflicting processes.
- **Firewall Blocking**: Logs may be sent by the vendor but blocked by the host's firewall. Ensure that the specific UDP/TCP port is allowed in `iptables`, `nftables`, or Windows Firewall on the Elastic Agent host.
- **Incorrect Format Selection**: If logs appear in Kibana but are not parsed (fields remain in the `message` string), ensure the vendor device is specifically set to **CEF** and not "Standard Syslog" or "RFC5424". CEF logs must start with the `CEF:0` header.
- **Mismatched Protocol**: If the vendor is sending via TCP but the Agent is listening on UDP (or vice-versa), no data will be received. Verify that the `Protocol` setting in the Kibana integration matches the vendor's `mode`.

## Ingestion Errors

- **Parsing Failures**: Check the `error.message` field in Discover. Common causes include non-standard CEF headers or missing mandatory fields (Version, Device Vendor, Device Product, Device Version, Signature ID, Name, Severity).
- **Timezone Mismatch**: If logs appear to be "from the future" or delayed, check the timezone settings on the vendor device. CEF logs often use UTC; ensure the Elastic Agent or the ingest pipeline is correctly interpreting the timestamp.
- **Truncated Logs**: Large CEF messages (especially those with long URLs or deep packet inspection data) may be truncated by the syslog transport. Increase the maximum message size in the integration's advanced settings if supported.

## Vendor Resources
- [Configure Syslog Monitoring - Palo Alto Networks](https://docs.paloaltonetworks.com/pan-os/11-1/pan-os-admin/monitoring/use-syslog-for-monitoring/configure-syslog-monitoring)
- [Troubleshoot CEF and Syslog via AMA connectors - Microsoft Learn](https://learn.microsoft.com/en-us/azure/sentinel/cef-syslog-ama-troubleshooting)

## Documentation sites
- [Configure Syslog Monitoring - Palo Alto Networks](https://docs.paloaltonetworks.com/pan-os/11-1/pan-os-admin/monitoring/use-syslog-for-monitoring/configure-syslog-monitoring)
- [Troubleshoot CEF and Syslog via AMA connectors - Microsoft Learn](https://learn.microsoft.com/en-us/azure/sentinel/cef-syslog-ama-troubleshooting)
