# Service Info

## Common use cases
Snort is a powerful open-source network intrusion detection system (IDS) and intrusion prevention system (IPS) that monitors network traffic in real-time to identify and block malicious activity. This integration allows organizations to centralize Snort alerts within the Elastic Stack for enhanced visibility and correlation across the following scenarios:
- **Threat Detection and Alerting:** Monitor network traffic for known attack signatures, such as SQL injection, cross-site scripting (XSS), and buffer overflows, to receive immediate notifications of security breaches.
- **Network Traffic Analysis:** Analyze historical network logs to identify patterns of suspicious behavior or unauthorized access attempts that may indicate a persistent threat or reconnaissance activity.
- **Compliance Auditing:** Maintain detailed records of network security events to satisfy regulatory requirements (such as PCI-DSS, HIPAA, or SOC2) that mandate continuous monitoring and logging of security alerts.
- **Incident Response and Forensics:** Provide security analysts with granular details about network events, including source/destination IPs, protocol types, and specific rule triggers, to accelerate investigation and remediation efforts.
- **Rule Performance Monitoring:** Evaluate which Snort rules are triggering most frequently to fine-tune security policies, manage `priority` levels, and reduce false positives.

## Data types collected
This integration collects logs from Snort deployments via the following datastreams:
- **snort.log (Logs):** This is the primary datastream for all Snort alert and event data. It provides:
    - **Intrusion Alerts:** Detailed logs containing signature IDs, timestamps, and descriptive messages about detected network threats (e.g., `snort.log.msg`).
    - **Network Metadata:** Information regarding the packet headers, including **Source IP** (`source.ip`), **Destination IP** (`destination.ip`), **Source Port** (`source.port`), and **Destination Port** (`destination.port`).
    - **Transport Layer Details:** Identification of protocols like `tcp`, `udp`, and `icmp` mapped to the `network.protocol` field.
    - **Structured JSON Logs:** For Snort 3, detailed event data exported in JSON format, capturing complex fields like `pkt_len` (packet length) and `dir` (direction).
    - **Syslog Events:** Standardized system log messages sent via UDP or TCP, containing priority levels and facility codes (e.g., `local7`).
    - **File-based Logs:** Raw text alerts parsed from paths such as `/var/log/snort/alert_json.txt` or `/var/log/snort/alert_fast`.

## Compatibility
The Snort integration is compatible with the following versions and configurations:
- **Snort 3 (Snort++)**: Fully supported (v3.0+) using the `snort.lua` configuration format for JSON and Syslog output.
- **Snort 2.9.x**: Supported (v2.9.13+) via the legacy `snort.conf` configuration using `alert_fast` and `alert_syslog` output plugins.
- **Operating Systems**: Compatible with most Linux distributions (Ubuntu, CentOS, Debian, RHEL) where Snort and Elastic Agent are deployed.
- **Output Formats**: Supports `JSON`, `Fast Alert`, and `Syslog` formats.
- **Elastic Stack**: Requires version **7.14.0** or higher.

## Scaling and Performance
To ensure optimal performance in high-traffic environments, consider the following:
1. **Optimize Output Method:** For environments exceeding 1Gbps of inspected traffic, use the **JSON File** output method in Snort 3. This reduces the CPU overhead compared to direct Syslog forwarding over the network.
2. **Buffer Management:** When using Syslog, ensure the `syslog_port` is not congested. Use the **TCP** input type for reliable delivery if the network experiences packet loss.
3. **Log Rotation:** Use the `logrotate` utility to manage the size of alert files. A sample configuration for `/var/log/snort/*.txt` should include `daily`, `rotate 7`, and `compress` to manage disk space.
4. **Resource Allocation:** 
   - Ensure the Elastic Agent host has at least **2GB of RAM** dedicated to log processing.
   - For high-volume links, increase the **Queue Size** in the Elastic Agent's advanced settings to prevent backpressure.

# Set Up Instructions

## Vendor prerequisites
1. **Administrative Access:** Ensure you have `sudo` or root access to the Snort host to modify `snort.lua` or `snort.conf` and to restart the service.
2. **Snort Installation:** Verify Snort is installed and running using `snort -V`.
3. **Log Permissions:** The Elastic Agent must have read access to the Snort log directory. Run `sudo chmod 755 /var/log/snort` and `sudo chmod 644 /var/log/snort/alert_json.txt`.
4. **Network Access:** If using Syslog, ensure the host's firewall allows traffic on the configured port (e.g., `udp/514` or `tcp/9514`).

## Elastic prerequisites
- **Elastic Agent:** An Elastic Agent must be installed on the Snort host or a central log aggregator and enrolled in a Fleet policy.
- **Elastic Stack Version:** Version **7.14.0** or higher is required to support the ingestion pipelines.
- **Fleet Connectivity:** Ensure the agent is "Healthy" in the **Fleet** > **Agents** tab in Kibana.

## Vendor set up steps

### For Snort 3 (JSON File or Syslog):
1. Open the Snort 3 configuration file: `sudo nano /usr/local/etc/snort/snort.lua`.
2. Locate the `outputs` section.
3. **To enable JSON output (Recommended):** Add the following block to capture all necessary fields:
   ```lua
   alert_json = {
       file = true,
       limit = 100,
       fields = 'timestamp pkt_num proto pkt_gen pkt_len dir src_ap dst_ap rule action msg class priority'
   }
   ```
4. **To enable Syslog output:** Add the `alert_syslog` block:
   ```lua
   alert_syslog = {
       facility = 'local7',
       level = 'alert'
   }
   ```
5. Test the configuration: `snort -c /usr/local/etc/snort/snort.lua -T`.
6. Restart the service: `sudo systemctl restart snort`.

### For Snort 2.9.x (Fast Alert or Syslog):
1. Open the Snort 2 configuration file: `sudo vi /etc/snort/snort.conf`.
2. Navigate to the output plugins section (Step #6).
3. **To enable Fast Alert logging:** Add: `output alert_fast: alert_fast`.
4. **To enable Syslog logging:** Add: `output alert_syslog: LOG_LOCAL7 LOG_ALERT`.
5. Restart Snort: `sudo service snort restart`.

## Kibana set up steps

1. In Kibana, navigate to **Management** > **Integrations**.
2. Search for **Snort** and click on the integration tile.
3. Click **Add Snort**.
4. Configure the integration settings:
    - **Log file (input: logfile)**:
        - **Paths**: Enter the absolute path to the log file, such as `/var/log/snort/alert_json.txt`.
    - **UDP (input: udp)** or **TCP (input: tcp)**:
        - **Listen Host**: Set to `0.0.0.0` to listen on all interfaces or the specific internal IP.
        - **Listen Port**: Enter the port number (e.g., `514` or `9514`) configured in your Syslog daemon.
5. **Dataset name**: Leave as the default `snort.log`.
6. **Custom configurations**:
    - **Tags**: Add tags like `snort-ids` or `dmz-traffic` to help filter data in Discover.
    - **Processors**: (Optional) Add `drop_event` processors if you wish to ignore specific rule IDs or non-critical alerts at the agent level.
7. Click **Save and continue**.
8. Select the **Agent Policy** associated with your Snort host.
9. Click **Save and deploy**.

# Validation Steps

## Trigger Data Flow on Snort:
1. Generate a test alert by triggering a simple rule (e.g., an ICMP echo request if ping rules are enabled):
   `ping -c 4 <SNORT_IP_ADDRESS>`
2. Simulate a malicious request (assuming standard web rules are active):
   `curl http://localhost/?item=/etc/passwd`
3. Verify the log file is being written: `tail -f /var/log/snort/alert_json.txt`.

## Check Data in Kibana:
1. Navigate to **Analytics** > **Discover**.
2. Select the `logs-*` index pattern.
3. Filter the results using the following query: `data_stream.dataset : "snort.log"`.
4. Ensure the following fields are populated:
   - `event.dataset` should be `snort.log`.
   - `snort.log.msg` should contain the rule description.
   - `source.ip` and `destination.ip` should reflect your test traffic.
   - `event.severity` should map the Snort priority level.
5. Open the **[Logs Snort] Overview** dashboard to see visualized alert trends.

# Troubleshooting

## Common Configuration Issues
- **Permission Denied**: If the agent cannot read logs, check permissions with `ls -l /var/log/snort/`. Use `sudo chown root:elastic-agent /var/log/snort/alert_json.txt` if necessary.
- **Port Binding Error**: If the Elastic Agent fails to start the UDP/TCP input, check for port conflicts using `sudo netstat -plnt | grep <PORT_NUMBER>`.
- **Missing Fields**: If using Snort 3, ensure the `fields` string in `snort.lua` includes `timestamp`, `src_ap`, and `dst_ap`. Missing fields may cause the ingestion pipeline to fail.

## Ingestion Errors
- **Grok Failures**: If you see the `_grokparsefailure` tag, verify that you are not using a custom output format in Snort that deviates from the standard `alert_fast` or `alert_json` formats.
- **Timezone Offsets**: If logs appear in the future or past, check the host system time with `date` and ensure the Elastic Agent policy has the correct timezone configured if not using UTC.

## Vendor Resources
- Snort 3 Output Configuration Guide
- [Snort 3 JSON Alert Fields Reference](#alert_json)
- Snort 2.9.x Manual (PDF)

# Documentation sites
- [Snort 3 Official Documentation](start/configuration)
- [Snort Rule Writing Reference](start/rules)
- [Snort 3 FAQ](faq)