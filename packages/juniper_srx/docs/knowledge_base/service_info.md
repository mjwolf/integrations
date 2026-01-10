# Service Info

## Common use cases

*   **Security Monitoring and Auditing:** Track firewall session creations, denials, and closures to identify potential security threats or unauthorized access attempts.
*   **Compliance and User Activity Tracking:** Monitor control-plane logs for user authentications and system configuration changes (UI_COMMIT) to meet regulatory compliance requirements.
*   **Network Troubleshooting:** Analyze system events and traffic flow logs to diagnose connectivity issues and monitor the health of Juniper SRX devices.

## Data types collected

*   **Logs:** System logs (control-plane) and security logs (data-plane).
*   **Events:** Firewall session events, authentication events, and system operational messages.

## Compatibility

*   Juniper SRX Series devices.
*   Junos OS (Standard and FIPS versions).
*   Tested against Junos OS 12.1 and later versions supporting `stream` mode logging.

## Scaling and Performance

*   **Stream Mode:** For high-traffic environments, configuring security logs in `stream` mode is recommended to minimize CPU impact on the Routing Engine.
*   **Throughput:** Performance depends on the device hardware; use `sd-syslog` format to ensure structured data is handled efficiently by the Elastic Agent.

# Set Up Instructions

## Vendor prerequisites

*   Administrative access to the Juniper SRX Command Line Interface (CLI).
*   Network connectivity between the SRX device and the Elastic Agent.
*   The SRX device must be in a state where the `security` and `system` hierarchies are configurable.

## Elastic prerequisites

*   Elastic Agent must be installed and enrolled in Fleet.
*   The "Juniper SRX" integration must be added to an Elastic Agent policy.
*   The Syslog port (default 514) must be open on the host running the Elastic Agent to receive UDP/TCP traffic.

## Vendor set up steps

1.  Log in to the Juniper SRX CLI and enter configuration mode:
    ```shell
    configure
    ```
2.  **Configure System (Control-Plane) Logging:**
    ```shell
    set system syslog host <elastic-agent-ip> any info
    set system syslog host <elastic-agent-ip> port <port>
    set system syslog host <elastic-agent-ip> match "(UI_COMMIT:)|(FLOW_SESSION_CREATE)|(FLOW_SESSION_DENY)|(FLOW_SESSION_CLOSE)"
    ```
3.  **Configure Security (Data-Plane) Logging:**
    ```shell
    set security log mode stream
    set security log source-address <srx-ip-address>
    set security log stream elastic-agent format sd-syslog
    set security log stream elastic-agent host <elastic-agent-ip>
    set security log stream elastic-agent port <port>
    ```
4.  Apply the configuration:
    ```shell
    commit
    exit
    ```

## Kibana set up steps

1.  In Kibana, go to **Management > Integrations**.
2.  Search for **Juniper SRX** and click **Add Juniper SRX**.
3.  Configure the integration settings:
    *   **Syslog Host:** Set to `0.0.0.0` or the specific IP the agent listens on.
    *   **Syslog Port:** Match the port configured on the SRX device (e.g., `514`).
4.  Assign the integration to the relevant **Elastic Agent policy**.
5.  Click **Save and continue**.

# Validation Steps

1.  **Check Data Stream:** Navigate to **Observability > Logs > Explorer** (or Discover) and filter by `event.dataset : "juniper_srx.log"`.
2.  **Trigger Test Event:** Perform a configuration commit on the SRX or trigger a firewall policy violation to generate a log entry.
3.  **Verify Dashboard:** Open the **[Metrics Juniper SRX] Overview** dashboard in Kibana to ensure visualizations are populating correctly.

# Troubleshooting

## Common Configuration Issues

*   **Logs Not Sent:** Ensure the `commit` command was executed after configuration. Verify that the `source-address` in the security log stream is reachable by the Elastic Agent.
*   **Port Conflicts:** Ensure no other process is using the Syslog port on the Elastic Agent host.
*   **Firewall Blocks:** Verify that intermediary firewalls allow traffic on the configured UDP/TCP port from the SRX to the Elastic Agent.

## Ingestion Errors

*   **Parsing Failures:** Ensure the SRX is using `format sd-syslog`. Non-structured syslog formats may result in `error.message` fields or unparsed message bodies.
*   **Timestamp Mismatch:** Check that the SRX device and the Elastic Agent host have synchronized clocks via NTP to avoid indexing issues.

## API Authentication Errors

Not specified (Integration uses Syslog/UDP/TCP; does not utilize vendor APIs for data collection).

## Vendor Resources

*   [Juniper SRX Troubleshooting Guide](https://supportportal.juniper.net/s/article/SRX-Getting-Started-Configure-System-Logging)
*   [Junos OS System Log Messages Reference](https://www.juniper.net/documentation/us/en/software/junos/network-mgmt/topics/topic-map/system-logging.html)
*   [Tufin Knowledge Base - SRX Syslog Config](https://forum.tufin.com/support/kc/latest/Content/Suite/2829.htm)

# Documentation sites

*   [Juniper Networks Official Documentation](https://www.juniper.net/documentation/)
*   [Junos OS Security Configuration Guide](https://www.juniper.net/documentation/us/en/software/junos/security-policies/index.html)
*   [ManageEngine KB - SRX CLI Configuration](https://pitstop.manageengine.com/portal/en/kb/articles/juniper-srx-configuration-from-cli)