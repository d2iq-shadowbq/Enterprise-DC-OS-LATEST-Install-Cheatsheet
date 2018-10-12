# Install Enterprise DC/OS 1.12 BETA Cluster Nodes
I suggest you start with the master node and waid for it to become fully healthy.  That way, you have a pretty good assurance that all of your prerequisites were done correctly.

## Set Region and Zone Labels on All Nodes
These files are referenced by the "fault-domain-defect" script run at agent startup

### Deploy Fault Domain "region" File
```
sudo cat > /var/region << 'EOF'
<region-name here>
EOF
```

### Deploy Fault Domain "zone" File
```
sudo cat > /var/zone << 'EOF'
<zone-name here>
EOF
```

## Install Enterprise DC/OS Cluster Node Software
First SSH into the individual node and execute the command based on its node type

### Install Master Node Software
```
mkdir /tmp/dcos && cd /tmp/dcos
curl -O http://<Bootstrap-IP-Address:Port>/dcos_install.sh && sudo bash dcos_install.sh master
```
----------------------------------------------------------------->

### Install Public Node Software
```
mkdir /tmp/dcos && cd /tmp/dcos
curl -O http://<Bootstrap-IP-Address:Port>/dcos_install.sh && sudo bash dcos_install.sh slave_public
```
----------------------------------------------------------------->>

### Install Private Node Software
```
mkdir /tmp/dcos && cd /tmp/dcos
curl -O http://<Bootstrap-IP-Address:Port>/dcos_install.sh && sudo bash dcos_install.sh slave
```
----------------------------------------------------------------->>>
