# Service Info

## Common use cases

- **Network Health Monitoring:** Monitor system-level events, hardware status, and interface flaps across Cisco Nexus data center switches.
- **Security Auditing:** Track user authentication attempts, configuration changes, and command execution via audit logs to maintain compliance.
- **Fault Troubleshooting:** Rapidly identify the root cause of network outages by correlating syslog severity levels with specific system component failures.

## Data types collected

- **Logs:** System messages (syslog) including kernel, environment, and interface logs.
- **Events:** Audit logs for administrative actions and configuration changes.

## Compatibility

- **Cisco NX-OS:** Compatible with devices running Cisco NX-OS, including Cisco Nexus 3000, 5000, 7000, and 9000 Series switches.
- **Tested Versions:** Documentation specifically references support for NX-OS Release 10.1(x) and 10.6(x).

## Scaling and Performance

- **Control Plane Limits:** Cisco Nexus devices handle syslog generation via the control plane; excessive logging (e.g., severity level 7) during high-traffic events can impact CPU utilization.
- **Transmission:** Syslog is typically sent over UDP, which has no flow control; ensure sufficient bandwidth between the management/loopback interface and the Elastic Agent.
- **Persistence:** NX-OS stores buffered logs locally in `/var/log/external/`, which persist across reloads, providing a fallback if the syslog server is temporarily unreachable.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** Connect to the switch via SSH or console with a user account assigned the `network-admin` role.
- **Network Connectivity:** Ensure the switch has a route to the Elastic Agent and that UDP port 514 (or your configured port) is open on intermediate firewalls.
- **Source Interface:** A loopback or management interface should be configured and reachable to ensure consistent log sourcing.

## Elastic prerequisites

- **Elastic Agent:** Install and enroll an Elastic Agent on a host reachable by the Cisco Nexus switch.
- **Integration Policy:** Add the "Cisco Nexus" integration to an Elastic Agent policy.
- **Input Configuration:** Note the IP address and port (default 514) configured for the syslog input in the integration settings.

## Vendor set up steps

1.  **Enter Global Configuration Mode:**
    ```bash
    switch# configure terminal
    ```

2.  **Configure the Syslog Server:**
    Add the Elastic Agent as a destination.
    ```bash
    switch(config)# logging server <elastic-agent-ip>
    ```

3.  **Specify Logging Severity Level (Optional):**
    Set the severity level to control volume. Level 5 (notification) is recommended for production.
    ```bash
    switch(config)# logging server <elastic-agent-ip> 5
    ```

4.  **Specify the Syslog Facility (Optional):**
    The default is `local7`. If you changed this in the Elastic integration settings, match it here:
    ```bash
    switch(config)# logging server <elastic-agent-ip> facility local7
    ```

5.  **Set the Source Interface:**
    Explicitly define the interface from which logs originate.
    ```bash
    switch(config)# logging source-interface loopback0
    ```

6.  **Enable Timestamps:**
    ```bash
    switch(config)# logging timestamp
    ```

7.  **Verify and Save:**
    ```bash
    switch(config)# show logging server
    switch(config)# copy running-config startup-config
    ```

## Kibana set up steps

1.  In Kibana, go to **Management** > **Integrations**.
2.  Search for and select **Cisco Nexus**.
3.  Click **Add Cisco Nexus**.
4.  Configure the integration name and the **Syslog Host** (the IP the agent listens on) and **Syslog Port** (default 514).
5.  Select the **Existing host policy** where your Elastic Agent is enrolled.
6.  Click **Save and continue**.

# Validation Steps

1.  **Check Switch Output:** Run `show logging server` on the switch to confirm the status is "active" or "enabled."
2.  **Verify Data Ingestion:** In Kibana, go to **Logs** > **Stream** and filter by `event.dataset: cisco_nexus.log`.
3.  **Inspect Dashboard:** Navigate to the **[Logs Cisco Nexus] Overview** dashboard in Kibana to see visualized log data.
4.  **Trigger Test Event:** Change a description on a dummy interface (e.g., `interface loopback99`, `description TEST`) and check for the corresponding audit log in Kibana.

# Troubleshooting

## Common Configuration Issues

- **Firewall Blockage:** If logs don't appear, verify that UDP port 514 is open on the host firewall and any network firewalls between the switch and the agent.
- **Unreachable Source IP:** If `logging source-interface` is configured but that IP is not routable to the agent, the packets will be dropped.
- **VRF Misconfiguration:** On Nexus switches, ensure the syslog server is reachable via the VRF being used (e.g., `logging server <IP> vrf management`).

## Ingestion Errors

- **Timestamp Parsing:** If `error.message` indicates timestamp issues, ensure `logging timestamp` is enabled on the switch. 
- **Incorrect Facility:** If logs are missing, verify that the `facility` configured on the switch matches the input facility expected by the Elastic integration.

## API Authentication Errors

- **Not specified:** This integration primarily uses Syslog (UDP/TCP) and does not typically use API-based polling for logs.

## Vendor Resources

- [Cisco NX-OS Troubleshooting Tools Guide](https://www.ciscopress.com/articles/article.asp?p=2928194&seqNum=6)
- [Cisco Nexus 9000 Series Troubleshooting Guide](https://www.manualslib.com/manual/1258982/Cisco-Nexus-9000-Series.html)
- [Cisco Support Community - Network Management](https://community.cisco.com/t5/network-management/bd-p/5851-discussions-network-management)

# Documentation sites

- [Cisco Nexus 3600 System Management Guide](https://www.cisco.com/c/en/us/td/docs/dcn/nx-os/nexus3600/106x/configuration/sys-mgmt/cisco-nexus-3600-switch-nx-os-system-management-configuration-guide-106x/m-configuring-system-message-logging-10x.html)
- [Cisco Nexus 9000 System Management Guide](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/nexus9000/sw/106x/config-guides/sys-mgmt/cisco-nexus-9000-series-nx-os-system-management-configuration-guide-release-106x/chapter-1.html)
- [Elastic Cisco Nexus Integration Reference](https://docs.elastic.co/integrations/cisco_nexus)