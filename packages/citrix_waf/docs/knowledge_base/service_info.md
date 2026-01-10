# Service Info

## Common use cases

- **Security Threat Monitoring:** Monitor and alert on common web attacks such as SQL injection, cross-site scripting (XSS), and buffer overflows detected by the Citrix WAF.
- **Compliance Auditing:** Maintain detailed records of blocked and allowed traffic to satisfy regulatory requirements like PCI-DSS and HIPAA for web application security.
- **Forensic Analysis:** Investigate security incidents by analyzing detailed CEF-formatted logs to understand the nature of malicious requests and targeted vulnerabilities.

## Data types collected

- **Logs:** Security events and audit logs containing detailed information about web request violations, client IP addresses, and firewall actions.
- **Events:** Real-time security alerts formatted in Common Event Format (CEF) for standardized parsing.

## Compatibility

- **Citrix NetScaler (ADC):** Compatible with current releases including 13.x and 12.x.
- **Log Format:** Requires Common Event Format (CEF) to be enabled on the appliance.

## Scaling and Performance

Citrix NetScaler appliances are high-performance application delivery controllers; however, logging volume can impact CPU utilization during high-traffic bursts. It is recommended to use the `LOCAL` syslog facilities and dedicated management interfaces for log offloading to minimize performance impact on data plane traffic.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** SSH access to the NetScaler command-line interface (CLI) with administrative privileges.
- **Feature Licensing:** An active license for the Web App Firewall (AppFirewall) feature must be enabled on the NetScaler appliance.
- **Network Connectivity:** Unrestricted network path from the NetScaler management or SNIP address to the Elastic Agent on the configured syslog port (typically UDP 514 or a custom port).

## Elastic prerequisites

- **Elastic Agent:** An active Elastic Agent installed on a host reachable by the NetScaler appliance.
- **Integration Policy:** The Citrix WAF integration must be added to an Elastic Agent policy with the UDP or TCP port configured to match the NetScaler syslog action.

## Vendor set up steps

1.  **Log in to the NetScaler CLI:** Connect to your NetScaler appliance using an SSH client.
2.  **Enable CEF Logging Format:** Execute the following command to ensure logs are sent in a format compatible with Elastic:
    ```sh
    set appfw settings CEFLogging on
    ```
3.  **Create a Syslog Action:** Define the destination Elastic Agent IP and logging parameters:
    ```sh
    add audit syslogAction elastic-agent-waf-action <elastic-agent-ip> -logLevel ALL -logFacility LOCAL5
    ```
4.  **Create an Advanced Syslog Policy:** Define the rule for triggering log transmission:
    ```sh
    add audit syslogPolicy elastic-agent-waf-policy true elastic-agent-waf-action
    ```
5.  **Bind the Syslog Policy Globally:** Apply the policy specifically to the Web App Firewall context:
    ```sh
    bind audit syslogGlobal -policyName elastic-agent-waf-policy -priority 100 -globalBindType APPFW_GLOBAL
    ```

## Kibana set up steps

1.  In Kibana, navigate to **Management > Integrations**.
2.  Search for **Citrix WAF** and select it.
3.  Click **Add Citrix WAF**.
4.  Configure the **Syslog Host** and **Syslog Port** to match the values used in the NetScaler `syslogAction`.
5.  Assign the integration to an existing or new **Agent Policy**.
6.  Click **Save and continue**.

# Validation Steps

1.  **Generate Test Traffic:** Perform a known violation (e.g., append `<script>alert(1)</script>` to a URL protected by a WAF profile) to trigger a security log.
2.  **Verify Data Stream:** Navigate to **Kibana > Discover** and search for `data_stream.dataset: "citrix_waf.log"`.
3.  **Check Dashboards:** Open the `[Logs Citrix WAF] Overview` dashboard to verify that security events are populating the visualizations.
4.  **Confirm Field Mapping:** Ensure fields like `citrix.waf.action` and `source.ip` are correctly extracted from the CEF message.

# Troubleshooting

## Common Configuration Issues

- **CEF Format Disabled:** If logs appear as unstructured strings, ensure `set appfw settings CEFLogging on` has been executed.
- **Network Routing:** Ensure the NetScaler SNIP or NSIP has a valid route to the Elastic Agent IP and that no intermediate firewalls are blocking the syslog port.
- **Policy Priority:** If multiple syslog policies exist, ensure the `APPFW_GLOBAL` binding has a priority that does not conflict with higher-priority drop rules.

## Ingestion Errors

- **Parsing Failures:** If `error.message` contains "Provided grok patterns did not match," verify that the NetScaler is not adding custom prefixes or timestamps to the syslog header that deviate from standard CEF.
- **Timezone Mismatch:** If logs appear with the wrong timestamp, verify the NetScaler system time and timezone settings match the expected UTC reporting in Elastic.

## API Authentication Errors

Not specified (Integration relies on Syslog push; no API authentication is required for log transmission).

## Vendor Resources

- [NetScaler AppFirewall Documentation](https://docs.netscaler.com/en-us/citrix-adc/current-release/application-firewall.html)
- [Troubleshooting NetScaler Syslog Issues](https://support.citrix.com/article/CTX121731)
- [NetScaler Support Knowledge Center](https://support.citrix.com/)

# Documentation sites

- [NetScaler AppFirewall Logs Reference](https://docs.netscaler.com/en-us/citrix-adc/current-release/application-firewall/logs.html)
- [NetScaler CEF Logging Configuration Guide](https://support.citrix.com/external/article/CTX691236/netscaler-appfirewall-configuration-cef.html)
- [Elastic Integration for Citrix NetScaler WAF](https://docs.elastic.co/en/integrations/citrix_waf)