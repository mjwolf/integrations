# Service Info

## Common use cases

The GitHub integration for Elastic enables organizations to centralize visibility into their software development lifecycle, security posture, and administrative actions. By ingesting data from GitHub Enterprise Cloud and Organizations, security teams and administrators can monitor activity across their entire codebase and collaborator base.

*   **Security Compliance Auditing:** Monitor GitHub Audit Logs to track administrative changes, repository access, and membership modifications to ensure compliance with internal security policies and regulatory requirements.
*   **Vulnerability Management:** Centralize security alerts from GitHub Advanced Security (GHAS), including Code Scanning, Secret Scanning, and Dependabot, to prioritize remediation efforts and identify high-risk repositories.
*   **Development Operations Monitoring:** Track GitHub Issues and Pull Requests to gain insights into development velocity, project health, and team collaboration patterns across repositories or entire organizations.
*   **Threat Detection:** Use streamed audit logs to detect anomalous behaviors, such as unauthorized repository deletions, unusual login locations, or bulk permission changes that may indicate a compromised account.

## Data types collected

This integration collects the following data types:
- **GHAS Code Scanning (logs):** Collect GitHub Advanced Security Code Scanning alerts via the API, including vulnerability details and coding errors found in repositories.
- **GHAS Secret Scanning (logs):** Collect GitHub Advanced Security Secret Scanning alerts via the API, identifying leaked credentials or secrets within the code.
- **GitHub Audit Logs (logs):** Collect GitHub audit logs via the API to track user activity and administrative changes.
- **GitHub Audit Logs from Azure Event Hub (logs):** Collect GitHub audit logs from Azure Event Hub for high-volume streaming environments.
- **Collect Audit logs via AWS S3 or SQS (logs):** Collect Audit logs via AWS S3 or SQS input for cloud-native log ingestion.
- **GitHub Audit Logs from Azure Blob Storage (logs):** Collect GitHub audit logs from Azure Blob Storage. This data stream ingests logs stored in Azure containers to provide visibility into GitHub enterprise actions.
- **GitHub Audit Logs from Google Cloud Storage (logs):** Collect GitHub audit logs from Google Cloud Storage. This data stream ingests audit logs stored in GCS buckets for long-term retention or multi-cloud security analysis.
- **GitHub Security Advisories data (logs):** Collect GitHub Security Advisories data from the GitHub REST API. This provides a stream of global security vulnerabilities tracked by GitHub.
- **GitHub Issues (logs):** Collect GitHub issues as events via the API to monitor project management and collaboration.
- **GHAS Dependabot (logs):** Collect GitHub Advanced Security Dependabot alerts via the API to identify vulnerable dependencies.

## Compatibility

The **GitHub** integration is compatible with the following deployment types:
- **GitHub Enterprise Cloud** (SaaS/Cloud deployment).
- **GitHub Organizations** (part of an enterprise plan with audit log access).
- **GitHub Advanced Security (GHAS):** Required for specific data streams including Code Scanning, Secret Scanning, and Dependabot alerts.
- **Note:** This integration is **not compatible** with GitHub Enterprise Server (on-premise).

## Scaling and Performance

To ensure optimal performance and reliability when monitoring GitHub at scale, consider the following:

*   **Transport/Collection Considerations:** For standard monitoring of repositories and issues, API polling via `httpjson` is efficient. However, for high-volume GitHub Enterprise Cloud environments generating massive audit trails, using log streaming to intermediate storage like **AWS S3**, **Azure Event Hub**, or **Google Cloud Storage** is recommended. This avoids GitHub API rate limits and ensures reliable delivery by decoupling the collection from GitHub's real-time API availability.
*   **Data Volume Management:** Manage event volume by using configuration filters. For issues, you can specify the **State** (e.g., `open`) or **Labels** to ignore high-frequency noise. For audit logs, streaming allows for pre-filtering at the cloud provider level. Ensure **Interval** settings are tuned to balance data freshness against API quota consumption; for low-change data like Security Advisories, use the default `24h` interval.
*   **Elastic Agent Scaling:** For enterprise environments with thousands of repositories, distribute collection across multiple Elastic Agents. Assign high-volume cloud streaming inputs (like AWS SQS or Azure Event Hub) to dedicated agents with higher CPU and memory allocations to handle the ingestion throughput and parsing overhead.

# Set Up Instructions

## Vendor prerequisites

1. **Administrative Access:** 
    - For Enterprise Cloud: You must be an **Enterprise Owner**.
    - For Enterprise Server: You must be a **Site Administrator**.
    - For Organizations: You must be an **Organization Owner**.
2. **Plan Requirements:** A GitHub plan that includes audit log access (typically GitHub Enterprise) is required for audit log data streams.
3. **Personal Access Token (PAT):** Generate a PAT (Classic or Fine-grained) with specific scopes: `read:audit_log`, `security_events`, `repo`, `public_repo`, or `admin:enterprise` depending on the data stream being collected.
4. **Network Connectivity:** Ensure the Elastic Agent has outbound HTTPS access (port 443) to `api.github.com` or the internal GitHub Enterprise Server URL.
5. **Cloud Resources (Optional):** If using log streaming, the destination (AWS S3, Azure Event Hub, GCS) must be provisioned and accessible to both GitHub and the Elastic Agent.

## Elastic prerequisites

1.  **Elastic Agent:** An Elastic Agent must be installed and enrolled in Fleet on a host with internet access to GitHub's API or your chosen cloud storage provider.
2.  **Elastic Stack Version:** Ensure you are using a supported version of Kibana and Elasticsearch (typically version 8.0 or later) that meets the integration's requirements.
3.  **Connectivity:** The agent host must have outbound HTTPS (port 443) access to `https://api.github.com` or the respective cloud endpoints (e.g., `s3.amazonaws.com`).

## Vendor set up steps

### For API Polling Collection (PAT):
1. Log in to GitHub and navigate to **Settings** > **Developer settings** > **Personal access tokens** > **Tokens (classic)**.
2. Click **Generate new token (classic)** and provide a descriptive name.
3. Select the required scopes:
   - For Audit Logs: `read:audit_log` (Enterprise admins also need `admin:enterprise`).
   - For Code/Secret Scanning: `security_events` and `repo`.
   - For Issues: `repo` or specific read permissions.
4. Click **Generate token** and copy the value immediately.
5. In the Elastic integration configuration, paste this token into the **Personal Access Token** field.

### For Audit Log Streaming (AWS S3):
1. Log in as an Enterprise Owner and navigate to your enterprise settings.
2. Select **Settings** > **Audit Log** > **Log streaming**.
3. Choose **Amazon S3** and enter your **Bucket Name** and **Region**.
4. Provide authentication via **Access keys** or **OIDC** credentials.
5. Click **Check endpoint** to confirm GitHub can write to the bucket, then click **Save**.
6. Ensure the Elastic Agent is configured with the same AWS credentials and bucket details to ingest the data.

### For Audit Log Streaming (Azure Event Hubs):
1. In your Azure Portal, create an **Event Hubs Namespace** and an **Event Hub**.
2. Create a **Shared Access Policy** with `Listen` and `Send` permissions.
3. In GitHub Enterprise settings, go to **Log streaming** > **Azure Event Hubs**.
4. Enter the **Instance Name** and the **Connection String** from your Azure policy.
5. Save the configuration and verify the stream is active.

### Vendor Set up Resources

- [Streaming the audit log for your enterprise - GitHub Enterprise Cloud Docs](https://docs.github.com/en/enterprise-cloud@latest/admin/monitoring-activity-in-your-enterprise/reviewing-audit-logs-for-your-enterprise/streaming-the-audit-log-for-your-enterprise)
- [Managing your personal access tokens - GitHub Docs](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)

## Kibana set up steps

Follow these steps to configure the integration inputs in Kibana.

### Collecting logs from GitHub using AWS S3 or AWS SQS
1.  In Kibana, navigate to **Management > Integrations** and search for **GitHub**.
2.  Click **Add GitHub** and find the **Collect logs from GitHub using AWS S3 or AWS SQS** section.
3.  Configure the following variables:
    *   **Collect logs via S3 Bucket** (`collect_s3_logs`) - Enable the toggle to collect via S3 bucket; by default, it collects via SQS.
    *   **Access Key ID** (`access_key_id`) - First part of your AWS access key.
    *   **Secret Access Key** (`secret_access_key`) - Second part of your AWS access key.
    *   **[SQS] Region** (`region`) - The AWS region of the endpoint.
    *   **Session Token** (`session_token`) - Required when using temporary security credentials.
    *   **[S3] Bucket ARN** (`bucket_arn`) - ARN of the S3 bucket to be polled.
    *   **[S3] Bucket Prefix** (`bucket_list_prefix`) - Prefix to apply for the list request to the S3 bucket.
    *   **[S3] Interval** (`interval`) - Listing poll interval (default `120s`).
    *   **Number of Workers** (`number_of_workers`) - Number of workers processing S3/SQS objects (default `5`).
    *   **[SQS] Queue URL** (`queue_url`) - URL of the SQS queue to receive messages from.
    *   **[SQS] Visibility Timeout** (`visibility_timeout`) - Duration messages are hidden after retrieval (default `300s`).
    *   **[SQS] API Timeout** (`api_timeout`) - Maximum duration of AWS API calls (default `120s`).
    *   **Preserve original event** (`preserve_original_event`) - Preserves a raw copy in `event.original`.
    *   **Shared Credential File** (`shared_credential_file`) - Directory of the shared credentials file.
    *   **Credential Profile Name** (`credential_profile_name`) - Profile name in the shared credentials file.
    *   **Role ARN** (`role_arn`) - AWS IAM Role to assume.
    *   **Default AWS Region** (`default_region`) - Default region if no other region is set.
    *   **Endpoint** (`endpoint`) - URL of the entry point for an AWS web service.
    *   **FIPS Enabled** (`fips_enabled`) - Use `s3-fips` service endpoints (default `false`).
    *   **[SQS] File Selectors** (`file_selectors`) - List of selectors (regex) to limit processed files.
    *   **External ID** (`external_id`) - External ID for assuming a role.
    *   **Tags** (`tags`) - Tags to include (default `['forwarded', 'github.audit']`).
    *   **Preserve duplicate custom fields** (`preserve_duplicate_custom_fields`) - Preserve fields copied to ECS.
    *   **Processors** (`processors`) - Custom processors for the agent.
    *   **Proxy URL** (`proxy_url`) - URL to proxy connections.
    *   **SSL Configuration** (`ssl`) - SSL configuration options.

### Collecting logs from GitHub via API (Code Scanning)
1.  Navigate to the **Collect GitHub logs via API** input for the **GHAS Code Scanning** stream.
2.  Configure the following variables:
    *   **Personal Access Token** (`access_token`) - The GitHub PAT with required scopes.
    *   **Repository owner** (`owner`) - The owner of the GitHub Repository (Organization or User).
    *   **Repository** (`repo`) - Specific repository name; if empty, all repositories of the owner are ingested.
    *   **HTTP Client Timeout** (`http_client_timeout`) - Connection timeout (default `60s`).
    *   **Interval** (`interval`) - Interval at which alerts are pulled (default `10m`).
    *   **Tags** (`tags`) - Default: `['forwarded', 'github-code-scanning']`.
    *   **Preserve original event** (`preserve_original_event`) - Adds raw copy to `event.original`.
    *   **API URL.** (`api_url`) - The API URL without path (default `https://api.github.com`).
    *   **SSL Configuration** (`ssl`) - SSL options for the HTTP client.
    *   **Proxy URL** (`proxy_url`) - URL for proxying connections.
    *   **Processors** (`processors`) - Logic for filtering or enhancing data.

### Collecting logs from GitHub via API (Secret Scanning)
1.  Navigate to the **Collect GitHub logs via API** input for the **GHAS Secret Scanning** stream.
2.  Configure the following variables:
    *   **Personal Access Token** (`access_token`) - The GitHub PAT with `admin` or `security_events` scope.
    *   **Repository owner** (`owner`) - The owner of the GitHub Repository.
    *   **Repository** (`repo`) - The specific GitHub Repository name.
    *   **Hide Secret** (`hide_secret`) - Set to `false` to reveal the full secret in alerts (default `true`).
    *   **HTTP Client Timeout** (`http_client_timeout`) - Connection timeout (default `60s`).
    *   **Interval** (`interval`) - Pull interval (default `10m`).
    *   **Tags** (`tags`) - Default: `['forwarded', 'github-secret-scanning']`.
    *   **Preserve original event** (`preserve_original_event`) - Adds raw copy to `event.original`.
    *   **API URL.** (`api_url`) - The API URL without path (default `https://api.github.com`).
    *   **SSL Configuration** (`ssl`) - SSL options.
    *   **Proxy URL** (`proxy_url`) - Proxy server URL.
    *   **Processors** (`processors`) - Custom processing logic.

### Collecting logs from GitHub via API (Audit Logs)
1.  Navigate to the **Collect GitHub logs via API** input for the **GitHub Audit Logs** stream.
2.  Configure the following variables:
    *   **Personal Access Token** (`access_token`) - Requires `read:audit_log` scope.
    *   **Organization Name** (`organization`) - The GitHub organization name or ID.
    *   **Enterprise Name** (`enterprise`) - The GitHub enterprise name or ID.
    *   **Initial Interval** (`initial_interval`) - Initial polling lookback duration (default `730h`).
    *   **HTTP Client Timeout** (`http_client_timeout`) - Connection timeout (default `60s`).
    *   **Interval** (`interval`) - Interval at which logs are pulled (default `1h`).
    *   **Tags** (`tags`) - Default: `['forwarded', 'github-audit']`.
    *   **Preserve original event** (`preserve_original_event`) - Adds raw copy to `event.original`.
    *   **API URL.** (`api_url`) - The API URL without path (default `https://api.github.com`).
    *   **SSL Configuration** (`ssl`) - SSL options.
    *   **Proxy URL** (`proxy_url`) - Proxy server URL.
    *   **Processors** (`processors`) - Custom processing logic.

### Collecting logs from GitHub via API (Issues)
1.  Navigate to the **Collect GitHub logs via API** input for the **GitHub Issues** stream.
2.  Configure the following variables:
    *   **Personal Access Token** (`access_token`) - The GitHub PAT.
    *   **Repository owner** (`owner`) - The owner of the GitHub Repository.
    *   **Repository** (`repo`) - The GitHub Repository name.
    *   **State** (`state`) - Indicates the state of issues to return (default `all`).
    *   **Filter** (`filter`) - Sorts of issues to return: `assigned`, `created`, `mentioned`, `subscribed`, `repos`, `all` (default `all`).
    *   **Labels** (`labels`) - Comma-separated list of label names (e.g., `bug,ui`).
    *   **Since** (`since`) - Timestamp in ISO 8601 format to filter by update time.
    *   **HTTP Client Timeout** (`http_client_timeout`) - Connection timeout (default `60s`).
    *   **Interval** (`interval`) - Pull interval (default `10m`).
    *   **Tags** (`tags`) - Default: `['forwarded', 'github-issues']`.
    *   **Preserve original event** (`preserve_original_event`) - Adds raw copy to `event.original`.
    *   **API URL.** (`api_url`) - The API URL without path (default `https://api.github.com`).
    *   **SSL Configuration** (`ssl`) - SSL options.
    *   **Proxy URL** (`proxy_url`) - Proxy server URL.
    *   **Processors** (`processors`) - Custom processing logic.

### Collecting logs from GitHub via API (Dependabot)
1.  Navigate to the **Collect GitHub logs via API** input for the **GHAS Dependabot** stream.
2.  Configure the following variables:
    *   **Personal Access Token** (`access_token`) - The GitHub PAT for GraphQL authentication.
    *   **Repository owner** (`owner`) - The owner of the GitHub Repository.
    *   **Repository** (`repo`) - The specific GitHub Repository.
    *   **HTTP Client Timeout** (`http_client_timeout`) - Connection timeout (default `60s`).
    *   **Interval** (`interval`) - Pull interval (default `10m`).
    *   **Tags** (`tags`) - Default: `['forwarded', 'github-dependabot']`.
    *   **Preserve original event** (`preserve_original_event`) - Adds raw copy to `event.original`.
    *   **API URL.** (`api_url`) - The API URL without path (default `https://api.github.com`).
    *   **SSL Configuration** (`ssl`) - SSL options.
    *   **Proxy URL** (`proxy_url`) - Proxy server URL.
    *   **Processors** (`processors`) - Custom processing logic.

### Collect GitHub logs from Azure Event Hub
1.  Select the **Collect GitHub logs from Azure Event Hub** input.
2.  Configure the following variables:
    *   **Event Hub** (`eventhub`) - The name of the Azure Event Hub.
    *   **Consumer Group** (`consumer_group`) - Dedicated consumer group (default `$Default`).
    *   **Connection String** (`connection_string`) - The Azure Event Hubs connection string.
    *   **Storage Account** (`storage_account`) - Account name for state/offset storage.
    *   **Storage Account Key** (`storage_account_key`) - Key to authorize access to the storage account.
    *   **Storage Account Container** (`storage_account_container`) - Container for checkpoint data.
    *   **Resource Manager Endpoint** (`resource_manager_endpoint`) - Custom resource manager endpoint.
    *   **Preserve original event** (`preserve_original_event`) - Adds raw copy to `event.original`.
    *   **Tags** (`tags`) - Default: `['forwarded', 'github-audit']`.
    *   **Preserve duplicate custom fields** (`preserve_duplicate_custom_fields`) - Keep fields copied to ECS.
    *   **Processors** (`processors`) - Custom processing logic.

### Collect GitHub logs from Azure Blob Storage
1.  Select the **Collect GitHub logs from Azure Blob Storage.** input.
2.  Configure the following variables:
    *   **Account Name** (`account_name`) - Azure storage account name.
    *   **Collect logs using OAuth2 authentication** (`oauth2`) - Enable to use OAuth2 (default `false`).
    *   **Client ID (OAuth2)** (`client_id`) - Required if OAuth2 is enabled.
    *   **Client Secret (OAuth2)** (`client_secret`) - Required if OAuth2 is enabled.
    *   **Tenant ID (OAuth2)** (`tenant_id`) - Required if OAuth2 is enabled.
    *   **Service Account Key** (`service_account_key`) - Azure access key from the storage account.
    *   **Maximum number of workers** (`number_of_workers`) - Workers per container (default `3`).
    *   **Polling** (`poll`) - Continuous polling for new documents (default `true`).
    *   **Polling interval** (`poll_interval`) - Interval between polls (default `15s`).
    *   **Containers** (`containers`) - YAML list of container names and specific overrides.
    *   **Service Account URI** (`service_account_uri`) - Azure connection string.
    *   **Storage URL** (`storage_url`) - Custom storage URL format.
    *   **File Selectors** (`file_selectors`) - Regex patterns to limit files processed locally.
    *   **Timestamp Epoch** (`timestamp_epoch`) - Filter files older than this unix epoch value.
    *   **Expand Event List From Field** (`expand_event_list_from_field`) - Split bundled messages into events.
    *   **Preserve original event** (`preserve_original_event`) - Adds raw copy to `event.original`.
    *   **Tags** (`tags`) - Default: `['forwarded', 'github.audit']`.
    *   **Preserve duplicate custom fields** (`preserve_duplicate_custom_fields`) - Keep original fields.
    *   **Processors** (`processors`) - Custom processing logic.

### Collect GitHub logs from Google Cloud Storage
1.  Select the **Collect GitHub logs from Google Cloud Storage.** input.
2.  Configure the following variables:
    *   **Project Id** (`project_id`) - Your Google Cloud project ID (default `my-project-id`).
    *   **Credentials JSON Key** (`service_account_key`) - JSON service account credentials string.
    *   **Maximum number of workers** (`number_of_workers`) - Workers per bucket (default `3`).
    *   **Polling** (`poll`) - Continuous polling for new documents (default `true`).
    *   **Polling Interval** (`poll_interval`) - Interval between polls (default `15s`).
    *   **Buckets** (`buckets`) - YAML list of bucket names and overrides.
    *   **Alternative Host** (`alternative_host`) - Override for default storage host.
    *   **Credentials File Path** (`service_account_file`) - Path to the service account credentials file.
    *   **File Selectors** (`file_selectors`) - Regex patterns to filter bucket files locally.
    *   **Timestamp Epoch** (`timestamp_epoch`) - Filter objects older than this unix epoch value.
    *   **Expand Event List From Field** (`expand_event_list_from_field`) - Split bundled messages into events.
    *   **Preserve original event** (`preserve_original_event`) - Adds raw copy to `event.original`.
    *   **Tags** (`tags`) - Default: `['forwarded', 'github.audit']`.
    *   **Preserve duplicate custom fields** (`preserve_duplicate_custom_fields`) - Keep original fields.
    *   **Processors** (`processors`) - Custom processing logic.

### Collect GitHub Security Advisories data via API
1.  Select the **Collect GitHub Security Advisories data via API.** input.
2.  Configure the following variables:
    *   **API key** (`api_key`) - Personal Access Token for GitHub REST API authentication.
    *   **Advisory type** (`advisory_type`) - The type of security advisory to collect.
    *   **Interval** (`interval`) - Duration between API requests (default `24h`).
    *   **API URL** (`api_url`) - URL for the GitHub Security Advisories REST API (default `https://api.github.com/advisories`).
    *   **Batch Size** (`batch_size`) - Results per API response (max `100`).
    *   **Enable request tracing** (`enable_request_tracer`) - Logs requests for debugging (default `false`).
    *   **Preserve original event** (`preserve_original_event`) - Adds raw copy to `event.original`.
    *   **Tags** (`tags`) - Default: `['forwarded', 'github-security-advisories']`.
    *   **Processors** (`processors`) - Custom processing logic.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on GitHub:
*   **Audit Log activity:** In your organization, rename a test repository or update its description to generate an audit event.
*   **Security Alert activity:** If GHAS is enabled, push a commit with a dummy secret or a known vulnerable dependency to trigger a Secret Scanning or Dependabot alert.
*   **Issue activity:** Create a new issue in a monitored repository with the title "Elastic Integration Validation".
*   **Administrative activity:** Modify a user's permissions or add a new collaborator to a repository.

### 2. Check Data in Kibana:
1.  Navigate to **Analytics > Discover**.
2.  Select the `logs-*` data view.
3.  Enter the KQL filter: `data_stream.dataset : "github.audit"`
4.  Verify logs appear in the table. Expand a log entry and confirm these fields:
    *   `event.dataset` (should match `github.audit`)
    *   `github.organization` or `github.repository`
    *   `event.action` (e.g., `repo.create` or `issue.opened`)
    *   `message` (the raw log payload)
5.  Navigate to **Analytics > Dashboards** and search for "GitHub" to view the pre-built dashboards for Audit and Security events.

# Troubleshooting

## Common Configuration Issues

- **404 Not Found Errors**: This often occurs when using fine-grained Personal Access Tokens that lack the necessary read permissions for "Metadata" and "Issues". Ensure both are set to at least "Read-only".
- **Empty Datastreams**: If the integration is running but no data is appearing, verify the PAT scopes. For enterprise logs, the `admin:enterprise` scope is mandatory in addition to `read:audit_log`.
- **Organization Issue Limit**: The GitHub API has a hard limit of 30,000 issues when querying at the organization level. If your organization exceeds this, logs may stop or appear incomplete. Switch to repository-level polling for large orgs.
- **Streaming Verification**: If cloud-streamed audit logs are not appearing, use the "Check endpoint" button in the GitHub Log streaming UI to ensure GitHub can successfully reach your AWS/Azure/GCP destination.

## Ingestion Errors

- **Parsing Failures**: Check the `error.message` field in Kibana Discover. This often occurs if GitHub updates their JSON schema and the integration's ingest pipeline requires an update.
- **Zero Data Processing**: If `github.issues.is_pr` is being used for filtering, ensure that the data actually contains pull requests. If the integration runs but returns no results, check if the `State` parameter is too restrictive (e.g., searching for `closed` issues in a repository with only `open` issues).

## API Authentication Errors

- **Token Expiration**: GitHub Personal Access Tokens can expire. Check the Elastic Agent logs for `401 Unauthorized` errors and refresh the token in the Kibana integration settings if necessary.
- **Rate Limiting**: If you encounter `403 Forbidden` errors related to rate limits, increase the `Interval` setting in the Kibana configuration (e.g., from 10m to 30m) to reduce polling frequency.

## Vendor Resources

*   [GitHub REST API Documentation](https://docs.github.com/en/rest)
*   [GitHub App Permissions](https://docs.github.com/en/rest/authentication/permissions-required-for-github-apps?apiVersion=latest)
*   [Organization audit log actions](https://docs.github.com/en/organizations/keeping-your-organization-secure/reviewing-the-audit-log-for-your-organization#audit-log-actions)

# Documentation sites

*   [GitHub REST API Documentation](https://docs.github.com/en/rest)
*   [Streaming the audit log for your enterprise](https://docs.github.com/en/enterprise-cloud@latest/admin/monitoring-activity-in-your-enterprise/reviewing-audit-logs-for-your-enterprise/streaming-the-audit-log-for-your-enterprise)
