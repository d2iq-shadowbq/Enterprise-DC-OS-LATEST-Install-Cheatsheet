### Debug Tips

1. Start with 1 master and make sure it goes all the way through

  - If you didn't start with a brand new OS, then plan on this failing

  - Then a single private and single public node
    
  - Avoid pre-flight to force installation (just doesn't seem to work when needed)
  
    - bash dcos_install.sh master --disable-preflight

2. Most issues are in the config.yaml, ip-detect / public-ip-detect scripts or NTP

  - Config.yaml

    - Check Master / Node IPs 3 times to make sure that they are correct
    
  - ip-detect

    - Check in each node
    
    - If "master" defined make sure it is set properly
    
    - In node (to update): /opt/mesosphere/bin/detect-ip
    
  - ip-detect-public

    - Need to document
    
  - NTP

    - Disable timecheck: Single master (or multi-masters on VMs (rebooted) - vulnerable though)
    
      - /opt/mesosphere/etc/check_time.env
    
3. In the master node (or whatever is failing)

- Priorities:

  - systemctl status dcos-*
        
    - Find failured services then dig into logs

  - Zookeeper (http://\<ip\>:8181/exhibitor/v1/ui/index.html)
        
    - journalctl -fu dcos-exhibitor

  - Admin Router
    
    - journalctl -fu dcos-admin*?
    
  - Mesos Master(?)
  
    - journalctl -fu dcos-mesos-master?
    
4. In the private node
 
- dcos-net.service (has to come up first)
- systemctl status dcos-mesos-slave.service
  - journalctl -fu dcos-mesos-slave.service
- dcos-adminrouter-agent.service

[Official Triage](https://mesosphere.lightning.force.com/lightning/r/Knowledge__kav/ka0f1000000kCJRAA2/view)

### General Install Triage Steps
1. Collect a DC/OS log bundle
2. Verify you have a valid IP detect﻿⁠⁠⁠⁠ script, functioning DNS resolvers to bind the DC/OS services to, and that all nodes are synchronized with NTP.
- Exhibitor
- Mesos master
- Mesos DNS
- DNS Forwarder (Spartan)
- DC/OS Marathon
- Jobs
- Admin Router
  - extra lines
  - white space
  - special or hidden characters
  - hostname -f returns the FQDN
  - hostname -s returns the short hostname
 
### IP detect script
You must have a valid [ip-detect](https://docs.mesosphere.com/1.11/installing/ent/custom/advanced/) script. You can manually run ip-detect on all the nodes in your cluster or check /opt/mesosphere/bin/detect_ip on an existing installation to ensure that it returns a valid IP address. A valid IP address does not have:

It is recommended that you use the ip-detect [examples](https://docs.mesosphere.com/1.11/installing/ent/custom/advanced/). 
 
### DNS resolvers
You must have working DNS resolvers, specified in your [config.yaml](https://docs.mesosphere.com/1.11/installing/ent/custom/configuration/configuration-parameters/#resolvers) file. It is recommended that you have forward and reverse lookups for FQDNs, short hostnames, and IP addresses. It is possible for DC/OS to function in environments without valid DNS support, but the following must work to support DC/OS services, including Spark:


You should sanity check the output of hostnamectl on all of your nodes as well.


When troubleshooting problems with a DC/OS installation, you should explore the components in this sequence:

Be sure to check that all services are up and healthy on the masters before checking the agents.

### NTP
Network Time Protocol (NTP) must be enabled on all nodes for clock synchronization. By default, during DC/OS startup you will receive an error if this is not enabled.
You can check if NTP is enabled by running one of these commands, depending on your OS and configuration: 

```
ntptime
adjtimex -p
timedatectl
```
 
1. Ensure that firewalls and any other connection-filtering mechanisms are not interfering with cluster component communications. TCP, UDP, and ICMP must be permitted.

Ensure that services that bind to port 53, which is required by DNS Forwarder (dcos-spartan.service), are disabled and stopped. For example:

```
sudo systemctl disable dnsmasq && sudo systemctl stop dnsmasq
```

2. Verify that Exhibitor is up and running at [http://<MASTER_IP>:8181/exhibitor]. If Exhibitor is not up and running:

- [SSH](https://docs.mesosphere.com/1.11/administering-clusters/sshcluster/) to your master node and enter this command to check the Exhibitor service logs:

```
journalctl -flu dcos-exhibitor
```

- Verify that /tmp is mounted without noexec. If it is mounted with noexec, Exhibitor will fail to bring up ZooKeeper because Java JNI won’t be able to exec a file it creates in /tmp and you will see multiple permission denied errors in the log. To repair /tmp mounted with noexec:

  - Enter this command:

```
mount -o remount,exec /tmp
```

  - Check the output of /exhibitor/v1/cluster/status and verify that it shows the correct number of masters and that all of them are "serving" but only one of them is designated as "isLeader": true

For example, [SSH](https://docs.mesosphere.com/1.11/administering-clusters/sshcluster/) to your master node and enter this command:

```
curl -fsSL http://localhost:8181/exhibitor/v1/cluster/status | python -m json.tool
[
    {
        "code": 3,
        "description": "serving",
        "hostname": "10.0.6.70",
        "isLeader": false
    },
    {
        "code": 3,
        "description": "serving",
        "hostname": "10.0.6.69",
        "isLeader": false
    },
    {
        "code": 3,
        "description": "serving",
        "hostname": "10.0.6.68",
        "isLeader": true
    }
]
```

**Note:** Running this command in multi-master configurations can take up to 10-15 minutes to complete. If it doesn’t complete after 10-15 minutes, you should carefully review the journalctl -flu dcos-exhibitor logs.

 
3. Verify whether you can ping the DNS Forwarder (ready.spartan). If not, review the DNS Dispatcher service logs: ﻿⁠⁠⁠⁠

```
journalctl -flu dcos-spartan﻿⁠⁠⁠⁠
```

4. Verify that you can ping ⁠⁠⁠⁠leader.mesos and ﻿⁠⁠⁠⁠master.mesos. If not:
 
- Review the Mesos-DNS service logs with this command:

```
journalctl -flu dcos-mesos-dns﻿⁠⁠⁠⁠
```
 
- If you are able to ping ready.spartan, but not leader.mesos, review the Mesos master service logs by using this command:

```
journalctl -flu dcos-mesos-master
```

The Mesos masters must be up and running with a leader elected before Mesos-DNS can generate its DNS records from ⁠⁠⁠\/state.⁠⁠⁠⁠



 
 
[Official Debugging](https://docs.d2iq.com/mesosphere/dcos/1.13/installing/troubleshooting/)
    
4. Forget about uninstallation to update something

    - https://dcosjira.atlassian.net/browse/DCOS-250
 
    - If you want to try: https://groups.google.com/a/dcos.io/forum/#!topic/users/vDJyTXfEqqU
