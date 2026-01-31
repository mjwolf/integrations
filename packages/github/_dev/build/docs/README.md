# GitHub Integration for Elastic

> **Note**: This documentation was generated using AI and should be reviewed for accuracy.

## Overview

The GitHub integration for Elastic enables you to collect audit logs, security alerts, and project activity from GitHub Enterprise Cloud and Organizations. By ingesting this data, you gain centralized visibility into your software development lifecycle, security posture, and administrative actions. This integration helps security teams and administrators monitor activity across their entire codebase and collaborator base.

This integration facilitates:
- Security compliance auditing: Track administrative changes, repository access, and membership modifications to ensure compliance with security policies and regulatory requirements.
- Vulnerability management: Centralize security alerts from GitHub Advanced Security (GHAS), including Code Scanning, Secret Scanning, and Dependabot, to prioritize remediation efforts.
- Development operations monitoring: Track GitHub Issues and Pull Requests to gain insights into development velocity, project health, and team collaboration patterns.
- Threat detection: Use streamed audit logs to detect anomalous behaviors, such as unauthorized repository deletions or unusual login locations that may indicate a compromised account.

### Compatibility

The GitHub integration is compatible with the following deployment types:
- GitHub Enterprise Cloud (SaaS deployment)
- GitHub Organizations that are part of an enterprise plan with audit log access
- GitHub Advanced Security (GHAS), which is required for Code Scanning, Secret Scanning, and Dependabot alerts

This integration is not compatible with GitHub Enterprise Server (on-premise).

### How it works

This integration collects data from GitHub using several methods depending on the data stream and your environment:
- API collection: The integration primarily uses the GitHub REST API to pull logs, alerts, and metadata directly from GitHub.
- Cloud ingestion: For high-volume streaming environments, you can configure the integration to ingest audit logs from AWS S3, AWS SQS, Azure Event Hub, Azure Blob Storage, or Google Cloud Storage.

The Elastic Agent retrieves the data from these sources, processes the events, and sends them to your Elastic deployment where they can be monitored and analyzed.

## What data does this integration collect?

The GitHub integration collects log messages and event data from various GitHub features to help you monitor security, compliance, and development activities. You'll collect data through the following data streams:

*   GitHub Audit Logs (`audit`): Collects audit logs via the API to track user activity and administrative changes. You can also ingest these logs from cloud-native sources including AWS S3, AWS SQS, Azure Event Hub, Azure Blob Storage, and Google Cloud Storage.
*   GHAS Code Scanning (`code_scanning`): Collects GitHub Advanced Security alerts via the API, including vulnerability details and coding errors found in your repositories.
*   GHAS Secret Scanning (`secret_scanning`): Collects alerts that identify leaked credentials, tokens, or other secrets within your code.
*   GHAS Dependabot (`dependabot`): Collects alerts via the API to identify vulnerable dependencies in your projects.
*   GitHub Issues (`issues`): Collects issue events via the API to help you monitor project management and collaboration.
*   GitHub Security Advisories (`security_advisories`): Collects data from the GitHub REST API to provide a stream of global security vulnerabilities tracked by GitHub.

### Supported use cases

You can use the information collected by this integration to address several security and operational requirements:

*   Security auditing and compliance: Track user actions and administrative changes to meet regulatory compliance and maintain an audit trail of your GitHub organization or enterprise.
*   Vulnerability management: Identify and track vulnerabilities in your code and dependencies by centralizing alerts from Dependabot, code scanning, and global security advisories.
*   Secret leak prevention: Quickly respond to secret leaks detected by secret scanning to prevent unauthorized access to your systems and data.
*   Development workflow monitoring: Gain visibility into project management trends and team collaboration by analyzing issue events.
*   Cloud-native log aggregation: Stream high-volume audit logs through cloud storage or messaging services for long-term retention and multi-cloud security analysis.

## What do I need to use this integration?

To use this integration, you'll need to satisfy several vendor and Elastic-specific requirements.

### Vendor prerequisites

You must have specific permissions and plan levels depending on the GitHub environment you're monitoring:
- Administrative access:
    - For GitHub Enterprise Cloud, you must be an **Enterprise Owner**.
    - For GitHub Enterprise Server, you must be a **Site Administrator**.
    - For GitHub Organizations, you must be an **Organization Owner**.
- Plan requirements: Your GitHub plan must include audit log access (typically GitHub Enterprise) to use the `audit` data stream.
- Personal Access Token (PAT): You'll need to generate a PAT (Classic or Fine-grained) with scopes relevant to the data streams you enable:
    - `read:audit_log`
    - `security_events`
    - `repo`
    - `public_repo`
    - `admin:enterprise`
- Network connectivity: Your Elastic Agent host must have outbound HTTPS access (port `443`) to `api.github.com` or your internal GitHub Enterprise Server URL.
- Cloud resources: If you're using log streaming, ensure the destination (AWS S3, Azure Event Hub, or Google Cloud Storage) is provisioned and accessible to both GitHub and the Elastic Agent.

### Elastic prerequisites

Before you begin, ensure you have the following Elastic components ready:
- Elastic Agent: You must have an Elastic Agent installed and enrolled in Fleet on a host with access to GitHub's API or your chosen cloud storage provider.
- Elastic Stack version: You must be using Elastic Stack version `8.0.0` or later.
- Connectivity: The host running the Elastic Agent must have outbound HTTPS (port `443`) access to `https://api.github.com` or your respective cloud service endpoints.

## How do I deploy this integration?

### Agent-based deployment

Elastic Agent must be installed. For more details, check the Elastic Agent [installation instructions](https://www.elastic.co/guide/en/fleet/current/elastic-agent-installation.html). You can install only one Elastic Agent per host.

Elastic Agent is required to stream data from the syslog or log file receiver and ship the data to Elastic, where the events will then be processed via the integration's ingest pipelines.

To add the GitHub integration to your Elastic Agent policy:
1. In Kibana, navigate to **Management > Integrations**.
2. Search for `GitHub` and select the integration.
3. Click **Add GitHub**.
4. Configure the integration settings as described in the following sections.
5. Assign the integration to an Elastic Agent policy.
6. Click **Save and continue**.

### Set up steps in GitHub

You can configure GitHub to provide data to Elastic using several methods.

#### For API polling collection (PAT)
1. Log in to GitHub and navigate to **Settings** > **Developer settings** > **Personal access tokens** > **Tokens (classic)**.
2. Click **Generate new token (classic)** and provide a descriptive name.
3. Select the required scopes:
    - For Audit Logs: `read:audit_log` (Enterprise admins also need `admin:enterprise`).
    - For Code/Secret Scanning: `security_events` and `repo`.
    - For Issues: `repo` or specific read permissions.
4. Click **Generate token** and copy the value immediately.
5. In the Elastic integration configuration, paste this token into the **Personal Access Token** field.

#### For audit log streaming (AWS S3)
1. Log in as an Enterprise Owner and navigate to your enterprise settings.
2. Select **Settings** > **Audit Log** > **Log streaming**.
3. Choose **Amazon S3** and enter your **Bucket Name** and **Region**.
4. Provide authentication via **Access keys** or **OIDC** credentials.
5. Click **Check endpoint** to confirm GitHub can write to the bucket, then click **Save**.
6. Ensure the Elastic Agent is configured with the same AWS credentials and bucket details to ingest the data.

#### For audit log streaming (Azure Event Hubs)
1. In your Azure Portal, create an **Event Hubs Namespace** and an **Event Hub**.
2. Create a **Shared Access Policy** with `Listen` and `Send` permissions.
3. In GitHub Enterprise settings, go to **Log streaming** > **Azure Event Hubs**.
4. Enter the **Instance Name** and the **Connection String** from your Azure policy.
5. Save the configuration and verify the stream is active.

#### Vendor resources

For more information, refer to the following GitHub documentation:
- [Streaming the audit log for your enterprise](https://docs.github.com/en/enterprise-cloud@latest/admin/monitoring-activity-in-your-enterprise/reviewing-audit-logs-for-your-enterprise/streaming-the-audit-log-for-your-enterprise)
- [Managing your personal access tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)

### Set up steps in Kibana

Follow these steps to configure the integration inputs in Kibana. Choose the configuration that matches the data collection method you set up in GitHub.

#### Collecting logs from GitHub using AWS S3 or AWS SQS
1. In Kibana, navigate to the **Collect logs from GitHub using AWS S3 or AWS SQS** section.
2. Configure the following variables:
    * **Collect logs via S3 Bucket** (`collect_s3_logs`): Enable the toggle to collect via S3 bucket; by default, it collects via SQS.
    * **Access Key ID** (`access_key_id`): First part of your AWS access key.
    * **Secret Access Key** (`secret_access_key`): Second part of your AWS access key.
    * **[SQS] Region** (`region`): The AWS region of the endpoint.
    * **Session Token** (`session_token`): Required when using temporary security credentials.
    * **[S3] Bucket ARN** (`bucket_arn`): ARN of the S3 bucket to be polled.
    * **[S3] Bucket Prefix** (`bucket_list_prefix`): Prefix to apply for the list request to the S3 bucket.
    * **[S3] Interval** (`interval`): Listing poll interval (default `120s`).
    * **Number of Workers** (`number_of_workers`): Number of workers processing S3/SQS objects (default `5`).
    * **[SQS] Queue URL** (`queue_url`): URL of the SQS queue to receive messages from.
    * **[SQS] Visibility Timeout** (`visibility_timeout`): Duration messages are hidden after retrieval (default `300s`).
    * **[SQS] API Timeout** (`api_timeout`): Maximum duration of AWS API calls (default `120s`).
    * **Preserve original event** (`preserve_original_event`): Preserves a raw copy in `event.original`.
    * **Shared Credential File** (`shared_credential_file`): Directory of the shared credentials file.
    * **Credential Profile Name** (`credential_profile_name`): Profile name in the shared credentials file.
    * **Role ARN** (`role_arn`): AWS IAM Role to assume.
    * **Default AWS Region** (`default_region`): Default region if no other region is set.
    * **Endpoint** (`endpoint`): URL of the entry point for an AWS web service.
    * **FIPS Enabled** (`fips_enabled`): Use `s3-fips` service endpoints (default `false`).
    * **[SQS] File Selectors** (`file_selectors`): List of selectors (regex) to limit processed files.
    * **External ID** (`external_id`): External ID for assuming a role.
    * **Tags** (`tags`): Tags to include (default `['forwarded', 'github.audit']`).
    * **Preserve duplicate custom fields** (`preserve_duplicate_custom_fields`): Preserve fields copied to ECS.
    * **Processors** (`processors`): Custom processors for the agent.
    * **Proxy URL** (`proxy_url`): URL to proxy connections.
    * **SSL Configuration** (`ssl`): SSL configuration options.

#### Collecting logs from GitHub via API (Code Scanning)
1. Navigate to the **Collect GitHub logs via API** input for the `code_scanning` stream.
2. Configure the following variables:
    * **Personal Access Token** (`access_token`): The GitHub PAT with required scopes.
    * **Repository owner** (`owner`): The owner of the GitHub Repository (Organization or User).
    * **Repository** (`repo`): Specific repository name; if empty, all repositories of the owner are ingested.
    * **HTTP Client Timeout** (`http_client_timeout`): Connection timeout (default `60s`).
    * **Interval** (`interval`): Interval at which alerts are pulled (default `10m`).
    * **Tags** (`tags`): Default: `['forwarded', 'github-code-scanning']`.
    * **Preserve original event** (`preserve_original_event`): Adds raw copy to `event.original`.
    * **API URL** (`api_url`): The API URL without path (default `https://api.github.com`).
    * **SSL Configuration** (`ssl`): SSL options for the HTTP client.
    * **Proxy URL** (`proxy_url`): URL for proxying connections.
    * **Processors** (`processors`): Logic for filtering or enhancing data.

#### Collecting logs from GitHub via API (Secret Scanning)
1. Navigate to the **Collect GitHub logs via API** input for the `secret_scanning` stream.
2. Configure the following variables:
    * **Personal Access Token** (`access_token`): The GitHub PAT with `admin` or `security_events` scope.
    * **Repository owner** (`owner`): The owner of the GitHub Repository.
    * **Repository** (`repo`): The specific GitHub Repository name.
    * **Hide Secret** (`hide_secret`): Set to `false` to reveal the full secret in alerts (default `true`).
    * **HTTP Client Timeout** (`http_client_timeout`): Connection timeout (default `60s`).
    * **Interval** (`interval`): Pull interval (default `10m`).
    * **Tags** (`tags`): Default: `['forwarded', 'github-secret-scanning']`.
    * **Preserve original event** (`preserve_original_event`): Adds raw copy to `event.original`.
    * **API URL** (`api_url`): The API URL without path (default `https://api.github.com`).
    * **SSL Configuration** (`ssl`): SSL options.
    * **Proxy URL** (`proxy_url`): Proxy server URL.
    * **Processors** (`processors`): Custom processing logic.

#### Collecting logs from GitHub via API (Audit Logs)
1. Navigate to the **Collect GitHub logs via API** input for the `audit` stream.
2. Configure the following variables:
    * **Personal Access Token** (`access_token`): Requires `read:audit_log` scope.
    * **Organization Name** (`organization`): The GitHub organization name or ID.
    * **Enterprise Name** (`enterprise`): The GitHub enterprise name or ID.
    * **Initial Interval** (`initial_interval`): Initial polling lookback duration (default `730h`).
    * **HTTP Client Timeout** (`http_client_timeout`): Connection timeout (default `60s`).
    * **Interval** (`interval`): Interval at which logs are pulled (default `1h`).
    * **Tags** (`tags`): Default: `['forwarded', 'github-audit']`.
    * **Preserve original event** (`preserve_original_event`): Adds raw copy to `event.original`.
    * **API URL** (`api_url`): The API URL without path (default `https://api.github.com`).
    * **SSL Configuration** (`ssl`): SSL options.
    * **Proxy URL** (`proxy_url`): Proxy server URL.
    * **Processors** (`processors`): Custom processing logic.

#### Collecting logs from GitHub via API (Issues)
1. Navigate to the **Collect GitHub logs via API** input for the `issues` stream.
2. Configure the following variables:
    * **Personal Access Token** (`access_token`): The GitHub PAT.
    * **Repository owner** (`owner`): The owner of the GitHub Repository.
    * **Repository** (`repo`): The GitHub Repository name.
    * **State** (`state`): Indicates the state of issues to return (default `all`).
    * **Filter** (`filter`): Sorts of issues to return: `assigned`, `created`, `mentioned`, `subscribed`, `repos`, `all` (default `all`).
    * **Labels** (`labels`): Comma-separated list of label names (e.g., `bug,ui`).
    * **Since** (`since`): Timestamp in ISO 8601 format to filter by update time.
    * **HTTP Client Timeout** (`http_client_timeout`): Connection timeout (default `60s`).
    * **Interval** (`interval`): Pull interval (default `10m`).
    * **Tags** (`tags`): Default: `['forwarded', 'github-issues']`.
    * **Preserve original event** (`preserve_original_event`): Adds raw copy to `event.original`.
    * **API URL** (`api_url`): The API URL without path (default `https://api.github.com`).
    * **SSL Configuration** (`ssl`): SSL options.
    * **Proxy URL** (`proxy_url`): Proxy server URL.
    * **Processors** (`processors`): Custom processing logic.

#### Collecting logs from GitHub via API (Dependabot)
1. Navigate to the **Collect GitHub logs via API** input for the `dependabot` stream.
2. Configure the following variables:
    * **Personal Access Token** (`access_token`): The GitHub PAT for GraphQL authentication.
    * **Repository owner** (`owner`): The owner of the GitHub Repository.
    * **Repository** (`repo`): The specific GitHub Repository.
    * **HTTP Client Timeout** (`http_client_timeout`): Connection timeout (default `60s`).
    * **Interval** (`interval`): Pull interval (default `10m`).
    * **Tags** (`tags`): Default: `['forwarded', 'github-dependabot']`.
    * **Preserve original event** (`preserve_original_event`): Adds raw copy to `event.original`.
    * **API URL** (`api_url`): The API URL without path (default `https://api.github.com`).
    * **SSL Configuration** (`ssl`): SSL options.
    * **Proxy URL** (`proxy_url`): Proxy server URL.
    * **Processors** (`processors`): Custom processing logic.

#### Collect GitHub logs from Azure Event Hub
1. Select the **Collect GitHub logs from Azure Event Hub** input.
2. Configure the following variables:
    * **Event Hub** (`eventhub`): The name of the Azure Event Hub.
    * **Consumer Group** (`consumer_group`): Dedicated consumer group (default `$Default`).
    * **Connection String** (`connection_string`): The Azure Event Hubs connection string.
    * **Storage Account** (`storage_account`): Account name for state/offset storage.
    * **Storage Account Key** (`storage_account_key`): Key to authorize access to the storage account.
    * **Storage Account Container** (`storage_account_container`): Container for checkpoint data.
    * **Resource Manager Endpoint** (`resource_manager_endpoint`): Custom resource manager endpoint.
    * **Preserve original event** (`preserve_original_event`): Adds raw copy to `event.original`.
    * **Tags** (`tags`): Default: `['forwarded', 'github-audit']`.
    * **Preserve duplicate custom fields** (`preserve_duplicate_custom_fields`): Keep fields copied to ECS.
    * **Processors** (`processors`): Custom processing logic.

#### Collect GitHub logs from Azure Blob Storage
1. Select the **Collect GitHub logs from Azure Blob Storage.** input.
2. Configure the following variables:
    * **Account Name** (`account_name`): Azure storage account name.
    * **Collect logs using OAuth2 authentication** (`oauth2`): Enable to use OAuth2 (default `false`).
    * **Client ID (OAuth2)** (`client_id`): Required if OAuth2 is enabled.
    * **Client Secret (OAuth2)** (`client_secret`): Required if OAuth2 is enabled.
    * **Tenant ID (OAuth2)** (`tenant_id`): Required if OAuth2 is enabled.
    * **Service Account Key** (`service_account_key`): Azure access key from the storage account.
    * **Maximum number of workers** (`number_of_workers`): Workers per container (default `3`).
    * **Polling** (`poll`): Continuous polling for new documents (default `true`).
    * **Polling interval** (`poll_interval`): Interval between polls (default `15s`).
    * **Containers** (`containers`): YAML list of container names and specific overrides.
    * **Service Account URI** (`service_account_uri`): Azure connection string.
    * **Storage URL** (`storage_url`): Custom storage URL format.
    * **File Selectors** (`file_selectors`): Regex patterns to limit files processed locally.
    * **Timestamp Epoch** (`timestamp_epoch`): Filter files older than this unix epoch value.
    * **Expand Event List From Field** (`expand_event_list_from_field`): Split bundled messages into events.
    * **Preserve original event** (`preserve_original_event`): Adds raw copy to `event.original`.
    * **Tags** (`tags`): Default: `['forwarded', 'github.audit']`.
    * **Preserve duplicate custom fields** (`preserve_duplicate_custom_fields`): Keep original fields.
    * **Processors** (`processors`): Custom processing logic.

#### Collect GitHub logs from Google Cloud Storage
1. Select the **Collect GitHub logs from Google Cloud Storage.** input.
2. Configure the following variables:
    * **Project Id** (`project_id`): Your Google Cloud project ID (default `my-project-id`).
    * **Credentials JSON Key** (`service_account_key`): JSON service account credentials string.
    * **Maximum number of workers** (`number_of_workers`): Workers per bucket (default `3`).
    * **Polling** (`poll`): Continuous polling for new documents (default `true`).
    * **Polling Interval** (`poll_interval`): Interval between polls (default `15s`).
    * **Buckets** (`buckets`): YAML list of bucket names and overrides.
    * **Alternative Host** (`alternative_host`): Override for default storage host.
    * **Credentials File Path** (`service_account_file`): Path to the service account credentials file.
    * **File Selectors** (`file_selectors`): Regex patterns to filter bucket files locally.
    * **Timestamp Epoch** (`timestamp_epoch`): Filter objects older than this unix epoch value.
    * **Expand Event List From Field** (`expand_event_list_from_field`): Split bundled messages into events.
    * **Preserve original event** (`preserve_original_event`): Adds raw copy to `event.original`.
    * **Tags** (`tags`): Default: `['forwarded', 'github.audit']`.
    * **Preserve duplicate custom fields** (`preserve_duplicate_custom_fields`): Keep original fields.
    * **Processors** (`processors`): Custom processing logic.

#### Collect GitHub Security Advisories data via API
1. Select the **Collect GitHub Security Advisories data via API.** input.
2. Configure the following variables:
    * **API key** (`api_key`): Personal Access Token for GitHub REST API authentication.
    * **Advisory type** (`advisory_type`): The type of security advisory to collect.
    * **Interval** (`interval`): Duration between API requests (default `24h`).
    * **API URL** (`api_url`): URL for the GitHub Security Advisories REST API (default `https://api.github.com/advisories`).
    * **Batch Size** (`batch_size`): Results per API response (max `100`).
    * **Enable request tracing** (`enable_request_tracer`): Logs requests for debugging (default `false`).
    * **Preserve original event** (`preserve_original_event`): Adds raw copy to `event.original`.
    * **Tags** (`tags`): Default: `['forwarded', 'github-security-advisories']`.
    * **Processors** (`processors`): Custom processing logic.

### Validation

After configuration is complete, verify that data is flowing correctly.

#### Trigger data flow on GitHub
You can trigger data flow by performing the following actions:
* **Audit Log activity**: In your organization, rename a test repository or update its description to generate an audit event.
* **Security Alert activity**: If GHAS is enabled, push a commit with a dummy secret or a known vulnerable dependency to trigger a Secret Scanning or Dependabot alert.
* **Issue activity**: Create a new issue in a monitored repository with the title `Elastic Integration Validation`.
* **Administrative activity**: Modify a user's permissions or add a new collaborator to a repository.

#### Check data in Kibana
Follow these steps to confirm data is appearing in Elastic:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "github.audit"`.
4. Verify logs appear in the table. Expand a log entry and confirm these fields:
    * `event.dataset` (should match `github.audit`)
    * `github.organization` or `github.repository`
    * `event.action` (e.g., `repo.create` or `issue.opened`)
    * `message` (the raw log payload)
5. Navigate to **Analytics > Dashboards** and search for `GitHub` to view the pre-built dashboards for Audit and Security events.

## Troubleshooting

For help with Elastic ingest tools, check the [common problems documentation](https://www.elastic.co/docs/troubleshoot/ingest/fleet/common-problems).

### Common configuration issues

You may encounter the following issues when you configure the GitHub integration:

- 404 Not Found errors: This often occurs if you use fine-grained personal access tokens that don't have the required read permissions for `Metadata` and `Issues`. Ensure both are set to at least `Read-only`.
- Empty data streams: If the integration's running but you don't see any data, verify your token scopes. For enterprise logs, the `admin:enterprise` scope is mandatory in addition to `read:audit_log`.
- Organization issue limit: The GitHub API has a limit of 30,000 issues when you query at the organization level. If your organization exceeds this, logs might stop or appear incomplete. You'll need to switch to repository-level polling for large organizations.
- Streaming verification: If cloud-streamed audit logs aren't appearing, use the `Check endpoint` button in the GitHub log streaming UI to confirm GitHub can reach your AWS, Azure, or GCP destination.
- Parsing failures: Check the `error.message` field in Kibana Discover. This often happens if GitHub updates their JSON schema and the integration's ingest pipeline requires an update.
- Token expiration: GitHub personal access tokens can expire. Check your Elastic Agent logs for `401 Unauthorized` errors and refresh the token in the integration settings if you need to.
- Rate limiting: If you see `403 Forbidden` errors related to rate limits, increase the `Interval` setting in your configuration (for example, from `10m` to `30m`) to reduce how often the agent polls the API.

### Vendor resources

For more information about GitHub's API and audit logging, refer to the following resources:

- [GitHub REST API documentation](https://docs.github.com/en/rest)
- [GitHub App permissions](https://docs.github.com/en/rest/authentication/permissions-required-for-github-apps?apiVersion=latest)
- [Organization audit log actions](https://docs.github.com/en/organizations/keeping-your-organization-secure/reviewing-the-audit-log-for-your-organization#audit-log-actions)
- [Streaming the audit log for your enterprise](https://docs.github.com/en/enterprise-cloud@latest/admin/monitoring-activity-in-your-enterprise/reviewing-audit-logs-for-your-enterprise/streaming-the-audit-log-for-your-enterprise)
- [Managing your personal access tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)

## Performance and scaling

For more information on architectures that can be used for scaling this integration, check the [Ingest Architectures](https://www.elastic.co/docs/manage-data/ingest/ingest-reference-architectures) documentation.

To ensure optimal performance and reliability when you monitor GitHub at scale, consider the following:
- Transport and collection considerations: For standard monitoring of repositories and issues, API polling through `httpjson` is efficient. However, for high-volume GitHub Enterprise Cloud environments that generate massive audit trails, you should use log streaming to intermediate storage like AWS S3, Azure Event Hub, or Google Cloud Storage. This avoids GitHub API rate limits and ensures reliable delivery by decoupling your collection from GitHub's real-time API availability.
- Data volume management: You can manage event volume by using configuration filters. For issues, you can specify the `state` (for example, `open`) or `labels` to ignore high-frequency noise. For audit logs, streaming allows for pre-filtering at the cloud provider level. You'll want to tune `interval` settings to balance data freshness against API quota consumption; for low-change data like Security Advisories, you can use the default `24h` interval.
- Elastic Agent scaling: For enterprise environments with thousands of repositories, you should distribute collection across multiple Elastic Agents. Assign high-volume cloud streaming inputs, such as AWS SQS or Azure Event Hub, to dedicated agents with higher CPU and memory allocations to handle the ingestion throughput and parsing overhead.

## Reference

### Inputs used

{{ inputDocs }}

### API usage

The GitHub integration interacts with the following APIs to collect data:
*   [GitHub REST API](https://docs.github.com/en/rest) — Used as the primary interface for fetching audit logs, security alerts, issues, and advisory data.
*   [Organization audit log API](https://docs.github.com/en/rest/orgs/audit-log) — Provides access to the activity logs for organizations.
*   [Enterprise audit log API](https://docs.github.com/en/rest/enterprise-admin/audit-log) — Provides access to the activity logs for enterprises.
*   [GitHub Advanced Security APIs](https://docs.github.com/en/rest/code-scanning) — Used to retrieve code scanning, secret scanning, and Dependabot alerts.

### Data streams

#### audit

The `audit` data stream provides events from the GitHub Audit Log, which includes actions performed by members of your organization or enterprise. This data stream supports the following:
*   Access control changes
*   Repository creation and deletion
*   Team and member management
*   Security setting updates

##### audit fields

{{ fields "audit" }}

##### audit sample event

{{ event "audit" }}

#### code_scanning

The `code_scanning` data stream provides events from GitHub Advanced Security (GHAS) code scanning alerts. It helps you monitor:
*   Vulnerabilities found in your code
*   Alert status changes (open, closed, or dismissed)
*   Tool-specific findings from engines like CodeQL

##### code_scanning fields

{{ fields "code_scanning" }}

##### code_scanning sample event

{{ event "code_scanning" }}

#### dependabot

The `dependabot` data stream provides events from GitHub Advanced Security (GHAS) Dependabot alerts. This stream includes information about:
*   Vulnerable dependencies in your repositories
*   Security advisory details related to dependencies
*   Remediation status of dependency alerts

##### dependabot fields

{{ fields "dependabot" }}

##### dependabot sample event

{{ event "dependabot" }}

#### issues

The `issues` data stream provides events from GitHub Issues. It tracks activities such as:
*   Issue creation and closing
*   Label assignments
*   Milestone updates
*   Comments and reactions

##### issues fields

{{ fields "issues" }}

##### issues sample event

{{ event "issues" }}

#### secret_scanning

The `secret_scanning` data stream provides events from GitHub Advanced Security (GHAS) secret scanning. It monitors your repositories for:
*   Exposed secrets like API keys, tokens, and certificates
*   Alert status and location of found secrets
*   Resolution and bypass actions

##### secret_scanning fields

{{ fields "secret_scanning" }}

##### secret_scanning sample event

{{ event "secret_scanning" }}

#### security_advisories

The `security_advisories` data stream collects data from the GitHub Advisory Database using the GitHub REST API. This stream provides:
*   Global security advisories for open-source projects
*   Severity levels and CVSS scores
*   Affected versions and patched versions for vulnerabilities

##### security_advisories fields

{{ fields "security_advisories" }}

##### security_advisories sample event

{{ event "security_advisories" }}
