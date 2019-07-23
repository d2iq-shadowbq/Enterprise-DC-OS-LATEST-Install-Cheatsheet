### Production Readiness

#### This doc is currently based from 1.9 written by Vishnu, so just a reference at this point...

Some production-readiness recommendations I made to PTC/Thingworx that I thought y'all may find useful:

1. Docker Engine Configuration: Enable --live-restore (requires Docker 1.12 or higher on RHEL/CentOS 7.3 and DC/OS 1.9+) and set --log-
driver to none globally

  - https://docs.docker.com/engine/admin/live-restore/

  - https://docs.docker.com/engine/admin/logging/overview/
    
2. Do not run Consul or any other software on the DC/OS Master nodes.

3. Configure MOUNT volumes for your Data Services (Kafka, Cassandra and Elasticsearch). DC/OS will auto-detect valid mount points of the form / dcos/volume[n] e.g., /dcos/volume0 /dcos/volume1... and configure them as MOUNT volumes as can be reviewed in /var/lib/
dcos-mesos-resources

  - https://docs.mesosphere.com/1.9/storage/mount-disk-resources/ 
    
  - https://mesos.apache.org/documentation/latest/multiple-disk/

4. Configure Marathon's unreachableStrategy and killSelection application definition directives to configure how it should react in the presence of network partitions but note 
  - https://jira.mesosphere.com/browse/MARATHON-7515

  - https://mesosphere.github.io/marathon/docs/configure-task-handling.html
    
5. Also review upgradeStrategy to configure how tasks are replaced when configuration updates are made

  - https://mesosphere.github.io/marathon/docs/deployments.html#rolling-restarts
  - https://mesosphere.github.io/marathon/docs/rest-api.html#upgrade-strategy
  - https://mesosphere.github.io/marathon/docs/persistent-volumes.html#upgradingrestarting-stateful-applications
        
6. For AWS: use Terraform to cloud-init/cloud-config Ansible (or your favorite config management system) to trigger the DC/OS Advanced Install process (bash dcos_install.sh <master|slave|slave_public>) but follow the best practices for AWS from our CloudFormation templates to configure your VPC, Security Groups, Elastic Load Balancers, Autoscaling Groups (but know about the implications for Stateful services when you tell an ASG to scale down the instance count, otherwise works well
for Stateless services) etc.,

  - https://github.com/dcos/dcos/blob/master/gen/aws/templates/cloudformation.json

7. Use the DC/OS Diagnostics Toolkit (3dt) System Health APIs to monitor the health of the various DC/OS Components: https://dcos.io/docs/latest/monitoring/#system-health-http-api-endpoint

8. You can troubleshoot Marathon API deployments by inspecting the v2/ queue endpoint: https://mesosphere.github.io/marathon/docs/rest-api.html#queue

9. Come DC/OS 1.9.1 you can setup DNSMasq to enable custom DNS name resolution order on the agents (.consul -> Consul servers, .mesos -> Spartan -> Mesos-DNS, everything else -> PTC/Thingworx DNS servers): https://github.com/dcos/dcos/pull/1491
    
#### Advanced Recommendations:
Ref: https://mesos.apache.org/documentation/latest/configuration/
Note: You'll have to set MESOS_ENFORCE_CONTAINER_DISK_QUOTA=true in / var/lib/dcos/mesos-slave-common for Quota to be enforced.  Enable XFS Quota-based Isolation Support in DC/OS to prevent agent crashes due when frameworks consume all of the disk space in /var/lib/mesos.  I've created an issue and a PR to turn on the --enable-xfs-disk-isolator Apache Mesos build flag:
    JIRA: https://jira.mesosphere.com/browse/DCOS_OSS-1232
    PR: https://github.com/dcos/dcos/pull/1648
            
Use the DC/OS Metrics API to consume metrics for Kafka, Cassandra, Elastic (and dcos-commons SDK-derived frameworks, in general) from the system/v1/ metrics/v0/containers/<container-id>/app endpoint and that you may choose to write a metrics plugin for <insert_monitoring_platform> in the longer
term: https://github.com/dcos/dcos-metrics/tree/master/plugins
      Note: We're working on a revamp of the Metrics and Logging API to make it easier
to consume in future releases.
 I also strongly recommend that you default MESOS_CGROUPS_LIMIT_SWAP to true in /var/lib/dcos/mesos-slave-common - https://jira.mesosphere.com/browse/DCOS_OSS-710 and https://githSensuub.com/dcos/dcos/pull/1326 (should ship as the default in DC/OS 1.10)
Note: This doesn't yet work on Ubuntu (on Azure) due to: https://github.com/dcos/dcos/pull/1326#issuecomment-308252387

DC/OS (Name) Spaces:
e.g.,
/dev/sales/enterprise/crm/leadgen
/qa/sales/enterprise/crm/leadgen
/prod/sales/enterprise/crm/leadgen
Secrets Validation Rules:
The following secrets may be accessed by the application:
/secret
/some/secret
/some/app/secret
/some/app/id/secret
But these secrets are inaccessible to the application:
/some/ap/secret
/some/appp/secret
/some/app/id/secret/too
/admin/secret
/admin/super/secret

I recommend that you setup and deploy your applications into namespaces of the form (or appropriate equivalent of): /<Environment>/<Business_Unit>/<Sub-Businsess_Unit>/<Project>/<App>
Given an app residing in the following DC/OS Namespace (a.k.a Marathon App ID): /some/app/id

Service Account Credentials as required by Marathon-LB and infrastructure- oriented services should be stored in a disjoint namespace from application
services under /dcos
e.g., /dcos/root/marathon-lb/serviceCredential to prevent accidental or malicious access to the Root Marathon’s Marathon-LB Service Account
Credential (Private Key)
 Note: I'd recommend that you place the secret at the leaf of the common path.

  i.e., if you intend for an app /dev/foo/bar to access a secret, create the secret
at dev/foo/bar/secret
 If the secret was instead defined at dev/foo/secret then any application
prefixed at /dev/foo
 e.g., /dev/foo/bar and /dev/foo/baz will be able to access dev/foo/
secret

DC/OS IAM ACLs Reference: https://docs.mesosphere.com/security/perms-
     reference/
 Sample Marathon App Definition with Secrets:
{
  "id": "/dev/foo/bar/web",
  "instances": 1,
  "cpus": 0.1,
  "mem": 32,
  "env": {
    "DEV_FOO_BAR_WEB_SECRET": {
      "secret": "secret0"
} 
  },
  "secrets": {
    "secret0": {
      "source": "dev/foo/bar/web/secret"
    }
  },
  "portDefinitions": [
    {
      "port": 0,
      "protocol": "tcp",
      "name": "web",
      "labels": {
      "VIP_0": "/dev.foo.bar.web:80"
      }
    }
  ],
  "healthChecks": [
    {
      "cmd": "env | sort && /opt/mesosphere/bin/python -m http.server $PORT0",
      "portIndex": 0,
      "path": "/",
      "gracePeriodSeconds": 30,
      "intervalSeconds": 20,
      "timeoutSeconds": 10,
      "maxConsecutiveFailures": 3
      } 
    ]
}

Note: DC/OS 1.10 will support frameworks that are deployed into Application Groups but for now you can manually override the DCOS_SERVICE_NAME, FRAMEWORK_NAME, FRAMEWORK_PRINCIPAL, FRAME WORK_ROLE and VIP labels
 
e.g., Given a framework (Kafka) being deployed into: "id": "/dev/foo/bar/kafka", We have to override the following fields in env: "FRAMEWORK_NAME": "dev-foo-bar-kafka", "FRAMEWORK_PRINCIPAL": "dev-foo-bar-kafka-principal" "FRAMWEWORK_ROLE": "dev-foo-bar-kafka-role".  We have to override the following fields in labels: "DCOS_SERVICE_NAME": "dev-foo-bar-kafka",
And override the Kafka Scheduler API VIP label in portDefinitions
  "portDefinitions": [
    {
      "port": 0,
      "protocol": "tcp",
      "name": "api",
      "labels": {
        "VIP_0": "/api.dev.foo.bar.kafka:80"
      }
    }
  ],
You can generate the Scheduler's app definition with: "protocol": "MESOS_HTTP",
  dcos package describe --app --render --options=<your_framework_options_and_overrides> <framework>
e.g.,
1. dcos package describe --app --render --options=kafka-options.json kafka > kafka.json
2. Edit kafka.json to implement the manual overrides to enable the framework to be deployed into the Marathon Application Group successfully
3. dcos marathon app add kafka.json
