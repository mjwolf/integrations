# Service Info

The Citrix Web App Firewall (WAF) integration is designed to provide comprehensive visibility into the security posture of your web applications by ingesting logs from Citrix ADC and NetScaler devices. 

## Common use cases

The Citrix Web App Firewall (WAF) integration provides comprehensive visibility into security events and traffic filtered by your Citrix ADC or NetScaler appliances. By ingesting these logs into the Elastic Stack, security teams can achieve the following:

- **Threat Detection and Prevention:** Monitor incoming web requests for evidence of malicious activity. This includes identifying and blocking attacks such as **SQL injection**, **cross-site scripting (XSS)**, **buffer overflows**, and unauthorized modifications to sensitive customer data.
- **Compliance and Auditing:** Maintain a historical record of all blocked and allowed web traffic to meet strict regulatory requirements like **PCI-DSS**, **HIPAA**, or **GDPR**. These frameworks require detailed logging of access to sensitive business information and evidence of active security monitoring.
- **Security Incident Forensics:** Investigate security breaches by analyzing detailed **CEF-formatted logs**. Analysts can use fields like `source.ip`, `url.path`, and `event.action` to understand the source, nature, and outcome of specific attacks against web applications.
- **WAF Policy Optimization:** Review log data to identify false positives where legitimate traffic might be getting blocked. This allows administrators to refine application firewall signatures and security profiles, ensuring a high security posture without impacting user experience.

## Data types collected

This integration collects the following data streams:

- **Citrix WAF logs (UDP):** Collect Citrix WAF logs (via Syslog). This stream is designed for high-performance ingestion using the `udp` input type to collect security events formatted in CEF.
- **Citrix WAF logs (TCP):** Collect Citrix WAF logs (via Syslog). This stream utilizes the `tcp` input type to ensure reliable, connection-oriented delivery of security events from the Citrix appliance.
- **Citrix WAF logs (logfile):** Collect Citrix WAF logs. This stream uses the `logfile` input type to ingest events directly from a file system, suitable for local Agent deployments or centralized log servers.

The integration is optimized for Common Event Format (CEF) and maps vendor-specific details into the `citrix` field group and Elastic Common Schema (ECS) fields.

## Compatibility

- **Kibana Version:** This integration requires Kibana version **^8.11.0** or **^9.0.0**.
- **Citrix Versions:** Tested against and compatible with **Citrix ADC 13.1** and **NetScaler 10.0**.
- **General Compatibility:** Compatible with any Citrix/NetScaler version that supports **CEF (Common Event Format) logging** and external syslog forwarding.

## Scaling and Performance

To ensure optimal performance in high-volume environments, consider the following:

- **Transport/Collection Considerations:** When configuring the integration via network inputs, users must choose between **UDP** and **TCP**. UDP offers lower latency and less overhead on the Citrix appliance but does not guarantee delivery. **TCP** is recommended for environments where log integrity is critical, as it provides a reliable transport mechanism to ensure no security events are dropped during network congestion.
- **Data Volume Management:** To manage high volumes of log data, it is recommended to use specific expressions in the Citrix Audit Policy instead of a catch-all `true` expression. Configure the appliance to forward only necessary events (e.g., security violations and critical system errors) to avoid overwhelming the ingest pipeline with informational background noise.
- **Elastic Agent Scaling:** For high-throughput environments processing thousands of events per second, deploy multiple Elastic Agents behind a network load balancer. This distributes the Syslog ingestion load and provides high availability. Ensure the host machines have sufficient CPU and memory to handle the parsing of CEF-formatted messages.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have administrative credentials for the Citrix ADC or NetScaler management GUI or CLI.
- **CEF Logging License:** Ensure that your Citrix license supports Web Application Firewall features and that CEF logging can be enabled.
- **Network Connectivity:** The Citrix appliance must be able to reach the Elastic Agent host over the configured Syslog port (default **9001**). Ensure firewall rules permit traffic on this port.
- **Target IP Address:** You must know the IP address or hostname of the machine where the Elastic Agent is installed.
- **Syslog Facility Knowledge:** Identify an unused syslog facility (e.g., **LOCAL2**) on your appliance to prevent log collisions with other system services.

## Elastic prerequisites

- **Kibana Version:** Ensure your Elastic Stack is running version **8.11.0** or newer.
- **Elastic Agent:** An active Elastic Agent must be installed and enrolled in Fleet.
- **Network Access:** The host running the Elastic Agent must be configured to allow inbound traffic on the ports specified for TCP or UDP collection (default `9001`).
- **Integration Installation:** The Citrix WAF integration must be added to the Agent policy through the Kibana Integrations UI.

## Vendor set up steps

### For Syslog Collection (TCP/UDP):

1. **Log in to the Citrix ADC GUI:** Open your web browser and navigate to the management IP of your Citrix appliance.
2. **Create a Syslog Audit Server:**
   - Navigate to **Configuration > System > Auditing > Syslog**.
   - Go to the **Servers** tab and click **Add**.
   - **Name:** Enter a name like `elastic-agent-target`.
   - **IP Address:** Enter the IP address of your Elastic Agent.
   - **Transport:** Select **TCP** or **UDP** (must match your Kibana configuration).
   - **Port:** Enter the port number (e.g., `9001`).
   - **Log Levels:** Check all levels from **EMERGENCY** to **INFORMATIONAL**.
   - **Log Facility:** Select a facility such as **LOCAL2**.
   - Click **Create**.
3. **Create a Syslog Audit Policy:**
   - Go to the **Policies** tab and click **Add**.
   - **Name:** Enter a name like `elastic-syslog-policy`.
   - **Server:** Select the server created in the previous step.
   - **Expression:** Enter `true` to capture all events, or a specific expression to filter logs.
   - Click **Create**.
4. **Bind the Policy globally for WAF:**
   - In the **Auditing > Syslog** section, click **Advanced Policy Global Bindings**.
   - Select your new policy and click **Select**.
   - Set the **Bind Point** to `APPFW_GLOBAL`.
   - Set a **Priority** (e.g., `100`) and click **Bind**.

### Enabling CEF Logging:

1. **Configure Engine Settings:**
   - Navigate to **Security > Citrix Web App Firewall**.
   - In the right-hand pane under **Settings**, click **Change Engine Settings**.
   - Locate the **CEF Logging** checkbox and ensure it is **enabled**. This is a critical requirement for the Elastic integration to parse fields correctly.
   - Click **OK** to apply changes.

## Kibana set up steps

1. In Kibana, navigate to **Integrations** > **Citrix Web App Firewall**.
2. Click **Add Citrix Web App Firewall**.
3. Follow the prompts to add the integration to an existing Elastic Agent policy or create a new one.
4. Configure the input types based on your vendor setup:

### Collecting logs from Citrix Web App Firewall via UDP
- Select the input **Collect logs from Citrix Web App Firewall via UDP**.
- **Listen Address** (`udp_host`): The bind address to listen for UDP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
- **Listen Port** (`udp_port`): The UDP port number to listen on. Default: `9001`.
- **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.

### Collecting logs from Citrix Web App Firewall via TCP
- Select the input **Collect logs from Citrix Web App Firewall via TCP**.
- **Listen Address** (`tcp_host`): The bind address to listen for TCP connections. Set to `0.0.0.0` to bind to all available interfaces. Default: `localhost`.
- **Listen Port** (`tcp_port`): The TCP port number to listen on. Default: `9001`.
- **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.

### Collecting logs from Citrix Web App Firewall via file
- Select the input **Collect logs from Citrix Web App Firewall via file**.
- **Paths** (`paths`): Provide the list of paths to the Citrix WAF log files. Default: `['/var/log/citrix-waf.log']`.
- **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.

5. Save the integration. The Elastic Agent will automatically update its configuration.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on Citrix WAF:
- **Generate Security Events:** Access a web application protected by the Citrix WAF and perform actions that trigger security violations, such as entering a simple SQL injection string (`' OR 1=1 --`) into a search field or a Cross-Site Scripting (XSS) payload (`<script>alert(1)</script>`).
- **Trigger Management Activity:** Log out and log back into the Citrix ADC/NetScaler administration interface to generate audit logs.
- **Configuration Toggle:** Navigate to a non-production firewall profile and toggle a setting (e.g., change a block action to a log action) to generate configuration audit events.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "citrix_waf.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are correctly populated:
   - `event.dataset` (should be `citrix_waf.log`)
   - `source.ip` (the IP address of the client triggering the event)
   - `event.action` or `event.outcome` (showing if the request was blocked or allowed)
   - `message` (containing the raw CEF payload)
5. Navigate to **Analytics > Dashboards** and search for "Citrix Web App Firewall" to view pre-built visualizations and verify the dashboard is populating with data.

# Troubleshooting

## Common Configuration Issues

- **CEF Format Not Enabled**: If logs appear as unstructured strings in the `message` field without specialized Citrix fields, ensure that **CEF Logging** is enabled in the Citrix WAF Engine Settings.
- **Port Binding Conflicts**: If the Elastic Agent fails to start the input, check if another service is already using port **9001**. You can use `netstat -ano | grep 9001` on Linux or `Get-NetTCPConnection -LocalPort 9001` on Windows to identify conflicting processes.
- **Firewall Blocking Traffic**: If no logs are received, verify that the network firewall between the Citrix appliance and the Elastic Agent host allows UDP or TCP traffic on the configured port.
- **Incorrect Bind Point**: If security events are missing but audit logs are present, ensure that the Syslog Policy is bound to the `APPFW_GLOBAL` bind point specifically.

## Ingestion Errors

- **Parsing Failures**: Check the `error.message` field in Kibana. If CEF headers are malformed, the integration may fail to map fields correctly.
- **Mapping Issues**: Ensure that the `citrix` field group is present. If data is in `event.original` but not in ECS fields, verify the version of Citrix WAF matches the supported versions (ADC 13.1 / NetScaler 10.0).
- **Timezone Mismatch**: If logs appear with the wrong timestamp, check the system time on both the Citrix appliance and the Elastic Agent host.

## Vendor Resources

- [How to Send Application Firewall Messages to a Separate Syslog Server](https://support.citrix.com/external/article?articleUrl=CTX138973-how-to-send-application-firewall-messages-to-a-separate-syslog-server)
- [How to Send NetScaler Application Firewall Logs to Syslog Server and NS.log](https://support.citrix.com/external/article?articleUrl=CTX483235-send-logs-to-external-syslog-server&language=en_US)

## Documentation sites

- [How to Send Application Firewall Messages to a Separate Syslog Server](https://support.citrix.com/external/article?articleUrl=CTX138973-how-to-send-application-firewall-messages-to-a-separate-syslog-server)
- [How to Send NetScaler Application Firewall Logs to Syslog Server and NS.log](https://support.citrix.com/external/article?articleUrl=CTX483235-send-logs-to-external-syslog-server&language=en_US)
