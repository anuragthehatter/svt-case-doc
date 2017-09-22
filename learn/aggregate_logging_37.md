# Aggregate Logging

[Guideline](https://docs.openshift.org/latest/install_config/aggregate_logging.html)

## Installation via Ansible (Internal)
Installation is performed on master.


Check the logging driver:

```sh
# docker info 
...
Logging Driver: journald
...
```

### Modify the inventory file

journald with mux:

```sh
[OSEv3:children]                               
masters                                        
etcd                                           

[masters]                                      
ec2-54-186-165-141.us-west-2.compute.amazonaws.com                                              

[etcd]                                         
ec2-54-186-165-141.us-west-2.compute.amazonaws.com                                              
[OSEv3:vars]                                   
deployment_type=openshift-enterprise                                                          

openshift_deployment_type=openshift-enterprise                                                
openshift_release=v3.7.0
#The 3 following vars have to match the master config
openshift_portal_net=172.24.0.0/14
osm_cluster_network_cidr=172.20.0.0/14
osm_host_subnet_length=9


openshift_logging_install_logging=true         
openshift_logging_master_url=https://ec2-54-186-165-141.us-west-2.compute.amazonaws.com:8443    
openshift_logging_master_public_url=https://ec2-54-186-165-141.us-west-2.compute.amazonaws.com:8443
openshift_logging_kibana_hostname=kibana.apps.0922-mtp.qe.rhcloud.com                              
openshift_logging_namespace=logging            
openshift_logging_image_prefix=registry.ops.openshift.com/openshift3/                         
openshift_logging_image_version=v3.7.0           
openshift_logging_es_cluster_size=3            
openshift_logging_es_pvc_dynamic=true          
openshift_logging_es_pvc_size=50Gi             
openshift_logging_fluentd_use_journal=true     
openshift_logging_fluentd_read_from_head=false 
openshift_logging_use_mux=true                 
openshift_logging_mux_client_mode=maximal      
openshift_logging_use_ops=false                

openshift_logging_fluentd_cpu_limit=1000m      
openshift_logging_mux_cpu_limit=1000m          
openshift_logging_kibana_cpu_limit=200m        
openshift_logging_kibana_proxy_cpu_limit=100m  
openshift_logging_es_memory_limit=9Gi          
openshift_logging_fluentd_memory_limit=1Gi     
openshift_logging_mux_memory_limit=2Gi         
openshift_logging_kibana_memory_limit=1Gi      
openshift_logging_kibana_proxy_memory_limit=256Mi                                             

openshift_logging_mux_file_buffer_storage_type=pvc                                            
openshift_logging_mux_file_buffer_pvc_name=logging-mux-pvc                                    
openshift_logging_mux_file_buffer_pvc_size=30Gi

```


* Update hostnames and openshift_logging_image_version to match your cluster
* Upadte the subdomain for the kibana_hostname which can be found in master-config.yaml or in your flexy job output.
  It will be apps.mmdd-xxx.qe.rhcloud.com.
  
  ```sh
  # grep "qe.rhcloud.com" /etc/origin/master/master-config.yaml 
  subdomain:  "apps.<mmdd>-<xxx>.qe.rhcloud.com"
  # grep -i "masterPublicURL" /etc/origin/master/master-config.yaml
  # grep -i "masterURL" /etc/origin/master/master-config.yaml
  # docker images | grep ose
  ```

Note that <code>openshift_logging_fluentd_use_journal</code> tells _fluentd_ to checkout container logs from _journald_.



### Run [the playbook](https://github.com/openshift/openshift-ansible/blob/master/playbooks/byo/openshift-cluster/openshift-logging.yml)

```sh
# ansible-playbook -i /tmp/inv.file openshift-ansible/playbooks/byo/openshift-cluster/openshift-logging.yml
```

Check the parameter's meaning [here](https://docs.openshift.org/latest/install_config/aggregate_logging.html#install-config-aggregate-logging).

## Verify

```sh
# oc project logging
# oc get all -o wide
NAME                                                REVISION   DESIRED   CURRENT   TRIGGERED BY
deploymentconfigs/logging-curator                   1          1         1         config
deploymentconfigs/logging-es-data-master-39f9joda   1          1         1         config
deploymentconfigs/logging-es-data-master-43lag4ik   1          1         1         config
deploymentconfigs/logging-es-data-master-j763alrc   1          1         1         config
deploymentconfigs/logging-kibana                    1          1         1         config
deploymentconfigs/logging-mux                       1          1         1         config

NAME                    HOST/PORT                             PATH      SERVICES         PORT      TERMINATION          WILDCARD
routes/logging-kibana   kibana.apps.0922-mtp.qe.rhcloud.com             logging-kibana   <all>     reencrypt/Redirect   None

NAME                                         READY     STATUS    RESTARTS   AGE       IP            NODE
po/logging-curator-1-xzmk3                   1/1       Running   0          4m        172.20.2.17   ip-172-31-21-185.us-west-2.compute.internal
po/logging-es-data-master-39f9joda-1-p2qg2   1/1       Running   0          4m        172.23.0.7    ip-172-31-5-234.us-west-2.compute.internal
po/logging-es-data-master-43lag4ik-1-z383x   1/1       Running   0          4m        172.21.0.5    ip-172-31-23-229.us-west-2.compute.internal
po/logging-es-data-master-j763alrc-1-v1g9g   1/1       Running   0          4m        172.20.0.6    ip-172-31-10-173.us-west-2.compute.internal
po/logging-fluentd-03nz4                     1/1       Running   0          3m        172.22.0.3    ip-172-31-5-155.us-west-2.compute.internal
po/logging-fluentd-633f9                     1/1       Running   0          3m        172.20.2.19   ip-172-31-21-185.us-west-2.compute.internal
po/logging-fluentd-8qk3x                     1/1       Running   0          3m        172.21.0.6    ip-172-31-23-229.us-west-2.compute.internal
po/logging-fluentd-c9wh1                     1/1       Running   0          3m        172.23.0.8    ip-172-31-5-234.us-west-2.compute.internal
po/logging-fluentd-v59b7                     1/1       Running   0          3m        172.20.0.7    ip-172-31-10-173.us-west-2.compute.internal
po/logging-kibana-1-0xmzz                    2/2       Running   0          4m        172.20.2.15   ip-172-31-21-185.us-west-2.compute.internal
po/logging-mux-1-kb1h9                       1/1       Running   0          3m        172.20.2.20   ip-172-31-21-185.us-west-2.compute.internal

NAME                                   DESIRED   CURRENT   READY     AGE       CONTAINER(S)          IMAGE(S)                                                                                                                      SELECTOR
rc/logging-curator-1                   1         1         1         4m        curator               registry.ops.openshift.com/openshift3/logging-curator:v3.7.0                                                                  component=curator,deployment=logging-curator-1,deploymentconfig=logging-curator,logging-infra=curator,provider=openshift
rc/logging-es-data-master-39f9joda-1   1         1         1         4m        elasticsearch         registry.ops.openshift.com/openshift3/logging-elasticsearch:v3.7.0                                                            component=es,deployment=logging-es-data-master-39f9joda-1,deploymentconfig=logging-es-data-master-39f9joda,logging-infra=elasticsearch,provider=openshift
rc/logging-es-data-master-43lag4ik-1   1         1         1         4m        elasticsearch         registry.ops.openshift.com/openshift3/logging-elasticsearch:v3.7.0                                                            component=es,deployment=logging-es-data-master-43lag4ik-1,deploymentconfig=logging-es-data-master-43lag4ik,logging-infra=elasticsearch,provider=openshift
rc/logging-es-data-master-j763alrc-1   1         1         1         4m        elasticsearch         registry.ops.openshift.com/openshift3/logging-elasticsearch:v3.7.0                                                            component=es,deployment=logging-es-data-master-j763alrc-1,deploymentconfig=logging-es-data-master-j763alrc,logging-infra=elasticsearch,provider=openshift
rc/logging-kibana-1                    1         1         1         4m        kibana,kibana-proxy   registry.ops.openshift.com/openshift3/logging-kibana:v3.7.0,registry.ops.openshift.com/openshift3/logging-auth-proxy:v3.7.0   component=kibana,deployment=logging-kibana-1,deploymentconfig=logging-kibana,logging-infra=kibana,provider=openshift
rc/logging-mux-1                       1         1         1         3m        mux                   registry.ops.openshift.com/openshift3/logging-fluentd:v3.7.0                                                                  component=mux,deployment=logging-mux-1,deploymentconfig=logging-mux,logging-infra=mux,provider=openshift

NAME                     CLUSTER-IP      EXTERNAL-IP   PORT(S)     AGE       SELECTOR
svc/logging-es           172.24.228.83   <none>        9200/TCP    5m        component=es,provider=openshift
svc/logging-es-cluster   172.26.91.224   <none>        9300/TCP    5m        component=es,provider=openshift
svc/logging-kibana       172.27.70.64    <none>        443/TCP     4m        component=kibana,provider=openshift
svc/logging-mux          172.26.41.150   <none>        24284/TCP   3m        component=mux,provider=openshift


# oc get ds
NAME              DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE-SELECTOR                AGE
logging-fluentd   5         5         5         5            5           logging-infra-fluentd=true   8m

# oc get ds
NAME              DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE-SELECTOR                AGE
logging-fluentd   5         5         5         5            5           logging-infra-fluentd=true   8m

# oc get pvc
NAME              STATUS    VOLUME                                     CAPACITY   ACCESSMODES   STORAGECLASS   AGE
logging-es-0      Bound     pvc-741c716f-9fa0-11e7-8793-027497ece8ac   50Gi       RWO           gp2            9m
logging-es-1      Bound     pvc-7d0bc3d0-9fa0-11e7-8793-027497ece8ac   50Gi       RWO           gp2            9m
logging-es-2      Bound     pvc-86039fb9-9fa0-11e7-8793-027497ece8ac   50Gi       RWO           gp2            9m
logging-mux-pvc   Bound     pvc-9d812f54-9fa0-11e7-8793-027497ece8ac   30Gi       RWO           gp2            8m

# oc get pv
NAME                                       CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS    CLAIM                     STORAGECLASS   REASON    AGE
pvc-741c716f-9fa0-11e7-8793-027497ece8ac   50Gi       RWO           Delete          Bound     logging/logging-es-0      gp2                      10m
pvc-7d0bc3d0-9fa0-11e7-8793-027497ece8ac   50Gi       RWO           Delete          Bound     logging/logging-es-1      gp2                      10m
pvc-86039fb9-9fa0-11e7-8793-027497ece8ac   50Gi       RWO           Delete          Bound     logging/logging-es-2      gp2                      9m
pvc-9d812f54-9fa0-11e7-8793-027497ece8ac   30Gi       RWO           Delete          Bound     logging/logging-mux-pvc   gp2                      9m


# POD=logging-es-data-master-39f9joda-1-p2qg2
# oc exec $POD -- curl --connect-timeout 2 -s -k --cert /etc/elasticsearch/secret/admin-cert --key /etc/elasticsearch/secret/admin-key https://logging-es:9200/_cat/indices?v
health status index                                                           pri rep docs.count docs.deleted store.size pri.store.size 
green  open   project.default.7516c4b9-9f95-11e7-ace5-027497ece8ac.2017.09.22   1   0      18099            0     17.1mb         17.1mb 
green  open   .searchguard.logging-es-data-master-j763alrc                      1   2          5            0     91.3kb         30.4kb 
green  open   .kibana                                                           1   0          1            0      3.1kb          3.1kb 
green  open   .searchguard.logging-es-data-master-43lag4ik                      1   2          5            0     91.3kb         30.4kb 
green  open   .searchguard.logging-es-data-master-39f9joda                      1   2          5            0     91.3kb         30.4kb 
green  open   project.logging.fe039967-9f9f-11e7-8793-027497ece8ac.2017.09.22   1   0        711            0    738.1kb        738.1kb 
green  open   .operations.2017.09.22                                            1   0      25688            0       13mb           13mb 

```

If we need to redeplay the logging stack, we can delete logging project and recreate it, and then rerun the above playbook:

```sh
# oc delete project logging
# oadm new-project logging --node-selector=""
```

## Search (logs in Kibana)
Aggregate logging in Openshift collects, stores, and indexes logs genrated in the cluster. Eg, docker container logs.
Copy a keyword in the log entries, input it in the search box on Kibana web UI. We should see it in the returned results.

* On the top of navigation tree, choose <code>.all</code> which search all indecies in ElasticSearch.
* Choose a proper time range, *the last 15 mins* is the default.



```sh
# oc new-project aaa
Now using project "aaa" on server "https://ip-172-31-5-155.us-west-2.compute.internal:8443".

# oc create -f https://raw.githubusercontent.com/hongkailiu/svt-case-doc/master/files/rc_test.yaml
replicationcontroller "frontend-1" created

# oc get pod -o wide
NAME               READY     STATUS    RESTARTS   AGE       IP           NODE
frontend-1-pklzq   1/1       Running   0          5m        172.23.0.9   ip-172-31-5-234.us-west-2.compute.internal

# curl 172.23.0.9:8080
{"version":"0.0.1","ips":["127.0.0.1","::1","172.23.0.9","fe80::858:acff:fe17:9"],"now":"2017-09-22T14:53:27.044323764Z"}root@ip-172-31-5-155: ~ # curl -H "Content-Type: application/json" -X POST -d '{"line":"abcd"}' http://172.23.0.9:8080/logs
201 - Log entries created.

# oc logs frontend-1-pklzq
2017-09-22T14:48:19.780+0000 Debug ▶ DEBU 001 [aaa]
2017-09-22T14:56:46.245+0000 Info ▶ INFO 002 [abcd]

# oc exec -n logging $POD -- curl --connect-timeout 2 -s -k --cert /etc/elasticsearch/secret/admin-cert --key /etc/elasticsearch/secret/admin-key https://logging-es:9200/_cat/indices?v
health status index                                                           pri rep docs.count docs.deleted store.size pri.store.size 
...
green  open   project.aaa.dbe525d1-9fa4-11e7-8793-027497ece8ac.2017.09.22       1   0          2            0       48kb           48kb 

```

## docker logs

### journald
Check [docker config for logging](https://docs.docker.com/engine/admin/logging/overview/#supported-logging-drivers):

```sh
# docker info | grep "Logging Driver"
Logging Driver: journald
```

In this case, this is <code>journald</code>.

[Retrieve the container logs](https://docs.docker.com/engine/admin/logging/journald/#retrieving-log-messages-with-journalctl)

```sh
# ssh ip-172-31-5-234.us-west-2.compute.internal
# docker ps | grep frontend-1-pklzq
4ab7d665e371        docker.io/hongkailiu/svt-go@sha256:6b9d8e51c68409d58e925ef4a04b3bb5411a9cd63e360627a7a43ad82c87d691                                   "./svt/svt http"         18 minutes ago      Up 18 minutes                           k8s_helloworldfrontend-1-pklzq_aaa_0d355469-9fa5-11e7-8793-027497ece8ac_0
f8ffed0ff298        registry.ops.openshift.com/openshift3/ose-pod:v3.7.0-0.126.4                                                                          "/usr/bin/pod"           18 minutes ago      Up 18 minutes                           k8s_POD_frontend-1-pklzq_aaa_0d355469-9fa5-11e7-8793-027497ece8ac_0

# #journalctl -b CONTAINER_NAME=<CONTAINER_NAME>

# journalctl -b CONTAINER_NAME=k8s_helloworld_frontend-1-pklzq_aaa_0d355469-9fa5-11e7-8793-027497ece8ac_0
-- Logs begin at Fri 2017-09-22 12:38:05 UTC, end at Fri 2017-09-22 15:04:47 UTC. --
Sep 22 14:48:19 ip-172-31-5-234.us-west-2.compute.internal dockerd-current[11733]: 2017-09-22T14:48:19.780+0000 Debug ▶ DEBU 00
Sep 22 14:56:46 ip-172-31-5-234.us-west-2.compute.internal dockerd-current[11733]: 2017-09-22T14:56:46.245+0000 Info ▶ INFO 002
```

### json-file

Change _docker daemon_ options by <code>/etc/docker/daemon.json</code>:

```sh
# cat /etc/docker/daemon.json 
{
"log-driver": "json-file"
}
# systemctl restart docker
# docker info | grep "Logging Driver"
Logging Driver: json-file
```
Log file locations:

```sh
# ls /var/lib/docker/containers/1962de2f6e3f645fa20e21c107763f71d7f0db1fce9e82021b79a68d043be35a/1962de2f6e3f645fa20e21c107763f71d7f0db1fce9e82021b79a68d043be35a-json.log
```

*Note* that if the logging driver of docker is changed. Logging stack needs to be reinstalled in order for _fluentd_ to redecide where to pick logs up. 


### json-file without mux


## Logging test tool
Check [this](https://github.com/openshift/svt/blob/master/openshift_scalability/content/logtest/ocp_logtest-README.md)
out.

## Reference

[1]. https://medium.com/@yoanis_gil/logging-with-docker-part-1-b23ef1443aac

[2]. http://www.projectatomic.io/blog/2015/04/logging-docker-container-output-to-journald/

[3]. https://www.loggly.com/ultimate-guide/using-journalctl/
