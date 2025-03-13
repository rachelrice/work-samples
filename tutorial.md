# Deploy Your First Kubernetes Cluster

The goal of this tutorial is for you to become familiar with the basics of [Kubernetes](https://kubernetes.io/), a popular platform for deploying applications.

This tutorial teaches you how to:

* Create a Kubernetes **cluster** using [Kind](https://kind.sigs.k8s.io/).
* Interact with the cluster using **kubectl**.
* Create an **application** configuration.
* **Deploy** an application to the cluster.
* **Access** the application.

# Prerequisites

* A package manager or [Go](https://go.dev/doc/install/) installed.
* [kind](https://kind.sigs.k8s.io/docs/user/quick-start#installation) installed.
* [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl) installed.
* [Docker Desktop](https://docs.docker.com/desktop/) installed and available.

# Create Cluster

Use **Kind** to create a Kubernetes cluster. Kind is a tool for developing and testing Kubernetes clusters locally. Kind consists of [Go](https://go.dev/) packages and [Docker](https://www.docker.com/) images.

Issue the following command to create a cluster.

```shell
kind create cluster
```

The command output shows the progress and informs you when the cluster is ready to use.

```shell
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.32.2) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/
```

## Get Cluster Status

Use the command line tool [kubectl](https://kubernetes.io/docs/reference/kubectl/) to get the Kubernetes cluster status. Kubectl uses the [Kubernetes API](https://kubernetes.io/docs/concepts/overview/kubernetes-api/) to communicate with a cluster's [control plane](https://kubernetes.io/docs/reference/glossary/?fundamental=true#term-control-plane). A control plane is a set of components that manage a Kubernetes cluster.

Issue the following command to get the control plane information for the cluster.

```shell
kubectl cluster-info --context kind-kind
```

The command output shows the Kubernetes control plane and CoreDNS IP addresses.

```shell
Kubernetes control plane is running at https://127.0.0.1:49993
CoreDNS is running at https://127.0.0.1:49993/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

> :information_source: INFO  
> The control plane and CoreDNS port (49993) is an example.

# Deploy Application

Create an **application configuration** in a [YAML](https://yaml.org/) file and deploy it to the Kubernetes cluster. Application configuration can contain a [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) and a [Service](https://kubernetes.io/docs/concepts/services-networking/service/). A Deployment describes the desired state of an application, while a Service describes an application's network settings.

## Create Application Configuration

Create a file called **app.yaml** with the following configuration.

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: web
  name: web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: web
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:1.0
        name: hello-app
        resources: {}
status: {}
---
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: web
  name: web
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: web
  type: NodePort
status:
  loadBalancer: {}
```

The file defines a **Deployment** and a **Service** for an application called **web**. The Deployment tells Kubernetes to keep one [Pod](https://kubernetes.io/docs/concepts/workloads/pods/) active with [Google's "Hello App" Docker image](https://console.cloud.google.com/artifacts/docker/google-samples/us/gcr.io/hello-app/sha256:173d532964a8cfd250b01878a46c18ccb303bcf21855b0fb9ac445bc486d1dbc/) deployed. The Service exposes the [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport) port.

## Apply Application Configuration

Issue the following command to apply the configuration defined in **app.yaml** to the Kubernetes cluster.

```shell
kubectl apply -f app.yaml
```

The command output confirms creation of the **Deployment** and **Service**.

```shell
deployment.apps/web created
service/web created
```

## Get Pod Status

Issue the following command to get the status of the **Pod**.

```shell
kubectl get pods --context kind-kind
```

The command output shows one active Pod with a name beginning with **web**.

```shell
NAME                   READY   STATUS    RESTARTS   AGE
web-75995f7dbf-95bql   1/1     Running   0          2m17s
```

> :information_source: INFO  
> The Pod name (web-75995f7dbf-95bql) is an example.

# Access Application

Get the name of the application's **Pod**, then forward the exposed **NodePort** port (8080) from the **Service** to port 8080 on your local network so you can access the application in your browser.

## Get Pod Name

Issue the following command to get the **Pod** name.

```shell
PODNAME=$(kubectl get pods --template '{{range .items}}{{.metadata.name}}{{end}}' --selector=app=web)
```

Issue the following command to display the Pod name.

```shell
echo $PODNAME
```

The command output shows the Pod name from [Get Pod Status](#get-pod-status).

```shell
web-75995f7dbf-95bql
```

> :information_source: INFO  
> The Pod name (web-75995f7dbf-95bql) is an example.

## Expose NodePort

Issue the following command to forward the exposed **NodePort** port 8080 to port 8080 on your local network.

```shell
kubectl port-forward $PODNAME 8080:8080
```

The command output shows that port forwarding is active.

```shell
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
```

## Visit Welcome Page

In a browser window, visit **localhost:8080**.

```
Hello, world!
Version: 1.0.0
Hostname: web-75995f7dbf-qfhgs
```

The browser displays the "Hello, world!" application, including the Pod name from [Get Pod Name](#get-pod-name).

# Cleanup

1. In the terminal window where you issued the port forwarding command, enter `Ctrl + C` to stop port forwarding.
1. Issue the following command to delete the Kubernetes cluster.

    ```shell
    kind delete cluster
    ```

    The command output confirms deletion of the cluster and nodes.

    ```shell
    Deleting cluster "kind" ...
    Deleted nodes: ["kind-control-plane"]
    ```
1. Delete the **app.yml** file.
1. Quit **Docker Desktop**.

# Next Steps

In this tutorial, you learned how to:

* Create a Kubernetes **cluster** using kind.
* Manage the Kubernetes cluster using **kubectl**.
* Define a **Deployment** and **Service** configuration.
* **Deploy** an application to the Kubernetes cluster.
* Find the name of the application **Pod**.
* Expose the application **port** with port forwarding.
* **Access** the application welcome page in your browser.

For future learning, we recommend the following tutorials:

* [Explore your Kubernetes application](https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-intro/)
* [Scale your Kubernetes application](https://kubernetes.io/docs/tutorials/kubernetes-basics/scale/scale-intro/)