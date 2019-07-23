### Debug Tips

1. Start with 1 master and make sure it goes all the way through

    - If you didn't start with a brand new OS, then plan on this failing

    - Then a single private and single public node

2. Most issues are in the config.yaml and ip-detect / public-ip-detect scripts

- Config.yaml

    - Check Master / Node IPs 3 times to make sure that they are correct
    
- ip-detect

    - Check in each node
    
    - If "master" defined make sure it is set properly
    
    - In node (to update): /opt/mesosphere/bin/detect-ip
    
- public-ip-detect

    - Need to document
    
3. In the master node (or whatever is failing)

- Priorities:

    - Zookeeper

    - Admin Router
    
    - 
    
4. Forget about uninstallation to update something
 
    - If you want to try: https://groups.google.com/a/dcos.io/forum/#!topic/users/vDJyTXfEqqU
