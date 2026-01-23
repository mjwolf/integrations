# Service Info

The Endace integration for Elastic allows organizations to combine high-fidelity network recording with the search and analytics power of the Elastic Stack. Endace specializes in packet capture and network visibility, providing a "single source of truth" for network events. By integrating Endace flow data and packet metadata into Elastic, users can achieve seamless transitions from high-level security alerts to packet-level forensics. The primary advantage of this integration is the addition of an `event.reference` field, which provides a direct, time-synced pivot link to the Endace management UI for deeper investigation.

## Common use cases

- **Security Incident Response:** When a security alert is triggered in Elastic, analysts can use the clickable pivot link in the `event.reference` field to immediately view the raw packet capture in Endace, reducing the Mean Time to Resolution (MTTR).
- **Network Performance Troubleshooting:** Network engineers can monitor flow statistics in Kibana dashboards and, upon identifying latency or packet loss, pivot to Endace Vision to analyze the specific traffic conversations.
- **Threat Hunting and IP Reputation:** Use the integrated IP Reputation lookup to check suspicious IPs against Endace's historical record or external services like VirusTotal and Talos directly from the Elastic Security UI.
- **Forensic Auditing:** Maintain long-term flow records in Elasticsearch while keeping the massive raw packet data on Endace storage, using the integration to bridge the two for compliance and detailed audit trails.

## Data types collected

This integration collects the following types of data:

- **NetFlow Logs (Data Stream: `log`):** Collect NetFlow logs using the netflow input. This stream includes metadata about network conversations such as source and destination IP addresses, port numbers, protocols, byte counts, and packet counts exported from Endace devices.
- **Flows (Data Stream: `flow`):** Track Network Flows generated via the Network Packet Capture (`packet`) input. This is typically used with an Endace vProbe to provide detailed transport-layer statistics, session information, and enriched packet metadata.
- **Enriched Metadata:** Both data streams are enriched with the `event.reference` field, which contains a URL formatted for the Endace UI, including parameters for datasources, tools, and specific time windows.
- **Process Information:** If enabled within the Packet input, network traffic events can be enriched with details about the specific host processes associated with the network connections.

## Compatibility

The **Endace** integration is compatible with the following versions and requirements:
- **Kibana Version:** Requires Kibana version `^8.13.0` or `^9.0.0`.
- **Elastic Subscription:** This integration requires a **Basic** subscription or higher.
- **Operating Systems:** The Elastic Agent can run on **Linux** or **Windows**. 
- **Npcap (Windows Only):** Windows deployments of the Network Packet Capture feature require **Npcap**. The integration provides this library, but users can choose to use an existing installation by setting the **Never Install Npcap on Windows** (`never_install`) option to `true`.
- **Hardware Requirements:** Requires an operational **Endace** appliance running **EndaceOSM** with the **EndaceFlow** application enabled for NetFlow-based collection.

## Scaling and Performance

To ensure optimal performance in high-volume environments, consider the following:
- **Transport/Collection Considerations:** For NetFlow collection, the integration uses the UDP protocol on port **2055** by default. While UDP is faster for flow transmission, ensure the network path between the Endace appliance and the Elastic Agent is low-latency to minimize packet drops. For direct packet capture via vProbe, place the Elastic Agent on a host with high-performance NICs to handle wire-speed metadata generation.
- **Data Volume Management:** Configure the Endace appliance to forward only necessary flow records. Use BPF (Berkeley Packet Filters) to limit capture to specific subnets or high-value protocols. You can further manage volume by adjusting the **View Window Time** (`endace_view_window`) variable to control the granularity of the visualization window sent to the Elastic Stack.
- **Elastic Agent Scaling:** In high-throughput environments (10Gbps+), deploy multiple Elastic Agents behind a network load balancer to distribute the incoming UDP NetFlow traffic. Ensure the host machines have sufficient CPU resources to handle the `packet` input protocol decoding and ECS remapping.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have administrative credentials for the EndaceProbe or vProbe web interface and CLI to configure exporters.
- **Network Visibility:** The Endace appliance must be positioned to see the network traffic you intend to monitor (e.g., via SPAN port or network TAP).
- **License Requirements:** Ensure your Endace device has the necessary licenses for NetFlow generation or vProbe deployment.
- **Connectivity:** Port **UDP 2055** (or your custom configured port) must be open between the Endace appliance and the server hosting the Elastic Agent.
- **URL Configuration:** You must know the base URL of your Endace appliance (e.g., `https://myvprobe.com`) to configure pivot links.

## Elastic prerequisites

- **Elastic Agent:** An Elastic Agent must be installed and enrolled in a Fleet policy.
- **Fleet Management:** Access to the Kibana Fleet UI to add and configure the Endace integration.
- **Connectivity:** The Elastic Agent must have a stable network connection to the Elasticsearch cluster and the Endace data source.
- **Version Requirements:** Ensure the Elastic Stack is running version **8.13.0** or higher.

## Vendor set up steps

### Method 1: Configure Endace to Send NetFlow via Syslog

1. Log in to the administrative interface of your **EndaceProbe**.
2. Navigate to the **Flow Exporter** or **NetFlow Configuration** section (usually under Configuration or External Logging).
3. Select **Add New Collector** or **Add Target**.
4. Set the **Collector IP** to the IP address of the server running the Elastic Agent.
5. Set the **Destination Port** to `2055` (the default port for the integration).
6. Select the **NetFlow Version** (v9 or IPFIX is recommended for better field mapping).
7. Configure the **Active/Inactive Timeout** (standard values are 60s for active and 15s for inactive).
8. Save the configuration and ensure the exporter status is marked as **Enabled**.

### Method 2: Install Elastic Agent on an Endace vProbe

1. Access the command-line interface (CLI) of your **Endace vProbe**.
2. Download the Elastic Agent package compatible with the vProbe's operating system (typically Linux).
3. Enroll the Agent into your Fleet policy using the enrollment token provided in the Kibana Fleet UI.
4. Ensure the `packet` capture driver has the necessary permissions to bind to the monitoring interfaces.
5. In Kibana, navigate to **Fleet > Agents** and verify the Agent appears as **Healthy**.

### Required Post-Ingestion Configuration

To enable the interactive features of the integration, you must perform these steps in **Kibana > Dev Tools**:

1. **Enable Clickable Pivot Links:**
   Run the following command to format the reference field as a URL:
   ```json
   POST kbn:/api/data_views/data_view/logs-*/fields
   {
     "fields": {
       "event.reference": {
         "format": {
           "id": "url"
         }
       }
     }
   }
   ```

2. **Configure IP Reputation Service:**
   Run the following command, replacing `<Your Endace appliance url>` with your actual base URL:
   ```json
   POST kbn:/api/kibana/settings
   {"changes":{"securitySolution:ipReputationLinks": "[ { \"name\": \"Endace\", \"url_template\": \"https://<Your Endace appliance url>/vision2/v1/pivotintovision/?datasources=tag:all&title=Untitled&reltime=12h&sip={{ip}}&tools=conversations_by_ipaddress\" }, { \"name\": \"virustotal.com\", \"url_template\": \"https://www.virustotal.com/gui/search/{{ip}}\" }, { \"name\": \"talosIntelligence.com\", \"url_template\": \"https://talosintelligence.com/reputation_center/lookup?search={{ip}}\" } ]"}}
   ```

## Kibana set up steps

1. In Kibana, navigate to **Integrations** and search for **Endace**.
2. Click **Add Endace**.
3. Configure the global integration variables:
   - **Endace UI URL** (`endace_url`): The base URL for the Endace UI (e.g., `https://myvprobe.com`).
   - **Endace Datasources** (`endace_datasources`): The datasource within Endace. Default: `tag:rotation-file`.
   - **Endace Tools** (`endace_tools`): The tools within Endace for pivoting. Default: `trafficOverTime_by_app,conversations_by_ipaddress`.
   - **View Window Time** (`endace_view_window`): The window of visualization in minutes, centered on the event timestamp. Default: `10`.

4. Configure the specific inputs:

### Collecting Endace Flow logs using the netflow input
- **UDP host to listen on** (`host`): The IP address to listen on. Default: `0.0.0.0`.
- **UDP port to listen on** (`port`): The UDP port to listen on for NetFlow. Default: `2055`.
- **Internal Networks** (`internal_networks`): List of CIDR ranges describing internal IP addresses. Default: `[private]`. This determines `source.locality` and `destination.locality`.

### Collecting network traffic. Use this if using Endace vProbe
- **Interface** (`interface`): The network interface to listen on for network traffic (e.g., `eth0`).
- **GeoIP enrich IP addresses** (`geoip_enrich`): Perform GeoIP enrichment on IP addresses. Default: `True`.
- **Monitor Processes** (`monitor_processes`): Enrich network traffic events with information about the associated process.
- **Map root Packetbeat fields to ECS** (`map_to_ecs`): Remap non-ECS fields to their correct ECS fields. This is recommended to ensure dashboard compatibility.

5. Save the integration to apply the policy to your Elastic Agents.

### Post-Setup: Enable Clickable Pivot Links
To make the `event.reference` field clickable, navigate to **Management > Dev Tools** and execute the following:
```json
POST kbn:/api/data_views/data_view/logs-*/fields
{
    "fields": {
        "event.reference": {
            "format":{
              "id": "url"
            }
        }
    }
}
```

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on Endace:
- **Generate Network Traffic:** From a client on a network segment monitored by the EndaceProbe, generate traffic by visiting a website or using `curl` to reach an external resource.
- **Verify Exporter Status:** Log in to the Endace appliance CLI or UI and check the flow exporter statistics to confirm that packets are being exported to the IP of the Elastic Agent.
- **Authentication event:** Log out and log back into the Endace administration interface to ensure system events are generated.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "endace.log"`
4. Verify logs appear. Expand a log entry and confirm these fields:
   - `event.dataset` (should be `endace.log` or `endace.flow`)
   - `source.ip` and/or `destination.ip`
   - `event.action` or `event.outcome`
   - `message` (the raw log payload)
   - `event.reference` (should contain the formatted pivot URL)
5. Navigate to **Analytics > Dashboards** and search for "Endace" to verify visualizations are populating.

# Troubleshooting

## Common Configuration Issues

- **Pivot Links Not Clickable:** If the `event.reference` field appears as plain text, the Kibana Dev Tools API call to update the data view field format was either not run or failed. Re-run the `POST kbn:/api/data_views/data_view/logs-*/fields` command.
- **Port Conflict:** If the Elastic Agent fails to start the `netflow` input, check if another service is already using UDP port 2055 using `netstat -tuln | grep 2055`.
- **Npcap on Windows:** On Windows systems, the Network Packet Capture component requires Npcap. If flows are not appearing, ensure the bundled Npcap driver was correctly installed or that your manual installation is functional.
- **Firewall Blockage:** Ensure that any intermediate firewalls between the Endace appliance and the Elastic Agent server are permitting UDP traffic on the configured port.

## Ingestion Errors

- **ECS Mapping Issues:** If fields appear with a `_grokparsefailure`, ensure that `map_to_ecs` is set to `true` in the integration settings.
- **Empty Pivot URLs:** If the `event.reference` field contains an incorrect URL, check the `endace_url` variable in the integration configuration; it must include the protocol (e.g., `https://`).
- **Parsing Failures:** Check the `error.message` field in Kibana for any ingestion pipeline errors, which may occur if the NetFlow version sent by Endace does not match the integration's expectations.

## Vendor Resources

- EndaceFlow NetFlow Generator
- EndaceOSM Operating System

## Documentation sites

- [Endace | Elastic integrations](https://www.elastic.co/docs/reference/integrations/endace)
- Refer to the official vendor website for product-specific administration and configuration guides.
