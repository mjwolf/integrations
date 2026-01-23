# Service Info

The Cisco Aironet integration allows for the ingestion and analysis of logs from Cisco Wireless LAN Controllers (WLC). This integration is essential for network administrators who need to monitor wireless infrastructure health, track client connectivity, and maintain security across their wireless ecosystem.

## Common use cases
- **Network Performance Monitoring:** Analyze system messages to identify access point failures, radio interference, or controller performance bottlenecks that impact wireless client connectivity.
- **Security Auditing and Compliance:** Monitor authentication events, rogue access point detection, and administrative changes to the WLC configuration to ensure compliance with corporate security policies.
- **Client Connectivity Troubleshooting:** Utilize detailed logs to investigate why specific clients are unable to associate, authenticate, or maintain a stable connection to the wireless network.
- **Infrastructure Health Management:** Track system-level events such as software upgrades, reboots, and hardware warnings across the Cisco Aironet and Catalyst 9800 controller fleet.

## Data types collected
This integration collects logs from Cisco Aironet instances via multiple transport methods. All collected logs are mapped to the Elastic Common Schema (ECS) to ensure consistency across your observability data.

- **Cisco Aironet logs (UDP)**: Collect Cisco Aironet logs. This data stream captures standard syslog messages sent over the network using the UDP protocol.
- **Cisco Aironet logs (TCP)**: Collect Cisco Aironet logs. This data stream provides a reliable transport mechanism for system events to ensure no data is lost during network congestion.
- **Cisco Aironet logs (File)**: Collect Cisco Aironet logs from file. This is ideal for scenarios where logs are already being written to a local disk or shared volume on the server where the Elastic Agent is running.

These data streams capture:
- **System Event Logs:** Detailed logs covering controller boot sequences, software updates, and hardware status messages.
- **Access Point Logs:** Individual logs from managed APs, including radio status, client association details, and environment-specific events.
- **Security Logs:** Authentication logs, firewall event notifications, and management access attempts.

## Compatibility
This integration is compatible with **Cisco Wireless LAN Controllers (WLC)** and requires **Kibana version ^8.11.0 or ^9.0.0**.

Supported operating systems and hardware include:
- **Cisco AireOS-based controllers** (e.g., 3504, 5520, 8540 series).
- **Cisco Catalyst 9800 Series Wireless Controllers** running **Cisco IOS XE**.
- Tested versions are not specified, but it supports standard Cisco system message formats used across modern AireOS and IOS XE releases.

## Scaling and Performance

To ensure optimal performance in high-volume wireless environments, consider the following:
- **Transport/Collection Considerations:** This integration supports UDP, TCP, and file-based collection. While **UDP** (default port `9009`) offers the lowest overhead for high-volume environments, packets may be dropped during network congestion. **TCP** (default port `9009`) is recommended for environments requiring guaranteed delivery. For high-throughput controllers, ensure that the Elastic Agent host has sufficient network buffer depth to handle bursts of syslog traffic.
- **Data Volume Management:** Manage ingest volume by configuring the **Syslog Level** on the Cisco WLC. Using **Informational** (level 6) or **Debugging** (level 7) provides the most detail but significantly increases the volume of data stored in Elastic. For most production environments, **Notifications** (level 5) or **Warnings** (level 4) provides an optimal balance between operational visibility and storage overhead.
- **Elastic Agent Scaling:** In environments managing a large number of Access Points, deploy multiple Elastic Agents behind a network load balancer to distribute the incoming syslog traffic. This provides both high availability and horizontal scalability. Ensure the CPU resources on the Agent hosts are monitored, as the regex-based parsing of complex Cisco log strings can be CPU-intensive under heavy load.

# Set Up Instructions

## Vendor prerequisites
- **Administrative Access:** You must have administrative credentials for the Cisco WLC web interface and CLI.
- **Network Connectivity:** Ensure the Cisco WLC and all managed APs have a clear network path to the Elastic Agent. Firewalls must allow traffic on the configured port (default **9009** or the standard syslog port **514**).
- **IP Address Information:** Have the IP address of the machine running the Elastic Agent ready for the syslog target configuration.
- **License Requirements:** No specific additional licenses are required beyond the standard Cisco WLC software capable of syslog forwarding.
- **System Time:** Ensure the WLC and Elastic Agent are NTP-synchronized to ensure accurate timestamping of events.

## Elastic prerequisites
- **Kibana Version:** This integration requires Kibana version **^8.11.0** or **^9.0.0**.
- **Elastic Agent Installation:** An Elastic Agent must be installed and enrolled in a policy via Fleet.
- **Connectivity:** The Elastic Agent must be hosted on a system with a reachable IP address that can accept incoming UDP/TCP traffic from the Cisco WLC on the designated port (default `9009`).

## Vendor set up steps

### For Cisco AireOS-based WLCs:
1. Log in to the Cisco WLC web administration interface.
2. Navigate to the **Management** tab at the top, then select **Logs > Config** from the left-hand sidebar.
3. Locate the **Syslog Server IP Address** field and enter the IP address of the system where the Elastic Agent is running.
4. Click the **Add** button. You may configure up to three syslog servers for redundancy.
5. In the **Syslog Level** dropdown menu, select **Informational** (Level 6) to capture standard operational logs.
6. In the **Syslog Facility** dropdown, select a facility such as **Local7**.
7. Click **Apply** in the top right corner to save the changes and begin streaming logs.

### For Cisco Catalyst 9800 Series WLCs (IOS XE):
1. Log in to the Catalyst 9800 WLC web interface.
2. Navigate to **Troubleshooting > Logs**.
3. Under the **Log Settings** section, ensure the **Syslog** level is set to **Informational** or **Debugging** depending on your visibility requirements.
4. Click the **Manage Syslog Servers** button.
5. In the **IP Configuration** section, click **Add**.
6. Set the **Server Type** to IPv4 or IPv6 and enter the IP address of your Elastic Agent.
7. Click the checkmark icon and then select **Apply to Device**.
8. To include logs from the Access Points, navigate to **Configuration > Tags & Profiles > AP Join**.
9. Select the relevant **AP Join Profile**, go to **Management > Device**, and configure the **System Log** section with the Elastic Agent's IP, desired Trap Value (Level), and Facility.
10. Click **Update & Apply to Device**.

## Kibana set up steps

### Collecting logs from Cisco Aironet via UDP
1. In Kibana, navigate to **Integrations** and search for **Cisco Aironet**.
2. Click **Add Cisco Aironet**.
3. Follow the prompts to add the integration to an existing Elastic Agent policy or create a new one.
4. Configure the following variables for the UDP input:
   - **Listen Address** (`udp_host`): The bind address to listen for UDP connections. Set to `0.0.0.0` to bind to all available interfaces. (Default: `0.0.0.0`)
   - **Listen Port** (`udp_port`): The UDP port number to listen on. (Default: `9009`)
   - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. (Default: `False`)
5. Save and deploy the integration.

### Collecting logs from Cisco Aironet via TCP
1. In Kibana, navigate to **Integrations** and search for **Cisco Aironet**.
2. Click **Add Cisco Aironet**.
3. Follow the prompts to add the integration to an Elastic Agent policy.
4. Configure the following variables for the TCP input:
   - **Listen Address** (`tcp_host`): The bind address to listen for TCP connections. Set to `0.0.0.0` to bind to all available interfaces. (Default: `0.0.0.0`)
   - **Listen Port** (`tcp_port`): The TCP port number to listen on. (Default: `9009`)
   - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. (Default: `False`)
5. Save and deploy the integration.

### Collecting logs from Cisco Aironet via file
1. In Kibana, navigate to **Integrations** and search for **Cisco Aironet**.
2. Click **Add Cisco Aironet**.
3. Follow the prompts to add the integration to an Elastic Agent policy.
4. Configure the following variables for the file input:
   - **Paths** (`paths`): List of paths to the log files that should be monitored. (Default: `['/var/log/cisco-aironet.log']`)
   - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event, added to the field `event.original`. (Default: `False`)
5. Save and deploy the integration.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on Cisco Aironet:
- **Authentication Event:** Attempt to log into the WLC administration interface. A successful login or a deliberate failure will generate security logs.
- **Configuration Event:** Enter configuration mode and update a non-disruptive setting, such as an AP description or a location string, and click **Apply**.
- **Interface/AP Event:** In a test environment, administratively disable and then re-enable a test SSID or a specific radio on an Access Point to trigger state-change logs.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "cisco_aironet.log"`
4. Verify logs appear. Expand a log entry and confirm these fields:
   - `event.dataset` (should be `cisco_aironet.log`)
   - `source.ip` (the IP address of the WLC)
   - `event.outcome` (e.g., `success` or `failure`)
   - `message` (the raw log payload)
5. Navigate to **Analytics > Dashboards** and search for "Cisco Aironet" to verify that the data is populating the pre-built visualizations.

# Troubleshooting

## Common Configuration Issues
- **Port Conflicts**: Ensure that no other service is using the port (default `9009`) on the Elastic Agent host. Use `netstat -tulpn | grep 9009` on Linux to check for listener conflicts.
- **Firewall Restrictions**: If logs are not appearing, check the local firewall on the Elastic Agent host (e.g., `iptables` or `firewalld`) and the network infrastructure to ensure UDP/TCP traffic from the WLC is permitted.
- **Mismatched Protocol**: Ensure the protocol selected in the Kibana input (UDP vs TCP) matches the protocol configured on the Cisco WLC. Note that AireOS often defaults to UDP.
- **Incorrect Syslog Server Entry**: Verify that the "Add" button was clicked after entering the IP address in the AireOS WLC interface; simply typing the IP is often insufficient without applying the "Add" action.

## Ingestion Errors
- **Parsing Failures**: If logs appear in Discover but contain the `_grokparsefailure` or `_jsonparsefailure` tags, the log format may be non-standard. Check if the WLC is using a customized syslog header that deviates from standard RFC formats.
- **Timezone Mismatch**: If logs appear to be "missing" but are actually in the future or far past, check the NTP settings on the WLC. Use the `@timestamp` field in Kibana to verify the ingestion time versus the event time.
- **Identifying Issues**: In Kibana Discover, add the field `error.message` to your columns to quickly see specific reasons for processing failures.

## Vendor Resources
- [Configure Syslog Server on Wireless LAN Controllers - Cisco](https://www.cisco.com/c/en/us/support/docs/wireless/4100-series-wireless-lan-controllers/107252-WLC-Syslog-Server.html)
- [Cisco Catalyst 9800 Series Wireless Controller Software Configuration Guide - Enabling Syslog Messages](https://www.cisco.com/c/en/us/td/docs/wireless/controller/9800/17-17/config-guide/b_wl_17_17_cg/m_syslog_server.html)
- [Cisco WLC System Message Guide](https://www.cisco.com/c/en/us/support/wireless/wireless-lan-controller-software/products-system-message-guides-list.html)
- [Cisco WLC Configuration Best Practices (Syslog)](https://www.cisco.com/c/en/us/td/docs/wireless/controller/technotes/8-6/b_Cisco_Wireless_LAN_Controller_Configuration_Best_Practices.html#ID-1115-000000b2)

## Documentation sites
- [Cisco WLC System Message Guide](https://www.cisco.com/c/en/us/support/wireless/wireless-lan-controller-software/products-system-message-guides-list.html)
- [Configure Syslog Server on Wireless LAN Controllers - Cisco](https://www.cisco.com/c/en/us/support/docs/wireless/4100-series-wireless-lan-controllers/107252-WLC-Syslog-Server.html)
- [Cisco Catalyst 9800 Series Wireless Controller Software Configuration Guide - Enabling Syslog Messages](https://www.cisco.com/c/en/us/td/docs/wireless/controller/9800/17-17/config-guide/b_wl_17_17_cg/m_syslog_server.html)
