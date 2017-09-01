# GlusterFS test

## PVC only

### Run test

```sh
# oc get storageclass 
NAME                TYPE
glusterfs-storage   kubernetes.io/glusterfs   
gp2 (default)       kubernetes.io/aws-ebs

# cd svt/openshift_scalability
# #change the storage class name in content/pvc-templates/pvc-parameters.yaml
# vi content/pvc-templates/pvc-parameters.yaml
...
parameters:
          - STORAGE_CLASS: "glusterfs-storage"
...

# python -u cluster-loader.py -v -f content/pvc-templates/pvc-parameters.yaml
```

### Check results

```sh
# oc get pvc -n pvcproject0 
NAME            STATUS    VOLUME                                     CAPACITY   ACCESSMODES   STORAGECLASS        AGE
pvcoaq5xs0m0y   Bound     pvc-5e5bb0bf-88f7-11e7-9936-027c10993e42   1Gi        RWO           glusterfs-storage   15s
# oc get pv
NAME                                       CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS    CLAIM                       STORAGECLASS        REASON    AGE
pvc-5e5bb0bf-88f7-11e7-9936-027c10993e42   1Gi        RWO           Delete          Bound     pvcproject0/pvcoaq5xs0m0y   glusterfs-storage             23s
```

## Pods with pvc

### Run test

```sh
# vi content/fio/fio-parameters.yaml
...
        parameters:
          - STORAGE_CLASS: "glusterfs-storage" # this is name of storage class to use
          - STORAGE_SIZE: "3Gi" # this is size of PVC mounted inside pod
          - MOUNT_PATH: "/mydata"
          - DOCKER_IMAGE: "openshift/hello-openshift"

...

# python -u cluster-loader.py -v -f content/fio/fio-parameters.yaml
```

### Check results

```sh
# oc get pvc -n fiotest0 
NAME            STATUS    VOLUME                                     CAPACITY   ACCESSMODES   STORAGECLASS        AGE
pvc2gtuc3bg6t   Bound     pvc-ba6a0669-88f6-11e7-9936-027c10993e42   3Gi        RWO           glusterfs-storage   1m
pvcftorwcfrvs   Bound     pvc-bab33521-88f6-11e7-9936-027c10993e42   3Gi        RWO           glusterfs-storage   1m
pvco64n6v4aqo   Bound     pvc-ba1e917b-88f6-11e7-9936-027c10993e42   3Gi        RWO           glusterfs-storage   1m

# oc get pod -n fiotest0
NAME            READY     STATUS    RESTARTS   AGE
fio-pod-2km4f   1/1       Running   0          1m
fio-pod-b6222   1/1       Running   0          1m
fio-pod-tb559   1/1       Running   0          1m

# oc volumes pod -n fiotest0 fio-pod-2km4f
pods/fio-pod-2km4f
  pvc/pvcftorwcfrvs (allocated 3GiB) as persistentvolume
    mounted at /mydata
  secret/default-token-zswxf as default-token-zswxf
    mounted at /var/run/secrets/kubernetes.io/serviceaccount
```


## Put together
_Note_ that [heketivolumemonitor.py](https://github.com/ekuric/openshift/blob/master/cns/heketivolumemonitor.py) and [check_pvpods.py](https://github.com/ekuric/openshift/blob/master/cns/check_pods/check_pvpods.py) have not been pushed to SVT repo yet.

```sh
# #change the fio-parameters.yaml to create 8 pvc (via 8 templates in 1 project)
# python -u cluster-loader.py -v -f content/fio/fio-parameters.yaml
# #get secret https://github.com/hongkailiu/svt-case-doc/blob/master/storage/glusterFS.md#heketi
# python ./cns/heketivolumemonitor.py --urllocation http://heketi-storage-glusterfs.0901-9aw.qe.rhcloud.com/volumes --projectname fiotest0 --port 80 --action create --secret 'ReSm+3X58fPATPIym9Z+wYwxj9Zeq/th+b0JRLuUZA4=' --numvol 8
# #token is oc client token
# python ./cns/check_pods/check_pvpods.py -H ec2-54-200-250-50.us-west-2.compute.amazonaws.com --token 1_PsL7BFOXiH3uft9srRra65UGC91fdKJOZV4CmQLuI --check_all -ns fiotest0

```

The result files are generated _heketivol.txt_ and _[pod|pv|pvc]*.[json|csv]_


TODO Draw graph
```sh
$ sudo yum install gcc gcc-c++ libgcc python=devel numpy
$ pip install pandas prettytable matplotlib
```
