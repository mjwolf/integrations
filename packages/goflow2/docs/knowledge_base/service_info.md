# Service Info

## Common use cases

GoFlow2 is used for network traffic monitoring and capacity planning by aggregating flow data from diverse routers and switches into a unified format. It facilitates security forensics and DDoS detection by providing high-visibility into network-level IP traffic, source/destination patterns, and protocol distribution. Additionally, it supports peering and transit analysis to optimize routing costs and performance across multi-vendor network environments.

## Data types collected

This integration primarily collects network flow logs. The data includes fields for source and destination IP addresses, port numbers, protocol information, autonomous system (AS) numbers, interface indices, and packet/byte counts.

## Compatibility

GoFlow2 is compatible with major network flow protocols including NetFlow v5/v9, IPFIX, and sFlow v5. It has been tested as a lightweight collector capable of running on most Linux distributions and within Docker environments.

## Scaling and Performance

GoFlow2 is designed for high-performance horizontal scaling and can process thousands of flows per second using Go's native concurrency features. Performance is largely dependent on the underlying disk I/O when using the file transport and the allocated CPU resources for packet decoding.

# Set Up Instructions

## Vendor prerequisites

A network environment with devices capable of exporting NetFlow v5/v9, IPFIX, or sFlow is required. You must have administrative access to the host or container environment to run the GoFlow2 binary and permissions to write to the designated log directory (e.g., `/var/log/`).

## Elastic prerequisites

The Elastic Agent must be installed on the same host as GoFlow2 or have access to the shared volume where the JSON logs are written. The agent policy requires a `filestream` input configured to monitor the specific path of the GoFlow2 output file.

## Vendor set up steps

1. **Install GoFlow2**: Download the latest binary from the [GitHub releases page](https://github.com/netsampler/goflow2/releases) or use the Docker image: `docker run -p 6343:6343/udp -p 2055:2055/udp netsampler/goflow2:latest`.
2. **Define Listeners**: Configure the collector to listen on the appropriate ports for your exporters (default 6343 for sFlow, 2055 for NetFlow/IPFIX).
3. **Configure File Output**: Use the `-transport file` flag to direct flow data to a local file.
4. **Set Format to JSON**: Append the `-format json` flag to ensure the output is in a machine-readable format for the Elastic Agent.
5. **Run Command**: Execute the collector with the combined configuration: `./goflow2 -listen 'sflow://:6343,netflow://:2055' -transport file -transport.file /var/log/goflow2.log -format json`.

## Kibana set up steps

1. Log in to Kibana and navigate to **Management > Integrations**.
2. Search for and select the **Custom Logs** or **Filestream** integration.
3. Add the integration to your Elastic Agent policy.
4. Set the **Log file path** to the exact path configured in GoFlow2 (e.g., `/var/log/goflow2.log`).
5. Under **Custom configurations**, ensure any specific JSON parsing processors are added if you wish to map GoFlow2 fields directly to ECS (Elastic Common Schema).

# Validation Steps

1. **Verify GoFlow2 Process**: Ensure the GoFlow2 service is running and actively listening on the UDP ports using `netstat -ulnp`.
2. **Check Log Generation**: Confirm that the log file is being populated with JSON data using `tail -f /var/log/goflow2.log`.
3. **Inspect Data in Kibana**: Use **Discover** in Kibana to search for incoming data from the specific file path or integration name.
4. **Trigger Flow Data**: Force a network flow from a router or use a flow generator tool to verify real-time ingestion.

# Troubleshooting

## Common Configuration Issues

Connectivity issues often arise from host firewalls (iptables/ufw) or cloud security groups blocking UDP ports 2055 or 6343. If the log file is empty, verify that your network devices are correctly pointed to the GoFlow2 IP address and that the `-transport.file` path is writable by the GoFlow2 process.

## Ingestion Errors

Parsing errors typically occur if the GoFlow2 output format is not set to `json`, causing the Elastic Agent to treat the entire log line as a single string. Ensure the `-format json` flag is present in the GoFlow2 startup command and that the Elastic Agent has the appropriate permissions to read the log file.

## API Authentication Errors

Not specified. GoFlow2 file transport does not utilize API-based authentication for local file writing.

## Vendor Resources

- [GoFlow2 GitHub Issues](https://github.com/netsampler/goflow2/issues)
- [GoFlow2 Troubleshooting Discussions](https://github.com/netsampler/goflow2/discussions)

# Documentation sites

- [Official GoFlow2 GitHub Repository](https://github.com/netsampler/goflow2)
- [Installation and Setup Guide](https://deepwiki.com/netsampler/goflow2/1.2-installation-and-setup)
- [GoFlow2 Go Package Documentation](https://pkg.go.dev/github.com/netsampler/goflow2/v2)