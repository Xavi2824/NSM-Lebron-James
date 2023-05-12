# Elastic Information
**Docket** - execute queries and merge into one single pcap
---
**Suricata** - Port independent Protocol analysis
---
**Filebeat** - lightweight data transfer... can be used to transport Suricata and Zeek with little processing power.
---
**Kafka** - "Topics" for Zeek, Suricata, FSF  
    Zookeeper keeps track of all nodes that join/leave Kafka.
# Elastic Config
starting the kit command  
`lxc list`  
`lxc start --all`  
`sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0`  
`sudo vi /etc/sysctl.conf`  
`:%d`  
**Change network-scripts etho0 to this**
```  
DEVICE=eth0
BOOTPROTO=none
ONBOOT=yes
HOSTNAME=elastic
NM_CONTROLLED=no
TYPE=Ethernet
IPADDR=10.81.139.20
GATEWAY=10.81.139.1
PREFIX=24
```
**Change etc hosts file to this**  
`sudo vi /etc/hosts`
```  
127.0.0.1 localhost
10.81.139.10 repo
10.81.139.20 sensor
10.81.139.30 elastic0
10.81.139.31 elastic1
10.81.139.32 elastic2
10.81.139.40 pipeline0
10.81.139.41 pipeline1
10.81.139.42 pipeline2
10.81.139.50 kibana
```  
**Shelling into node cmd line** example  
`ssh elastic@10.81.139.223`  

**Generating SSH keys/ SSH config items**  
so now all you have to do to SSH into a node is "ssh kibana", you do this on the host pc.  
`sudo vi ~/.ssh/config`   
```
Host repo
  HostName repo
  User elastic
Host sensor
  HostName sensor
  User elastic
Host elastic0
  HostName elastic0
  User elastic
Host elastic1
  HostName elastic1
  User elastic
Host elastic2
  HostName elastic2
  User elastic
Host pipeline0
  HostName pipeline0
  User elastic
Host pipeline1
  HostName pipeline1
  User elastic
Host pipeline2
  HostName pipeline2
  User elastic
Host kibana
  HostName kibana
  User elastic 
  ``` 
`ssh-keygen`   from the ubuntu home device (hit enter, enter, etc)

**NEED FOR LOOP**  

for host in sensor repo elastic{0..2} pipeline{0..2} kibana; do ssh-copy-id $host; done


**Enable creation of File sharing (ONLY IN REPO)**  
`sudo yum install nginx`  


**Unzips the all class file and places it /usr/share/nginx**

`sudo unzip ~/all-class-files.zip -d /usr/share/nginx`  

**renames the all class files into "fileshare"**  

`sudo mv /usr/share/nginx/all-class-files/ /usr/share/nginx/fileshare`  

**SOMETHING ABOUT EMERGING THREATS BEING ADDED AS WELL**  

`sudo mv ~/emerging.rules.tar.gz /usr/share/nginx/fileshare`



**Creating a fileshare.conf for nginx**  
`sudo vi /etc/nginx/conf.d/fileshare.conf`  

```
server {
  listen 8000;
  location / {
    root /usr/share/nginx/fileshare;
    autoindex on;
    index index.html index.htm;
  }
}  
```
**Add listening port 8000 from the fileshare conf.d config to the firewall and restart it, then verify with the "list-all" cmd.**  

`sudo firewall-cmd --add-port=8000/tcp --permanent`  
`sudo firewall-cmd --reload`  
`sudo firewall-cmd --list-all`  


**Start and enable a service, in this case, starting and enabling nginx service**  

`sudo systemctl enable --now nginx`  


**List socket status**  
`ss -lnt`  

**navigate to repo url with port 8000**  
10.81.139:8000  



---
---





# **MAY 2, 2023 NOTES (DAY 2)** 
  

Configuring FileShare and Local packages

Installing Yum utilities cmd

`sudo yum install yum-utils`  

**creating another local repo, syncing with the centos**  

`sudo reposync -l --repoid=extras --download_path=/repo/local-extras`  

**then do a ll tack repo to check for confirmation**  

`ll /repo`  

**Generating the repo updates file into database type file for local-extras file STEPS**  

`sudo yum install createrepo`  
`sudo createrepo local-extras`  
`cd /repo`  
`sudo vi /etc/nginx/conf.d/packages.conf`
---  
**Creating a nginx config to listen on port 8008, VIA root /repo directory**  

```
server {
  listen 8008;
  location / {
    root /repo;
    autoindex on;
    index index.html index.htm;
  }
} 
```   

**Allowing new port through the firewall**  
`sudo firewall-cmd --add-port=8008/tcp --permanent`  
`sudo firewall-cmd --reload`  
`sudo firewall-cmd --list-all`  

**have to start/restart nginx service systemctl command**  

`sudo systemctl restart nginx`  
`sudo systemctl status nginx`  

**Navigate to repo ip address via port 8008 through web browser**  


**SSH into the Sensor and install zeek**  

`sudo yum install zeek`  
# START HERE FOR STEP 2 AFTER SETTING UP STATIC IP FOR CONTAINERS (below!) 
       
**make a directory for archive in sensor**  

`mkdir ~/archive`  
`ll /etc/yum.repos.d`  
`sudo mv /etc/yum.repos.d/* ~/archive/`  

**pulling local.repo from the repo:8000 web browser into the sensor directory**  

`sudo curl -LO http://repo:8000/local.repo`  

**You then alter that local.repo file to adjust the baseurl to "repo:8008", while also removing carrot symbols**  

`sudo vi /etc/yum.repos.d/local.repo`  

```[local-base]
name=local-base
baseurl=http://repo:8008/local-base/
enabled=1
gpgcheck=0

[local-rocknsm-2.5]
name=local-rocknsm-2.5
baseurl=http://repo:8008/local-rocknsm-2.5/
enabled=1
gpgcheck=0

[local-elasticsearch-7.x]
name=local-elasticsearch-7.x
baseurl=http://repo:8008/local-elastic-7.x/
enabled=1
gpgcheck=0

[local-epel]
name=local-epel
baseurl=http://repo:8008/local-epel/
enabled=1
gpgcheck=0

[local-extras]
name=local-extras
baseurl=http://repo:8008/local-extras/
enabled=1
gpgcheck=0

[local-updates]
name=local-updates
baseurl=http://repo:8008/local-updates/
enabled=1
gpgcheck=0
```  

**Generate a YUM cache**  

`sudo yum makecache fast` 

--- 

#  **SECURING REPO AND FILESHARE**  

---

1. `mkdir ~/certs`  
2. cd into new directory  
3. `openssl genrsa -des3 -out localCA.key 2048`  
  passphrase: training
4. `openssl req -x509 -new -nodes -key localCA.key -sha256 -days 1095 -out localCA.crt`  
  Items: Mountain View, Elastic, Security, Instructor, US, CA, no email address  

5. `openssl genrsa -out repo.key 2048`
6. `openssl req -new -key repo.key -out repo.csr`  
    re enter same items from step 4.   

7.  `sudo vi ~/certs/repo.ext`  
```  authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = repo
IP.1 = 10.81.139.10
```  
8. **Generate the repo certificate**  
    `openssl x509 -req -in repo.csr -CA localCA.crt -CAkey localCA.key -CAcreateserial -out repo.crt -days 365 -sha256 -extfile repo.ext`  

9. `sudo vi ~/certs/repo.ext`  
   `sudo mv repo.crt /etc/nginx`  
   `sudo mv repo.key /etc/nginx`  
10. cd into conf.d  
   `cd /etc/nginx/conf.d`  
   navigate to nginx through browser: "repo:8000/nginx/"  
   `sudo curl -LO http://repo:8000/nginx/proxy.conf`  
11. Disable port http port 80 to prevent confusion.  
   `sudo vi /etc/nginx/nginx.conf`  
   comment out line number 39,40, and 41. "#".  
12. `sudo vi /etc/nginx/conf.d/packages.conf ` and `sudo vi /etc/nginx/conf.d/fileshare.conf`
  
   Change the listening address to loopback address, 127.0.0.1:8008 for both these files above.  


   ```server {

  listen 127.0.0.1:8008;

  location / {

    root /repo;

    autoindex on;

    index index.html index.htm;

  }

   }```  


13. Add ports 80 and 443 to the firewall.  
    `sudo firewall-cmd --add-port={80,443}/tcp --permanent`  
    `sudo firewall-cmd --relaod`  
    `sudo firewall-cmd --list-all`

14. Restart the nginx service.  
    `sudo systemctl restart nginx`  
    `sudo systemctl status nginx`  
--- 
---
---
## **Shell into the sensor for this change: MODIFYING THE /etc/yum.repos local repo file to change the baseurl to "https://repo/packages/local blah blah blah"**  

`sudo vi /etc/yum.repos.d/local.repo`  
``` [local-base]
name=local-base
baseurl=https://repo/packages/local-base/
enabled=1
gpgcheck=0

[local-rocknsm-2.5]
name=local-rocknsm-2.5
baseurl=https://repo/packages/local-rocknsm-2.5/
enabled=1
gpgcheck=0

[local-elasticsearch-7.x]
name=local-elasticsearch-7.x
baseurl=https://repo/packages/local-elastic-7.x/
enabled=1
gpgcheck=0

[local-epel]
name=local-epel
baseurl=https://repo/packages/local-epel/
enabled=1
gpgcheck=0

[local-extras]
name=local-extras
baseurl=https://repo/packages/local-extras/
enabled=1
gpgcheck=0

[local-updates]
name=local-updates
baseurl=https://repo/packages/local-updates/
enabled=1
gpgcheck=0
```  

**for loop to automate every node to make same adjustment as previous step**  

`for host in elastic{0..2} pipeline{0..2} kibana; do scp ~/archive/* $host:/home/elastic ; done`

**Next Step is to check every node and do these commands next; every single node except REPO...will take awhile**  
---  
Purpose is to get the nodes to point towards our repo container (in archive file) for resources instead of to CentOS. 

`mkdir ~/archive`  
`sudo mv /etc/yum.repos.d/* ~/archive`  
`sudo vi /etc/yum.repos.d/local.repo`  

```[local-base]
name=local-base
baseurl=https://repo/packages/local-base/
enabled=1
gpgcheck=0

[local-rocknsm-2.5]
name=local-rocknsm-2.5
baseurl=https://repo/packages/local-rocknsm-2.5/
enabled=1
gpgcheck=0

[local-elasticsearch-7.x]
name=local-elasticsearch-7.x
baseurl=https://repo/packages/local-elastic-7.x/
enabled=1
gpgcheck=0

[local-epel]
name=local-epel
baseurl=https://repo/packages/local-epel/
enabled=1
gpgcheck=0

[local-extras]
name=local-extras
baseurl=https://repo/packages/local-extras/
enabled=1
gpgcheck=0

[local-updates]
name=local-updates
baseurl=https://repo/packages/local-updates/
enabled=1
gpgcheck=0 
```   

---  --- ------------------------------------------------



**Certificate stuff on the repo box**  

`for host in sensor elastic{0..2} pipeline{0..2} kibana; do scp ~/certs/localCA.crt elastic@$host:/home/elastic/localCA.crt; done`  

**for each node, perform these 3 steps/cmds**  

`sudo mv ~/localCA.crt /etc/pki/ca-trust/source/anchors/localCA.crt`  
`sudo update-ca-trust`  
`sudo yum makecache fast`

**from the Ubuntu host machine, copy the localCA.crt file thing from the repo device into the local machine**  

`sudo scp elastic@repo:/home/elastic/certs/localCA.crt ~/localCA.crt`  

**steps for navigating to the browser**  
 get them from Bland  

 ---

 # DAY 3, 3MAY, 2023 NOTES  

 **Configuring the Capture Interface**  

 1. SSH into sensor  
 2. `sudo yum install ethtool`  
   ethtool is a tool for managing NICs'.  
 3. `ip a`  
   eth0 interface is used for management, eth1 interface is used for capture.   
 4. `sudo ethtool -k eth1`  
   the "-k" is the flag for show features.  
 5. `sudo curl -LO https://repo/fileshare/interface.sh`  

 6. `cat interface.sh`  
 7. `sudo chmod +x interface.sh`  
 8. `sudo ./interface.sh eth1`  
   verify changes by running same command from step 4.  
 9. `sudo vi /sbin/ifup-local`  
   similair to previous script ran, except this code enters sensor into promisic mode.  
   ``` #!/bin/bash
if [[ "$1" == "eth1" ]]
then
for i in rx tx sg tso ufo gso gro lro rxvlan txvlan
do
/usr/sbin/ethtool -K $1 $i off
done
/usr/sbin/ethtool -N $1 rx-flow-hash udp4 sdfn
/usr/sbin/ethtool -N $1 rx-flow-hash udp6 sdfn
/usr/sbin/ethtool -n $1 rx-flow-hash udp6
/usr/sbin/ethtool -n $1 rx-flow-hash udp4
/usr/sbin/ethtool -C $1 rx-usecs 10
/usr/sbin/ethtool -C $1 adaptive-rx off
/usr/sbin/ethtool -G $1 rx 4096

/usr/sbin/ip link set dev $1 promisc on

fi  ```  

10. Change the permissions of ifup-local.  
   `sudo chmod +x /sbin/ifup-local`  
11. `sudo vi /etc/sysconfig/network-scripts/ifup`  
  At the very bottom, add this line of code under "exec" line,   
  ``` if [ -x /sbin/ifup-local ]; then
/sbin/ifup-local ${DEVICE}
fi```  

12. `sudo vi /etc/sysconfig/network-scripts/ifcfg-eth1`  
```
DEVICE=eth1  
BOOTPROTO=none  
ONBOOT=yes  
NM_CONTROLLED=no  
TYPE=Ethernet  
```  
13. Do a systemctl restart of the network  
    `sudo systemctl restart network`  

14. `sudo ethtool -k eth1`  


15. Now we run traffic through the eth1 interface  
    `sudo tcpdump -nn -i eth1`  
    `sudo tcpdump -nn -i eth1 '!port 22'`  


    ---  


    **Stenographer Steps**  

1.  `sudo yum install stenographer`  
2. `sudo yum install which` 
3. `cd /etc/stenographer/`  
4. `sudo vi /etc/stenographer/config` 
**Make these changes listed below, doing this will tell where steno will place its data; then you will actually make that directory in step 5.**   
```{
  "Threads": [
    { "PacketsDirectory": "/data/stenographer/packets"
    , "IndexDirectory": "/data/stenographer/index"
    , "MaxDirectoryFiles": 30000
    , "DiskFreePercentage": 30
    }
  ]
  , "StenotypePath": "/usr/bin/stenotype"
  , "Interface": "eth1"
  , "Port": 1234
  , "Host": "127.0.0.1"
  , "Flags": []
  , "CertPath": "/etc/stenographer/certs"
}
```  
5. `sudo mkdir -p /data/stenographer/{packets,index}` 

  

6. Changing user and group ownership of /data/stenographer to "stenographer:stenographer".  
   `sudo chown -R stenographer:stenographer /data/stenographer`  
7. Generating keys for access to stenographer.  
   `sudo stenokeys.sh stenographer stenographer` 
   **can check with cmd below**   
    `ll /etc/stenographer/certs`   
8. Start and enable stenographer  
   `sudo systemctl enable stenographer --now`  
9. Monitor traffic through the terminal within the /data/stenographer/packets directory.  
   `watch ls -al /data/stenographer/packets/`  



   **SURICATA STEPS**  

  1. `sudo yum install suricata` starting from the sensor device  
  2. `sudo -s`  
  3. `ll /etc/suricata`  
  4. `vi /etc/suricata/suricata.yaml`  
  Change line **56**: /data/suricata/  
  Change line **60**: no (change from yes to no)  
  change line **76**: no (change from yes to no)  
  Change line **404**: no (change from yes to no)  
  Change line **557**: no (change from yes to no)  
  Change line **580**: eth1 (change from eth0 to eth1)  
  Change line **582**: threads: 3 (remove pound/comment sign and change from auto to 3)  
  Change line **981,982,983**: Uncomment all 3 line items, change line **982** and **983** from "suri" to "suricata".  
  Change line **1500**: no (change from yes to no)    
  Change line **1516**: no (change from yes to no)
  Change line **1521**: no (change from yes to no)
  Change line **1527**: no (change from yes to no)
  Change line **1536**: no (change from yes to no)  

  5. `vi /etc/sysconfig/suricata`  
  Change the last line of this config file to this:  
  ```OPTIONS="--af-packet=eth1 --user suricata --group suricata "```  
  6. Go back into `vi /etc/suricata/suricata.yaml`  
  Change line **1434**: yes (change from no to yes)  
  Change line **1452**: cpu [0]  
  Change line **1458**: medium [1]  
  Change line **1459**: high [2]  
  Change line **1460**: default set to "high".  
  Change line **1461**: default set to "high"


  7. `sudo suricata-update add-source emergingthreats https://repo/fileshare/emerging.rules.tar.gz` 

  8. `sudo suricata-update` update takes a minute or two. 
  9. `sudo mkdir -p /data/suricata`  
  
  10. `sudo chown -R suricata:suricata /data/suricata`  

  11. Way to check to make sure suricata is functioning properly:  
  `ll /data/suricata`   

  # DAY 4 MAY THE 4TH BE WITH YOU MOFO  

  **STEPS FOR INSTALLING ZEEK (on sensor)**  

  1. `sudo yum install zeek`
    
  2. `sudo yum install zeek-plugin-af_packet`   
  allows af_packet on the nic to upload to zeek. 

  3. `sudo yum install zeek-plugin-kafka`  

  4. `ll /etc/zeek`  
  5. `cat /etc/zeek/networks.cfg`  
  6. **`sudo vi /etc/zeek/zeekctl.cfg`**  
  Line item **67**: `LogDir = /data/zeek` 
  Add this to Line item **68**: `lb_custom.InterfacePrefix=af_packet::`  
  This line item entry tells Zeek to use af_packet  
  7. **`sudo vi /etc/zeek/node.cfg`**  
Setting up as a cluster config, right now setup as a Standalone  

Line item **8-11**: comment them out (add # to each line)  


Line item **16-31**: UNcomment each line (leave worker 2 section commented)  

Add this to Line item **23**: pin_cpus=1  

Line item **32/31**: eth1 (change from eth0 to eth1)  

Add this to Line item **33**: `lb_method=custom`  
Add this to line item **34**: `lb_procs=2`   
Add this to line item **35**: `pin_cpus=2,3`  
Add this to line item **36**: `env_vars=fanout_id=77`  

**Bonus command to look at your cpu (not required)**  
`lscpu -e`  

8. `sudo mkdir /usr/share/zeek/site/scripts`  
making a scripts directory...cuz i felt like it.  

9. `cd /usr/share/zeek/site/scripts`  

10. `sudo curl -LO https://repo/fileshare/zeek/*`  
**YOU WILL INDIVIDUALLY CURL EACH FILE FROM THE ZEEK FILE LOCATION WITHIN THE REPOSITORY, except for local.zeek  (navigate to repo/fileshare within the browser, should have .)**  

11. `cd ..`  

12. `sudo vi /usr/share/zeek/site/local.zeek`  
Go to bottom using "Shift g" 

Add this to line item **104** and **105**: `@load ./scripts/afpacket.zeek` and `@load ./scripts/extension.zeek`  

Add this to line item **107**: `redef ignore_checksums = T;`



13. `sudo mkdir -p /data/zeek`  
14. `sudo chown -R zeek: /data/zeek`  
    `sudo chown -R zeek: /etc/zeek`   
    `sudo chown -R zeek: /usr/share/zeek`  
    `sudo chown -R zeek: /usr/bin/zeek`  
    `sudo chown -R zeek: /usr/bin/capstats`   
    `sudo chown -R zeek: /var/spool/zeek`  
15. `sudo /sbin/setcap cap_net_raw,cap_net_admin=eip /usr/bin/zeek`  
16. `sudo /sbin/setcap cap_net_raw,cap_net_admin=eip /usr/bin/capstats`  
17. Verify previous commands in steps 15 and 16 with:  
`sudo getcap /usr/bin/zeek`  
`sudo getcap /usr/bin/capstats`  
 **IMPORTANT STEP**  
18. `sudo -u zeek zeekctl deploy`  
 Check status command (the -u is to make sure you are running this as a user):  
 `sudo -u zeek zeekctl status`  

 Checking for functionality in zeek:  
 `ll /data/zeek/current`  
 `cat /data/zeek/current/conn.log`
---
---

**INSTALL FSF STEPS (sensor)**  

1. `sudo yum install fsf`  
2. `sudo vi /opt/fsf/fsf-server/conf/config.py`  
**REFER TO WORD DOCUMENT FOR CHANGES SINCE VI WANTS TO BE STUPID**  
3. `sudo mkdir -p /data/fsf/archive`  

4. `sudo chown -R fsf: /data/fsf`  **may have to repeat this step if you ever restart the fsf service**

5. `sudo vi /opt/fsf/fsf-client/conf/config.py`  
Change line item **9**: Change from 127.0.0.1 to 'localhost' (keep the comma after it).  
6. `sudo vi /usr/lib/systemd/system/fsf.service`  
Confirm everything is ok in here (it is, who cares.)  
7. `sudo systemctl enable fsf --now`  
8. `journalctl -xeu <service>`  
this command helps show the user what error is occuring with service. 
9. Running a client Python script (use this to verify that fsf is working; the idea is to to scan any file) 
`/opt/fsf/fsf-client/fsf_client.py --full interface.sh`  
10. `cd /usr/share/zeek/site/`

    `sudo vi /usr/share/zeek/site/local.zeek`   
    go to the bottom (shift g) 

    Add this to line item **106,107,108**: CHECK WORD DOCUMENT AGAIN, CUZ BULLSHIT.  
11. `sudo -u zeek zeekctl stop`  
    `sudo -u zeek zeekctl deploy`  
    `sudo -u zeek zeekctl status`  
12. `cd /data/fsf`  
    `cd /data/zeek`  
    make sure that all the .log, .lock 's are there. ("dbg.lock, daemon.log, dbg.log, conn.log, **ROCKOUT.LOG**" etc...) If they arent there you suck.  


  ---  
  --- 
  --- 
  # **DAY 5 MAY 5, 2023 NOTES**  

  **Steps for installing/configuring KAFKA**  

  1. **starting within pipeline0**  
  `sudo yum install kafka zookeeper` 
  2. `sudo mkdir -p /data/zookeeper`  
     `sudo chown -R zookeeper: /data/zookeeper/` 

  3. `sudo vi /etc/zookeeper/zoo.cfg`  
  
 # where zookeeper will store its data
 dataDir=/data/zookeeper

 # what port should clients like kafka connect on
 clientPort=2181

 # how many clients should be allowed to connect, 0 = unlimited
 maxClientCnxns=0

 # list of zookeeper nodes to make up the cluster
 # First port is how followers and leaders communicate
 # Second port is used during the election process to determine a leader
 server.1=pipeline0:2888:3888
 server.2=pipeline1:2888:3888
 server.3=pipeline2:2888:3888

 # more than one zookeeper node will have a unique server id.
 # Ex.) server.1, server.2, etc..

 # milliseconds in which zookeeper should consider a single tick
 tickTime=2000

 # amount of ticks a follow has to connect and sync with the leader
 initLimit=5

 # amount of ticks a follower has to sync with a leader before being dropped
 syncLimit=2```



4. `sudo touch /data/zookeeper/myid` 

5. `sudo chown zookeeper: /data/zookeeper/myid`  

6. `echo '3' | sudo tee /data/zookeeper/myid`  

**CHANGE EACH PIPELINE FROM 1,2,3 OR for loop script wont work below.**  

7. `cat /data/zookeeper/myid`  

8. `sudo firewall-cmd --add-port={2181,2888,3888}/tcp --permanent`  

9. `sudo firewall-cmd --reload`

    Can check with `sudo firewall-cmd --list-ports`  


**Dont start zookeeper yet! Not until you have repeated these steps up until this point on the other pipeline nodes**  

10. After all pipeline nodes are configured, start zookeeper service on each one:  
`sudo systemctl enable --now zookeeper`  

`sudo systemctl status zookeeper`  

11. From the ubuntu box run this for loop to help verify that I am clustered:  

`for host in pipeline{0..2}; do (echo "stats" | nc $host 2181 -q 2); done`  

Should show something along the lines of "follower, leader, follwer, etc..."  if not then you just suck.  

**STEPS FOR INSTALLING KAFKA**  

1. `sudo mkdir -p /data/kafka`
   
   `sudo chown kafka: /data/kafka`  

2. `sudo cp /etc/kafka/server{.properties,.properties.bk}`  

can check by: `cd /etc/kafka`  and looking within 

3. `sudo vi /etc/kafka/server.properties`  

**CHECK WORD DOC FOR NOTES CUZ HOLY SHIT THIS LOOKS BAD**  

Be sure to change the "broker.id" as well as the "pipeline0" in 2 other lines for **EACH NODE**

4. `sudo firewall-cmd --add-port=9092/tcp --permanent`  
   
   `sudo firewall-cmd --reload`  

   **REPEAT THESE STEPS FOR EACH PIPELINE BEFORE CONTINUING**  

5. `sudo systemctl enable --now kafka`  
   `sudo systemctl status kafka`  

6. From pipeline0 only; run these set of script commands:  
   
   `sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server pipeline0:9092 --create --topic test --partitions 3 --replication-factor 3`


   `sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server pipeline0:9092 --list`  


   `sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server pipeline0:9092 --describe --topic test` 

   `sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server pipeline0:9092 --delete --topic test`   


7. GO TO YOUR SENSOR FOR THIS:  
`cd /usr/share/zeek/site/scripts/`  

ll into this and make sure "kafka.zeek" is there.  

8. go back to PIPELINE0:  
 
 `sudo /usr/share/kafka/bin/kafka-topics.sh --create --zookeeper pipeline0:2181 --replication-factor 3 --partitions 3 --topic zeek-raw` 

 THEN DO THIS ONE:  
 `sudo /usr/share/kafka/bin/kafka-topics.sh --describe --zookeeper pipeline0:2181 --topic zeek-raw`     


 9. GO BACK TO SENSOR AND DO THIS:  

 `sudo vi /usr/share/zeek/site/local.zeek`  

 Add this line to the very bottom underneath the previous scripts "@load blah blah blah"  

 `@load ./scripts/kafka.zeek` 
 


 
 
 10. Redploy zeek on the sensor:  

`sudo -u zeek zeekctl deploy`  
`sudo -u zeek zeekctl status`  

11. Validate Streaming from pipeline0:  

`sudo /usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server pipeline0:9092 --topic zeek-raw`  

**May have to do this step first (from the sensor)**:  `sudo vi kafka.zeek` 
``` 
@load Apache/Kafka/logs-to-kafka

redef Kafka::topic_name = "zeek-raw";
redef Kafka::json_timestamps = JSON::TS_ISO8601;
redef Kafka::tag_json = F;
redef Kafka::kafka_conf = table(
    ["metadata.broker.list"] = "pipeline0:9092,pipeline1:9092,pipeline2:9092");

event zeek_init() &priority=-5
{
    for (stream_id in Log::active_streams)
    {
        if (|Kafka::logs_to_send| == 0 || stream_id in Kafka::logs_to_send)
        {
            local filter: Log::Filter = [
                $name = fmt("kafka-%s", stream_id),
                $writer = Log::WRITER_KAFKAWRITER,
                $config = table(["stream_id"] = fmt("%s", stream_id))
            ];

            Log::add_filter(stream_id, filter);
        }
    }
}
```  








12. From the sensor:  
`sudo -u zeek zeekctl deploy`  
Then go back into pipeline0 and run the traffic script listed in step 11, after playing traffic through ubuntu box "ping 8.8.8.8"




---  
---  
---  

**DAY 6 MAY 8, 2023 NOTES (MONDAY)**  


**STEPS FOR INSTALLING FILEBEAT**  

1. From the sensor:  
`sudo yum install filebeat -y`  

2. `sudo mv /etc/filebeat/filebeat{.yml,.yml.bk}`  
takes the filebeat.yml and replaces it with filebeat.yml.bk.

3. `sudo curl -LO https://repo/fileshare/filebeat/filebeat.yml`  

`cat filebeat.yml` to make sure it worked and its there.  

4. `sudo vi /etc/filebeat/filebeat.yml`  

Change line **34** "hosts" to include all pipeline nodes {0-2}

``` hosts: ["pipeline0:9092","pipeline1:9092","pipeline2:9092"]```  

5. Go over to pipeline0 and enter this command to verify that there is only one topic enabled in kafka, "zeek-raw":

`sudo /usr/share/kafka/bin/kafka-topics.sh --list --zookeeper pipeline0:2181`

6. Now we enable the fsf-raw and suricata-raw topics within KAFKA with these long AF commands:  

`sudo /usr/share/kafka/bin/kafka-topics.sh --create --zookeeper pipeline0:2181 --replication-factor 3 --partitions 3 --topic fsf-raw`  



`sudo /usr/share/kafka/bin/kafka-topics.sh --create --zookeeper pipeline0:2181 --replication-factor 3 --partitions 3 --topic suricata-raw`  

Verify with first command in step 5.  

7. Back to sensor node, start filebeat service but DONT enable it:  
`sudo systemctl start filebeat`

8. generate traffic in the sensor, `curl google.com`  
Then go to pipeline0 and run these to check for **suricata** and **fsf**:  

`sudo /usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server pipeline0:9092 --topic suricata-raw --from-beginning`  

`sudo /usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server pipeline0:9092 --topic fsf-raw --from-beginning`  

 ---  
 ---  

 
 **STEPS FOR INSTALLING ELASTICSEARCH (multi-node)** 

 1. Go into your elastic0 node:  
 `sudo yum install elasticsearch -y`  

 2. `sudo mv /etc/elasticsearch/elasticsearch{.yml,.yml.bk}`  

 3. From home elastic:  
 
 `sudo curl -LO https://repo/fileshare/elasticsearch/elasticsearch.yml`  

 4. `sudo vi /etc/elasticsearch/elasticsearch.yml` steps for **multi-node** Elasticsearch


 ```Check WORD  
```
5. `STEP HAS BEEN REMOVED`  

6. `sudo chmod 640 /etc/elasticsearch/elasticsearch.yml`  

7. `sudo mkdir /usr/lib/systemd/system/elasticsearch.service.d`  

8. `sudo chmod 755 /usr/lib/systemd/system/elasticsearch.service.d`  

9. `sudo vi /usr/lib/systemd/system/elasticsearch.service.d/override.conf` supposed to be blank  

```[Service]
LimitMEMLOCK=infinity
```  

10. `sudo chmod 644 /usr/lib/systemd/system/elasticsearch.service.d/override.conf`  

    `sudo systemctl daemon-reload`  

11. Allocating memory properly for elasticsearch (not doing this can lead to poor kit performance).  

`sudo vi /etc/elasticsearch/jvm.options.d/jvm_override.conf`  

``` 
-Xms2g
-Xmx2g
```


12. `sudo mkdir -p /data/elasticsearch`  
    
    `sudo chown elasticsearch: /data/elasticsearch/`  
    
    `sudo chmod 755 /data/elasticsearch/`  

13. `sudo firewall-cmd --add-port={9200,9300}/tcp --permanent`  
    `sudo firewall-cmd --reload`  


    ---  
    --- 
    ---  
   
   
    **MULTI-NODE STEPS FOR ELASTICSEARCH (repeat previous steps, except the .yml step 4 line 949...this is the one you will change)**  


STEP FUCKING 1.   
 
`sudo chown -R root:elasticsearch elasticsearch.yml`   
if not letting you, then force it with `sudo -s` .


STEP FUCKING 2. `sudo systemctl daemon-reload`  

STEP FUCKING 3. `sudo systemctl start elasticsearch` 
                `sudo systemctl enable elasticsearch --now`

If having trouble starting check with this command:  

`journalctl -xeu elasticsearch`  



**REPEAT THESE STEPS ON EACH ELASTIC NODE**  

To test functionality run this from elastic0 (can be any node):  
`curl elastic0:9200/_cat/nodes`  

`curl elastic0:9200/_cat/nodes?v`

---  
---  
---  


**STEPS FOR SETTING UP KIBANA (SWITCH TO KIBANA NODE)**  

1. `sudo yum install kibana`  

2. `sudo mv /etc/kibana/kibana{.yml,.yml.bk}`  

3. `sudo vi /etc/kibana/kibana.yml`  
```
server.port: 5601
server.host: localhost
server.name: kibana
elasticsearch.hosts: ["http://elastic0:9200","http://elastic1:9200","http://elastic2:9200"]  
```  

4. `sudo yum install nginx`  

5. `sudo vi /etc/nginx/conf.d/kibana.conf`  
```server {
  listen 80;
  server_name kibana;
  proxy_max_temp_file_size 0;

  location / {
    proxy_pass http://127.0.0.1:5601/;

    proxy_redirect off;
    proxy_buffering off;

    proxy_http_version 1.1;
    proxy_set_header Connection "Keep-Alive";
    proxy_set_header Proxy-Connection "Keep-Alive";

  }

}  
```

6. `sudo vi /etc/nginx/nginx.conf`  
Comment out those 3 lines from before (way back in day 1-2): Lines **39-41**  under "server".  

7. `sudo firewall-cmd --add-port=80/tcp --permanent`  

   `sudo firewall-cmd --reload`  

8. `sudo systemctl enable nginx --now`  

   `sudo systemctl enable kibana --now`  

   **Give kibana a few minutes to load up before accessing the website** 


9. navigate to kibana via browser using the full ip address "10.81.139.50"  

10. `sudo curl -LO https://repo/fileshare/kibana/ecskibana.tar.gz` 

11. `tar -zxvf ecskibana.tar.gz`  
Uncompresses file

12. `sudo yum install jq -y`  

13. `sudo ./import-index-templates.sh http://elastic0:9200`  
Do this command from within the directory that this template was curled from. 
     `cd ~/elastic/ecskibana`

Pushes over our indexes to pipeline0.

---  
---  
---  


# **DAY 8 MAY 10, 2023**  

**STEPS FOR SETTING UP LOGSTASH**  

1. Starting from pipeline0:  
`sudo yum install logstash -y`  

2. `sudo curl -LO https://repo/fileshare/logstash/logstash.tar.gz`  

3. `sudo tar -zxvf logstash.tar.gz -C /etc/logstash`  

4. `sudo chown logstash: /etc/logstash/conf.d/*`  

5. `sudo chmod -R 744 /etc/logstash/conf.d/ruby`  

6. `cd /etc/logstash/conf.d/`  
   `ll`  
   To check inside and make sure its populated with log entries.  

7. `sudo vi logstash-100-input-kafka-zeek.conf`  
Within this config, enter this within escape mode:  

```%s/127.0.0.1:9092/pipeline0:9092,pipeline1:9092,pipeline2:9092/g```  OR  

you can run this on whichever pipeline node you're on:  

`sudo sed -i s/127.0.0.1:9092/pipeline0:9092,pipeline1:9092,pipeline2:9092/g logstash-100-input-kafka-{suricata,fsf,zeek}.conf`  

8.  `sudo vi logstash-9999-output-elasticsearch.conf`  

**Either Within this directory**  you can run this command outside the config and on the pipeline0 node:  

`sudo sed -i 's/"127.0.0.1"/"elastic0", "elastic1", "elastic2"/g' logstash-9999-output-elasticsearch.conf`  

9. Can run this command to test your output/verify everything is working:  

`sudo -u logstash /usr/share/logstash/bin/logstash -t --path.settings /etc/logstash`  

---  
---  
---  

**REPEAT THESE PREVIOUS STEPS ONCE YOU REACH THIS POINT ON EACH PIPELINE NODE**  

10. `sudo systemctl enable logstash --now` 

11. Go to Kibana webpage; navigate to stack management--index patterns. **(Keep in mind that logstash service takes a while to fully load up).**  Generate index patterns for Suricata, Zeek, and FSF (check word document for screenshot).  

After that navigate to the discover page to make sure stuff is populating.  

To enable dank mode, click on "Stack Management" then on "Advanced Settings in the bottom left and find enable dark mode.  





































 
  

 


  



  















    






 















    
























