{{- generatedHeader }}
{{/*
This template can be used as a starting point for writing documentation for your new integration. For each section, fill in the details
described in the comments.

Find more detailed documentation guidelines in https://www.elastic.co/docs/extend/integrations/documentation-guidelines
*/}}
# Palo Alto Next-Gen Firewall Integration for Elastic

## Overview
The Palo Alto Next-Gen Firewall integration for Elastic enables you to collect and parse logs from your Palo Alto Networks Next-Generation Firewalls (NGFWs). This integration allows you to monitor network traffic, analyze security threats, and gain insights into your network's performance and security posture. By leveraging the Elastic Stack, you can search, visualize, and alert on your firewall data, making it easier to detect and respond to security incidents.

This integration facilitates the collection of various log types, including traffic, threat, and system logs, providing a comprehensive view of your network activity.

### Compatibility
This integration is compatible with Palo Alto Networks PAN-OS. For specific version compatibility, please refer to the official Palo Alto Networks documentation for log forwarding.

### How it works
This integration works by using Elastic Agent to receive log data from Palo Alto Next-Gen Firewalls. The firewalls can be configured to send logs via syslog (TCP or UDP) to the Elastic Agent, or the agent can be configured to read logs from a file. The agent then processes and forwards the logs to your Elastic deployment, where they are parsed and indexed.

## What data does this integration collect?
The Palo Alto Next-Gen Firewall integration collects the following types of logs:
* **Traffic:** Detailed logs of all traffic passing through the firewall, including source and destination IP addresses, ports, protocols, and applications.
* **Threat:** Logs related to security threats detected by the firewall, such as malware, vulnerabilities, and command-and-control (C2) traffic.
* **System:** Logs related to the health and status of the firewall itself, including configuration changes, system errors, and administrative activities.
* **Configuration:** Logs related to changes in the firewall's configuration.
* **HIP Match:** Logs related to Host Information Profile (HIP) matching, which is used to enforce security policies based on the endpoint's security posture.
* **URL Filtering:** Logs of web traffic that is allowed or blocked by the firewall's URL filtering policies.

### Supported use cases
This integration supports a variety of use cases, including:
* **Security Monitoring:** Detect and respond to security threats by analyzing threat logs and correlating them with other security data.
* **Network Troubleshooting:** Troubleshoot network connectivity issues by analyzing traffic logs and identifying network anomalies.
* **Compliance:** Meet compliance requirements by collecting and archiving firewall logs.
* **Application Monitoring:** Monitor the usage and performance of applications on your network.

## What do I need to use this integration?
To use this integration, you need:
* A running Elastic deployment.
* An installed Elastic Agent.
* A Palo Alto Networks Next-Generation Firewall with a valid license.
* Network connectivity between the firewall and the Elastic Agent.
* The ability to configure log forwarding on your Palo Alto Networks firewall.

## How do I deploy this integration?

### Agent-based deployment

Elastic Agent must be installed. For more details, check the Elastic Agent [installation instructions](docs-content://reference/fleet/install-elastic-agents.md). You can install only one Elastic Agent per host.

Elastic Agent is required to stream data from the syslog or log file receiver and ship the data to Elastic, where the events will then be processed via the integration's ingest pipelines.

### Onboard / configure
To collect logs from your Palo Alto Next-Gen Firewall, you need to configure log forwarding on the firewall to send logs to the Elastic Agent. You can do this by following the instructions in the Palo Alto Networks documentation.

Here are the general steps:
1.  **Configure a Syslog Server Profile:** In the PAN-OS web interface, create a syslog server profile that points to the IP address and port of the Elastic Agent.
2.  **Create a Log Forwarding Profile:** Create a log forwarding profile that uses the syslog server profile you created in the previous step.
3.  **Apply the Log Forwarding Profile to Security Policies:** Apply the log forwarding profile to your security policies to specify which logs should be forwarded to the Elastic Agent.

For detailed instructions, refer to the official Palo Alto Networks documentation: [Configure Log Forwarding](https://docs.paloaltonetworks.com/pan-os/10-2/pan-os-admin/monitoring/configure-log-forwarding)

### Validation
After configuring the integration, you can validate that it is working by checking for data in Kibana. You can do this by navigating to the Discover app and searching for `data_stream.dataset : "panw.panos"`.

## Troubleshooting

For help with Elastic ingest tools, check [Common problems](https://www.elastic.co/docs/troubleshoot/ingest/fleet/common-problems).

If you are not receiving logs, check the following:
*   Ensure that there is network connectivity between the Palo Alto Networks firewall and the Elastic Agent.
*   Verify that the syslog server profile and log forwarding profile are configured correctly on the firewall.
*   Check the Elastic Agent logs for any errors.

## Scaling

For more information on architectures that can be used for scaling this integration, check the [Ingest Architectures](https://www.elastic.co/docs/manage-data/ingest/ingest-reference-architectures) documentation.

## Reference

### panos

The `panos` data stream provides events from Palo Alto Networks Next-Generation Firewalls.

#### panos fields

{{ fields "panos" }}

#### panos sample event

{{ event "panos" }}

### Inputs used
{{ inputDocs }}

### API usage
This integration does not use any APIs to collect data.
