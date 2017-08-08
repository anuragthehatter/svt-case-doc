# Docker Registry

## Doc

* [Deploying a Registry on Existing Clusters](https://docs.openshift.com/container-platform/3.5/install_config/registry/deploy_registry_existing_clusters.html)

## Use filesystem driver for docker-registry

### Check the current setting (Optional)

```sh
# oc exec docker-registry-5-3skdd -- cat /etc/registryconfig/config.yml
```


### Create PVC for registry (assumes AWS dynamic provisioning)
Use [registry_pvc.yaml](../files/registry_pvc.yaml): 

```sh
# oc create -f registry_pvc.yaml 
# oc get pvc
# oc get pv
# oc volume deploymentconfigs/docker-registry --add --name=registry-storage -t pvc \
    --claim-name=registry --overwrite -m /registry
```

### Configure docker-registry to use filesystem
Use [registry_secret.yaml](../files/registry_secret.yaml)

```sh
# oc secrets new dockerregistry registry_secret.yaml
# oc volume dc/docker-registry --add --name=dockersecrets -m /etc/registryconfig --type=secret --secret-name=dockerregistry
# oc env dc/docker-registry REGISTRY_CONFIGURATION_PATH=/etc/registryconfig/registry_secret.yaml
```

### Set filesystem threads limit (Optional)

```sh
oc env dc/docker-registry REGISTRY_STORAGE_FILESYSTEM_MAXTHREADS=100
```
