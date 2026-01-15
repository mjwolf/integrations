# Service Info

## Common use cases

The Juniper SRX integration allows organizations to ingest and analyze security and system logs from Juniper SRX Series Gateways, providing deep visibility into network traffic, security threats, and system health.
- **Security Threat Detection:** Monitor Intrusion Detection and Prevention (IDP/IPS) logs and Screen (Denial of Service) events to identify and respond to malicious activities in real-time.
- **Network Traffic Auditing:** Track firewall session starts, closures, and denials to maintain a complete audit trail of network communication and ensure policy compliance.
- **User Activity Monitoring:** Analyze authentication and VPN logs to track remote access patterns and identify potentially compromised user accounts or unauthorized access attempts.
- **System Health and Maintenance:** Monitor hardware alerts, configuration changes, and system errors to proactively manage the availability and reliability of the firewall infrastructure.

## Data types collected

This integration can collect the following types of data:
- **Firewall Session Logs:** Detailed records of traffic flows, including source/destination IPs, ports, protocols, and policy actions (RT_FLOW).
- **IDP and Screen Logs:** Events related to signature-based threat detection and automated DoS protection (RT_IDP, RT_IDS).
- **Security Authentication Logs:** Records of management access attempts and VPN user authentication (RT_AUTH).
- **System and Hardware Logs:** Operational logs regarding chassis health, Junos OS system messages, and configuration changes.
- **Data Formats:** Logs are ingested via Syslog using the **structured-data brief** format (RFC 5424) for optimal parsing performance.
- **Metrics:** While primarily log-based, session counts and event frequencies can be derived from the ingested security logs.

## Compatibility

This integration is compatible with **Juniper SRX Series** devices running **Junos OS version 12.1X46D10 and higher**. It supports logs formatted in standard Junos Syslog format and structured syslog format (RFC5424). Performance may vary based on the specific hardware model and the volume of security events generated.

## Scaling and Performance

- **Transport/Collection Considerations:** The integration primarily uses Syslog over UDP or TCP. UDP is recommended for high-performance environments where low overhead is prioritized, though TCP should be considered if reliable delivery (no packet loss) is a regulatory or technical requirement. Ensure the Elastic Agent is positioned with low latency to the SRX devices to prevent buffer overflows.
- **Data Volume Management:** To prevent overwhelming the Elastic Stack, use the Junos `match` filter at the source to send only relevant security events. For example, filtering for "RT_IDP|RT_IDS|RT_FLOW" ensures that only high-value security and traffic logs are transmitted, while noisy system informational messages are dropped at the device level.
- **Elastic Agent Scaling:** A single Elastic Agent can handle several thousand events per second (EPS) from multiple SRX devices. For high-volume enterprise environments, deploy multiple Elastic Agents behind a network load balancer (for TCP) or use Anycast (for UDP) to distribute the processing load and provide high availability.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** Root or super-user CLI access to the Junos OS command-line interface is required to modify system syslog settings.
- **Network Connectivity:** The Juniper SRX must have a route to the Elastic Agent IP address and be allowed to communicate over the configured Syslog port (e.g., UDP/TCP 9002).
- **Logging Mode Knowledge:** For newer Junos versions (19.3R1+), you must be prepared to switch the security logging mode from "stream" to "event" for syslog compatibility.
- **Resource Availability:** Ensure the SRX device has sufficient CPU and memory headroom to handle the overhead of generating and transmitting structured syslog data.

## Elastic prerequisites

- **Elastic Agent:** An active Elastic Agent must be installed and enrolled in Fleet on a host that is reachable by the Juniper SRX.
- **Integration Policy:** The Juniper SRX integration must be added to an Elastic Agent policy.
- **Port Availability:** The host running the Elastic Agent must have the configured Syslog port (e.g., 9002) open in its local firewall.

## Vendor set up steps

### For System and Security Log Stream Collection:

1.  Log in to the Juniper SRX device's command-line interface (CLI) using SSH or console.
2.  Enter configuration mode:
    ```
    configure
    ```
3.  Set the IP address of the Elastic Agent as the remote syslog host for system (control plane) logs:
    ```
    set system syslog host <elastic-agent-ip> any any
    ```
4.  Configure the structured log format to ensure the Elastic Agent can accurately parse the Junos fields:
    ```
    set system syslog host <elastic-agent-ip> structured-data
    ```
5.  Set the security logging mode to `stream` to offload high-volume traffic logs from the Routing Engine to the Packet Forwarding Engine:
    ```
    set security log mode stream
    set security log source-address <srx-source-ip>
    ```
6.  Define the log stream destination pointing to the Elastic Agent. This ensures security traffic logs are sent directly to the collector:
    ```
    set security log stream elastic_stream format sd-syslog
    set security log stream elastic_stream host <elastic-agent-ip>
    set security log stream elastic_stream port <port-number>
    ```
    *(Note: Replace `<port-number>` with the port configured in the Elastic integration, typically 9514 or 514).*
7.  Enable logging on specific security policies to generate traffic data. This is required as Junos does not log traffic by default:
    ```
    set security policies from-zone trust to-zone untrust policy allow-internet then log session-close
    ```
8.  Commit the configuration to apply the changes:
    ```
    commit
    ```
9.  Verify the status of the log stream:
    ```
    show security log stream
    ```

## Kibana set up steps

### For Juniper SRX Integration Configuration:

1.  Log in to Kibana and navigate to **Management > Integrations**.
2.  Search for **Juniper SRX** and click on the integration tile.
3.  Click **Add Juniper SRX**.
4.  Select the relevant **Data stream** (e.g., `Juniper SRX logs`).
5.  Configure the **Syslog Host** to `0.0.0.0` to listen on all interfaces or provide a specific IP address where the agent is reachable.
6.  Set the **Syslog Port** to the value matching your SRX configuration (e.g., `9514`).
7.  Set the **Protocol** (e.g., `udp` or `tcp`) to match the SRX settings.
8.  Under **Advanced settings**, ensure the **Internal timezone** is correctly set if the SRX is not sending logs in UTC.
9.  Click **Save and continue**, then **Add agent integration to policy** to deploy the configuration to your Elastic Agents.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from Juniper SRX to the Elastic Stack.

### 1. Trigger Data Flow on Juniper SRX:
- **Generate configuration event:** Enter configuration mode and make a non-disruptive change (like adding a description) then commit: `set system description "Elastic Logging Test"`, followed by `commit`.
- **Trigger interface event:** Log out and log back into the CLI or Web UI to generate an authentication and session event.
- **Generate traffic events:** From a host behind the SRX, initiate several connection attempts (e.g., `ping 8.8.8.8` or browse a website) that match a security policy where `log session-close` is enabled.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the following KQL filter: `data_stream.dataset : "juniper_srx.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `juniper_srx.log`)
   - `source.ip` and `destination.ip` (for traffic/security logs)
   - `event.action` or `event.outcome` (e.g., `session-close` or `allow`)
   - `juniper.srx.tag` or `juniper.srx.hostname`
   - `message` (containing the raw Junos log payload)
5. Navigate to **Analytics > Dashboards** and search for "Juniper SRX" to view the pre-built traffic and system dashboards.

# Troubleshooting

## Common Configuration Issues

- **Logs not appearing (Routing Engine vs PFE)**: If you see system logs but no traffic logs, verify that `set security log mode stream` is configured. Traffic logs are handled differently than system logs on the SRX architecture.
- **Source Address Mismatch**: If the SRX sends logs from an interface IP that the Elastic Agent does not recognize or that is blocked by an ACL, logs will be dropped. Explicitly set `set security log source-address` to a known, reachable IP.
- **Structured Data Requirement**: If logs appear as unparsed strings in the `message` field, ensure that `structured-data` is enabled in the system syslog configuration and `sd-syslog` is set for the security log stream.
- **Clock Skew**: If logs appear in the future or the past, ensure the SRX device and the Elastic Agent are both synchronized to the same NTP source. Use `show system ntp status` on the SRX.

## Ingestion Errors

- **Parsing Failures**: If logs contain the `_grokparsefailure` or `_jsonparsefailure` tags, the Juniper SRX might be sending logs in a non-standard format. Ensure the SRX is using the default Junos syslog format and that "structured-data" is enabled if required by the specific version of the integration.
- **Mapping Issues**: If specific fields are missing, check the `error.message` field in Kibana. This can happen if the log message length exceeds the default Syslog buffer or if a Junos OS update has changed the log header format.
- **Truncated Logs**: Large IDP or UTM logs can sometimes be truncated by the network transport. If logs appear incomplete, consider switching from UDP to TCP or increasing the `max_message_size` in the Elastic Agent integration settings.

## Vendor Resources

- [SRX Getting Started - Configure System Logging - Juniper Networks](https://supportportal.juniper.net/s/article/SRX-Getting-Started-Configure-System-Logging)
- [Overview of System Logging | Junos OS | Juniper Networks](https://www.juniper.net/documentation/us/en/software/junos/network-mgmt/topics/topic-map/system-logging.html)

## Documentation sites

- [Juniper SRX IDP (IDS/IPS) and SCREEN (DoS) Logs to Splunk](https://icookservers.blog/2017/10/13/juniper-srx-idp-and-screen-logs-to-splunk/)
- Refer to the official vendor website.
