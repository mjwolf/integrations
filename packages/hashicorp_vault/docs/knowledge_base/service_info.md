# Service Info

## Common use cases

The HashiCorp Vault integration for Elastic allows organizations to monitor and secure their secret management infrastructure by ingesting comprehensive audit logs.
- **Security Auditing and Compliance:** Track every request and response sent to the Vault server to maintain a complete audit trail for regulatory compliance.
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

This integration is compatible with **HashiCorp Vault** (Open Source and Enterprise) versions 1.x and higher. The integration relies on standard Syslog protocols (RFC 3164 or RFC 5424) for data ingestion. This integration has been tested on HashiCorp Vault 1.11.

## Scaling and Performance

- **Transport/Collection Considerations:** This integration utilizes Syslog as the primary transport mechanism. While UDP offers lower latency, TCP is strongly recommended for Vault audit logs. Audit logs often contain large JSON payloads that can exceed the standard UDP MTU, leading to packet fragmentation or message truncation.
- **Data Volume Management:** Vault audit logs can generate significant data volume in high-traffic environments. To manage this, administrators can use Vault's `audit_filter` string to exclude specific high-volume paths (e.g., health checks or specific read-heavy KV paths) from being logged, thereby reducing the load on both the syslog daemon and the Elastic Stack.
- **Elastic Agent Scaling:** For high-throughput Vault clusters, a single Elastic Agent may become a bottleneck. It is recommended to deploy Elastic Agents in a highly available configuration behind a network load balancer. This allows for horizontal scaling of log ingestion and ensures that the syslog stream is not interrupted during Agent maintenance or updates.

# Set Up Instructions

## Vendor prerequisites
- **Administrative Access:** You must have a Vault token with `sudo` capabilities or a policy that allows enabling and configuring audit devices.

The following prerequisites are required on the HashiCorp Vault side:

### For Audit Log Collection

- **File Audit Device**: A file audit device must be enabled with write permissions to a directory accessible by Vault
- **Socket Audit Device** (alternative): A socket audit device can be configured to stream logs to a TCP endpoint where Elastic Agent is listening

### For Operational Log Collection

- **JSON Log Format**: Vault must be configured to output logs in JSON format (set `log_format = "json"` in Vault configuration)
- **File Access**: The Vault operational log file must be accessible by Elastic Agent for collection

### For Metrics Collection

- **Vault Token**: A Vault token with read access to the `/sys/metrics` API endpoint
- **Telemetry Configuration**: Vault telemetry must be configured with `disable_hostname = true` and `enable_hostname_label = true` is recommended
- **Network Access**: The Elastic Agent must be able to reach the Vault API endpoint (default: `http://localhost:8200`)

## Elastic prerequisites

- **Elastic Agent Status:** An Elastic Agent must be installed and successfully enrolled in Fleet.
- **Integration Installation:** The "HashiCorp Vault" integration must be added to an Elastic Agent policy.
- **Connectivity:** Ensure the Elastic Agent host is listening on the network interface and port specified in the integration configuration and that local firewalls (e.g., `iptables` or `ufw`) permit the traffic.

## Vendor set up steps

### Setting up Audit Logs (File Audit Device)

1. Create a directory for audit logs on each Vault server:
```bash
mkdir /var/log/vault
```

2. Enable the file audit device in Vault:
```bash
vault audit enable file file_path=/var/log/vault/audit.json
```

3. Configure log rotation to prevent disk space issues. Example using `logrotate`:
```bash
tee /etc/logrotate.d/vault <<'EOF'
/var/log/vault/audit.json {
    rotate 7
    daily
    compress
    delaycompress
    missingok
    notifempty
    extension json
    dateext
    dateformat %Y-%m-%d.
    postrotate
        /bin/systemctl reload vault || true
    endscript
}
EOF
```

### Setting up Audit Logs (Socket Audit Device)

1. Note the IP address and port where Elastic Agent will be listening (default: port 9007)

2. Enable the socket audit device in Vault (substitute your Elastic Agent IP):
```bash
vault audit enable socket address=${ELASTIC_AGENT_IP}:9007 socket_type=tcp
```

**Note**: Configure the integration in Kibana first before enabling the socket audit device, as Vault will test the connection.

**Warning: Risk of Unresponsive Vault with TCP Socket Audit Devices**: If a TCP socket audit log destination (like the Elastic Agent)
becomes unavailable, Vault may block and stop processing all requests until the connection is restored. This can lead to a service outage.
To mitigate this risk, HashiCorp strongly recommends that a socket audit device is configured as a secondary device, alongside a primary,
non-socket audit device (like the `file` audit device). For more details, see the official documentation on [Blocked Audit Devices](https://developer.hashicorp.com/vault/docs/audit/socket#configuration).

### Setting up Operational Logs

Configure Vault to output logs in JSON format by adding to your Vault configuration file:
```hcl
log_format = "json"
```

Direct Vault's log output to a file that Elastic Agent can read.

### Setting up Metrics

1.  Configure Vault telemetry in your Vault configuration file:
    ```hcl
    telemetry {
      disable_hostname = true
      enable_hostname_label = true
    }
    ```
    Restart the Vault server after saving this file.
2.  Create a Vault policy file that grants read access to the metrics endpoint.
    ```hcl
    path "sys/metrics" {
      capabilities = ["read"]
    }
    ```
3.  Apply the policy.
    ```bash
    vault policy write read-metrics metrics-policy.hcl
    ```
4.  Create a Vault token with this policy:
    ```bash
    vault token create -policy="read-metrics" -display-name="elastic-agent-token"
    ```
    Save the token value, it will be needed to complete configuring the integration in Kibana.

## Kibana set up steps

### Configure the Vault Integration:
1. In Kibana, navigate to **Management > Integrations**
2. Search for "HashiCorp Vault" and select the integration
3. Click **Add HashiCorp Vault**
4. Configure the integration based on your data collection needs:
   **For Audit Logs (File)**:
   - Enable the "Logs from file" --> "Audit logs (file audit device)" input
   - Specify the file path (default: `/var/log/vault/audit*.json*`)
   - Optionally enable "Preserve original event" to keep raw logs
   **For Audit Logs (TCP Socket)**:
   - Enable the "Logs from TCP socket" input
   - Configure the listen address (default: `localhost`) and port (default: `9007`)
   - If Vault will connect remotely, set listen address to `0.0.0.0`
   **For Operational Logs**:
   - Enable the "Logs from file" --> "Operation logs" input
   - Specify the log file path (default: `/var/log/vault/log*.json*`)
   **For Metrics**:
   - Enable the "Metrics" input
   - Enter the Vault host URL (default: `http://localhost:8200`)
   - Provide the Vault token with read access to `/sys/metrics`
   - Optionally configure SSL settings if using HTTPS
   - Adjust the collection period if needed (default: `30s`)
5. Configure the agent policy and select the agent to run this integration
6. Click **Save and continue** to deploy the integration

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from Vault to the Elastic Stack.

### 1. Trigger Data Flow on HashiCorp Vault:

- **Generate Authentication Event**: Perform a login operation using the Vault CLI: `vault login -method=token`.
- **Generate Secret Access Event**: Read a test secret from the KV store: `vault kv get secret/test_connection`.
- **Generate Administrative Event**: List the enabled audit devices to trigger an internal API call: `vault audit list`.

### 2. Check Data in Kibana:

1.  Navigate to **Analytics > Discover**.
   - Select the appropriate data view for each data stream:
     - `logs-hashicorp_vault.audit-*` for audit logs
     - `logs-hashicorp_vault.log-*` for operational logs
     - `metrics-hashicorp_vault.metrics-*` for metrics
   - Confirm that events are appearing with recent timestamps
2.  Navigate to **Analytics > Dashboards** and search for **[Logs HashiCorp Vault] Audit Overview** to view the pre-built security visualizations.

# Troubleshooting

## Common Configuration Issues

- **Vault stops responding to requests**: You may have a blocked audit device. This can happen if a TCP socket destination is unavailable or a file
audit device cannot write to disk. Review Vault's operational logs for errors related to audit logging. For more information on identifying and
resolving this, see the [Blocked Audit Device Behavior](https://developer.hashicorp.com/vault/tutorials/monitoring/blocked-audit-devices#blocked-audit-device-behavior) tutorial.
- **Vault Service Blocking**: If the syslog daemon stops or the destination is unreachable, Vault may stop responding to requests to prevent un-audited actions. **Solution**: Ensure a secondary `file` audit device is enabled so Vault has a fallback destination for logs.
- **Rsyslog Filtering Failures**: If logs are reaching the server but not being forwarded, the `$programname` filter might be incorrect. **Solution**: Check the raw syslog file on the Vault server to see the exact tag being used and update `/etc/rsyslog.d/90-vault-forward.conf` to match.
- **SELinux/AppArmor Restrictions**: On hardened systems, the syslog daemon may be blocked from sending data over network ports. **Solution**: Update security policies (e.g., `semanage port -a -t syslogd_port_t -p tcp 9001`) to allow the connection.
- **Port Mismatch**: The port defined in the Elastic Agent policy must exactly match the port used in the rsyslog forwarding rule. **Solution**: Verify both configurations and use `netstat -tulnp` on the Agent host to confirm it is listening.

## Ingestion Errors

- **JSON Parsing Failures**: If the `message` field contains the log but structured fields are missing, the syslog header might be improperly formatted. Ensure Vault is not adding double headers and that the Elastic Agent is configured for the correct Syslog format (RFC3164 vs RFC5424).
- **Truncated Logs**: If you see `error.message` indicating invalid JSON, the logs may be truncated. This often happens with UDP transport. Switching the rsyslog configuration to TCP (`@@`) and the Elastic Agent input to TCP usually resolves this.
- **Mapping Conflicts**: Check for `error.type: "mapping_conflict"` in Kibana. This can occur if a custom field in a Vault secret path has the same name as an ECS field but a different data type.

## API Authentication Errors
### Token expired or invalid
- Generate a new Vault token with appropriate permissions
- Update the integration configuration in Kibana with the new token
- For long-running deployments, use a token with an appropriate TTL or create a periodic token
### Permission denied errors
- Verify the token has a policy granting read access to `/sys/metrics`
- Check Vault audit logs for permission denial details
- Example policy for metrics access:
```hcl
path "sys/metrics" {
  capabilities = ["read"]
}
```

## Vendor Resources

- [HashiCorp Vault Deployment Guide](https://developer.hashicorp.com/vault/tutorials/day-one-raft/raft-deployment-guide)
- [HashiCorp Vault Audit Devices](https://developer.hashicorp.com/vault/docs/audit)
- [HashiCorp Vault File Audit Device](https://developer.hashicorp.com/vault/docs/audit/file)
- [HashiCorp Vault Socket Audit Device](https://developer.hashicorp.com/vault/docs/audit/socket)
- [HashiCorp Vault Telemetry Configuration](https://developer.hashicorp.com/vault/docs/configuration/telemetry)
- [HashiCorp Vault Troubleshooting](https://developer.hashicorp.com/vault/docs/troubleshoot)
- [Syslog - Audit Devices | Vault | HashiCorp Developer](https://developer.hashicorp.com/vault/docs/audit/syslog)

# Documentation sites

- https://zrubi.hu/en/2025/hcp-vault-audit-logs-to-siem/
- https://community.ibm.com/community/user/blogs/surajprakash-vidhani/2025/11/10/hashicorp-vault-logs-to-qradar-syslog
