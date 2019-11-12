---
description: >-
    [Contour](https://projectcontour.io) is an Ingress controller for Kubernetes that works by deploying the Envoy proxy as a reverse proxy and load balancer. Contour supports dynamic configuration updates out of the box while maintaining a lightweight profile.

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

When the command completes, we can leverage the following command to get information for our lab

```bash
lab-info
```

Copy your External FQDN to your editor window, we will leverage this later!

### Step 2: Install Project Contour

We will start by installing Project Contour in it's default state.

**NOTE - Specific to THIS Lab** - We will need to edit our Service afterwards to enable external IP access. Our instances are not configured for dynamic load balancers. This is not a typical step needed in most environments.

Run:

```bash
kubectl apply -f https://projectcontour.io/quickstart/contour.yaml
```

To confirm Contour has been installed, we can execute...

```bash
kubectl get pods, svc -A
```

Which will list all Pods and Services within our cluster.