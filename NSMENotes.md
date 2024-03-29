# NSM Engineer Course

## Table of Contents
  1) [Installing & Configuring CentOS](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#installing--configuring-centos)
  2) [Download REPO stuff](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#download-repo-stuff)
  3) [Install & Configure Stenographer](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#installing--configuring-stenographer)
  4) [Install & Configure Suricata](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#installing--configuring-suricata)
  5) [Install & Configure FSF](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#installing--configuring-fsf)
  6) [Kafka & Zookeeper](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#kafka)
  7) [Installing & Configuring Zeek](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#installing--configuring-zeek)
  8) [Filebeat](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#filebeat)
  9) [Logstash](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#logstash)
  10) [Elasticsearch](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#elasticsearch)
  11) [Kibana](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#kibana)
  12) [Troubleshooting](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#troubleshooting)
  13) [Resource Allocation for 1Gb/s Capture](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#resource-allocation-for-1gbs-capture)

# pfSense Installation

## Hardware Setup
    1. ESC - enter BIOS Setup
    2. F11 - boot new

    - this is a fast setting, be ready!!

## Initial Installation
    1. A - accept
    2. I - install (OK)
    3. <enter> - default US Keymap
    4. Auto - Guided Disk Setup
    5. Accept All Defaults
    <br> ... install runs ...

## Initial Config

1) Assign Interfaces

```
1 - assign interface ,
n - skip vlan Setup
en0 - WAN Interface
en1 - LAN Interface
(nothing to finish) Enter
y - proceed

reloading....

```
2) Set interface IPs

```
2 - set WAN interface
1 - WAN Interface
y - dhcp
n - dhcp6
blank for "none"
y - http webconfig protocol

2 - LAN interface
172.16.10.1/24
<Enter> for none
<Enter> for none - disable ipv6
y - enable dhcp
172.16.10.100 - range start
172.16.10.254 - range end
y - http webconfig protocol

```

## Finishing Up (Wizard)

#### In order to access the router, complete the following: <br>
  1. Set your lab machine to static ip 172.16.xx.10
  2. plug into the LAN interface (LAN2 on front face) and browse to 172.16.xx.1
  3. complete first login to web Guided

default creds:
```
admin / pfsense
```

### Wizerd > Pfsense Setup > General Information

```
set hostname sg10-pfsense localdomain == local.lan
primary dns
uncheck block private networks
LAN ip address - 172.16.10.1
subnet mask - 24
set (and document) new admin password - New password is: training

Add a WAN rule to allow laptops to connect to sensors

  - Source: 192.168.2.0/24 any any any

Disable DNS resolver

Enable DNS forwarder

Add a custom option under DNS Forwarder to allow RFC 1918 rebinding of lab-repo

rebind-domain-ok=/lab-repo/
```

# Day 2

## Installing & Configuring CentOS

[Return To Top](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#nsm-engineer-course)



  1. Plug in CentOS USB
  2. Click on Network & hostname
  3. Change Hostname to sg##.lab.lan
  4. Click "Apply"
  5. Configure top Interface
    - Click on IPv4
    - Click "Method"
    - Set to auto dhcp
  6. Click "Done"
  7. Click ipv6
    - Click "Ignore"
  8. Click "Save" & "Done"

  9. Go to KDUMP
    - Disable KDUMP
    - Click "Done"

  10. Click "Installation Destination"
    - Check both of the disks
    - Click "Done"
    - Reclaim Space
    - Delete all
    - Reclaim Space
    - Click "Installation Destination"
    - Click "I will configure partitioning"
    - Click "Done"
    - Click here to create them automatically

    11. Configuring Storage
      - /home
        - Desired Capacity - 1GiB
      - /
        - Desired Capacity - 1GiB
      - swap
        - Desired Capacity - 1GiB

   ### Add Mount Points
      - Click "+" on the bottom left
      #### Set all to 1GiB
      - /tmp
      - /var
      - /var/log
      - /var/log/audit
      - /var/tmp
      - /data/stenographer
      - /data/suricata
      - /data/kafka
      - /data/elasticsearch
      - /data/zeek
      - /data/fsf

      - /
        - Volume Group
          - Modify
            - Only select the drive that's ~500G
            - Click "Save"

      - /data/zeek
        - Create New Volume Group
          - Select ~1TB drive
          - Name it CentOS_data
          - Click "Save"
      - ###### Add all /data/* mount points to the volume group CentOS_data that you created above

      #### The above was done to separate the ingress/egress data from the system data. That way if one of the logs fills up it doesn't crash the whole system.

      ### Configure storage space for all of the new mount Points
        - /tmp - 5 GiB
        - /var - 50 GiB
        - /var/log - 50 GiB
        - /var/log/audit - 50 GiB
        - /var/tmp - 10 GiB
        - /home - 50 GiB
        - swap - 10 GiB
        - /data/stenographer - 500 GiB  
        - /data/Suricata - 25 GiB
        - /data/kafka - 100 Gib
        - /data/zeek - 25 GiB
        - /data/fsf -  10 GiB
        - /data/elasticsearch - leave blank (do this at the end)
        - / - leave blank (do this at the end)
        ###### Leaving the storage size blank automatically allocates the remaining storage to that mount point(s)

        - Click "Done" & 'Accept'
        - Click begin Installation  
          - don't create root password
        - User creation
          - Create an admin User
          - Password is: training
          - check the box to "Make the user an admin"
          - Click done

        - Unplug USB when it reboots

## Post Install
  1. check hostname:
    `hostname `
    - if you didn't set your hostname correctly
    `sudo hostnamectl set-hostname  `
    2. Check that ipv6 is disabled:

```vim
sudo vi /etc/sysctl.conf
Go to the last Line
o - to open a line below

net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1

ESC
:wq!
```

```vim
sudo vi /etc/hosts
delete the ipv6 line (::1)
navigate to the line and type dd
:wq!
```

```vim
sudo vi /etc/sysconfig/network-scripts/ifcfg-enp5s0

BOOTPROTO = dhcp

ONBOOT = yes

:wq!

sudo systemctl restart network
sudo systemctl status network
```

## Tap & SPAN

 - 1st port into pfSense
 - 2nd port into sensor
 - 5th port also into sensor

## Interface Optimization

#### ethtool

  - utility for modifying a network card. We can turn off any kind of offloading or anything that the network card would normally do so that we can capture the traffic in raw format. Allowing the NIC to interact with the traffic before we receive it is not best practice.
#### [Suricata SEPTun Guide - optimization for processing network data](https://www.github.com/pevma/SEPTun/blob/master/SEPTun.pdf)

#### Setup the capture Interface

```

```

## Day 3

### Download repo stuff

[Return To Top](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#nsm-engineer-course)

  #### SSH into sensors

  ```bash

  sudo -s

  cd .ssh
  rm known_hosts

  ssh username@sensorIP

  cd /etc/yum.repos.d/
  ls
  mv CentOS-* /tmp/

  ls (should now be empty)

  curl -L -O http://192.168.2.20/share/local.repo

  yum makecache fast

  yum install tcpdump

  tcpdump -i eno1

  curl -L -O http://192.168.2.20/share/interface.sh

  mv interface.sh /tmp/

  cd /tmp/

  chmod 777 interface.sh

  sudo ./interface.sh eno1

  vi /etc/sysconfig/network-scripts/ifcfg-eno1

  sudo vi ifcfg-eno1

    BOOTPROTO=none

    ONBOOT=yes

    set =no to on all IPV6 settings

    :wq!

systemctl restart network

systemctl status network
  ```

## Installing & Configuring stenographer

[Return To Top](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#nsm-engineer-course)

```bash
sudo -s
yum install stenographer -y

which stenotype

vi /etc/stenographer/config

  "PacketsDirectory": "/data/stenographer/packets"
  "IndexDirectory": "/data/stenographer/index"
  "Interface": "eno1"

  :wq!

stenokeys.sh stenographer stenographer

which stenographer
  should say /bin/stenographer


mkdir -p /data/stenographer/{packets,index}

cd /data
ls stenographer (should see index and packets)

cd ..

chown -R stenographer:stenographer stenographer/

ll #to check that ownership was changed

systemctl start stenographer
systemctl status stenographer

cd stenographer/packets
ls (should see random string of numbers...this takes a bit)


```

## Installing & Configuring Suricata

[Return To Top](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#nsm-engineer-course)


```bash
yum install suricata -y

vi /etc/suricata/suricata.yaml
  :set nu
  :56
  default-log-dir: /data/suricata
  ESC

  :60
  enabled: no
  ESC

  :76
  enabled: no
  ESC

  :404
  enabled: no
  ESC

  :557
  enabled: no
  ESC

  :580
  - interface: eno1
  ESC

  :582
  threads: 4 #Don't forget to uncomment this line

  :wq!
```

```bash
vi /etc/sysconfig/suricata
  OPTIONS="--af-packet=eno1 --user suricata"
  ESC
  :wq!

lscpu -e

suricata-update add-source emerging-threats http://192.168.2.20/share/emerging.rules.tar

suricata-update #Will take a second

cd /data

chown -R suricata:suricata suricata/

systemctl start suricata
systemctl status suricata

```

## Installing & Configuring fsf

[Return To Top](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#nsm-engineer-course)

```bash
yum install fsf -y

vi /opt/fsf/fsf-server/conf/config.py
  SCANNER_CONFIG 'LOG_PATH' : '/data/fsf'
                 'YARA_PATH' : '/var/lib/yara-rules/rules.yara'
                 'PID_PATH' : '/run/fsf/fsf.pid'
                 'EXPORT_PATH' : '/data/fsf/archive'
                 delete 'scan log' from 'ACTIVE_LOGGING_MODULES'
  SERVER_CONFIG 'IP_ADDRESS' : "localhost" #Make sure to put this in quotes and leave the comma afterwards
  ESC
  :wq!

vi /usr/lib/systemd/system/fsf.service #Instead of VI you could also just cat this file- it's short
  verify PIDFile is in /run/fsf/fsf.pid
  ESC
  :wq!

cd /data

mkdir -p /data/fsf/archive

chown -R fsf:fsf fsf/

vi /opt/fsf/fsf-client/conf/config.py
  SEVER_CONFIG 'IP_ADDRESS' : ['localhost']
  ESC
  :wq!

firewall-cmd --add-port=5800/tcp --permanent
firewall-cmd --reload
systemctl start fsf

cd ~

vi random.txt
  asldkfja'sdlfj'alsdfj'aljskdf'lajksfdl
  asdkfjalsdkfj;lakjsdf;lkajsdlfjalsdfkj
  ESC
  :wq!

/opt/fsf/fsf-client/fsf_client.py --full random.txt #Should return something

cd /data/fsf
  rockout.log should be populated from the above command


```

# Day 4

## Kafka

[Return To Top](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#nsm-engineer-course)

 ### How Kafka works
 - Producer - endpoints
    - Data Origin
    - Zeek, Suricata, FSF
 - Broker - kafka
    - Topics
      - We have a topic for each Data Origin (zeek, Suricata, fsf)
      - Replication factor is created per topic
 - Consumer - SIEM
    - Subscribe to Topics
    - Consumer offset is used to keep data in order. The Consumer uses it to track where it is in the data sent from broker and is aused to request data from the Broker
      - A.K.A. Data Reliability

- Kafka workers
  - Multiple workers
  - Each topic will have a worker assigned as a leader
    - This worker can talk internally to the other workers as well as talk with zookeeper
    - Zookeeper:
      - Tracks the metadata about the cluster itself (The Binding that ties all of the wokers together)
      - Zookeeper is being replaced with KRAFT within the next few years



### Installing & Configuring kafka

```bash
sudo yum install kafka librdkafka zookeeper -y
```
#### Zookeeper must be started before Kafka!!!
- Zookeeper performs the election process & forms a quorum
```bash
sudo -s
vi /etc/zookeeper/zoo.cfg
  initLimit=5
  syncLimit=2
  clientPort=2181
  maxClientCnxns=0 #uncomment

  #Add directly below maxClientCnxns
  server.1=localhost:2182:2183
  ESC
  :wq!

cd /var/lib/zookeeper
touch myid
chown zookeeper:zookeeper myid
echo "1" > myid
cat myid #Verify that 1 was written to myid

vi /etc/kafka/server.properties
  :set nu

  :21
  broker.id=1
  :31
  listeners=PLAINTEXT://localhost:9092 #Uncomment this line
  :36
  advertized.listeners=PLAINTEXT://localhost:9092 #Uncomment
  :60
  log.dirs=/data/kafka/logs
  :103
  log.retention.hours=12
  :123
  enable.zookeeper=true #add on line 123 above zookeeper.connect
  zookeeper.connect=localhost:2181

  delete.topic.enable=true #Add to the bottom of the zookeeper section right above group coordinator settings

  ESC
  :wq!
```
```bash
sudo -s
vi /usr/share/kafka/config/producer.properties
  :set nu
  :21
  bootstrap.servers=localhost:9092
  #zookeeper
  zookeeper.connect=localhost:2181 #Add directly under line 21
  ESC
  :wq!

vi /usr/share/kafka/config/consumer.properties
  :set nu
  :19
  bootstrap.servers=localhost:9092
  #Zookeeper
  zookeeper.connect=localhost:2181
  zookeeper.connect.timeout.ms=6000 #Add these lines right after line 19
  ESC
  :wq!

mkdir -p /data/kafka/logs
chown -R kafka:kafka /data/kafka

firewall-cmd --permanent --add-port={9092,2181,2182,2183}/tcp
firewall-cmd --reload #Use this instead of systemctl because restarting the firewall will bring down all of the firewall ports while it restarts

firewall-cmd --list-all #To check that your rules applied

systemctl start zookeeper
watch systemctl status zookeeper #Don't have to do this
systemctl status zookeeper -l

systemctl start kafka
systemctl status kafka -l

cd /usr/share/kafka/bin #.sh script used to troubleshoot/configure kafka after deployment

./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 8 --topic zeek-raw

./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 8 --topic suricata-raw

./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 8 --topic fsf-raw

# Check that the topics were created successfully- don't have to do this

./kafka-topics.sh --describe --zookeeper localhost:2181 --topic zeek-raw

./kafka-topics.sh --describe --zookeeper localhost:2181 --topic suricata-raw

./kafka-topics.sh --describe --zookeeper localhost:2181 --topic fsf-raw

#Script to run that shows you the data that is in your topics
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic zeek-raw --from-beginning

```
## Installing & Configuring zeek

[Return To Top](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#nsm-engineer-course)

### Zeek can NOT hyperthread. It can only be pinned on physical cores.
  - #### 160-200 Mbps per physical CPU core.

```bash
sudo -s

yum install zeek zeek-plugin-kafka zeek-plugin-af_packet -y

cd /etc/zeek

#Don't need to go to networks.cfg - this is just if you want to add stuff
vi networks.cfg
  # Can insert tags in here that will tag certain IP's as certain thing (e.x. servers, googleDNS 8.8.8.8, Cloudflare DNS 1.1.1.1, etc.)
  #Needs CIDR (so for 8.8.8.8 it would be: 8.8.8.8/32)
  # Takes IPv4 & IPv6.

vi zeekctl.cfg
  #Configure the Log Rotation Intervals & Log Archive Period
  :set nu
  :67
  LogDir = /data/zeek/logs

vi node.cfg
  :set nu
  comment out lines 8-11
  Delete everything worker2 down
  Uncomment the cluster (should have a logger, manager, proxy, and worker)

  #Add this line under the manager section before the proxy section
  pin_cpus=0

  #Worker-1
  interface=af_packet::eno1
  lb_method=custom
  lb_procs=2
  pin_cpus=1,2
  env_vars=fanout_id=93
  ESC
  :wq!
```

```bash
sudo -s
cd /usr/share/zeek/site

mkdir scripts
cd scripts/

#Download zeek scripts
curl -L -O http://192.168.2.20/share/zeek_scripts.tar.gz

#unzip downloaded file
tar -zxvf zeek_scripts.tar.gz

#Remove zipped file (no longer needed)
rm -rf zeek_scripts.tar.gz

cd zeek_scripts

vi kafka.zeek
  ["metadata.broker.list"] = ("localhost:9092");

  ESC
  :wq!

cd /usr/share/zeek/site

vi local.zeek
  #zeek scripts to enable/disable

  #Add to the end of the file
  @load scripts/zeek_scripts/json
  @load scripts/zeek_scripts/afpacket
  @load scripts/zeek_scripts/kafka
  @load scripts/zeek_scripts/extension
  @load scripts/zeek_scripts/extract-files
  @load scripts/zeek_scripts/fsf

  :wq!

#Don't have to go to /usr/bin but it is here for troubleshooting
#Zeek binary lives in /usr/bin
cd /usr/bin

cd /data/zeek
mkdir logs

zeekctl deploy

```
```bash
cd /usr/share/kafka/bin

./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic zeek-raw --from-beginning
```

# NSM Engineer Week 2

## Shipping Data

### Filebeat

[Return To Top](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#nsm-engineer-course)
  - Ships data to Kafka Topics (From Suricata (eve.json) & FSF (rockout.log))
  - Logstash then pulls from Kafka.
    - This is better than pushing data to logstash so you don't accidentally overwhelm the VMHost

```bash
sudo -s

yum install filebeat -y

cd /etc/filebeat
# filebeat.yml - Inputs and Outputs
mv /etc/filebeat/filebeat.yml /etc/filebeat/filebeat.yml.bak # Makes this a backup file

#Download the edited yaml file
curl -L -O http://192.168.2.20/share/filebeat.yml

systemctl start filebeat
systemctl enable filebeat
systemctl status filebeat

/usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic fsf-raw --from-beginning

/usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic suricata-raw --from-beginning

```

### Logstash

[Return To Top](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#nsm-engineer-course)

  - Input - Where data comes from
  - Filter - Data Normalization & Data Enrichment
  - Output - Send data to elasticsearch

  - Default port logstash has to have open to ingest beats: 5044

  #### Data Normalization & Enrichment
    - Data Normalization:
      - ECS (Elastic Common Schema)
        - Transform/ Mutate (translate) incoming fields into a uniform field within elasticsearch
        - "Data Schema"
    - Data Enrichment
      - e.x. - Correlating an IP address with a geolocation from a database and saving it to a new field
    - Best practice is to create multiple filter files so it is easier to edit/ troubleshoot

```bash
sudo -s

cd /

yum install logstash -y #Takes a second

cd /etc/logstash
curl -L -O http://192.168.2.20/share/logstash.tar
tar -xvf logstash.tar

cd conf.d
ll # Should see a LOT of files

chown logstash:logstash /etc/logstash/conf.d/*
chmod -R 744 /etc/logstash/conf.d/ruby/
rm -f /etc/logstash/logstash.tar

systemctl start logstash

#Base configuration file for logstash
/etc/logstash/logstash.yml

#File to edit the JVM OPTIONS
/etc/logstash/jvm.options
  - Minimum memory of 4GB, Max of 8GB




```

## Elasticsearch

[Return To Top](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#nsm-engineer-course)

#### Node Types
- Nodes refer to the instances (so it can be different physical boxes or multiple instances on one server)
- Index (ECS-Zeek-Network-DATE)
  - collection of data/documents (logs)
    - Shard/ Replica
      - Smaller than an Index
      - Single worker (lucene query/index)
      - Searchable data
      - Replica's
        - Stored backups of data
        - If one node goes down you don't lose data
        - Think of it like a RAID array across nodes
        - Replication Factor: The term/setting to set how many copies of data are kept
- Cluster - Multiple Nodes
  - Each node can have properties attributed to them
- Node
  - Master Node
    - Manages the Cluster
    - Should have no more than 3 Master Nodes
    - Do it in odd numbers (kibana doesn't like even numbers for the election process)
    - The Master nodes that aren't currently master (so they are master eligible) are normally used as extra data nodes.
    - Not very resource intensive
    - Master node communicates over port 9300
  - Data
    - "Workhorses" - Index Data
  - Ingest
    - Index Pipelines (Alternative to Logstash)
    - Even if not used you still need one (Can just assign it to a data node)
  - Machine Learning
    - If you don't have a dedicated ML node it gets thrown on a data node
    - Very resource intensive
  - Coordinating
    - Node that is specifically not given any attributes
    - Only job is to search
    - Can increase performance of the searches
  - Transform

  ##### Nodes go through an election process to determine the Master Node

  - Port 9200 is listening port/ API communications
  - Port 9300 is internal node communications

```bash
sudo -s

yum install elasticsearch  -y

cd /data
ll #Check for elasticsearch
chown -R elasticsearch:elasticsearch elasticsearch/

vi /etc/elasticsearch/elasticsearch.yml
  :set nu
  :17
  cluster.name: sg-1 #Uncomment this line
  :23
  node.name: sg-1 #Uncomment this line
  :33
  path.data: /data/elasticsearch
  :43 #Uncomment
  :55
  network.host: _local:ipv4_ #Uncomment this line
  ESC
  :wq!

#This is where you set the heap size (should be half of the available memory or 30GB, whichever comes first)
vi /etc/elasticsearch/jvm.options
  :22
  -Xms4g
  :23
  -Xmx4g

mkdir /usr/lib/systemd/system/elasticsearch.service.d
chmod 755 /usr/lib/systemd/system/elasticsearch.service.d/

vi /usr/lib/systemd/system/elasticsearch.service.d/override.conf
  [Service]
  LimitMEMLOCK=infinity
  ESC
  :wq!

systemctl daemon-reload

firewall-cmd --add-port={9200,9300}/tcp --permanent
firewall-cmd --reload
systemctl start elasticsearch
systemctl status elasticsearch

#API calls to test that elasticsearch is running
curl localhost:9200/_cat/nodes
curl localhost:9200/_cat/indices

```

## Kibana

[Return To Top](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#nsm-engineer-course)

```bash
sudo -s

yum install kibana -y

vi /etc/kibana/kibana.yml
  :set nu
  :7
  # Uncomment this line 
  server.host: "172.16.10.101" #The IP of your sensor
  :28 #Uncomment this line
  ESC
  :wq!

firewall-cmd --add-port=5601/tcp --permanent
firewall-cmd --reload


systemctl start kibana
systemctl status kibana
```
### Navigate to Kibana Sensor IP:5601

  - Go to management -> Dev tools #This is to query elasticsearch's API in kibana
    - GET _cat/nodes
    - GET _cat/indices
  - Go to management -> Stack Management
    - Click on Index patterns
      - ecs-*
        - @timestamp
        - Click "Create Index Pattern"
      - ecs-zeek-*
        - @timestamp
        - Click "Create Index Pattern"
      - ecs-suricata-*
        - @timestamp
        - Click "Create Index Pattern"
      - fsf-*
        - @timestamp
        - Click "Create Index Pattern"

  - Go to Discover


### Elasticsearch Mappings

```bash
sudo -s

cd ~
curl -L -O http://192.168.2.20/share/ecskibana.tar
tar -xvf ecskibana.tar

cd ecskibana
./import-index-templates.sh

```
## Troubleshooting

[Return To Top](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#nsm-engineer-course)
- Make sure the group ID is configured correctly in logstash in the input pipeline files located in /etc/logstash/conf.d
  - Date plugin is used in logstash filter to edit @timestamp to be the same as ts and then delete the ts field
  - Mutate plugin is used to rename/create fields
  - Output of a Logstash pipeline is the destination (in our case it's elasticsearch)

Kibana isn't returning any data
  1. Check API's in Kibana
  2. Check status of elasticsearch
  3. df -h, du -h
  4. Check kafka status
  5. Run the long command to see if kafka is getting data
  6. Check to see if zookeeper is running
  7. Check filebeat
  8. Check FSF

  ### Start at the issue and work your way back

  tcpdump -i capture_interface #check that we are collecting traffic

```bash

Logstash bootloop is caused by an incorrect configuration
  - journalctl -xe

zeekctl status (zeeks alternative to systemctl)
cd /data/zeek/logs/current #Should see conn.log, http.log, files.log

df -h #Used to see partitions/mounts

cd /data
  du -h #Check partition size

cd /var/log #See log info about services
  If logstash cant connect to elasticsearch then the outputs is configured incorrectly

Dont forget about chown files and chmod files

ss -lnt #Gives a list of sockets open

Port 9600 #Logstash

Port 1234 #Stenographer

cd /data
ll #To check ownership & permissions of files & directories
zeek should be owned by root... the only one we dont change

firewall--cmd --list-ports #To see open ports in the firewall
firewall--cmd --reload

```

## Resource Allocation for 1Gb/s Capture

[Return To Top](https://github.com/jonezy35/NSM-Engineer/blob/main/NSMENotes.md#nsm-engineer-course)

###### Unless otherwise specified core = thread

#### Sensor Services - Total CPU: 24 <ins>*physical*</ins> cores - Total RAM: 64GB RAM
 - Steno
   - 2 Cores
   - 4-6GB RAM
 - Zeek
   - 160Mbps per **physical** core
   - 5 **physical** cores per Gb/s of data
   - 16GB of RAM
 - Suricata
   - 18 Cores
   - 16 GB RAM
   - Depends on how many rules to match against which makes it the hardest to figure out
 - FSF
   - 2 Cores
   - 2 GB RAM
 - Kafka
   - 6 Cores
   - 12 GB RAM
 - OS
   - 1 CPU Core
    - 1 GB RAM
 - ESXI
     - 2 core
     - 4 GB RAM
 - Filebeat
    - 1 Core
    - 1 GB RAM
 - Zookeeper
    - 2 Cores
    - 2 GB RAM

#### VM Host - Total CPU: 8 <ins>physical</ins> Cores - Total RAM: 58 GB RAM
  - Logstash
    - 6 Cores
     - 8 GB RAM
  - Elasticsearch (These are resources **per node**)
    - 2-8 Cores
    - 31-64 GB RAM
  - Kibana
     - 2-4 Cores
     - 4 GB RAM

#### For Single Node Deployment - Total CPU: 34 <ins>physical</ins> cores - Total RAM: 128 GB RAM
