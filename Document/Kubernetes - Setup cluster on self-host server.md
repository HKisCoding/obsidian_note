## Cluster: 
Kubernetes cluster is **a set of nodes that run containerized applications**.
A cluster contains a control plane and one or more compute machines, or nodes.
The **control plane** maintain the desired state of the cluster:
- Which applications or workloads running 
- Docker images using 
- Resources 
- Other configuration 
**Nodes:** a virtual or physical machine --> Running Pods
**Pods:** Group of one or multiple containers that need to work together. Pods that are running inside Kubernetes are running on a private, isolated network. By default they are visible from other pods and services within the same Kubernetes cluster, but not outside that network.

## Tools: [minikube](https://minikube.sigs.k8s.io/docs/)
Quickly sets up a local Kubernetes cluster on macOS, Linux, and Windows.
Replacement for Docker Desktop 

### Create Kubernetes Cluster 
```shell 
minikube start
```
View Cluster details
```shell
kubectl cluster-info
```
View Nodes in the cluster 
```shell
kubectl get nodes
```

### Configure Kubernetes 
The kubectl config file is a configuration file that stores all the information necessary to interact with a Kubernetes cluster.

A context is a group of access parameters.
Each context contains a Kubernetes cluster, a user, and a namespace.
The current context is the cluster that is currently the default for kubectl:
all kubectl commands run against that cluster

To view the config file: 
```shell 
kubectl config view
```

Switching multiple Kubernetes clusters:
```shell 
kubectl config use-context
```

Check current context 
```shell 
kubectl config current-context
```

###  Deploy in minikube cluster 
Kubernetes will choose where to deploy application based on Node available resources.

Deploy app with kubernetes 
```shell
kubectl create deployment deploy_type/deploy_name --image= app_image_location 
```

Check application configures: 
Look for existing Pods:
```shell
kubectl get pods
```

To view what containers are inside that Pod and what images are used to build those containers / resources used: 
```shell
kubectl describe pods
```

View containers logs:
```shell
kubectl logs $POD_NAME
```

Expose the application outside the cluster: 
Use `NodePort` as parameter.
```shell
kubectl expose deploy_type/deploy_name --type="NodePort" --port $port
```

### Scale the deployment 
1. Expose the deployment 
2. Check Replica Set created by Deployment:
	Run `kubectl get deployments` 
	```shell
NAME                  READY   UP-TO_DATE   AVAILABLE   AGE
kubernetes-bootcamp   1/1     1            1           11m
```
-   _NAME_ lists the names of the Deployments in the cluster.
-   _READY_ shows the ratio of CURRENT/DESIRED replicas
-   _UP-TO-DATE_ displays the number of replicas that have been updated to achieve the desired state.
-   _AVAILABLE_ displays how many replicas of the application are available to your users.
-   _AGE_ displays the amount of time that the application has been running.

```shell 
kubectl get rs
```
3. Scale the Deployment 
```shell
kubectl scale deploy_type/deploy_name --replicas=4
```

Check number of Pods:
```shell
kubectl get pods -o wide
```
