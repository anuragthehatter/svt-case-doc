# code-generator

## doc

* https://blog.openshift.com/kubernetes-deep-dive-code-generation-customresources/
* https://github.com/openshift-evangelists/crd-code-generation

## How2Use

Example repo: [codegen](https://github.com/hongkailiu/test-go/tree/master/codegen)

```bash
$ git clone https://github.com/hongkailiu/test-go.git
$ cd test-go

$ make code-gen-clean
$ make code-gen

$ oc new-project ttt
$ oc create -f codegen/artifacts/crd.yaml
$ oc create -n ttt -f codegen/artifacts/cr.yaml

### glide up -v
$ make build-code-gen

$ ./build/example -kubeconfig ~/.kube/config
svtGo example with Size 2

```


## Debugging

```bash
### versions have to match
cat glide.yaml
- package: k8s.io/client-go
  version: kubernetes-1.12.1
- package: k8s.io/code-generator
  version: kubernetes-1.12.1
```