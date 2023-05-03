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
`sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0`  
`sudo vi /etc/sysctl.conf`  
`:%d`  
**Change network-scripts etho0 to this**
```  
DEVICE=eth0
BOOTPROTO=none
ONBOOT=yes
HOSTNAME=elastic0
NM_CONTROLLED=no
TYPE=Ethernet
IPADDR=10.81.139.30
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
`ssh-keygen`   

**NEED FOR LOOP**  

for host in sensor repo elastic{0..2} pipeline{0..2} kibana; do ssh-copy-id $host; done


**Enable creation of File sharing**  
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
`sudo createrepo local-extras`
---  
**Creating a nginx config to listen on port 8008, VIA root /repo directory**  

```server {
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

**make a directory for archive**  

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


   ``` server {

  listen 127.0.0.1:8008;

  location / {

    root /repo;

    autoindex on;

    index index.html index.htm;

  }

}
```  
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

fi  
```  
10. Change the permissions of ifup-local.  
   `sudo chmod +x /sbin/ifup-local`  
11. `sudo vi /etc/sysconfig/network-scripts/ifup`  
  At the very bottom, add this line of code under "exec" line,   
  ``` if [ -x /sbin/ifup-local ]; then
/sbin/ifup-local ${DEVICE}
fi
```  
12. `sudo vi /etc/sysconfig/network-scripts/ifcfg-eth1`  
```DEVICE=eth1
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
  Change line **1506**: no (change from yes to no)  
  Change line **1521**: no (change from yes to no)
  Change line **1527**: no (change from yes to no)
  Change line **1536**: no (change from yes to no)  

  5. `vi /etc/sysconfig/suricata`  
  Change the last line of this config file to this:  
  ```OPTIONS="--af-packet=eth1 --user suricata --group suricata "```  
  6. Go back into `vi /etc/sysconfig/suricata`  
  Change line 1439: yes (change from no to yes)  
  Change line 1457: cpu ["0-2"]  
  Change line 1464: medium [1]  
  Change line 1465: high [2]  
  Change line 1466: default set to "high".  


  7. `sudo suricata-update add-source emergingthreats https://repo/fileshare/emerging.rules.tar.gz` 

  8. `sudo suricata-update` update takes a minute or two. 
  9. `sudo mkdir -p /data/suricata`  
  
  10. `sudo chown -R suricata:suricata /data/suricata`  












    
























