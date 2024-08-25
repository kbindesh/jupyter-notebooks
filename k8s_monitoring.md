# K8s Monitoring and Logging

This chapter describes how to monitor Kubernetes cluster components and applications and get infrastructure-level, system-level, and application-level logs to serve as a source for log analytics or further troubleshooting.

In this chapter, we’re going to cover the following topics:
- Monitoring on a cluster node
- Monitoring applications on a Kubernetes cluster
- Managing logs at the cluster node and pod levels
- Managing container stdout and stderr logs

## 01. Monitoring on a cluster node

- In Kubernetes, Metrics Server collects CPU/memory metrics and to some extent adjusts the resources needed by containers automatically.
- Metrics Server collects those metrics every 15 seconds from the kubelet agent and then exposes them in the API server of the Kubernetes master via the Metrics API.
- Users can use the kubectl top command to access metrics collected by Metrics Server.
- At the time of writing this chapter, Metrics Server supports scaling up to 5,000 Kubernetes worker nodes.

### 1.1 Checking whether Metrics Server is installed

```
# To get the list of all the node in cluster
kubectl get nodes

# To check the metrics for the worker node (here minikube)
kubectl top node minikube

OR

kubectl get pods -n kube-system | grep metrics-server
```

### 1.2 Installing Metrics Server in your current Kubernetes cluster

- If you’re on a vanilla Kubernetes cluster, you can install Metrics Server by deploying a YAML definition
or through Helm charts; the latter will require Helm to be installed.

```
# Using a YAML manifest file
kubectl apply -f https://github.com/kubernetes-sigs/metricsserver/releases/latest/download/components.yaml

# YAML with HA version
kubectl apply -f https://github.com/kubernetes-sigs/metricsserver/releases/latest/download/high-availability.yaml
```

### 1.3 Using Helm charts

```
# Add a repo
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/

# Install the Helm charts
helm upgrade --install metrics-server metrics-server/metricsserver
```

### 1.4 Using minikube add-ons

```
minikube addons list
minikube addons list | grep metrics-server

# Enable minikube addons for metrics-server
minikube addons enable metrics-server

# To check the metrics-server components
kubectl get pod,svc -n kube-system
OR
kubectl get pods -n kube-system | grep metrics-server
```

## 02. Checking out CPU/memory metrics

```
# Syntax
kubectl top node <node name>

kubectl top node minikube
```

## 03. Monitoring applications on a Kubernetes cluster

### 3.1 Create a sample pod (app)

```
kubectl run nginx --image=nginx
```

### 3.2 Monitoring the resource usage of an application

```
kubectl top pod nginx

# For multicontainer Pod
kubectl top pod <pod_name> --containers

# to show all the metrics of all the Pods across different namespaces
k top pod -A | k top pod --all-namespaces

kubectl top pod --sort-by=cpu
kubectl top pod –-sort-by=memory
kubectl top pod -A --sort-by=memory
```

### 3.3 Checking application details

```
kubectl describe pod nginx
kubectl describe pod coredns-64897985d-brqfl -n kube-system
kubectl describe pod nginx > mypod.yaml
```

### 3.4 Monitoring cluster events

```
kubectl get events
kubectl get events --sort-by=.metadata.creationTimestamp

# To collect the events during a deployment
kubectl get events --watch
```

## 4. Managing logs at the cluster node and Pod levels

```
kubectl describe node minikube

# Checking the node status
kubectl get node minikube -o wide
```

## 5. Managing container stdout and stderr logs

- In the Unix and Linux OSs, there are three I/O streams, called STDIN, STDOUT, and STDERR.
- STDOUT is usually a command’s normal output, and STDERR is typically used to output error messages.
- Kubernetes uses the kubectl logs <podname> command to log STDOUT and STDERR.

```
kubectl logs nginx
```
