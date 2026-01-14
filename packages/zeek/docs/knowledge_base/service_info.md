# Service Info

The Zeek integration for Elastic Agent provides a robust framework for transforming network traffic data into actionable security intelligence. By monitoring network activity at scale, this integration allows security analysts and network administrators to maintain deep visibility into their environment by converting raw packet captures into structured, searchable events.

## Common use cases

- **Network Threat Hunting:** Leverage detailed protocol logs (DNS, HTTP, SSL/TLS, etc.) to identify anomalous patterns, beaconing behavior, or lateral movement within the network.
- **Incident Response and Forensics:** Use the rich, high-fidelity metadata captured by Zeek to reconstruct the chain of events during a security breach, providing context that standard firewall logs often miss.
- **Compliance and Auditing:** Maintain long-term records of network connections and service usage to meet regulatory requirements (e.g., PCI-DSS, HIPAA) and ensure all internal security policies are being followed.
- **Performance Monitoring:** Analyze protocol-specific metrics, such as SSL certificate expiration dates or DNS response times, to identify service degradations and misconfigured network infrastructure.
- **Lateral Movement Detection:** Monitor internal traffic logs for unusual RDP or SSH connections between internal hosts that do not typically communicate.
- **Protocol Enforcement:** Identify use of non-standard ports for common protocols or the use of insecure protocols like Telnet or unencrypted HTTP.

## Data types collected

This integration collects structured logs from a wide variety of Zeek log files. Each log type is mapped to a specific Elastic data stream and follows the Elastic Common Schema (ECS):

- **Connection Logs (`zeek.connection`):** Provides metadata for every TCP/UDP/ICMP connection, including durations, byte counts, and connection states.
- **DNS Activity (`zeek.dns`):** Records of DNS queries and responses, including requested domains, record types, and response codes.
- **HTTP Traffic (`zeek.http`):** Detailed analysis of HTTP requests and responses, capturing methods, URIs, user agents, and status codes.
- **Encrypted Traffic (`zeek.ssl`, `zeek.x509`):** Metadata about SSL/TLS handshakes, certificate chains, and encryption strength.
- **Authentication Logs (`zeek.ssh`, `zeek.kerberos`, `zeek.radius`):** Monitoring of login attempts and authentication protocols across the network.
- **File Analysis (`zeek.files`, `zeek.pe`):** Metadata about files transferred over the network, including hashes (MD5, SHA1, SHA256) and MIME types.
- **Network Services (`zeek.dhcp`, `zeek.ftp`, `zeek.smtp`, `zeek.snmp`, `zeek.sip`, `zeek.smb`):** Protocol-specific details for common infrastructure and communication services.
- **Security Context (`zeek.notice`, `zeek.weird`, `zeek.intel`):** Alerts for anomalous activity, protocol violations, and matches against threat intelligence indicators.
- **Industrial Control Systems (`zeek.dnp3`, `zeek.modbus`):** Monitoring of specialized ICS/SCADA protocols for industrial environments.

## Compatibility

- **Zeek Version:** This integration is compatible with **Zeek (formerly Bro)** version 3.0 and higher. Full support is provided for modern LTS releases including **Zeek 4.0, 5.0, and 6.0**.
- **Log Format:** The Zeek instance must be configured for **JSON output** for the integration to parse fields correctly.

## Scaling and Performance

- **JSON Processing Overhead:** While JSON logging is required for high-fidelity parsing, it increases CPU and disk I/O load on the sensor. Ensure the hardware has sufficient IOPS to handle simultaneous write (Zeek) and read (Agent) operations.
- **Log Rotation:** It is highly recommended to use `zeekctl` to manage log rotation. Elastic Agent should be configured to monitor the `current` directory, while older logs are compressed and moved to prevent disk exhaustion.
- **High-Volume Environments:**
    - For environments exceeding 1Gbps, use Zeek's **Cluster Mode** to distribute traffic across multiple worker nodes.
    - Deploy an **Elastic Agent** on each Zeek worker node to distribute the ingestion load and minimize network overhead.
    - Consider using **AF_PACKET** or **PF_RING** for efficient packet capture on high-speed interfaces to reduce dropped packets.

# Set Up Instructions

## Vendor prerequisites

1. **Administrative Access:** Root or sudo privileges on the Zeek sensor are required to modify configuration files and restart services.
2. **JSON Support:** The Zeek installation must include the `policy/tuning/json-logs.zeek` script, which is part of the standard distribution.
3. **Disk Space:** JSON logs are typically **2x-3x larger** than standard TSV logs. Ensure sufficient free disk space for log buffering and rotation.

## Elastic prerequisites

1. **Elastic Agent Deployment:** A deployed and enrolled Elastic Agent in an active Agent Policy. Refer to the [Elastic Agent Installation Guide](/docs/reference/fleet/install-elastic-agents) for setup instructions.
2. **Permissions:** The Elastic Agent service account must have read permissions for the Zeek log directory.
    - Example: `sudo setfacl -R -m u:elastic-agent:rx /opt/zeek/logs/`

## Vendor set up steps

1.  **Access the Sensor:** Log into the Zeek sensor via SSH.
2.  **Locate Site Configuration:** Navigate to the site-specific configuration directory, typically found at `/usr/local/zeek/share/zeek/site/` or `/opt/zeek/share/zeek/site/`.
3.  **Edit local.zeek:** Open the `local.zeek` file for editing:
    ```bash
    sudo nano /opt/zeek/share/zeek/site/local.zeek
    ```
4.  **Enable JSON Policy:** Add the following line to the bottom of the file to ensure all logs are output in JSON format:
    ```zeek
    @load policy/tuning/json-logs
    ```
5.  **Verify Configuration:** Run the `zeekctl` check command to ensure there are no syntax errors in your scripts:
    ```bash
    sudo zeekctl check
    ```
6.  **Deploy Changes:** Apply the new policy and restart the Zeek processes:
    ```bash
    sudo zeekctl deploy
    ```
7.  **Confirm Output:** Verify that the log files now contain JSON objects instead of tab-separated values:
    ```bash
    head -n 1 /opt/zeek/logs/current/conn.log
    ```

## Kibana set up steps

1.  **Navigate to Integrations:** In Kibana, go to **Management** > **Integrations** and search for **Zeek**.
2.  **Add Zeek:** Click **Add Zeek**.
3.  **Configure Input Type:**:
    - **Log file:**
        - **Paths:** Enter the absolute path to your Zeek logs. The default for many installations is `/opt/zeek/logs/current/*.log`.
4.  **Configure Integration Variables:**
    - **Tags:** Add custom strings (e.g., `zeek-sensor-north-dc`) to the **Tags** field to simplify filtering in multi-sensor environments.
    - **Processors:** Use the **Processors** text area to add YAML configuration for data enrichment, such as `add_host_metadata` or `add_fields`.
    - **Ignore Older:** Set the `ignore_older` value (e.g., `72h`) to prevent the agent from processing very old logs during initial startup.
5.  **Apply Policy:** Select the **Existing Policy** that is currently assigned to your Zeek sensor host.
6.  **Save and Deploy:** Click **Save and continue**. The Elastic Agent will automatically receive the updated configuration and begin shipping network logs to the Elastic Stack.

# Validation Steps

1.  **Trigger Network Activity:**
    - Generate identifiable traffic on the Zeek sensor to ensure logs are being generated:
      ```bash
      curl https://www.elastic.co
      nslookup elastic.co
      ```
    - Verify logs are updating locally by checking the file timestamps: `ls -lh /opt/zeek/logs/current/`.

2.  **Verify Using Dashboards:**
    - Navigate to **Analytics** > **Dashboard**.
    - Search for and open the **[Logs Zeek] Overview** dashboard.
    - Confirm that visualizations for "Top Talkers," "Connection Count," and "Network Protocols" are displaying live data.


2.  **Check Data in Kibana Discover:**
    - Navigate to **Analytics** > **Discover**.
    - Select the index pattern `logs-*` or `metrics-*`.
    - Enter the following filter in the search bar to verify the connection data stream:
      `data_stream.dataset : "zeek.connection"`
    - Click **Refresh**. Verify that fields such as `source.ip`, `destination.port`, `zeek.connection.state`, and `network.bytes` are populated with data.

# Troubleshooting

## Common Configuration Issues

- **Logs are still in TSV format**: The integration requires JSON log output. Ensure `zeekctl deploy` was executed after modifying `local.zeek`. If the issue persists, check the end of `local.zeek` for any `redef LogAscii::use_json = F;` lines that might be manually disabling JSON output.
- **Permission Denied**: The Elastic Agent user (often `elastic-agent` or `root`) must have access to the log directory. Use Access Control Lists (ACLs) for granular permissions:
  `sudo setfacl -R -m u:elastic-agent:rx /opt/zeek/logs/current/`
- **Missing Log Files**: Zeek only creates specific log files (like `dns.log` or `http.log`) when that specific activity is detected. If a log file is missing, generate relevant traffic on the network.

## Ingestion Errors

- **JSON Parsing Errors**: If fields are not mapping correctly, check the `error.message` field in Kibana. This can occur if custom Zeek scripts are modified to produce non-standard JSON fields that conflict with the Elastic Common Schema.
- **Timestamp Mismatch**: If logs appear in the "future" or "past" in Kibana, check the sensor's system clock using the `date` command and ensure the Elastic Agent's system timezone matches the environment expectations.

## Vendor Resources

- [Zeek JSON Logging Reference](https://docs.zeek.org/en/master/scripts/policy/tuning/json-logs.zeek.html)
- [Zeek Logging Framework Overview](https://docs.zeek.org/en/master/frameworks/logging.html)
- [Zeek Log Formats](https://docs.zeek.org/en/master/log-formats.html)
- [Zeek Control (zeekctl) Documentation](https://deepwiki.com/zeek/zeekctl/1-overview)

# Documentation sites

- [Official Zeek Documentation Home](https://docs.zeek.org/en/master/index.html)
- [Zeek Quick Start Guide](https://docs.zeek.org/en/lts/quickstart.html)
