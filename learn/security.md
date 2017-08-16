# Security

## Doc

* [linux kernel security](https://www.linux.com/learn/overview-linux-kernel-security-features), [selinux](selinux.md)
* [Dan Walsh's Blog](http://danwalsh.livejournal.com/76358.html), [security@docker](https://docs.docker.com/engine/security/security/), [security@container](https://opensource.com/business/14/7/docker-security-selinux)
* [sc@k8s](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
* [secure-k8s](https://blog.openshift.com/securing-kubernetes/)
* oc: [authentication](https://docs.openshift.com/container-platform/3.6/architecture/additional_concepts/authentication.html) and [authorization](https://docs.openshift.com/container-platform/3.6/architecture/additional_concepts/authorization.html), [scc](https://docs.openshift.org/latest/architecture/additional_concepts/authorization.html#security-context-constraints), [sa](https://docs.openshift.org/latest/dev_guide/service_accounts.html), [configure-sa](https://docs.openshift.com/container-platform/3.6/admin_guide/service_accounts.html), [manage-policy](https://docs.openshift.com/container-platform/3.6/admin_guide/manage_authorization_policy.html)

In order to solve the tasks listed below, read doc:
* [sa&scc-blog](https://blog.openshift.com/understanding-service-accounts-sccs/)


## Concept list
user, group, rule, role, binding, policy, scc, sa.

## SCC

SCC defines what a pod can do. Those 7 SCCs come out of the box (_restricted_ is default):

```sh
# oc get scc
NAME               PRIV      CAPS      SELINUX     RUNASUSER          FSGROUP     SUPGROUP    PRIORITY   READONLYROOTFS   VOLUMES
anyuid             false     []        MustRunAs   RunAsAny           RunAsAny    RunAsAny    10         false            [configMap downwardAPI emptyDir persistentVolumeClaim projected secret]
hostaccess         false     []        MustRunAs   MustRunAsRange     MustRunAs   RunAsAny    <none>     false            [configMap downwardAPI emptyDir hostPath persistentVolumeClaim projected secret]
hostmount-anyuid   false     []        MustRunAs   RunAsAny           RunAsAny    RunAsAny    <none>     false            [configMap downwardAPI emptyDir hostPath nfs persistentVolumeClaim projected secret]
hostnetwork        false     []        MustRunAs   MustRunAsRange     MustRunAs   MustRunAs   <none>     false            [configMap downwardAPI emptyDir persistentVolumeClaim projected secret]
nonroot            false     []        MustRunAs   MustRunAsNonRoot   RunAsAny    RunAsAny    <none>     false            [configMap downwardAPI emptyDir persistentVolumeClaim projected secret]
privileged         true      [*]       RunAsAny    RunAsAny           RunAsAny    RunAsAny    <none>     false            [*]
restricted         false     []        MustRunAs   MustRunAsRange     MustRunAs   RunAsAny    <none>     false            [configMap downwardAPI emptyDir persistentVolumeClaim projected secret]
```

## SA

SA gives a way to impersonate. Those 3 SCCs come out of the box: builder/deployer to run build/deploy pods, default to run other pods, or do any object operation.

```sh
# oc get sa
NAME       SECRETS   AGE
builder    2         14h
default    3         14h
deployer   2         14h
registry   4         14h
router     2         14h
```

## User in the container

Use [pvc_ebs.yaml](../files/pvc_ebs.yaml) and [rs_bbb.yaml](../files/rs_test.yaml).

```sh
# #Do NOT make redhat as admin of cluster
# oc login -u redhat -p <secret>
# oc create -f pvc_ebs.yaml
# oc get pvc
NAME      STATUS    VOLUME                                     CAPACITY   ACCESSMODES   STORAGECLASS   AGE
pvc-ebs   Bound     pvc-06e77f8a-82af-11e7-aeaf-02c36d111da6   1Gi        RWO           gp2            11m
# oc new-project bbb
# create -f rs_bbb.yaml
# oc get pod
NAME               READY     STATUS    RESTARTS   AGE
frontend-1-6cf7h   1/1       Running   0          11m
# oc exec frontend-1-6cf7h -- id
uid=1000080000 gid=0(root) groups=0(root),1000080000
# oc exec frontend-1-6cf7h -- whoami
whoami: cannot find name for user ID 1000080000
command terminated with exit code 1
# oc exec frontend-1-6cf7h -- touch /myapp/aaa.txt
touch: cannot touch '/myapp/aaa.txt': Permission denied
command terminated with exit code 1
# oc exec frontend-1-6cf7h -- ls -Z /myapp
drwxr-xr-x. 1000 1000 system_u:object_r:svirt_sandbox_file_t:s0:c4,c9 svt
-rw-r--r--. root root system_u:object_r:svirt_sandbox_file_t:s0:c4,c9 svt-0.0.1-Linux-x86_64.tar.gz
# oc exec frontend-1-6cf7h -- touch /mydata/aaa.txt
# oc exec frontend-1-6cf7h -- ls -Z /mydata/
-rw-r--r--. 1000080000 1000080000 system_u:object_r:svirt_sandbox_file_t:s0:c4,c9 aaa.txt
drwxrwS---. root       1000080000 system_u:object_r:svirt_sandbox_file_t:s0:c4,c9 lost+found
```

Observation:

* _permission denied_ on <code>/myapp</code>: This is different from <code>docker run</code> command.
* We can write on the mounted volume. This is cool. At least we get a folder we can write.

As pointed out by [Graham Dumpleton](http://blog.dscpl.com.au/2015/12/random-user-ids-when-running-docker.html), the user in docker container is a random UID, not root.


## Task A: Why cannot Jenkens run?
Use [pod_jenkins.yaml](../files/pod_jenkins.yaml)

```sh
# oc create -f pod_jenkins.yaml
# oc logs web
touch: cannot touch '/var/jenkins_home/copy_reference_file.log': Permission denied
Can not write to /var/jenkins_home/copy_reference_file.log. Wrong volume permissions?
```

Not we can explain why it did not work. Jenkins cannot write file on <code>jenkins_home</code>. Let us use volume to fix this problem:

Use [pod_jenkins_volume.yaml](../files/pod_jenkins_volume.yaml)

```sh
# oc create -f pod_jenkins_volume.yaml
# oc get pod
NAME      READY     STATUS    RESTARTS   AGE
web       1/1       Running   0          8m
```

## Task B: Why cannot we deploy Nginx pod?
Use [pod_nginx.yaml}(../files/pod_nginx.yaml)

```sh
# oc create -f pod_nginx.yaml
# oc get pod
NAME      READY     STATUS             RESTARTS   AGE
web       0/1       CrashLoopBackOff   6          9m
# oc logs web
2017/08/16 19:05:29 [warn] 1#1: the "user" directive makes sense only if the master process runs with super-user privileges, ignored in /etc/nginx/nginx.conf:2
nginx: [warn] the "user" directive makes sense only if the master process runs with super-user privileges, ignored in /etc/nginx/nginx.conf:2
2017/08/16 19:05:29 [emerg] 1#1: mkdir() "/var/cache/nginx/client_temp" failed (13: Permission denied)
nginx: [emerg] mkdir() "/var/cache/nginx/client_temp" failed (13: Permission denied)
```

Same here: It seems that nginx tries to create a folder <code>/var/cache/nginx/client_temp</code> where Permission IS denied.
The solution is the same. Let attach a volume:

Use [pod_nginx_volume.yaml}(../files/pod_nginx_volume.yaml)

```sh
# oc create -f pod_nginx_volume.yaml
# oc get pod
NAME      READY     STATUS             RESTARTS   AGE
web       0/1       CrashLoopBackOff   6          9m
# oc logs web
2017/08/16 20:13:02 [warn] 1#1: the "user" directive makes sense only if the master process runs with super-user privileges, ignored in /etc/nginx/nginx.conf:2
nginx: [warn] the "user" directive makes sense only if the master process runs with super-user privileges, ignored in /etc/nginx/nginx.conf:2
2017/08/16 20:13:02 [emerg] 1#1: bind() to 0.0.0.0:80 failed (13: Permission denied)
nginx: [emerg] bind() to 0.0.0.0:80 failed (13: Permission denied)
```

We need 80 port, which means root access?

Use [pod_nginx_volume_privileged.yaml](../files/pod_nginx_volume_privileged.yaml)

```sh
# oc create -f pod_nginx_volume_privileged.yaml
Error from server (Forbidden): error when creating "pod_nginx.yaml": pods "web" is forbidden: unable to validate against any security context constraint: [provider restricted: .spec.containers[0].securityContext.privileged: Invalid value: true: Privileged containers are not allowed]
```

Two options to workaround this:

1. Get an nginx running as non-root user, not using root access, to be more precise. Preferred by OC, may not by users. Run nginx with 8080 port?
2. Give root to pod where nginux is running. You need to what you are doing.

Grand permission via <code>scc</code>

Use [scc_super.yaml](../files/scc_super.yaml)

```sh
# only cluster-admin can manage scc
# oc config use-context default/ec2-54-244-1-118-us-west-2-compute-amazonaws-com:8443/system:admin
# oc create -f scc_aaa.yaml
# oc config use-context bbb/ip-172-31-37-36-us-west-2-compute-internal:8443/redhat
# oc create -f pod_nginx_volume_privileged.yaml
# oc get pod -o wide
NAME      READY     STATUS    RESTARTS   AGE       IP            NODE
web       1/1       Running   0          1m        172.20.0.23   ip-172-31-37-36.us-west-2.compute.internal
# curl -s -o /dev/null -w "%{http_code}" 172.20.0.23
200

```

TODO: Can we do better than <code>privileged: true</code>?

