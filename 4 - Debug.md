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




 
[Official Debugging](https://docs.d2iq.com/mesosphere/dcos/1.13/installing/troubleshooting/)
    
4. Forget about uninstallation to update something

    - https://dcosjira.atlassian.net/browse/DCOS-250
 
    - If you want to try: https://groups.google.com/a/dcos.io/forum/#!topic/users/vDJyTXfEqqU
