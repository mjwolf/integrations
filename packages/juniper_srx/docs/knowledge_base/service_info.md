# Service Info

The Juniper SRX integration provides a comprehensive solution for monitoring and analyzing security events, traffic flows, and system performance across Juniper SRX series gateways. By ingesting high-fidelity logs, security teams can gain real-time visibility into network perimeter defenses and internal traffic patterns.

## Common use cases

The Juniper SRX integration allows for the seamless ingestion and parsing of security and traffic logs from Juniper SRX Series Services Gateways into the Elastic Stack. By leveraging this integration, organizations can gain deep visibility into their network security posture and traffic patterns.

*   **Security Incident Monitoring:** Track and analyze security events such as intrusion detection (**RT_IDP**) and screen events (**RT_IDS**) to identify and respond to potential threats in real-time.
*   **Traffic Flow Analysis:** Monitor session creation, closure, and denial events (**RT_FLOW**) to understand network utilization, identify top talkers, and troubleshoot connectivity issues.
*   **Web and Content Filtering:** Audit web traffic and content filtering actions (**RT_UTM**) to ensure compliance with corporate policies and detect attempts to access malicious or blocked URLs.
*   **Malware and Threat Intelligence:** Leverage advanced security logs from Juniper Advanced Anti-Malware (**RT_AAMW**) and Security Intelligence (**RT_SECINTEL**) to identify infected hosts and blocked malicious IPs or files.

## Data types collected

This integration collects several categories of security and network data, mapped to the Elastic Common Schema (ECS). The following data streams are available:

*   **Juniper SRX logs**:
    *   **Collect Juniper SRX logs via TCP**: This data stream collects all supported Juniper SRX log types when the device is configured to send syslog over a TCP connection.
    *   **Collect Juniper SRX logs via UDP**: This data stream collects all supported Juniper SRX log types when the device is configured to send syslog over a UDP connection.
    *   **Read Juniper SRX logs from a file**: This data stream reads Juniper SRX logs from a local file or mounted volume, parsing the structured data format into ECS fields.

Supported log types within these streams include:
*   **Firewall Session Logs:** Detailed information on session creation, teardown, and denials, including source/destination IPs, ports, and protocols.
*   **Intrusion Detection/Prevention (IDP) Logs:** Events related to signature-based attacks and application-level DDoS mitigation.
*   **Unified Threat Management (UTM) Logs:** Records of web filtering (permitted/blocked), anti-virus detections, anti-spam actions, and content filtering.
*   **Advanced Anti-Malware (AAMW) Logs:** Logs regarding malware detection, host infection status, and automated mitigation actions.
*   **Security Intelligence (SecIntel) Logs:** Data on actions taken against known malicious actors based on threat feeds.

## Compatibility
The **Juniper SRX** integration is designed to work with SRX Series devices running JunOS. 
- **Elastic Stack Requirement:** The integration requires **Elastic Stack version ^8.11.0 or ^9.0.0**. 
- **Format Requirement:** The device must support `structured-data` syslog output.
- **Feature Compatibility:** While most SRX Series devices are supported, specific log types (like AAMW or UTM) require the corresponding software features to be enabled and licensed on the JunOS device.

## Scaling and Performance

To ensure optimal performance in high-volume environments, consider the following:
- **Transport/Collection Considerations:** For high-reliability environments, **TCP** is recommended as it ensures guaranteed delivery of log events, preventing data loss during network congestion. **UDP** may be used for higher throughput with lower overhead, but it lacks delivery guarantees and may drop packets under extreme load. If the Elastic Agent is co-located with the log files, the **filestream** input provides the most robust collection method with built-in rotation handling and state persistence.
- **Data Volume Management:** To manage high volumes of security logs (especially RT_FLOW), it is recommended to filter logs at the source using JunOS syslog severity levels (e.g., `any info`). Avoid forwarding `debug` level logs to the Elastic Agent unless active troubleshooting is required. Enabling `stream` mode for security logs on the SRX is critical for performance to avoid impacting the control plane during high traffic periods.
- **Elastic Agent Scaling:** For high-throughput environments, deploy multiple Elastic Agents behind a network load balancer to distribute incoming syslog traffic. Ensure the Agent host has sufficient CPU and memory to handle the parsing overhead of structured syslog data, and consider dedicated Agent policies for high-volume network security streams.

# Set Up Instructions

## Vendor prerequisites

1. **Administrative Access:** Root or super-user CLI access to the Juniper SRX device is required to modify the `[edit security log]` and `[edit system syslog]` hierarchies.
2. **Network Connectivity:** The SRX device must have a clear network path to the Elastic Agent. If a firewall exists between the SRX and the Agent, ensure TCP or UDP port **9006** (or your custom configured port) is open.
3. **License Requirements:** Ensure the SRX has active licenses for advanced features like IDP, UTM, or AAMW if you intend to collect those specific log types.
4. **Structured Logging Support:** The device must support the `sd-syslog` format. Verify this by checking the available options under the `set security log format` command.

## Elastic prerequisites

1. **Kibana Version:** This integration requires Kibana version **8.11.0** or higher.
2. **Elastic Agent:** An Elastic Agent must be installed and enrolled in a policy via Fleet.
3. **Integration Policy:** The Juniper SRX integration must be added to the Agent's policy with the correct input settings (TCP, UDP, or File).
4. **Network Visibility:** The Elastic Agent must be reachable at the IP address specified in the SRX configuration on the designated syslog port (default `9006`).

## Vendor set up steps

### For Syslog (TCP/UDP) Collection:

1. **Enter Configuration Mode:** Log in to the SRX CLI and enter configuration mode.
   ```cli
   configure
   ```
2. **Set Logging to Stream Mode:** Change the security log mode from event (local) to stream (remote).
   ```cli
   set security log mode stream
   ```
3. **Configure Structured Data Format:** This is mandatory for the integration to parse logs correctly.
   ```cli
   set security log format sd-syslog
   ```
4. **Define the Elastic Agent Destination:** Create a log stream and point it to the Elastic Agent's IP address.
   ```cli
   set security log stream elastic-stream host <elastic-agent-ip-address> port 9006
   ```
5. **Specify Source Interface:** Set the source IP address for the syslog packets to ensure they are sent from the correct interface.
   ```cli
   set security log source-address <srx-source-ip-address>
   ```
6. **Enable Session Logging:** In recent Junos versions, you must explicitly enable logging for session events.
   ```cli
   set security application-tracking log-session-create
   set security application-tracking log-session-close
   ```
7. **Commit Changes:** Apply the configuration to the device.
   ```cli
   commit
   ```

### For Logfile Collection:

1. **Configure Local Logging:** If using an intermediary log collector that writes to disk, configure the SRX to log to a file in structured format.
   ```cli
   set system syslog file srx-security-logs.log any any
   set system syslog file srx-security-logs.log structured-data
   ```
2. **Set Permissions:** Ensure the Elastic Agent service account has read permissions for the directory where the log file is stored (e.g., `/var/log/`).
3. **Rotate Logs:** Implement log rotation (e.g., using `logrotate`) to prevent disk exhaustion while ensuring the Agent has time to process the data before rotation.

## Kibana set up steps

1. In Kibana, navigate to **Management > Integrations**.
2. Search for and select **Juniper SRX**.
3. Click **Add Juniper SRX**.
4. Configure the desired inputs based on your deployment strategy:

### Collecting syslog from Juniper SRX via TCP.
This input collects Juniper SRX logs via TCP. This is the recommended method for reliable log delivery.
- **Syslog Host** (`syslog_host`): The interface the Agent should listen on for incoming TCP connections. Default: `localhost`.
- **Syslog Port** (`syslog_port`): The TCP port the Agent should listen on. Ensure this matches the port configured on your SRX device. Default: `9006`.
- **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.

### Collecting syslog from Juniper SRX via UDP.
This input collects Juniper SRX logs via UDP. This method is preferred for high-performance logging where minor data loss is acceptable.
- **Syslog Host** (`syslog_host`): The interface the Agent should listen on for incoming UDP datagrams. Default: `localhost`.
- **Syslog Port** (`syslog_port`): The UDP port the Agent should listen on. Ensure this matches the port configured on your SRX device. Default: `9006`.
- **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.

### Collecting syslog from Juniper SRX via file.
This input reads Juniper SRX logs from a file on the local filesystem of the Elastic Agent.
- **Paths** (`paths`): List of paths to the log files to be monitored. Default: `['/var/log/juniper-srx.log']`.
- **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.

5. Follow the prompts to add the integration to an existing Elastic Agent policy or create a new one.
6. Save the integration. The Elastic Agent will automatically update its configuration.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on Juniper SRX:
- **Generate configuration event:** Enter and exit configuration mode on the SRX CLI (e.g., `configure` then `exit`) or perform a `commit` to trigger system audit logs.
- **Generate security traffic:** Attempt to access a blocked resource or perform a port scan against an interface protected by the SRX to trigger `RT_FLOW` deny or `RT_IDS` screen events.
- **Generate session logs:** Establish a new SSH or HTTPS connection through the firewall to trigger `RT_FLOW_SESSION_CREATE` and `RT_FLOW_SESSION_CLOSE` events.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the following KQL filter: `data_stream.dataset : "juniper_srx.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `juniper_srx.log`)
   - `source.ip` and/or `destination.ip` (verify for RT_FLOW events)
   - `event.action` or `event.outcome` (e.g., `session-close` or `permit`)
   - `message` (containing the raw log payload)
5. Navigate to **Analytics > Dashboards** and search for "Juniper SRX" to view pre-built visualizations for traffic and security events.

# Troubleshooting

## Common Configuration Issues

- **Format Mismatch (No Parsing)**: If logs are appearing in Kibana but are not being parsed into fields, verify that the SRX is configured with `structured-data` and the `brief` option. Logs in standard syslog or BSD format will not be correctly handled by this integration.
- **Network Port Conflicts**: If the Elastic Agent fails to start the input, ensure that no other service is using port `9006` (TCP or UDP) on the Agent host. Use `netstat -ano | grep 9006` to check.
- **Logs Not Sent by SRX**: Verify that the security policies on the SRX have `then log session-init` or `then log session-close` configured. Without policy-level logging, the device will not generate flow events.
- **Source Address Issues**: Ensure the `source-address` for security logging is reachable by the Elastic Agent and is not being dropped by any intermediary firewalls or ACLs.

## Ingestion Errors

- **Parsing Failures**: If the `error.message` field contains parsing errors, check if the SRX is sending "brief" format or if additional custom tags are being added that deviate from the expected `structured-data` schema.
- **Timezone Mismatches**: If logs appear with the wrong timestamp, verify the system time and timezone settings on both the Juniper SRX and the Elastic Agent host.
- **Field Mapping Issues**: Ensure the `event.dataset` is correctly set to `juniper_srx.log`. If it is not, the ingest pipeline may not be applied to the incoming data.

## Vendor Resources
- [Juniper SRX Product Documentation](https://www.juniper.net/documentation/en_US/release-independent/junos/information-products/pathway-pages/srx-series/product/)
- [SRX Getting Started - Configure System Logging](https://supportportal.juniper.net/s/article/SRX-Getting-Started-Configure-System-Logging)
- [Overview of System Logging | Junos OS | Juniper Networks](https://www.juniper.net/documentation/us/en/software/junos/network-mgmt/topics/topic-map/system-logging.html)

## Documentation sites

- [Juniper Docs: Structured Data Configuration](https://www.juniper.net/documentation/us/en/software/junos/cli-reference/topics/ref/statement/structured-data-edit-system.html)
- [SRX Getting Started - Configure System Logging](https://supportportal.juniper.net/s/article/SRX-Getting-Started-Configure-System-Logging)
- [Junos OS Application Tracking Documentation](https://www.juniper.net/documentation/us/en/software/junos/application-identification/topics/topic-map/security-application-tracking.html)
- [Juniper SRX Product Documentation](https://www.juniper.net/documentation/en_US/release-independent/junos/information-products/pathway-pages/srx-series/product/)
