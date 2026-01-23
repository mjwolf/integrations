# Service Info

## Common use cases

The Arista NG Firewall integration provides comprehensive visibility into network traffic, security events, and system performance by ingesting logs from Arista (formerly Untangle) devices into the Elastic Stack. This enables organizations to maintain a robust security posture and optimize network operations.

-   **Security Monitoring and Threat Detection:** Monitor Intrusion Prevention System (IPS) logs and Firewall events to identify and respond to unauthorized access attempts or malicious network activity in real-time.
-   **Web Traffic Analysis:** Utilize Web Filter and HTTP Request/Response events to audit user activity, ensure compliance with corporate browsing policies, and identify high-bandwidth web applications.
-   **Network Performance Troubleshooting:** Track Interface and Session Statistics to identify bottlenecks, monitor bandwidth consumption per interface, and diagnose connectivity issues across the network fabric.
-   **Administrative Auditing:** Capture Admin Login events and configuration changes to maintain a clear audit trail of who accessed the firewall management interface and what modifications were made.

## Data types collected

This integration collects several categories of network and security data to provide a holistic view of your firewall's operations. Based on the data stream configuration, the following streams are collected:

-   **Arista NG Firewall logs (log):** This data stream handles the collection of all firewall-related events via the `tcp` or `udp` input. It is used to **Collect Arista NG Firewall logs via TCP** or **Collect Arista NG Firewall logs via UDP**. It supports the following event types:
    *   **Firewall Event:** Detailed records of traffic filtering decisions. This includes specific data on packets allowed or blocked based on defined security policies, access control rules, and source/destination metadata.
    *   **Intrusion Prevention System (IPS) Logs:** Alerts and signature matches identifying potential network attacks, exploit attempts, or anomalies detected by the IDS/IPS engine.
    *   **Web Activity Logs:** Comprehensive records of HTTP requests and responses, including requested URLs, categories (via Web Filter), and specific web filter actions taken against user traffic.
    *   **Admin Login Events:** Audit logs for administrative access, capturing login attempts, successful authentications, and management activities within the Arista interface.
    *   **Session Information:** Metadata regarding active and closed network sessions, providing visibility into source/destination IP addresses, ports, and protocol usage.
    *   **System Metrics:** Operational data such as interface statistics, CPU/memory resource utilization, and general system health indicators.

## Compatibility

-   **Arista Version:** This integration supports Arista NG Firewall (previously known as Untangle NG Firewall) versions that support standard Syslog output via the **Events** management interface.
-   **Elastic Stack:** Requires Kibana version `^8.11.0` or `^9.0.0`.
-   **Subscription:** This integration is available with a **Basic** Elastic subscription.

## Scaling and Performance

To ensure optimal performance in high-volume network environments, consider the following:

-   **Transport/Collection Considerations:** This integration supports both **TCP** and **UDP** protocols. While UDP is faster for syslog transmission and provides lower overhead for high-throughput environments, **TCP** is recommended for environments where log delivery guarantees are required. TCP ensures no log messages are lost due to network congestion or packet loss, though it introduces slightly higher overhead.
-   **Data Volume Management:** To manage data volume and reduce load on the Elastic Stack, configure **Syslog Rules** on the Arista appliance to forward only necessary events. Avoid forwarding debug-level logs or non-critical system notifications at the source to significantly reduce the ingest rate and storage requirements.
-   **Elastic Agent Scaling:** For high-throughput environments, deploy multiple Elastic Agents behind a network load balancer to distribute the incoming syslog traffic evenly. Place Agents as close to the data source as possible to minimize network latency and ensure consistent collection during peak traffic periods.

# Set Up Instructions

## Vendor prerequisites

1.  **Administrative Access:** You must have administrative credentials to access the Arista NG Firewall web interface.
2.  **Network Connectivity:** The Arista device must be able to reach the Elastic Agent host over the network on the configured port (default **9010**).
3.  **Firewall Rules:** Ensure that any internal firewalls or Access Control Lists (ACLs) allow traffic from the Arista NG Firewall IP address to the Elastic Agent IP on the specified port.
4.  **Platform Feature:** The **Events** module must be accessible within the Arista configuration to define syslog output rules.

## Elastic prerequisites

1.  **Elastic Agent:** An Elastic Agent must be installed and successfully enrolled in a policy via Fleet.
2.  **Network Listener:** The Elastic Agent must be hosted on a system that can listen for incoming network connections on the ports specified in the integration configuration (default **9010**).
3.  **Stack Version:** Ensure you are running a compatible version of the Elastic Stack (Elasticsearch and Kibana version `8.11.0` or higher).

## Vendor set up steps

Arista NG Firewall supports forwarding logs via the standard Syslog protocol. Follow these steps to configure the remote syslog target:

1.  Log in to your Arista NG Firewall administrative interface.
2.  Navigate to the **Config** tab on the top menu bar.
3.  From the left-hand navigation pane, select **Events**.
4.  Click on the **Syslog** tab to access the remote syslog configuration settings.
5.  Check the box next to **Enable remote syslog** to activate the syslog client.
6.  In the **Host** field, enter the IP address or DNS hostname of the server where the Elastic Agent is running.
7.  In the **Port** field, enter the port number that the Elastic Agent is configured to listen on (default is `9010`).
8.  In the **Protocol** dropdown menu, select either `UDP` or `TCP` to match the input type you intend to configure in Kibana.
9.  Navigate to the **Syslog Rules** sub-tab within the Events section.
10. Click the **Add** button to create a new rule:
    *   Ensure **Enable** is selected.
    *   Set **Class** to **All** (or select specific classes like Firewall, IPS, or Web Filter).
    *   Leave **Conditions** blank to capture all events.
    *   Ensure the **Remote Syslog** checkbox at the bottom of the rule window is checked.
11. Click **Done** and then click **Save** at the bottom right to apply the changes.

## Kibana set up steps

In Kibana, add the Arista NG Firewall integration to an Elastic Agent policy. You must configure the input types based on your vendor setup.

### Collects logs from Arista NG Firewall via TCP
Use this input to collect Arista NG Firewall logs via TCP.

1.  In Kibana, navigate to **Management > Integrations**.
2.  Search for and select **Arista NG Firewall**.
3.  Click **Add Arista NG Firewall**.
4.  Under the **Arista NG Firewall logs** section, locate the **Collects logs from Arista NG Firewall via TCP** input.
5.  Configure the following variables:
    *   **TCP host to listen on** (`tcp_host`): Enter the interface address the agent should bind to. Default: `localhost`. Use `0.0.0.0` to listen on all available network interfaces.
    *   **TCP Port to listen on** (`tcp_port`): Specify the port number to receive syslog data. Default: `9010`.
    *   **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
6.  Follow the prompts to save the integration and deploy it to an Elastic Agent policy.

### Collects logs from Arista NG Firewall via UDP
Use this input to collect Arista NG Firewall logs via UDP.

1.  In Kibana, navigate to **Management > Integrations**.
2.  Search for and select **Arista NG Firewall**.
3.  Click **Add Arista NG Firewall**.
4.  Under the **Arista NG Firewall logs** section, locate the **Collects logs from Arista NG Firewall via UDP** input.
5.  Configure the following variables:
    *   **UDP host to listen on** (`udp_host`): Enter the interface address the agent should bind to. Default: `localhost`. Use `0.0.0.0` to listen on all available network interfaces.
    *   **UDP Port to listen on** (`udp_port`): Specify the port number to receive syslog data. Default: `9010`.
    *   **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. Default: `False`.
6.  Follow the prompts to save the integration and deploy it to an Elastic Agent policy.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on Arista NG Firewall:
-   **Authentication event:** Log out of the Arista NG Firewall administrative interface and log back in to trigger an "Admin Login Event".
-   **Web traffic event:** From a client machine behind the firewall, browse to several websites to generate "HTTP Request Event" and "Web Filter Event" logs.
-   **Firewall/Configuration event:** Navigate to the Arista configuration, modify a setting (such as a rule description), and click **Save** to generate system and configuration events.
-   **Interface event:** If possible, briefly toggle a non-critical interface to generate interface statistics and state change logs.

### 2. Check Data in Kibana:
1.  Navigate to **Analytics > Discover**.
2.  Select the `logs-*` data view.
3.  Enter the KQL filter: `data_stream.dataset : "arista_ngfw.log"`
4.  Verify logs appear in the timeline. Expand a log entry and confirm the presence of these fields:
    *   `event.dataset` (should match `arista_ngfw.log`)
    *   `source.ip` and/or `destination.ip` (to confirm network traffic identification)
    *   `event.action` or `event.outcome` (e.g., allowed, blocked, or login)
    *   `message` (the raw log payload)
5.  Navigate to **Analytics > Dashboards** and search for "Arista NG Firewall" to verify that the pre-built visualizations are populating with data.

# Troubleshooting

## Common Configuration Issues

-   **Port Conflict**: If the Elastic Agent fails to start the syslog listener, check if another process is using the configured port (e.g., 9010). Use commands like `netstat -ano | grep 9010` to identify conflicting services.
-   **Network Security Groups/Firewalls**: If no data is appearing in Kibana, verify that the host running the Elastic Agent has its local firewall (e.g., iptables, firewalld, or Windows Firewall) configured to allow inbound traffic on the selected TCP/UDP port.
-   **Syslog Rule Not Enabled**: In the Arista interface, ensure that the specific "Syslog Rule" created has the "Remote Syslog" checkbox checked. If this is unchecked, events will be logged locally but not forwarded to the Agent.
-   **Incorrect Hostname/IP**: Double-check that the "Host" field in the Arista Syslog settings points exactly to the IP address of the Elastic Agent. Avoid using loopback addresses (127.0.0.1) on the Arista side.

## Ingestion Errors

-   **Parsing Failures**: If logs appear in Kibana but contain the tag `_parsing_failure`, the syslog format from the Arista device might not match the expected pattern. Ensure that the Arista device is using standard syslog formatting and that no custom log prefixes have been added.
-   **Time Synchronization**: If logs appear with incorrect timestamps, verify that both the Arista NG Firewall and the Elastic Agent host are synchronized via NTP. Large time drifts can cause events to appear outside of the default Discover time range.
-   **Identify via Error Fields**: Check for the `error.message` field in Discover to see specific processing errors encountered by the ingest pipeline.

## Vendor Resources

-   [Arista NG Firewall Events Configuration](https://wiki.edge.arista.com/index.php/Events)
-   Arista NG Firewall Product Page

## Documentation sites

-   [Arista NG Firewall Events Configuration](https://wiki.edge.arista.com/index.php/Events)
-   Refer to the official vendor website for additional resources.
