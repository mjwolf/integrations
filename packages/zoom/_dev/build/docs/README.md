# Zoom Integration for Elastic

## Overview
The Zoom integration for Elastic enables you to collect and analyze logs and events from your Zoom account. By leveraging Zoom's webhooks, this integration provides real-time visibility into activities such as meetings, webinars, recordings, and user management.

This integration facilitates several key use cases, including:
*   **Security Monitoring**: Track security-relevant events like changes to meeting settings, password updates, and user deactivations.
*   **Operational Insight**: Monitor meeting health, participant activity, and overall usage of the Zoom platform.
*   **Compliance Auditing**: Maintain a detailed log of user activities and system events for audit and compliance purposes.

### How it works
This integration operates by setting up an HTTP endpoint on the Elastic Agent. You configure a "Webhook-only" application within your Zoom account to send event notifications to this secure endpoint. The Elastic Agent then receives these webhooks, processes them, and sends them to your Elastic deployment for analysis and visualization.

## What data does this integration collect?
The Zoom integration collects event data based on the event subscriptions you configure in your Zoom webhook application. This can include a wide range of events, such as:
*   Meeting events (e.g., `meeting.started`, `meeting.ended`, `meeting.participant_joined`)
*   User events (e.g., `user.created`, `user.updated`, `user.deleted`)
*   Recording events (e.g., `recording.started`, `recording.completed`)
*   Webinar events (e.g., `webinar.started`, `webinar.ended`)
*   Account events (e.g., `account.updated`)

For a full list of available webhook events, refer to the [official Zoom webhook reference](https://developers.zoom.us/docs/api/rest/webhook-reference/#events).

## What do I need to use this integration?
*   A Zoom account with administrator privileges to create and manage applications in the Zoom App Marketplace.
*   An Elastic Agent installed on a host that is publicly accessible over the internet, so it can receive webhook callbacks from Zoom.
*   A valid TLS/SSL certificate to secure the communication between Zoom and the Elastic Agent. Zoom requires all webhook endpoints to use HTTPS.

## How do I deploy this integration?

### Agent-based deployment

Elastic Agent must be installed on a publicly accessible server. For more details, check the Elastic Agent [installation instructions](docs-content://reference/fleet/install-elastic-agents.md). You can install only one Elastic Agent per host.

The Elastic Agent will create an HTTP listener to receive webhooks from Zoom and forward them to Elastic.

### Onboard / configure

Configuring the Zoom integration is a two-part process: first, you configure the webhook in Zoom, and then you configure the integration in Kibana.

#### 1. Configure Zoom Webhook

1.  **Log in to the Zoom App Marketplace**: Navigate to the [Zoom App Marketplace](https://marketplace.zoom.us/) and sign in with an administrator account.
2.  **Create a new app**: In the **Develop** dropdown menu, select **Build App**. Choose **Webhook Only** as the app type.
3.  **Provide App Information**: Fill in the required application details, such as the App Name.
4.  **Configure Event Subscription**:
    *   Navigate to the **Feature** tab.
    *   Toggle on **Event Subscription**.
    *   Click **Add new event subscription**.
    *   For the **Event notification endpoint URL**, you will need to construct the URL where your Elastic Agent is listening. This URL must be publicly accessible and use HTTPS. The format is `https://<your_elastic_agent_host>:<port>/<webhook_path>`.
        *   `<your_elastic_agent_host>`: The public IP address or hostname of your Elastic Agent.
        *   `<port>`: The port you will configure in the Elastic integration settings (e.g., `8181`).
        *   `<webhook_path>`: The URL path you will configure in the Elastic integration settings (e.g., `/zoom-webhook`).
5.  **Validate the Endpoint (Optional but Recommended)**:
    *   Zoom provides a mechanism to validate your endpoint URL upon configuration. To do this, you must temporarily enable `CRC validation` in the integration settings in Kibana and provide the **Secret Token** from Zoom.
    *   Once validated, you can choose to disable CRC validation and rely on the custom header verification method below for ongoing security.
6.  **Secure Your Webhook (Recommended)**:
    *   To ensure that the events received by the agent are genuinely from Zoom, it is critical to use a verification method.
    *   In the **Event Subscription** feature page in Zoom, under **Add custom header**, you can define a custom header and a secret value. For example:
        *   **Header Key**: `Authorization`
        *   **Header Value**: A secure, randomly generated secret value.
    *   You will use these same values in the integration settings in Kibana.
7.  **Subscribe to Events**:
    *   Click **Add events**.
    *   Select all the events you want to monitor. Each event you subscribe to will be sent to your Elastic Agent.
8.  **Activate the App**: Once configured, ensure your app is activated.

#### 2. Configure the Integration in Elastic

1.  In Kibana, navigate to **Management > Integrations**.
2.  In the search bar, type **Zoom** and select the integration.
3.  Click **Add Zoom**.
4.  Configure the integration settings:
    *   **Listen Address**: Set this to `0.0.0.0` to allow the agent to listen for traffic from any network interface.
    *   **Listen Port**: Enter the port number you specified in the Zoom endpoint URL (e.g., `8181`).
    *   **Webhook path**: Enter the URL path you specified in the Zoom endpoint URL (e.g., `/zoom-webhook`).
    *   **Zoom Custom Header**: Enter the header key you configured in Zoom (e.g., `Authorization`).
    *   **Zoom Custom Header value**: Enter the secret value you configured for the custom header in Zoom.
5.  **Configure TLS/SSL**:
    *   Under the **Advanced options**, locate the **TLS** settings.
    *   You **must** enable TLS as Zoom requires HTTPS.
    *   Provide the path to your SSL/TLS `certificate` and `key` files on the agent's host machine.
    *   Alternatively, you can use a reverse proxy (like NGINX or Apache) to handle TLS termination and forward unencrypted traffic to the agent locally. If you use a reverse proxy, you do not need to configure TLS within the integration itself.
6.  Click **Save and continue**. This will deploy the policy to your Elastic Agent.

### Validation
To validate that the integration is working:
1.  Perform an action in Zoom that you have subscribed to (e.g., start a meeting or log in).
2.  In Kibana, navigate to the **Discover** tab.
3.  Search for `data_stream.dataset: "zoom.webhook"`. The corresponding Zoom event should appear within a few moments.
4.  You can also check the pre-built dashboards provided by the integration by navigating to **Dashboard** and searching for "Zoom".

## Troubleshooting

For help with Elastic ingest tools, check [Common problems](https://www.elastic.co/docs/troubleshoot/ingest/fleet/common-problems).

**Common issues for this integration include:**
*   **Connectivity Problems**: If events are not arriving, ensure that the host running the Elastic Agent is publicly accessible and that any firewalls or network security groups allow incoming traffic on the configured port from Zoom's IP addresses.
*   **TLS/SSL Errors**: Check the Elastic Agent logs for any TLS handshake errors. Ensure your certificate and key are valid, in the correct format, and accessible by the user running the agent process.
*   **Authentication Failures**: If you have configured a custom header, double-check that the header key and value in the integration settings exactly match what you configured in the Zoom App Marketplace.

## Reference

### webhook

The `webhook` data stream provides events from Zoom based on your webhook subscriptions.

#### webhook fields

{{ fields "webhook" }}

#### webhook sample event

{{ event "webhook" }}

### Inputs used
{{ inputDocs }}

### API usage
This integration uses the Zoom Webhook API to receive event data. For more information, see the [Zoom Webhook Documentation](https://developers.zoom.us/docs/api/rest/webhook-reference/).
