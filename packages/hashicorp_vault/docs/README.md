## Overview

The Hashicorp Vault integration for Elastic enables collection of audit logs, operational logs, and metrics to provide comprehensive visibility into the security and operational health of your Vault environment. You can monitor detailed audit trails, track internal server operations, and analyze performance metrics to ensure stability and compliance.

This integration facilitates:
- **Security monitoring**: Ingests audit logs to track all requests and responses, crucial for threat detection and compliance.
- **Operational visibility**: Collects internal server logs including startup, shutdown, and error messages.
- **Performance analysis**: Gathers telemetry data in Prometheus format to monitor memory usage, request latency, and secret engine activity.

### Compatibility

This integration has been tested with Hashicorp Vault version 1.11. It is expected to be compatible with other recent versions.

### How it works

The integration uses different methods to collect data depending on the source:

- **Audit logs**: Can be collected via:
  - **Syslog**: Vault sends logs to a local syslog daemon, which forwards them to Elastic Agent over TCP (recommended for production).
  - **File**: Elastic Agent monitors the file written by Vault's `file` audit device.
  - **TCP Socket**: Elastic Agent listens on a TCP port, and Vault streams logs directly to it.
- **Operational logs**: Elastic Agent monitors the file where Vault's standard output (stdout) is redirected. Logs must be in JSON format.
- **Metrics**: Elastic Agent periodically scrapes the Prometheus-compatible metrics endpoint exposed by Vault.

## What data does this integration collect?

This integration collects logs and metrics from HashiCorp Vault across three data streams:

* **`audit`**: Captures detailed audit logs, including authentication attempts, secret access, policy changes, and system operations. The logs contain hashed secret values (HMAC-SHA256) for security.
    * **Note**: It is critical to understand how Vault handles audit device failures. If an audit device cannot write logs, Vault may block requests to ensure no action is performed without an audit record. Refer to [Blocked Audit Devices](https://developer.hashicorp.com/vault/docs/audit/socket#configuration) and [Blocked Audit Device Behavior](https://developer.hashicorp.com/vault/tutorials/monitoring/blocked-audit-devices#blocked-audit-device-behavior) for details on configuration and behavior.
* **`log`**: Captures operational logs from the Vault server itself, including errors, warnings, and informational messages about the server's status.
* **`metrics`**: Collects a wide range of performance metrics, such as counters, gauges, and summaries related to runtime, secret engines, and token counts.

### Supported use cases

* **Security auditing**: Track every request and response to Vault to detect unauthorized access, policy violations, and suspicious activity.
* **Operational monitoring**: Monitor the health of your Vault cluster, diagnose issues using operational logs, and set up alerts for critical errors.
* **Performance analysis**: Analyze performance metrics to understand request latency, resource utilization, and identify performance bottlenecks.
* **Compliance**: Maintain a detailed, verifiable log of all activities for compliance with regulations like PCI DSS, SOX, and HIPAA.

## What do I need to use this integration?

Before you begin, ensure you have the following prerequisites configured:

- **Elastic Agent**: You need a Fleet-managed Elastic Agent installed on a host that can communicate with your HashiCorp Vault cluster.
- **HashiCorp Vault**: An active installation of HashiCorp Vault.

### Credentials and Permissions
- **Administrative Access**: To configure audit logging, you need a Vault token with `sudo` capabilities or a policy that allows enabling and configuring audit devices.
- **Metrics Access**: For metrics collection, you need a Vault token with read access to the `/sys/metrics` API endpoint.
  - Example policy capability: `path "sys/metrics" { capabilities = ["read"] }`

### Network Connectivity
- **Metrics Collection**: The Elastic Agent must be able to reach the Vault API endpoint (default: `http://localhost:8200`).
- **Audit Log Collection (Socket)**: If using the socket audit device, the Vault server must be able to initiate a TCP connection to the Elastic Agent (default port: `9007`).
- **Firewall Rules**: Ensure firewall rules allow traffic on the configured ports (e.g., 8200 for API, 9007 for audit logs).

### Vault Configuration
- **Operational Logs**: Vault must be configured to output logs in JSON format (`log_format = "json"`) for proper parsing.
- **Telemetry**: For metrics, Vault telemetry must be enabled with `disable_hostname = true` and `enable_hostname_label = true`.
- **Filesystem Access**: If collecting logs from files, the Elastic Agent requires read access to the Vault log directories (e.g., `/var/log/vault/`).
- **Safety Requirement**: If using the TCP socket audit device, HashiCorp strongly recommends configuring a secondary file-based audit device. This prevents Vault from blocking requests if the TCP connection to the Elastic Agent becomes unavailable.

## How do I deploy this integration?

### Agent-based deployment

The Elastic Agent is a unified agent that collects data from your systems and ships it to Elastic. To deploy this integration:

1. **Install Elastic Agent** on a host that has network access to both your Elastic deployment and the data source.
   - See the [Elastic Agent installation guide](https://www.elastic.co/guide/en/fleet/current/install-fleet-managed-elastic-agent.html).

2. **Enroll the agent** in Fleet:
   - In Kibana, go to **Management** → **Fleet** → **Agents**.
   - Click **Add agent** and follow the enrollment instructions.

3. **Add the integration** to an agent policy:
   - Go to **Management** → **Integrations**.
   - Search for **Hashicorp Vault**.
   - Click **Add Hashicorp Vault** and configure the settings.
   - Assign to an existing policy or create a new one.

**Network Requirements**

| Direction | Protocol | Port | Purpose |
|-----------|----------|------|----------|
| Agent → Elastic | HTTPS | 443 | Data shipping to Elasticsearch |
| Agent (local) | — | — | File read access required (Audit/Operational logs) |
| Source → Agent | TCP | 9007 (configurable) | Socket Audit Device (if enabled) |
| Agent → Source | HTTP | 8200 (configurable) | Metrics collection |

### Set up steps in Hashicorp Vault

Before using the integration, you must configure HashiCorp Vault to expose metrics and logs in a format that Elastic Agent can collect.

### Set up steps in Kibana

1. In Kibana, navigate to **Management** > **Integrations**.
2. Search for "HashiCorp Vault" and select the integration.
3. Click **Add HashiCorp Vault**.
4. Configure the integration inputs based on your setup:

   **For Audit Logs (File)**:
   - Enable the **Logs from file** > **Audit logs (file audit device)** input.
   - Specify the file path (default: `/var/log/vault/audit*.json*`).
   - Optionally enable **Preserve original event** to keep the raw log message.

   **For Audit Logs (TCP Socket)**:
   - Enable the **Logs from TCP socket** input.
   - Configure the **Listen address** (default: `localhost`) and **Port** (default: `9007`).
   - If Vault is connecting from a remote host, change the listen address to `0.0.0.0`.

   **For Operational Logs**:
   - Enable the **Logs from file** > **Operation logs** input.
   - Specify the log file path (default: `/var/log/vault/log*.json*`).

   **For Metrics**:
   - Enable the **Metrics** input.
   - Enter the **Hosts** URL (default: `http://localhost:8200`).
   - Enter the **Token** generated in the previous section.
   - Adjust the **Period** if needed (default: `30s`).

5. select the agent policy where you want to run this integration.
6. Click **Save and continue**.

### Validation

After deployment, you can verify that data is flowing correctly.

1. **Verify Elastic Agent status**
   - In Kibana, navigate to **Management** → **Fleet** → **Agents**.
   - Confirm the agent status shows **Healthy** (green).
   - Click the agent name to verify the Hashicorp Vault integration is listed and shows no errors.

2. **Trigger data flow on HashiCorp Vault**
   - **Generate Authentication Event**: Perform a login operation using the Vault CLI:
     ```bash
     vault login -method=token
     ```
   - **Generate Secret Access Event**: Read a test secret from the KV store:
     ```bash
     vault kv get secret/test_connection
     ```
   - **Generate Administrative Event**: List the enabled audit devices:
     ```bash
     vault audit list
     ```

3. **Check for incoming data in Discover**
   - Go to **Analytics** → **Discover**.
   - Set the time range to **Last 15 minutes** or longer.
   - Search for data from this integration using: `data_stream.dataset:hashicorp_vault*`
   - This will show ALL data streams for the integration (logs, metrics, etc.).
   - Verify documents are appearing with recent timestamps.

4. **Verify specific data streams**
   - **Audit Logs**: Filter for `data_stream.dataset:hashicorp_vault.audit`.
   - **Operational Logs**: Filter for `data_stream.dataset:hashicorp_vault.log`.
   - **Metrics**: Filter for `data_stream.dataset:hashicorp_vault.metrics`.

5. **Check dashboards**
   - Navigate to **Management** → **Integrations** → **Hashicorp Vault**.
   - Click the **Assets** tab to see available dashboards.
   - Open the **[Logs HashiCorp Vault] Audit Overview** dashboard to confirm visualizations are populated with data.

## Troubleshooting

### General debugging steps

### Vendor-specific issues

### Log file input troubleshooting

### TCP/Syslog input troubleshooting

### Prometheus metrics input troubleshooting

## Performance and scaling

For more information on architectures that can be used for scaling this integration, check the [Ingest Architectures](https://www.elastic.co/docs/manage-data/ingest/ingest-reference-architectures) documentation.

The **Hashicorp Vault** integration uses several input types, each with specific performance and scaling considerations.

### Log File Input (`logfile`)
- **Fault Tolerance**: The Elastic Agent tracks its position in each log file, ensuring no data is lost during agent restarts.
- **Scaling Guidance**:
  - For directories with many log files, use glob patterns (e.g., `/var/log/vault/*.json`) to monitor them efficiently.
  - Adjust the `harvester_limit` setting to control the number of concurrently harvested files and manage resource usage.
  - Use the `close_inactive` setting to release file handles for rotated logs. This is particularly important for high-volume systems where log rotation occurs frequently.

### TCP/Syslog Input (`tcp`)
- **Fault Tolerance**: TCP provides reliable, ordered delivery with acknowledgments, making it suitable for production environments where data integrity is critical.
- **Scaling Guidance**:
  - To scale, you can deploy multiple Elastic Agents and use a load balancer to distribute syslog traffic across them.
  - TCP naturally handles backpressure. If Elasticsearch is slow to ingest, the TCP connection will queue data.
  - Monitor connection limits on both the Vault host and the Elastic Agent host to ensure they can handle the required number of connections.
- **Critical Configuration**: When using the socket audit backend, Hashicorp Vault effectively blocks if the audit device (in this case, Elastic Agent) becomes unavailable or cannot read data fast enough. This can cause Vault to stop processing client requests.
  - Review the Hashicorp documentation on [Blocked Audit Devices](https://developer.hashicorp.com/vault/docs/audit/socket#configuration) to understand how to configure the `hmac_accessor`, `socket_type`, and potential fallbacks to prevent service outages.

### Prometheus Metrics Input (`prometheus`)
- **Fault Tolerance**: This input polls the Vault metrics endpoint. If a poll fails, data for that interval is lost, but the next successful poll will resume collection.
- **Scaling Guidance**:
  - Adjust the collection interval (default is often 10s or 1m) to balance data resolution with the load on the Vault API.
  - High-cardinality metrics can increase storage usage in Elasticsearch. If you do not need all metrics, consider filtering specific metric sets.
  - Ensure the Elastic Agent is located close to the Vault cluster (network-wise) to minimize latency during scrapes.

## Reference

### audit

The `audit` data stream provides events from Hashicorp Vault of the following types: audit logs.

These logs contain detailed information about requests and responses to Vault, allowing you to track access and actions performed against the secrets engine.

An example event for `audit` looks as following:

```json
{
    "@timestamp": "2023-09-26T13:07:49.743Z",
    "agent": {
        "ephemeral_id": "5bbd86cc-8032-432d-be82-fae8f624ed98",
        "id": "f25d13cd-18cc-4e73-822c-c4f849322623",
        "name": "docker-fleet-agent",
        "type": "filebeat",
        "version": "8.10.1"
    },
    "data_stream": {
        "dataset": "hashicorp_vault.audit",
        "namespace": "ep",
        "type": "logs"
    },
    "ecs": {
        "version": "8.17.0"
    },
    "elastic_agent": {
        "id": "f25d13cd-18cc-4e73-822c-c4f849322623",
        "snapshot": false,
        "version": "8.10.1"
    },
    "event": {
        "action": "update",
        "agent_id_status": "verified",
        "category": [
            "authentication"
        ],
        "dataset": "hashicorp_vault.audit",
        "id": "0b1b9013-da54-633d-da69-8575e6794ed3",
        "ingested": "2023-09-26T13:08:15Z",
        "kind": "event",
        "original": "{\"time\":\"2023-09-26T13:07:49.743284857Z\",\"type\":\"request\",\"auth\":{\"token_type\":\"default\"},\"request\":{\"id\":\"0b1b9013-da54-633d-da69-8575e6794ed3\",\"operation\":\"update\",\"namespace\":{\"id\":\"root\"},\"path\":\"sys/audit/test\"}}",
        "outcome": "success",
        "type": [
            "info"
        ]
    },
    "hashicorp_vault": {
        "audit": {
            "auth": {
                "token_type": "default"
            },
            "request": {
                "id": "0b1b9013-da54-633d-da69-8575e6794ed3",
                "namespace": {
                    "id": "root"
                },
                "operation": "update",
                "path": "sys/audit/test"
            },
            "type": "request"
        }
    },
    "host": {
        "architecture": "x86_64",
        "containerized": false,
        "hostname": "docker-fleet-agent",
        "id": "28da52b32df94b50aff67dfb8f1be3d6",
        "ip": [
            "192.168.80.5"
        ],
        "mac": [
            "02-42-C0-A8-50-05"
        ],
        "name": "docker-fleet-agent",
        "os": {
            "codename": "focal",
            "family": "debian",
            "kernel": "5.10.104-linuxkit",
            "name": "Ubuntu",
            "platform": "ubuntu",
            "type": "linux",
            "version": "20.04.6 LTS (Focal Fossa)"
        }
    },
    "input": {
        "type": "log"
    },
    "log": {
        "file": {
            "path": "/tmp/service_logs/vault/audit.json"
        },
        "offset": 0
    },
    "tags": [
        "preserve_original_event",
        "hashicorp-vault-audit"
    ]
}
```

**Exported fields**

| Field | Description | Type |
|---|---|---|
| @timestamp | Date/time when the event originated. This is the date/time extracted from the event, typically representing when the event was generated by the source. If the event source has no original timestamp, this value is typically populated by the first time the event was received by the pipeline. Required field for all events. | date |
| data_stream.dataset | The field can contain anything that makes sense to signify the source of the data. Examples include `nginx.access`, `prometheus`, `endpoint` etc. For data streams that otherwise fit, but that do not have dataset set we use the value "generic" for the dataset value. `event.dataset` should have the same value as `data_stream.dataset`. Beyond the Elasticsearch data stream naming criteria noted above, the `dataset` value has additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |
| data_stream.namespace | A user defined namespace. Namespaces are useful to allow grouping of data. Many users already organize their indices this way, and the data stream naming scheme now provides this best practice as a default. Many users will populate this field with `default`. If no value is used, it falls back to `default`. Beyond the Elasticsearch index naming criteria noted above, `namespace` value has the additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |
| data_stream.type | An overarching type for the data stream. Currently allowed values are "logs" and "metrics". We expect to also add "traces" and "synthetics" in the near future. | constant_keyword |
| ecs.version | ECS version this event conforms to. `ecs.version` is a required field and must exist in all events. When querying across multiple indices -- which may conform to slightly different ECS versions -- this field lets integrations adjust to the schema version of the events. | keyword |
| event.action | The action captured by the event. This describes the information in the event. It is more specific than `event.category`. Examples are `group-add`, `process-started`, `file-created`. The value is normally defined by the implementer. | keyword |
| event.category | This is one of four ECS Categorization Fields, and indicates the second level in the ECS category hierarchy. `event.category` represents the "big buckets" of ECS categories. For example, filtering on `event.category:process` yields all events relating to process activity. This field is closely related to `event.type`, which is used as a subcategory. This field is an array. This will allow proper categorization of some events that fall in multiple categories. | keyword |
| event.dataset | Event dataset | constant_keyword |
| event.id | Unique ID to describe the event. | keyword |
| event.kind | This is one of four ECS Categorization Fields, and indicates the highest level in the ECS category hierarchy. `event.kind` gives high-level information about what type of information the event contains, without being specific to the contents of the event. For example, values of this field distinguish alert events from metric events. The value of this field can be used to inform how these kinds of events should be handled. They may warrant different retention, different access control, it may also help understand whether the data is coming in at a regular interval or not. | keyword |
| event.module | Event module | constant_keyword |
| event.original | Raw text message of entire event. Used to demonstrate log integrity or where the full log message (before splitting it up in multiple parts) may be required, e.g. for reindex. This field is not indexed and doc_values are disabled. It cannot be searched, but it can be retrieved from `_source`. If users wish to override this and index this field, please see `Field data types` in the `Elasticsearch Reference`. | keyword |
| event.outcome | This is one of four ECS Categorization Fields, and indicates the lowest level in the ECS category hierarchy. `event.outcome` simply denotes whether the event represents a success or a failure from the perspective of the entity that produced the event. Note that when a single transaction is described in multiple events, each event may populate different values of `event.outcome`, according to their perspective. Also note that in the case of a compound event (a single event that contains multiple logical events), this field should be populated with the value that best captures the overall success or failure from the perspective of the event producer. Further note that not all events will have an associated outcome. For example, this field is generally not populated for metric events, events with `event.type:info`, or any events for which an outcome does not make logical sense. | keyword |
| event.type | This is one of four ECS Categorization Fields, and indicates the third level in the ECS category hierarchy. `event.type` represents a categorization "sub-bucket" that, when used along with the `event.category` field values, enables filtering events down to a level appropriate for single visualization. This field is an array. This will allow proper categorization of some events that fall in multiple event types. | keyword |
| hashicorp_vault.audit.auth.accessor | This is an HMAC of the client token accessor | keyword |
| hashicorp_vault.audit.auth.client_token | This is an HMAC of the client's token ID. | keyword |
| hashicorp_vault.audit.auth.display_name | Display name is a non-security sensitive identifier that is applicable to this auth. It is used for logging and prefixing of dynamic secrets. For example, it may be "armon" for the github credential backend. If the client token is used to generate a SQL credential, the user may be "github-armon-uuid". This is to help identify the source without using audit tables. | keyword |
| hashicorp_vault.audit.auth.entity_id | Entity ID is the identifier of the entity in identity store to which the identity of the authenticating client belongs to. | keyword |
| hashicorp_vault.audit.auth.external_namespace_policies | External namespace policies represent the policies authorized from different namespaces indexed by respective namespace identifiers. | flattened |
| hashicorp_vault.audit.auth.identity_policies | These are the policies sourced from the identity. | keyword |
| hashicorp_vault.audit.auth.metadata | This will contain a list of metadata key/value pairs associated with the authenticated user. | flattened |
| hashicorp_vault.audit.auth.no_default_policy | Indicates that the default policy should not be added by core when creating a token. The default policy will still be added if it's explicitly defined. | boolean |
| hashicorp_vault.audit.auth.policies | Policies is the list of policies that the authenticated user is associated with. | keyword |
| hashicorp_vault.audit.auth.policy_results.allowed |  | boolean |
| hashicorp_vault.audit.auth.policy_results.granting_policies.name |  | keyword |
| hashicorp_vault.audit.auth.policy_results.granting_policies.namespace_id |  | keyword |
| hashicorp_vault.audit.auth.policy_results.granting_policies.type |  | keyword |
| hashicorp_vault.audit.auth.remaining_uses |  | long |
| hashicorp_vault.audit.auth.token_issue_time |  | date |
| hashicorp_vault.audit.auth.token_policies | These are the policies sourced from the token. | keyword |
| hashicorp_vault.audit.auth.token_ttl |  | long |
| hashicorp_vault.audit.auth.token_type |  | keyword |
| hashicorp_vault.audit.entity_created | entity_created is set to true if an entity is created as part of a login request. | boolean |
| hashicorp_vault.audit.error | If an error occurred with the request, the error message is included in this field's value. | keyword |
| hashicorp_vault.audit.request.client_certificate_serial_number | Serial number from the client's certificate. | keyword |
| hashicorp_vault.audit.request.client_id |  | keyword |
| hashicorp_vault.audit.request.client_token | This is an HMAC of the client's token ID. | keyword |
| hashicorp_vault.audit.request.client_token_accessor | This is an HMAC of the client token accessor. | keyword |
| hashicorp_vault.audit.request.data | The data object will contain secret data in key/value pairs. | flattened |
| hashicorp_vault.audit.request.headers | Additional HTTP headers specified by the client as part of the request. | flattened |
| hashicorp_vault.audit.request.id | This is the unique request identifier. | keyword |
| hashicorp_vault.audit.request.mount_accessor |  | keyword |
| hashicorp_vault.audit.request.mount_type |  | keyword |
| hashicorp_vault.audit.request.namespace.id |  | keyword |
| hashicorp_vault.audit.request.namespace.path |  | keyword |
| hashicorp_vault.audit.request.operation | This is the type of operation which corresponds to path capabilities and is expected to be one of: create, read, update, delete, or list. | keyword |
| hashicorp_vault.audit.request.path | The requested Vault path for operation. | keyword |
| hashicorp_vault.audit.request.path.text | Multi-field of `hashicorp_vault.audit.request.path`. | text |
| hashicorp_vault.audit.request.policy_override | Policy override indicates that the requestor wishes to override soft-mandatory Sentinel policies. | boolean |
| hashicorp_vault.audit.request.remote_address | The IP address of the client making the request. | ip |
| hashicorp_vault.audit.request.remote_port | The remote port of the client making the request. | long |
| hashicorp_vault.audit.request.replication_cluster | Name given to the replication secondary where this request originated. | keyword |
| hashicorp_vault.audit.request.wrap_ttl | If the token is wrapped, this displays configured wrapped TTL in seconds. | long |
| hashicorp_vault.audit.response.auth.accessor |  | keyword |
| hashicorp_vault.audit.response.auth.client_token |  | keyword |
| hashicorp_vault.audit.response.auth.display_name |  | keyword |
| hashicorp_vault.audit.response.auth.entity_id |  | keyword |
| hashicorp_vault.audit.response.auth.external_namespace_policies |  | flattened |
| hashicorp_vault.audit.response.auth.identity_policies |  | keyword |
| hashicorp_vault.audit.response.auth.metadata |  | flattened |
| hashicorp_vault.audit.response.auth.no_default_policy |  | boolean |
| hashicorp_vault.audit.response.auth.num_uses |  | long |
| hashicorp_vault.audit.response.auth.policies |  | keyword |
| hashicorp_vault.audit.response.auth.token_issue_time |  | date |
| hashicorp_vault.audit.response.auth.token_policies |  | keyword |
| hashicorp_vault.audit.response.auth.token_ttl | Time to live for the token in seconds. | long |
| hashicorp_vault.audit.response.auth.token_type |  | keyword |
| hashicorp_vault.audit.response.data | Response payload. | flattened |
| hashicorp_vault.audit.response.headers | Headers will contain the http headers from the plugin that it wishes to have as part of the output. | flattened |
| hashicorp_vault.audit.response.mount_accessor |  | keyword |
| hashicorp_vault.audit.response.mount_type |  | keyword |
| hashicorp_vault.audit.response.redirect | Redirect is an HTTP URL to redirect to for further authentication. This is only valid for credential backends. This will be blanked for any logical backend and ignored. | keyword |
| hashicorp_vault.audit.response.warnings |  | keyword |
| hashicorp_vault.audit.response.wrap_info.accessor | The token accessor for the wrapped response token. | keyword |
| hashicorp_vault.audit.response.wrap_info.creation_path | Creation path is the original request path that was used to create the wrapped response. | keyword |
| hashicorp_vault.audit.response.wrap_info.creation_time | The creation time. This can be used with the TTL to figure out an expected expiration. | date |
| hashicorp_vault.audit.response.wrap_info.token | The token containing the wrapped response. | keyword |
| hashicorp_vault.audit.response.wrap_info.ttl | Specifies the desired TTL of the wrapping token. | long |
| hashicorp_vault.audit.response.wrap_info.wrapped_accessor | The token accessor for the wrapped response token. | keyword |
| hashicorp_vault.audit.type | Audit record type (request or response). | keyword |
| input.type |  | keyword |
| log.file.path | Full path to the log file this event came from, including the file name. It should include the drive letter, when appropriate. If the event wasn't read from a log file, do not populate this field. | keyword |
| log.offset |  | long |
| log.source.address | Source address (IP and port) of the log message. | keyword |
| message | For log events the message field contains the log message, optimized for viewing in a log viewer. For structured logs without an original message field, other fields can be concatenated to form a human-readable summary of the event. If multiple messages exist, they can be combined into one message. | match_only_text |
| nomad.allocation.id | Nomad allocation ID | keyword |
| nomad.namespace | Nomad namespace. | keyword |
| nomad.node.id | Nomad node ID. | keyword |
| nomad.task.name | Nomad task name. | keyword |
| related.ip | All of the IPs seen on your event. | ip |
| source.as.number | Unique number allocated to the autonomous system. The autonomous system number (ASN) uniquely identifies each network on the Internet. | long |
| source.as.organization.name | Organization name. | keyword |
| source.as.organization.name.text | Multi-field of `source.as.organization.name`. | match_only_text |
| source.geo.city_name | City name. | keyword |
| source.geo.continent_name | Name of the continent. | keyword |
| source.geo.country_iso_code | Country ISO code. | keyword |
| source.geo.country_name | Country name. | keyword |
| source.geo.location | Longitude and latitude. | geo_point |
| source.geo.region_iso_code | Region ISO code. | keyword |
| source.geo.region_name | Region name. | keyword |
| source.ip | IP address of the source (IPv4 or IPv6). | ip |
| source.port | Port of the source. | long |
| tags | List of keywords used to tag each event. | keyword |
| user.email | User email address. | keyword |
| user.id | Unique identifier of the user. | keyword |


### log

The `log` data stream provides events from Hashicorp Vault of the following types: operational logs.

Operational logs provide visibility into the behavior of the Vault server itself, including errors, warnings, and informational messages regarding the service state and backend storage.

An example event for `log` looks as following:

```json
{
    "@timestamp": "2023-09-26T13:09:08.587Z",
    "agent": {
        "ephemeral_id": "5bbd86cc-8032-432d-be82-fae8f624ed98",
        "id": "f25d13cd-18cc-4e73-822c-c4f849322623",
        "name": "docker-fleet-agent",
        "type": "filebeat",
        "version": "8.10.1"
    },
    "data_stream": {
        "dataset": "hashicorp_vault.log",
        "namespace": "ep",
        "type": "logs"
    },
    "ecs": {
        "version": "8.17.0"
    },
    "elastic_agent": {
        "id": "f25d13cd-18cc-4e73-822c-c4f849322623",
        "snapshot": false,
        "version": "8.10.1"
    },
    "event": {
        "agent_id_status": "verified",
        "dataset": "hashicorp_vault.log",
        "ingested": "2023-09-26T13:09:35Z",
        "kind": "event",
        "original": "{\"@level\":\"info\",\"@message\":\"proxy environment\",\"@timestamp\":\"2023-09-26T13:09:08.587324Z\",\"http_proxy\":\"\",\"https_proxy\":\"\",\"no_proxy\":\"\"}"
    },
    "hashicorp_vault": {
        "log": {
            "http_proxy": "",
            "https_proxy": "",
            "no_proxy": ""
        }
    },
    "host": {
        "architecture": "x86_64",
        "containerized": false,
        "hostname": "docker-fleet-agent",
        "id": "28da52b32df94b50aff67dfb8f1be3d6",
        "ip": [
            "192.168.80.5"
        ],
        "mac": [
            "02-42-C0-A8-50-05"
        ],
        "name": "docker-fleet-agent",
        "os": {
            "codename": "focal",
            "family": "debian",
            "kernel": "5.10.104-linuxkit",
            "name": "Ubuntu",
            "platform": "ubuntu",
            "type": "linux",
            "version": "20.04.6 LTS (Focal Fossa)"
        }
    },
    "input": {
        "type": "log"
    },
    "log": {
        "file": {
            "path": "/tmp/service_logs/log.json"
        },
        "level": "info",
        "offset": 709
    },
    "message": "proxy environment",
    "tags": [
        "preserve_original_event",
        "hashicorp-vault-log"
    ]
}
```

**Exported fields**

| Field | Description | Type |
|---|---|---|
| @timestamp | Date/time when the event originated. This is the date/time extracted from the event, typically representing when the event was generated by the source. If the event source has no original timestamp, this value is typically populated by the first time the event was received by the pipeline. Required field for all events. | date |
| data_stream.dataset | The field can contain anything that makes sense to signify the source of the data. Examples include `nginx.access`, `prometheus`, `endpoint` etc. For data streams that otherwise fit, but that do not have dataset set we use the value "generic" for the dataset value. `event.dataset` should have the same value as `data_stream.dataset`. Beyond the Elasticsearch data stream naming criteria noted above, the `dataset` value has additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |
| data_stream.namespace | A user defined namespace. Namespaces are useful to allow grouping of data. Many users already organize their indices this way, and the data stream naming scheme now provides this best practice as a default. Many users will populate this field with `default`. If no value is used, it falls back to `default`. Beyond the Elasticsearch index naming criteria noted above, `namespace` value has the additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |
| data_stream.type | An overarching type for the data stream. Currently allowed values are "logs" and "metrics". We expect to also add "traces" and "synthetics" in the near future. | constant_keyword |
| ecs.version | ECS version this event conforms to. `ecs.version` is a required field and must exist in all events. When querying across multiple indices -- which may conform to slightly different ECS versions -- this field lets integrations adjust to the schema version of the events. | keyword |
| event.dataset | Event dataset | constant_keyword |
| event.kind | This is one of four ECS Categorization Fields, and indicates the highest level in the ECS category hierarchy. `event.kind` gives high-level information about what type of information the event contains, without being specific to the contents of the event. For example, values of this field distinguish alert events from metric events. The value of this field can be used to inform how these kinds of events should be handled. They may warrant different retention, different access control, it may also help understand whether the data is coming in at a regular interval or not. | keyword |
| event.module | Event module | constant_keyword |
| event.original | Raw text message of entire event. Used to demonstrate log integrity or where the full log message (before splitting it up in multiple parts) may be required, e.g. for reindex. This field is not indexed and doc_values are disabled. It cannot be searched, but it can be retrieved from `_source`. If users wish to override this and index this field, please see `Field data types` in the `Elasticsearch Reference`. | keyword |
| file.path | Full path to the file, including the file name. It should include the drive letter, when appropriate. | keyword |
| file.path.text | Multi-field of `file.path`. | match_only_text |
| hashicorp_vault.log |  | flattened |
| input.type |  | keyword |
| log.file.path | Full path to the log file this event came from, including the file name. It should include the drive letter, when appropriate. If the event wasn't read from a log file, do not populate this field. | keyword |
| log.level | Original log level of the log event. If the source of the event provides a log level or textual severity, this is the one that goes in `log.level`. If your source doesn't specify one, you may put your event transport's severity here (e.g. Syslog severity). Some examples are `warn`, `err`, `i`, `informational`. | keyword |
| log.logger | The name of the logger inside an application. This is usually the name of the class which initialized the logger, or can be a custom name. | keyword |
| log.offset |  | long |
| message | For log events the message field contains the log message, optimized for viewing in a log viewer. For structured logs without an original message field, other fields can be concatenated to form a human-readable summary of the event. If multiple messages exist, they can be combined into one message. | match_only_text |
| tags | List of keywords used to tag each event. | keyword |


### metrics

The `metrics` data stream provides events from Hashicorp Vault of the following types: metrics.

This data stream collects telemetry data from the Vault `/v1/sys/metrics` endpoint, providing insights into performance, runtime statistics, and resource usage.

An example event for `metrics` looks as following:

```json
{
    "@timestamp": "2023-09-26T13:11:06.913Z",
    "agent": {
        "ephemeral_id": "3de3cc3a-6b9f-46c0-a268-200ac7dda214",
        "id": "f25d13cd-18cc-4e73-822c-c4f849322623",
        "name": "docker-fleet-agent",
        "type": "metricbeat",
        "version": "8.10.1"
    },
    "data_stream": {
        "dataset": "hashicorp_vault.metrics",
        "namespace": "ep",
        "type": "metrics"
    },
    "ecs": {
        "version": "8.17.0"
    },
    "elastic_agent": {
        "id": "f25d13cd-18cc-4e73-822c-c4f849322623",
        "snapshot": false,
        "version": "8.10.1"
    },
    "event": {
        "agent_id_status": "verified",
        "duration": 4669007,
        "ingested": "2023-09-26T13:11:09Z",
        "kind": "metric"
    },
    "hashicorp_vault": {
        "metrics": {
            "vault_barrier_estimated_encryptions": {
                "counter": 48,
                "rate": 0
            }
        }
    },
    "host": {
        "architecture": "x86_64",
        "containerized": false,
        "hostname": "docker-fleet-agent",
        "id": "28da52b32df94b50aff67dfb8f1be3d6",
        "ip": [
            "192.168.80.5"
        ],
        "mac": [
            "02-42-C0-A8-50-05"
        ],
        "name": "docker-fleet-agent",
        "os": {
            "codename": "focal",
            "family": "debian",
            "kernel": "5.10.104-linuxkit",
            "name": "Ubuntu",
            "platform": "ubuntu",
            "type": "linux",
            "version": "20.04.6 LTS (Focal Fossa)"
        }
    },
    "labels": {
        "host": "hashicorp_vault",
        "instance": "hashicorp_vault:8200",
        "job": "hashicorp_vault",
        "term": "1"
    },
    "metricset": {
        "period": 5000
    },
    "service": {
        "type": "hashicorp_vault"
    }
}
```

**Exported fields**

| Field | Description | Type | Metric Type |
|---|---|---|---|
| @timestamp | Date/time when the event originated. This is the date/time extracted from the event, typically representing when the event was generated by the source. If the event source has no original timestamp, this value is typically populated by the first time the event was received by the pipeline. Required field for all events. | date |  |
| agent.id | Unique identifier of this agent (if one exists). Example: For Beats this would be beat.id. | keyword |  |
| cloud.account.id | The cloud account or organization id used to identify different entities in a multi-tenant environment. Examples: AWS account id, Google Cloud ORG Id, or other unique identifier. | keyword |  |
| cloud.availability_zone | Availability zone in which this host, resource, or service is located. | keyword |  |
| cloud.instance.id | Instance ID of the host machine. | keyword |  |
| cloud.provider | Name of the cloud provider. Example values are aws, azure, gcp, or digitalocean. | keyword |  |
| cloud.region | Region in which this host, resource, or service is located. | keyword |  |
| container.id | Unique container id. | keyword |  |
| data_stream.dataset | The field can contain anything that makes sense to signify the source of the data. Examples include `nginx.access`, `prometheus`, `endpoint` etc. For data streams that otherwise fit, but that do not have dataset set we use the value "generic" for the dataset value. `event.dataset` should have the same value as `data_stream.dataset`. Beyond the Elasticsearch data stream naming criteria noted above, the `dataset` value has additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |  |
| data_stream.namespace | A user defined namespace. Namespaces are useful to allow grouping of data. Many users already organize their indices this way, and the data stream naming scheme now provides this best practice as a default. Many users will populate this field with `default`. If no value is used, it falls back to `default`. Beyond the Elasticsearch index naming criteria noted above, `namespace` value has the additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |  |
| data_stream.type | An overarching type for the data stream. Currently allowed values are "logs" and "metrics". We expect to also add "traces" and "synthetics" in the near future. | constant_keyword |  |
| ecs.version | ECS version this event conforms to. `ecs.version` is a required field and must exist in all events. When querying across multiple indices -- which may conform to slightly different ECS versions -- this field lets integrations adjust to the schema version of the events. | keyword |  |
| event.dataset | Event dataset | constant_keyword |  |
| event.duration | Duration of the event in nanoseconds. If `event.start` and `event.end` are known this value should be the difference between the end and start time. | long |  |
| event.kind | This is one of four ECS Categorization Fields, and indicates the highest level in the ECS category hierarchy. `event.kind` gives high-level information about what type of information the event contains, without being specific to the contents of the event. For example, values of this field distinguish alert events from metric events. The value of this field can be used to inform how these kinds of events should be handled. They may warrant different retention, different access control, it may also help understand whether the data is coming in at a regular interval or not. | keyword |  |
| event.module | Event module | constant_keyword |  |
| hashicorp_vault.metrics.\*.counter | Hashicorp Vault telemetry data from the Prometheus endpoint. | double | counter |
| hashicorp_vault.metrics.\*.histogram | Hashicorp Vault telemetry data from the Prometheus endpoint. | histogram |  |
| hashicorp_vault.metrics.\*.rate | Hashicorp Vault telemetry data from the Prometheus endpoint. | double | gauge |
| hashicorp_vault.metrics.\*.value | Hashicorp Vault telemetry data from the Prometheus endpoint. | double | gauge |
| host.name | Name of the host. It can contain what hostname returns on Unix systems, the fully qualified domain name (FQDN), or a name specified by the user. The recommended value is the lowercase FQDN of the host. | keyword |  |
| labels | Custom key/value pairs. Can be used to add meta information to events. Should not contain nested objects. All values are stored as keyword. Example: `docker` and `k8s` labels. | object |  |
| labels.auth_method | Authorization engine type. | keyword |  |
| labels.cluster | The cluster name from which the metric originated; set in the configuration file, or automatically generated when a cluster is created. | keyword |  |
| labels.creation_ttl | Time-to-live value assigned to a token or lease at creation. This value is rounded up to the next-highest bucket; the available buckets are 1m, 10m, 20m, 1h, 2h, 1d, 2d, 7d, and 30d. Any longer TTL is assigned the value +Inf. | keyword |  |
| labels.expiring |  | keyword |  |
| labels.gauge |  | keyword |  |
| labels.host |  | keyword |  |
| labels.instance |  | keyword |  |
| labels.job |  | keyword |  |
| labels.local |  | keyword |  |
| labels.mount_point | Path at which an auth method or secret engine is mounted. | keyword |  |
| labels.namespace | A namespace path, or root for the root namespace | keyword |  |
| labels.policy |  | keyword |  |
| labels.quantile |  | keyword |  |
| labels.queue_id |  | keyword |  |
| labels.term |  | keyword |  |
| labels.token_type | Identifies whether the token is a batch token or a service token. | keyword |  |
| labels.type |  | keyword |  |
| labels.version |  | keyword |  |
| service.address | Address where data about this service was collected from. This should be a URI, network address (ipv4:port or [ipv6]:port) or a resource path (sockets). | keyword |  |
| service.type | The type of the service data is collected from. The type can be used to group and correlate logs and metrics from one service type. Example: If logs or metrics are collected from Elasticsearch, `service.type` would be `elasticsearch`. | keyword |  |


### Inputs used

| Data Stream | Input Type | Description |
|-------------|------------|-------------|
| `audit` | `syslog` | Collects audit logs via syslog forwarding (recommended). |
| `audit` | `logfile` | Alternative method: monitors the audit log file directly. |
| `audit` | `tcp` | Alternative method: listens on a TCP socket for direct log streaming from Vault. |
| `log` | `logfile` | Monitors the operational log file. |
| `metrics` | `prometheus`| Scrapes the `/v1/sys/metrics` endpoint for Prometheus-formatted metrics. |

### API usage

The `metrics` data stream makes periodic HTTP GET requests to the `/v1/sys/metrics?format=prometheus` endpoint on the configured Vault server URL. The frequency of these requests is determined by the **Period** setting in the integration configuration. Be mindful of the load this may place on your Vault server, especially with very short collection intervals.

### Vendor documentation links

- [Blocked Audit Devices](https://developer.hashicorp.com/vault/docs/audit/socket#configuration)
- [Blocked Audit Device Behavior](https://developer.hashicorp.com/vault/tutorials/monitoring/blocked-audit-devices#blocked-audit-device-behavior)
- [HashiCorp Vault Deployment Guide](https://developer.hashicorp.com/vault/tutorials/day-one-raft/raft-deployment-guide)
- [HashiCorp Vault Audit Devices](https://developer.hashicorp.com/vault/docs/audit)
- [HashiCorp Vault File Audit Device](https://developer.hashicorp.com/vault/docs/audit/file)
- [HashiCorp Vault Socket Audit Device](https://developer.hashicorp.com/vault/docs/audit/socket)
- [HashiCorp Vault Telemetry Configuration](https://developer.hashicorp.com/vault/docs/configuration/telemetry)
- [HashiCorp Vault Troubleshooting](https://developer.hashicorp.com/vault/docs/troubleshoot)
- [Syslog - Audit Devices | Vault | HashiCorp Developer](https://developer.hashicorp.com/vault/docs/audit/syslog)
