# abuse.ch Integration for Elastic

> **Note**: This documentation was generated using AI and should be reviewed for accuracy.

## Overview

The `abuse.ch` integration for Elastic lets you collect actionable, community-driven threat intelligence from [abuse.ch](https://abuse.ch/). By ingesting these feeds, you can identify, track, and mitigate malware and botnet-related cyber threats within your environment. This integration provides visibility into malicious URLs, malware payloads, SSL certificates, and threat indicators, allowing for enhanced threat detection and event enrichment.

This integration facilitates:
- Detection of malicious activity by matching your logs against known threat indicators
- Enrichment of security events with context from global threat intelligence feeds
- Proactive defense against malware payloads and botnet infrastructure
- Identification of high-confidence threats like malicious URLs and blacklisted SSL certificates
- Security research using JA3 fingerprints to detect encrypted malicious communication

### Compatibility

This integration is compatible with the `v1` version of the `abuse.ch` `URLhaus`, `MalwareBazaar`, `ThreatFox`, and `SSLBL` APIs. It's designed to work with Elastic Stack version 8.0.0 or higher.

### How it works

This integration periodically queries the `abuse.ch` APIs to retrieve the latest threat intelligence indicators. It uses the `httpjson` input to fetch data from various data streams, including `ja3_fingerprints`, `malware`, `malwarebazaar`, `sslblacklist`, `threatfox`, and `url`. Once the data's retrieved, it's processed and mapped to the Elastic Common Schema (ECS), making it ready for use in dashboards, alerts, and security analytics. The Elastic Agent runs the collection process based on the interval you configure, ensuring your threat intelligence remains up to date.

## What data does this integration collect?

The abuse.ch integration collects threat intelligence indicators into the following datasets:
* `ja3_fingerprints`: Collects JA3 fingerprint based threat indicators identified by SSLBL via the [SSLBL API endpoint](https://sslbl.abuse.ch/blacklist/ja3_fingerprints.csv).
* `malware`: Collects malware payloads from URLs tracked by URLhaus via the [URLhaus Bulk API](https://urlhaus-api.abuse.ch/#payloads-recent).
* `malwarebazaar`: Collects malware payloads from MalwareBazaar via the [MalwareBazaar API](https://bazaar.abuse.ch/api/#latest_additions).
* `sslblacklist`: Collects SSL certificate based threat indicators denylisted on SSLBL via the [SSLBL API endpoint](https://sslbl.abuse.ch/blacklist/sslblacklist.csv).
* `threatfox`: Collects threat indicators from ThreatFox via the [ThreatFox API](https://threatfox.abuse.ch/api/#recent-iocs).
* `url`: Collects malware URL based threat indicators from URLhaus via the [URLhaus API](https://urlhaus.abuse.ch/api/#csv).

### Supported use cases

Integrating abuse.ch threat intelligence with Elastic Security provides a powerful solution for proactive defense and visibility into malicious activity. Key use cases include:
* Real-time threat detection: You can use Elastic Security to automatically generate alerts when Indicators of Compromise (IoCs) like malicious [IPs](https://www.elastic.co/docs/reference/security/prebuilt-rules/rules/threat_intel/threat_intel_indicator_match_address), [domains](https://www.elastic.co/docs/reference/security/prebuilt-rules/rules/threat_intel/threat_intel_indicator_match_url), or [hashes](https://www.elastic.co/docs/reference/security/prebuilt-rules/rules/threat_intel/threat_intel_indicator_match_hash) match your internal event or alert data.
* Threat hunting: You can query indicators across your environment to identify historical evidence of compromise or emerging threats.
* Alert enrichment: You can add context to existing security alerts by correlating them with known malicious actors and malware families identified by abuse.ch.
* Security visualization: You can power dashboards to track the prevalence and types of known threats relevant to your infrastructure.

## What do I need to use this integration?

### From Elastic

This integration requires the following Elastic components:
- Elastic Stack version 8.11.0 or higher.
- This integration installs [Elastic latest transforms](https://www.elastic.co/docs/explore-analyze/transforms/transform-overview#latest-transform-overview). For more details, see the [Transform](https://www.elastic.co/docs/explore-analyze/transforms/transform-setup) setup and requirements.

### From abuse.ch

To use this integration, you'll need an account and authentication credentials from abuse.ch. The integration requires an `Auth Key` (API key) for request authentication. Any requests you make without this key will be rejected by the abuse.ch APIs.

Your Elastic Agent host must have outbound HTTPS (port `443`) access to the abuse.ch API endpoints.

### Obtain an abuse.ch auth key

To get your API key, follow these steps:
1. Sign up for a new account, or log in to the [abuse.ch authentication portal](https://auth.abuse.ch).
2. Connect with at least one authentication provider: Google, GitHub, X, or LinkedIn.
3. Select **Save profile**.
4. In the **Optional** section, click the **Generate Key** button to generate your **Auth Key**.
5. Copy the generated **Auth Key**.

For more details, see the abuse.ch [Community First - New Authentication](https://abuse.ch/blog/community-first/) blog.

## How do I deploy this integration?

### Agent-based deployment

Elastic Agent must be installed. For more details, check the Elastic Agent [installation instructions](https://www.elastic.co/guide/en/fleet/current/elastic-agent-installation.html). You can install only one Elastic Agent per host.

Elastic Agent is required to pull data from the abuse.ch APIs and ship it to Elastic, where the events will then be processed via the integration's ingest pipelines.

### Agentless deployment

Agentless deployments are only supported in Elastic Serverless and Elastic Cloud environments. Agentless deployments provide a means to ingest data while avoiding the orchestration, management, and maintenance needs associated with standard ingest infrastructure. Using an agentless deployment makes manual agent deployment unnecessary, allowing you to focus on your data instead of the agent that collects it.

For more information, refer to [Agentless integrations](https://www.elastic.co/guide/en/serverless/current/security-agentless-integrations.html) and [Agentless integrations FAQ](https://www.elastic.co/guide/en/serverless/current/agentless-integration-troubleshooting.html).

### Set up steps in abuse.ch

To use this integration, you need an authentication key from abuse.ch.

1.  Visit the [abuse.ch website](https://abuse.ch/) to understand the different projects: URLhaus, MalwareBazaar, ThreatFox, and the SSL blocklist.
2.  Obtain an Auth Key by following the instructions provided in the [abuse.ch blog](https://abuse.ch/blog/community-first/). This key is required for all API requests made by the integration.

#### Vendor resources

- [abuse.ch - Community First (Auth Key Info)](https://abuse.ch/blog/community-first/)
- [MalwareBazaar API Documentation](https://mb-api.abuse.ch/api/v1/)
- [ThreatFox API Documentation](https://threatfox-api.abuse.ch/api/v1/)
- [URLhaus API Documentation](https://urlhaus-api.abuse.ch/api/v1/)

### Set up steps in Kibana

To configure the integration in Kibana, follow these steps:

1.  In Kibana, navigate to **Management > Integrations**.
2.  Search for **abuse.ch** and select the integration.
3.  Click **Add abuse.ch**.
4.  In the **Auth Key** field, enter your abuse.ch API key (replace the placeholder with your actual value).
5.  Select the data streams you want to enable (JA3 Fingerprints, Malware payloads, MalwareBazaar payloads, SSL Certificates, ThreatFox indicators, or Malware URLs).
6.  Configure the settings for each enabled data stream.

The following common settings are available for the data streams:

| Setting | Description |
|---|---|
| **URL** | The base URL of the specific abuse.ch API (e.g., `https://mb-api.abuse.ch/api/v1/`). |
| **Interval** | How often the agent polls the API for new data (e.g., `10m`, `24h`). |
| **IOC Expiration Duration** | The time after which an indicator is considered expired (default `90d`). |
| **Preserve original event** | If enabled, the raw API response is stored in the `event.original` field. |

Under **Advanced Options**, you can configure the following optional parameters:

| Setting | Description |
|---|---|
| **HTTP Client Timeout** | Duration before the connection times out (e.g., `30s`). |
| **Proxy URL** | The URL of a proxy server if your environment requires one for outgoing traffic. |
| **SSL Configuration** | Custom SSL settings for the HTTP client. See the [SSL documentation](https://www.elastic.co/guide/en/beats/filebeat/current/configuration-ssl.html#ssl-common-config) for details. |
| **Processors** | Custom processors to transform or filter data before it is sent to Elasticsearch. |
| **Enable request tracing** | **Warning:** Enabling this logs full requests/responses for debugging and can compromise security. Only use this for temporary troubleshooting. |

After you've configured the inputs, click **Save and continue**.

### Validation

To verify the integration is working and data is flowing, follow these steps:

1.  Navigate to **Management > Fleet > Agents** and confirm that the Elastic Agent status is **Healthy**.
2.  In Kibana, navigate to **Analytics > Discover**.
3.  Filter for the abuse.ch dataset by entering the following in the search bar: `data_stream.dataset : "ti_abusech.*"`.
4.  Confirm that documents are appearing with recent timestamps and that fields are correctly populated.
5.  Navigate to **Management > Stack Management > Data > Transforms** and search for "abuse.ch". Verify that all associated transforms are in a **Healthy** state.
6.  Navigate to **Analytics > Dashboards** and search for "abuse.ch". Open one of the pre-built dashboards, such as the **abuse.ch Overview** or **abuse.ch URLs**, to view your threat intelligence data.

## Troubleshooting

For help with Elastic ingest tools, check [Common problems](https://www.elastic.co/docs/troubleshoot/ingest/fleet/common-problems).

### Common configuration issues

You may encounter the following issues when configuring or using the abuse.ch integration:
- Authentication failures in the portal: When creating the Auth Key inside the [abuse.ch authentication portal](https://auth.abuse.ch/), make sure you connect at least one additional authentication provider to ensure seamless access to the platform.
- Missing data or ingestion errors: Check for captured ingestion errors inside Kibana. Ingestion errors, including API errors, are captured into the `error.message` field.
    1. Navigate to Analytics > Discover.
    2. In Search field names, search and add fields `error.message` and `data_stream.dataset` into the Discover view.
    3. Search for the dataset(s) enabled by this integration using a KQL query like `data_stream.dataset: ti_abusech.*`.
    4. Search for errors using the KQL query `error.message: *`. You can combine queries, for example: `data_stream.dataset: ti_abusech.url AND error.message: *`.
- API returns 403 Forbidden: abuse.ch APIs return this status when the Auth Key is invalid. The `error.message` field will show `query_status: unknown_auth_key` and `error.id` will show `403 Forbidden`. To fix this, regenerate the Auth Key in the [abuse.ch authentication portal](https://auth.abuse.ch/) and update the integration policy with the new key.
- API returns 500 Internal Server Error: This occurs when there is a problem with the abuse.ch service. The `error.message` field will show `POST:500 Internal Server Error (500)`. This is typically a temporary issue and ingestion should resume normally in the next request.
- Duplicate indicators appearing in search: Since this integration supports the expiration of Indicators of Compromise (IoCs) using an Elastic transform, threat indicators are present in both source and destination indices. This is an implementation detail necessary for properly expiring threat indicators.
- Increased storage usage: Because the latest copy of threat indicators is indexed in two places (source and destination indices), you should plan for additional storage requirements. You can tune ILM policies on source indices to manage their data retention period.

### Vendor resources

You can find additional information and support using these resources:
- [abuse.ch Authentication Portal](https://auth.abuse.ch/)
- [URLhaus Documentation](https://urlhaus.abuse.ch/api/)
- [MalwareBazaar Documentation](https://malwarebazaar.abuse.ch/api/)
- [ThreatFox Documentation](https://threatfox.abuse.ch/api/)

## Performance and scaling

For more information on architectures that can be used for scaling this integration, check the [Ingest Architectures](https://www.elastic.co/docs/manage-data/ingest/ingest-reference-architectures) documentation.

To ensure optimal performance and reliability when ingesting threat intelligence from abuse.ch at scale, you should consider the following:

*   Interval tuning: You can adjust the `interval` setting for each data stream to balance your need for fresh data against the resource usage of the Elastic Agent and the API capacity of the provider. For example, the MalwareBazaar payloads data stream defaults to `10m`. If you don't require updates this frequently, increasing this value can reduce processing overhead.
*   Memory and CPU allocation: The processing of large JSON responses from threat intelligence feeds can be memory-intensive. You should monitor the resource usage of your Elastic Agents and ensure they have sufficient allocations, especially if you're running multiple data streams or managing a high volume of indicators.
*   Data volume management: The `preserve_original_event` setting is disabled by default. Enabling this will store a raw copy of every event in the `event.original` field, which significantly increases the storage requirements in Elasticsearch. It's recommended to leave this disabled unless you have specific compliance or debugging needs.
*   Indicator expiration: The `ioc_expiration_duration` setting determines how long an indicator remains active before being considered expired. Managing this duration helps you control the size of your active threat intelligence indices and ensures that your detection engines aren't overwhelmed by stale data.

## Reference

### Inputs used

You can use the following inputs with this integration:
{{ inputDocs }}

### API usage

The datasets in this integration use these APIs to collect data:
*   `ja3_fingerprints`: [SSLBL API](https://sslbl.abuse.ch/blacklist/ja3_fingerprints.csv)
*   `malware`: [URLhaus Bulk API](https://urlhaus-api.abuse.ch/#payloads-recent)
*   `malwarebazaar`: [MalwareBazaar API](https://bazaar.abuse.ch/api/#latest_additions)
*   `sslblacklist`: [SSLBL API](https://sslbl.abuse.ch/blacklist/sslblacklist.csv)
*   `threatfox`: [ThreatFox API](https://threatfox.abuse.ch/api/#recent-iocs)
*   `url`: [URLhaus API](https://urlhaus.abuse.ch/api/#csv)

### Data streams

#### ja3_fingerprints

The `ja3_fingerprints` data stream provides events from the abuse.ch SSLBL of the following types: JA3 fingerprints and threat indicators.

##### ja3_fingerprints fields

{{ fields "ja3_fingerprints" }}

##### ja3_fingerprints sample event

{{ event "ja3_fingerprints" }}

#### malware

The `malware` data stream provides events from the abuse.ch URLhaus of the following types: malware payloads and threat indicators.

##### malware fields

{{ fields "malware" }}

##### malware sample event

{{ event "malware" }}

#### malwarebazaar

The `malwarebazaar` data stream provides events from the abuse.ch MalwareBazaar of the following types: malware payloads and threat indicators.

##### malwarebazaar fields

{{ fields "malwarebazaar" }}

##### malwarebazaar sample event

{{ event "malwarebazaar" }}

#### sslblacklist

The `sslblacklist` data stream provides events from the abuse.ch SSLBL of the following types: blacklisted SSL certificates and threat indicators.

##### sslblacklist fields

{{ fields "sslblacklist" }}

##### sslblacklist sample event

{{ event "sslblacklist" }}

#### threatfox

The `threatfox` data stream provides events from the abuse.ch ThreatFox of the following types: indicators of compromise and threat indicators.

##### threatfox fields

{{ fields "threatfox" }}

##### threatfox sample event

{{ event "threatfox" }}

#### url

The `url` data stream provides events from the abuse.ch URLhaus of the following types: malware URLs and threat indicators.

##### url fields

{{ fields "url" }}

##### url sample event

{{ event "url" }}

### Expiration of indicators of compromise (IOCs)

All abuse.ch datasets support indicator expiration. For the `url` dataset, the integration ingests a full list of active threat indicators during every polling interval. For other datasets—specifically `malware`, `malwarebazaar`, and `threatfox`—the integration expires threat indicators after the duration you specify in the `IOC Expiration Duration` setting.

The integration creates an [Elastic Transform](https://www.elastic.co/guide/en/elasticsearch/reference/current/transforms.html) for every source index to make sure you only see active threat indicators. Each transform creates a destination index named `logs-ti_abusech_latest.dest_*` that only contains active and unexpired threat indicators. The integration also updates indicator match rules and dashboards so they only list these active indicators.

The integration uses the following index patterns and aliases:

| Source data stream | Destination index pattern | Destination alias |
| :--- | :--- | :--- |
| `logs-ti_abusech.url-*` | `logs-ti_abusech_latest.dest_url-*` | `logs-ti_abusech_latest.url` |
| `logs-ti_abusech.malware-*` | `logs-ti_abusech_latest.dest_malware-*` | `logs-ti_abusech_latest.malware` |
| `logs-ti_abusech.malwarebazaar-*` | `logs-ti_abusech_latest.dest_malwarebazaar-*` | `logs-ti_abusech_latest.malwarebazaar` |
| `logs-ti_abusech.threatfox-*` | `logs-ti_abusech_latest.dest_threatfox-*` | `logs-ti_abusech_latest.threatfox` |

### ILM policy

To help with IoC expiration, the source data stream-backed indices (like `.ds-logs-ti_abusech.<data_stream_name>-*`) can contain duplicate entries from different polling intervals. The integration adds an ILM policy, `logs-ti_abusech.<data_stream_name>-default_policy`, to these source indices so they don't lead to unbounded growth. This policy deletes data from the source indices five days after the ingestion date.
