# Service Info

## Common use cases

- **Threat Hunting and Incident Response:** Execute point-in-time SQL queries across a distributed fleet to detect Indicators of Compromise (IoCs) such as unauthorized processes, unexpected kernel modules, or modified system files.
- **Compliance and Audit Monitoring:** Track system state changes over time to ensure alignment with security benchmarks (e.g., CIS) by logging changes to user accounts, firewall rules, and listening ports.
- **Asset Inventory and Management:** Maintain a real-time inventory of hardware specifications, installed software packages, and active network connections across all managed endpoints.

## Data types collected

- **Logs:** Collects `result` logs (the output of scheduled differential or snapshot queries) and `status` logs (operational health information about the osquery daemon itself).
- **Events:** Captures system-level events such as file integrity changes and hardware insertions through specific event-based tables (e.g., `file_events`, `hardware_events`).

## Compatibility

- **Operating Systems:** This syslog-based integration is primarily compatible with Linux distributions (e.g., Ubuntu, CentOS, Debian, RHEL) that utilize `rsyslog` or `syslog-ng`.
- **Osquery Versions:** Compatible with osquery versions 4.x and 5.x.
- **Elastic Agent:** Compatible with Elastic Agent 7.13.0 and later.

## Scaling and Performance

- **Watchdog Protection:** Osquery includes a built-in watchdog that monitors resource utilization; it will automatically kill any query that exceeds 10% CPU or 200MB of RAM by default.
- **Throughput:** Scaling is dependent on the frequency of scheduled queries and the volume of "differential" changes; high-frequency logging of rapidly changing tables (e.g., `processes`) can increase syslog overhead and network traffic.

# Set Up Instructions

## Vendor prerequisites

- **Permissions:** Root or sudo access is required to modify osquery flagfiles and system logging configurations.
- **Software:** Osquery (`osqueryd`) must be installed and running on the target host.
- **Logging Service:** A functional local syslog daemon (typically `rsyslog`) must be present to receive and forward logs.

## Elastic prerequisites

- **Elastic Agent:** Must be installed on a host reachable by the osquery nodes via the network.
- **Integration Policy:** An Elastic Agent policy must be configured with a "Custom Syslog" or "UDP/TCP" input to listen on a specific port for incoming data.

## Vendor set up steps

### Part 1: Configure osquery to Use the Syslog Logger
1. Locate your osquery configuration directory, typically `/etc/osquery`.
2. Open your osquery flagfile (usually `/etc/osquery/osquery.flags`) with a text editor.
3. Add or modify the following line to enable the syslog logger plugin:
   ```bash
   --logger_plugin=filesystem,syslog
   ```
4. Save the file and restart the daemon:
   ```bash
   sudo systemctl restart osqueryd
   ```

### Part 2: Configure rsyslog to Forward Logs
1. Create a new configuration file at `/etc/rsyslog.d/60-osquery.conf`.
2. Add the following configuration, replacing `<ELASTIC_AGENT_IP>` and `<ELASTIC_AGENT_PORT>` with your specific values:
   ```text
   # Forward osquery logs to Elastic Agent
   if $programname == 'osqueryd' then @@<ELASTIC_AGENT_IP>:<ELASTIC_AGENT_PORT>
   & stop
   ```
3. Save the file and restart the rsyslog service:
   ```bash
   sudo systemctl restart rsyslog
   ```

## Kibana set up steps

1. In Kibana, navigate to **Management > Integrations**.
2. Search for and select the **Custom UDP/TCP** or **Syslog** integration.
3. Configure the **Listen Host** (usually `0.0.0.0`) and the **Port** to match the port used in your `rsyslog` configuration.
4. Assign the integration to the appropriate **Agent Policy**.
5. Save and deploy the changes to your Elastic Agent.

# Validation Steps

1. **Verify Local Logging:** Check if osquery is successfully sending logs to the local syslog by running `tail -f /var/log/syslog | grep osqueryd`.
2. **Check Agent Connectivity:** Run `elastic-agent status` on the agent host to ensure the integration is healthy.
3. **Data Discovery:** Navigate to **Kibana > Discover** and search for `container.id : *` or filter by `event.dataset : "osquery.result"` to see incoming data in real-time.

# Troubleshooting

## Common Configuration Issues

- **Port Conflicts:** Ensure the port configured in Elastic Agent is not being used by another service on the host. Check using `netstat -tulpn | grep <PORT>`.
- **Flagfile Syntax:** Typos in the osquery flagfile can prevent the daemon from starting. Always validate the flagfile by running `osqueryd --config_check --flagfile=/etc/osquery/osquery.flags`.

## Ingestion Errors

- **Parsing Failures:** If data appears in Kibana with `error.message`, it often indicates the syslog header format is unexpected. Ensure the rsyslog template matches the expected Elastic Syslog format (RFC3164 or RFC5424).
- **Truncated Logs:** Very large query results may be truncated by the syslog daemon's maximum message size settings. Increase `$MaxMessageSize` in `rsyslog.conf` if necessary.

## API Authentication Errors

- Not specified (this integration uses direct syslog forwarding; authentication is handled at the Elastic Agent-to-Elasticsearch level via API Keys).

## Vendor Resources

- [osquery Troubleshooting Guide](https://osquery.readthedocs.io/en/stable/deployment/troubleshooting/)
- [osquery Slack Community](https://osquery.slack.com/)
- [rsyslog Documentation](https://www.rsyslog.com/doc/master/index.html)

# Documentation sites

- [osquery Official Documentation](https://osquery.readthedocs.io/)
- [osquery Logging Deployment Guide](https://osquery.readthedocs.io/en/stable/deployment/logging/)
- [Elastic Integrations Reference](https://www.elastic.co/docs/reference/integrations)