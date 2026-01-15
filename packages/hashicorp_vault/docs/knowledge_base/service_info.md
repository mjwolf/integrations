# Service Info

## Common use cases

The HashiCorp Vault integration for Elastic allows organizations to monitor and secure their secret management infrastructure by ingesting comprehensive audit logs.
- **Security Auditing and Compliance:** Track every request and response sent to the Vault server to maintain a complete audit trail for regulatory compliance (e.g., SOC2, PCI-DSS, HIPAA).
- **Detecting Unauthorized Access:** Identify patterns of failed authentication attempts, unauthorized path access, or unusual secret retrieval activity across the environment.
- **Monitoring Policy Effectiveness:** Analyze logs to ensure that Vault policies are correctly restricting access and that users or services are not attempting to access paths outside their scope.
- **Operational Troubleshooting:** Review detailed audit metadata to diagnose failed API calls, configuration errors, or latency issues within the Vault secret engine and authentication workflows.

## Data types collected

This integration can collect the following types of data from HashiCorp Vault instances:
- **Audit Logs:** Detailed records of every interaction with the Vault API, including the identity of the requester, the operation performed, and the response status.
- **Authentication Events:** Logs capturing login attempts, token renewals, and entity mappings for various auth methods (LDAP, AppRole, GitHub, etc.).
- **Secret Engine Operations:** Records of secret creation, reading, updating, and deletion (CRUD) operations across all enabled secret engines (KV, Transit, Database).
- **System Events:** Administrative actions such as audit device configuration, policy updates, and seal/unseal operations.
- **Data Formats:** Logs are typically generated in JSON format by Vault and encapsulated within Syslog wrappers for transport.
- **Default Syslog Path:** Logs are usually processed via the local syslog daemon (e.g., `/var/log/syslog` or `/var/log/messages`) before being forwarded.

## Compatibility

This integration is compatible with **HashiCorp Vault** (Open Source and Enterprise) versions 1.x and higher. It also supports **HCP Vault** (HashiCorp Cloud Platform) when configured to forward logs via a supported transport mechanism. The integration relies on standard Syslog protocols (RFC 3164 or RFC 5424) for data ingestion.

## Scaling and Performance

- **Transport/Collection Considerations:** This integration utilizes Syslog as the primary transport mechanism. While UDP offers lower latency, TCP (configured via `@@` in rsyslog) is strongly recommended for Vault audit logs. Audit logs often contain large JSON payloads that can exceed the standard UDP MTU, leading to packet fragmentation or message truncation.
- **Data Volume Management:** Vault audit logs can generate significant data volume in high-traffic environments. To manage this, administrators can use Vault's `audit_filter` string to exclude specific high-volume paths (e.g., health checks or specific read-heavy KV paths) from being logged, thereby reducing the load on both the syslog daemon and the Elastic Stack.
- **Elastic Agent Scaling:** For high-throughput Vault clusters, a single Elastic Agent may become a bottleneck. It is recommended to deploy Elastic Agents in a highly available configuration behind a network load balancer. This allows for horizontal scaling of log ingestion and ensures that the syslog stream is not interrupted during Agent maintenance or updates.

# Set Up Instructions

## Vendor prerequisites
- **Administrative Access:** You must have a Vault token with `sudo` capabilities or a policy that allows enabling and configuring audit devices.
- **Local Syslog Daemon:** The server running HashiCorp Vault must have a syslog service (like `rsyslog` or `syslog-ng`) installed and running.
- **Network Connectivity:** The Vault server must be able to communicate with the Elastic Agent over the configured TCP/UDP port (e.g., port 9001).
- **Redundancy Planning:** Administrative access to the Vault host filesystem is required to configure a secondary file-based audit device, which is a best practice to prevent Vault from hanging if the syslog service becomes unreachable.

## Elastic prerequisites

- **Elastic Agent Status:** An Elastic Agent must be installed and successfully enrolled in Fleet.
- **Integration Installation:** The "HashiCorp Vault" integration must be added to an Elastic Agent policy.
- **Connectivity:** Ensure the Elastic Agent host is listening on the network interface and port specified in the integration configuration and that local firewalls (e.g., `iptables` or `ufw`) permit the traffic.

## Vendor set up steps

### For Syslog-based Collection:

1. **Access the Vault Server:** SSH into the machine where HashiCorp Vault is running.
2. **Enable the Syslog Audit Device:** Run the Vault CLI command to start sending audit events to the local syslog service. It is recommended to use a specific facility and tag for easier filtering.
   ```sh
   vault audit enable syslog facility="LOCAL7" tag="vault-audit"
   ```
3. **Optional: Enable Raw Logging:** If you require unredacted logs for specific troubleshooting (caution: this logs sensitive data), use the `log_raw` flag:
   ```sh
   vault audit enable syslog facility="LOCAL7" tag="vault-audit" log_raw=true
   ```
4. **Configure rsyslog Forwarding:** Create a new configuration file for the forwarding rule to keep the main configuration clean.
   ```sh
   sudo vi /etc/rsyslog.d/60-vault-forward.conf
   ```
5. **Add the Forwarding Rule:** Insert the following lines, replacing `<ELASTIC_AGENT_IP>` with the actual IP of your Elastic Agent and `<PORT>` with the port configured in your Elastic Integration (e.g., 9514).
   ```text
   # Forward Vault audit logs to Elastic Agent
   if $programname == 'vault-audit' then @@<ELASTIC_AGENT_IP>:<PORT>
   & stop
   ```
   *Note: Using `@@` ensures TCP transport for reliable delivery.*
6. **Validate rsyslog Configuration:** Check for syntax errors in the rsyslog configuration before restarting.
   ```sh
   rsyslogd -N1
   ```
7. **Restart the Syslog Service:** Apply the changes by restarting rsyslog.
   ```sh
   sudo systemctl restart rsyslog
   ```
8. **Verify Local Logging:** Check the local system logs to ensure Vault is successfully sending data to the local daemon.
   ```sh
   tail -f /var/log/syslog | grep vault-audit
   ```

## Kibana set up steps

### Configure the Vault Integration:

1.  **Navigate to Integrations**: Log in to Kibana and go to **Management > Integrations**.
2.  **Locate the Integration**: Search for **HashiCorp Vault** and select it. Click on **Add HashiCorp Vault**.
3.  **Configure Integration Settings**:
    - Under the **Syslog** input section, set the **Syslog Host** to `0.0.0.0` to listen on all interfaces.
    - Set the **Syslog Port** to match the port you configured in your rsyslog settings (e.g., `9001`).
    - Select the **Protocol** (TCP is recommended to match the `@@` used in rsyslog).
4.  **Advanced Options**: In the "Internal usage" or "Tags" section, you may add `vault` to help with filtering, though the integration handles most mapping automatically.
5.  **Save and Deploy**: Click **Save and continue** and then **Add agent integration to policy** to deploy the configuration to your Elastic Agents.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from Vault to the Elastic Stack.

### 1. Trigger Data Flow on HashiCorp Vault:

- **Generate Authentication Event**: Perform a login operation using the Vault CLI: `vault login -method=token`.
- **Generate Secret Access Event**: Read a test secret from the KV store: `vault kv get secret/test_connection`.
- **Generate Administrative Event**: List the enabled audit devices to trigger an internal API call: `vault audit list`.

### 2. Check Data in Kibana:

1.  Navigate to **Analytics > Discover**.
2.  Select the `logs-*` data view.
3.  Enter the following KQL filter: `data_stream.dataset : "hashicorp_vault.audit"`
4.  Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
    - `event.dataset` (should be `hashicorp_vault.audit`)
    - `hashicorp_vault.audit.request.operation` (e.g., `update`, `read`)
    - `hashicorp_vault.audit.request.path` (e.g., `auth/token/lookup-self`)
    - `event.outcome` (e.g., `success` or `failure`)
    - `message` (containing the raw JSON audit payload)
5.  Navigate to **Analytics > Dashboards** and search for **[Logs HashiCorp Vault] Audit Overview** to view the pre-built security visualizations.

# Troubleshooting

## Common Configuration Issues

- **Vault Service Blocking**: If the syslog daemon stops or the destination is unreachable, Vault may stop responding to requests to prevent un-audited actions. **Solution**: Ensure a secondary `file` audit device is enabled so Vault has a fallback destination for logs.
- **Rsyslog Filtering Failures**: If logs are reaching the server but not being forwarded, the `$programname` filter might be incorrect. **Solution**: Check the raw syslog file on the Vault server to see the exact tag being used and update `/etc/rsyslog.d/90-vault-forward.conf` to match.
- **SELinux/AppArmor Restrictions**: On hardened systems, the syslog daemon may be blocked from sending data over network ports. **Solution**: Update security policies (e.g., `semanage port -a -t syslogd_port_t -p tcp 9001`) to allow the connection.
- **Port Mismatch**: The port defined in the Elastic Agent policy must exactly match the port used in the rsyslog forwarding rule. **Solution**: Verify both configurations and use `netstat -tulnp` on the Agent host to confirm it is listening.

## Ingestion Errors

- **JSON Parsing Failures**: If the `message` field contains the log but structured fields are missing, the syslog header might be improperly formatted. Ensure Vault is not adding double headers and that the Elastic Agent is configured for the correct Syslog format (RFC3164 vs RFC5424).
- **Truncated Logs**: If you see `error.message` indicating invalid JSON, the logs may be truncated. This often happens with UDP transport. Switching the rsyslog configuration to TCP (`@@`) and the Elastic Agent input to TCP usually resolves this.
- **Mapping Conflicts**: Check for `error.type: "mapping_conflict"` in Kibana. This can occur if a custom field in a Vault secret path has the same name as an ECS field but a different data type.

## Vendor Resources

- [Syslog - Audit Devices | Vault | HashiCorp Developer](https://developer.hashicorp.com/vault/docs/audit/syslog)
- [Audit Devices | Vault | HashiCorp Developer](https://developer.hashicorp.com/vault/docs/audit)

# Documentation sites

- https://zrubi.hu/en/2025/hcp-vault-audit-logs-to-siem/
- https://community.ibm.com/community/user/blogs/surajprakash-vidhani/2025/11/10/hashicorp-vault-logs-to-qradar-syslog
- Refer to the official vendor website for additional resources.
