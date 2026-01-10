# Service Info

## Common use cases

*   **Network Security Monitoring:** Detect and investigate suspicious network activities by analyzing protocol-specific logs such as DNS queries, SSL certificates, and HTTP requests.
*   **Incident Response:** Perform deep forensic analysis of network traffic after a security event by pivoting through connection logs (conn.log) to identify lateral movement or data exfiltration.
*   **Compliance and Auditing:** Maintain a historical record of all network transactions and protocol metadata for regulatory compliance requirements.

## Data types collected

This integration primarily collects **logs**. These logs encompass various network protocols and metadata, including connection details (TCP/UDP/ICMP), HTTP transactions, DNS lookups, SSL/TLS handshakes, and file metadata extracted from network streams.

## Compatibility

This integration is compatible with all modern versions of **Zeek (formerly Bro)** that support JSON output, including versions 3.x, 4.x, 5.x, and 6.x. It has been tested with standard Linux distributions (Ubuntu, CentOS/RHEL) where Zeek is deployed as a network sensor.

## Scaling and Performance

Zeek can generate significant volumes of log data depending on network throughput. Performance is primarily determined by Zeek's ability to process traffic (e.g., using AF_PACKET or PF_RING for high-speed capture) and the disk I/O available for writing JSON logs. For high-traffic environments, ensure the logging directory is on a high-performance filesystem and monitor the Elastic Agent's CPU/memory usage for log harvesting.

# Set Up Instructions

## Vendor prerequisites

*   **Zeek Installation:** A working installation of Zeek (locally or in a cluster).
*   **Administrative Access:** Sudo or root permissions to modify Zeek configuration files and create logging directories.
*   **Storage Space:** Sufficient disk space to store local JSON logs before they are harvested by the Elastic Agent.

## Elastic prerequisites

*   **Elastic Agent:** Must be installed on the same host where Zeek is running.
*   **Fleet Management:** An active Fleet Server or standalone Agent policy.
*   **Agent Permissions:** The Elastic Agent must have read permissions for the directory where Zeek writes its logs.

## Vendor set up steps

1.  **Locate Configuration:** Find your `local.zeek` file, typically located at `/opt/zeek/share/zeek/site/local.zeek` or `/usr/local/zeek/share/zeek/site/local.zeek`.
2.  **Enable JSON Output:** Add the following lines to the end of `local.zeek` to ensure logs are structured for Elastic:
    ```zeek
    # Reconfigure the default ASCII writer to output in JSON format
    redef LogAscII::use_json = T;

    # Specify a directory for the JSON logs
    redef Log::default_logdir = /var/log/zeek/json;
    ```
3.  **Prepare Directory:** Create the logging directory and assign the correct ownership (replace `zeek` with your Zeek user):
    ```bash
    sudo mkdir -p /var/log/zeek/json
    sudo chown zeek:zeek /var/log/zeek/json
    ```
4.  **Deploy Changes:** Restart or redeploy Zeek using `zeekctl`:
    ```bash
    sudo zeekctl check
    sudo zeekctl deploy
    ```

## Kibana set up steps

1.  **Add Integration:** In Kibana, go to **Management > Integrations** and search for "Zeek".
2.  **Configure Integration:** Select **Add Zeek** and choose the "Custom logs" or specific Zeek input if available in your version.
3.  **Define Paths:** In the configuration settings, set the **Log file path** to: `/var/log/zeek/json/*.log`.
4.  **Apply Policy:** Save the integration and ensure it is assigned to the Agent policy running on the Zeek sensor.

# Validation Steps

1.  **Check Local Logs:** Verify that files are being created in `/var/log/zeek/json/` and that they contain JSON-formatted text: `tail -f /var/log/zeek/json/conn.log`.
2.  **Monitor Agent Logs:** Check the Elastic Agent logs for successful harvesting: `elastic-agent logs`.
3.  **Discover Tab:** Navigate to **Kibana > Discover** and search for `event.dataset: "zeek.*"` to confirm data is appearing in Elasticsearch.
4.  **Dashboards:** Open the **[Logs Zeek] Overview** dashboard in Kibana to view visualized network traffic data.

# Troubleshooting

## Common Configuration Issues

*   **Logs remain in ASCII:** Ensure `redef LogAscII::use_json = T;` is correctly typed in `local.zeek` and that `zeekctl deploy` was executed.
*   **Permission Denied:** If Elastic Agent cannot read the logs, ensure the directory `/var/log/zeek/json` has read and execute permissions for the agent user (e.g., `chmod 755 /var/log/zeek/json`).
*   **Missing Directory:** If Zeek fails to start, verify that the directory defined in `Log::default_logdir` exists and is writable by the Zeek user.

## Ingestion Errors

*   **Parsing Failures:** If `error.message` appears, it is often due to mixed log formats in the same directory. Ensure only JSON logs are present in the path monitored by the Elastic Agent.
*   **Timezone Mismatch:** Zeek logs timestamps in UTC by default; ensure your Kibana view or ingest pipeline is not applying an incorrect offset.

## API Authentication Errors

Not specified (Zeek log collection via Elastic Agent is file-based and does not use a vendor API).

## Vendor Resources

*   [Zeek Troubleshooting Guide](https://docs.zeek.org/en/master/troubleshooting.html)
*   [Zeek Mailing List & Community Slack](https://zeek.org/community/)
*   [Zeek GitHub Issues](https://github.com/zeek/zeek/issues)

# Documentation sites

*   [Zeek Official Documentation](https://docs.zeek.org/)
*   [Zeek Logging Framework Reference](https://docs.zeek.org/en/master/frameworks/logging.html)
*   [Elastic Zeek Integration Guide](https://www.elastic.co/docs/reference/integrations/zeek)
*   [Zeek Scripting Reference](https://docs.zeek.org/en/master/script-reference/index.html)