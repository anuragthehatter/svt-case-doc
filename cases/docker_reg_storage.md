# Storage Test for docker registery

* Push test: Run concurrent build test.
* Pull test: Run scale up/down test.

## PVC backed up by gp2 volume.

Cluster: 1 master, 1 infra, 2 compute nodes where computes are equipped with 300g IOPS (2000) docker device (/dev/sdb).

Set up gp2 PVC as registery storage volume: [steps](../learn/docker_registry.md#use-filesystem-driver-for-docker-registry).

## Restrict nodes for build pods (if needed)

[Set up by master-config](https://docs.openshift.org/latest/install_config/build_defaults_overrides.html#install-config-build-defaults-overrides):

```sh
# vi /etc/origin/master/master-config.yaml
admissionConfig:
  pluginConfig:
    BuildDefaults:
      configuration:
        apiVersion: v1
        env: []
        nodeSelector:
          build: build
        kind: BuildDefaultsConfig

# systemctl restart atomic-openshift-master*
```

Verify this by trigger a round build and then

```sh
# oc get pod --all-namespaces -o wide | grep "\-build"
### All build pods are pending
# oc label node ip-172-31-3-115.us-west-2.compute.internal build=build
# oc get pod --all-namespaces -o wide | grep "\-build"
### All build pods are running on the above labelled node

```

## Concurrent builds

[Start pbench](../learn/pbench.md#use-pbench-in-the-test)

Prepare the project with cluster-loader:

```sh
# cd svt/openshift_performance/ci/scripts/
# ../../../openshift_scalability/cluster-loader.py -v -f ../content/conc_builds_nodejs.yaml 
# ../../ose3_perf/scripts/build_test.py -z -n 2 -a
```

Watching the output of the above build script.


## Result



