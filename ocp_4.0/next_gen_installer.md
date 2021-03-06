# Next-Gen Installer for OCP 4.0

* [repo](https://github.com/openshift/installer)

## Doc

* [gdoc@oc-dev](https://docs.google.com/document/d/1j7bhLXT_cIAjpMh_x2jeegtpE7495Mj5A-EcQsgZEDo/edit)
* [libvirt.on.gce@mike](https://github.com/mffiedler/ocp-svt/blob/master/svt-notes/OCP4/openshift4-libvirt-ocp.md)
* ravi: [playbook](https://github.com/chaitanyaenr/ocp-automation/pull/2), [gdoc](https://docs.google.com/document/d/1NilGxOee6DU6_Yim7TgQx6nN51qc2CvFNDqxgv-1NQ4/edit)

## Steps

### AWS

* Get output of `gpg2 --export --armor hongkliu@redhat.com`. See [how2](tools/gpg.md) 

* Fill the form in the above gdoc to get an IAM account on aws.

Then the application got replied (James Russell's email), all the credentials for the IAM user are encrypted as a file
in the attachment. Decrypt by:

```bash
$ gpg2 -d ./hongkliu\@redhat.com.openshift-dev.credentials.txt.gpg
```

The password policy is rather strict and we need to (the first time you login) change it with the pattern like the one
sent to you by admin.

Create a jump node (fedora29):

```bash
$ aws ec2 run-instances --image-id  ami-07e40fe5cf09f0d68 \
     --security-group-ids sg-5c5ace38 --count 1 --instance-type m5.2xlarge --key-name id_rsa_perf \
     --subnet subnet-4879292d --block-device-mappings "[{\"DeviceName\":\"/dev/sda1\", \"Ebs\":{\"VolumeSize\": 30, \"VolumeType\": \"gp2\"}}]" \
     --query 'Instances[*].InstanceId' \
     --tag-specifications="[{\"ResourceType\":\"instance\",\"Tags\":[{\"Key\":\"Name\",\"Value\":\"qe-hongkliu-fedora29-test\"}]}]"

```

Run playbook to set up a jump node:

```bash
$ ansible-playbook -i "<jump_node_public_dns>," playbooks/install_fedora_ocp4.yaml -e "aws_access_key_id=aaa aws_secret_access_key=bbb"

```

Launch a cluster:

```bash
$ cd go/src/github.com/openshift/installer/
### removed script: https://github.com/openshift/installer/commit/aff2e983f9438717dec2d182799ba7250035912d
### download terraform into bin folder
(obsoleted 20181211) $ ./hack/get-terraform.sh
### build openshift-install binary in bin folder
$ ./hack/build.sh
### launch a cluster:
### the kerberos_id in the openshift_env.sh file
### will be used in the name of the created instances
### change it to your own id before launching a cluster
$ source /tmp/openshift_env.sh
$ bin/openshift-install create cluster --log-level=debug

```

This will give us a 3-master/3-worker cluster with a bootstrap node which got terminated when
the launching procedure is complete. You can use your kerberos_id to filter out
the instances on ec2 console. _NOTE_: there are other resources too created by the installer.
 

```bash
$ export KUBECONFIG=${PWD}/auth/kubeconfig
$ kubectl cluster-info
Kubernetes master is running at https://hongkliu-api.devcluster.openshift.com:6443

$ oc get node
NAME                           STATUS    ROLES     AGE       VERSION
ip-10-0-130-33.ec2.internal    Ready     worker    30m       v1.11.0+d4cacc0
ip-10-0-151-141.ec2.internal   Ready     worker    30m       v1.11.0+d4cacc0
ip-10-0-167-107.ec2.internal   Ready     worker    29m       v1.11.0+d4cacc0
ip-10-0-27-83.ec2.internal     Ready     master    34m       v1.11.0+d4cacc0
ip-10-0-34-184.ec2.internal    Ready     master    34m       v1.11.0+d4cacc0
ip-10-0-9-10.ec2.internal      Ready     master    34m       v1.11.0+d4cacc0

### ssh to master node
$ oc describe node ip-10-0-27-83.ec2.internal | grep ExternalDNS
  ExternalDNS:  ec2-54-197-216-70.compute-1.amazonaws.com

$ ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -i ~/.ssh/libra.pem  core@ec2-54-197-216-70.compute-1.amazonaws.com
$ sudo -i
# 

### ssh to worker nodes
### Since there is no public dns associated with worker nodes,
### we have to ssh to worker nodes from a master node as jump node.
### Also need to have the private key file (mode 0600) on the jump node 

```

Destroy a cluster:

```bash
$ source /tmp/openshift_env.sh 
$ bin/openshift-install destroy cluster --log-level=debug


```

Number of instance in a role:

```bash
$ ./bin/openshift-install create install-config
$ vi install-config.yml

```

Instance type: Not working yet. Track with [installer/671](https://github.com/openshift/installer/issues/671).

```bash
$ bin/openshift-install create manifests
$ vi tectonic/99_openshift-cluster-api_master-machines.yaml
$ vi tectonic/99_openshift-cluster-api_worker-machineset.yaml 


```

See the used AMI:

```bash
$ aws ec2 describe-images --owner 531415883065 --output text

```

TODO:

* device mapping of the instances
* what to do when create/destroy does not work.

### libvert

#### libvert on AWS
https://www.reddit.com/r/aws/comments/993zbz/nested_virtualization_within_ec2_need_advice/


### libvert on GCE

Get an GCE instance:

1. Use `redhat` (google-account) to register for a 10 node Tectonic license at [https://account.coreos.com/](https://account.coreos.com/)
2. Download the pull secret and save it as `openshift-pull-secret.json`
3. install gcloud-cli and configure it. See [how2](../cloud/gce/gce.md#google-cloud-cli)

```bash
$ INSTANCE_NAME=hongkliu-ocp40-ttt
$ gcloud compute instances create "${INSTANCE_NAME}" \
    --image-family openshift4-libvirt \
    --zone us-east1-c \
    --min-cpu-platform "Intel Haswell" \
    --machine-type n1-standard-8 \
    --boot-disk-type pd-ssd --boot-disk-size 256GB \
    --metadata-from-file openshift-pull-secret=openshift-pull-secret.json
    

$ gcloud compute --project "openshift-gce-devel" ssh --zone "us-east1-c" "${INSTANCE_NAME}"
### the first time to run the above command, it will generate the key files and save them in ~/.ssh folder
### afterwards, it will use the generated key files to do the ssh

$ ll ~/.ssh/g*
-rw-------. 1 hongkliu hongkliu 1675 Nov  6 16:42 /home/hongkliu/.ssh/google_compute_engine
-rw-r--r--. 1 hongkliu hongkliu  410 Nov  6 16:42 /home/hongkliu/.ssh/google_compute_engine.pub
-rw-r--r--. 1 hongkliu hongkliu  189 Nov  6 16:43 /home/hongkliu/.ssh/google_compute_known_hosts


### we can also use the external IP (got it from the host) and the pub key to ssh the instance
$ ssh -i ~/.ssh/google_compute_engine.pub hongkliu@35.231.72.97


```

Create OCP 4.0 cluster

```bash
$ create-cluster nested
### be patient

$ oc get pod --all-namespaces

```

Lots of puzzles there:
* what is so special of `openshift4-libvirt` images?
* which part enables nested virtualization?
* [packer](https://www.packer.io/) seems a cool tool. Want to learn it.


## SVT cases:

For python version of cluster loader and concurrent build scripts, we can 

* copy the kube config to the right path:

    ```
    $ cp ${PWD}/auth/kubeconfig ~/.kube/config
    ```

* [set up python virtualenv](../tools/python.md).

## 4.0 hacking day: 20181206
Email:
* Christopher Alfonso: OCP 4.0 Installer - Jump on in!
* Eric Paris: Thursday kinda hack day, Friday hack day. For every single engineer.

http://try.openshift.com/

Please everyone, do not all try to use us-east-1, spread out across all
4 us regions. PLEASE also bring your cluster down when you are
finished. (Hint: the installer can do that too)

No matter what you try you need to fill in your information on [this spreadsheet](https://docs.google.com/spreadsheets/d/1X0DIEN_uxTzKAtVkEOe9lffq81DCgz2LJ2shMQl4i80).


My steps:

Open http://try.openshift.com/ with chrome (logged in with id obtained from https://developers.redhat.com/).

```bash
### Create fedora29 as above and ssh to it after running the above playbook
### but use the released binary instead of building it from src
$ fdr.sh ec2-34-217-108-124.us-west-2.compute.amazonaws.com
### the content is copied from http://try.openshift.com/
$ vi ~/.secrets/openshift_pull_secret.json

$ mkdir bin
$ cd bin/
$ curl -LO https://github.com/openshift/installer/releases/download/v0.5.0/openshift-install-linux-amd64
$ mv openshift-install-linux-amd64 openshift-install
$ chmod +x ./openshift-install

$ cd
$ curl -LO https://releases.hashicorp.com/terraform/0.11.10/terraform_0.11.10_linux_amd64.zip
$ sudo dnf install unzip -y
$ unzip terraform_0.11.10_linux_amd64.zip 

$ mv terraform bin/

$ openshift-install version
openshift-install v0.5.0
Terraform v0.11.10


```

Open `https://openshift-dev.signin.aws.amazon.com/console` with Firefox. Login with aws-account
obtained above. Switch the region to `us-west-1`.

```bash
###Choose us-west-1 (N. Cal)
$ vi ~/.aws/config
$ vi /tmp/openshift_env.sh

$ source /tmp/openshift_env.sh
$ mkdir aaa
$ cd aaa
$ openshift-install create cluster --log-level=debug

### checking things with oc

$ source /tmp/openshift_env.sh 
$ openshift-install destroy cluster --log-level=debug
```

## 4.0 hacking day: 20181207
* No running of the playbook `install_fedora_ocp4.yaml`.
* Interactive way of installer.

My steps:

Open http://try.openshift.com/ with chrome (logged in with id obtained from https://developers.redhat.com/).

```bash
$ fdr.sh ec2-34-217-108-124.us-west-2.compute.amazonaws.com

$ mkdir bin
$ cd bin/
$ curl -LO https://github.com/openshift/installer/releases/download/v0.5.0/openshift-install-linux-amd64
$ mv openshift-install-linux-amd64 openshift-install
$ chmod +x ./openshift-install

$ cd
$ curl -LO https://releases.hashicorp.com/terraform/0.11.10/terraform_0.11.10_linux_amd64.zip
$ sudo dnf install unzip -y
$ unzip terraform_0.11.10_linux_amd64.zip 
$ rm terraform_0.11.10_linux_amd64.zip

$ mv terraform bin/

$ openshift-install version
openshift-install v0.5.0
Terraform v0.11.10


```

Open `https://openshift-dev.signin.aws.amazon.com/console` with Firefox. Login with aws-account
obtained above. Switch the region to `us-west-2` (Oregon).

```bash
###Choose us-west-2
### set up aws account as indicated by the James Russell's email ~/.aws/{config|credentials}
### use `us-west-2`
$ vi ~/.aws/config


$ mkdir aaa
$ cd aaa

$ openshift-install create cluster 
? Email Address <secret>@redhat.com
? Password [? for help] ******
? SSH Public Key /home/fedora/.ssh/libra.pub
? Base Domain devcluster.openshift.com
? Cluster Name hongkliu
? Pull Secret {"auths":{"cloud.openshift.com":{"auth":"bla=","email":"<secret>@redhat.com"},"quay.io":{"auth":"=","email":"<secret>@redhat.com"}}}
? Platform aws
? Region us-west-2
INFO Using Terraform to create cluster...         

### checking things with oc

$ openshift-install destroy cluster

```

Notice

* `SSH Public Key /home/fedora/.ssh/libra.pub` does not show when you do not have a public key in the `~/.ssh/` folder.
```bash
$ ll ~/.ssh/libra.p*
-rw-------. 1 fedora fedora 1675 Dec  7 16:27 /home/fedora/.ssh/libra.pem
-rw-rw-r--. 1 fedora fedora  381 Dec  7 16:27 /home/fedora/.ssh/libra.pub

```

* `~/.aws/{config|credentials}` use `default` section for creating the cluster:
 
 
 ```bash
$ cat ~/.aws/config 
[default]
region = us-west-2
[fedora@ip-172-31-46-165 ~]$ cat ~/.aws/credentials 
[default]
aws_access_key_id = aaa
aws_secret_access_key = bbb


```

If you change default to eg, `openshift-dev` as indicated in the James Russell's email. It won't
work unless you set up by `export AWS_PROFILE="openshift-dev"`.

* More info
    * (Not tried myself) Walid found it might be helpful if we delete (or move it to other folder for backup reason)
`~/.terraform.d/`.

    * (did not find the password) https://github.com/openshift/installer/issues/411#issuecomment-444620427


## installer 0.6.0: 20181213

* default aws profile; no need for terraform installation.
* console pod is running but URL `https://console-openshift-console.apps.hongkliu.devcluster.openshift.com/` is not working
    * tried in the afternoon, the url is working. The username and password is at the end of the output of installer.
    * it is an `okd` deployment from the login page and the link to the doc page.
    * seems logout does not work. ^_^

## installer 0.8.0: 20190102

Hit the error below. Ignoring it seems OK.

```bash
$ openshift-install create cluster
? SSH Public Key /home/fedora/.ssh/libra.pub
? Platform aws
? Region us-west-2
ERROR list hosted zones: Throttling: Rate exceeded
        status code: 400, request id: 21f65da3-0ec9-11e9-b6a3-39da04a3cd7d 
? Base Domain devcluster.openshift.com

```

## installer 0.9.1: 20190109

Using aws account from Aleks:

```
$ openshift-install version
openshift-install v0.9.1

$ openshift-install create cluster
? SSH Public Key /home/fedora/.ssh/libra.pub
? Platform aws
? Region us-east-2
? Base Domain qe.devcluster.openshift.com
? Cluster Name hongkliu
? Pull Secret [? for help] *************************************************************************************************
INFO Creating cluster...            

```

Note that the base domain `qe.devcluster.openshift.com` is auto-filled by the installer.

Tested with Vikas' instructions: _This will install OCP images instead of OKD images._
[art-notes](https://mojo.redhat.com/groups/atomicopenshift/blog/2019/01/11/ocp-40-high-touch-beta-htb-notes-from-art)

```
###1. Please have your pull secret from try.openshift,com ready before you do this.
###2. Login (with your github id) to https://api.ci.openshift.org/console/catalog and note your userid and login token
###3. oc login https://api.ci.openshift.org --token <api.ci login token>
###Step 4:
$ docker login -u $(oc whoami) -p $(oc whoami -t) registry.svc.ci.openshift.org

###Step 5:
$ export _OPENSHIFT_INSTALL_RELEASE_IMAGE_OVERRIDE=registry.svc.ci.openshift.org/ocp/release:4.0.0-0.nightly-2019-01-08-152529
$ export OPENSHIFT_INSTALL_RELEASE_IMAGE_OVERRIDE=registry.svc.ci.openshift.org/ocp/release:4.0.0-0.nightly-2019-01-08-152529

###Step 6: Thanks to Jianlin
$ cat ~/.docker/config.json 
{
	"auths": {
		"registry.svc.ci.openshift.org": {
			"auth": "<secret>"
		}
	}
}

###Add an auth for "registry.svc.ci.openshift.org" into the pull secret.

```

```
### without step 6: 20190109
$ openshift-install create cluster
? SSH Public Key /home/fedora/.ssh/libra.pub
? Platform aws
? Region us-west-2
ERROR list hosted zones: Throttling: Rate exceeded
        status code: 400, request id: e0035898-1452-11e9-baee-cf7f448afe36 
? Base Domain devcluster.openshift.com
? Cluster Name hongkliu
? Pull Secret [? for help] *************************************************************************************************WARNING Found override for ReleaseImage. Please be warned, this is not advised 
INFO Creating cluster...                          
INFO Waiting up to 30m0s for the Kubernetes API... 
FATAL waiting for Kubernetes API: context deadline exceeded 

```

```
$ openshift-install create cluster --dir=./20190110.vikas
? SSH Public Key /home/fedora/.ssh/libra.pub
? Platform aws
? Region us-west-2
ERROR list hosted zones: Throttling: Rate exceeded
        status code: 400, request id: 19a9f38a-14e7-11e9-889f-b9e1f6d95d40 
? Base Domain devcluster.openshift.com
? Cluster Name hongkliu
? Pull Secret [? for help] *************************************************************************************************WARNING Found override for ReleaseImage. Please be warned, this is not advised 
INFO Creating cluster...                          
INFO Waiting up to 30m0s for the Kubernetes API... 
INFO API v1.11.0+406fc897d8 up                    
INFO Waiting up to 30m0s for the bootstrap-complete event... 
INFO Destroying the bootstrap resources...        
INFO Waiting up to 10m0s for the openshift-console route to be created... 
INFO Install complete!                            
INFO Run 'export KUBECONFIG=/home/fedora/20190110.vikas/auth/kubeconfig' to manage the cluster with 'oc', the OpenShift CLI. 
INFO The cluster is ready when 'oc login -u kubeadmin -p <secret>' succeeds (wait a few minutes). 
INFO Access the OpenShift web-console here: https://console-openshift-console.apps.hongkliu.devcluster.openshift.com 
INFO Login to the console with user: kubeadmin, password: <secret>

```

Checking:

```
$ oc get clusterversion version -o json | jq .status.current
{
  "payload": "registry.svc.ci.openshift.org/ocp/release@sha256:f820eaad16c66f08fe53acfccd9c27a665c36cf714aeefba44dc923432ac840e",
  "version": "4.0.0-0.nightly-2019-01-08-152529"
}

### some cool 4.0 commands:
$ oc adm release info --pullspecs
$ oc adm release extract --from=registry.svc.ci.openshift.org/ocp/release:4.0.0-0.nightly-2019-01-10-030048 --to=./abc

```

## troubleshooting

* From Mike: tip  - waiting for bootstrap complete too long -> ssh into the bootstrap node and journalctl the kubelet service.
* [troubleshooting.doc@github](https://github.com/openshift/installer/blob/master/docs/user/troubleshooting.md)