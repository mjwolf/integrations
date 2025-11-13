{{- generatedHeader }}
# Cisco FTD Integration for Elastic

## Overview

Use the Cisco Firepower Threat Defense (FTD) integration to collect logs from your Cisco FTD devices. This integration enables comprehensive monitoring, threat detection, and security analysis within the Elastic Stack. It parses syslog messages from Cisco FTD, giving you real-time visibility into network traffic, security events, and system activity. By centralizing these logs, you can enhance your security posture, streamline incident response, and gain deep insights into your network's operations.

### Compatibility

This integration is compatible with Cisco FTD devices that support syslog export. It requires Elastic Stack version 8.11.0 or newer.

### How it works

The integration receives syslog data sent from a Cisco FTD device. Configure the Elastic Agent to listen for these logs on a specific TCP or UDP port, or to read them from a log file. The agent then processes and parses the logs before sending them to Elasticsearch.

## Data collection

The Cisco FTD integration collects logs containing detailed information about:
*   **Connection Events**: Firewall traffic, network address translation (NAT), and connection summaries.
*   **Security Events**: Intrusion detection and prevention (IPS/IDS) alerts, file and malware protection events, and security intelligence data.
*   **System Events**: Device health, system status, and configuration changes.

### Use cases

- **Real-time Threat Detection**: Use Elastic SIEM to identify and respond to threats like malware, intrusions, and policy violations.
- **Network Traffic Analysis**: Visualize and analyze network traffic patterns to identify anomalies, troubleshoot connectivity issues, and optimize performance.
- **Security Auditing and Compliance**: Maintain a searchable archive of all firewall activity to support compliance requirements and forensic investigations.
- **Operational Monitoring**: Track the health and status of your FTD devices to ensure they are functioning correctly.

## Requirements

You need an Elastic Agent installed on a host that is reachable by your Cisco FTD device over the network. For more details, refer to the Elastic Agent [installation instructions](docs-content://reference/fleet/install-elastic-agents.md). You can install only one Elastic Agent per host.

The Elastic Agent streams data from the syslog or log file receiver to Elastic, where the integration's ingest pipelines process the events.

## Setup

### 1. Configure Cisco FTD to send syslog data

Configure your Cisco FTD device to forward syslog messages to the Elastic Agent. The specific steps may vary depending on whether you are using Firepower Device Manager (FDM) or Firepower Management Center (FMC).

1.  **Define the syslog server**:
    *   In your FDM or FMC interface, navigate to the syslog configuration section (for example, **Objects > Syslog Servers** or **Device > System Settings > Logging**).
    *   Add a new syslog server. Provide the IP address and port of the machine where the Elastic Agent is running.
    *   Ensure the protocol (TCP or UDP) matches the input you configure in the integration.

2.  **Configure logging rules**:
    *   Create or edit a logging rule to send specific event classes to the new syslog server.
    *   To ensure comprehensive data collection, send all relevant message IDs.

3.  **Deploy changes**:
    *   Save and deploy your configuration changes to the FTD device.

For detailed instructions, refer to the official Cisco documentation, such as [Configure Logging on FTD via FMC](https://www.cisco.com/c/en/us/support/docs/security/firepower-ngfw/200479-Configure-Logging-on-FTD-via-FMC.html).

### 2. Add the Cisco FTD integration in Elastic

1.  In Kibana, go to **Management > Integrations**.
2.  In the search bar, enter **Cisco FTD**.
3.  Click the integration, then click **Add integration**.
4.  Configure the integration settings. Select the input method that matches your Cisco FTD configuration (TCP, UDP, or log file).
    *   **For TCP/UDP**: Specify the `host` and `port` where the Elastic Agent should listen. This destination must match the configuration on your FTD device.
    *   **For Log File**: Provide the file `paths` for the agent to monitor.
5.  Click **Save and continue** to add the integration policy to an Elastic Agent.

### 3. Verify data is flowing

To validate the integration is working, go to the **Discover** tab in Kibana. Filter for the `cisco_ftd.log` dataset (`data_stream.dataset : "cisco_ftd.log"`) and verify that logs from your FTD device are being ingested. You can also check the pre-built dashboards for this integration by searching for "Cisco FTD" in the **Dashboards** section.

## Troubleshooting

For help with ingest tools, refer to [Common problems](https://www.elastic.co/docs/troubleshoot/ingest/fleet/common-problems).

The `cisco.ftd.security` field uses the [`flattened` datatype](https://www.elastic.co/guide/en/elasticsearch/reference/current/flattened.html) to handle a variable number of sub-fields. This limits your ability to run aggregations on its sub-fields.

To enable aggregations on common security-related fields, the integration automatically moves a known set of fields from `cisco.ftd.security` to `cisco.ftd.security_event`. If you need to run aggregations on additional fields within `cisco.ftd.security`, create a custom ingest pipeline to move them.

To create this pipeline:
1.  In Kibana, go to **Stack Management > Ingest Pipelines**.
2.  Click **Create Pipeline > New Pipeline**.
3.  Set the **Name** to `logs-cisco_ftd.log@custom`.
4.  Add a **Rename** processor:
    *   Set **Field** to the source field (for example, `cisco.ftd.security.threat_name`).
    *   Set **Target field** to the destination (for example, `cisco.ftd.security_event.threat_name`).
5.  Add more processors as needed and save the pipeline. This `@custom` pipeline is automatically applied to all incoming Cisco FTD logs.

## Scaling

For scaling guidance, refer to the [Ingest Architectures](https://www.elastic.co/docs/manage-data/ingest/ingest-reference-architectures) documentation.

## Reference

### log

The `log` data stream collects logs from Cisco Firepower Threat Defense (FTD) devices.

#### log fields

{{ fields "log" }}

#### log sample event

{{ event "log" }}


### Inputs used
{{ inputDocs }}
