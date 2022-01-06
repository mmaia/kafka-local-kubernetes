# Kafka on Kubernetes for local development

This document explains running single node local kafka infra on kubernetes it covers kafka, zookeeper and schema-registry in kubernetes using [Kind](https://kind.sigs.k8s.io/).

### Pre-reqs, install: 

- [Docker](https://docs.docker.com/get-docker/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl) 
- [Kind](https://kind.sigs.k8s.io/)

### Step by step instructions

1. Run kind specifying configuration: `kind create cluster --config=kind-config.yml`. This will start a kubernetes control plane + worker
2. Run kubernetes configuration for kafka `kubectl apply -f kafka-k8s`

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