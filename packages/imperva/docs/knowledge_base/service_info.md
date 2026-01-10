# Service Info

## Common use cases

*   **Real-time Threat Monitoring:** Detect and respond to web-based attacks such as SQL injection, Cross-Site Scripting (XSS), and automated bot activity identified by the Cloud WAF.
*   **Compliance and Auditing:** Maintain long-term storage of access logs and security events to meet regulatory requirements like PCI DSS, SOC2, and HIPAA.
*   **Traffic and Performance Analysis:** Analyze web traffic patterns and geographic distribution to optimize application delivery and identify anomalous traffic spikes.

## Data types collected

*   **Security Logs:** Detailed events regarding blocked or flagged suspicious activity, including attack type, client IP, and mitigation action.
*   **Access Logs:** Comprehensive records of all HTTP/HTTPS requests processed by the Cloud WAF, including status codes, request paths, and user agent strings.

## Compatibility

*   **Imperva Cloud WAF (formerly Incapsula):** Compatible with current SaaS offerings managed via the Imperva Cloud Security Console.
*   **Log Formats:** CEF (Common Event Format - recommended), LEEF (Log Event Extended Format), and W3C.
*   **Elastic Agent:** Compatible with any supported Elastic Agent version running the Custom Logs or Integrations input.

## Scaling and Performance

*   **Log Delivery:** Imperva pushes logs to the destination in near real-time. Throughput depends on the configured log level and site traffic volume.
*   **Data Optimization:** Built-in `.gz` compression is supported to reduce bandwidth and storage requirements during the transfer process.
*   **High Availability:** Performance is maintained through Imperva’s globally distributed network, ensuring logs are captured even during high-volume DDoS events.

# Set Up Instructions

## Vendor prerequisites

*   An active Imperva `my.imperva.com` account with Administrative permissions.
*   A subscription to Imperva Cloud WAF with log integration features enabled.
*   A target destination for log storage, such as an SFTP server, Amazon S3 bucket, or Azure Storage, accessible by both Imperva and the Elastic Agent.

## Elastic prerequisites

*   Elastic Agent installed and enrolled in a policy via Fleet.
*   Appropriate network permissions to allow the Elastic Agent to read files from the local directory or network share where Imperva logs are deposited.

## Vendor set up steps

1.  **Log In to Imperva:** Access your account at `my.imperva.com`.
2.  **Navigate to Log Configuration:** Go to **Account > Account Management** from the top menu, then select **SIEM Logs > Log Configuration** on the sidebar.
3.  **Create a New Connection:** Click **Add connection** in the Connections table. Select your delivery method (e.g., **SFTP**) and enter the required server credentials and directory path.
4.  **Configure the Log Type:** Click **Add log type** and select the **Cloud WAF** service.
5.  **Define Log Settings:** 
    *   **Log types:** Choose **Security Logs** or **Security Logs and Access Logs**.
    *   **Format:** Select **CEF** for optimal parsing with Elastic.
    *   **Compress logs:** Ensure the checkbox is selected to receive `.gz` files.
6.  **Enable the Configuration:** Toggle the **State** to **Enabled** to start the log push process.

## Kibana set up steps

1.  Log in to Kibana and navigate to **Management > Integrations**.
2.  Search for and select the **Imperva** integration (or use the **Custom Logs** integration if the specific package is unavailable).
3.  Click **Add Imperva**.
4.  Under **Log file path**, enter the absolute path to the directory where your SFTP server saves the Imperva log files (e.g., `/var/log/imperva/*.log.gz`).
5.  If using the Custom Logs integration, ensure the **Processors** section includes a CEF codec or appropriate grok patterns if using W3C format.
6.  Save the integration to the policy assigned to your Elastic Agent.

# Validation Steps

1.  **Verify File Delivery:** Check the SFTP/storage directory to ensure new `.log` or `.gz` files are appearing periodically.
2.  **Check Agent Status:** In Kibana, go to **Fleet > Agents** and verify the agent status is "Healthy."
3.  **Inspect Data in Discover:** Navigate to **Analytics > Discover** and filter for `event.dataset: "imperva.waf"` or `logs-*`. Ensure fields like `source.ip` and `imperva.waf.security_event` are populated.
4.  **Trigger Test Event:** Attempt a known "safe" attack pattern (like appending `?id=' OR 1=1` to a site URL) and verify the security log appears in the dashboard.

# Troubleshooting

## Common Configuration Issues

*   **Logs Not Appearing on Server:** Verify the SFTP credentials and directory permissions in the Imperva Console. Ensure the Imperva IP ranges are allow-listed in your firewall.
*   **Agent Not Reading Files:** Check the file path pattern in the Elastic Agent policy. Ensure the user running the Elastic Agent service has read/write permissions to the log directory to allow for file harvesting and registry tracking.

## Ingestion Errors

*   **Parsing Failures:** If using W3C or LEEF formats, the default Elastic parser may fail. Check the `error.message` field in Kibana for "Provided grok patterns did not match." Switch to the recommended **CEF** format in Imperva.
*   **Time Synchronization:** Ensure the SFTP server and Elastic Agent use NTP. Large clock skews between Imperva's timestamp and the ingestion timestamp can cause logs to appear "missing" in the default time range.

## API Authentication Errors

*   **Connection Denied:** Imperva may fail to connect to your log destination if the certificate for your SFTP/S3 endpoint is invalid or expired.
*   **Permission Denied:** Ensure the Imperva "Log Integration" user has specific "Write" and "Create" permissions in the target directory.

## Vendor Resources

*   [Imperva Log Integration Troubleshooting Guide](https://docs-cybersec.thalesgroup.com/bundle/cloud-application-security/page/settings/log-integration.htm)
*   [Imperva Community and Support Portal](https://community.imperva.com/)

# Documentation sites

*   [Imperva Cloud WAF Product Overview](https://www.imperva.com/products/web-application-firewall-waf/)
*   [Cloud WAF Log Integration Documentation](https://docs-cybersec.thalesgroup.com/bundle/cloud-application-security/page/settings/log-integration.htm)
*   [Imperva API Reference](https://docs.imperva.com/bundle/cloud-application-security/page/api/cloud-waf-api.htm)