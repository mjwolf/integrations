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
These inputs can be used with this integration:
<details>
<summary>cel</summary>

## Setup

For more details about the CEL input settings, check the [Filebeat documentation](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-input-cel.html).

Before configuring the CEL input, make sure you have:
- Network connectivity to the target API endpoint
- Valid authentication credentials (API keys, tokens, or certificates as required)
- Appropriate permissions to read from the target data source

### Collecting logs from CEL

To configure the CEL input, you must specify the `request.url` value pointing to the API endpoint. The interval parameter controls how frequently requests are made and is the primary way to balance data freshness with API rate limits and costs. Authentication is often configured through the `request.headers` section using the appropriate method for the service.

NOTE: To access the API service, make sure you have the necessary API credentials and that the Filebeat instance can reach the endpoint URL. Some services may require IP whitelisting or VPN access.

To collect logs via API endpoint, configure the following parameters:

- API Endpoint URL
- API credentials (tokens, keys, or username/password)
- Request interval (how often to fetch data)
</details>


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

**Exported fields**

| Field | Description | Type |
|---|---|---|
| @timestamp | Event timestamp. | date |
| abusech.ja3_fingerprints.deleted_at | The timestamp when the indicator is (will be) deleted. | date |
| abusech.ja3_fingerprints.urlhaus_reference | Link to URLhaus entry. | keyword |
| cloud.image.id | Image ID for the cloud instance. | keyword |
| data_stream.dataset | Data stream dataset name. | constant_keyword |
| data_stream.namespace | Data stream namespace. | constant_keyword |
| data_stream.type | Data stream type. | constant_keyword |
| event.dataset | Event dataset | constant_keyword |
| event.module | Event module | constant_keyword |
| host.containerized | If the host is a container. | boolean |
| host.os.build | OS build information. | keyword |
| host.os.codename | OS codename, if any. | keyword |
| input.type | Type of Filebeat input. | keyword |
| labels.interval | User-configured value for `Interval` setting. This is used in calculation of indicator expiration time. | keyword |
| labels.is_ioc_transform_source | Indicates whether an IOC is in the raw source data stream, or the in latest destination index. | constant_keyword |
| log.flags | Flags for the log file. | keyword |
| log.offset | Offset of the entry in the log file. | long |
| threat.feed.dashboard_id | Dashboard ID used for Kibana CTI UI | constant_keyword |
| threat.feed.name | Display friendly feed name | constant_keyword |
| threat.indicator.first_seen | The date and time when intelligence source first reported sighting this indicator. | date |
| threat.indicator.last_seen | The date and time when intelligence source last reported sighting this indicator. | date |
| threat.indicator.modified_at | The date and time when intelligence source last modified information for this indicator. | date |


##### ja3_fingerprints sample event

An example event for `ja3_fingerprints` looks as following:

```json
{
    "@timestamp": "2025-07-31T05:12:01.523Z",
    "abusech": {
        "ja3_fingerprints": {
            "deleted_at": "2025-07-31T06:10:34.470Z"
        }
    },
    "agent": {
        "ephemeral_id": "9a4132fc-38d5-43ec-a459-0ef108d28187",
        "id": "28fe4213-ba33-434e-8815-6bbc80c646d0",
        "name": "elastic-agent-82406",
        "type": "filebeat",
        "version": "8.19.0"
    },
    "data_stream": {
        "dataset": "ti_abusech.ja3_fingerprints",
        "namespace": "86925",
        "type": "logs"
    },
    "ecs": {
        "version": "8.11.0"
    },
    "elastic_agent": {
        "id": "28fe4213-ba33-434e-8815-6bbc80c646d0",
        "snapshot": false,
        "version": "8.19.0"
    },
    "event": {
        "agent_id_status": "verified",
        "category": [
            "threat"
        ],
        "dataset": "ti_abusech.ja3_fingerprints",
        "ingested": "2025-07-31T05:12:04Z",
        "kind": "enrichment",
        "original": "{\"first_ts\":\"2017-07-14T18:08:15Z\",\"ja3\":\"b386946a5a44d1ddcc843bc75336dfce\",\"last_ts\":\"2019-07-27T20:42:54Z\",\"reason\":\"Dridex\"}",
        "type": [
            "indicator"
        ]
    },
    "input": {
        "type": "cel"
    },
    "labels": {
        "interval": "1h"
    },
    "related": {
        "hash": [
            "b386946a5a44d1ddcc843bc75336dfce"
        ]
    },
    "tags": [
        "preserve_original_event",
        "forwarded",
        "abusech-ja3_fingerprints"
    ],
    "threat": {
        "indicator": {
            "description": "Dridex",
            "first_seen": "2017-07-14T18:08:15.000Z",
            "last_seen": "2019-07-27T20:42:54.000Z",
            "name": "b386946a5a44d1ddcc843bc75336dfce",
            "type": "software"
        }
    }
}
```

#### malware

The `malware` data stream provides events from the abuse.ch URLhaus of the following types: malware payloads and threat indicators.

##### malware fields

**Exported fields**

| Field | Description | Type |
|---|---|---|
| @timestamp | Event timestamp. | date |
| abusech.malware.deleted_at | The indicator expiration timestamp. | date |
| abusech.malware.ioc_expiration_duration | The configured expiration duration. | keyword |
| abusech.malware.signature | Malware family. | keyword |
| abusech.malware.virustotal.link | Link to the Virustotal report. | keyword |
| abusech.malware.virustotal.percent | AV detection in percent. | float |
| abusech.malware.virustotal.result | AV detection ratio. | keyword |
| cloud.image.id | Image ID for the cloud instance. | keyword |
| data_stream.dataset | Data stream dataset name. | constant_keyword |
| data_stream.namespace | Data stream namespace. | constant_keyword |
| data_stream.type | Data stream type. | constant_keyword |
| event.dataset | Event dataset | constant_keyword |
| event.module | Event module | constant_keyword |
| host.containerized | If the host is a container. | boolean |
| host.os.build | OS build information. | keyword |
| host.os.codename | OS codename, if any. | keyword |
| input.type | Type of Filebeat input. | keyword |
| labels.is_ioc_transform_source | Indicates whether an IOC is in the raw source data stream, or the in latest destination index. | constant_keyword |
| log.flags | Flags for the log file. | keyword |
| log.offset | Offset of the entry in the log file. | long |
| threat.feed.dashboard_id | Dashboard ID used for Kibana CTI UI | constant_keyword |
| threat.feed.name | Display friendly feed name | constant_keyword |
| threat.indicator.first_seen | The date and time when intelligence source first reported sighting this indicator. | date |
| threat.indicator.last_seen | The date and time when intelligence source last reported sighting this indicator. | date |
| threat.indicator.modified_at | The date and time when intelligence source last modified information for this indicator. | date |


##### malware sample event

An example event for `malware` looks as following:

```json
{
    "@timestamp": "2025-07-16T06:30:10.517Z",
    "abusech": {
        "malware": {
            "deleted_at": "2021-10-10T04:17:02.000Z",
            "ioc_expiration_duration": "5d"
        }
    },
    "agent": {
        "ephemeral_id": "c478eac0-6769-456a-8a26-d5d6cc86318d",
        "id": "5d0ab6a2-0351-4c94-8bfb-e268dee367e4",
        "name": "elastic-agent-40763",
        "type": "filebeat",
        "version": "8.18.0"
    },
    "data_stream": {
        "dataset": "ti_abusech.malware",
        "namespace": "70630",
        "type": "logs"
    },
    "ecs": {
        "version": "8.11.0"
    },
    "elastic_agent": {
        "id": "5d0ab6a2-0351-4c94-8bfb-e268dee367e4",
        "snapshot": true,
        "version": "8.18.0"
    },
    "event": {
        "agent_id_status": "verified",
        "category": [
            "threat"
        ],
        "dataset": "ti_abusech.malware",
        "ingested": "2025-07-16T06:30:13Z",
        "kind": "enrichment",
        "original": "{\"file_size\":\"1563\",\"file_type\":\"unknown\",\"firstseen\":\"2021-10-05 04:17:02\",\"imphash\":null,\"md5_hash\":\"9cd5a4f0231a47823c4adba7c8ef370f\",\"sha256_hash\":\"7c0852d514df7faf8fdbfa4f358cc235dd1b1a2d843cc65495d03b502e4099f2\",\"signature\":null,\"ssdeep\":\"48:yazkS7neW+mfe4CJjNXcq5Co4Fr1PpsHn:yrmGNt5mbP2n\",\"tlsh\":\"T109314C5E7822CA70B91AD69300C22D8C2F53EAF229E6686C3BDD4C86FA1344208CF1\",\"urlhaus_download\":\"https://urlhaus-api.abuse.ch/v1/download/7c0852d514df7faf8fdbfa4f358cc235dd1b1a2d843cc65495d03b502e4099f2/\",\"virustotal\":null}",
        "type": [
            "indicator"
        ]
    },
    "input": {
        "type": "cel"
    },
    "related": {
        "hash": [
            "9cd5a4f0231a47823c4adba7c8ef370f",
            "7c0852d514df7faf8fdbfa4f358cc235dd1b1a2d843cc65495d03b502e4099f2",
            "48:yazkS7neW+mfe4CJjNXcq5Co4Fr1PpsHn:yrmGNt5mbP2n",
            "T109314C5E7822CA70B91AD69300C22D8C2F53EAF229E6686C3BDD4C86FA1344208CF1"
        ]
    },
    "tags": [
        "preserve_original_event",
        "forwarded",
        "abusech-malware"
    ],
    "threat": {
        "indicator": {
            "confidence": "Not Specified",
            "file": {
                "hash": {
                    "md5": "9cd5a4f0231a47823c4adba7c8ef370f",
                    "sha256": "7c0852d514df7faf8fdbfa4f358cc235dd1b1a2d843cc65495d03b502e4099f2",
                    "ssdeep": "48:yazkS7neW+mfe4CJjNXcq5Co4Fr1PpsHn:yrmGNt5mbP2n",
                    "tlsh": "T109314C5E7822CA70B91AD69300C22D8C2F53EAF229E6686C3BDD4C86FA1344208CF1"
                },
                "size": 1563,
                "type": "unknown"
            },
            "first_seen": "2021-10-05T04:17:02.000Z",
            "name": "7c0852d514df7faf8fdbfa4f358cc235dd1b1a2d843cc65495d03b502e4099f2",
            "type": "file"
        }
    }
}
```

#### malwarebazaar

The `malwarebazaar` data stream provides events from the abuse.ch MalwareBazaar of the following types: malware payloads and threat indicators.

##### malwarebazaar fields

**Exported fields**

| Field | Description | Type |
|---|---|---|
| @timestamp | Event timestamp. | date |
| abusech.malwarebazaar.anonymous | Identifies if the sample was submitted anonymously. | long |
| abusech.malwarebazaar.code_sign.algorithm | Algorithm used to generate the public key. | keyword |
| abusech.malwarebazaar.code_sign.cscb_listed | Whether the certificate is present on the Code Signing Certificate Blocklist (CSCB). | boolean |
| abusech.malwarebazaar.code_sign.cscb_reason | Why the certificate is present on the Code Signing Certificate Blocklist (CSCB). | keyword |
| abusech.malwarebazaar.code_sign.issuer_cn | Common name (CN) of issuing certificate authority. | keyword |
| abusech.malwarebazaar.code_sign.serial_number | Unique serial number issued by the certificate authority. | keyword |
| abusech.malwarebazaar.code_sign.subject_cn | Common name (CN) of subject. | keyword |
| abusech.malwarebazaar.code_sign.thumbprint | Hash of certificate. | keyword |
| abusech.malwarebazaar.code_sign.thumbprint_algorithm | Algorithm used to create thumbprint. | keyword |
| abusech.malwarebazaar.code_sign.valid_from | Time at which the certificate is first considered valid. | date |
| abusech.malwarebazaar.code_sign.valid_to | Time at which the certificate is no longer considered valid. | keyword |
| abusech.malwarebazaar.deleted_at | The indicator expiration timestamp. | date |
| abusech.malwarebazaar.dhash_icon | In case the file is a PE executable: dhash of the samples icon. | keyword |
| abusech.malwarebazaar.intelligence.downloads | Number of downloads from MalwareBazaar. | long |
| abusech.malwarebazaar.intelligence.mail.Generic | Malware seen in generic spam traffic. | keyword |
| abusech.malwarebazaar.intelligence.mail.IT | Malware seen in IT spam traffic. | keyword |
| abusech.malwarebazaar.intelligence.uploads | Number of uploads from MalwareBazaar. | long |
| abusech.malwarebazaar.ioc_expiration_duration | The configured expiration duration. | keyword |
| cloud.image.id | Image ID for the cloud instance. | keyword |
| data_stream.dataset | Data stream dataset name. | constant_keyword |
| data_stream.namespace | Data stream namespace. | constant_keyword |
| data_stream.type | Data stream type. | constant_keyword |
| event.dataset | Event dataset | constant_keyword |
| event.module | Event module | constant_keyword |
| host.containerized | If the host is a container. | boolean |
| host.os.build | OS build information. | keyword |
| host.os.codename | OS codename, if any. | keyword |
| input.type | Type of Filebeat input. | keyword |
| labels.is_ioc_transform_source | Indicates whether an IOC is in the raw source data stream, or the in latest destination index. | constant_keyword |
| log.flags | Flags for the log file. | keyword |
| log.offset | Offset of the entry in the log file. | long |
| threat.feed.dashboard_id | Dashboard ID used for Kibana CTI UI | constant_keyword |
| threat.feed.name | Display friendly feed name | constant_keyword |
| threat.indicator.first_seen | The date and time when intelligence source first reported sighting this indicator. | date |
| threat.indicator.last_seen | The date and time when intelligence source last reported sighting this indicator. | date |
| threat.indicator.modified_at | The date and time when intelligence source last modified information for this indicator. | date |


##### malwarebazaar sample event

An example event for `malwarebazaar` looks as following:

```json
{
    "@timestamp": "2025-07-16T06:30:59.281Z",
    "abusech": {
        "malwarebazaar": {
            "anonymous": 0,
            "deleted_at": "2021-10-10T14:02:45.000Z",
            "intelligence": {
                "downloads": 11,
                "uploads": 1
            },
            "ioc_expiration_duration": "5d"
        }
    },
    "agent": {
        "ephemeral_id": "f5b70b3f-5d2b-4d55-96b0-dc8e46e10b9a",
        "id": "372b884d-d232-4e1e-806c-d08ae525f868",
        "name": "elastic-agent-37187",
        "type": "filebeat",
        "version": "8.18.0"
    },
    "data_stream": {
        "dataset": "ti_abusech.malwarebazaar",
        "namespace": "64456",
        "type": "logs"
    },
    "ecs": {
        "version": "8.11.0"
    },
    "elastic_agent": {
        "id": "372b884d-d232-4e1e-806c-d08ae525f868",
        "snapshot": true,
        "version": "8.18.0"
    },
    "event": {
        "agent_id_status": "verified",
        "category": [
            "threat"
        ],
        "dataset": "ti_abusech.malwarebazaar",
        "ingested": "2025-07-16T06:31:02Z",
        "kind": "enrichment",
        "original": "{\"anonymous\":0,\"code_sign\":[],\"dhash_icon\":null,\"file_name\":\"7a6c03013a2f2ab8b9e8e7e5d226ea89e75da72c1519e.exe\",\"file_size\":432640,\"file_type\":\"exe\",\"file_type_mime\":\"application/x-dosexec\",\"first_seen\":\"2021-10-05 14:02:45\",\"imphash\":\"f34d5f2d4577ed6d9ceec516c1f5a744\",\"intelligence\":{\"clamav\":null,\"downloads\":\"11\",\"mail\":null,\"uploads\":\"1\"},\"last_seen\":null,\"md5_hash\":\"1fc1c2997c8f55ac10496b88e23f5320\",\"origin_country\":\"FR\",\"reporter\":\"abuse_ch\",\"sha1_hash\":\"42c7153680d7402e56fe022d1024aab49a9901a0\",\"sha256_hash\":\"7a6c03013a2f2ab8b9e8e7e5d226ea89e75da72c1519e78fd28b2253ea755c28\",\"sha3_384_hash\":\"d63e73b68973bc73ab559549aeee2141a48b8a3724aabc0d81fb14603c163a098a5a10be9f6d33b888602906c0d89955\",\"signature\":\"RedLineStealer\",\"ssdeep\":\"12288:jhhl1Eo+iEXvpb1C7drqAd1uUaJvzXGyO2F5V3bS1jsTacr:7lL\",\"tags\":[\"exe\",\"RedLineStealer\"],\"telfhash\":null,\"tlsh\":\"T13794242864BFC05994E3EEA12DDCA8FBD99A55E3640C743301B4633B8B52B84DE4F479\"}",
        "type": [
            "indicator"
        ]
    },
    "input": {
        "type": "cel"
    },
    "related": {
        "hash": [
            "42c7153680d7402e56fe022d1024aab49a9901a0",
            "d63e73b68973bc73ab559549aeee2141a48b8a3724aabc0d81fb14603c163a098a5a10be9f6d33b888602906c0d89955",
            "7a6c03013a2f2ab8b9e8e7e5d226ea89e75da72c1519e78fd28b2253ea755c28",
            "T13794242864BFC05994E3EEA12DDCA8FBD99A55E3640C743301B4633B8B52B84DE4F479",
            "12288:jhhl1Eo+iEXvpb1C7drqAd1uUaJvzXGyO2F5V3bS1jsTacr:7lL",
            "1fc1c2997c8f55ac10496b88e23f5320",
            "f34d5f2d4577ed6d9ceec516c1f5a744"
        ]
    },
    "tags": [
        "preserve_original_event",
        "forwarded",
        "abusech-malwarebazaar",
        "exe",
        "RedLineStealer"
    ],
    "threat": {
        "indicator": {
            "file": {
                "extension": "exe",
                "hash": {
                    "md5": "1fc1c2997c8f55ac10496b88e23f5320",
                    "sha1": "42c7153680d7402e56fe022d1024aab49a9901a0",
                    "sha256": "7a6c03013a2f2ab8b9e8e7e5d226ea89e75da72c1519e78fd28b2253ea755c28",
                    "sha384": "d63e73b68973bc73ab559549aeee2141a48b8a3724aabc0d81fb14603c163a098a5a10be9f6d33b888602906c0d89955",
                    "ssdeep": "12288:jhhl1Eo+iEXvpb1C7drqAd1uUaJvzXGyO2F5V3bS1jsTacr:7lL",
                    "tlsh": "T13794242864BFC05994E3EEA12DDCA8FBD99A55E3640C743301B4633B8B52B84DE4F479"
                },
                "mime_type": "application/x-dosexec",
                "name": "7a6c03013a2f2ab8b9e8e7e5d226ea89e75da72c1519e.exe",
                "pe": {
                    "imphash": "f34d5f2d4577ed6d9ceec516c1f5a744"
                },
                "size": 432640
            },
            "first_seen": "2021-10-05T14:02:45.000Z",
            "geo": {
                "country_iso_code": "FR"
            },
            "marking": {
                "tlp": "CLEAR"
            },
            "name": "7a6c03013a2f2ab8b9e8e7e5d226ea89e75da72c1519e78fd28b2253ea755c28",
            "provider": "abuse_ch",
            "type": "file"
        },
        "software": {
            "alias": [
                "RedLineStealer"
            ]
        }
    }
}
```

#### sslblacklist

The `sslblacklist` data stream provides events from the abuse.ch SSLBL of the following types: blacklisted SSL certificates and threat indicators.

##### sslblacklist fields

**Exported fields**

| Field | Description | Type |
|---|---|---|
| @timestamp | Event timestamp. | date |
| abusech.sslblacklist.deleted_at | The timestamp when the indicator is (will be) deleted. | date |
| cloud.image.id | Image ID for the cloud instance. | keyword |
| data_stream.dataset | Data stream dataset name. | constant_keyword |
| data_stream.namespace | Data stream namespace. | constant_keyword |
| data_stream.type | Data stream type. | constant_keyword |
| event.dataset | Event dataset | constant_keyword |
| event.module | Event module | constant_keyword |
| host.containerized | If the host is a container. | boolean |
| host.os.build | OS build information. | keyword |
| host.os.codename | OS codename, if any. | keyword |
| input.type | Type of Filebeat input. | keyword |
| labels.interval | User-configured value for `Interval` setting. This is used in calculation of indicator expiration time. | keyword |
| labels.is_ioc_transform_source | Indicates whether an IOC is in the raw source data stream, or the in latest destination index. | constant_keyword |
| log.flags | Flags for the log file. | keyword |
| log.offset | Offset of the entry in the log file. | long |
| threat.feed.dashboard_id | Dashboard ID used for Kibana CTI UI | constant_keyword |
| threat.feed.name | Display friendly feed name | constant_keyword |
| threat.indicator.first_seen | The date and time when intelligence source first reported sighting this indicator. | date |
| threat.indicator.last_seen | The date and time when intelligence source last reported sighting this indicator. | date |
| threat.indicator.modified_at | The date and time when intelligence source last modified information for this indicator. | date |


##### sslblacklist sample event

An example event for `sslblacklist` looks as following:

```json
{
    "@timestamp": "2025-07-31T05:15:00.672Z",
    "abusech": {
        "sslblacklist": {
            "deleted_at": "2025-07-31T06:13:33.669Z"
        }
    },
    "agent": {
        "ephemeral_id": "80e31fdd-70e8-4156-9a0d-ad6d0d853888",
        "id": "01f51d20-e150-4b4e-a036-1746eb0c7285",
        "name": "elastic-agent-47845",
        "type": "filebeat",
        "version": "8.19.0"
    },
    "data_stream": {
        "dataset": "ti_abusech.sslblacklist",
        "namespace": "19255",
        "type": "logs"
    },
    "ecs": {
        "version": "8.11.0"
    },
    "elastic_agent": {
        "id": "01f51d20-e150-4b4e-a036-1746eb0c7285",
        "snapshot": false,
        "version": "8.19.0"
    },
    "event": {
        "agent_id_status": "verified",
        "category": [
            "threat"
        ],
        "dataset": "ti_abusech.sslblacklist",
        "ingested": "2025-07-31T05:15:03Z",
        "kind": "enrichment",
        "original": "{\"reason\":\"HijackLoader C\\u0026C\",\"sha1\":\"029c128ec7f6c5a62ea19f5ad525cd1487971ce4\",\"ts\":\"2025-06-25T06:50:28Z\"}",
        "type": [
            "indicator"
        ]
    },
    "input": {
        "type": "cel"
    },
    "labels": {
        "interval": "1h"
    },
    "related": {
        "hash": [
            "029c128ec7f6c5a62ea19f5ad525cd1487971ce4"
        ]
    },
    "tags": [
        "preserve_original_event",
        "forwarded",
        "abusech-sslblacklist"
    ],
    "threat": {
        "indicator": {
            "description": "HijackLoader C&C",
            "first_seen": "2025-06-25T06:50:28.000Z",
            "name": "029c128ec7f6c5a62ea19f5ad525cd1487971ce4",
            "type": "x509-certificate"
        }
    }
}
```

#### threatfox

The `threatfox` data stream provides events from the abuse.ch ThreatFox of the following types: indicators of compromise and threat indicators.

##### threatfox fields

**Exported fields**

| Field | Description | Type |
|---|---|---|
| @timestamp | Event timestamp. | date |
| abusech.threatfox.confidence_level | Confidence level between 0-100. | long |
| abusech.threatfox.deleted_at | The indicator expiration timestamp. | date |
| abusech.threatfox.ioc_expiration_duration | The configured expiration duration. | keyword |
| abusech.threatfox.malware | The malware associated with the IOC. | keyword |
| abusech.threatfox.tags | A list of tags associated with the queried malware sample. | keyword |
| abusech.threatfox.threat_type | The type of threat. | keyword |
| abusech.threatfox.threat_type_desc | The threat descsription. | keyword |
| cloud.image.id | Image ID for the cloud instance. | keyword |
| data_stream.dataset | Data stream dataset name. | constant_keyword |
| data_stream.namespace | Data stream namespace. | constant_keyword |
| data_stream.type | Data stream type. | constant_keyword |
| event.dataset | Event dataset | constant_keyword |
| event.module | Event module | constant_keyword |
| host.containerized | If the host is a container. | boolean |
| host.os.build | OS build information. | keyword |
| host.os.codename | OS codename, if any. | keyword |
| input.type | Type of Filebeat input. | keyword |
| labels.is_ioc_transform_source | Indicates whether an IOC is in the raw source data stream, or the in latest destination index. | constant_keyword |
| log.flags | Flags for the log file. | keyword |
| log.offset | Offset of the entry in the log file. | long |
| threat.feed.dashboard_id | Dashboard ID used for Kibana CTI UI | constant_keyword |
| threat.feed.name | Display friendly feed name | constant_keyword |
| threat.indicator.first_seen | The date and time when intelligence source first reported sighting this indicator. | date |
| threat.indicator.last_seen | The date and time when intelligence source last reported sighting this indicator. | date |
| threat.indicator.modified_at | The date and time when intelligence source last modified information for this indicator. | date |


##### threatfox sample event

An example event for `threatfox` looks as following:

```json
{
    "@timestamp": "2025-07-16T06:31:50.732Z",
    "abusech": {
        "threatfox": {
            "confidence_level": 100,
            "deleted_at": "2022-08-10T19:43:08.000Z",
            "ioc_expiration_duration": "5d",
            "malware": "win.asyncrat",
            "threat_type": "botnet_cc",
            "threat_type_desc": "Indicator that identifies a botnet command&control server (C&C)"
        }
    },
    "agent": {
        "ephemeral_id": "49a54718-d50a-45cf-8da6-597e14572d1b",
        "id": "07477042-3fd0-44e5-83e1-d33c53a1b34d",
        "name": "elastic-agent-57963",
        "type": "filebeat",
        "version": "8.18.0"
    },
    "data_stream": {
        "dataset": "ti_abusech.threatfox",
        "namespace": "90202",
        "type": "logs"
    },
    "ecs": {
        "version": "8.11.0"
    },
    "elastic_agent": {
        "id": "07477042-3fd0-44e5-83e1-d33c53a1b34d",
        "snapshot": true,
        "version": "8.18.0"
    },
    "event": {
        "agent_id_status": "verified",
        "category": [
            "threat"
        ],
        "dataset": "ti_abusech.threatfox",
        "id": "841537",
        "ingested": "2025-07-16T06:31:53Z",
        "kind": "enrichment",
        "original": "{\"confidence_level\":100,\"first_seen\":\"2022-08-05 19:43:08 UTC\",\"id\":\"841537\",\"ioc\":\"wizzy.hopto.org\",\"ioc_type\":\"domain\",\"ioc_type_desc\":\"Domain that is used for botnet Command\\u0026control (C\\u0026C)\",\"last_seen\":null,\"malware\":\"win.asyncrat\",\"malware_alias\":null,\"malware_malpedia\":\"https://malpedia.caad.fkie.fraunhofer.de/details/win.asyncrat\",\"malware_printable\":\"AsyncRAT\",\"reference\":\"https://tria.ge/220805-w57pxsgae2\",\"reporter\":\"AndreGironda\",\"tags\":[\"asyncrat\"],\"threat_type\":\"botnet_cc\",\"threat_type_desc\":\"Indicator that identifies a botnet command\\u0026control server (C\\u0026C)\"}",
        "type": [
            "indicator"
        ]
    },
    "input": {
        "type": "cel"
    },
    "tags": [
        "preserve_original_event",
        "forwarded",
        "abusech-threatfox",
        "asyncrat"
    ],
    "threat": {
        "indicator": {
            "confidence": "High",
            "description": "Domain that is used for botnet Command&control (C&C)",
            "first_seen": "2022-08-05T19:43:08.000Z",
            "marking": {
                "tlp": "WHITE"
            },
            "name": "wizzy.hopto.org",
            "provider": "AndreGironda",
            "reference": "https://tria.ge/220805-w57pxsgae2",
            "type": "domain-name",
            "url": {
                "domain": "wizzy.hopto.org"
            }
        },
        "software": {
            "name": "AsyncRAT",
            "reference": "https://malpedia.caad.fkie.fraunhofer.de/details/win.asyncrat"
        }
    }
}
```

#### url

The `url` data stream provides events from the abuse.ch URLhaus of the following types: malware URLs and threat indicators.

##### url fields

**Exported fields**

| Field | Description | Type |
|---|---|---|
| @timestamp | Event timestamp. | date |
| abusech.url.blacklists.spamhaus_dbl | If the indicator is listed on the spamhaus blacklist. | keyword |
| abusech.url.blacklists.surbl | If the indicator is listed on the surbl blacklist. | keyword |
| abusech.url.deleted_at | The timestamp when the indicator is (will be) deleted. | date |
| abusech.url.id | The ID of the indicator. | keyword |
| abusech.url.larted | Indicates whether the malware URL has been reported to the hosting provider (true or false). | boolean |
| abusech.url.last_online | Last timestamp when the URL has been serving malware. | date |
| abusech.url.reporter | The Twitter handle of the reporter that has reported this malware URL (or anonymous). | keyword |
| abusech.url.tags | A list of tags associated with the queried malware URL. | keyword |
| abusech.url.threat | The threat corresponding to this malware URL. | keyword |
| abusech.url.url_status | The current status of the URL. Possible values are: online, offline and unknown. | keyword |
| abusech.url.urlhaus_reference | Link to URLhaus entry. | keyword |
| cloud.image.id | Image ID for the cloud instance. | keyword |
| data_stream.dataset | Data stream dataset name. | constant_keyword |
| data_stream.namespace | Data stream namespace. | constant_keyword |
| data_stream.type | Data stream type. | constant_keyword |
| event.dataset | Event dataset | constant_keyword |
| event.module | Event module | constant_keyword |
| host.containerized | If the host is a container. | boolean |
| host.os.build | OS build information. | keyword |
| host.os.codename | OS codename, if any. | keyword |
| input.type | Type of Filebeat input. | keyword |
| labels.interval | User-configured value for `Interval` setting. This is used in calculation of indicator expiration time. | keyword |
| labels.is_ioc_transform_source | Indicates whether an IOC is in the raw source data stream, or the in latest destination index. | constant_keyword |
| log.flags | Flags for the log file. | keyword |
| log.offset | Offset of the entry in the log file. | long |
| threat.feed.dashboard_id | Dashboard ID used for Kibana CTI UI | constant_keyword |
| threat.feed.name | Display friendly feed name | constant_keyword |
| threat.indicator.first_seen | The date and time when intelligence source first reported sighting this indicator. | date |
| threat.indicator.last_seen | The date and time when intelligence source last reported sighting this indicator. | date |
| threat.indicator.modified_at | The date and time when intelligence source last modified information for this indicator. | date |


##### url sample event

An example event for `url` looks as following:

```json
{
    "@timestamp": "2025-07-16T06:32:41.644Z",
    "abusech": {
        "url": {
            "deleted_at": "2025-07-16T07:31:14.625Z",
            "id": "2786904",
            "threat": "malware_download",
            "url_status": "online"
        }
    },
    "agent": {
        "ephemeral_id": "8039c627-ea96-4027-8751-2ff7db77251b",
        "id": "9106f11b-d54d-46d0-8ace-39e4fff1157b",
        "name": "elastic-agent-41888",
        "type": "filebeat",
        "version": "8.18.0"
    },
    "data_stream": {
        "dataset": "ti_abusech.url",
        "namespace": "49664",
        "type": "logs"
    },
    "ecs": {
        "version": "8.11.0"
    },
    "elastic_agent": {
        "id": "9106f11b-d54d-46d0-8ace-39e4fff1157b",
        "snapshot": true,
        "version": "8.18.0"
    },
    "event": {
        "agent_id_status": "verified",
        "category": [
            "threat"
        ],
        "dataset": "ti_abusech.url",
        "ingested": "2025-07-16T06:32:44Z",
        "kind": "enrichment",
        "original": "{\"dateadded\":\"2024-03-19 11:34:09 UTC\",\"id\":\"2786904\",\"last_online\":\"2024-03-19 11:34:09 UTC\",\"reporter\":\"lrz_urlhaus\",\"tags\":[\"elf\",\"Mozi\"],\"threat\":\"malware_download\",\"url\":\"http://115.55.244.160:41619/Mozi.m\",\"url_status\":\"online\",\"urlhaus_link\":\"https://urlhaus.abuse.ch/url/2786904/\"}",
        "type": [
            "indicator"
        ]
    },
    "input": {
        "type": "cel"
    },
    "labels": {
        "interval": "1h"
    },
    "tags": [
        "preserve_original_event",
        "forwarded",
        "abusech-url",
        "elf",
        "Mozi"
    ],
    "threat": {
        "indicator": {
            "first_seen": "2024-03-19T11:34:09.000Z",
            "last_seen": "2024-03-19T11:34:09.000Z",
            "name": "http://115.55.244.160:41619/Mozi.m",
            "provider": "lrz_urlhaus",
            "reference": "https://urlhaus.abuse.ch/url/2786904/",
            "type": "url",
            "url": {
                "domain": "115.55.244.160",
                "extension": "m",
                "full": "http://115.55.244.160:41619/Mozi.m",
                "original": "http://115.55.244.160:41619/Mozi.m",
                "path": "/Mozi.m",
                "port": 41619,
                "scheme": "http"
            }
        }
    }
}
```

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
