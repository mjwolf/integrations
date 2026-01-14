# Zeek

> **Note**: This documentation was generated using AI and should be reviewed for accuracy.

## Overview

This integration collects logs from [Zeek](https://www.zeek.org/), a powerful, open-source network traffic analyzer formerly known as Bro. It ingests the various logs Zeek produces about the network traffic it analyzes, providing rich data for security monitoring, threat hunting, and network forensics.

To use this integration, you must configure Zeek to output its logs in JSON format. This is typically done by loading the `json-logs` policy. For more details, refer to the [Zeek JSON Logging Reference](https://docs.zeek.org/en/master/scripts/policy/tuning/json-logs.zeek.html).

### Compatibility

This integration has been tested with Zeek 2.6.1 but is expected to work with other versions of Zeek that support JSON-formatted logs. Zeek requires a Unix-like platform and currently supports Linux, FreeBSD, and macOS.

### How it works

The Zeek integration uses the Elastic Agent with a `logfile` input to collect logs directly from files. The agent monitors the specified log directory on your Zeek sensor, reads new log entries as they are written, parses the JSON format, and ships the data to your Elastic cluster. This provides a real-time stream of network metadata for analysis in Kibana.

## What data does this integration collect?

This integration collects a comprehensive set of logs from Zeek, each corresponding to a specific network protocol or activity type. These logs provide deep visibility into network behavior.

**Data Streams:**
- `capture_loss`: Packet loss rate data.
- `connection`: TCP/UDP/ICMP connection summaries.
- `dce_rpc`: Distributed Computing Environment/RPC data.
- `dhcp`: DHCP lease activity.
- `dnp3`: DNP3 industrial control protocol requests and replies.
- `dns`: DNS queries and responses.
- `dpd`: Dynamic protocol detection failures.
- `files`: File analysis results from network traffic.
- `ftp`: FTP commands and replies.
- `http`: HTTP requests and replies.
- `intel`: Matches from Zeek's Intelligence Framework.
- `irc`: IRC commands and responses.
- `kerberos`: Kerberos authentication data.
- `known_certs`: SSL/TLS certificates observed on the network.
- `known_hosts`: Timestamps for newly observed hosts.
- `known_services`: Timestamps for newly observed services.
- `modbus`: Modbus industrial control protocol commands and responses.
- `mysql`: MySQL protocol data.
- `notice`: Zeek notices and alerts.
- `ntlm`: NT LAN Manager (NTLM) authentication data.
- `ntp`: Network Time Protocol data.
- `ocsp`: Online Certificate Status Protocol (OCSP) data.
- `pe`: Portable Executable file analysis data.
- `radius`: RADIUS authentication attempts.
- `rdp`: Remote Desktop Protocol data.
- `rfb`: Remote Framebuffer (VNC) data.
- `signature`: Zeek signature engine matches.
- `sip`: Session Initiation Protocol data.
- `smb_cmd`: SMB commands.
- `smb_files`: SMB file transfers.
- `smb_mapping`: SMB tree mappings.
- `smtp`: SMTP transactions.
- `snmp`: SNMP messages.
- `socks`: SOCKS proxy requests.
- `software`: Details on software and services operated by network hosts.
- `ssh`: SSH connection data.
- `ssl`: SSL/TLS handshake information.
- `stats`: Zeek performance and resource statistics.
- `syslog`: Syslog messages captured from traffic.
- `traceroute`: Traceroute detections.
- `tunnel`: Tunneling protocol events (e.g., GRE, VXLAN).
- `weird`: Unusual or unexpected network activity.
- `x509`: X.509 certificate information and analysis.

### Supported use cases

The data collected by the Zeek integration supports a wide range of security and operational use cases, including:
- **Network Security Monitoring (NSM):** Gaining deep visibility into all network traffic to detect anomalies and threats.
- **Threat Hunting:** Proactively searching for indicators of compromise (IoCs) within network metadata.
- **Incident Response:** Investigating security alerts by analyzing historical connection logs, file transfers, and protocol-specific details.
- **Network Forensics:** Reconstructing network events and user activity during post-breach analysis.
- **Operational Intelligence:** Monitoring network performance, identifying misconfigurations, and understanding application behavior.

## What do I need to use this integration?

Before you begin, ensure you have the following prerequisites:

- **Administrative Access:** You need root or `sudo` privileges on the Zeek sensor to modify configuration files (`local.zeek`) and restart the Zeek service using `zeekctl`.
- **JSON Support:** Your Zeek installation must include the `policy/tuning/json-logs.zeek` script. This script is part of the standard Zeek distribution.
- **Disk Space:** JSON logs are typically 2 to 3 times larger than the standard tab-separated logs. Ensure you have sufficient free disk space on the Zeek sensor for log buffering and rotation.
- **Elastic Agent:** An Elastic Agent must be installed on the Zeek sensor or on a log collector with access to the Zeek log files.

## How do I deploy this integration?

### Agent-based deployment

This integration is deployed using the Elastic Agent. The agent runs on your Zeek sensor, collects the log files, and securely ships them to your Elastic deployment.

### Onboard and configure

Follow these steps to configure Zeek and set up the integration in Fleet.

#### 1. Configure Zeek for JSON output

You must enable JSON logging on your Zeek sensor for this integration to work correctly.

1.  **Access the Sensor:** Log into your Zeek sensor host using SSH or a terminal.
2.  **Locate Site Configuration:** Navigate to the site-specific configuration directory. The path is typically `/usr/local/zeek/share/zeek/site/` or `/opt/zeek/share/zeek/site/`.
3.  **Edit `local.zeek`:** Open the `local.zeek` file in a text editor.
    ```bash
    sudo nano /opt/zeek/share/zeek/site/local.zeek
    ```
4.  **Enable JSON Policy:** Add the following line to the end of the file. This instructs Zeek to write all logs in JSON format.
    ```zeek
    @load policy/tuning/json-logs
    ```
5.  **Verify Configuration:** Run `zeekctl check` to validate the syntax of your configuration files.
    ```bash
    sudo zeekctl check
    ```
6.  **Deploy Changes:** Apply the new configuration and restart the Zeek processes.
    ```bash
    sudo zeekctl deploy
    ```
7.  **Confirm Output Format:** Verify that log files in `/opt/zeek/logs/current/` are now being written in JSON format.
    ```bash
    # This should output a JSON object
    head -n 1 /opt/zeek/logs/current/conn.log
    ```

#### 2. Configure the integration in Fleet

1.  **Navigate to Integrations:** In Kibana, go to **Management** > **Integrations**.
2.  **Add Zeek Integration:** Search for "Zeek" and click on it to open the integration page. Click **Add Zeek**.
3.  **Configure Integration Settings:**
    - **Integration name:** Provide a descriptive name for your Zeek integration instance.
    - **Paths:** Enter the absolute path pattern for your Zeek log files. The default for many installations is `/opt/zeek/logs/current/*.log`.
    - **Tags:** (Optional) Add custom tags (e.g., `zeek-sensor-prod-dc1`) to help you filter and identify data from different sensors.
4.  **Save and Deploy:** Click **Save and continue**. This will add the new integration policy to any agents you have enrolled in Fleet. The Elastic Agent will automatically receive the updated configuration and begin shipping Zeek logs to Elasticsearch.

### Validation

After configuring the integration, you can validate that data is flowing into your Elastic cluster.

1.  **Generate Network Activity:** On a machine monitored by your Zeek sensor, generate some sample traffic to ensure logs are being created.
    ```bash
    # Generate some DNS and HTTP traffic
    curl https://www.elastic.co
    nslookup elastic.co
    ```
    On the Zeek sensor, verify that log files are being updated: `ls -lht /opt/zeek/logs/current/`.

2.  **Verify in Kibana Discover:**
    - In Kibana, navigate to the **Discover** app.
    - Select the `logs-zeek.*` data view.
    - Enter the following KQL query to filter for Zeek connection logs: `data_stream.dataset : "zeek.connection"`
    - You should see documents with fields like `source.ip`, `destination.port`, `zeek.connection.state`, and `network.bytes`.

3.  **Check Dashboards:**
    - Navigate to **Analytics** > **Dashboard**.
    - Search for and open the **[Logs Zeek] Overview** dashboard.
    - Confirm that visualizations for "Top Talkers," "Connection Count," and "Network Protocols" are populating with data.

## Troubleshooting

Here are some common issues and solutions.

**Common Configuration Issues**

- **Logs are still in TSV format:** The integration requires JSON log output. Ensure you executed `sudo zeekctl deploy` after adding `@load policy/tuning/json-logs` to `local.zeek`. Also, check `local.zeek` for any manual overrides like `redef LogAscii::use_json = F;` that might disable JSON output.
- **Permission Denied:** The Elastic Agent user (often `elastic-agent` or `root`) must have read permissions for the Zeek log directory and files. If the agent is running as a non-root user, you may need to adjust permissions. You can use Access Control Lists (ACLs) for this:
  ```bash
  sudo setfacl -R -m u:elastic-agent:rx /opt/zeek/logs/
  ```
- **Missing Log Files:** Zeek only creates specific log files (like `dns.log` or `http.log`) when it observes that type of traffic. If a log file is missing, generate the relevant network activity to trigger its creation.

**Ingestion Errors**

- **JSON Parsing Errors:** If you see `json.keys_under_root` errors or fields are not mapping correctly, check the `error.message` field in Kibana Discover. This can happen if custom Zeek scripts produce non-standard JSON that conflicts with the integration's schema.
- **Timestamp Mismatch:** If logs appear with incorrect timestamps, verify that the system clock and timezone are set correctly on the Zeek sensor host where the Elastic Agent is running.

**Vendor Resources**

- [Official Zeek Documentation Home](https://docs.zeek.org/en/master/index.html)
- [Zeek Quick Start Guide](https://docs.zeek.org/en/lts/quickstart.html)
- [Zeek JSON Logging Reference](https://docs.zeek.org/en/master/scripts/policy/tuning/json-logs.zeek.html)
- [Zeek Logging Framework Overview](https://docs.zeek.org/en/master/frameworks/logging.html)
- [Zeek Log Formats](https://docs.zeek.org/en/master/log-formats.html)
- [Zeek Control (zeekctl) Documentation](https://deepwiki.com/zeek/zeekctl/1-overview)

## Performance and scaling

The Zeek integration is designed to be efficient, but performance can be tuned for high-volume environments.

- **Fault Tolerance:** This integration uses a `logfile` input, which tracks the last read position for each log file in a registry. This ensures that the Elastic Agent can resume collecting logs from where it left off after a restart, preventing data loss.
- **Scaling Guidance:**
  - **Efficiently Monitor Files:** Use glob patterns (e.g., `/opt/zeek/logs/current/*.log`) to monitor all log files with a single input configuration.
  - **Control Resource Usage:** In environments with a very large number of log files, you can adjust the `harvester_limit` setting in the agent policy to control the number of concurrent file handles.
  - **Manage Log Rotation:** Use the `close_inactive` setting to configure how long the agent should keep a file handle open for an inactive (rotated) log file. This helps release system resources promptly.
  - **High-Volume Sensors:** For Zeek sensors that generate a very high volume of logs, ensure the host has sufficient I/O capacity, CPU, and memory for both Zeek and the Elastic Agent to run without contention.

## Reference

### Inputs used

This integration uses the `logfile` input (also known as `filestream`) to collect data from Zeek's log files.

### API usage (if applicable)

This integration does not use any external APIs for data collection.

### Data Stream Details

#### capture_loss
The `capture_loss` dataset collects the Zeek capture_loss.log file, which contains packet loss rate data.
{{fields "capture_loss"}}

#### connection
The `connection` dataset collects the Zeek conn.log file, which contains TCP/UDP/ICMP connection data.
{{fields "connection"}}

#### dce_rpc
The `dce_rpc` dataset collects the Zeek dce_rpc.log file, which contains Distributed Computing Environment/RPC data.
{{fields "dce_rpc"}}

#### dhcp
The `dhcp` dataset collects the Zeek dhcp.log file, which contains DHCP lease data.
{{fields "dhcp"}}

#### dnp3
The `dnp3` dataset collects the Zeek dnp3.log file which contains DNP3 requests and replies.
{{fields "dnp3"}}

#### dns
The `dns` dataset collects the Zeek dns.log file which contains DNS activity.
{{fields "dns"}}

#### dpd
The `dpd` dataset collects the Zeek dpd.log, which contains dynamic protocol detection failures.
{{fields "dpd"}}

#### files
The `files` dataset collects the Zeek files.log file, which contains file analysis results.
{{fields "files"}}

#### ftp
The `ftp` dataset collects the Zeek ftp.log file, which contains FTP activity.
{{fields "ftp"}}

#### http
The `http` dataset collects the Zeek http.log file, which contains HTTP requests and replies.
{{fields "http"}}

#### intel
The `intel` dataset collects the Zeek intel.log file, which contains intelligence data matches.
{{fields "intel"}}

#### irc
The `irc` dataset collects the Zeek irc.log file, which contains IRC commands and responses.
{{fields "irc"}}

#### kerberos
The `kerberos` dataset collects the Zeek kerberos.log file, which contains Kerberos data.
{{fields "kerberos"}}

#### known_certs
The `known_certs` dataset captures information about SSL/TLS certificates seen on the local network.
{{fields "known_certs"}}

#### known_hosts
The `known_hosts` dataset records a timestamp and an IP address when Zeek observes a new system on the local network.
{{fields "known_hosts"}}

#### known_services
The `known_services` dataset records a timestamp, IP, port number, protocol, and service when Zeek observes a system offering a new service.
{{fields "known_services"}}

#### modbus
The `modbus` dataset collects the Zeek modbus.log file, which contains Modbus commands and responses.
{{fields "modbus"}}

#### mysql
The `mysql` dataset collects the Zeek mysql.log file, which contains MySQL data.
{{fields "mysql"}}

#### notice
The `notice` dataset collects the Zeek notice.log file, which contains Zeek notices.
{{fields "notice"}}

#### ntlm
The `ntlm` dataset collects the Zeek ntlm.log file, which contains NT LAN Manager (NTLM) data.
{{fields "ntlm"}}

#### ntp
The `ntp` dataset collects the Zeek ntp.log file, which contains NTP data.
{{fields "ntp"}}

#### ocsp
The `ocsp` dataset collects the Zeek ocsp.log file, which contains Online Certificate Status Protocol (OCSP) data.
{{fields "ocsp"}}

#### pe
The `pe` dataset collects the Zeek pe.log file, which contains portable executable data.
{{fields "pe"}}

#### radius
The `radius` dataset collects the Zeek radius.log file, which contains RADIUS authentication attempts.
{{fields "radius"}}

#### rdp
The `rdp` dataset collects the Zeek rdp.log file, which contains RDP data.
{{fields "rdp"}}

#### rfb
The `rfb` dataset collects the Zeek rfb.log file, which contains Remote Framebuffer (RFB) data.
{{fields "rfb"}}

#### signature
The `signature` dataset collects the Zeek signature.log file, which contains Zeek signature matches.
{{fields "signature"}}

#### sip
The `sip` dataset collects the Zeek sip.log file, which contains SIP data.
{{fields "sip"}}

#### smb_cmd
The `smb_cmd` dataset collects the Zeek smb_cmd.log file, which contains SMB commands.
{{fields "smb_cmd"}}

#### smb_files
The `smb_files` dataset collects the Zeek smb_files.log file, which contains SMB file data.
{{fields "smb_files"}}

#### smb_mapping
The `smb_mapping` dataset collects the Zeek smb_mapping.log file, which contains SMB trees.
{{fields "smb_mapping"}}

#### smtp
The `smtp` dataset collects the Zeek smtp.log file, which contains SMTP transactions.
{{fields "smtp"}}

#### snmp
The `snmp` dataset collects the Zeek snmp.log file, which contains SNMP messages.
{{fields "snmp"}}

#### socks
The `socks` dataset collects the Zeek socks.log file, which contains SOCKS proxy requests.
{{fields "socks"}}

#### software
The `software` dataset collects details on applications operated by hosts on the local network.
{{fields "software"}}

#### ssh
The `ssh` dataset collects the Zeek ssh.log file, which contains SSH connection data.
{{fields "ssh"}}

#### ssl
The `ssl` dataset collects the Zeek ssl.log file, which contains SSL/TLS handshake info.
{{fields "ssl"}}

#### stats
The `stats` dataset collects the Zeek stats.log file, which contains memory, event, packet, and lag statistics.
{{fields "stats"}}

#### syslog
The `syslog` dataset collects the Zeek syslog.log file which contains syslog messages.
{{fields "syslog"}}

#### traceroute
The `traceroute` dataset collects the Zeek traceroute.log file, which contains traceroute detections.
{{fields "traceroute"}}

#### tunnel
The `tunnel` dataset collects the Zeek tunnel.log file, which contains tunneling protocol events.
{{fields "tunnel"}}

#### weird
The `weird` dataset collects the Zeek weird.log file, which contains unexpected network-level activity.
{{fields "weird"}}

#### x509
The `x509` dataset collects the Zeek x509.log file, which contains X.509 certificate info.
{{fields "x509"}}