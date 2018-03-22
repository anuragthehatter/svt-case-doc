# CNS: Usability Test

## heketi/block-provisioner pod can survive of node-draining

This case covers [bz-1548322](https://bugzilla.redhat.com/show_bug.cgi?id=1548322).

Label 2 nodes with `heketi=heketi` and `gfsbp=gfsbp` which is the node selector for
`dc/heketi-storage`.

Then create 300 pods with glusterfs PVCs which usually takes about 30 mins.
During the creation, drain the node where heketi is running.

Expected result: 300 pods in running state.

Extra work on the case: no

### Steps

Install CNS with 1000g and 3 replicas.

Label 2 nodes with `heketi=heketi`:

```sh
# oc label node ip-172-31-28-169.us-west-2.compute.internal heketi=heketi gfsbp=gfsbp
# oc label node ip-172-31-33-215.us-west-2.compute.internal heketi=heketi gfsbp=gfsbp

# oc patch -n glusterfs deploymentconfigs/glusterblock-storage-provisioner-dc --patch '{"spec": {"template": {"spec": {"nodeSelector": {"gfsbp": "gfsbp"}}}}}'
# oc patch -n glusterfs deploymentconfigs/heketi-storage --patch '{"spec": {"template": {"spec": {"nodeSelector": {"heketi": "heketi"}}}}}'
```

Start to create 200 pods with glusterfs-storage PVCs as described [here](glusterFS_stress.md#run-test)

Before the above finished (about 30 pod are running), do the following in anther terminal:

```sh
# heketi_node=$(oc get pod -n glusterfs -o wide | grep heketi | awk '{print $7}')
# oc label node ${heketi_node} heketi-
# oc rollout latest heketi-storage -n glusterfs
```

Or delete the heketi pod directly:

```sh
# oc delete pod -n glusterfs "$(oc get pod -n glusterfs | grep heketi | awk '{print $1}')"
```

There should be 200 pods in Running states.

delete block-provisioner pod:

```sh
# oc delete pod -n glusterfs "$(oc get pod -n glusterfs | grep "glusterblock-storage-provisioner" | awk '{print $1}')"
```


## No impact on App. pods when glusterfs pods get restarted

Similar to [bz 1555063](https://bugzilla.redhat.com/show_bug.cgi?id=1555063).

To know which glusterfs pod to kill:

```
### Run it in the fio pod:
sh-4.2$ grep gluster /proc/mounts
172.31.3.148:vol_10cfce21dbcac9cc5620ab378d76e017 /mnt/pvcmount fuse.glusterfs rw,relatime,user_id=0,group_id=0,default_permissions,allow_other,max_read=131072 0 0
```

100 pods on 2 compute nodes with glusterfs PVCs and the pods write logs onto files on PVCs.

Restart glusterfs pod one after another with a reasonable interval: drain
node or delete pod or remove label `glusterfs=storage-host` on the
glusterfs node which is the node selector for ds/glusterfs-storage.

Expected result: all logs are fine with correct content.

Extra work on the case: Need to implement the logic with Mike's logging
tool or start from scratch.

### Steps

```sh
# cd svt/openshift_scalability
# 1 projects, 1 template
# vi content/fio/fio-parameters.yaml
...
        file: ./content/fio/fio-template3.json
        parameters:
          - STORAGE_CLASS: "glusterfs-storage" # this is name of storage class to use
          - STORAGE_SIZE: "3Gi" # this is size of PVC mounted inside pod
          - MOUNT_PATH: "/mnt/pvcmount"
          - DOCKER_IMAGE: "docker.io/hongkailiu/ocp-logtest:20180307"
          - INITIAL_FLAGS: "-o /mnt/pvcmount/test.log --line-length 1024 --word-length 7 --rate 30000 --time 0 --fixed-line --num-lines 900000\n"

tuningsets:
  - name: default
    templates:
      stepping:
        stepsize: 5
        pause: 0 ms
      rate_limit:
        delay: 0 ms
...

# python -u cluster-loader.py -v -f content/fio/fio-parameters.yaml -p 1

# Kill the glusterfs-storage one after another but always ensure that the previous kill one is recreated and ready
# oc get pod -n glusterfs | grep "glusterfs-storage"
glusterfs-storage-8jpbt                       1/1       Running   0          2h
glusterfs-storage-bl9kz                       1/1       Running   0          2h
glusterfs-storage-dvl7c                       1/1       Running   0          2h


# oc delete pod -n glusterfs glusterfs-storage-8jpbt
pod "glusterfs-storage-8jpbt" deleted
# oc delete pod -n glusterfs glusterfs-storage-bl9kz
pod "glusterfs-storage-bl9kz" deleted
# oc delete pod -n glusterfs glusterfs-storage-dvl7c
pod "glusterfs-storage-dvl7c" deleted

### Or:
# oc label node ${glusterfs_node} glusterfs-
# oc label node ${glusterfs_node} glusterfs=storage-host

### Write the logs in the way like the bz
sh-4.2$ while true; do echo $(date) | tee -a /mnt/pvcmount/ttt.log; sleep 1; done 
```

Expected result: 900000 line of logs is written onto the file after 30 mins.

```sh
# oc get pod --all-namespaces | grep fio | grep Running | awk '{print $2}' | while read i; do oc exec -n fioatest0 $i -- wc -l /mnt/pvcmount/test.log; done
# oc get pod --all-namespaces | grep fio | awk '{print $2}' | while read i; do oc exec -n fioatest0 $i -- tail -n 1 /mnt/pvcmount/test.log ; done | awk '{print $10}'
# oc get pod --all-namespaces | grep fio | grep Running | awk '{print $2}' | while read i; do oc exec -n fioatest0 $i -- rm -f /mnt/pvcmount/test.log; done
oc delete pod -n fioatest0 --all

$ oc rsh -n fioctest0 fio-0-zpkhp
sh-4.2$ tail /mnt/pvcmount/test.log 
sh-4.2$ cat /mnt/pvcmount/test.log | wc -l 
### If client quorum is not met (2 gluster pods were killed in our setting), then we can see this:
### ref: https://docs.gluster.org/en/latest/Administrator%20Guide/Split%20brain%20and%20ways%20to%20deal%20with%20it/
...
Thu Mar 22 14:58:38 UTC 2018
tee: /mnt/pvcmount/ttt.log: Read-only file system

```

Observations:

* to provision new PVCs, all 3 glusterfs pods needs to be running
* for writing on existing PVCs, at least 2 out of 3 glusterfs pods needs to be running.

## PV resizing
Doc: https://www.humblec.com/glusterfs-dynamic-provisioner-online-resizing-of-glusterfs-pvs-in-kubernetes-v-1-8/
