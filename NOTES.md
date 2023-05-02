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
7.   
    
























