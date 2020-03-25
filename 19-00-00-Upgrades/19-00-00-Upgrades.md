# Upgrades #

## Upgrade from 6.x

The update includes packages:

- Energy-logserver-data-node
- Energy-logserver-client-node

### Upgrade Energy Logserver Data Node

1. Upload Package

```bash
scp ./energy-logserver-data-node-7.0.1-1.el7.x86_64.rpm root@hostname:~/
```

2. Check Cluster Status

```bash
export CREDENTIAL="logserver:logserver"

curl -s -u $CREDENTIAL localhost:9200/_cluster/health?pretty
```

​	Output:

```bash
{
    "cluster_name" : "elasticsearch",
    "status" : "green",
    "timed_out" : false,
    "number_of_nodes" : 1,
    "number_of_data_nodes" : 1,
    "active_primary_shards" : 25,
    "active_shards" : 25,
    "relocating_shards" : 0,
    "initializing_shards" : 0,
    "unassigned_shards" : 0,
    "delayed_unassigned_shards" : 0,
    "number_of_pending_tasks" : 0,
    "number_of_in_flight_fetch" : 0,
    "task_max_waiting_in_queue_millis" : 0,
    "active_shards_percent_as_number" : 100.0
   }
```

3. Upgrade Energy Logserver Package

```bash
yum update ./energy-logserver-data-node-7.0.1-1.el7.x86_64.rpm
```

​	Output:

```bash
Loaded plugins: fastestmirror
Examining ./energy-logserver-data-node-7.0.1-1.el7.x86_64.rpm: energy-logserver-data-node-7.0.1-1.el7.x86_64
Marking ./energy-logserver-data-node-7.0.1-1.el7.x86_64.rpm as an update to energy-logserver-data-node-6.1.8-1.x86_64
Resolving Dependencies
--> Running transaction check
---> Package energy-logserver-data-node.x86_64 0:6.1.8-1 will be updated
---> Package energy-logserver-data-node.x86_64 0:7.0.1-1.el7 will be an update
--> Finished Dependency Resolution

Dependencies Resolved
=======================================================================================================================================================================================
 Package                                        Arch                       Version                            Repository                                                          Size
=======================================================================================================================================================================================
Updating:
 energy-logserver-data-node                     x86_64                     7.0.1-1.el7                        /energy-logserver-data-node-7.0.1-1.el7.x86_64                     117 M

Transaction Summary
=======================================================================================================================================================================================
Upgrade  1 Package

Total size: 117 M
Is this ok [y/d/N]: y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : energy-logserver-data-node-7.0.1-1.el7.x86_64                                                                                                                       1/2
Removed symlink /etc/systemd/system/multi-user.target.wants/elasticsearch.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/elasticsearch.service to /usr/lib/systemd/system/elasticsearch.service.
  Cleanup    : energy-logserver-data-node-6.1.8-1.x86_64                                                                                                                           2/2
  Verifying  : energy-logserver-data-node-7.0.1-1.el7.x86_64                                                                                                                       1/2
  Verifying  : energy-logserver-data-node-6.1.8-1.x86_64                                                                                                                           2/2

Updated:
  energy-logserver-data-node.x86_64 0:7.0.1-1.el7

Complete!
```

4. Verification of Configuration Files

Please, verify your Elasticsearch configuration and JVM configuration in files:

- /etc/elasticsearch/jvm.options – check JVM HEAP settings and another parameters

```bash
grep Xm /etc/elasticsearch/jvm.options <- old configuration file
## -Xms4g
## -Xmx4g
# Xms represents the initial size of total heap space
# Xmx represents the maximum size of total heap space
-Xms600m
-Xmx600m
```

```bash
cp /etc/elasticsearch/jvm.options.rpmnew /etc/elasticsearch/jvm.options
cp: overwrite ‘/etc/elasticsearch/jvm.options’? y
```

```bash
vim /etc/elasticsearch/jvm.options
```

- /etc/elasticsearch/elasticsearch.yml – verify elasticsearch configuration file

- compare exiting /etc/elasticsearch/elasticsearch.yml and /etc/elasticsearch/elasticsearch.yml.rpmnew
  5. Start and enable Elasticsearch service

If everything went correctly, we will restart the Elasticsearch instance:

```bash
systemctl restart elasticsearch
systemctl reenable elasticsearch
```

```bash
systemctl status elasticsearch

● elasticsearch.service - Elasticsearch
   Loaded: loaded (/usr/lib/systemd/system/elasticsearch.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2020-03-18 16:50:15 CET; 57s ago
     Docs: http://www.elastic.co
 Main PID: 17195 (java)
   CGroup: /system.slice/elasticsearch.service
           └─17195 /etc/alternatives/jre/bin/java -Xms512m -Xmx512m -Djava.security.manager -Djava.security.policy=/usr/share/elasticsearch/plugins/elasticsearch_auth/plugin-securi...

Mar 18 16:50:15 migration-01 systemd[1]: Started Elasticsearch.
Mar 18 16:50:25 migration-01 elasticsearch[17195]: SSL not activated for http and/or transport.
Mar 18 16:50:33 migration-01 elasticsearch[17195]: SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
Mar 18 16:50:33 migration-01 elasticsearch[17195]: SLF4J: Defaulting to no-operation (NOP) logger implementation
Mar 18 16:50:33 migration-01 elasticsearch[17195]: SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for futher details.
```

6. Check cluster/indices status and Elasticsearch version

Invoke curl command to check the status of Elasticsearch:

```bash
curl -s -u $CREDENTIAL localhost:9200/_cluster/health?pretty
{
  "cluster_name" : "elasticsearch",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 25,
  "active_shards" : 25,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```

```bash
curl -s -u $CREDENTIAL localhost:9200
{
  "name" : "node-1",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "igrASEDRRamyQgy-zJRSfg",
  "version" : {
    "number" : "7.3.2",
    "build_flavor" : "oss",
    "build_type" : "rpm",
    "build_hash" : "1c1faf1",
    "build_date" : "2019-09-06T14:40:30.409026Z",
    "build_snapshot" : false,
    "lucene_version" : "8.1.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

If everything went correctly, we should see 100% allocated shards in cluster health. However, while connection on port 9200/TCP we can observe a new version of Elasticsearch.

### Upgrade Energy Logserver client Node

1. Upload packages

- Upload new rpm by scp/ftp:

```bash
scp ./energy-logserver-client-node-7.0.1-1.el7.x86_64.rpm root@hostname:~/
```

- Backup report logo file.

2. Uninstall old version Energy Logserver GUI

- Remove old package:

```bash
systemctl stop kibana alert
```

```bash
yum remove energy-logserver-client-node
Loaded plugins: fastestmirror
Resolving Dependencies
--> Running transaction check
---> Package energy-logserver-client-node.x86_64 0:6.1.8-1 will be erased
--> Finished Dependency Resolution

Dependencies Resolved

=======================================================================================================================================================================================
 Package                                           Arch                        Version                        Repository                                                          Size
=======================================================================================================================================================================================
Removing:
 energy-logserver-client-node                      x86_64                      6.1.8-1                        @/energy-logserver-client-node-6.1.8-1.x86_64                      802 M

Transaction Summary
=======================================================================================================================================================================================
Remove  1 Package

Installed size: 802 M
Is this ok [y/N]: y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Erasing    : energy-logserver-client-node-6.1.8-1.x86_64                                                                                                                         1/1
warning: file /usr/share/kibana/plugins/node_modules.tar: remove failed: No such file or directory
warning: /etc/kibana/kibana.yml saved as /etc/kibana/kibana.yml.rpmsave
  Verifying  : energy-logserver-client-node-6.1.8-1.x86_64                                                                                                                         1/1

Removed:
  energy-logserver-client-node.x86_64 0:6.1.8-1

Complete!
```

3. Install new version

- Install new package:

  ```bash
  yum install ./energy-logserver-client-node-7.0.1-1.el7.x86_64.rpm
  Loaded plugins: fastestmirror
  Examining ./energy-logserver-client-node-7.0.1-1.el7.x86_64.rpm: energy-logserver-client-node-7.0.1-1.el7.x86_64
  Marking ./energy-logserver-client-node-7.0.1-1.el7.x86_64.rpm to be installed
  Resolving Dependencies
  --> Running transaction check
  ---> Package energy-logserver-client-node.x86_64 0:7.0.1-1.el7 will be installed
  --> Finished Dependency Resolution
  
  Dependencies Resolved
  
  =======================================================================================================================================================================================
   Package                                         Arch                      Version                           Repository                                                           Size
  =======================================================================================================================================================================================
  Installing:
   energy-logserver-client-node                    x86_64                    7.0.1-1.el7                       /energy-logserver-client-node-7.0.1-1.el7.x86_64                    1.2 G
  
  Transaction Summary
  =======================================================================================================================================================================================
  Install  1 Package
  
  Total size: 1.2 G
  Installed size: 1.2 G
  Is this ok [y/d/N]: y
  Downloading packages:
  Running transaction check
  Running transaction test
  Transaction test succeeded
  Running transaction
    Installing : energy-logserver-client-node-7.0.1-1.el7.x86_64                                                                                                                     1/1
  Generating a 2048 bit RSA private key
  ..............................................................................................+++
  ...........................................................................................................+++
  writing new private key to '/etc/kibana/ssl/kibana.key'
  -----
  Removed symlink /etc/systemd/system/multi-user.target.wants/alert.service.
  Created symlink from /etc/systemd/system/multi-user.target.wants/alert.service to /usr/lib/systemd/system/alert.service.
  Removed symlink /etc/systemd/system/multi-user.target.wants/kibana.service.
  Created symlink from /etc/systemd/system/multi-user.target.wants/kibana.service to /usr/lib/systemd/system/kibana.service.
  Removed symlink /etc/systemd/system/multi-user.target.wants/cerebro.service.
  Created symlink from /etc/systemd/system/multi-user.target.wants/cerebro.service to /usr/lib/systemd/system/cerebro.service.
    Verifying  : energy-logserver-client-node-7.0.1-1.el7.x86_64                                                                                                                     1/1
  
  Installed:
    energy-logserver-client-node.x86_64 0:7.0.1-1.el7
  
  Complete!
  ```

4. Start Energy Logserver GUI

   Add service:

   - ​	Kibana
   - Cerebro
   - Alert

   to autostart, add port ( 5602/TCP ) for Cerebro. 

   Run them and check status:

   ```bash
   firewall-cmd –permanent –add-port 5602/tcp
   firewall-cmd –reload
   ```

   ```bash
   systemctl enable kibana cerebro alert
   Created symlink from /etc/systemd/system/multi-user.target.wants/kibana.service to /usr/lib/systemd/system/kibana.service.
   Created symlink from /etc/systemd/system/multi-user.target.wants/cerebro.service to /usr/lib/systemd/system/cerebro.service.
   Created symlink from /etc/systemd/system/multi-user.target.wants/alert.service to /usr/lib/systemd/system/alert.service.
   [root@migration-01 vagrant]#
   [root@migration-01 vagrant]# systemctl start kibana cerebro alert
   [root@migration-01 vagrant]# systemctl status kibana cerebro alert
   ● kibana.service - Kibana
      Loaded: loaded (/usr/lib/systemd/system/kibana.service; enabled; vendor preset: disabled)
      Active: active (running) since Thu 2020-03-19 14:46:52 CET; 2s ago
    Main PID: 12399 (node)
      CGroup: /system.slice/kibana.service
              └─12399 /usr/share/kibana/bin/../node/bin/node --no-warnings --max-http-header-size=65536 /usr/share/kibana/bin/../src/cli -c /etc/kibana/kibana.yml
   
   Mar 19 14:46:52 migration-01 systemd[1]: Started Kibana.
   
   ● cerebro.service - Cerebro
      Loaded: loaded (/usr/lib/systemd/system/cerebro.service; enabled; vendor preset: disabled)
      Active: active (running) since Thu 2020-03-19 14:46:52 CET; 2s ago
    Main PID: 12400 (java)
      CGroup: /system.slice/cerebro.service
              └─12400 java -Duser.dir=/opt/cerebro -Dconfig.file=/opt/cerebro/conf/application.conf -cp -jar /opt/cerebro/lib/cerebro.cerebro-0.8.4-launcher.jar
   
   Mar 19 14:46:52 migration-01 systemd[1]: Started Cerebro.
   
   ● alert.service - Alert
      Loaded: loaded (/usr/lib/systemd/system/alert.service; enabled; vendor preset: disabled)
      Active: active (running) since Thu 2020-03-19 14:46:52 CET; 2s ago
    Main PID: 12401 (elastalert)
      CGroup: /system.slice/alert.service
              └─12401 /opt/alert/bin/python /opt/alert/bin/elastalert
   
   Mar 19 14:46:52 migration-01 systemd[1]: Started Alert.
   
   ```

   
