# Service Info

## Common use cases

*   **Corporate Web Monitoring:** Monitor internal employee internet usage to ensure compliance with security policies and detect unauthorized access to restricted web categories.
*   **Bandwidth Optimization:** Analyze cache hit/miss ratios and content types to optimize web delivery and reduce wide area network (WAN) costs in enterprise environments.
*   **Security Threat Detection:** Identify high-frequency requests, abnormal traffic patterns, or access to known malicious domains through detailed analysis of proxy access logs.

## Data types collected

This integration primarily collects **logs**, specifically access logs (tracking client requests, destinations, and status codes) and cache logs (system-level events and errors related to the proxy service).

## Compatibility

This integration is compatible with Squid versions 3.x, 4.x, 5.x, and 6.x. It supports standard NCSA and Squid native log formats, as well as custom log formats defined via the `logformat` directive.

## Scaling and Performance

Squid is designed for high-performance caching and can handle thousands of concurrent connections. When forwarding logs via UDP, the performance impact is minimal; however, using TCP ensures delivery at the cost of slight overhead. For high-volume environments, it is recommended to use Squid’s SMP (Symmetric Multi-Processing) features to distribute logging tasks across multiple workers.

# Set Up Instructions

## Vendor prerequisites

*   A running instance of Squid Proxy with root or sudo access to the host machine.
*   Read/write permissions for the Squid configuration file (typically located at `/etc/squid/squid.conf`).
*   Network connectivity between the Squid host and the Elastic Agent on the specified UDP, TCP, or Syslog ports.

## Elastic prerequisites

*   Elastic Agent must be installed and enrolled in a policy via Fleet.
*   The "Custom Logs" or "Squid" integration must be added to the agent policy.
*   The Elastic Agent must be configured with a listener (UDP, TCP, or Syslog) matching the port and protocol chosen in the Squid configuration.

## Vendor set up steps

1.  **Locate the Squid Configuration File**: Open your `squid.conf` file, usually found at `/etc/squid/squid.conf` or `/usr/local/squid/etc/squid.conf`.
2.  **Configure Log Forwarding**: Add one of the following directives to forward logs to your Elastic Agent (replace `ELASTIC_AGENT_IP` and `PORT` with your agent's details):
    *   **UDP (Recommended)**: `access_log udp://ELASTIC_AGENT_IP:PORT`
    *   **TCP**: `access_log tcp://ELASTIC_AGENT_IP:PORT`
    *   **Syslog**: `access_log syslog:local0.info`
3.  **Define Log Format**: To ensure optimal parsing, use the native squid format:
    ```
    logformat squid %ts.%03tu %6tr %>a %Ss/%03>Hs %<st %rm %ru %un %Sh/%<a %mt
    access_log udp://ELASTIC_AGENT_IP:PORT logformat=squid
    ```
4.  **Apply Changes**: Reload the configuration by running `sudo squid -k reconfigure` or restart the service using `sudo systemctl restart squid`.

## Kibana set up steps

1.  In Kibana, navigate to **Management > Integrations**.
2.  Search for and select the **Squid** integration.
3.  Click **Add Squid** and select the agent policy where your Elastic Agent is enrolled.
4.  Under the **Log stream** settings, configure the **Listen Port** and **Listen Address** to match the `access_log` settings defined in your `squid.conf`.
5.  Click **Save and continue** to deploy the configuration to your agents.

# Validation Steps

1.  **Generate Test Traffic**: From a client configured to use the Squid proxy, visit several websites to generate access log entries.
2.  **Verify Service Status**: Ensure Squid is running without errors by checking `systemctl status squid`.
3.  **Check Data Ingestion**: In Kibana, go to **Discover** and filter for `event.dataset : "squid.access"`. You should see incoming log entries within 1-2 minutes.
4.  **Review Dashboards**: Open the **[Logs Squid] Overview** dashboard to verify that visualizations for traffic volume and status codes are populating correctly.

# Troubleshooting

## Common Configuration Issues

*   **Connectivity Refused**: Ensure that the Elastic Agent host firewall allows inbound traffic on the specific UDP/TCP port used in `squid.conf`.
*   **Permission Denied**: If using Syslog, ensure the system's syslog daemon (rsyslog or syslog-ng) has permissions to forward to external network destinations.
*   **Pathing Errors**: Ensure the `access_log` directive in `squid.conf` does not conflict with local file logging; both can be used simultaneously if specified on separate lines.

## Ingestion Errors

*   **Parsing Failures**: If the `error.message` field indicates a parsing error, verify that the `logformat` in `squid.conf` exactly matches the expected Squid native format. 
*   **Timestamp Mismatch**: Ensure the Squid server and Elastic Agent host have synchronized clocks via NTP to avoid data appearing in the future or past.

## API Authentication Errors

*   **Access Denied**: Squid typically uses ACLs (Access Control Lists) for traffic, not for log forwarding. However, if using an authenticated proxy, ensure the `%un` (username) field is being correctly captured in the log format for Elastic to map identity fields.

## Vendor Resources

*   [Squid Configuration Knowledge Base](http://wiki.squid-cache.org/KnowledgeBase/)
*   [Squid FAQ - Troubleshooting](http://wiki.squid-cache.org/SquidFaq/Troubleshooting)
*   [Squid Mailing List Archives](http://www.squid-cache.org/Support/mailing-lists.html)

# Documentation sites

*   [Squid Official Documentation](http://www.squid-cache.org/Doc/)
*   [Squid access_log Directive Reference](http://www.squid-cache.org/Doc/config/access_log/)
*   [Squid Wiki Configuration Examples](http://wiki.squid-cache.org/ConfigExamples)