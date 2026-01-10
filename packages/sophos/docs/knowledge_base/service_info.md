# Service Info

## Common use cases

- **Network Security Monitoring**: Monitor firewall traffic and identify blocked connections or unauthorized access attempts across the infrastructure.
- **Threat Detection and Incident Response**: Analyze log data to identify patterns indicative of cyberattacks, such as SQL injection or port scanning, and respond to security incidents.
- **Compliance Reporting**: Maintain a centralized audit trail of network activity and administrative changes for regulatory requirements like GDPR, HIPAA, or PCI-DSS.

## Data types collected

- **Logs**: Detailed syslog messages including firewall traffic, web filtering events, application control, and administrative system activities.

## Compatibility

- **Sophos XG Firewall (SFOS)**: Version 17.x and later.
- **Sophos UTM**: Version 9.x and later.
- **Elastic Agent**: Tested against standard syslog inputs.

## Scaling and Performance

- Sophos devices support multiple syslog server destinations; performance depends on the hardware capacity of the Sophos appliance and the network throughput of the syslog facility. 
- For high-volume environments, use multiple Elastic Agents or a dedicated load balancer to handle incoming syslog traffic on port 514 (UDP/TCP).

# Set Up Instructions

## Vendor prerequisites

- Administrative access to the Sophos XG Firewall or Sophos UTM web interface.
- Connectivity between the Sophos appliance and the server hosting the Elastic Agent.
- For Sophos XG, ensure the device is configured to use the "Central Reporting Format" for compatibility with the Elastic parser.

## Elastic prerequisites

- An active Elastic Stack (Elasticsearch and Kibana) environment.
- Elastic Agent installed on a host reachable by the Sophos device.
- The Sophos integration added to an Elastic Agent policy with the correct Syslog host and port defined (default 514).

## Vendor set up steps

### Sophos XG Firewall (SFOS)
1. Log in to the Sophos XG Firewall admin console.
2. Navigate to **System > System Services > Log Settings**.
3. In the **Syslog Servers** section, click **Add**.
4. Configure the server:
   - **Name**: `elastic-agent`
   - **IP Address**: The IP address of your Elastic Agent host.
   - **Port**: `514` (or your configured port).
   - **Facility**: `DAEMON`.
   - **Severity**: `Information`.
   - **Format**: `Central Reporting Format` (Required).
5. Click **Save**.
6. Scroll down on the **Log Settings** page and check the boxes for the log types (e.g., Firewall, IPS, Web Filter) you wish to forward to the new syslog server.
7. Click **Apply**.

### Sophos UTM
1. Log in to the Sophos UTM admin console.
2. Navigate to **Logging & Reporting > Log Settings > Remote Syslog Server**.
3. Toggle **Remote Syslog Status** to **Enabled**.
4. Click **Add** to define a new server and enter the Elastic Agent IP and Port (default 514).
5. Click **Save**.
6. Switch to the **Remote Syslog Log Selection** tab.
7. Select the desired logs (e.g., Firewall, Web Filtering, Intrusion Prevention) and move them to the "Selected" list.
8. Click **Apply**.

## Kibana set up steps

1. In Kibana, go to **Management > Integrations**.
2. Search for **Sophos** and select the integration.
3. Click **Add Sophos**.
4. Configure the integration:
   - Select the **Logs** checkbox.
   - Set the **Syslog Host** (e.g., `0.0.0.0` to listen on all interfaces).
   - Set the **Syslog Port** to match your Sophos device configuration (default `514`).
5. Choose the **Existing Policy** where your Elastic Agent is enrolled.
6. Click **Save and continue**.

# Validation Steps

1. **Check Data Stream**: Navigate to **Management > Dev Tools** and run `GET logs-sophos.firewall-default/_search` to verify documents are arriving.
2. **Review Dashboards**: Open the **[Logs Sophos] Overview** dashboard in Kibana to ensure visualizations are populating with traffic data.
3. **Verify Field Mapping**: Ensure the `event.dataset` field is correctly identified as `sophos.firewall` or similar to confirm the parser is working.

# Troubleshooting

## Common Configuration Issues

- **Port Conflicts**: Ensure no other service (like rsyslog or syslog-ng) is already using port 514 on the Elastic Agent host.
- **Firewall Restrictions**: Verify that local firewalls (iptables, firewalld, or Windows Firewall) on the Elastic Agent host allow inbound traffic on the configured syslog port.
- **Incorrect Log Format**: If logs appear as raw text without parsed fields, double-check that Sophos XG is set to **Central Reporting Format**.

## Ingestion Errors

- **Parsing Failures**: If `error.message` appears in the logs, it typically indicates a non-standard syslog header. Ensure the Sophos device is not adding custom prefixes to syslog messages.
- **Timezone Mismatch**: If logs appear with the wrong timestamp, verify the timezone settings on both the Sophos appliance and the Elastic Agent host.

## API Authentication Errors

- Not specified (This integration primarily uses Syslog and does not require API credentials for standard log forwarding).

## Vendor Resources

- [Sophos Firewall: Add a syslog server](https://docs.sophos.com/nsg/sophos-firewall/20.0/Help/en-us/webhelp/onlinehelp/AdministratorHelp/SystemServices/LogSettings/SyslogServerAdd/)
- [Sophos UTM: Remote Syslog Configuration](https://docs.sophos.com/nsg/sophos-utm/utm/9.717/help/en-us/content/utm/utmadminguide/loggingsettingsremotesyslogserver.htm)
- [Sophos Community Support](https://community.sophos.com/)

# Documentation sites

- [Sophos Product Documentation Home](https://docs.sophos.com/)
- [Elastic Sophos Integration Reference](https://docs.elastic.co/integrations/sophos)
- [Filebeat Sophos Module Reference](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-sophos.html)