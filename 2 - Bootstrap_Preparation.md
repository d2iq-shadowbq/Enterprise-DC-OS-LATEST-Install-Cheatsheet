# Enterprise DC/OS - Prepare Bootstrap Node
After installing the [Prerequisites](https://github.com/jdyver/Enterprise-DC-OS-LATEST-Install-Cheatsheet/blob/master/1%20-%20Prerequisites.md) on all nodes including the bootstrap nodes, do the following on the server designated as the Bootstrap node.

### Get to the Bootstrap's shell

### Create Bootstrap Directories in Home Directory
```
mkdir -p dcos-install/genconf
cd dcos-install
```

### Create IP-detect
For cloud specific ip-detect scripts, [please see the docs](https://docs.d2iq.com/mesosphere/dcos/2.0/installing/production/deploying-dcos/installation/#use-the-aws-metadata-server).

### Check network interface name on-prem
Should primarily use this ip-detect (route based)

- Change the MASTER_IP to a master's IP : )
```
#!/usr/bin/env bash
set -o nounset -o errexit
MASTER_IP=172.28.128.3
echo $(/usr/sbin/ip route show to match $MASTER_IP | grep -Eo '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' | tail -1) | tr -d '\n'
```

If the above works, then skip down to 'Create Fault Domain' section

Otherwise: Need to know for ip-detect scripts below
```
ip addr

ifconfig
```
These are two examples of creating an ip-detect script.  This directly below assumes that the primary ethernet interface is `eth0`.
```
cat > genconf/ip-detect << 'EOF'
#!/usr/bin/env bash
set -o nounset -o errexit
export PATH=/usr/sbin:/usr/bin:$PATH
echo $(ip addr show eth0 | grep -Eo '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' | head -1) | tr -d '\n'
EOF
```
CentOS (on VMware - ens165, 192; on AWS - ens5): Would like this to pick up ens*

- Change the INTERFACE_LABEL below
```
cat > genconf/ip-detect << 'EOF'
#!/usr/bin/env bash
set -o nounset -o errexit
export PATH=/usr/sbin:/usr/bin:$PATH
INTERFACE_LABEL=ens192               # If necessary, will eventually make this automatic
echo $(ip addr show $INTERFACE_LABEL | grep -Eo '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' | head -1) | tr -d '\n'
EOF
```

### Create Fault Domain Detect Script
This example reads 2 files (/var/region and /var/zone) to populate the @region and @zone labels on dcos nodes.  For other examples , please see the documentation  
```
cat > genconf/fault-domain-detect << 'EOF'
#!/bin/bash
ZONE_FILE=/var/zone
REGION_FILE=/var/region
REGION=$(cat ${REGION_FILE})
ZONE=$(cat ${ZONE_FILE})
echo "{\"fault_domain\":{\"region\":{\"name\": \"${REGION}\"},\"zone\":{\"name\": \"${ZONE}\"}}}"
EOF
```

### Create public-ip-detect
The following script uses the `dig` command and opendns to determine the public ip of the cluster.
```
cat > genconf/public-ip-detect << 'EOF'
#!/usr/bin/env bash
dig +short myip.opendns.com @resolver1.opendns.com
EOF
```

### Create license.txt file
```
cat > genconf/license.txt << 'EOF'
<Insert-License-File-Contents-Here>
EOF
```

### Create config.yaml

[Official Options:](https://docs.d2iq.com/mesosphere/dcos/2.0/installing/production/deploying-dcos/installation/#enterprise-template-enterprise)
This is an example config.yaml file with the minimum variables set.  The one optional variable set is the userID and password.  This is highly suggested if this cluster is to be at all exposed to the internet.  there are plenty of bitcoin miners out there that are looking for clusters with default user IDs and passwords.
```
cat > genconf/config.yaml << 'EOF'
bootstrap_url: http://<Bootstrap-IP-Address>:80
cluster_name: 'Cluster Name'
fault_domain_enabled: True
ip_detect_public_filename: genconf/public-ip-detect
exhibitor_storage_backend: static
master_discovery: static
master_list:
- <Master-IP-Address> 
resolvers:
- <DNS-Server-1>
- <DNS Server-2>
security: permissive
superuser_password_hash: <HashGoesHere>
superuser_username: <default-user-ID>
ssh_user: <default-ssh-user-ID
EOF
```

### Get the Bits
```
curl -O <https://downloads.mesosphere.io/blah blah blah (get the most recent url from the support site)>
```

### Create Password Hash
```
sudo bash dcos_generate_config.ee.sh --hash-password <default-user-ID>
```
copy the text output of the command and replace `<HashGoesHere>` in the config.yaml file you created earlier with it.

### Create the Docker Container
```
sudo bash dcos_generate_config.ee.sh
```

### Deploy the Docker Container
```
sudo docker run -d -p 80:80 -v ${PWD}/genconf/serve:/usr/share/nginx/html:ro nginx
```
### Verify Container Launched
```
sudo docker ps
```

[Move on to Installing Nodes (Master, Private Agent, and Public Agent)](https://github.com/jdyver/Enterprise-DC-OS-LATEST-Install-Cheatsheet/blob/master/3%20-%20Installation.md)

