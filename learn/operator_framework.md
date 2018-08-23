# Operator

https://www.redhat.com/en/blog/introducing-operator-framework-building-apps-kubernetes?sc_cid=701f2000000tucqAAA&src=fridayfive4email

https://thenewstack.io/coreos-says-red-hat-will-help-introduce-openshift-to-operators/

## Openshift monitoring

[Installation](https://github.com/openshift/openshift-ansible/tree/master/playbooks/openshift-monitoring)

```sh
$ ansible-playbook -i aaa/ openshift-ansible/playbooks/openshift-monitoring/config.yml

# oc project openshift-monitoring
Now using project "openshift-monitoring" on server "https://ip-172-31-20-26.us-west-2.compute.internal:8443".
root@ip-172-31-20-26: ~ # oc get all
NAME                                               READY     STATUS    RESTARTS   AGE
pod/alertmanager-main-0                            3/3       Running   0          2m
pod/alertmanager-main-1                            3/3       Running   0          2m
pod/alertmanager-main-2                            3/3       Running   0          1m
pod/cluster-monitoring-operator-7f956789fc-gr4x5   1/1       Running   0          5m
pod/grafana-6bd78bcd6d-hbfpq                       2/2       Running   0          4m
pod/kube-state-metrics-58d4dd6b44-9lnf7            3/3       Running   0          1m
pod/prometheus-k8s-0                               4/4       Running   1          3m
pod/prometheus-k8s-1                               4/4       Running   1          2m
pod/prometheus-operator-7fff695789-rcz4m           1/1       Running   0          5m

NAME                                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
service/alertmanager-main             ClusterIP   172.24.248.184   <none>        9094/TCP            2m
service/alertmanager-operated         ClusterIP   None             <none>        9093/TCP,6783/TCP   2m
service/cluster-monitoring-operator   ClusterIP   None             <none>        8080/TCP            4m
service/grafana                       ClusterIP   172.26.74.14     <none>        3000/TCP            4m
service/kube-state-metrics            ClusterIP   None             <none>        8443/TCP,9443/TCP   1m
service/node-exporter                 ClusterIP   None             <none>        9100/TCP            1m
service/prometheus-k8s                ClusterIP   172.25.65.6      <none>        9091/TCP            3m
service/prometheus-operated           ClusterIP   None             <none>        9090/TCP            3m
service/prometheus-operator           ClusterIP   None             <none>        8080/TCP            5m

NAME                           DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
daemonset.apps/node-exporter   0         0         0         0            0           beta.kubernetes.io/os=linux   1m

NAME                                          DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cluster-monitoring-operator   1         1         1            1           5m
deployment.apps/grafana                       1         1         1            1           4m
deployment.apps/kube-state-metrics            1         1         1            1           1m
deployment.apps/prometheus-operator           1         1         1            1           5m

NAME                                                     DESIRED   CURRENT   READY     AGE
replicaset.apps/cluster-monitoring-operator-7f956789fc   1         1         1         5m
replicaset.apps/grafana-6bd78bcd6d                       1         1         1         4m
replicaset.apps/kube-state-metrics-58d4dd6b44            1         1         1         1m
replicaset.apps/prometheus-operator-7fff695789           1         1         1         5m

NAME                                 DESIRED   CURRENT   AGE
statefulset.apps/alertmanager-main   3         3         2m
statefulset.apps/prometheus-k8s      2         2         3m

NAME                                         HOST/PORT                                                             PATH      SERVICES            PORT      TERMINATION   WILDCARD
route.route.openshift.io/alertmanager-main   alertmanager-main-openshift-monitoring.apps.0822-eoo.qe.rhcloud.com             alertmanager-main   web       reencrypt     None
route.route.openshift.io/grafana             grafana-openshift-monitoring.apps.0822-eoo.qe.rhcloud.com                       grafana             https     reencrypt     None
route.route.openshift.io/prometheus-k8s      prometheus-k8s-openshift-monitoring.apps.0822-eoo.qe.rhcloud.com                prometheus-k8s      web       reencrypt     None

# oc get pvc
NAME                                       STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
alertmanager-main-db-alertmanager-main-0   Bound     pvc-dff44949-a60e-11e8-8dc1-026cee33ed60   2Gi        RWO            gp2            2m
alertmanager-main-db-alertmanager-main-1   Bound     pvc-e928b593-a60e-11e8-8dc1-026cee33ed60   2Gi        RWO            gp2            2m
alertmanager-main-db-alertmanager-main-2   Bound     pvc-f6fd1bb6-a60e-11e8-8dc1-026cee33ed60   2Gi        RWO            gp2            2m
prometheus-k8s-db-prometheus-k8s-0         Bound     pvc-af66d092-a60e-11e8-8dc1-026cee33ed60   50Gi       RWO            gp2            4m
prometheus-k8s-db-prometheus-k8s-1         Bound     pvc-ca971a98-a60e-11e8-8dc1-026cee33ed60   50Gi       RWO            gp2            3m

```

Uninstall:

```sh
# vi aaa/2.file
[OSEv3:vars]
...
openshift_cluster_monitoring_operator_install=false
### then, rerun the above playbook
```

Control the PVC size and sc:
```
###https://github.com/openshift/openshift-ansible/blob/master/roles/openshift_cluster_monitoring_operator/defaults/main.yml#L46-L47
# vi aaa/2.file
[OSEv3:vars]
...
openshift_cluster_monitoring_operator_prometheus_storage_capacity=60Gi
openshift_cluster_monitoring_operator_alertmanager_storage_capacity=6Gi

###No support yet to control the size of PVC via variables
###But we can change the src to achieve that
~/openshift-ansible # git diff
diff --git a/roles/openshift_cluster_monitoring_operator/templates/cluster-monitoring-operator-config.j2 b/roles/openshift_clu
index dbb3120..dc72650 100644
--- a/roles/openshift_cluster_monitoring_operator/templates/cluster-monitoring-operator-config.j2
+++ b/roles/openshift_cluster_monitoring_operator/templates/cluster-monitoring-operator-config.j2
@@ -27,6 +27,7 @@ data:
         cluster: {{ openshift_cluster_monitoring_operator_cluster_id }}
       volumeClaimTemplate:
         spec:
+          storageClassName: {{ openshift_cluster_monitoring_operator_prometheus_storage_class_name }}
           resources:
             requests:
               storage: {{ openshift_cluster_monitoring_operator_prometheus_storage_capacity }}
@@ -40,6 +41,7 @@ data:
 {% endif %}
       volumeClaimTemplate:
         spec:
+          storageClassName: {{ openshift_cluster_monitoring_operator_alertmanager_storage_class_name }}
           resources:
             requests:
               storage: {{ openshift_cluster_monitoring_operator_alertmanager_storage_capacity }}

# vi aaa/2.file
[OSEv3:vars]
...
openshift_cluster_monitoring_operator_prometheus_storage_class_name=glusterfs-storage
openshift_cluster_monitoring_operator_alertmanager_storage_class_name=glusterfs-storage

### Then rerun the playbook

# oc get pvc -n openshift-monitoring
NAME                                       STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS        AGE
alertmanager-main-db-alertmanager-main-0   Bound     pvc-b83923d3-a61d-11e8-8dc1-026cee33ed60   6Gi        RWO            glusterfs-storage   1m
alertmanager-main-db-alertmanager-main-1   Bound     pvc-be637c95-a61d-11e8-8dc1-026cee33ed60   6Gi        RWO            glusterfs-storage   58s
alertmanager-main-db-alertmanager-main-2   Bound     pvc-c4e2ea1b-a61d-11e8-8dc1-026cee33ed60   6Gi        RWO            glusterfs-storage   47s
prometheus-k8s-db-prometheus-k8s-0         Bound     pvc-a586b9da-a61d-11e8-8dc1-026cee33ed60   60Gi       RWO            glusterfs-storage   1m
prometheus-k8s-db-prometheus-k8s-1         Bound     pvc-adc373fe-a61d-11e8-8dc1-026cee33ed60   60Gi       RWO            glusterfs-storage   1m

```

## Openshift metering

[Installation](https://github.com/openshift/openshift-ansible/tree/master/playbooks/openshift-metering)

```sh
$ ansible-playbook -i aaa/ openshift-ansible/playbooks/openshift-metering/config.yml
# oc project openshift-metering
# oc get all
NAME                                      READY     STATUS    RESTARTS   AGE
pod/hdfs-datanode-0                       1/1       Running   0          4m
pod/hdfs-namenode-0                       1/1       Running   0          4m
pod/hive-metastore-0                      1/1       Running   0          4m
pod/hive-server-0                         1/1       Running   0          4m
pod/metering-operator-df67bb6cb-4qt5t     2/2       Running   0          6m
pod/presto-coordinator-78b49dfb5c-pgs79   1/1       Running   0          4m
pod/reporting-operator-5f7fbc9559-vjlgw   1/1       Running   0          4m

NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)               AGE
service/hdfs-datanode                ClusterIP   None             <none>        50010/TCP             4m
service/hdfs-datanode-web            ClusterIP   172.27.204.227   <none>        50075/TCP             4m
service/hdfs-namenode                ClusterIP   None             <none>        8020/TCP              4m
service/hdfs-namenode-proxy          ClusterIP   172.25.34.80     <none>        8020/TCP              4m
service/hdfs-namenode-web            ClusterIP   172.25.69.3      <none>        50070/TCP             4m
service/hive-metastore               ClusterIP   172.25.249.119   <none>        9083/TCP              4m
service/hive-server                  ClusterIP   172.27.221.255   <none>        10000/TCP,10002/TCP   4m
service/presto                       ClusterIP   172.25.142.126   <none>        8080/TCP,8082/TCP     4m
service/reporting-operator           ClusterIP   172.26.55.47     <none>        8080/TCP              4m
service/reporting-operator-metrics   ClusterIP   172.26.121.101   <none>        8082/TCP              4m

NAME                                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/metering-operator    1         1         1            1           6m
deployment.apps/presto-coordinator   1         1         1            1           4m
deployment.apps/presto-worker        0         0         0            0           4m
deployment.apps/reporting-operator   1         1         1            1           4m

NAME                                            DESIRED   CURRENT   READY     AGE
replicaset.apps/metering-operator-df67bb6cb     1         1         1         6m
replicaset.apps/presto-coordinator-78b49dfb5c   1         1         1         4m
replicaset.apps/presto-worker-7f95dcbd64        0         0         0         4m
replicaset.apps/reporting-operator-5f7fbc9559   1         1         1         4m

NAME                              DESIRED   CURRENT   AGE
statefulset.apps/hdfs-datanode    1         1         4m
statefulset.apps/hdfs-namenode    1         1         4m
statefulset.apps/hive-metastore   1         1         4m
statefulset.apps/hive-server      1         1         4m

# oc get pvc
NAME                                 STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
hdfs-datanode-data-hdfs-datanode-0   Bound     pvc-d73d0e3a-a61f-11e8-8dc1-026cee33ed60   5Gi        RWO            gp2            4m
hdfs-namenode-data-hdfs-namenode-0   Bound     pvc-d74c9d88-a61f-11e8-8dc1-026cee33ed60   5Gi        RWO            gp2            4m
hive-metastore-db-data               Bound     pvc-d71e7c9e-a61f-11e8-8dc1-026cee33ed60   5Gi        RWO            gp2            4m

```

Uninstall:

```sh
$ ansible-playbook -i aaa/ openshift-ansible/playbooks/openshift-metering/uninstall.yml
```


Control the PVC size and sc:

```sh
### the default config is here:
~/openshift-ansible # grep -irn "metering.openshift.io/v1alpha1" .
./roles/openshift_metering/files/operator/metering.yaml:1:apiVersion: metering.openshift.io/v1alpha1
### we can replace it by setting var openshift_metering_config
### https://github.com/openshift/openshift-ansible/tree/master/roles/openshift_metering
### the syntax: https://github.com/operator-framework/operator-metering/blob/master/manifests/metering-config/custom-storageclass-values.yaml

# mkdir /tmp/openshift_metering_config
# vi /tmp/openshift_metering_config/metering.yaml
### content is here: https://raw.githubusercontent.com/hongkailiu/svt-case-doc/master/files/metering.yaml

### then rerun the playbook with this var file
# ansible-playbook -i aaa/ openshift-ansible/playbooks/openshift-metering/config.yml --extra-vars "@/tmp/openshift_metering_config/metering.yaml"

# oc get pvc
NAME                                 STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS        AGE
hdfs-datanode-data-hdfs-datanode-0   Bound     pvc-dc0220c1-a6d0-11e8-b49c-0279bbe13b54   15Gi       RWO            glusterfs-storage   2m
hdfs-namenode-data-hdfs-namenode-0   Bound     pvc-dc11ccb9-a6d0-11e8-b49c-0279bbe13b54   15Gi       RWO            glusterfs-storage   2m
hive-metastore-db-data               Bound     pvc-dbd5f44c-a6d0-11e8-b49c-0279bbe13b54   15Gi       RWO            glusterfs-storage   2m

```

pvc type/size for hdfs does NOT work. [bz-1620222](https://bugzilla.redhat.com/show_bug.cgi?id=1620222).
