# Service Info

ModSecurity is a powerful open-source Web Application Firewall (WAF) engine that provides real-time HTTP traffic monitoring, logging, and access control for web applications. This integration allows users to ingest ModSecurity audit logs into the Elastic Stack for deep security analysis and threat hunting.

## Common use cases

- **Identifying Web Attacks:** Detect and analyze common web-based attacks such as SQL Injection (SQLi), Cross-Site Scripting (XSS), and Local File Inclusion (LFI) by monitoring matched rule IDs and transaction details.
- **Compliance and Auditing:** Maintain a comprehensive audit trail of all HTTP transactions, including request and response headers, to meet regulatory requirements like PCI-DSS or GDPR.
- **WAF Rule Tuning:** Analyze false positives by reviewing the specific log parts that triggered a rule, allowing security teams to refine ModSecurity directives and reduce noise.
- **Security Posture Monitoring:** Correlate ModSecurity events with other web server logs (Apache or Nginx) in Kibana to gain a holistic view of the attack surface and identify persistent threat actors.

## Data types collected

This integration can collect the following types of data:
- **ModSecurity Audit Logs:** Comprehensive transaction details including request/response headers and bodies, typically formatted in a multi-part text format.
- **Web Server Error Logs:** Brief alert messages generated when a rule is matched, often containing the source IP, rule ID, and descriptive message.
- **Data Formats:** Syslog (for real-time alerts) and Plain Text/Native ModSecurity Audit format (for detailed logs).
- **Log File Paths:** Standard paths include `/var/log/modsec_audit.log` for dedicated audit logs and `/var/log/apache2/error.log` or `/var/log/httpd/error_log` for web server integration.

## Compatibility

This integration is compatible with **ModSecurity** versions 2.9.x and 3.x (libmodsecurity). It supports deployments on **Apache HTTP Server** and **Nginx**. The Elastic Agent must be running version 7.14.0 or higher to support the latest integration features.

## Scaling and Performance

- **Transport/Collection Considerations:** This integration utilizes logfile collection. Because ModSecurity logs can grow rapidly under heavy traffic, it is critical to use the `Serial` log type for sequential writing. Ensure that system-level log rotation (e.g., `logrotate`) is configured to prevent the log file from growing indefinitely and consuming available disk space, which could lead to service degradation.
- **Data Volume Management:** To manage high volumes of data, users should configure the `SecAuditEngine` to `RelevantOnly`. This ensures that only transactions that trigger a rule or result in an error are logged, significantly reducing noise and storage costs compared to logging every single HTTP request. Additionally, adjusting `SecAuditLogParts` allows you to exclude high-bandwidth parts like the response body if they are not required for analysis.
- **Elastic Agent Scaling:** In high-throughput environments with thousands of requests per second, a single Elastic Agent should be deployed on each web server node to handle local log harvesting. Ensure the host system has sufficient CPU and memory to handle both the web server processes and the Agent's file-reading operations. For massive clusters, utilize Fleet to manage Agent policies centrally and ensure consistent filtering across all nodes.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** Root or sudo-level permissions on the web server (Apache or Nginx) are required to modify configuration files and restart services.
- **ModSecurity Installation:** The ModSecurity module must be installed and enabled within the web server environment.
- **Network Connectivity:** If using Syslog, ensure that the firewall allows traffic between the web server and the Elastic Agent on the configured port (e.g., UDP/TCP 514 or 1514).
- **Permissions:** The user account running the Elastic Agent must have read access to the ModSecurity log files (e.g., `/var/log/modsec_audit.log`).
- **Disk Space:** Ensure adequate disk space is available for audit logs, as detailed ModSecurity logs can grow rapidly under heavy traffic.

## Elastic prerequisites

- **Elastic Agent:** An active Elastic Agent must be installed and enrolled in Fleet on a host reachable by the ModSecurity server.
- **Integration Policy:** A ModSecurity integration policy must be created and assigned to the Agent's policy.
- **Network Access:** Ensure that firewall rules permit inbound traffic on the designated Syslog port and protocol at the Elastic Agent's IP address.

## Vendor set up steps

### For Logfile Collection:

1.  **Locate the Configuration File:** Identify your primary ModSecurity configuration file. On most systems, this is located at `/etc/modsecurity/modsecurity.conf` or included within the web server's site-specific configuration.
2.  **Enable the Audit Engine:** Open the configuration file in a text editor and ensure the `SecAuditEngine` directive is active.
    ```apache
    SecAuditEngine RelevantOnly
    ```
    *Note: Use `RelevantOnly` to log only suspicious events, or `On` for full transaction auditing.*
3.  **Configure the Log Type:** Set the audit log type to `Serial` to ensure all transaction parts are written to a single file that the Elastic Agent can follow.
    ```apache
    SecAuditLogType Serial
    ```
4.  **Define the Log Destination:** Specify the absolute path where the log file will be created. This path must be provided later in the Kibana configuration.
    ```apache
    SecAuditLog /var/log/modsec_audit.log
    ```
5.  **Select Log Parts:** Configure which components of the HTTP transaction should be included in the audit log. The following string is a standard recommendation for security analysis:
    ```apache
    SecAuditLogParts ABIJDEFHZ
    ```
6.  **Verify Permissions:** Create the log file manually if it does not exist and set the correct ownership:
    ```bash
    sudo touch /var/log/modsec_audit.log
    sudo chown www-data:www-data /var/log/modsec_audit.log
    sudo chmod 640 /var/log/modsec_audit.log
    ```
7.  **Apply Changes:** Restart or reload your web server to apply the new logging directives.
    *   For Apache: `sudo systemctl reload apache2`
    *   For Nginx: `sudo systemctl reload nginx`

## Kibana set up steps

### For Logfile Collection (Recommended):
1.  Navigate to **Management > Integrations** in Kibana and search for **ModSecurity**.
2.  Click **Add ModSecurity** to start the configuration wizard.
3.  Under the **Log file** input settings, set the **Paths** field to the location of your audit log: `/var/log/modsec_audit.log`.
4.  Ensure the **Dataset name** is set to `modsecurity.log`.
5.  Scroll down to **Advanced options** if you need to configure custom tags or processors for the incoming data.
6.  Click **Save and continue**, then select the appropriate **Agent Policy** to deploy the configuration.

### For Syslog Collection:
1.  Navigate to **Management > Integrations** and select **ModSecurity**.
2.  Select the **Syslog** input type during the configuration.
3.  Set the **Syslog Host** to `0.0.0.0` to listen on all interfaces or a specific IP address.
4.  Set the **Syslog Port** to the port you intend to send logs to (e.g., `1514`).
5.  Ensure the **Protocol** (TCP or UDP) matches the configuration used in the vendor setup steps.
6.  Click **Save and continue** and deploy the policy to your Elastic Agent.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from ModSecurity to the Elastic Stack.

### 1. Trigger Data Flow on ModSecurity:
- **Generate Security Alert:** Use `curl` from a remote machine to attempt a basic directory traversal attack against your protected web server: `curl "http://<your-server-ip>/?file=/etc/passwd"`.
- **Trigger Rule Match:** Attempt to inject a simple script tag into a URL parameter: `curl "http://<your-server-ip>/?search=<script>alert(1)</script>"`.
- **Restart Service Event:** Perform a graceful restart of the Apache service to generate initial startup logs: `sudo systemctl reload apache2`.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the following KQL filter: `data_stream.dataset : "modsecurity.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `modsecurity.log`)
   - `source.ip` (should show the IP of the machine that ran the curl commands)
   - `event.severity` (should reflect the severity level of the triggered rule)
   - `modsecurity.rule.id` (confirming the specific WAF rule that was triggered)
   - `message` (containing the raw ModSecurity alert text)
5. Navigate to **Analytics > Dashboards** and search for "ModSecurity" to view pre-built visualizations.

# Troubleshooting

## Common Configuration Issues

- **Incompatible Audit Log Type**: If logs are not appearing or appear as fragmented entries, ensure `SecAuditLogType` is set to `Serial`. The `Concurrent` log type creates many small files which are more difficult for the agent to track without specific glob pattern configuration.
- **Permission Denied for Elastic Agent**: The Elastic Agent typically runs as a specific user (e.g., `root` or `elastic-agent`). If the log file `/var/log/modsec_audit.log` has permissions like `600` owned by `www-data`, the Agent will be unable to read it. Use `chmod 644` or `640` with appropriate group ownership to resolve this.
- **Web Server Reload Required**: Changes to `modsecurity.conf` do not take effect immediately. If logs are not being written to the new path, ensure the web server service has been reloaded (`systemctl reload nginx` or `apache2`).
- **Incorrect Path Globbing**: If using wildcards in the Kibana "Paths" field (e.g., `/var/log/modsec/*.log`), ensure the directory structure exists. If the directory is missing, the Agent may fail to initialize the file watcher.

## Ingestion Errors

- **Parsing Failures**: If logs appear in Kibana but contain a `log.flags: dissection_failure` or `error.message` field, the log format might deviate from the expected ModSecurity Serial format. Check if custom headers or non-standard characters are being injected into the audit logs.
- **Multi-line Message Issues**: ModSecurity Serial logs are multi-line by nature. If the integration is not correctly identifying the start and end of a transaction (delimited by the `---XXXXXX---Z--` footer), individual transactions may be split into multiple Elastic documents. Ensure the integration's built-in multi-line settings are active.
- **Incomplete Log Parts**: If specific fields like `source.ip` or `http.request.headers` are missing, verify that the `SecAuditLogParts` directive in your vendor config includes the relevant sections (A for headers, B for request, etc.).

## Vendor Resources

- [ModSecurity Frequently Asked Questions (FAQ) - GitHub](https://github.com/owasp-modsecurity/ModSecurity/wiki/ModSecurity-Frequently-Asked-Questions-(FAQ))
- [Storage and Logging | microsoft/ModSecurity | DeepWiki](https://deepwiki.com/microsoft/ModSecurity/7-storage-and-logging)

## Documentation sites

- [ModSecurity Frequently Asked Questions (FAQ) - GitHub](https://github.com/owasp-modsecurity/ModSecurity/wiki/ModSecurity-Frequently-Asked-Questions-(FAQ))
- [Storage and Logging | microsoft/ModSecurity | DeepWiki](https://deepwiki.com/microsoft/ModSecurity/7-storage-and-logging)
