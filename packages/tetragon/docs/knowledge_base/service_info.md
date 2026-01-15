# Service Info

## Common use cases

The Tetragon integration for Elastic Agent enables the collection of rich security and observability data directly from the Linux kernel using eBPF. It is designed to provide deep visibility into the runtime behavior of containers and hosts within a Kubernetes environment.
- **Runtime Security Monitoring:** Detect and alert on suspicious process executions, such as unauthorized shells or unexpected binary execution in production containers.
- **Network Observability and Security:** Track network socket connections, including source and destination IP addresses and ports, to identify potential data exfiltration or unauthorized lateral movement.
- **Compliance and Auditing:** Maintain a detailed log of system calls and file access patterns to meet regulatory requirements for monitoring sensitive data access and administrative actions.
- **Privilege Escalation Detection:** Monitor for changes in process capabilities or namespace escapes that could indicate a container breakout or host-level compromise.

## Data types collected

This integration can collect the following types of data:
- **Security Observability Events:** Detailed records of process execution (execve), exit codes, and process metadata.
- **Network Metadata:** Logs of network socket operations, including connection attempts, lifecycle events, and protocol-specific details.
- **File System Events:** Information regarding file access, modifications, and permission changes captured via eBPF probes.
- **JSON Event Stream:** All data is exported in a structured JSON format, typically via the `export-stdout` container logs.
- **Log Paths:** Data is collected from container log paths, typically located at `/var/log/containers/tetragon*export-stdout*.log` on the Kubernetes node.

## Compatibility

The **Tetragon** integration is designed for environments running **Tetragon v0.8.0 or higher**, typically deployed within **Kubernetes** clusters or as a standalone systemd service on Linux hosts. It requires an **Elastic Agent** version compatible with the JSON log input, preferably **version 8.x or higher**.

## Scaling and Performance

- **Transport/Collection Considerations:** Tetragon defaults to exporting events via a JSON stream to stdout. For most environments, this is highly efficient as it leverages standard Kubernetes log rotation and collection mechanisms. For high-throughput environments where log file I/O becomes a bottleneck, the gRPC export method should be considered to stream events directly to a collector without writing to disk.
- **Data Volume Management:** To manage event volume, users should configure Tetragon's internal filtering (TracingPolicies) to limit the types of events captured. Filtering at the source reduces the CPU overhead of the Tetragon agent and prevents the Elastic Agent from processing unnecessary security noise, such as frequent heartbeat events or known safe process executions.
- **Elastic Agent Scaling:** When collecting logs from Tetragon, the Elastic Agent should be deployed as a DaemonSet to ensure local collection on every node. In high-volume clusters, ensure the Elastic Agent has sufficient memory and CPU limits to handle the bursty nature of security events, and consider increasing the number of output workers if ingestion lag is observed.

# Set Up Instructions

## Vendor prerequisites

- **Administrative Access:** You must have `cluster-admin` permissions for the Kubernetes cluster where Tetragon is deployed.
- **Helm 3.x:** The Tetragon deployment requires Helm 3 for managing the installation and configuration of the Cilium/Tetragon repository.
- **Network Connectivity:** The gRPC port (default `54321`) must be open and accessible within the cluster.
- **Tetragon Installation:** Tetragon must be installed using the Cilium Helm chart with the gRPC server explicitly enabled.
- **Resource Requirements:** Ensure the Kubernetes nodes have sufficient resources to run eBPF programs and the Tetragon agent pods.

## Elastic prerequisites

- **Elastic Agent:** An active Elastic Agent must be enrolled in Fleet and assigned to a policy that includes the Tetragon integration.
- **Network Access:** The Elastic Agent must have a clear network path to the Tetragon gRPC Service endpoint, typically resolved via Kubernetes DNS.

## Vendor set up steps

### For gRPC Collection via Kubernetes:

1.  **Add the Cilium Helm Repository**:
    Open a terminal with `kubectl` access and add the repository:
    ```bash
    helm repo add cilium https://helm.cilium.io/
    helm repo update
    ```

2.  **Configure the gRPC Server**:
    Create a file named `values.yaml` to override default settings. It is critical to set the address to `0.0.0.0` so it listens on the container's network interface rather than just the loopback.
    ```yaml
    tetragon:
      grpc:
        enabled: true
        address: "0.0.0.0:54321"
    ```

3.  **Install or Upgrade Tetragon**:
    Apply the configuration to your cluster. If installing for the first time:
    ```bash
    helm install tetragon cilium/tetragon -n kube-system --values values.yaml
    ```
    If Tetragon is already running, upgrade the existing release:
    ```bash
    helm upgrade tetragon cilium/tetragon -n kube-system --values values.yaml
    ```

4.  **Create a Kubernetes Service**:
    To provide a stable DNS name for the gRPC stream, create a file named `tetragon-service.yaml`:
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: tetragon-grpc
      namespace: kube-system
    spec:
      selector:
        app.kubernetes.io/name: tetragon
      ports:
        - name: grpc
          protocol: TCP
          port: 54321
          targetPort: 54321
    ```

5.  **Deploy the Service**:
    Run the following command to apply the service manifest:
    ```bash
    kubectl apply -f tetragon-service.yaml
    ```

6.  **Verify the Endpoint**:
    Ensure the service has discovered the Tetragon pods by checking for endpoints:
    ```bash
    kubectl get endpoints tetragon-grpc -n kube-system
    ```

## Kibana set up steps

1.  **Navigate to Integrations**:
    Log in to Kibana, go to the main menu, and navigate to **Management > Integrations**.
2.  **Search for Tetragon**:
    In the search bar, type "Tetragon" and select the integration tile.
3.  **Add the Integration**:
    Click **Add Tetragon**. Select the existing Agent Policy you wish to add this integration to.
4.  **Configure gRPC Address**:
    In the configuration screen, locate the **gRPC Server Address** field. Enter the internal Kubernetes DNS name for the service created in the vendor setup steps:
    `tetragon-grpc.kube-system.svc.cluster.local:54321`
5.  **Review Settings**:
    Ensure the protocol and any additional stream settings match your Tetragon deployment.
6.  **Save and Deploy**:
    Click **Save and Continue** and then **Roll out changes** to deploy the updated policy to your Elastic Agents.

# Validation Steps

After configuration is complete, follow these steps to verify data is flowing correctly from Tetragon to the Elastic Stack.

### 1. Trigger Data Flow on Tetragon:
- **Execute a Test Process:** Log into a pod in your cluster and execute a command that Tetragon monitors by default:
  `kubectl exec -it [pod-name] -- /bin/sh -c "whoami"`
- **Generate Network Traffic:** Trigger a network connection from within a monitored container:
  `kubectl exec -it [pod-name] -- curl http://google.com`
- **View Sensitive Files:** Simulate an audit event by attempting to read a sensitive system file (ensure this is done in a safe, test environment):
  `kubectl exec -it [pod-name] -- cat /etc/shadow`

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the following KQL filter: `data_stream.dataset : "tetragon.log"`
4. Verify logs appear in the results. Expand a log entry and confirm these fields are populated:
   - `event.dataset` (should be `tetragon.log`)
   - `process.name` (should show `whoami`, `curl`, or `cat`)
   - `destination.ip` (for the curl command test)
   - `event.action` (e.g., `process_exec` or `socket_connect`)
   - `message` (containing the raw gRPC event details)
5. Navigate to **Analytics > Dashboards** and search for "Tetragon" to view pre-built visualizations and verify that the data is being parsed correctly into charts.

# Troubleshooting

## Common Configuration Issues

- **gRPC Server Listening on Localhost**: If the Elastic Agent cannot connect, ensure that the Tetragon `values.yaml` was updated with `address: "0.0.0.0:54321"`. If it remains at the default `localhost`, the gRPC server will refuse connections from outside its own container.
- **Service Selector Mismatch**: If the `tetragon-grpc` service shows no endpoints, the `selector` in the service manifest might not match the labels on your Tetragon pods. Run `kubectl get pods --show-labels -n kube-system` to verify the correct labels and update the service manifest accordingly.
- **DNS Resolution Failures**: If the Agent logs show "could not resolve host", ensure that the Elastic Agent has permission to query the Kubernetes DNS service and that the namespace in the address (`kube-system`) is correct.

## Ingestion Errors

- **Protobuf Decoding Failures**: If you see errors in the Elastic Agent logs related to decoding gRPC messages, verify that the version of Tetragon you are running is supported by the current version of the integration. Significant changes in Tetragon's Protobuf definitions can cause parsing issues.
- **Missing Fields in Discover**: If the `message` field is populated but specific ECS fields like `process.name` are missing, check the `error.message` field in Kibana. This often indicates a mapping failure or an unexpected event format that the integration's ingest pipeline does not yet handle.
- **High Latency or Dropped Events**: If events are missing during high-load periods, check the Tetragon pod logs for "dropped events" messages. This usually means the eBPF perf buffer is full and requires tuning the `tracing-policy` or increasing Tetragon's internal buffer sizes.

## Vendor Resources

- [Tetragon Concepts: Events](https://tetragon.io/docs/concepts/events/)
- [Tetragon Configuration Options (DeepWiki)](https://deepwiki.com/cilium/tetragon/5.2-configuration-options)

## Documentation sites

- [https://tetragon.io/docs/concepts/events/](https://tetragon.io/docs/concepts/events/)
- Refer to the official vendor website for product administration and configuration guides.
