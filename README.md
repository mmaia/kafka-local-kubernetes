# Kafka on Kubernetes for local development

This document explains running single node local kafka infra on kubernetes it covers kafka, zookeeper and
schema-registry in kubernetes using [Kind](https://kind.sigs.k8s.io/).

For now there are 2 full setups in this repo, bot using Kind to host the kubernetes control plane and workers, one is
using Kubernetes Storage Classes and the other is using the "old" Persistent Volumes and Persistent Volume Claims
explicitly.

1. storage-class-setup folder has the setup running with the default Kind Storage Class and Stateful sets.
2. pv-pvc-setup folder has the setup running with Persistent Volumes and Persistent Volume Claims explicitly.

### Pre-reqs, install:

- [Docker](https://docs.docker.com/get-docker/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)
- [Kind](https://kind.sigs.k8s.io/)

### Step by step instructions to run storage-class-setup

1. Open a terminal and cd to the storage-class-setup folder.
2. Create a folder called "tmp", this is where the storage will be automatically provisioned by the default Kind storage
   class.
3. Run kind specifying configuration: `kind create cluster --config=kind-config.yml`. This will start a kubernetes
   control plane + worker
4. Run kubernetes configuration for kafka `kubectl apply -f kafka-k8s`
5. When done stop kubernetes objects: `kubectl delete -f kafka-k8s` and then if you want also stop the kind cluster
   which:
   will also delete the storage on the host machine: `kind delete cluster`

### Step by step instructions to run pv-pvc-setup folder

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

## Useful kubectl and kind commands

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