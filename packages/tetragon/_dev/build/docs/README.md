# Cilium Tetragon Integration for Elastic

## Overview

The Cilium Tetragon integration for Elastic collects security observability logs from containerized applications in Kubernetes environments. It uses eBPF (extended Berkeley Packet Filter) technology to provide deep, kernel-level visibility into process executions, system calls, and other activities with minimal performance overhead.

This integration facilitates:
- **Kubernetes Security Observability**: Monitor and analyze security events from containerized applications, providing real-time visibility into kernel-level activities.
- **Runtime Threat Detection**: Detect and respond to security threats by analyzing process behavior, file access patterns, and system call activities.
- **Compliance and Audit Logging**: Capture detailed audit trails of process executions, exits, and kernel probe events with full Kubernetes context for compliance and forensic analysis.
- **Container Security Monitoring**: Track process lifecycle events and system activities within containers, correlating security events with Kubernetes metadata.

### Compatibility

This integration is compatible with the following:

- **Kubernetes**: Requires a running Kubernetes cluster where Tetragon can be deployed as a DaemonSet.
- **Linux Kernel**: Requires a Linux kernel with eBPF support.
- **Elastic Stack**: Requires Kibana version 8.13.0 or higher, or 9.0.0 or higher.
- **Integration Type**: Uses Filebeat for log forwarding. Elastic Agent is not supported for this integration.

### How it works

The Cilium Tetragon integration operates by deploying Tetragon as a DaemonSet within a Kubernetes cluster. Tetragon uses eBPF to monitor kernel-level activities and writes events to a log file on each node. A Filebeat container, running as a sidecar in the Tetragon pod, reads these logs and forwards them to your Elastic deployment for analysis and visualization.

## What data does this integration collect?

The Cilium Tetragon integration collects the following types of log data, enriched with Kubernetes metadata (pod names, namespaces, labels, etc.):

- **Process Execution Events** (`process_exec`): Details about process starts, including executable paths, arguments, user IDs, and parent-child relationships.
- **Process Exit Events** (`process_exit`): Information about process terminations, including exit status and signals.
- **Kernel Probe Events** (`process_kprobe`): Data from system calls and kernel functions, such as file operations, capability checks, and custom policy-based events.

### Supported use cases

- **Runtime Threat Detection**: Detect suspicious activities like unexpected process executions or privilege escalations within containers.
- **Security Auditing**: Maintain a comprehensive log of all process and system call activities for compliance with security standards.
- **Incident Investigation**: Use detailed, contextualized logs to investigate security incidents and understand the impact of a breach.
- **Container Security Monitoring**: Gain deep visibility into the behavior of applications running in your Kubernetes environment.

## What do I need to use this integration?

### Prerequisites

#### Vendor Prerequisites

- A running Kubernetes cluster with nodes that support eBPF.
- Helm 3.x installed to deploy Tetragon via Helm charts.
- Administrative access to the Kubernetes cluster to create resources (e.g., DaemonSets, ConfigMaps) in the `kube-system` namespace.

#### Elastic Prerequisites

- An Elasticsearch and Kibana deployment (version 8.13.0+ or 9.0.0+).
- Elasticsearch credentials (username and password) with permissions to create and write to indices.
- Network connectivity from the Kubernetes cluster to your Elasticsearch endpoint.

## How do I deploy this integration?

This integration uses Filebeat deployed as a sidecar to the Tetragon agent. It does not use a standard Elastic Agent installation.

### Set up steps in Cilium Tetragon

#### Step 1: Create Filebeat ConfigMap

First, create a `ConfigMap` in the `kube-system` namespace to hold the Filebeat configuration. This configuration tells Filebeat where to find the Tetragon log files and how to send them to Elasticsearch.

1.  Save the following content as `filebeat-cfgmap.yaml`. Remember to replace the placeholders for your Elasticsearch host, username, and password.

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
        path.data: /usr/share/filebeat/data
        processors:
          - timestamp:
              field: "time"
              layouts:
                - '2006-01-02T15:04:05Z'
                - '2006-01-02T15:04:05.999Z'
                - '2006-01-02T15:04:05.999-07:00'
              test:
                - '2019-06-22T16:33:51Z'
                - '2019-11-18T04:59:51.123Z'
                - '2020-08-03T07:10:20.123456+02:00'
        setup.template.name: logs
        setup.template.pattern: "logs-cilium_tetragon.*"
        output.elasticsearch:
          hosts: ["https://<elasticsearch_host>"]
          username: "<elasticsearch_username>"
          password: "<elasticsearch_password>"
          index: logs-cilium_tetragon.log-default
    ```

2.  Apply the configuration to your cluster:

    ```shell
    kubectl create -f filebeat-cfgmap.yaml
    ```

#### Step 2: Install Tetragon with a Filebeat Sidecar

Next, create a Helm values file to configure the Tetragon deployment. This configuration will add Filebeat as a sidecar container to the Tetragon pods.

1.  Save the following content as `filebeat-helm-values.yaml`:

    ```yaml
    export:
      securityContext:
        runAsUser: 0
        runAsGroup: 0
      stdout:
        enabledCommand: false
        enabledArgs: false
        image:
          override: "docker.elastic.co/beats/filebeat:8.15.3"
        extraVolumeMounts:
          - name: filebeat-config
            mountPath: /usr/share/filebeat/filebeat.yml
            subPath: filebeat.yml
          - name: filebeat-data
            mountPath: /usr/share/filebeat/data
    extraVolumes:
      - name: filebeat-data
        hostPath:
          path: /var/run/cilium/tetragon/filebeat
          type: DirectoryOrCreate
      - name: filebeat-config
        configMap:
          name: filebeat-configmap
          items:
            - key: filebeat.yml
              path: filebeat.yml
    ```

2.  Add the Cilium Helm repository and install Tetragon using your custom values file:

    ```shell
    helm repo add cilium https://helm.cilium.io
    helm repo update
    helm install tetragon -f filebeat-helm-values.yaml cilium/tetragon -n kube-system
    ```

#### Step 3: Verify the Deployment

Check the status of the Tetragon DaemonSet to ensure it has been rolled out successfully:

```shell
kubectl rollout status -n kube-system ds/tetragon -w
```

### Set up steps in Kibana

#### Step 1: Install Integration Assets

1.  In Kibana, navigate to **Management > Integrations**.
2.  Search for "Cilium Tetragon" and select the integration.
3.  Click **Add Cilium Tetragon**.
4.  Since this integration does not use Elastic Agent, select **Add integration only**.
5.  Confirm the installation to load the necessary assets, such as dashboards, visualizations, and ingest pipelines.

#### Step 2: Verify Data Stream

1.  Navigate to **Management > Dev Tools > Index Management**.
2.  Select the **Data Streams** tab.
3.  Look for the `logs-cilium_tetragon.log-default` data stream and verify that the document count is increasing.

### Validation Steps

1.  **Verify Tetragon Pods**: Check that the Tetragon pods are running in the `kube-system` namespace:
    ```shell
    kubectl get pods -n kube-system -l app.kubernetes.io/name=tetragon
    ```

2.  **Check Filebeat Logs**: Inspect the logs of the `export-stdout` container (the Filebeat sidecar) in one of the Tetragon pods:
    ```shell
    kubectl logs -n kube-system <tetragon-pod-name> -c export-stdout
    ```
    Look for messages indicating a successful connection to Elasticsearch.

3.  **Check Data in Kibana**:
    - In Kibana, navigate to **Discover**.
    - Select the `logs-cilium_tetragon.log-default` data view.
    - Verify that events are appearing with recent timestamps and contain fields like `process_exec`, `process_exit`, and Kubernetes metadata.

4.  **Generate Test Events**:
    - Exec into any pod in your cluster: `kubectl exec -it <any-pod> -- /bin/sh`
    - Run a few commands like `ls` or `whoami`.
    - Verify that the corresponding `process_exec` and `process_exit` events appear in Kibana.

## Troubleshooting

### Common Configuration Issues

-   **No data in Elasticsearch**:
    -   Verify the Filebeat ConfigMap was created correctly in the `kube-system` namespace.
    -   Check the logs of the Tetragon pods and the Filebeat sidecar for errors.
    -   Confirm your Elasticsearch credentials and host URL are correct in the ConfigMap.
    -   Test network connectivity from your Kubernetes cluster to the Elasticsearch endpoint.

-   **Tetragon pods are not starting**:
    -   Use `kubectl describe pod -n kube-system <tetragon-pod-name>` to check for events like image pull errors or scheduling issues.
    -   Ensure your worker nodes have a Linux kernel version that supports eBPF (kernel 4.9+ recommended, 5.3+ for full features).

-   **Missing event types**:
    -   Check the `tetragon.exportAllowList` and `tetragon.exportDenyList` Helm values in your deployment to ensure you are not filtering out expected events.

### Ingestion Errors

-   **Events have an `error.message` field**:
    -   Examine the error message to identify parsing or processing issues.
    -   Ensure the Tetragon version is compatible with the integration's ingest pipeline.
    -   Verify that the Filebeat timestamp processor configuration matches the timestamp format in the Tetragon logs.

-   **Missing Kubernetes metadata**:
    -   Confirm that Tetragon has the necessary RBAC permissions to access the Kubernetes API.
    -   Check for network policies that might block Tetragon's access to the Kubernetes API server.

### Vendor Resources

-   [Tetragon Official Documentation](https://tetragon.io/docs/)
-   [Tetragon Installation Guide](https://tetragon.io/docs/installation/kubernetes/)
-   [Tetragon GitHub Repository](https://github.com/cilium/tetragon)

## Performance and scaling

-   **eBPF-based Architecture**: Tetragon's use of eBPF provides deep kernel observability with minimal performance overhead.
-   **Event Filtering**: To manage data volume, you can configure `tetragon.exportAllowList` and `tetragon.exportDenyList` in your Helm values to control which events are exported.
-   **DaemonSet Deployment**: Tetragon scales automatically as you add or remove nodes in your Kubernetes cluster.

For more information on architectures that can be used for scaling this integration, check the [Ingest Architectures](https://www.elastic.co/docs/manage-data/ingest/ingest-reference-architectures) documentation.

## Reference

### log

The `log` data stream collects security observability events from Cilium Tetragon. It includes details on process execution, process exit, and kernel probe events, enriched with Kubernetes metadata.

#### log fields

{{ fields "log" }}

### Inputs used

{{ inputDocs }}
