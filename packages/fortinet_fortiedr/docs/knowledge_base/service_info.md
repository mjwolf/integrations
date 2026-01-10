# Service Info

## Common use cases

*   **Endpoint Threat Monitoring:** Centralize security alerts from FortiEDR agents to detect and respond to advanced persistent threats (APTs) and malware across the enterprise.
*   **Incident Response & Forensics:** Correlate FortiEDR security events with network and cloud logs in the Elastic SIEM to perform deep-dive root cause analysis of security breaches.
*   **Compliance Auditing:** Maintain long-term storage of endpoint security activities and administrative changes for regulatory compliance requirements (e.g., PCI-DSS, HIPAA).

## Data types collected

*   **Logs:** Security events (malware detections, policy violations), system events (manager and collector logs), and administrative audit logs.

## Compatibility

*   **FortiEDR Versions:** Compatible with FortiEDR Central Manager version 4.0, 5.0, 6.0, and 7.x.
*   **Elastic Stack:** Tested against Elastic Stack versions 8.11.0 and higher.

## Scaling and Performance

*   **Throughput:** Performance is largely determined by the number of endpoints and the "Event Forwarding" configuration in the FortiEDR Central Manager. For high-volume environments (10,000+ endpoints), it is recommended to use multiple Elastic Agents behind a load balancer to receive Syslog data.
*   **Protocol:** UDP is faster but lacks delivery guarantees; TCP is recommended for reliable log delivery in enterprise environments.

# Set Up Instructions

## Vendor prerequisites

*   **Access:** Administrator access to the FortiEDR Central Manager console.
*   **Network:** Outbound connectivity from the FortiEDR Central Manager to the Elastic Agent host on the designated Syslog port (e.g., TCP/UDP 9004).
*   **License:** An active FortiEDR license that includes SIEM integration/Syslog forwarding capabilities.

## Elastic prerequisites

*   **Elastic Agent:** An active Elastic Agent installed on a host reachable by the FortiEDR Central Manager.
*   **Integration:** The `Fortinet FortiEDR` integration package must be installed in Kibana.
*   **Port Configuration:** Ensure the host firewall allows traffic on the configured port (default is often 9004 for this integration).

## Vendor set up steps

1.  Log in to the **FortiEDR Central Manager** console.
2.  Navigate to **Administration** > **Settings** (in newer versions, look for **Event Handling** or **Integrations**).
3.  Click on the **Syslog** tab or the **Connectors** section.
4.  Click **Add** to create a new Syslog destination.
5.  Configure the following settings:
    *   **Name:** Elastic SIEM
    *   **Server:** Enter the IP address or FQDN of your Elastic Agent host.
    *   **Port:** Enter the port configured in your Elastic integration (e.g., 9004).
    *   **Protocol:** Select TCP or UDP (match your Elastic integration setting).
    *   **Format:** Select the standard FortiEDR syslog format (the integration is designed to parse the native FortiEDR output).
6.  Select the **Severity** and **Event Types** you wish to forward (e.g., Security Events, System Events).
7.  Click **Save**.

## Kibana set up steps

1.  In Kibana, go to **Management** > **Integrations**.
2.  Search for **Fortinet FortiEDR** and select it.
3.  Click **Add Fortinet FortiEDR**.
4.  Under **Integration Settings**, configure:
    *   **Syslog Host:** Set to `0.0.0.0` to listen on all interfaces or provide a specific IP.
    *   **Syslog Port:** Ensure this matches the port configured in the FortiEDR Central Manager (e.g., 9004).
5.  (Optional) Expand **Advanced options** to configure custom tags or internal log level settings.
6.  Click **Save and continue** to add the integration to an Elastic Agent policy.
7.  Deploy the policy to the Elastic Agent host.

# Validation Steps

1.  **Check Connectivity:** Verify that the Elastic Agent host is listening on the configured port using `netstat -an | grep <port>`.
2.  **Trigger Test Event:** In the FortiEDR console, perform a test action that generates a log, such as an administrative logout/login or an "Exclusion" change.
3.  **Verify Data Flow:** Go to **Kibana** > **Discover**.
4.  Filter for `event.dataset : "fortinet_fortiedr.event"`.
5.  Confirm that fields such as `fortinet.fortiedr.event.type`, `message`, and `source.ip` are populated correctly.
6.  **Check Dashboards:** Open the **[Logs Fortinet FortiEDR] Overview** dashboard to verify visual data representation.

# Troubleshooting

## Common Configuration Issues

*   **Firewall Blockage:** If no data is appearing, ensure that the network firewall and the local host firewall (iptables/firewalld/Windows Firewall) allow traffic from the FortiEDR Manager to the Elastic Agent on the specified port.
*   **Incorrect Port/Protocol:** Double-check that both the FortiEDR Manager and the Elastic Integration are using the exact same protocol (TCP vs. UDP) and port number.

## Ingestion Errors

*   **Parsing Failures:** If data is visible but marked with `error.message`, it may be due to an unsupported log format. Ensure the FortiEDR Manager is using the default syslog format and not a customized LEEF/CEF format unless the integration specifically supports it.
*   **Field Mapping Issues:** Check for `event.outcome: failure` in logs to see if specific fields are failing ECS (Elastic Common Schema) validation.

## API Authentication Errors

*   **Not specified:** This integration primarily uses Syslog for data ingestion. If using an API-based collector (where applicable), ensure the REST API key has "Read-only" or "Security Admin" permissions.

## Vendor Resources

*   [FortiEDR Documentation Library](https://docs.fortinet.com/product/fortiedr/)
*   [FortiEDR Administration Guide - Syslog Configuration](https://docs.fortinet.com/document/fortiedr/7.2.0/administration-guide/109591/syslog)
*   [Fortinet Knowledge Base](https://kb.fortinet.com/)

# Documentation sites

*   [Elastic Fortinet FortiEDR Integration Docs](https://docs.elastic.co/en/integrations/fortinet_fortiedr)
*   [Fortinet Product Overview](https://www.fortinet.com/products/endpoint-security/fortiedr)
*   [Elastic Common Schema (ECS) Reference](https://www.elastic.co/guide/en/ecs/current/index.html)