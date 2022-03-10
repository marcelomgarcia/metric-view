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
* Network host setting: `192.168.56.10`
* Cluster Initial Master nodes: `atta`
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

### Basic Security

Adding [basic security](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/security-minimal-setup.html) to Elasticsearch. First remove the `initial_master_nodes` option, the enable the security features.

```
vagrant@atta:~$ sudo grep initial_master /etc/elasticsearch/elasticsearch.yml
#cluster.initial_master_nodes: ["atta"]
vagrant@atta:~$ 
vagrant@atta:~$ sudo tail -n 4 /etc/elasticsearch/elasticsearch.yml
# https://www.elastic.co/guide/en/elasticsearch/reference/7.16/configuring-stack-security.html
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
discovery.type: single-node
vagrant@atta:~$
```

Setting the password for the buit-in users. 

```
vagrant@atta:/usr/share/elasticsearch/bin$ sudo ./elasticsearch-setup-passwords interactive
Initiating the setup of passwords for reserved users elastic,apm_system,kibana,kibana_system,logstash_system,beats_system,remote_monitoring_user.
You will be prompted to enter passwords as the process progresses.
Please confirm that you would like to continue [y/N]y


Enter password for [elastic]: 
Reenter password for [elastic]: 
Enter password for [apm_system]: 
Reenter password for [apm_system]: 
Enter password for [kibana_system]: 
Reenter password for [kibana_system]: 
Enter password for [logstash_system]: 
Reenter password for [logstash_system]: 
Enter password for [beats_system]: 
Reenter password for [beats_system]: 
Enter password for [remote_monitoring_user]: 
Reenter password for [remote_monitoring_user]: 
Changed password for user [apm_system]
Changed password for user [kibana_system]
Changed password for user [kibana]
Changed password for user [logstash_system]
Changed password for user [beats_system]
Changed password for user [remote_monitoring_user]
Changed password for user [elastic]
vagrant@atta:/usr/share/elasticsearch/bin$
```

The [password](https://www.saotn.org/generate-random-passwords-openssl/) was created using the `openssl` command, for example to create a random 12 bytes base64 sequence

```
vagrant@atta:~$ openssl rand -base64 12
NyPr4rt1w6MAMmA8
vagrant@atta:~$
```

Next we configure Kibana to use the `kibana_system` password to connect to elasticsearch

```
vagrant@atta:/usr/share/kibana$ sudo more /etc/kibana/kibana.yml 
(...)
elasticsearch.username: "kibana_system"
vagrant@atta:/usr/share/kibana$
vagrant@atta:/usr/share/kibana/bin$ sudo ./kibana-keystore add elasticsearch.password
Enter value for elasticsearch.password: ****************
vagrant@atta:/usr/share/kibana/bin$
vagrant@atta:/usr/share/kibana/bin$ sudo systemctl start kibana
```

Finally we can log into Kibana using the _elastic_ user

```
http://192.168.56.10:5601/app/home#/
```

## Beats

Testing the output of `metricbeat`

```
vagrant@hopper:~$ sudo metricbeat test output
elasticsearch: http://192.168.56.10:9200...
  parse url... OK
  connection...
    parse host... OK
    dns lookup... OK
    addresses: 192.168.56.10
    dial up... OK
  TLS... WARN secure connection disabled
  talk to server... ERROR 401 Unauthorized: {"error":{"root_cause":[{"type":"security_exception","reason":"missing authentication credentials for REST request [/]","header":{"WWW-Authenticate":"Basic realm=\"security\" charset=\"UTF-8\""}}],"type":"security_exception","reason":"missing authentication credentials for REST request [/]","header":{"WWW-Authenticate":"Basic realm=\"security\" charset=\"UTF-8\""}},"status":401}
vagrant@hopper:~$ 
```

Listing the modules available, that can be _enabled_ or _disabled_

```
agrant@hopper:~$ sudo metricbeat modules list
Enabled:
system 
                                      
Disabled:
activemq
aerospike
airflow
apache
(...)
```

To configure user authentication, enter the credentials on the `metricbeat.yml` file

```
vagrant@hopper:~$ sudo cat /etc/metricbeat/metricbeat.yml
(...)
# ---------------------------- Elasticsearch Output ----------------------------                                                                          
output.elasticsearch:                                                                                                                                     
  # Array of hosts to connect to.                                            
  hosts: ["192.168.56.10:9200"]                                                                                                                           
  (...)

  # Authentication credentials - either API key or username/password.
  #api_key: "id:api_key"
  username: "elastic"
  password: "abc123"
(...)
```

Test the output again, and check if the beat is talking with the server

```
vagrant@hopper:~$ sudo metricbeat test output
elasticsearch: http://192.168.56.10:9200...
(...)
  talk to server... OK
  version: 7.17.1
vagrant@hopper:~$ 
```

If everything is OK, restart the service

```
vagrant@hopper:~$ sudo systemctl restart metricbeat 
```
