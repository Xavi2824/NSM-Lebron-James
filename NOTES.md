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
`sudo vi /etc/sysconfig/network-scripts/ifcfg-etho0`  
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
so now all you have to do to SSH into a node is "ssh kibana"
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


**Enable creation of File sharing**  
`sudo yum install nginx`  


**Unzips the all class file and places it /usr/share/nginx**

`sudo unzip ~/all-class-files.zip -d /usr/share/nginx`  

**renames the all class files into "fileshare"**  

`sudo mv /usr/share/nginx/all-class-files/ /usr/share/nginx/fileshare`

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


**List ports included**  
`ss -lnt`  

**navigate to repo url with port 8000**  
10.81.139:8000
