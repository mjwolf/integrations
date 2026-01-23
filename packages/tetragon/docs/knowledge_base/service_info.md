# Service Info

## Common use cases

The Cilium Tetragon integration is designed to provide deep runtime security observability and enforcement for Kubernetes environments by ingesting detailed event logs into the Elastic Stack.
- **Runtime Security Monitoring:** Detect and alert on suspicious process executions, file access, and network activity within Kubernetes pods in real-time using eBPF-based monitoring.
- **Compliance Auditing:** Maintain a comprehensive audit trail of system-level activities, such as sensitive system calls or privilege escalations, to satisfy regulatory requirements like SOC2 or PCI-DSS.
- **Threat Hunting:** Use detailed JSON event data to identify lateral movement or container escapes by analyzing process lineages and unexpected binary executions across the cluster.
- **Forensic Analysis:** Investigate security incidents by correlating Tetragon logs with other Elastic data sources, providing a clear timeline of events leading up to a security breach.

## Data types collected

This integration collects structured observability data via the following data stream:

- **log (logs):** This data stream captures security event logs from Tetragon, providing the primary source of observability data. 
    - **Security Event Logs:** Detailed logs of system activity including process lifecycle (`process_exec`, `process_exit`), file integrity monitoring, and network socket activity.
    - **JSON Format:** All data is exported in structured JSON format, allowing for rich metadata and nested field mapping in Elasticsearch.
    - **Log Paths:** Data is specifically collected from the `/var/run/cilium/tetragon/*.log` directory within the Kubernetes node or pod environment.
    - **System Metadata:** Enriched information including Kubernetes pod names, namespaces, container IDs, and process credentials (UID/GID).

## Compatibility

The **Cilium Tetragon** integration is compatible with the following versions:
- **Elastic Stack:** Kibana and Elasticsearch versions `^8.13.0` or `^9.0.0` are required for full support of the integration assets, including ingest pipelines and dashboards.
- **Filebeat:** Tested and supported using Filebeat version `8.15.3` (or newer) as a sidecar container to harvest logs.
- **Kubernetes:** Requires a running cluster (v1.22+) with Tetragon installed via official methods.

## Scaling and Performance

To ensure optimal performance in high-volume Kubernetes environments, consider the following:
- **Transport/Collection Considerations:** This integration utilizes the `filestream` input to monitor JSON logs written to a local volume. This method ensures high throughput and low latency by reading directly from the filesystem. To maintain performance, ensure that the underlying storage for `/var/run/cilium/tetragon/` has sufficient IOPS to handle concurrent writes from Tetragon and reads from the Elastic Agent or Filebeat.
- **Data Volume Management:** Users can significantly reduce the volume of data ingested by utilizing Tetragon's native filtering capabilities. By configuring the **tetragon.exportAllowList** and **tetragon.exportDenyList** in the Helm values, you can exclude high-volume, low-value events at the source, reducing the processing load on both the collection agent and the Elastic Stack.
- **Elastic Agent Scaling:** For high-throughput Kubernetes clusters, deploy Elastic Agent as a DaemonSet to distribute the collection load across all nodes. This horizontal scaling approach ensures that as the cluster grows, the log collection capacity increases linearly. Ensure that the Elasticsearch cluster is appropriately sized to handle the aggregate peak ingest rate from all nodes.

# Set Up Instructions

## Vendor prerequisites

- **Kubernetes Cluster Access:** You must have administrative access to a Kubernetes cluster with `kubectl` configured and authenticated.
- **Helm Package Manager:** The `helm` CLI must be installed (version 3.x is recommended) to deploy the Tetragon chart.
- **Namespace Permissions:** Permissions to create and manage resources (ConfigMaps, Pods, Deployments) within the `kube-system` namespace.
- **Local Storage Access:** Tetragon must be configured to write logs to a `hostPath` (typically `/var/run/cilium/tetragon`) so the collection agent can access the log files.
- **Elasticsearch Credentials:** A valid Elasticsearch URL, username, and password with permissions to write to the `logs-tetragon.*` index.

## Elastic prerequisites

- **Integration Installation:** The Cilium Tetragon integration assets must be installed in Kibana via the Integrations app prior to data ingestion to ensure dashboards and mappings are available.
- **Elastic Stack Connectivity:** Kubernetes nodes must have network connectivity to the Elasticsearch endpoint (typically port `443` or `9200`) to ship logs successfully.
- **Filebeat Image Access:** The cluster must be able to pull the `docker.elastic.co/beats/filebeat:8.15.3` image from the Elastic container registry.

## Vendor set up steps

### For File Monitoring via Filebeat Sidecar:

1.  **Create the Filebeat Configuration:** Create a file named `filebeat-cfgmap.yaml` to define how Filebeat should process and ship the Tetragon logs.
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: filebeat-configmap
      namespace: kube-system
    data:
      filebeat.yml: |
        filebeat.inputs:
          - type: filestream
            id: tetragon-log
            enabled: true
            paths:
              - /var/run/cilium/tetragon/*.log
        output.elasticsearch:
          hosts: ["https://<your-elasticsearch-host>:9200"]
          username: "<your-elasticsearch-username>"
          password: "<your-elasticsearch-password>"
          index: "logs-tetragon.log-default"
    ```
2.  **Apply the ConfigMap:** Deploy the configuration to your Kubernetes cluster:
    ```bash
    kubectl apply -f filebeat-cfgmap.yaml
    ```
3.  **Prepare the Helm Values:** Create a `tetragon-helm-values.yaml` file to inject the Filebeat sidecar into the Tetragon DaemonSet. This ensures every node running Tetragon also runs a Filebeat instance.
    ```yaml
    tetragon:
      export:
        image:
          override: "docker.elastic.co/beats/filebeat:8.15.3"
        extraVolumeMounts:
          - name: filebeat-config
            mountPath: /usr/share/filebeat/filebeat.yml
            subPath: filebeat.yml
            readOnly: true
          - name: tetragon-logs
            mountPath: /var/run/cilium/tetragon
            readOnly: true
          - name: filebeat-data
            mountPath: /usr/share/filebeat/data
        extraVolumes:
          - name: filebeat-config
            configMap:
              name: filebeat-configmap
          - name: tetragon-logs
            hostPath:
              path: /var/run/cilium/tetragon
              type: DirectoryOrCreate
          - name: filebeat-data
            emptyDir: {}
    ```
4.  **Add the Cilium Repository:** Update your Helm repositories:
    ```bash
    helm repo add cilium https://helm.cilium.io
    helm repo update
    ```
5.  **Install Tetragon:** Deploy Tetragon using the custom values file:
    ```bash
    helm install tetragon cilium/tetragon --namespace kube-system -f tetragon-helm-values.yaml
    ```
6.  **Verify Pod Status:** Ensure the Tetragon pods are running and show `2/2` containers ready:
    ```bash
    kubectl -n kube-system get pods -l app.kubernetes.io/name=tetragon
    ```

## Kibana set up steps

### log : filestream
1. In Kibana, navigate to **Management > Integrations**.
2. Search for and select **Cilium Tetragon**.
3. Click **Add Cilium Tetragon**.
4. Configure the integration settings. Note that for this integration, the manual Filebeat sidecar handles data collection:
   - **Integration name**: Provide a unique name for this integration instance.
   - **Description**: Add an optional description.
5. Review the configuration. There are no additional user-facing variables (`vars`) required for this input in the Kibana UI as collection is performed via the Filebeat sidecar configuration.
6. Click **Save and continue**.
7. Click **Add integration only** to install the index templates, ingest pipelines, and dashboards.

# Validation Steps

After configuration is complete, verify that data is flowing correctly.

### 1. Trigger Data Flow on Tetragon:
- **Process execution:** Execute a command within any pod in the cluster: `kubectl exec -it <pod-name> -- ls /etc`.
- **Network activity:** Use `curl` inside a container to reach an external URL: `kubectl exec -it <pod-name> -- curl -I https://www.elastic.co`.
- **Pod lifecycle event:** Deploy a temporary test pod: `kubectl run validation-test --image=busybox -- sleep 30`. Tetragon will log the container start and process initialization.

### 2. Check Data in Kibana:
1. Navigate to **Analytics > Discover**.
2. Select the `logs-*` data view.
3. Enter the KQL filter: `data_stream.dataset : "tetragon.log"`
4. Verify logs appear. Expand a log entry and confirm these fields:
   - `event.dataset` (should be `tetragon.log`)
   - `process.name` (e.g., `ls`, `curl`, or `sleep`)
   - `event.action` (e.g., `process_exec` or `socket_connect`)
   - `message` (the raw log payload from Tetragon)
5. Navigate to **Analytics > Dashboards** and search for "Cilium Tetragon" to view the pre-built security overview.

# Troubleshooting

## Common Configuration Issues

- **File Permissions**: If Filebeat logs show "permission denied" when accessing `/var/run/cilium/tetragon/`, ensure the `securityContext` for the Filebeat sidecar is set to `runAsUser: 0` (root) or that the user has read access to the shared volume.
- **Namespace Mismatch**: The ConfigMap must exist in the same namespace as the Tetragon deployment (typically `kube-system`). If they are mismatched, the sidecar pod will fail to mount the configuration volume.
- **Volume Mount Path**: Ensure that the `mountPath` in the Helm values matches the `paths` defined in the Filebeat configuration. If Tetragon writes logs to a different directory than Filebeat monitors, no data will be ingested.
- **Index Template Conflicts**: If data is appearing with incorrect mappings or as `keyword` fields, ensure you installed the integration assets in Kibana before starting the data flow.

## Ingestion Errors

- **JSON Parsing Failures**: If the `message` field is present but other fields are not parsed, ensure Tetragon is configured to export in JSON format (this is the default for file exports).
- **Mapping Conflicts**: If you see `mapper_parsing_exception`, ensure you are using the integration-provided index name `logs-tetragon.log-default` to utilize the correct ECS templates.
- **Event Filtering**: If expected events are missing, check the `tetragon.exportAllowList` in your Helm values to ensure the desired event types (e.g., `process_exec`) are not being filtered out at the source.

## Vendor Resources

- Tetragon Official Documentation
- [Tetragon Kubernetes Installation Guide](installation/kubernetes/)
- [Tetragon Official Documentation - Configuration](installation/configuration/)
- [Tetragon Official Documentation - Deploy as a container](installation/container/)

## Documentation sites

- [Elastic integration reference for Cilium Tetragon](https://www.elastic.co/docs/reference/integrations/cilium_tetragon)
- Official Tetragon Documentation
