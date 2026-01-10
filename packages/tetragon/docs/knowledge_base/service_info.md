# Service Info

## Common use cases

*   **Runtime Security Monitoring:** Real-time detection of unauthorized process executions, privilege escalations, and suspicious system calls within Kubernetes clusters.
*   **Compliance and Auditing:** Maintain detailed, immutable audit trails of process lifecycles (exec/exit) and file integrity events for regulatory requirements.
*   **Network Observability:** Monitor and log transparent network activity, including socket connections and DNS queries, at the kernel level to identify lateral movement.

## Data types collected

*   **Logs:** JSON-formatted security events and process metadata.
*   **Events:** eBPF-based system events including `PROCESS_EXEC`, `PROCESS_EXIT`, `PROCESS_KPROBE`, and networking traces.

## Compatibility

*   **Operating Systems:** Linux environments with kernel version 4.19 or greater (5.4+ recommended for full feature support).
*   **Orchestration:** Kubernetes environments supported via Helm deployments.
*   **Tested Kernels:** Stable long-term support kernels 4.19, 5.4, 5.10, 5.15, and bpf-next.

## Scaling and Performance

Tetragon utilizes eBPF for in-kernel event filtering, ensuring low overhead even in high-throughput environments. Performance can be optimized by applying `allowlist` or `denylist` filters in the Helm configuration to reduce the volume of logs exported to the standard output and subsequently processed by the Elastic Agent.

# Set Up Instructions

## Vendor prerequisites

*   Kubernetes cluster with Helm 3 installed.
*   Linux nodes with eBPF support (Kernel 4.19+).
*   Appropriate RBAC permissions to deploy resources into the `kube-system` namespace.

## Elastic prerequisites

*   Elastic Agent deployed as a DaemonSet within the Kubernetes cluster.
*   An active Agent Policy configured in Fleet.
*   Permissions to modify the Agent Policy to add "Custom Logs" or "Kubernetes Container Logs" inputs.

## Vendor set up steps

1.  **Enable Log Export:** Configure the Tetragon Helm deployment to enable the `export-stdout` sidecar container. Create a `values.yaml`:
    ```yaml
    tetragon:
      export:
        stdout:
          enabled: true
    ```
2.  **Install via Helm:** Add the Cilium repository and install Tetragon:
    ```bash
    helm repo add cilium https://helm.cilium.io
    helm repo update
    helm install tetragon cilium/tetragon -n kube-system -f values.yaml
    ```
3.  **Optional Filtering:** To limit log volume, add an `allowlist` to your `values.yaml`:
    ```yaml
    tetragon:
      export:
        allowlist: |
          { "event_set": ["PROCESS_EXEC", "PROCESS_EXIT"], "namespace": ["default"] }
    ```

## Kibana set up steps

1.  Navigate to **Management > Integrations** in Kibana and select **Custom Logs** or **Kubernetes**.
2.  In the integration settings, define the log path to target Tetragon containers: `/var/log/containers/tetragon*.log`.
3.  Add a **JSON Processor** to the input configuration to ensure the Tetragon JSON fields are correctly parsed into individual fields in Elasticsearch.
4.  Assign the configuration to the Agent Policy used by your Kubernetes DaemonSet and save the integration.

# Validation Steps

1.  **Check Pod Status:** Verify the Tetragon pods are running:
    ```bash
    kubectl get pods -n kube-system -l app.kubernetes.io/name=tetragon
    ```
2.  **Verify Log Stream:** Monitor the live JSON output from the dedicated export container:
    ```bash
    kubectl logs -n kube-system -l app.kubernetes.io/name=tetragon -c export-stdout -f
    ```
3.  **Confirm in Kibana:** Open **Kibana Discover** and search for `container.name : "export-stdout"`. Verify that fields such as `process.exec.path` or `event_values` are appearing correctly.

# Troubleshooting

## Common Configuration Issues

*   **Logs Not Exported:** Ensure `tetragon.export.stdout.enabled` is set to `true` in the Helm values. If disabled, the sidecar container will not exist.
*   **Elastic Agent Permissions:** If logs are not reaching Elastic, verify the Elastic Agent has the `read` permission for container log files on the host filesystem.

## Ingestion Errors

*   **Parsing Failures:** If data appears as a raw string in the `message` field, ensure a JSON processor is enabled in the Elastic Agent policy.
*   **Truncated Logs:** High event volumes may cause log rotation issues; ensure the Elastic Agent is configured to track rotated files.

## API Authentication Errors

*   **RBAC Denials:** Tetragon relies on Kubernetes RBAC for its components. Ensure the Tetragon service account has sufficient permissions to monitor the host kernel and namespaces.
*   **Security Context:** On hardened clusters, ensure the Tetragon daemonset is permitted to run as `privileged: true` to access eBPF subsystems.

## Vendor Resources

*   [Tetragon Troubleshooting Guide](https://tetragon.io/docs/troubleshooting/)
*   [Tetragon FAQ](https://tetragon.io/docs/installation/faq/)
*   [Cilium Slack Community (#tetragon channel)](https://slack.cilium.io/)

# Documentation sites

*   **Product Documentation:** [https://tetragon.io/docs/](https://tetragon.io/docs/)
*   **Event Reference:** [https://tetragon.io/docs/concepts/events/](https://tetragon.io/docs/concepts/events/)
*   **GitHub Repository:** [https://github.com/cilium/tetragon](https://github.com/cilium/tetragon)
*   **Logging Configuration:** [https://tetragon.io/docs/installation/helm-chart/](https://tetragon.io/docs/installation/helm-chart/)