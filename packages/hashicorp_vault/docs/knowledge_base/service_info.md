# Service Info

## Common use cases

*   **Security Monitoring and Compliance:** Capture a tamper-evident record of every request and response sent to the Vault API to meet regulatory requirements (e.g., SOC2, PCI-DSS).
*   **Access Auditing:** Monitor authentication attempts and secret access to detect potential unauthorized activity or lateral movement within the infrastructure.
*   **Operational Troubleshooting:** Debug failed authentication requests, policy denials, and malformed API calls by analyzing the detailed error information provided in audit logs.

## Data types collected

This integration collects **Logs** specifically from the Vault Audit Device. These logs are structured in **JSON format** and contain detailed metadata about every API interaction with the Vault server.

## Compatibility

This integration is compatible with:
*   **HashiCorp Vault:** All versions supporting the `syslog` audit device.
*   **Operating Systems:** Unix-like systems (Linux, BSD) that support a local syslog daemon (e.g., `rsyslog` or `syslog-ng`).
*   **Elastic Agent:** Version 7.16.0 or higher is recommended for full integration support.

## Scaling and Performance

Vault's audit system is designed for high reliability; if Vault is unable to write to any of its configured audit devices, it will block all further requests to ensure no unrecorded actions occur. For high-throughput environments, use reliable TCP forwarding and ensure the Elastic Agent is provisioned with sufficient resources to handle the log volume. It is highly recommended to configure multiple audit devices (e.g., syslog and a local file) to prevent service outages due to a single device failure.

# Set Up Instructions

## Vendor prerequisites

*   A running HashiCorp Vault cluster on a Unix-like operating system.
*   Administrative privileges (root or equivalent policy) to enable and manage audit devices.
*   A local syslog daemon (`rsyslog` or `syslog-ng`) installed and running on the Vault server.
*   Network connectivity between the Vault server and the Elastic Agent.

## Elastic prerequisites

*   Elastic Agent installed on a host reachable by the Vault server's syslog daemon.
*   An active Elastic Agent policy with the HashiCorp Vault integration added.
*   Firewall rules allowing traffic on the designated port (e.g., TCP 9002) from the Vault server to the Elastic Agent.

## Vendor set up steps

1.  **Enable the Syslog Audit Device:** Connect to your Vault server CLI and enable the device with specific facility and tag settings for easier filtering:
    ```bash
    vault audit enable syslog facility="LOCAL7" tag="vault"
    ```
2.  **Verify Configuration:** Ensure the device is active by running:
    ```bash
    vault audit list -detailed
    ```
3.  **Configure Local Syslog Forwarding:** Create a new configuration file for your syslog daemon (e.g., `/etc/rsyslog.d/60-vault-forward.conf`) and add a rule to forward logs to the Elastic Agent:
    ```text
    if $syslogfacility-text == 'local7' then @@<elastic-agent-ip>:<elastic-agent-port>
    ```
    *(Replace `<elastic-agent-ip>` and `<elastic-agent-port>` with your specific details; `@@` denotes TCP).*
4.  **Restart Syslog:** Apply the changes by restarting the service:
    ```bash
    sudo systemctl restart rsyslog
    ```
5.  **Enable Fallback Device (Optional):** To prevent Vault from blocking requests if syslog fails, enable a file-based audit device:
    ```bash
    vault audit enable file file_path=/var/log/vault_audit.log
    ```

## Kibana set up steps

1.  Log in to Kibana and navigate to **Management > Integrations**.
2.  Search for and select the **HashiCorp Vault** integration.
3.  Click **Add HashiCorp Vault**.
4.  Under the **Audit Logs** configuration, set the **Listen Address** (e.g., `0.0.0.0`) and **Listen Port** (e.g., `9002`) to match your `rsyslog` forwarding configuration.
5.  Select the **Protocol** (TCP or UDP) used in your vendor setup.
6.  Assign the integration to an existing or new **Agent Policy** and click **Save and Continue**.

# Validation Steps

1.  **Generate Audit Traffic:** Perform a simple operation in Vault, such as listing secrets or checking token status:
    ```bash
    vault token lookup
    ```
2.  **Check Local Logs:** Verify that the syslog daemon is receiving the logs locally (e.g., `tail -f /var/log/syslog`).
3.  **Verify in Kibana:** Navigate to **Observability > Logs** or **Discover** and search for `event.dataset: vault.audit` or `service.type: vault`.
4.  **Review Dashboard:** Open the **[Logs Vault] Overview** dashboard in Kibana to see visualized audit event data.

# Troubleshooting

## Common Configuration Issues

*   **Vault Performance Degradation:** If Vault becomes unresponsive, check if the audit device is blocked. Use `vault status` to check the health and ensure the syslog daemon is accepting connections.
*   **Connectivity Issues:** Ensure the Elastic Agent is bound to the correct network interface and that the port is not blocked by `iptables`, `ufw`, or external cloud firewalls.
*   **SELinux/AppArmor:** On some systems, security modules may prevent `rsyslog` from sending data over the network. Check audit logs (`/var/log/audit/audit.log`) for permission denials.

## Ingestion Errors

*   **Malformed JSON:** If logs appear with `error.message`, ensure that Vault is sending raw JSON. Vault audit logs are JSON-formatted by default, but intermediate syslog processors might prepend timestamps or headers that interfere with parsing.
*   **Facility Mismatch:** Ensure the `facility` defined in the Vault audit enable command matches the filter used in the `rsyslog` configuration.

## API Authentication Errors

*   **Permission Denied:** Ensure the Vault token used for setup has the `sudo` capability on the `sysys/audit` path.
*   **Invalid Policy:** If audit logs show 403 errors for specific users, verify that their associated policies grant sufficient access to the requested secret paths.

## Vendor Resources

*   [HashiCorp Help Center: Blocked Audit Logging](https://hashicorp1674582558.zendesk.com/hc/en-us/articles/17325040782739-Blocked-Audit-Logging)
*   [Vault Troubleshooting Guide](https://developer.hashicorp.com/vault/docs/troubleshooting)
*   [Vault Community Forum](https://discuss.hashicorp.com/c/vault/13)

# Documentation sites

*   [Vault Audit Devices Overview](https://developer.hashicorp.com/vault/docs/audit)
*   [Syslog Audit Device Documentation](https://developer.hashicorp.com/vault/docs/audit/syslog)
*   [Vault API Reference](https://developer.hashicorp.com/vault/api-docs)
*   [Elastic Integration for HashiCorp Vault](https://docs.elastic.co/integrations/vault)