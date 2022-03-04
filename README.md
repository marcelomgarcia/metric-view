# Metric Visualization

Using ELK stack to visualize metrics of the system.

## Vagrant

Define 3 VMs for the cluster

| VM | IP | Function |
| --- | --- | --- |
| atta | 192.168.56.10 | Elasticsearch and Kibana |
| flik | 192.168.56.11 | Logstash and beats |
| hopper | 192.168.56.12 | Beats |

`Hopper` will send data to Logstash running on `flik,` which in turn, will send its own metrics plus the metrics from `hopper` to `atta`. Finally the Kibana dashboard will be `atta` too.

