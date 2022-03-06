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

## Elasticsearch

The [configuration of Elasticsearch on Ubuntu](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/deb.html) is done editing the file `/etc/elasticsearch/elasticsearch.yml.` For Debian/Ubuntu package defines a system configuration file, where sysetm variables, like `ES_JAVA_HOME` or `MAX_OPEN_FILES` can be defined. One very important variable is `ES_HOME` that points to `/usr/share/elasticsearch.`

The [following parameters were set](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/important-settings.html):

* Cluster name: `metric-vis`
* Network host setting (TODO)
* Cluster Initial Master nodes (TODO)
* Cluster backup (TODO)

But no [security settings](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/secure-settings.html), or [auditing security settings](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/auditing-settings.html) were applied so far (TODO). 

Basic configuration 

```
vagrant@atta:~$ sudo grep -v '^#' /etc/elasticsearch/elasticsearch.yml
cluster.name: metric-vis
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
network.host: 192.168.56.10
cluster.initial_master_nodes: ["atta"]
vagrant@atta:~$
```

For Kibana

```
vagrant@atta:~$ sudo grep -v '^#' /etc/kibana/kibana.yml | grep -v '^$'
server.port: 5601
server.host: "192.168.56.10"
server.publicBaseUrl: "http://192.168.56.10"
elasticsearch.hosts: ["http://192.168.56.10:9200"]
vagrant@atta:~$ 
```

