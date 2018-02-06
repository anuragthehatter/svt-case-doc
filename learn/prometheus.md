# Prometheus

## doc

* [prometheus.io](https://prometheus.io/)
* [prometheus@github](https://github.com/prometheus)


## [Get started](https://prometheus.io/docs/introduction/getting_started/)

Tested on Fedora26:

## Prometheus@oc

* [doc@origin](https://github.com/openshift/origin/tree/master/examples/prometheus): useful prometheus queries.
* [p@oc-blog](https://blog.openshift.com/tag/prometheus/)
* [gap-archetecture](https://blog.openshift.com/monitoring-openshift-three-tools/)

## Installation

* On a new cluster:

```sh
### the inv. file contains those vars:
openshift_hosted_prometheus_deploy=true
openshift_prometheus_image_prefix=registry.reg-aws.openshift.com:443/openshift3/
openshift_prometheus_image_version=v3.9
openshift_prometheus_proxy_image_prefix=registry.reg-aws.openshift.com:443/openshift3/
openshift_prometheus_proxy_image_version=v3.9
openshift_prometheus_alertmanager_image_prefix=registry.reg-aws.openshift.com:443/openshift3/
openshift_prometheus_alertmanager_image_version=v3.9
openshift_prometheus_alertbuffer_image_prefix=registry.reg-aws.openshift.com:443/openshift3/
openshift_prometheus_alertbuffer_image_version=v3.9

$ ansible-playbook -i /tmp/2.file openshift-ansible/playbooks/deploy_cluster.yml
```

* On an existing cluster:

```sh
### the inv. file contains same vars as above.
$ ansible-playbook -i /tmp/2.file openshift-ansible/playbooks/openshift-prometheus/config.yml
```

Example of inv. file is [here](https://github.com/openshift/openshift-ansible/tree/master/roles/openshift_prometheus) and the meaning of inv. vars is [here](https://github.com/openshift/openshift-ansible/blob/master/inventory/hosts.example).

```sh
# oc project openshift-metrics
# oc get all
NAME                      DESIRED   CURRENT   AGE
statefulsets/prometheus   1         1         9m

NAME                  HOST/PORT                                                     PATH      SERVICES       PORT      TERMINATION   WILDCARD
routes/alertmanager   alertmanager-openshift-metrics.apps.0206-hl6.qe.rhcloud.com             alertmanager   <all>     reencrypt     None
routes/alerts         alerts-openshift-metrics.apps.0206-hl6.qe.rhcloud.com                   alerts         <all>     reencrypt     None
routes/prometheus     prometheus-openshift-metrics.apps.0206-hl6.qe.rhcloud.com               prometheus     <all>     reencrypt     None

NAME              READY     STATUS    RESTARTS   AGE
po/prometheus-0   6/6       Running   0          9m

NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
svc/alertmanager   ClusterIP   172.25.166.106   <none>        443/TCP   9m
svc/alerts         ClusterIP   172.26.146.214   <none>        443/TCP   9m
svc/prometheus     ClusterIP   172.26.220.29    <none>        443/TCP   9m

```

Make `redhat` as a cluster admin. With browser, open `https://prometheus-openshift-metrics.apps.0206-hl6.qe.rhcloud.com` and login with `redhat`.

### Uninstallation

```sh
### Seems not working yet
$ ansible-playbook -i /tmp/2.file openshift-ansible/playbooks/openshift-prometheus/uninstall.yml
```

`oc delete project openshift-metrics` should be OK for test.


