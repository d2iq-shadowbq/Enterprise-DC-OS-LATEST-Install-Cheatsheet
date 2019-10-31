# Install Enterprise DC/OS Cluster Nodes
[Back to bootstrap steps](https://github.com/jdyver/Enterprise-DC-OS-LATEST-Install-Cheatsheet/blob/master/2%20-%20Bootstrap_Preparation.md)

I suggest you start with the master node and wait for it to become fully healthy.  That way, you have a pretty good assurance that all of your prerequisites were done correctly.  In fact, I would install one of each type of node, and wait for it to become healthy in the dashboard before installing everything enmass.

## Set Region and Zone Labels on All Nodes

**Skip this section if you don't have multiple domains**

These files are referenced by the "fault-domain-defect" script run at agent startup

**Deploy Fault Domain "region" File**
Enter the @region where the node will live in place of `<region-name here>`
```
sudo cat > /var/region << 'EOF'
<region-name here>
EOF
```

**Deploy Fault Domain "zone" File**
Enter the @zone where the node will live in place of `<zone-name here>`
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

