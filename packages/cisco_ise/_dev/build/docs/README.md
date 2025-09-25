# Cisco ISE Integration for Elastic

## Overview

The Cisco ISE integration collects and parses logs from [Cisco Identity Services Engine](https://www.cisco.com/c/en/us/products/security/identity-services-engine/index.html) (ISE). These logs provide detailed insights into network authentications, user and device activities, and policy enforcement, enabling security monitoring and threat detection use cases.

This integration captures logs sent from Cisco ISE via the syslog protocol (TCP or UDP) or from log files.

### Compatibility

This integration has been tested with Cisco ISE server version 3.1.0.518.

### How it works

The integration receives syslog data from a configured Cisco ISE instance over TCP or UDP, or can read logs directly from a file. The Elastic Agent processes these logs, parsing them into structured events that are then sent to Elasticsearch.

## What data does this integration collect?

This integration collects syslog messages from Cisco ISE. For a detailed list and description of all possible syslog messages, refer to the official [Cisco ISE Syslog Messages Reference Guide](https://www.cisco.com/c/en/us/td/docs/security/ise/syslog/Cisco_ISE_Syslogs/m_SyslogsList.html).

### Supported use cases

Integrating Cisco ISE logs with Elastic provides a powerful solution for monitoring network access control and security policies. Key use cases include:
* Real-time monitoring of user and endpoint authentications.
* Detecting and investigating policy violations and potential security threats.
* Auditing and reporting on network access for compliance purposes.
* Correlating ISE data with other sources in Elastic SIEM for comprehensive threat hunting.

## What do I need to use this integration?

Elastic Agent must be installed to stream data from the syslog or log file receiver. For more details, check the Elastic Agent [installation instructions](docs-content://reference/fleet/install-elastic-agents.md). You can install only one Elastic Agent per host.

## How do I deploy this integration?

### Onboard / configure

#### 1. Configure Cisco ISE to send logs

You must configure Cisco ISE to send syslog messages to the Elastic Agent.

1.  Log in to the Cisco ISE Administrator Portal.
2.  Navigate to **Administration** > **System** > **Logging** > **Remote Logging Targets**.
3.  Click **Add**.
4.  In the configuration panel, enter a **Name** for the logging target (e.g., `Elastic-Agent`).
5.  Enter the **IP Address/Hostname** and **Port** of the Elastic Agent that is configured to receive the syslog data.
6.  Select the appropriate **Target Type** (TCP Syslog or UDP Syslog) that matches your integration input configuration in Elastic.
7.  Set the **Maximum Length** to **8192**. This is crucial to prevent message truncation and ensure proper parsing.
8.  Click **Submit** to save the configuration.
9.  Verify that the new target appears in the **Remote Logging Targets** list.

![Cisco ISE server setup image](../img/cisco-ise-setup.png)

#### 2. Add the Cisco ISE integration in Elastic

1.  In Kibana, navigate to **Management** > **Integrations**.
2.  In the search bar, type **Cisco ISE**.
3.  Select the **Cisco ISE** integration and click **Add Cisco ISE**.
4.  Configure the integration with the appropriate input settings (TCP, UDP, or filestream) that match your Cisco ISE configuration.
    *   For TCP/UDP, specify the host and port where the Elastic Agent should listen for incoming syslog data.
5.  Click **Save and continue**.

### Validation

After configuring the integration, you can validate that data is flowing by navigating to the **Discover** tab in Kibana and filtering for `data_stream.dataset : "cisco_ise.log"`. You can also check the pre-built dashboards associated with this integration.

## Troubleshooting

For help with Elastic ingest tools, check [Common problems](https://www.elastic.co/docs/troubleshoot/ingest/fleet/common-problems).

A common issue is the truncation of log messages. To prevent this, it is highly recommended to set the **Maximum Message Length** to **8192** in the Cisco ISE logging target configuration. Some logs from Cisco ISE can be large, and failure to set this value may result in message truncation and parsing errors.

## Scaling

For more information on architectures that can be used for scaling this integration, check the [Ingest Architectures](https://www.elastic.co/docs/manage-data/ingest/ingest-reference-architectures) documentation.

## Reference

### log

The `log` data stream collects all general syslog messages sent from Cisco ISE.

#### log fields

{{ fields "log" }}

#### Example event

{{ event "log" }}

### Inputs used
{{ inputDocs }}
