# Enterprise DC/OS latest Prerequisites Installation Guide
## Updated for 1.13

The prerequisites are to be run one every cluster node (Masters, Public Agent, Private Agent) including the Bootstrap node

## Compatibility Matrix

[DC/OS 1.13](https://docs.mesosphere.com/version-policy/)

## Prepare Node

### Log In and change to SU
```
sudo su
```

### Allow SUDO commands to run without password
```
sudo visudo
```

### OPTIONAL: Set PS1 for each node
edit /etc/bashrc

\# \[ "$PS1" = "\\s-\\v\\\$ " \] && PS1="\[\u@\h \w\]\\$ "

PS1='\u@\H:\w\$ ' 

### Update Centos to 7.5 If Necessary
```
sudo cat /etc/redhat-release
sudo yum check-update
sudo yum update -y
sudo reboot
cat /etc/redhat-release
```

### Install Utility/Helper Applications
```
sudo yum install -y yum-utils xz bash net-utils bind-utils coreutils gawk gettext grep iproute util-linux curl ipset sed wget net-tools unzip
```

### Install and Configure NTP
Time synchronization is especially important for the master nodes to converge into a cluster control plane.  Please ensure that they are coirrectly synced prior to instlling the system.  Common places of trouble are excessive d=rift preventing resync, or BIOS hardware clocks that are not properly synchronized
```
sudo yum remove -y chrony
sudo yum install -y ntp
sudo systemctl enable ntpd
sudo systemctl start ntpd
sudo timedatectl set-ntp 1
```

### Install JQ
```
sudo wget http://stedolan.github.io/jq/download/linux64/jq
sudo chmod +x ./jq
sudo cp jq /usr/bin
```

### Stop # Disable "firewalld" | "dnsmasq" | SELINUX to Permissive | Add Groups | Set Locale
```
sudo systemctl stop firewalld && sudo systemctl disable firewalld
sudo systemctl stop dnsmasq && sudo systemctl disable dnsmasq.service
sudo sed -i 's/SELINUX=enforcing/SELINUX=permissive/g' /etc/selinux/config
cat /etc/selinux/config | grep SELINUX=permissive
sudo groupadd nogroup && sudo groupadd docker
sudo localectl set-locale LANG=en_US.utf8
```

### Reboot
```
sudo reboot
```

### Log Back In and change to SU
```
sudo su
```

### Install, Start, and Enable Docker
Got to figure this out by OS
```
mkfs -t xfs -n ftype=1 /dev/sdc1
```

Docker 18.09 Install:
```
curl -O  https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-18.09.2-3.el7.x86_64.rpm
curl -O https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-cli-18.09.2-3.el7.x86_64.rpm
curl -O https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
yum -y localinstall ./containerd*.rpm ./docker*.rpm || true
systemctl start docker
systemctl enable docker

sudo docker pull nginx
sudo docker ps
```

### Reboot Again
```
sudo reboot
```
[Procees to step "2" and prepare your bootstrap node.](https://github.com/jdyver/Enterprise-DC-OS-LATEST-Install-Cheatsheet/blob/master/2%20-%20Bootstrap%20Preparation.md)
