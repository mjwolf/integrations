# Service Info

## Common use cases

*   **Network Traffic Auditing:** Monitor allowed and denied connections to identify potential security breaches or policy violations across the corporate perimeter.
*   **VPN Session Monitoring:** Track Remote Access VPN (AnyConnect) and Site-to-Site VPN connections, including login/logout events, session durations, and assigned IP addresses.
*   **System Health and Threat Detection:** Identify hardware failures, configuration changes, and high-severity security events like Shun operations or DoS attack detection.

## Data types collected

This integration primarily collects **Logs** in syslog format, which include connection events, system status messages, security alerts, and VPN session details.

## Compatibility

*   **Cisco ASA:** Compatible with Cisco ASA software versions 9.x (documented version 9.23).
*   **Deployment Models:** Supports physical ASA appliances, ASAv (virtual), and Firepower devices running ASA software.

## Scaling and Performance

Performance is dependent on the ASA hardware model and the configured logging severity. For high-volume environments, it is recommended to use **UDP** to avoid performance impact on the firewall, or ensure `logging permit-hostdown` is configured if using **TCP**. Large-scale deployments can utilize Cisco ASA Clustering to distribute traffic and logging load across multiple nodes.

# Set Up Instructions

## Vendor prerequisites

*   Administrative access (Enable password) to the Cisco ASA Command-Line Interface (CLI).
*   Network connectivity between the Cisco ASA and the host running the Elastic Agent.
*   Valid license for Cisco ASA operations (Base license minimum).

## Elastic prerequisites

*   Elastic Agent must be installed and enrolled in a policy.
*   A "Cisco ASA" integration must be added to the agent policy.
*   The firewall must be able to reach the Elastic Agent's IP address on the configured UDP or TCP port (default 514 or 1470).

## Vendor set up steps

1.  **Enter Global Configuration Mode:**
    ```sh
    enable
    configure terminal
    ```
2.  **Enable Logging:**
    ```sh
    logging enable
    ```
3.  **Configure the Syslog Server:**
    Replace `[interface_name]` with the egress interface and `[elastic_agent_ip]` with the agent's address.
    ```sh
    logging host [interface_name] [elastic_agent_ip] [tcp|udp]/[port]
    ```
4.  **Set Logging Severity:**
    ```sh
    logging trap informational
    ```
5.  **Enable Timestamps:**
    ```sh
    logging timestamp
    ```
6.  **Set Device ID:**
    ```sh
    logging device-id hostname
    ```
7.  **Configure TCP Fail-Safe (Required for TCP):**
    ```sh
    logging permit-hostdown
    ```

## Kibana set up steps

1.  Log in to Kibana and navigate to **Management > Integrations**.
2.  Search for **Cisco ASA** and select it.
3.  Click **Add Cisco ASA**.
4.  Configure the **Syslog Host** and **Port** to match the values used in the CLI setup.
5.  Select the **Agent Policy** where your Elastic Agent is enrolled.
6.  Click **Save and Continue** to deploy the integration.

# Validation Steps

1.  **Check ASA Statistics:** Run `show logging` on the ASA to verify that "Messages logged" and "Syslog logging" counters are incrementing.
2.  **Test Connection:** Generate a log event (e.g., attempt an unauthorized SSH login or trigger a firewall rule).
3.  **Verify in Kibana:** Navigate to **Observability > Logs** or the **[Logs Cisco ASA] Overview** dashboard to confirm data is appearing.
4.  **Check Field Mapping:** Ensure fields such as `cisco.asa.message_id` and `source.ip` are correctly populated.

# Troubleshooting

## Common Configuration Issues

*   **No Data Flowing:** Ensure the ASA interface used in `logging host` has a route to the Elastic Agent. Check for intermediate firewalls blocking port 514 (UDP) or 1470 (TCP).
*   **ASA Blocking Traffic:** If using TCP logging without `logging permit-hostdown`, the ASA will stop passing traffic if the Elastic Agent becomes unreachable. Always verify this setting is active.
*   **Clock Skew:** If logs appear with incorrect timestamps, ensure `logging timestamp` is enabled and the ASA is synchronized via NTP.

## Ingestion Errors

*   **Parsing Failures:** If `error.message` appears, it is often due to non-standard syslog formats. Ensure no custom syslog headers are added via the ASA CLI.
*   **Field Mismatches:** If the Elastic Agent is receiving data but fields are missing, verify that the Cisco ASA integration version matches your Elastic Stack version.

## API Authentication Errors

Not specified (Cisco ASA syslog integration uses push-based syslog rather than pull-based APIs).

## Vendor Resources

*   [Cisco ASA Troubleshooting Guide](https://www.cisco.com/c/en/us/support/security/asa-5500-series-next-generation-firewalls/products-troubleshooting-guides-list.html)
*   [Cisco Support Community - Security](https://community.cisco.com/t5/security-knowledge-base/tkb-p/4461-kb-security)
*   [NetworkLessons: Cisco ASA Syslog Configuration](https://networklessons.com/cisco/asa-firewall/cisco-asa-syslog-configuration)

# Documentation sites

*   [Cisco Secure Firewall ASA CLI Configuration Guide](https://www.cisco.com/c/en/us/td/docs/security/asa/asa923/configuration/general/asa-923-general-config/monitor-syslog.html)
*   [Cisco ASA Syslog Message Guide](https://www.cisco.com/c/en/us/td/docs/security/asa/syslog/b_syslog.html)
*   [Elastic Integration Documentation for Cisco ASA](https://docs.elastic.co/integrations/cisco_asa)