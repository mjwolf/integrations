# Service Info

## Common use cases

The Blue Coat ProxySG (Symantec/Broadcom) integration enables organizations to ingest and analyze web traffic logs, providing deep visibility into network security and user activity. By collecting these logs, security teams can perform the following:
- **Security Threat Detection:** Monitor `cs-categories` and `x-virus-id` to identify users attempting to access malicious domains or downloading infected files.
- **Compliance Auditing:** Track user authentication events via `cs-username` and `cs-auth-groups` to ensure corporate policy adherence and meet regulatory requirements for internet usage.
- **Network Performance Monitoring:** Analyze `time-taken`, `duration`, and `dnslookup-time` to identify bottlenecks in the proxy infrastructure or slow external web services.
- **Traffic Analysis and Filtering:** Evaluate the effectiveness of web filtering policies by reviewing `sc-filter-result` and `s-action` to see which requests are being blocked or allowed.

## Data types collected

This integration can collect the following types of data:
- **Access Logs:** Detailed records of every HTTP, HTTPS, and FTP request processed by the proxy, typically formatted in W3C Extended Log File Format (ELFF).
- **Event Logs:** System-level information including service restarts, hardware health status, administrative logins, and configuration changes.
- **Syslog Data:** Standardized log messages sent over UDP or TCP, containing priority headers, timestamps, and the log payload.
- **Security Metadata:** Information regarding categorized URLs, risk scores, and threat levels associated with specific web requests.

## Compatibility

This integration is compatible with **Blue Coat ProxySG** and **Advanced Secure Gateway (ASG)** appliances. It is designed to work with appliances running **SGOS version 6.x and higher**. Compatibility depends on the ability to configure custom log formats and TCP-based log streaming within the Management Console.

## Scaling and Performance

- **Transport/Collection Considerations:** For high-volume environments, using TCP for syslog transport is recommended to ensure delivery reliability, though it introduces overhead on the ProxySG. If maximum appliance performance is the priority and occasional log loss is acceptable during peak bursts, UDP may be utilized.
- **Data Volume Management:** To reduce the load on both the ProxySG and the Elastic Stack, administrators should use the Visual Policy Manager (VPM) to log only necessary traffic. High-volume, low-risk traffic (such as internal health checks) should be excluded from access logging to minimize ingestion costs and processing latency.
- **Elastic Agent Scaling:** A single Elastic Agent can typically handle several thousand events per second (EPS) from Bluecoat. For large-scale deployments with multiple ProxySG clusters, consider deploying multiple Elastic Agents behind a load balancer (such as HAProxy or an F5 BigIP) to distribute the syslog traffic and provide high availability.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** Full administrative credentials for the ProxySG/ASG Management Console and the Visual Policy Manager (VPM).
- **Network Connectivity:** The ProxySG appliance must have network reachability to the Elastic Agent on the designated TCP port (default is often 514 or a custom high port).
- **Firewall Rules:** Ensure any intermediate firewalls permit TCP traffic from the ProxySG Management IP or Virtual IP to the Elastic Agent's listener IP.
- **Licensing:** An active license for ProxySG/ASG that includes standard access logging capabilities.
- **Knowledge Requirements:** You must identify the IP address of the Elastic Agent and decide on a dedicated TCP port for the log stream.

## Elastic prerequisites

- **Elastic Agent:** An Elastic Agent must be installed and enrolled in a policy via Fleet.
- **Connectivity:** The host running the Elastic Agent must be reachable from the ProxySG appliance over the configured TCP port. Ensure host-based firewalls (iptables, firewalld, or Windows Firewall) allow the incoming connection.
- **Integration Package:** The `bluecoat_proxysg` integration must be added to the Agent policy.

## Vendor set up steps

### For Custom TCP Syslog Collection:

1.  **Log in to the Blue Coat Management Console.** Navigate to the management IP of your appliance and log in with admin credentials.
2.  **Create a Custom Log Format:**
    *   Navigate to **Configuration > Access Logging > Formats**.
    *   Click **New** to create a new format.
    *   Enter `elastic-agent-kv` as the **Format Name**.
    *   In the format string text area, paste the following string (ensure no leading/trailing spaces):
        ```
        <111>1 $(date)T$(x-bluecoat-hour-utc):$(x-bluecoat-minute-utc):$(x-bluecoat-second-utc)$(s-computername) bluecoat - splunk_format - c-ip=$(c-ip) rs-Content-Type=$(quot)$(rs(Content-Type))$(quot) cs-auth-groups=$(cs-auth-groups) cs-bytes=$(cs-bytes) cs-categories=$(cs-categories) cs-host=$(cs-host) cs-ip=$(cs-ip) cs-method=$(cs-method) cs-uri-port=$(cs-uri-port) cs-uri-scheme=$(cs-uri-scheme) cs-User-Agent=$(quot)$(cs(User-Agent))$(quot) cs-username=$(cs-username) dnslookup-time=$(dnslookup-time) duration=$(duration) rs-status=$(rs-status) rs-version=$(rs-version) s-action=$(s-action) s-ip=$(s-ip) service.name=$(service.name) service.group=$(service.group) s-supplier-ip=$(s-supplier-ip) s-supplier-name=$(s-supplier-name) sc-bytes=$(sc-bytes) sc-filter-result=$(sc-filter-result) sc-status=$(sc-status) time-taken=$(time-taken) x-exception-id=$(x-exception-id) x-virus-id=$(x-virus-id) c-url=$(quot)$(url)$(quot) cs-Referer=$(quot)$(cs(Referer))$(quot)
        ```
    *   Click **OK** and perform a syntax check if prompted.
3.  **Configure the Log Upload Client:**
    *   Navigate to **Configuration > Access Logging > Logs**.
    *   Select the **Upload Client** tab.
    *   Choose your primary access log file from the list.
    *   Change the **Client type** to **Custom Client**.
    *   Click **Settings** and enter the **Host** (Elastic Agent IP) and the **Port** (e.g., 11514).
    *   Click **OK** and then **Apply**.
4.  **Assign the Format and Schedule:**
    *   Go to the **General Settings** tab under **Logs**.
    *   Set the **Log Format** to the `elastic-agent-kv` format created in Step 2.
    *   Set "Save the log file as" to **text file**.
    *   In the **Upload Schedule** tab, set the upload schedule to **continuously**.
    *   Click **Apply**.
5.  **Enable Logging in Visual Policy Manager (VPM):**
    *   Go to **Configuration > Policy > Visual Policy Manager > Launch**.
    *   Create or edit a **Web Access Layer**.
    *   Add a rule, right-click the **Action** column, and select **Set**.
    *   Create a **New** "Modify Access Logging" object.
    *   Select the log file configured in Step 3 and click **OK**.
    *   Click **Install Policy** to apply changes to the appliance.

## Kibana set up steps

1.  **Navigate to Integrations:** In Kibana, go to **Management > Integrations** and search for **Blue Coat ProxySG**.
2.  **Add Integration:** Click **Add Blue Coat ProxySG**.
3.  **Configure Integration Settings:**
    *   Select **Log** as the input type (usually listed under Syslog/TCP).
    *   Set the **Listen Host** to `0.0.0.0` or the specific IP of the Elastic Agent interface.
    *   Set the **Listen Port** to match the port configured on the ProxySG appliance (e.g., `11514`).
    *   Under **Advanced options**, ensure the protocol is set to **tcp**.
4.  **Select Agent Policy:** Choose the policy that is currently applied to your Elastic Agent.
5.  **Save and Deploy:** Click **Save and continue**, then **Add Agent to Policy** to deploy the configuration changes.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from Blue Coat ProxySG to the Elastic Stack.

### 1. Trigger Data Flow on Blue Coat ProxySG:
- **Generate web traffic:** From a workstation using the ProxySG as its gateway, browse to several different websites (e.g., a news site, a search engine) to generate access logs.
- **Trigger a Block Event:** Attempt to access a URL that falls into a category blocked by your corporate policy to generate a "denied" log entry.
- **Verify Transmission:** On the ProxySG Management Console, go to **Reports > Access Logging**, select your log, and click **Show Log Tail** to confirm the appliance is generating the KV-formatted logs.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the following KQL filter: `data_stream.dataset : "bluecoat_proxysg.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `bluecoat_proxysg.log`)
   - `source.ip` (should show the client's IP address)
   - `url.original` (should contain the requested URL)
   - `bluecoat_proxysg.sc_filter_result` (should show "OBSERVED", "DENIED", etc.)
   - `bluecoat_proxysg.cs_categories` (should show the web category like "Search Engines")
5. Navigate to **Analytics > Dashboards** and search for "Blue Coat ProxySG" to view the pre-built traffic overview dashboards.

# Troubleshooting

## Common Configuration Issues

- **Policy Not Installed**: Changes made in the Visual Policy Manager do not take effect until the **Install Policy** button is clicked. If logs are missing, verify that the "Modify Access Logging" action is active in the VPM.
- **Incorrect Log Format**: If logs are appearing in Kibana but fields are not being parsed, ensure the "Log Format" assigned to the log object in ProxySG matches the parser expected by the integration (typically the default 'main' or 'bcreportermain_v1' format).
- **Buffer Delays**: If logs appear in large chunks or with significant delay, check the **Send partial buffer after** setting. If set too high (e.g., 60 seconds), data will not stream in real-time.
- **TCP Connection Reset**: If the ProxySG logs show "Connection Reset by Peer," ensure the Elastic Agent is running and that no local firewall (like `iptables` or Windows Firewall) is blocking the incoming TCP port on the Agent host.

## Ingestion Errors

- **Parsing Failures**: Check the `error.message` field in Discover. If it contains "provided data does not match the expected format," inspect the `message` field to see if the ProxySG is sending headers or non-standard characters that interfere with the Grok/Dissect patterns.
- **Timestamp Mismatches**: If logs appear with the wrong time, verify the NTP settings on both the ProxySG appliance and the Elastic Agent host. Mismatched timezones can cause logs to appear in the "future" or "past" in Discover.
- **Field Truncation**: For extremely long URLs, the ProxySG may truncate logs. Adjust the maximum log line length in the ProxySG global settings if necessary to prevent truncated JSON or Syslog messages.

## Vendor Resources

- [Enable Syslog on the ProxySG/ASG - Broadcom](https://knowledge.broadcom.com/external/article/166081/enable-syslog-on-the-proxysgasg.html)
- [Sending Access Logs to a Syslog server - Broadcom](https://knowledge.broadcom.com/external/article/166529/sending-access-logs-to-a-syslog-server.html)

## Documentation sites

- [Sending Access Logs to a Syslog server - Broadcom](https://knowledge.broadcom.com/external/article/166529/sending-access-logs-to-a-syslog-server.html)
- [Configure logging in your Blue Coat ProxySG appliance for the Splunk Add-on for Symantec Blue Coat ProxySG](https://docs.splunk.com/Documentation/AddOns/released/BlueCoatProxySG/Setup)
- Refer to the official vendor website for additional resources.
