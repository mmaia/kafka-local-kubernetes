# Kafka on Kubernetes for local development

This document explains running single node local kafka infra on kubernetes it covers kafka, zookeeper and
schema-registry in kubernetes using [Kind](https://kind.sigs.k8s.io/).

For now there are 3 full setups in this repo, all using Kind to host the kubernetes control plane and workers, one is
using Kubernetes Storage Classes, the other is using the "old" Persistent Volumes and Persistent Volume Claims
explicitly and there's also a setup using [Helm Charts](https://helm.sh/), which is, BTW, my favorite.

1. storage-class-setup folder has the setup running with the default Kind Storage Class and Stateful sets.
2. pv-pvc-setup folder has the setup running with Persistent Volumes and Persistent Volume Claims explicitly.
3. helm foler has the setup using Helm, it uses the same Kubernetes approach as 1. storage-class-setup using Helm 
   instead of plain Kubernetes which brings flexibility. The Helm setup is kept simple.


### Pre-reqs, install:

- [Docker](https://docs.docker.com/get-docker/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)
- [Kind](https://kind.sigs.k8s.io/)
- [Helm](https://helm.sh/) - Only required if you want to run the Helm setup.

### Step-by-step instructions to run storage-class-setup

1. Open a terminal and cd to the storage-class-setup folder.
2. Create a folder called "tmp", this is where the storage will be automatically provisioned by the default Kind storage
   class.
3. Run kind specifying configuration: `kind create cluster --config=kind-config.yml`. This will start a kubernetes
   control plane + worker
4. Run kubernetes configuration for kafka `kubectl apply -f kafka-k8s`
5. When done stop kubernetes objects: `kubectl delete -f kafka-k8s` and then if you want also stop the kind cluster
   which:
   will also delete the storage on the host machine: `kind delete cluster`

> After running `kind create...` command on step 3 above, if you have the images already downloaded in your local docker
> registry you should load them on Kind so it won't try to download the images every time, to do that use: 
> `kind load docker-image $image_name` i.e - `kind load docker-image confluentinc/cp-kafka:7.0.1`
> You can check the loaded messages entering one of the Kind docker containers(worker or control plane) and use `crictl images`

### Step-by-step instructions to run pv-pvc-setup folder

This setup gives you more control over where and how the local storage will be provisioned on the host computer but it's
slightly more complex as it uses the Persistent Volumes and Persistent Volume Claims explicitly.

1. Open a terminal and cd to the pv-pvc-setup folder.
2. Create the folders on your local host machine so the persistent volumes can be persisted to the file system and you
   will be able to restart the kafka and zookeeper cluster without loosing data from topic. Restarting the kind cluster
   will delete the contents of persistent volumes, this is by design how Kind works when having
   a `propagation: Bidirectional` configuration. To create the folders you'll need to have `uid` and `gid` from same
   user running kind cluster otherwise the persistent folders will not be properly persisted. i.e - make sure to
   create `tmp/kafka-data`, `tmp/zookeeper-data/data` and `tmp/zookeeper-data/log`
   from same level where the `kind-config.yaml` file you're running kind with or it won't work as expected.
3. Run kind specifying configuration: `kind create cluster --config=kind-config.yml`. This will start a kubernetes
   control plane + worker
4. Run kubernetes configuration for kafka `kubectl apply -f kafka-k8s`
5. When done stop kubernetes objects: `kubectl delete -f kafka-k8s` and then if you want also stop the kind cluster
   which:
   will also delete the storage on the host machine: `kind delete cluster`

> Check Kind docker containers with Kubernetes control-plane and worker running with `docker ps`


### Step-by-step to running with Helm

This setup gives you the same initial setup from the `storage-class-setup` with the added flexibility and convenience of 
Helm Charts. 

1. Open a terminal and cd to helm folder
2. Start Kind running: `kind create cluster --config=kind-config.yml`
3. Run `helm install ${pick_a_name} local-kafka-dev`. i.e - `helm install local-kafka local-kafka-dev`
4. When done stop kafka setup using `helm uninstall ${name_picked_step_3}`, you may also stop Kind if you want: `kind delete cluster`

## Useful kubectl commands

Check kubernetes pods, services, volumes health:

 - `kubectl get pods` - a list of all pods
 - `kubectl get svc` - a list of all services
 - `kubectl get pv` - a list of all persistent volumes
 - `kubectl get pvc` - a list of all persistent volume claims
 - `kubectl describe pod $pod_name` - describe a specific pod
 - `kubectl logs $pod_name` - get logs for a specific pod
 - `kubectl exec -it $pod_name -- bash` - enters container and run a bash shell in a specific pod

> The last command opens a terminal session in a running kubernetes container with bash context. Use `exit` to quit.

## Stop kubernetes and kind

- `kubectl delete -f kafka-local-config`
- `kind delete cluster`