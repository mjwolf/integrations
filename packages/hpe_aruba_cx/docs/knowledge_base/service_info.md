# Service Info

## Common use cases

The ArubaOS-CX (AOS-CX) integration allows you to ingest syslog data from your Aruba switching infrastructure into the Elastic Stack. This provides centralized visibility into the health, security, and operational status of your network fabric.
- **Network Performance Monitoring:** Track interface state changes, link flaps, and protocol transitions (OSPF, BGP) to identify and resolve connectivity issues before they impact users.
- **Security Auditing:** Monitor authentication attempts (AAA), configuration changes, and access-control list (ACL) hits to detect unauthorized access or malicious internal activity.
- **Operational Compliance:** Maintain a long-term record of system events and administrative actions required for regulatory compliance and internal security policies.
- **Proactive Hardware Health:** Capture alerts related to power supplies, fan speeds, and temperature thresholds to prevent hardware failures and maintain uptime.

## Data types collected

This integration can collect the following types of data:
- **System Logs:** General operational events including boot sequences, firmware updates, and system-level errors.
- **Network Events:** Information regarding physical interface status, VLAN modifications, and Spanning Tree Protocol (STP) changes.
- **Security Logs:** Audit trails for CLI access, SSH logins, and configuration mode entries/exits.
- **Protocol Logs:** Detailed messaging for routing protocols (OSPF, BGP) and Link Layer Discovery Protocol (LLDP) neighbors.
- **Hardware Metrics:** Log-based alerts for hardware components such as transceivers (DOM), power units, and cooling systems.
- **Data Format:** Data is ingested via the standard Syslog protocol in RFC 5424 or RFC 3164 formats, typically transmitted as plaintext over UDP or TCP.

## Compatibility

This integration is compatible with the **ArubaOS-CX (AOS-CX)** switching platform.
- Supported hardware includes the **6000, 6100, 6200, 6300, 6400, 8320, 8325, 8360, and 8400** series switches.
- It is designed for switches running **AOS-CX version 10.x** and higher.

## Scaling and Performance

- **Transport/Collection Considerations:** For high-traffic core switches, TCP is recommended to ensure reliable delivery of logs, though it introduces slightly more overhead on the switch CPU. UDP is preferred for edge switches where performance is critical and occasional log loss is acceptable. Ensure the network MTU is sufficient to prevent fragmentation of large syslog packets.
- **Data Volume Management:** To prevent overwhelming the Elastic Agent, use the `logging severity` command at the source to filter out high-volume `debug` and `info` messages. We recommend starting with `warning` or `notice` levels and only increasing verbosity during active troubleshooting sessions to manage data ingestion costs and storage.
- **Elastic Agent Scaling:** A single Elastic Agent can typically handle syslog streams from dozens of Aruba switches. However, in large campus or data center environments, deploy multiple Agents across different subnets or use a load balancer in front of an Agent group to provide high availability and horizontal scaling for high-volume log ingestion.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have `admin` or `manager` level credentials to access the ArubaOS-CX Command Line Interface (CLI).
- **Network Connectivity:** The switch must have a routable path to the Elastic Agent IP address on the configured Syslog port (defaulting to 514, 9001, or 9002).
- **Firewall Rules:** Ensure any intermediate firewalls or Access Control Lists (ACLs) allow traffic for the chosen protocol (UDP or TCP) and port.
- **System Clock:** It is highly recommended to configure NTP on the Aruba CX switch to ensure accurate timestamps in the ingested logs.
- **Management IP:** Identify the management IP address of the switch to correctly filter and identify sources in Kibana.

## Elastic prerequisites

- **Elastic Agent:** An Elastic Agent must be installed on a host reachable by the switches and enrolled in a policy via Fleet.
- **Integration Installed:** The HPE Aruba CX integration must be added to the Elastic Agent policy in Kibana.
- **Network Access:** The host running the Elastic Agent must have its local firewall configured to allow inbound traffic on the port specified in the integration settings.

## Vendor set up steps

### For CLI-based Collection:

1.  Access the ArubaOS-CX switch via SSH or console and log in with administrative credentials.
2.  Enter the global configuration mode:
    ```shell
    switch# configure
    ```
3.  Add the Elastic Agent as a remote syslog destination using the `logging` command. Replace `<agent_ip>` with the IP address of your Elastic Agent:
    ```shell
    switch(config)# logging <agent_ip>
    ```
4.  (Optional) Specify the severity level of events to be sent. The default is `info`. For production environments, `notice` is often sufficient:
    ```shell
    switch(config)# logging <agent_ip> severity notice
    ```
5.  Specify the VRF if the Agent is reached through a specific management interface:
    ```shell
    switch(config)# logging <agent_ip> vrf mgmt
    ```
6.  Verify the configuration by displaying the logging status:
    ```shell
    switch(config)# show logging config
    ```
7.  Save the running configuration to the startup configuration to ensure it persists after a reboot:
    ```shell
    switch(config)# write memory
    ```

### For GUI-based Collection (Network Operations/Aruba Central):

1.  Log in to the **Aruba Network Operations** app (Aruba Central).
2.  Navigate to the specific switch or group of switches you intend to configure.
3.  Click on **Manage > Device** for the selected AOS-CX switch.
4.  In the configuration sidebar, navigate to **System > Logging**.
5.  Locate the **Logging Servers** table and click the **+ (plus)** icon to add a new server.
6.  Enter the **FQDN or IP address** of the host where the Elastic Agent is running.
7.  Set the **Level** to the desired minimum severity (e.g., `Notice` or `Warning`).
8.  Select the appropriate **VRF** (typically `Management` or `Default`) depending on your network topology.
9.  Click **Apply** and then **Save** to push the configuration to the device.

## Kibana set up steps

### Configure the HPE Aruba CX Integration:

1.  In Kibana, navigate to **Management > Integrations**.
2.  Search for **HPE Aruba CX** and select the integration.
3.  Click **Add HPE Aruba CX**.
4.  Configure the integration settings to match your switch configuration:
    - **Syslog Host:** Set this to `0.0.0.0` to listen on all available interfaces or a specific IP address of the Elastic Agent host.
    - **Syslog Port:** Enter the port number (e.g., `514`).
    - **Protocol:** Select either `udp` or `tcp` to match your switch CLI settings.
5.  Under **Advanced options**, ensure the internal data stream names are left as default unless your organizational standards require specific naming.
6.  Select the **Existing policy** where your Elastic Agent is enrolled.
7.  Click **Save and Continue**, then **Save and Deploy** to push the configuration to the agent.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from the Aruba CX switch to the Elastic Stack.

### 1. Trigger Data Flow on HPE Aruba CX:
- **Generate configuration event:** Enter and exit configuration mode to trigger an audit log:
  ```bash
  configure terminal
  exit
  ```
- **Trigger interface event:** Administratively shut down and then re-enable a non-production interface to generate a state-change log:
  ```bash
  interface 1/1/X
  shutdown
  no shutdown
  exit
  ```
- **Generate authentication event:** Log out of your current SSH session and log back in to generate authentication and session start logs.

### 2. Check Data in Kibana:
1.  Navigate to **Analytics > Discover**.
2.  Select the `logs-*` data view.
3.  Enter the following KQL filter: `data_stream.dataset : "hpe_aruba_cx.log"`
4.  Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
    - `event.dataset` (should be `hpe_aruba_cx.log`)
    - `log.syslog.priority` (verify it matches the severity set on the switch)
    - `service.type` (should be `hpe_aruba_cx`)
    - `hpe_aruba_cx.mnemonic` (or similar vendor-specific code in the message)
    - `message` (containing the raw log payload from the switch)
5.  Navigate to **Analytics > Dashboards** and search for "HPE Aruba CX" to view the pre-built dashboards and confirm visualizations are populating with data.

# Troubleshooting

## Common Configuration Issues

- **Incorrect VRF Assignment**: If logs are not arriving, check if the switch is using the `default` VRF to reach an Agent located on the `management` network. Use `logging <agent_ip> vrf mgmt` if the Agent is connected via the OOBM port.
- **Firewall Blockage**: Ensure that the host running the Elastic Agent allows inbound traffic on the configured port. Use `tcpdump -i any port 514` on the Agent host to verify if packets are reaching the operating system.
- **Severity Filtering**: If specific events (like link up/down) are missing, verify that the `logging severity` level on the switch is not set too high (e.g., set to `error` when the event is `info`).
- **Time Synchronization**: If logs appear in the past or future, ensure both the switch and the Elastic Agent are synchronized via NTP. AOS-CX logs often use local time which can cause ingestion offsets if the integration timezone is not configured.

## Ingestion Errors

- **Parsing Failures**: If logs appear with a `_grokparsefailure` or `_jsonparsefailure` tag, check if the switch is using a non-standard syslog format. The integration expects standard AOS-CX formatting.
- **Format Mismatch**: Aruba switches can sometimes be configured for different logging formats (e.g., CEF). Ensure the switch is using its native syslog output for the best compatibility with the `aruba_cx` package.
- **Field Mapping Issues**: Check the `error.message` field in Kibana Discover for details on why a specific log line could not be processed. This often indicates a change in the log mnemonic or message structure in newer AOS-CX firmware.

## Vendor Resources

- [ArubaOS-CX Switching Platform | FortiSIEM 7.2.1 | Fortinet Document Library](https://docs.fortinet.com/document/fortisiem/7.2.1/external-systems-configuration-guide/259298/arubaos-cx-switching-platform)
- Refer to the official vendor website for additional resources.

## Documentation sites

- [ArubaOS-CX Switching Platform | FortiSIEM 7.2.1 | Fortinet Document Library](https://docs.fortinet.com/document/fortisiem/7.2.1/external-systems-configuration-guide/259298/arubaos-cx-switching-platform)
- Refer to the official vendor website.
