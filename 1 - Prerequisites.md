# Enterprise DC/OS 1.12 Prerequisites Installation Guide

The prerequisites are to be run one every cluster node (Masters, Public Agent, Private Agent) including the Bootstrap node

## Prepare Node

### Log In and change to SU
```
sudo su
```

### Allow SUDO commands to run without password
```
sudo visudo
```
scroll down and remove the `#` from the line that specifies whether or not to require password when running `sudo` commands

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

### Disable ipV6 (This section suspect)
```
sudo sed -i -e 's/Defaults    requiretty/#Defaults    requiretty/g' /etc/sudoers
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1
sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1
```

### Stop # Disable "firewalld"
```
sudo systemctl stop firewalld && sudo systemctl disable firewalld
```

### Stop # Disable "dnsmasq"
```
sudo systemctl stop dnsmasq && sudo systemctl disable dnsmasq.service
```

### Set SE Linux to Permissive
```
sudo sed -i 's/SELINUX=enforcing/SELINUX=permissive/g' /etc/selinux/config
```

### Add Groups
```
sudo groupadd nogroup &&
sudo groupadd docker
```

### Set Locale
```
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

### Install, Start, and Enable Docker CE with OverlayFS on CentOS/RedHat

The official Mesosphere supported version of Docker is the Red Hat fork of version 1.13.1-1el7.  It can be installed as below.
```
echo ">>> Install Docker"
curl -fLsSv --retry 20 -Y 100000 -y 60 -o /tmp/docker-engine-1.13.1-1.el7.centos.x86_64.rpm \
 https://yum.dockerproject.org/repo/main/centos/7/Packages/docker-engine-1.13.1-1.el7.centos.x86_64.rpm
curl -fLsSv --retry 20 -Y 100000 -y 60 -o /tmp/docker-engine-selinux-1.13.1-1.el7.centos.noarch.rpm \
 https://yum.dockerproject.org/repo/main/centos/7/Packages/docker-engine-selinux-1.13.1-1.el7.centos.noarch.rpm
yum -y localinstall /tmp/docker*.rpm || true
systemctl start docker
systemctl enable docker

sudo docker pull nginx
sudo docker ps
```

### Reboot Again
```
sudo reboot
```
Procees to step "2" and prepare your bootstrap node.
