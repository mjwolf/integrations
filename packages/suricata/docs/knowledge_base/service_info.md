# Service Info

## Common use cases

*   **Intrusion Detection and Prevention (IDS/IPS):** Real-time monitoring of network traffic to identify and block known threats, exploits, and malware command-and-control (C2) communications.
*   **Network Security Monitoring (NSM):** Collecting detailed protocol metadata (HTTP, DNS, TLS) and flow data to perform post-incident forensic investigations and threat hunting.
*   **Protocol Compliance and Auditing:** Tracking the use of specific protocols and encryption standards across the network to ensure compliance with internal security policies.

## Data types collected

This integration collects comprehensive network telemetry in the form of **Logs** and **Events** via the Suricata EVE JSON format. Specific data points include alerts, network flows, anomalies, and rich protocol metadata for HTTP, DNS, TLS, SMTP, SSH, and file transactions.

## Compatibility

*   **Suricata:** Versions 4.0.0 and newer are supported (Suricata 6.x and 7.x are recommended for full feature support).
*   **Elastic Stack:** Compatible with Elastic Agent versions 7.14.0 and higher.
*   **Operating Systems:** Linux-based distributions (Ubuntu, CentOS, Debian, RHEL) where Suricata is natively supported.

## Scaling and Performance

Suricata can generate high volumes of data depending on network throughput. To ensure performance, use the `regular` filetype for EVE logs and ensure the underlying storage has sufficient IOPS. For high-traffic environments (10Gbps+), it is recommended to tune Suricata worker threads and use a dedicated ingestion layer to prevent log processing delays.

# Set Up Instructions

## Vendor prerequisites

*   Suricata must be installed and running on the target host.
*   Root or sudo privileges are required to modify `suricata.yaml` and restart the service.
*   Adequate disk space for stored JSON logs, as EVE output can grow rapidly.

## Elastic prerequisites

*   Elastic Agent must be installed on the same host as Suricata or have access to the log files via a network share.
*   The Elastic Agent must be enrolled in a policy that includes the "Suricata" integration.
*   Read permissions for the Elastic Agent user on the Suricata log directory (typically `/var/log/suricata/`).

## Vendor set up steps

1.  **Locate the configuration**: Find your `suricata.yaml` file, typically located at `/etc/suricata/suricata.yaml`.
2.  **Enable EVE logging**: Open the file and locate the `outputs` section. Ensure the `eve-log` section is enabled with `enabled: yes` and `filetype: regular`.
3.  **Configure event types**: Under the `types` subsection of `eve-log`, list the protocols and events to monitor (e.g., alert, anomaly, http, dns, tls, flow).
4.  **Validate syntax**: Run `sudo suricata -T -c /etc/suricata/suricata.yaml -v` to ensure the configuration is valid.
5.  **Restart Suricata**: Apply changes by restarting the service: `sudo systemctl restart suricata`.

## Kibana set up steps

1.  Log in to Kibana and navigate to **Management > Integrations**.
2.  Search for **Suricata** and select it.
3.  Click **Add Suricata** to configure the integration.
4.  Specify the **Log file path** (default is `/var/log/suricata/eve.json`) in the integration settings.
5.  Select the **Agent Policy** where you want to apply this integration and click **Save and continue**.

# Validation Steps

1.  **Check log generation**: Run `tail -f /var/log/suricata/eve.json` on the host to confirm Suricata is writing JSON entries.
2.  **Verify Agent status**: In Kibana, go to **Fleet > Agents** and ensure the agent status is "Healthy."
3.  **Inspect Ingested Data**: Use **Discover** in Kibana to search for `event.dataset: suricata.eve`. You should see incoming documents.
4.  **Review Dashboards**: Open the **[Logs Suricata] Overview** dashboard in Kibana to visualize alerts and network activity.

# Troubleshooting

## Common Configuration Issues

*   **Permissions Errors**: If Elastic Agent cannot read `eve.json`, ensure the agent user (usually `elastic-agent` or `root`) has read permissions for the file and execute permissions for the parent directories.
*   **Incorrect File Path**: Verify that the `filename` in `suricata.yaml` matches the path configured in the Kibana integration settings.
*   **Suricata Restart Failure**: If Suricata fails to restart, check `journalctl -u suricata` for syntax errors in the YAML file.

## Ingestion Errors

*   **Malformed JSON**: If logs appear in `error.message` as parsing failures, ensure that Suricata is not outputting multi-line formatting or custom log formats that deviate from standard EVE JSON.
*   **Timestamp Mismatch**: If data isn't appearing in the current time range, check the system clock on the Suricata host; Elastic Agent uses the `@timestamp` field from the log.

## API Authentication Errors

Not specified (Suricata integration primarily uses file-based ingestion rather than API-based collection).

## Vendor Resources

*   [Suricata Troubleshooting Guide](https://deepwiki.com/anpa6841/suricata-lab/6-troubleshooting-and-faqs)
*   [Suricata Community Forums](https://forum.suricata.io/)
*   [Suricata Discord/Support Channels](https://suricata.io/community/)

# Documentation sites

*   [Suricata EVE JSON Output Guide](https://docs.suricata.io/en/latest/output/eve/eve-json-output.html)
*   [Official Suricata Documentation](https://docs.suricata.io/en/latest/index.html)
*   [Elastic Suricata Integration Reference](https://docs.elastic.co/integrations/suricata)