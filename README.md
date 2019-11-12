---
description: >-
    Contour is an Ingress controller for Kubernetes that works by deploying the Envoy proxy as a reverse proxy and load balancer. Contour supports dynamic configuration updates out of the box while maintaining a lightweight profile.

    Contour also introduces a new ingress API (HTTPProxy) which is implemented via a Custom Resource Definition (CRD). Its goal is to expand upon the functionality of the Ingress API to allow for a richer user experience as well as solve shortcomings in the original design.
---

# Project Contour: Introduction

### Key Capabilities

* **Secure Ingress Delegation** - Ingress Delegation allows multiple cluster operator teams to self-manage service access across multiple namespaces in a cluster. This functionality protects users from assigning the same path or route configuration to different services which could cause significant outages.
* **Dynamic Reconfiguration** - Updates to Envoy configurations can be applied "live" without the need to restart a load balancer.

### Environment Overview

Your lab environment is hosted within public cloud. It is a single node Kubernetes instance. All necessary binaries have been installed for Kubernetes to function on this instance. In order to access this environment we will be leveraging a platform called [Strigo.io](https://strigo.io). When accessing via Strigo, you will notice you have several windows available to assist you with this lab.

* Terminal Windows
* Code Editor Window
* Web Browser Windows

## Lab Goals

In this lab, we will be working with a simple HTTPProxy configuration for allowing access to a web based application hosted within a Kubernetes Cluster. We will Install and Configure Project Contour, install our custom application, and configure access to the service which is created as part of the application deployment. We will then add an additional HTTPProxy route to allow access to a secondary service.

### Step 1: Start Kubernetes

Execute the following command to get started!

```bash
k8s-start
```

### Step 2: Copy Information

**Run:**

```bash
lab-info
```

Copy down your **Public Hostname** and **Public IP**. We will use this later.

### Step 2: Install Project Contour

We will start by installing Project Contour in it's default state.

**Run:**

```bash
kubectl apply -f https://projectcontour.io/quickstart/contour.yaml
```

To confirm Contour has been installed, we can execute...

```bash
kubectl get pods, svc -A
```

Which will list all Pods and Services within our cluster.

**Important Note - Specific to THIS Lab** - We will need to edit our Service afterwards to enable external IP access. Our instances are not configured for dynamic load balancers. This is not a typical step needed in most environments.

```bash
kubectl edit svc envoy -n projectcontour
```

Here we will edit our service to allow external access to our Kubernetes Node External IP on Port 80. We will need to force an update to the Envoy proxy service.

```bash
kubectl apply -f https://gist.githubusercontent.com/codyde/5cc4eea515dba6970ef7e39848b73042/raw/7b203ae926e50d68ff75116212bba4aef327691a/envoy-update.yaml --force
```

Once this is applied, we will need to patch in our externalIP configuration. **Replace the REPLACEME phrase below with the External IP you wrote down earlier

```bash
kubectl patch svc envoy -p '{"spec": {"externalIPs": ["REPLACEME"]}}'
```

Finally, we can confirm our service is configured by running 

```bash
kubectl get svc -n projectcontour
```

You should see the envoy service listed running on ports 80 and 443, example output is below!

```bash
NAME      TYPE        CLUSTER-IP       EXTERNAL-IP    PORT(S)          AGE
contour   ClusterIP   10.96.102.194    <none>         8001/TCP         11m
envoy     ClusterIP   10.106.231.185   13.57.49.145   443/TCP,80/TCP   2m41s
```