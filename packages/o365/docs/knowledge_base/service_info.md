# Service Info

## Common use cases
The Microsoft Office 365 integration for Elastic allows organizations to ingest and analyze activity data from across the Microsoft 365 ecosystem, providing visibility into user actions, administrative changes, and security events.
- **Security Monitoring and Threat Detection:** Monitor for unauthorized access attempts, suspicious login activity in Microsoft Entra ID (Azure AD), and unusual administrative privilege escalations. This includes tracking password resets, multi-factor authentication (MFA) changes, and account locking.
- **Data Loss Prevention (DLP) Oversight:** Collect and analyze DLP events to identify when sensitive information is shared or accessed in violation of corporate policies across Exchange and SharePoint. This helps identify internal data leaks or external exfiltration attempts.
- **Compliance and Auditing:** Maintain a long-term record of all system and user actions required for regulatory compliance (such as GDPR, HIPAA, or SOC2) by centralizing Office 365 audit logs. This ensures that historical data is available for forensic investigations.
- **Service Administration Tracking:** Track changes to mailbox permissions in Exchange Online, file sharing activities in SharePoint, and configuration modifications in the Microsoft 365 tenant. Monitor who accessed which documents and when permissions were escalated.

## Data types collected
This integration can collect the following types of data:
- **Microsoft Office 365 audit logs:** Comprehensive records of user and admin activities across various O365 services, collected via the Office 365 Management Activity API.
- **Authentication Logs:** Detailed records of sign-in attempts, MFA challenges, and application access events from Azure Active Directory/Microsoft Entra ID.
- **Content-Specific Logs:** Activity logs for specific workloads including SharePoint file operations, Exchange mailbox access, and General O365 service events.
- **DLP Events:** Specialized logs detailing matches against Data Loss Prevention policies across all supported O365 services.

This integration includes the following data streams:
- **Collect audit logs (logs):** This data stream collects Microsoft Office 365 audit logs using the modern Common Expression Language (CEL) input. It provides rich telemetry including user actions, system events, and policy changes formatted as JSON objects.
- **DEPRECATED - Collect audit logs (logs):** A legacy data stream used for older installations. Please deactivate this option and instead use the one described above. This option collects audit logs using the Management Activity API through a deprecated method.

## Compatibility
The Microsoft Office 365 integration is designed to be compatible with:
- **Microsoft Office 365 Management Activity API v1.0**
- **Microsoft Entra ID (formerly Azure Active Directory)**
- **Microsoft Purview (for Audit and DLP features)**
- Supported workloads include **Audit.AzureActiveDirectory**, **Audit.Exchange**, **Audit.SharePoint**, **Audit.General**, and **DLP.All**.

## Scaling and Performance

To ensure optimal performance in high-volume environments, consider the following:
- **Transport/Collection Considerations:** This integration relies on API polling via the Office 365 Management Activity API. Configure the **Interval** (default `3m`) and **Batch Interval** (default `1h`) to balance data freshness with API rate limits. Be aware that Microsoft documents a latency of up to 12 hours for some data to become available for collection; adjusting the interval too low may result in empty API responses if the data is not yet published by Microsoft.
- **Data Volume Management:** Use the **Content Type** configuration to filter for only required workloads at the source. For example, if identity data is the priority, collecting only `Audit.AzureActiveDirectory` significantly reduces volume compared to enabling all types. Additionally, use the **Processors** setting to drop unnecessary fields or entire events at the Elastic Agent level before they are transmitted to the Elastic Stack.
- **Elastic Agent Scaling:** For high-throughput environments or large tenants with thousands of users, a single Agent may reach resource limits. Deploy multiple Elastic Agents and distribute the workload by configuring different Agent policies to collect specific **Content Types** or by assigning different Agents to monitor different **Directory (tenant) IDs**.

# Set Up Instructions

## Vendor prerequisites
- **Microsoft 365 Tenant Admin Access:** Required to enable auditing and grant admin consent for API permissions.
- **Audit Logging Enabled:** Audit search must be turned on in the Microsoft Purview portal for logs to be generated.
- **Microsoft Entra ID App Registration:** An application must be registered in the Azure Portal to provide the necessary OAuth 2.0 credentials (**Application (client) ID**, **Directory (tenant) ID**, and **Client Secret**).
- **API Permissions:** The registered application must be granted `ActivityFeed.Read` (and optionally `ActivityFeed.ReadDlp`) permissions under the Office 365 Management APIs.
- **Network Connectivity:** The Elastic Agent must have outbound HTTPS access (port 443) to `manage.office.com` and `login.microsoftonline.com`.

## Elastic prerequisites
- **Elastic Agent:** An active Elastic Agent must be installed and enrolled in Fleet.
- **Connectivity:** Secure network communication must be established between the Elastic Agent and the Elastic Stack (Elasticsearch and Kibana).

## Vendor set up steps

### For API-based Collection (Standard):

1. **Register Application in Microsoft Entra ID:**
   - Log in to the [Azure Portal](https://portal.azure.com).
   - Navigate to **Microsoft Entra ID** > **App registrations** > **New registration**.
   - Name it (e.g., `Elastic-O365`) and select **Accounts in this organizational directory only (Single tenant)**. Click **Register**.
   - Save the **Application (client) ID** and **Directory (tenant) ID**.

2. **Configure API Permissions:**
   - In the App menu, go to **API permissions** > **Add a permission**.
   - Select **Office 365 Management APIs** > **Application permissions**.
   - Expand **ActivityFeed** and check `ActivityFeed.Read` and `ActivityFeed.ReadDlp`.
   - Click **Add permissions**, then click **Grant admin consent for [Your Org]** and confirm.

3. **Create Client Secret:**
   - Go to **Certificates & secrets** > **New client secret**.
   - Provide a description and expiration, then click **Add**.
   - **Immediately copy the Secret Value** (not the Secret ID).

4. **Enable Auditing in Microsoft Purview:**
   - Access the [Microsoft Purview portal](https://purview.microsoft.com).
   - Navigate to **Solutions** > **Audit**.
   - If auditing is not enabled, click the banner to **Start recording user and admin activity**. (Note: Activation may take up to 48 hours).

### Vendor Set up Resources

- [Get started with Office 365 Management APIs - Microsoft Learn](https://learn.microsoft.com/en-us/office/office-365-management-api/get-started-with-office-365-management-apis) - Official Microsoft onboarding documentation.

## Kibana set up steps

### Collect audit logs using the Management Activity API
1. In Kibana, navigate to **Management > Integrations** and search for **Microsoft Office 365**.
2. Click **Add Microsoft Office 365** and select an Elastic Agent policy.
3. Configure the following fields for the **Collect audit logs using the Management Activity API** input:
   - **Base URL of Office Management API** (`url`): The base endpoint for the API. Default: `https://manage.office.com`.
   - **Interval** (`interval`): How often the API is polled. Default: `3m`.
   - **Directory (tenant) ID** (`azure_tenant_id`): The unique Directory (tenant) ID from the Entra ID portal.
   - **Application (client) ID** (`client_id`): Client ID used for OAuth 2.0 authentication.
   - **Client Secret** (`client_secret`): Client secret used for OAuth 2.0 authentication.
   - **OAuth 2.0 Token URL** (`token_url`): Base URL endpoint for generating tokens. Default: `https://login.microsoftonline.com`.
   - **Content Type** (`content_types`): Comma separated list of content types to fetch. Supported: `Audit.AzureActiveDirectory, Audit.Exchange, Audit.SharePoint, Audit.General, DLP.All`.
   - **Initial Interval** (`initial_interval`): Initial interval for the first API call (max 7 days). Default: `167h55m`.
   - **Batch Interval** (`batch_interval`): Interval for each API request (max 24h). Default: `1h`.
   - **Preserve original event** (`preserve_original_event`): Preserves a raw copy of the original event in `event.original`. Default: `False`.
   - **Token Scopes** (`token_scopes`): Scopes for OAuth 2.0 token. Default: `['https://manage.office.com/.default']`.
   - **OAuth2 Endpoint Params** (`oauth_endpoint_params`): Endpoint Params used for OAuth2 authentication as YAML. Default: `grant_type: client_credentials`.
   - **Maximum Executions Per Interval** (`max_executions`): Max number of executions without waiting for interval. Default: `10000`.
   - **Maximum Age** (`maximum_age`): Hard maximum age limit for requested data. Default: `167h55m`.
   - **Resource SSL Configuration** (`resource_ssl`): SSL configuration including `certificate_authorities` or `verification_mode`.
   - **Resource Timeout** (`resource_timeout`): Duration before HTTP client connection times out. Default: `60s`.
   - **Resource Proxy** (`resource_proxy_url`): Proxy configuration in the form `http[s]://<user>:<password>@<server>:<port>`.
   - **Resource Retry Max Attempts** (`resource_retry_max_attempts`): Max retries for the HTTP client. Default: `5`.
   - **Resource Retry Wait Min** (`resource_retry_wait_min`): Minimum time to wait before a retry. Default: `1s`.
   - **Resource Retry Wait Max** (`resource_retry_wait_max`): Maximum time to wait before a retry. Default: `60s`.
   - **Resource Redirect Forward Headers** (`resource_redirect_forward_headers`): Forward headers in case of a redirect. Default: `false`.
   - **Resource Redirect Headers Ban List** (`resource_redirect_headers_ban_list`): Headers that will NOT be forwarded if redirect forwarding is enabled.
   - **Resource Redirect Max Redirects** (`resource_redirect_max_redirects`): Max number of redirects to follow. Default: `10`.
   - **Resource Rate Limit** (`resource_rate_limit_limit`): Value of the response that specifies the total limit.
   - **Resource Rate Limit Burst** (`resource_rate_limit_burst`): The maximum burst size for resource requests.
   - **Processors** (`processors`): Used to reduce fields or enhance events with metadata in the agent.
   - **Tags** (`tags`): Custom tags for the events. Default: `['forwarded', 'o365-cel']`.
   - **Enable request tracing** (`enable_request_tracer`): Logs requests and responses for debugging. Enabling this compromises security. Default: `False`.
4. Click **Save and continue**.

### Please deactivate this option and instead use the one described above. This option collects audit logs using the Management Activity API through a deprecated method.
1. Use this input only if you have a legacy configuration requirement.
2. Configure the following fields:
   - **Application (client) ID** (`application_id`): The Client ID of the registered Entra ID application.
   - **Client secret (API key)** (`client_secret`): The secret value generated in the Entra ID portal.
   - **Path to certificate file** (`certificate`): File path to the certificate for authentication.
   - **Path to private key file** (`key`): File path to the private key.
   - **Private key passphrase** (`key_passphrase`): The passphrase for the private key.
   - **Directory (tenant) IDs** (`tenants`): List of tenant IDs. Default: `['tenant-id']`.
   - **Directory (tenant) domains mapping** (`tenant_names`): Mapping of tenant IDs to domain names.
   - **Content types** (`content_type`): List of workloads to collect. Default: `['Audit.AzureActiveDirectory', 'Audit.Exchange', 'Audit.SharePoint', 'Audit.General', 'DLP.All']`.
   - **Preserve original event** (`preserve_original_event`): Preserves the raw event in `event.original`. Default: `False`.
   - **Advanced API settings** (`api`): Advanced YAML configuration for the API.
   - **Tags** (`tags`): Custom tags. Default: `['forwarded', 'o365-audit']`.
   - **Processors** (`processors`): YAML configuration for agent-side processors.
3. Click **Save and continue**.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from Microsoft Office 365 to the Elastic Stack.

### 1. Trigger Data Flow on Microsoft Office 365:
- **Generate Administrative Event:** Create a new test user or modify an existing user's group membership within the **Microsoft Entra admin center**.
- **Generate Exchange Event:** Send a test email between two accounts within the tenant or modify a mailbox permission setting in the **Exchange admin center**.
- **Generate SharePoint Event:** Upload a new document to a **SharePoint** site or modify the sharing permissions of an existing file to trigger a `FileUploaded` or `FileModified` event.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "o365.audit"`
4. Verify logs appear. Expand a log entry and confirm these fields:
   - `event.dataset` (should be `o365.audit`)
   - `o365.audit.UserId` (confirm it contains the UPN of the user who triggered the event)
   - `event.action` (confirm it matches the action performed, e.g., `Add user` or `FileDownloaded`)
   - `message` (confirm the raw JSON payload is present)
5. Navigate to **Analytics > Dashboards** and search for "Microsoft Office 365" to verify pre-built visualizations are populating.

# Troubleshooting

## Common Configuration Issues

- **Initial Latency**: When a subscription is first enabled, there is a documented Microsoft API delay of up to 12 hours. Do not troubleshoot connectivity until this window has passed.
- **Admin Consent Missing**: If permissions are added in Azure but "Grant admin consent" was not clicked, the API will return 403 Forbidden errors. Ensure the status in the Azure Portal shows "Granted".
- **Auditing Not Enabled**: If the Purview Audit log is not turned on, the API will return empty results even if the integration is correctly configured. Verify auditing status in the Microsoft Purview portal.
- **CEL Migration Errors**: If upgrading from a version older than 1.18.0, ensure the Elastic Stack is at least 8.7.1. Certificate-based authentication is not supported in the newer CEL input.

## Ingestion Errors
- **CEL Parsing Failures**: If events are not appearing or have `error.message` fields, check the Agent logs for CEL expression errors. This usually indicates a change in the upstream JSON schema.
- **Missing Fields**: Some fields may only exist for specific workloads (e.g., `o365.audit.FileName` only exists for SharePoint/OneDrive). Check `event.dataset` to ensure you are looking at the correct event type.
- **Request Tracing**: Enable **Enable request tracing** in the Kibana integration settings to log raw requests and responses to the agent's filesystem. Use this only for debugging as it can log sensitive data.

## API Authentication Errors
- **Token URL Mismatch**: Ensure the `OAuth 2.0 Token URL` matches your Azure environment (e.g., `https://login.microsoftonline.com`).
- **Expired Secret**: Verify the Client Secret in Azure has not expired. Expired secrets will cause the OAuth flow to fail with "invalid_client".
- **ADAL vs MSAL**: The legacy `o365audit` input used ADAL, which is being retired. If you encounter authentication failures on the deprecated input, migrate immediately to the CEL input.

## Vendor Resources

- [Office 365 Management Activity API Reference](https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-reference)
- [Audit Log Record Type Documentation](https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-schema#auditlogrecordtype)
- [Microsoft Purview Audit Log Setup](https://learn.microsoft.com/en-us/purview/audit-log-enable-disable)

# Documentation sites
- [Office 365 Management Activity API Reference](https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-reference)
- [Enable Audit Log Search](https://learn.microsoft.com/en-us/purview/audit-log-enable-disable)
- [Register an App in Microsoft Entra ID](https://learn.microsoft.com/en-us/entra/identity-platform/quickstart-register-app)
- [Elastic CEL Input Documentation](https://www.elastic.co/docs/reference/beats/filebeat/filebeat-input-cel)
