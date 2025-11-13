# Cisco FTD Integration for Elastic

## Overview

The Cisco FTD integration for Elastic collects logs from Cisco Firepower Threat Defense (FTD) devices. This integration provides real-time visibility into network traffic, security events, and potential threats by parsing and visualizing FTD logs within the Elastic Stack.

This integration facilitates security monitoring, threat detection, and network troubleshooting by enabling you to:
- Analyze traffic patterns and identify anomalies.
- Detect and respond to security threats like malware, intrusions, and policy violations.
- Maintain compliance with audit trails of network activity.

### Compatibility

This integration is compatible with syslog messages from Cisco Firepower Threat Defense devices. For specific version compatibility, please refer to the official Cisco documentation for your device.

### How it works

The integration works by receiving logs from Cisco FTD devices via syslog (over TCP or UDP) or by reading from a log file. An Elastic Agent collects this data, which is then processed by an ingest pipeline that parses the logs, extracts key fields, and enriches the data, making it ready for analysis in Kibana.

## What data does this integration collect?

The Cisco FTD integration collects logs containing data about network traffic, security events, and system activity. This includes information on connections, intrusions, file and malware events, and system-level messages.

### Supported use cases

Integrating Cisco FTD with the Elastic Stack provides a powerful solution for transforming raw firewall logs into actionable intelligence, enhancing security and operational visibility. This enables advanced use cases such as:
- **Real-time Threat Detection**: Utilize Elastic SIEM to identify and hunt for threats as they occur.
- **Network Traffic Analysis**: Use intuitive Kibana dashboards to analyze network flows, identify top talkers, and monitor bandwidth usage.
- **Security Posture Management**: Gain insights into policy enforcement and identify potential misconfigurations or security gaps.
- **Incident Response**: Accelerate investigation by correlating FTD logs with other data sources within Elastic.

## What do I need to use this integration?

Elastic Agent must be installed on a host that can receive syslog messages or access the log files from your Cisco FTD devices. For more details, check the Elastic Agent [installation instructions](docs-content://reference/fleet/install-elastic-agents.md). You can install only one Elastic Agent per host.

## How do I deploy this integration?

### Agent-based deployment

Elastic Agent is required to stream data from the syslog or log file receiver and ship the data to Elastic, where the events will then be processed via the integration's ingest pipelines.

### Onboard / configure

To collect logs, you must configure your Cisco FTD device to send logs to the Elastic Agent. Configuration steps can vary depending on your specific device model and management platform (Firepower Management Center or Firepower Device Manager).

#### Configure syslog forwarding

1.  Log in to your Firepower Management Center (FMC) or Firepower Device Manager (FDM).
2.  Navigate to the platform settings or device settings where logging and syslog configurations are managed.
3.  Configure a new syslog server or alerting destination.
    *   **Host/IP Address**: Enter the IP address of the server where the Elastic Agent is running.
    *   **Port**: Enter the port the Elastic Agent is configured to listen on (e.g., default `9003`).
    *   **Protocol**: Select TCP or UDP to match the Elastic Agent input configuration.
4.  Specify which event classes or severity levels you want to forward.
5.  Save and deploy the changes.

For detailed, device-specific instructions, refer to the official [Cisco Firepower documentation](https://www.cisco.com/c/en/us/support/security/firepower-management-center/products-installation-and-configuration-guides-list.html).

#### Configure log file collection

If you are collecting logs from a file:
1.  Configure your Cisco FTD device to export logs to a file on a system accessible by the Elastic Agent.
2.  In the integration settings in Fleet, provide the full path to the log file(s) (e.g., `/var/log/cisco-ftd.log`).

### Validation

To validate that the integration is working, navigate to the **Dashboards** section in Kibana and search for "Cisco FTD". If data is being ingested correctly, the dashboards will be populated with information from your FTD logs.

## Troubleshooting

For help with Elastic ingest tools, check [Common problems](https://www.elastic.co/docs/troubleshoot/ingest/fleet/common-problems).

### Handling security fields

Due to the variable nature of sub-fields present under `cisco.ftd.security`, this field is mapped as a [`flattened` datatype](https://www.elastic.co/guide/en/elasticsearch/reference/current/flattened.html). This limits certain operations, such as aggregations, on its sub-fields.

To enable aggregations on specific security fields, a new field, `cisco.ftd.security_event`, has been introduced. You can move fields from `cisco.ftd.security` to `cisco.ftd.security_event` by creating a custom ingest pipeline.

To create a custom pipeline for this purpose:
1. In Kibana, navigate to **Stack Management** > **Ingest Pipelines**.
2. Click **Create Pipeline** > **New Pipeline**.
3. Name the pipeline `logs-cisco_ftd.log@custom`.
4. Add processors to the pipeline. For example, to move `threat_name` from `cisco.ftd.security` to `cisco.ftd.security_event`, add a **Rename** processor:
    - **Field**: `cisco.ftd.security.threat_name`
    - **Target field**: `cisco.ftd.security_event.threat_name`
5. You can also add a **Convert** processor if you need to change the data type of the renamed field.

This `@custom` pipeline is automatically applied at the end of the default processing pipeline, allowing you to perform aggregations on the newly structured fields under `cisco.ftd.security_event`.

## Scaling

For more information on architectures that can be used for scaling this integration, check the [Ingest Architectures](https://www.elastic.co/docs/manage-data/ingest/ingest-reference-architectures) documentation.

## Reference

### log

The `log` data stream collects logs from Cisco Firepower Threat Defense (FTD) devices.

#### log fields

{{ fields "log" }}

#### Sample Event

{{ event "log" }}

### Inputs used
{{ inputDocs }}
