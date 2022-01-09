# Kafka on Kubernetes for local development

This document explains running single node local kafka infra on kubernetes it covers kafka, zookeeper and schema-registry in kubernetes using [Kind](https://kind.sigs.k8s.io/).

For now there are 2 full setups in this repo, bot using Kind to host the kubernetes control plane and workers, one is using Kubernetes Storage Classes and the other is using the "old" Persistent Volumes and Persistent Volume Claims explicitly.

1. storage-class-setup folder has the setup running with Storage Class. (WIP - Not working yet)
2. pv-pvc-setup folder has the setup running with Persistent Volumes and Persistent Volume Claims explicitly. This is working,
to use it cd to the pv-pvc-setup folder and follow the instructions below.

### Pre-reqs, install: 

- [Docker](https://docs.docker.com/get-docker/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl) 
- [Kind](https://kind.sigs.k8s.io/)

### Step by step instructions

1. Create the folders on your local host machine so the persistent volumes can be persisted to the file system and you will
be able to restart the kafka and zookeeper cluster without loosing data from topic. Restarting the kind cluster will delete 
the contents of persistent volumes, this is by design how Kind works when having a `propagation: Bidirectional` configuration. 
To create the folders you'll need to have `uid` and `gid` from same user running kind cluster otherwise the persistent folders
will not be properly persisted. i.e - make sure to create `tmp/kafka-data`, `tmp/zookeeper-data/data` and `tmp/zookeeper-data/log`
from same level where the `kind-config.yaml` file you're running kind with or it won't work as expected.
2. Run kind specifying configuration: `kind create cluster --config=kind-config.yml`. This will start a kubernetes control plane + worker
3. Run kubernetes configuration for kafka `kubectl apply -f kafka-k8s`

> Check Kind docker containers with Kubernetes control-plane and worker running with `docker ps`


## Useful kubectl and kind commands

Check kubernetes pods, services, volumes health: 

    - kubectl get pods
    - kubectl get svc
    - kubectl get pv
    - kubectl get pvc
    - kubectl describe pod $pod_name
    - kubectl logs $pod_name
    - kubectl exec -it $pod_name -- bash

> The last command opens a terminal session in a running kubernetes container with bash context. Use `exit` to quit.


## Stop kubernetes and kind 
   - `kubectl delete -f kafka-local-config`
   - `kind delete cluster`