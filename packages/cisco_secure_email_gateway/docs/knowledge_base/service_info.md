# Service Info

## Common use cases

- **Threat Detection and Response:** Monitor email traffic for phishing attempts, malware distribution, and spam patterns to secure the organization's communication perimeter.
- **Compliance Auditing:** Maintain long-term audit trails of all inbound and outbound email events to meet regulatory requirements and internal security policies.
- **Mail Delivery Troubleshooting:** Analyze consolidated event logs to diagnose delivery failures, latency issues, and configuration errors in the email pipeline.

## Data types collected

This integration primarily collects **Logs**. Specifically, it processes **Consolidated Event Logs** and **Mail Logs** transmitted via the Syslog protocol, preferably in Common Event Format (CEF).

## Compatibility

- **Vendor Versions:** Cisco Secure Email Gateway (ESA) running AsyncOS 15.x, 16.x, and later versions.
- **Format:** Supports Common Event Format (CEF) for standardized field mapping.
- **Deployment:** Compatible with on-premises hardware appliances and virtual instances.

## Scaling and Performance

For high-volume environments processing millions of messages daily, it is recommended to use **TCP** for log transport to ensure reliable delivery. Throughput can be scaled by deploying multiple Elastic Agents behind a load balancer or by configuring multiple Log Subscriptions on the Cisco appliance to distribute the log load.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** Log in credentials with "System Administrator" privileges for the Cisco Secure Email Gateway web interface.
- **Feature Licensing:** Ensure the appliance has an active license for email security features to generate relevant logs.
- **Network Connectivity:** The gateway must have network access to the Elastic Agent on the designated Syslog port (e.g., TCP/514).

## Elastic prerequisites

- **Elastic Agent:** An active Elastic Agent enrolled in Fleet.
- **Integration Policy:** The `Cisco Secure Email Gateway` integration must be added to the agent's policy.
- **Protocol Configuration:** The integration must be configured to listen on the same protocol (TCP or UDP) and port specified in the Cisco appliance settings.

## Vendor set up steps

1. Log in to the web interface of your Cisco Secure Email Gateway appliance.
2. Navigate to **System Administration > Log Subscriptions**.
3. Click **Add Log Subscription**.
4. Select the log type **Consolidated Event Logs**. This ensures logs are sent in the standardized Common Event Format (CEF).
5. Configure the subscription details:
    - **Subscription Name**: Enter a name like `Elastic-Agent-Syslog`.
    - **Retrieval Method**: Select **Syslog Push**.
6. Define the syslog server settings:
    - **Hostname**: Enter the IP address of your Elastic Agent.
    - **Protocol**: Select **TCP** (recommended) or **UDP**.
    - **Port**: Specify the port (default is `514`).
    - **Facility**: Choose a facility (e.g., `Local1`).
7. Click **Submit**.
8. Click the **Commit Changes** button at the top of the page and confirm to activate the configuration.

## Kibana set up steps

1. In Kibana, go to **Management > Integrations**.
2. Search for **Cisco Secure Email Gateway** and select it.
3. Click **Add Cisco Secure Email Gateway**.
4. Configure the integration settings:
    - **Syslog Host**: Set to `0.0.0.0` to listen on all interfaces or the specific IP of the agent host.
    - **Syslog Port**: Enter the port configured in step 6 above (e.g., `514`).
    - **Protocol**: Select the matching protocol (TCP or UDP).
5. Assign the integration to an existing **Agent Policy**.
6. Click **Save and continue**.

# Validation Steps

1. **Verify Connectivity:** Ensure the Elastic Agent is "Healthy" in **Fleet > Agents**.
2. **Generate Test Traffic:** Send a test email through the gateway to trigger log generation.
3. **Check Data Stream:** Go to **Kibana > Discover** and filter by `data_stream.dataset : "cisco_secure_email_gateway.email_gateway"`.
4. **Dashboard Inspection:** Open the **[Logs Cisco Secure Email Gateway] Overview** dashboard to verify that visualizations are populating with data.

# Troubleshooting

## Common Configuration Issues

- **Uncommitted Changes:** Logging will not start until the "Commit Changes" button is clicked in the Cisco ESA web interface.
- **Firewall Restrictions:** Ensure that intermediate firewalls or host-based firewalls (iptables/firewalld) allow traffic on the configured Syslog port.
- **Protocol Mismatch:** If the Cisco appliance is set to TCP and the Elastic integration is set to UDP, no data will be ingested.

## Ingestion Errors

- **Parsing Failures:** If `error.message` appears in the logs, ensure the Log Subscription on the Cisco device is set to **Consolidated Event Logs**. Generic "Mail Logs" may not match the expected CEF parser.
- **Timezone Offsets:** If logs appear with the wrong timestamp, verify the system time and timezone settings on both the Cisco appliance and the Elastic Agent host.

## API Authentication Errors

Not specified (This integration uses Syslog push and does not typically require API-based authentication for standard log ingestion).

## Vendor Resources

- [Cisco Secure Email Gateway Troubleshooting Guides](https://www.cisco.com/c/en/us/support/security/email-security-appliance/series-resources.html)
- [Cisco Content Security Log Types Reference](https://www.cisco.com/c/en/us/support/docs/security/email-security-appliance/118841-technote-esa-00.html)

# Documentation sites

- [Cisco Secure Email Gateway Product Overview](https://www.cisco.com/c/en/us/products/security/email-security/index.html)
- [AsyncOS 15.0 User Guide - Logging Section](https://www.cisco.com/c/en/us/td/docs/security/security_management/sma/sma15-0-1/user_guide/b_sma_admin_guide_15_0_1/b_NGSMA_Admin_Guide_chapter_01100.html)
- [Elastic Integration Documentation](https://docs.elastic.co/integrations/cisco_secure_email_gateway)