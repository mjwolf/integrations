# Service Info

The ModSecurity integration allows you to ingest, parse, and visualize audit logs from the ModSecurity Web Application Firewall (WAF). This integration is essential for maintaining visibility into the security posture of your web applications and identifying malicious activity in real-time.

## Common use cases

The ModSecurity integration allows you to ingest and analyze audit logs from your Web Application Firewall (WAF) to maintain a secure and observable web environment.
- **Threat Detection and Analysis:** Monitor web traffic for common attacks such as SQL Injection (SQLi), Cross-Site Scripting (XSS), and Local File Inclusion (LFI) by analyzing ModSecurity's triggered rules.
- **Regulatory Compliance Auditing:** Maintain a persistent record of all transactions that trigger security rules to meet compliance requirements such as PCI-DSS, HIPAA, or GDPR.
- **WAF Rule Tuning:** Identify false positives by examining the specific parts of the HTTP transaction that triggered a rule, allowing security teams to refine rule sets for better accuracy.
- **Incident Response:** Provide security analysts with detailed forensic data regarding malicious requests, including source IP addresses, request headers, and the specific ModSecurity rules that were violated.

## Data types collected

This integration collects the following data types to provide a comprehensive view of WAF activity:

- **Modsecurity Audit Log (auditlog):** Collect modsecurity audit logs. This data stream includes detailed transaction metadata such as source/destination IP addresses, HTTP request headers, response codes, matching rule IDs, and specific actions taken by the WAF (e.g., "Allow", "Deny").
- **Structured JSON Logs:** All data is collected in JSON format to ensure high-fidelity structured parsing and consistent field mapping within the Elastic Common Schema (ECS).
- **Transaction Metadata:** Capture detailed forensic information about web requests that trigger security rules, enabling deep analysis of attack patterns.

Log files are typically monitored at paths such as `/var/log/modsec-audit*` or other configured locations defined in the integration settings.

## Compatibility

- **Elastic Stack Requirements:** This integration requires Kibana version **8.11.0** or higher.
- **ModSecurity version:** This integration is compatible with **ModSecurity v3** (also known as LibModSecurity). It has been specifically tested and verified with:
    - **ModSecurity v3** utilizing the **Nginx connector**.
    - **ModSecurity v3** utilizing the **Apache connector**.
    - The logs must be configured in **JSON format** and use the **Serial** audit log type to ensure compatibility with the Elastic Agent's `logfile` input.

## Scaling and Performance

To ensure optimal performance in high-volume environments, consider the following:
- **Transport/Collection Considerations:** This integration utilizes the `logfile` input type to ingest data. For optimal performance, the `SecAuditLogType Serial` directive must be used on the vendor side. This ensures that logs are written to a single file sequentially, allowing the Elastic Agent to efficiently tail the file and manage file pointers without the performance penalty associated with `Concurrent` logging modes which create thousands of small files.
- **Data Volume Management:** ModSecurity can generate large volumes of data depending on traffic. It is recommended to set `SecAuditEngine RelevantOnly` to only log transactions that trigger rules. Additionally, to reduce the ingest load and storage requirements, avoid including part `K` in the `SecAuditLogParts` configuration, as this part contains the full rule list which can lead to extremely large events.
- **Elastic Agent Scaling:** For high-throughput web environments, deploy an Elastic Agent on each individual web server host to distribute the processing load. If log rotation is frequent due to high volume, ensure the Agent has sufficient CPU and memory resources to keep pace with the ingestion before logs are rotated out of scope.

# Set Up Instructions

## Vendor prerequisites

1. **Administrative Access:** You must have root or sudo-level permissions on the host server to modify web server configurations and ModSecurity settings.
2. **YAJL Library:** ModSecurity must be compiled with the `yajl` library to support JSON output. On many systems, this requires the `yajl-dev` package to be present during the build process.
3. **Write Permissions:** The web server process (e.g., `www-data` or `apache`) must have write permissions to the directory where the audit logs are stored.
4. **Elastic Agent Permissions:** The Elastic Agent service user must have read permissions for the ModSecurity JSON log file (e.g., `/var/log/modsec-audit*`).
5. **Web Server Knowledge:** Familiarity with Nginx or Apache configuration file structures and service management commands.

## Elastic prerequisites

1. **Elastic Stack Version:** Ensure your Elastic Stack is at version **8.11.0** or higher.
2. **Elastic Agent Enrollment:** An Elastic Agent must be installed on the host and enrolled in a policy managed by Fleet.
3. **Network Connectivity:** The host must have outbound network connectivity to the Elastic Stack on the required ports (typically 443 or 9200/5601).
4. **Read Permissions:** The Elastic Agent service user must have read permissions for the ModSecurity JSON audit log files (default path: `/var/log/modsec-audit*`).
5. **Kibana Access:** A user with permissions to add integrations and edit Agent policies in Kibana.

## Vendor set up steps

### For Logfile Collection (JSON Format):

1. **Locate Configuration:** Find your primary ModSecurity configuration file, typically named `modsecurity.conf`. This is usually located in `/etc/modsecurity/` or defined within your web server's site-specific configuration.
2. **Enable Audit Engine:** Locate the `SecAuditEngine` directive and ensure it is set to `On` to capture security events.
   ```ini
   SecAuditEngine On
   ```
3. **Set Log Format to JSON:** Configure the output format by adding the following line. Note that if this directive is missing or set to `Native`, the integration will fail to parse the logs.
   ```ini
   SecAuditLogFormat JSON
   ```
4. **Configure Logging Mechanism:** Set the log type to `Serial` so that all events are written to a single file for the Elastic Agent to monitor.
   ```ini
   SecAuditLogType Serial
   ```
5. **Define Log Path:** Specify the destination for the audit logs. This path must match the path configured later in Kibana.
   ```ini
   SecAuditLog /var/log/modsec_audit.json
   ```
6. **Configure Audit Parts:** Define which transaction details are included. **CRITICAL:** Use exactly `ABDEFHIJZ`. Do not include the `K` part, as long rule lists can cause parsing errors in the ingest pipeline.
   ```ini
   SecAuditLogParts ABDEFHIJZ
   ```
7. **Apply Changes:** Restart your web server to apply the new logging configuration.
   - For Nginx: `sudo systemctl restart nginx`
   - For Apache: `sudo systemctl restart apache2` or `sudo systemctl restart httpd`
8. **Verify Output:** Perform a basic test (e.g., a curl request with a suspicious header) to ensure the `/var/log/modsec_audit.json` file is being populated with JSON-formatted data.

## Kibana set up steps

### Collecting modsecurity audit logs
1. In Kibana, navigate to **Management > Integrations** and search for **ModSecurity**.
2. Click **Add ModSecurity**.
3. Follow the prompts to add the integration to an existing Elastic Agent policy or create a new one.
4. Under the **ModSecurity audit logs** section, configure the following variables:
   - **Paths** (`paths`): Specify the list of paths to the ModSecurity audit log files. Default: `['/var/log/modsec-audit*']`.
   - **Preserve original event** (`preserve_original_event`): Toggle this to preserve a raw copy of the original event in the `event.original` field. Default: `False`.
   - **Timezone Offset** (`tz_offset`): By default, datetimes in the logs will be interpreted as relative to the timezone configured in the host where the agent is running. If ingesting logs from a host on a different timezone, use this field to set the timezone offset. Acceptable formats are canonical IDs (e.g. `Europe/Amsterdam`), abbreviated (e.g. `EST`), or an `HH:mm` differential (e.g. `-05:00`) from UTC. Default: `local`.
5. Click **Save and continue** to deploy the integration. The Elastic Agent will automatically update its configuration and begin tailing the specified files.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly.

### 1. Trigger Data Flow on ModSecurity:
- **Generate Security Event:** Attempt to access a sensitive file path via your web server to trigger a standard ModSecurity rule: `curl "http://localhost/index.php?file=/etc/passwd"`
- **Trigger XSS Alert:** Send a request containing a script tag: `curl "http://localhost/?q=<script>alert(1)</script>"`
- **Check Local Log:** Run `tail -n 5 /var/log/modsec_audit.json` to confirm that the transaction was logged locally in JSON format.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "modsecurity.auditlog"`
4. Verify logs appear. Expand a log entry and confirm these fields:
   - `event.dataset` (should be `modsecurity.auditlog`)
   - `source.ip` (the client IP address)
   - `http.request.method` (e.g., `GET`)
   - `event.outcome` (e.g., `deny` or `allow`)
   - `message` (the raw log payload)
5. Navigate to **Analytics > Dashboards** and search for "ModSecurity" to verify visualizations are populating.

# Troubleshooting

## Common Configuration Issues

- **Incorrect Log Format**: If logs are appearing in a flat-text format rather than JSON, the Elastic Agent will fail to parse them. Ensure `SecAuditLogFormat JSON` is explicitly set in `modsecurity.conf`.
- **Missing Audit Log Parts**: If critical fields like request headers are missing in Kibana, verify that `SecAuditLogParts` includes the necessary letters (specifically `B` for request headers and `F` for response headers).
- **Log Path Mismatch**: If no data appears in Kibana, double-check that the **Paths** variable in the Kibana integration configuration matches the actual file path defined in the `SecAuditLog` directive on the server.
- **Permissions Denied**: The Elastic Agent may not have permission to read the log file. Ensure the agent user has read access to the directory and the file: `ls -l /var/log/modsec_audit.json`.

## Ingestion Errors

- **Parsing Failures (Excessive Length)**: If you see errors related to message size or truncated JSON, verify that the `K` part has been removed from `SecAuditLogParts`. This part can generate extremely large strings that exceed the buffer limits of the ingest pipeline.
- **Timestamp Mismatch**: If logs appear with the wrong timestamp in Discover, adjust the **Timezone Offset** setting in the Kibana integration configuration to match the host server's local time settings.
- **Identifying Failures**: Use Kibana to search for logs with the field `error.message` populated. This will often contain the specific reason why the ingest pipeline failed to process a ModSecurity event.

## Vendor Resources

- [ModSecurity GitHub Repository](https://github.com/owasp-modsecurity/ModSecurity)
- [ModSecurity Reference Manual (v3.x)](https://github.com/owasp-modsecurity/ModSecurity/wiki/Reference-Manual-(v3.x))

## Documentation sites

- [ModSecurity Reference Manual (v3.x)](https://github.com/owasp-modsecurity/ModSecurity/wiki/Reference-Manual-(v3.x))
- [Elastic ModSecurity Integration Reference](https://www.elastic.co/docs/reference/integrations/modsecurity)
