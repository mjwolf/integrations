# Service Info

The Fortinet FortiProxy integration allows organizations to ingest and analyze web proxy logs within the Elastic Stack, providing visibility into web traffic, security threats, and user behavior. By centralizing these logs, security teams can correlate proxy activity with other data sources to identify sophisticated attacks and ensure policy compliance.

## Common use cases
- **Security Monitoring and Threat Detection:** Identify malicious web requests, access to known command-and-control (C2) domains, and potential data exfiltration attempts by analyzing proxy traffic patterns. Detect anomalies in user behavior that may indicate compromised credentials.
- **Compliance and Auditing:** Maintain long-term records of user web activity to meet regulatory requirements (such as PCI-DSS, HIPAA, or GDPR) regarding internet usage, data access logs, and administrative configuration changes.
- **Network Troubleshooting:** Use proxy logs to diagnose connectivity issues, identify misconfigured web applications, or investigate performance bottlenecks in web delivery by analyzing response times and HTTP status codes.
- **User Activity Analytics:** Generate reports on top-visited domains, bandwidth usage per user, and blocked content categories to optimize corporate web filtering policies and manage network resources effectively.

## Data types collected
This integration collects several categories of logs from Fortinet FortiProxy, which are mapped to specific data streams:

- **Fortinet FortiProxy logs (tcp):** Collect Fortinet FortiProxy logs using the tcp input. This stream is designed for reliable transmission of system and security events.
- **Fortinet FortiProxy logs (udp):** Collect Fortinet FortiProxy logs using the udp input. This stream is optimized for high-volume log transmission with minimal overhead.
- **Fortinet FortiProxy logs (filestream):** Collect Fortinet FortiProxy logs using filestream input. This stream monitors local log files on the host where the Elastic Agent is installed.

Within these streams, the following information is typically captured:
- **System Event Logs:** Records related to the operation of the FortiProxy unit, including administrator logins, configuration changes, and system health status.
- **Traffic and Web Filtering Logs:** Detailed records of web requests, including URL categories, HTTP status codes, and security actions (allow, block, or monitor).
- **Data Formats:** Logs are ingested in Syslog format, specifically utilizing the Fortinet key-value pair style within the syslog payload.
- **File-based Logs:** When using the filestream input, the integration monitors local log files, defaulting to `/var/log/fortinet-fortiproxy.log`.

## Compatibility
- **Vendor Compatibility:** The integration is compatible with **Fortinet FortiProxy** versions 7.x and above. It has been specifically tested against versions up to 7.4.3. While versions newer than 7.4.3 are expected to be compatible due to the standardized syslog format used by Fortinet, they have not been formally validated.
- **Elastic Stack Requirement:** This integration requires Kibana version `^8.12.2` or `^9.0.0`.

## Scaling and Performance

To ensure optimal performance in high-volume environments, consider the following:
- **Transport/Collection Considerations:** For environments where delivery guarantees are required, use **TCP** (`set mode reliable`). TCP ensures no log messages are lost due to network congestion, though it introduces slightly higher overhead. For maximum throughput where occasional data loss is acceptable, **UDP** provides lower latency and reduced CPU overhead on the FortiProxy appliance.
- **Data Volume Management:** Configure the FortiProxy CLI to filter logs at the source to prevent overwhelming the ingest pipeline. Use the `config log syslogd filter` command on the vendor device to exclude noisy, low-value events or set the `severity` level to `warning` or higher for production environments.
- **Elastic Agent Scaling:** For high-throughput deployments processing thousands of events per second (EPS), deploy multiple Elastic Agents behind a network load balancer to distribute the traffic. Place Agents close to the data source to minimize network latency and distribute the parsing load across multiple host systems.

# Set Up Instructions

## Vendor prerequisites
- **Administrative Access:** Access to the FortiProxy CLI or Web UI with permissions to modify `log syslogd` settings.
- **Network Connectivity:** Unrestricted network access from the FortiProxy appliance to the Elastic Agent on the configured port (default **514**).
- **Firmware Requirement:** Ensure the device is running a supported version (7.x through 7.4.3).
- **System Information:** Knowledge of the Elastic Agent's IP address and the desired protocol (TCP or UDP).

## Elastic prerequisites
- **Kibana Version:** Ensure your Elastic Stack is running version **8.12.2** or higher.
- **Agent Enrollment:** Elastic Agent must be installed and enrolled in a policy via Fleet.
- **Integration Install:** The Fortinet FortiProxy integration must be added to the Agent policy.
- **Host Availability:** Ensure the Elastic Agent host is reachable by the FortiProxy device on the configured syslog port.

## Vendor set up steps

### For Syslog (UDP or TCP) Collection:
You must use the FortiProxy Command Line Interface (CLI) to configure the remote syslog destination.

1.  Log in to the FortiProxy CLI via SSH or the web-based console.
2.  Enter the syslog configuration context:
    ```bash
    config log syslogd setting
    ```
3.  Enable the syslog daemon:
    ```bash
    set status enable
    ```
4.  Specify the IP address of the Elastic Agent:
    ```bash
    set server <ELASTIC_AGENT_IP>
    ```
5.  Configure the transport mode. Use `udp` for standard syslog or `reliable` for TCP:
    ```bash
    set mode reliable
    ```
    *Note: If using `reliable`, the integration expects RFC6587 framing.*
6.  Set the destination port to match your Kibana input configuration (default is `514`):
    ```bash
    set port <PORT_NUMBER>
    ```
7.  Ensure the log format is set to `default` (standard key-value pairs):
    ```bash
    set format default
    ```
8.  (Optional) Set the syslog facility:
    ```bash
    set facility local7
    ```
9.  Save and apply the configuration:
    ```bash
    end
    ```

## Kibana set up steps

1. In Kibana, navigate to **Management > Integrations**.
2. Search for and select **Fortinet FortiProxy**.
3. Click **Add Fortinet FortiProxy**.
4. Configure the desired input types:

### Collecting logs from Fortinet FortiProxy instances via tcp input.
- **Listen Address** (`syslog_host`): The bind address to listen for TCP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
- **Listen Port** (`syslog_port`): The TCP port number to listen on. Default: `514`.
- **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.

### Collecting logs from Fortinet FortiProxy instances via udp input.
- **Listen Address** (`syslog_host`): The bind address to listen for UDP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
- **Listen Port** (`syslog_port`): The UDP port number to listen on. Default: `514`.
- **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.

### Collecting logs from Fortinet FortiProxy instances via filestream input.
- **Paths** (`paths`): The list of relative or absolute paths to the log files where FortiProxy logs are stored. Default: `['/var/log/fortinet-fortiproxy.log']`.
- **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.

5. Click **Save and continue** to deploy the integration to your Elastic Agent policy.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on Fortinet FortiProxy:
- **Generate Web Traffic:** Use a client machine configured to use the FortiProxy as its gateway and browse to several public websites to generate access logs.
- **Interface event:** Toggle a non-production test interface in the CLI: `config system interface`, `edit <interface>`, `set status down`, then `set status up`.
- **Authentication event:** Log out and log back into the FortiProxy administration interface to trigger a system audit event.
- **Policy Change:** Create a temporary test policy and then delete it to generate configuration change logs.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "fortinet_fortiproxy.log"`
4. Verify logs appear in the results table. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should match `fortinet_fortiproxy.log`)
   - `source.ip` (the IP address of the client or the proxy)
   - `event.action` (the action taken by the proxy, e.g., `passthrough`, `block`, or `deny`)
   - `message` (the raw FortiProxy syslog payload)
5. Navigate to **Analytics > Dashboards** and search for "Fortinet FortiProxy" to view the pre-built overview dashboards and verify visual data representation.

# Troubleshooting

## Common Configuration Issues
- **Incorrect TCP Framing**: When using the `reliable` mode on FortiProxy, the device defaults to RFC6587 framing. Ensure the Elastic Agent TCP input is not configured for a different framing method (like octet counting) which would cause parsing failures.
- **Port Conflicts**: Ensure that the port specified (e.g., `514`) is not already in use by another service on the Elastic Agent host. Use `netstat -lnp` or `ss -lnp` on Linux to verify port availability.
- **Firewall Obstructions**: Verify that intermediate firewalls or host-based firewalls (iptables/firewalld) are allowing traffic from the FortiProxy IP to the Agent port.
- **Log Status Disabled**: If no logs are received, verify that `set status enable` was executed under `config log syslogd setting` on the vendor device.

## Ingestion Errors
- **Parsing Failures**: If logs appear in Kibana but contain the `_grokparsefailure` tag, verify that the `format` in FortiProxy is set to `default`. Check the `error.message` field in Discover for specific clues about why the message failed to parse.
- **Mapping Conflicts**: Ensure that no custom ingest pipelines or older versions of the integration are conflicting with the current mapping schema.
- **Field Missing**: If specific fields like `source.ip` are missing, verify that the FortiProxy is actually including that metadata in its syslog output by checking the raw `message` field.

## Vendor Resources
- [FortiProxy CLI Reference: config log syslogd setting](https://docs.fortinet.com/document/fortiproxy/7.4.3/cli-reference/294620/config-log-syslogd-setting)
- [FortiProxy Administration Guide: Log & Report](https://docs.fortinet.com/document/fortiproxy/7.4.3/administration-guide/357693/log-report)

## Documentation sites
- [config log syslogd setting | FortiProxy 7.4.2 | Fortinet Document Library](https://docs.fortinet.com/document/fortiproxy/7.4.2/cli-reference/293620/config-log-syslogd-setting)
- [FortiProxy CLI Reference: config log syslogd setting](https://docs.fortinet.com/document/fortiproxy/7.4.3/cli-reference/294620/config-log-syslogd-setting)
