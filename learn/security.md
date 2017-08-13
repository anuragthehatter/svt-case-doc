# Security

## Doc

* [linux kernel security](https://www.linux.com/learn/overview-linux-kernel-security-features), [selinux](selinux.md)
* [Dan Walsh's Blog](http://danwalsh.livejournal.com/76358.html), [security@docker](https://docs.docker.com/engine/security/security/), [security@container](https://opensource.com/business/14/7/docker-security-selinux)
* [sc@k8s](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
* [secure-k8s](https://blog.openshift.com/securing-kubernetes/)
* oc: [authentication](https://docs.openshift.com/container-platform/3.6/architecture/additional_concepts/authentication.html) and [authorization](https://docs.openshift.com/container-platform/3.6/architecture/additional_concepts/authorization.html)

In order to solve the tasks listed below, read doc:
* [sa@oc](https://docs.openshift.org/latest/dev_guide/service_accounts.html)
* [scc@oc](https://docs.openshift.org/latest/architecture/additional_concepts/authorization.html#security-context-constraints)
* [sa&scc-blog](https://blog.openshift.com/understanding-service-accounts-sccs/)

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
```sh
# oc get sa
NAME       SECRETS   AGE
builder    2         14h
default    3         14h
deployer   2         14h
registry   4         14h
router     2         14h
```

## Task A: Why cannot we deploy Nginx pod?



## Task B: Why cannot Jenkens pod have access to PVC?
