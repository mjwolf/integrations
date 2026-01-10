# Service Info

## Common use cases

*   **Security Auditing:** Monitor administrative changes, user logins, and account-level permission updates to ensure compliance and detect unauthorized access.
*   **Meeting Compliance:** Track meeting activities, including start/end times and participant joins/leaves, to audit internal communication policies.
*   **Recording Management:** Oversee the creation, deletion, and access patterns of cloud recordings to protect sensitive recorded content.

## Data types collected

This integration primarily collects **Logs** and **Events**. Data is received as real-time event notifications via Zoom Webhooks, covering categories such as Meeting, Webinar, Recording, User, and Account activity.

## Compatibility

This integration is compatible with all Zoom plans that provide access to the Zoom App Marketplace (Pro, Business, Education, and Enterprise). It supports Zoom Webhook API v2 and requires an Elastic Agent version compatible with HTTP-based integrations.

## Scaling and Performance

The integration uses a push-based webhook model, which is more efficient than polling and scales based on the volume of events generated within the Zoom account. For high-volume environments, ensure the Elastic Agent endpoint is hosted on infrastructure that can handle burst traffic and maintain response times below 3 seconds to satisfy Zoom’s webhook delivery requirements.

# Set Up Instructions

## Vendor prerequisites

*   A Zoom account with Developer/Administrative privileges.
*   Access to the [Zoom App Marketplace](https://marketplace.zoom.us/).
*   A publicly accessible HTTPS endpoint URL (provided by Elastic Agent) to receive webhook payloads.

## Elastic prerequisites

*   Elastic Stack (Elasticsearch, Kibana) version 7.14 or higher.
*   An enrolled Elastic Agent with a policy that includes the "Zoom" integration.
*   The Elastic Agent must be reachable from the internet over HTTPS (typically port 443 or a custom port).

## Vendor set up steps

1.  **Log in to the Zoom App Marketplace:** Navigate to [https://marketplace.zoom.us/](https://marketplace.zoom.us/) and sign in with administrative credentials.
2.  **Create a Webhook App:** Click **Develop** > **Build App**, select **Webhook Only**, and click **Create**.
3.  **Basic Information:** Enter an app name (e.g., `Elastic-Agent-Zoom`), provide your company details, and click **Continue**.
4.  **Enable Event Subscriptions:** In the **Feature** tab, toggle **Event Subscriptions** to on.
5.  **Configure Endpoint:** Click **Add new event subscription**. Enter a name and paste the **Event notification endpoint URL** generated during the Elastic integration setup.
6.  **Select Events:** Click **Add Events** and select all desired event types (e.g., Meeting, Recording, User activity). Click **Done**.
7.  **Activate:** Click **Save** and then **Continue** to activate the app.

## Kibana set up steps

1.  In Kibana, go to **Management** > **Integrations**.
2.  Search for and select the **Zoom** integration.
3.  Click **Add Zoom**.
4.  Configure the integration name and select the Agent policy.
5.  Note the **Listen Address** and **Port** settings (ensure these match your network configuration for the Elastic Agent).
6.  (Optional) Define a **Webhook Secret** if you intend to verify the Zoom signature for enhanced security.
7.  Click **Save and continue**.

# Validation Steps

1.  **Trigger a Test Event:** Start and then end a Zoom meeting or log out and log back in to the Zoom portal.
2.  **Check Data Stream:** In Kibana, navigate to **Analytics** > **Discover**.
3.  **Filter Results:** Search for `event.dataset : "zoom.audit"` or `data_stream.dataset : "zoom.webhook"`.
4.  **Verify Fields:** Ensure fields such as `zoom.event`, `user.email`, and `source.ip` are being populated correctly.

# Troubleshooting

## Common Configuration Issues

*   **Endpoint Unreachable:** Ensure the Elastic Agent's host has a public IP or is behind a load balancer/reverse proxy that allows traffic from Zoom’s IP ranges.
*   **SSL/TLS Requirements:** Zoom requires a valid, publicly trusted SSL certificate for the webhook endpoint. Self-signed certificates will typically fail validation.
*   **Port Blocking:** Verify that the configured port (e.g., 443) is open on the host firewall and any intermediate network firewalls.

## Ingestion Errors

*   **Parsing Failures:** If `error.message` appears in the logs, verify that the Zoom app is sending the expected JSON schema.
*   **Missing Fields:** Ensure that all necessary event types were selected in the Zoom App Marketplace "Add Events" configuration.

## API Authentication Errors

*   **Signature Verification Failed:** If using a Secret Token, ensure the token in the Zoom App Marketplace matches the `webhook_secret` configured in the Elastic integration settings.
*   **Deactivated App:** Check the Zoom App Marketplace to ensure the Webhook app status is "Active."

## Vendor Resources

*   [Zoom Webhook Documentation](https://developers.zoom.us/docs/api/rest/webhook-reference/)
*   [Zoom Developer Support](https://devsupport.zoom.us/hc/en-us)
*   [Zoom Marketplace Status](https://status.zoom.us/)

# Documentation sites

*   [Zoom API Reference](https://developers.zoom.us/docs/api/)
*   [Elastic Integration for Zoom](https://docs.elastic.co/integrations/zoom)
*   [Zoom Webhook Event Types List](https://developers.zoom.us/docs/api/rest/webhook-reference/#events)