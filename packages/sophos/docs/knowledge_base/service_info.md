# Service Info

## Common use cases

The Sophos integration allows organizations to ingest security events and network traffic data from Sophos Firewall (XG/SFOS) devices into the Elastic Stack. This provides centralized visibility into network threats and system health.
- **Security Monitoring and Threat Detection:** Identify potential intrusions by analyzing Intrusion Prevention System (IPS) and Antivirus logs in real-time within Kibana dashboards.
- **Compliance Auditing:** Maintain long-term records of administrative changes, system access, and firewall configuration modifications to satisfy regulatory requirements like PCI-DSS or HIPAA.
- **Network Traffic Analysis:** Visualize traffic patterns, application usage, and bandwidth consumption across different firewall rules to optimize network performance and security policies.
- **Incident Response:** Investigate security breaches by correlating Sophos firewall events with logs from other data sources using the Elastic Common Schema (ECS).

## Data types collected

This integration can collect the following types of data from Sophos Firewall devices via the Syslog protocol:
- **Firewall Logs:** Detailed records of allowed and denied traffic, including source/destination IP addresses, ports, protocols, and rule IDs.
- **Security Logs:** Events related to Intrusion Prevention (IPS), Advanced Threat Protection (ATP), Antivirus scanning, and Heartbeat status.
- **System Logs:** Operational data including system health, service status, high availability (HA) events, and administrative audit trails (logins/config changes).
- **Web and App Logs:** Information on web categories visited, application signatures identified, and policy enforcement actions (block/warn/allow).
- **Data Formats:** All data is received in **Standard Syslog** format (RFC 5424) or the device-specific key-value pair format, which is then mapped to the Elastic Common Schema (ECS).

## Compatibility

The Sophos integration is designed for devices running the **Sophos Firewall (SFOS)** operating system.
- **Tested Versions:** This integration is compatible with **SFOS v19.0**, **SFOS v19.5**, **SFOS v20.0**, and **SFOS v21.0**.
- **Limitations:** Legacy Sophos UTM (SG series) or Sophos XG running older firmware may require custom mapping if the log format deviates significantly from the SFOS standard.

## Scaling and Performance

- **Transport/Collection Considerations:** This integration primarily uses Syslog over UDP or TCP. While UDP offers lower overhead and higher throughput, it does not guarantee delivery. For mission-critical security environments where data integrity is paramount or where encryption is required, TCP with TLS (Secure Log Transmission) should be configured to ensure reliable and encrypted delivery to the Elastic Agent.
- **Data Volume Management:** Sophos Firewalls can generate massive amounts of data, particularly "Allowed" traffic logs. To manage volume, administrators should use the **Log Settings** menu in SFOS to select only necessary log modules (e.g., IPS and System Health) and configure firewall rules to only log "dropped" traffic unless "allowed" traffic is specifically needed for compliance.
- **Elastic Agent Scaling:** For high-volume environments exceeding 5,000 Events Per Second (EPS), it is recommended to deploy multiple Elastic Agents behind a load balancer. A single Agent's capacity is determined by available CPU and memory; ensure the host machine has at least 4 vCPUs and 8GB of RAM when acting as a dedicated Syslog collector for multiple high-traffic firewalls.

# Set Up Instructions

## Vendor prerequisites

Before configuring the integration, ensure the following requirements are met on the Sophos Firewall:
- **Administrative Access:** You must have an administrator account with Read-Write permissions for **System services** and **Rules and policies**.
- **Network Connectivity:** The Sophos Firewall must have a clear network path to the Elastic Agent's IP address. Ensure any intermediate firewalls allow traffic on the configured syslog port (default is `514`).
- **Firmware Version:** Ensure the firewall is running a supported version of SFOS (preferably v18.0 or later) to support the "Standard syslog protocol" format.
- **Encryption Requirements:** If using "Secure log transmission," you must have the appropriate CA certificates imported into both the Sophos Firewall and the Elastic Agent host.
- **License Requirements:** A valid Base Firewall license is required to access log settings and send external syslog.

## Elastic prerequisites

- **Elastic Agent Deployment:** An Elastic Agent must be installed and successfully enrolled in a policy managed by Fleet.
- **Integration Installation:** The "Sophos" integration package must be added to the Agent policy.
- **Inbound Connectivity:** The host running the Elastic Agent must be configured to allow inbound traffic on the Syslog port (e.g., `514/UDP` or `514/TCP`) through its local OS firewall (iptables, firewalld, or Windows Firewall).

## Vendor set up steps

### For Syslog Collection:
1. Log in to your Sophos Firewall web administration console as an administrator.
2. Navigate to **System services** > **Log settings** using the left-hand navigation menu.
3. Locate the **Syslog servers** section and click the **Add** button to define a new destination.
4. Fill in the configuration fields for the Elastic Agent:
    *   **Name**: Enter a unique identifier, such as `Elastic_Agent_Collector`.
    *   **IP address/domain**: Provide the IP address or FQDN of the server hosting the Elastic Agent.
    *   **Port**: Enter the port number (default is `514`). Ensure this matches the port you will configure in Kibana.
    *   **Secure log transmission**: Toggle this "On" if you require TLS encryption; otherwise, leave it disabled for standard UDP/TCP.
    *   **Facility**: Choose a facility code (e.g., `LOCAL0`). This is used for log categorization.
    *   **Severity level**: Select `Information` or `Debug` to ensure all relevant security data is captured.
    *   **Format**: Select **Standard syslog protocol**. This is critical for the Elastic Agent to correctly parse the log fields.
5. Click **Save** to apply the server configuration.
6. Return to the main **Log settings** page. In the list of log modules, find the column corresponding to your new Syslog server.
7. Check the boxes for all data types you wish to export: **Firewall**, **IPS**, **Antivirus**, **Anti-spam**, **Content filtering**, **Web filter**, **App filter**, and **System health**.
8. Click **Apply** at the bottom of the page to start the stream.
9. **CRITICAL:** To receive traffic logs, you must enable logging on individual firewall rules. Navigate to **Rules and policies** > **Firewall rules**, edit your active rules, scroll to the bottom, and ensure **Log firewall traffic** is enabled.

## Kibana set up steps

### For Sophos Syslog Input:

1.  In Kibana, navigate to **Management > Integrations**.
2.  Search for **Sophos** and select the integration tile.
3.  Click **Add Sophos** to start the configuration.
4.  Select the **Sophos Firewall logs** data stream and configure the following:
    *   **Syslog Host**: Set this to `0.0.0.0` to listen on all available interfaces, or provide a specific interface IP.
    *   **Syslog Port**: Enter the port number (e.g., `514` or `5514`) that matches the value you entered in the Sophos Firewall console.
    *   **Protocol**: Select either `udp` or `tcp` based on your vendor-side configuration.
5.  (Optional) Expand **Advanced options** to configure internal tags or custom log delimiters if your environment requires them.
6.  Choose the existing **Agent policy** that is applied to your target Elastic Agent.
7.  Click **Save and continue**, then **Add agent integration** to deploy the configuration to your Elastic Agent.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from Sophos to the Elastic Stack.

### 1. Trigger Data Flow on Sophos:
- **Generate Web Traffic:** From a workstation located behind the Sophos Firewall, browse to several external websites. This will trigger "Firewall" and "Web Filter" log entries.
- **Generate Authentication Event:** Log out of the Sophos Firewall web administrator console and log back in immediately. This generates "System" or "Authentication" logs.
- **Generate Configuration Event:** Navigate to any setting in the firewall (e.g., a description field in a network object), make a minor change, and click Save. This triggers an audit/system log.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the following KQL filter: `data_stream.dataset : "sophos.firewall"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `sophos.firewall` or similar based on the specific log type)
   - `source.ip` and `destination.ip` (containing the IP addresses of the test traffic)
   - `event.action` (e.g., `allowed`, `denied`, or `logged`)
   - `sophos.firewall.log_component` (identifying the module such as `firewall` or `http`)
   - `message` (containing the raw syslog string from the Sophos device)
5. Navigate to **Analytics > Dashboards** and search for "Sophos" to view the pre-built **[Logs Sophos] Overview** dashboard and verify that visualizations are populating with data.

# Troubleshooting

## Common Configuration Issues

- **Firewall Rule Logging Disabled**: One of the most common issues is that while the Syslog server is configured globally, individual firewall rules are not set to "Log firewall traffic." Verify this in **Rules and policies > Firewall rules**.
- **Incorrect Syslog Format**: If logs are appearing in Kibana as "original_message" only with parsing errors, ensure the Sophos Firewall is set to **Standard syslog protocol** and NOT "Device standard format (legacy)" or "Central Reporting format."
- **Network Port Conflict**: If the Elastic Agent fails to start the syslog listener, check if another service (like rsyslog or syslog-ng) is already bound to port 514 on the host machine. Use `netstat -tulpn | grep 514` to identify conflicting services.
- **Facility Code Mismatch**: If you are using multiple syslog integrations on one Agent, ensure each uses a unique Port or Facility code if you are doing any custom routing.

## Ingestion Errors

- **Parsing Failures**: If logs appear in Kibana but have a `tags: ["_grokparsefailure"]`, check the **Format** setting in the Sophos Syslog configuration. It must be set to the default "Sophos" format; using "CEF" or other custom formats will break the default Elastic parser.
- **Timezone Mismatches**: If logs appear to be delayed or from the future, verify that the Sophos Firewall and the Elastic Agent host are synchronized via NTP and that the correct timezone is set on the Sophos appliance.
- **Field Mapping Issues**: Check the `error.message` field in Discover. If a mapping conflict occurs, ensure that you are using the latest version of the Sophos integration and that no legacy index templates are interfering with the data stream.

## Vendor Resources

- [Log settings - Sophos Firewall](https://docs.sophos.com/nsg/sophos-firewall/21.0/help/en-us/webhelp/onlinehelp/AdministratorHelp/SystemServices/LogSettings/index.html)
- [Add a syslog server - Sophos Firewall](https://docs.sophos.com/nsg/sophos-firewall/21.0/help/en-us/webhelp/onlinehelp/AdministratorHelp/SystemServices/LogSettings/SyslogServerAdd/index.html)

# Documentation sites

- [Log settings - Sophos Firewall](https://docs.sophos.com/nsg/sophos-firewall/21.0/help/en-us/webhelp/onlinehelp/AdministratorHelp/SystemServices/LogSettings/index.html)
- [Add a syslog server - Sophos Firewall](https://docs.sophos.com/nsg/sophos-firewall/21.0/help/en-us/webhelp/onlinehelp/AdministratorHelp/SystemServices/LogSettings/SyslogServerAdd/index.html)
- Refer to the official Sophos documentation portal for further SFOS configuration guides.
