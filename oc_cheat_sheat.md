
## [journalctl](https://www.loggly.com/ultimate-guide/using-journalctl/)

```sh
# systemctl list-unit-files --all
# journalctl -u atomic-openshift-node.service
```

## authentication

```sh
$ oc whoami
$ oc login -u redhat -p <password>
$ oc login -u system:admin
$ oc adm policy add-cluster-role-to-user cluster-admin redhat
###
# htpasswd -b /etc/origin/master/htpasswd redhat <password>
```

## check on which node a pod runs

```sh
$ oc get pods -o wide
```

## delete project

```sh
$ oc delete project -l purpose=test
$ for i in {110..119}; do oc delete project "proj$i"; done
$ oc get projects | cut -f1 -d" " | grep -E "proj[0-9]+" | while read i; do oc delete project $i; done
```

## delete build

```
$ oc get builds --all-namespaces | grep Fail | grep -E "proj[0-9]+" | while read i; do awk '{system("oc delete build -n " $1 "  " $2)}'; done
```

## Clean docker images

```sh
# docker images | grep -E "proj[0-9]+" | awk '{print $3}' | while read i; do docker rmi $i; done
```

## Show sha of docker images

```sh
# docker images --digests
```

## Extend docker fs

```sh
# oc get nodes --no-headers | cut -f1 -d" " | while read i; do ssh -n "$i" 'xfs_growfs -d /var/lib/docker/overlay2'; done
```

## remove a computing node from cluster

  - on the computing node

  ```sh
  # systemctl stop atomic-openshift-node
  # systemctl disable atomic-openshift-node
  ```

  - on a master node

  ```sh
  # oc get nodes
  # oc delete node <node_name>
  ```
## restart ectd

```sh
# systemctl restart etcd
```

## get log from container in pod

```sh
$ oc logs <pod_name> -c kibana --loglevel=10
```

## get all objects of a project

```sh
$ oc project <project_name>
$ oc get all
```

## new-app

```sh
# oc new-app https://github.com/dev-tool-index/calculator-monitor-docker
```

## config and restart master

```sh
# vi /etc/origin/master/master-config.yaml
# systemctl restart atomic-openshift-master
```

## Config maximal volumes

### 3.6 and HA

```sh
# #on all masters:
# vi /etc/sysconfig/atomic-openshift-master-controllers
...
KUBE_MAX_PD_VOLS=60

# systemctl daemon-reload
# systemctl restart atomic-openshift-master-controllers
```

## yum

```
### list all pbench-fio versions
# yum --showduplicates list pbench-fio
### install a specific version
yum install pbench-fio-2.14
```


## cns

```sh
# clean up pv left-overs
# oc get pv | grep fio | awk '{print $1}' | while read i; do oc delete pv $i; done
# oc get pv --no-headers | grep -v etcd | awk '{print $1}' | while read i; do oc delete pv $i; done

# systemctl stop lvm2-lvmetad.service 
# systemctl stop lvm2-lvmetad.socket 
# systemctl start lvm2-lvmetad.socket 
# systemctl start lvm2-lvmetad.service 

# rm -rf /var/lib/heketi/
# oc get node -l glusterfs=storage-host --no-headers | grep compute | awk '{print $1}' | while read line; do ssh -n "${line}" 'rm -rf /var/lib/heketi/'; done

# oc get pod -n glusterfs -o yaml | grep "image:" | sort -u

# oc get node --show-labels | grep gluster | awk '{print $1}' | while read i; do oc adm manage-node $i --schedulable=false; done
# oc get node | grep cns | awk '{print $1}' | while read i; do oc adm manage-node $i --schedulable=false; done

# oc get pod -n glusterfs | grep glusterfs-storage | awk '{print $1}' | while read line; do oc exec -n glusterfs "$line" -- systemctl status gluster-blockd.service; done | grep Active
# oc get pod -n glusterfs | grep glusterfs-storage | awk '{print $1}' | while read line; do oc exec -n glusterfs "$line" -- systemctl status glusterd.service; done | grep Active
```

### ocp node role

```sh
$ oc label node hongkliu-310-bbb-nrr-1 node-role.kubernetes.io/infra=true
```

## Jenkins image is missing for master branch

```sh
$ oc get imagestreamtag -n openshift | grep jenkins
$ oc get imagestream -n openshift | grep jenkins
$ oc edit imagestream -n openshift jenkins
$ oc import-image -n openshift jenkins
```
## oc explain

```sh
$ oc explain DeploymentConfig.spec.template.spec
```

## pbench-fio leftover

```sh
# ps -ef | grep pbench | grep fio | awk '{print $2}' | while read i; do kill "$i"; done
# oc get node --no-headers | awk '{print $1}' | while read line; do ssh -n "${line}" 'ps -ef | grep pbench'; done
# oc get node --no-headers | awk '{print $1}' | while read line; do ssh -n "${line}" 'pbench-clear-results'; done

```
