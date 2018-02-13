# Prometheus

## doc

* [prometheus.io](https://prometheus.io/)
* [prometheus@github](https://github.com/prometheus)


## Prometheus@oc

* [doc@origin](https://github.com/openshift/origin/tree/master/examples/prometheus): useful prometheus queries.
* [p@oc-blog](https://blog.openshift.com/tag/prometheus/)
* [prometheus-architecture](https://prometheus.io/docs/introduction/overview/#architecture) and [gap-architecture](https://blog.openshift.com/monitoring-openshift-three-tools/)

### Concepts

* [data model](https://prometheus.io/docs/concepts/data_model/): time series, metric names, labels, [jobs and instances](https://prometheus.io/docs/concepts/jobs_instances/), samples
* Metrics types: [histogram](https://prometheus.io/docs/concepts/metric_types/#histogram) and [its practice](https://prometheus.io/docs/practices/histograms/)

### Installation

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

Make `redhat` as a cluster admin. With browser, open `https://prometheus-openshift-metrics.apps.0206-hl6.qe.rhcloud.com` and login with `redhat`. Then have fun with [prometheus queries](https://github.com/hongkailiu/svt-case-doc/blob/master/learn/prometheus.md).

* names of metrics: the dropdown menu besides the Execute button contains the names of metrics.
* type of a metic: ???
* labels of a metric: search using the name of the metric and its labels shows up as return

Use Prometheus rest API:

```sh
### Copy cookies and url from browser's developer tool.
### Query openshift_build_info metric
$ curl -k --cookie "f81241e3a913aa890fb02ba92a29f1be=a08156ec5229f6eb1fda52ff623fdbfc; _oauth_proxy=cmVkaGF0QGNsdXN0ZXIubG9jYWw=|1517942370|9TAlGQ4L1j3oI5rsgutBwDpRrUI=" "https://prometheus-openshift-metrics.apps.0206-hl6.qe.rhcloud.com/api/v1/query_range?query=openshift_build_info&start=1517943409.293&end=1518029809.293&step=345&_=1517942400799"
```

Check the config file for Prometheus:

```sh
# oc rsh -c prometheus prometheus-0
sh-4.2$ ps auxwww | grep prometheus | grep config
1000130+      1  0.6  1.4 349568 229172 ?       Ssl  16:41   1:58 /bin/prometheus --storage.tsdb.retention=6h --config.file=/etc/prometheus/prometheus.yml --web.listen-address=localhost:9090

sh-4.2$ cat /etc/prometheus/prometheus.yml
sh-4.2$ ls /etc/prometheus/
prometheus.rules  prometheus.yml

### Those 2 files are provided by a configMap
# oc get configMap prometheus -o yaml

# curl --key /etc/origin/master/admin.key --cert /etc/origin/master/admin.crt -k https://127.0.0.1:8444/metrics
```

### Uninstallation

```sh
### Seems not working yet
$ ansible-playbook -i /tmp/2.file openshift-ansible/playbooks/openshift-prometheus/uninstall.yml
```

`oc delete project openshift-metrics` should be OK for test.

### Storage related metrics
Docs

* [g-doc](https://docs.google.com/document/d/1Fh0T60T_y888LsRwC51CQHO75b2IZ3A34ZQS71s_F0g/edit#heading=h.ys6pjpbasqdu)
* k8s-design: [cloudprovider-storage-metrics](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/cloud-provider/cloudprovider-storage-metrics.md); [volume-metrics](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/storage/volume-metrics.md)

Test:

Metrics names: 

* cloudprovider-storage-metrics: `cloudprovider_aws_api_request_duration_seconds` and `cloudprovider_aws_api_request_errors`
* volume-metrics: `storage_operation_duration_seconds` and `storage_operation_errors_total`
  
Use `storage_operation_duration_seconds` in the prometheus UI, then auto-completion show there are 3 metrics: The basename of the metric is `storage_operation_duration_seconds` and histogram stores 3 metrics for histogram type.

* storage_operation_duration_seconds_count: number of values
* storage_operation_duration_seconds_sum: sum of values
* storage_operation_duration_seconds_bucket: number of values in the bucket

Let us focus on count with this metrics `storage_operation_duration_seconds_count{instance="172.31.49.63:8444",job="kubernetes-controllers",operation_name="volume_provision",volume_plugin="kubernetes.io/aws-ebs"}`. The value is `1`. This means k8s has 1 `volume_provision`. How much time did it take to perform the operations?  This is shown in `storage_operation_duration_seconds_sum{instance="172.31.49.63:8444",job="kubernetes-controllers",operation_name="volume_provision",volume_plugin="kubernetes.io/aws-ebs"}` as value `0.749798528`.

`storage_operation_duration_seconds_bucket{instance="172.31.49.63:8444",job="kubernetes-controllers",operation_name="volume_provision",volume_plugin="kubernetes.io/aws-ebs"}` means the number of values in the bucket. The lowerbound is set up by the server side. Since our only value is `0.749798528`, `storage_operation_duration_seconds_bucket{instance="172.31.49.63:8444",job="kubernetes-controllers",le="0.5",operation_name="volume_provision",volume_plugin="kubernetes.io/aws-ebs"}` has value 0 while `storage_operation_duration_seconds_bucket{instance="172.31.49.63:8444",job="kubernetes-controllers",le="0.5",operation_name="volume_provision",volume_plugin="kubernetes.io/aws-ebs"}` has value 1.

Use cluster-loader to create pods with PVCs and check those numbers again. Things start to make sense.
