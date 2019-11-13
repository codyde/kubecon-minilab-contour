---
description: >-
    Contour is an Ingress controller for Kubernetes that works by deploying the Envoy proxy as a reverse proxy and load balancer. Contour supports dynamic configuration updates out of the box while maintaining a lightweight profile.

    Contour also introduces a new ingress API (HTTPProxy) which is implemented via a Custom Resource Definition (CRD). Its goal is to expand upon the functionality of the Ingress API to allow for a richer user experience as well as solve shortcomings in the original design.
---

# Project Contour: Introduction

### Key Capabilities

* **Secure Ingress Delegation** - Ingress Delegation allows cluster operator teams to self-manage service access across multiple namespaces in a cluster. Enabled by limiting which namespaces have access to configure virtualhosts and TLS credentials.
* **Dynamic Reconfiguration** - Updates to Envoy configurations can be applied "live" without the need to restart a load balancer.
* **Enhanced Load Balancing Capabilities** - Load balance multiple services from a single route, as well as allow service weighting and other load balancing strategies without leveraging annotations.

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

### Step 3: Install Project Contour

We will start by installing Project Contour in it's default state.

**Run:**

```bash
kubectl apply -f https://projectcontour.io/quickstart/contour.yaml
```

To confirm Contour has been installed, we can execute the following command which will list all Pods and Services within our cluster.

```bash
kubectl get pods, svc -A
```

**_Important Note - Specific to This Lab_** - We will need to edit our Service afterwards to enable external IP access. Our instances are not configured for dynamic load balancers. This is not a typical step needed in most environments.

We will pull down a local copy of the updated Envoy service YAML and replace the externalIPs section with the external IP for our Kubernetes instance. We will assign an environment variable to the External IP address to make this easier.

```bash
export externalip=$(curl -s http://169.254.169.254/latest/meta-data/public-ipv4)
wget https://gist.githubusercontent.com/codyde/5cc4eea515dba6970ef7e39848b73042/raw/e925ca9ec0d623572c1aa768cc0287f904f87b0a/envoy-update.yaml
sed -i 's/REPLACEME/'"$externalip"'/g' envoy-update.yaml
kubectl apply -f envoy-update.yaml --force
```

After these are applied, we can confirm our service has been updated with the external IP by entering...

```bash
kubectl get svc -n projectcontour
```

You should see the envoy service listed running on ports 80 and 443, example output is below (your IP will be different).

```bash
NAME      TYPE        CLUSTER-IP       EXTERNAL-IP    PORT(S)          AGE
contour   ClusterIP   10.96.102.194    <none>         8001/TCP         11m
envoy     ClusterIP   10.106.231.185   13.57.49.145   443/TCP,80/TCP   2m41s
```

If all was configured correct, you'll see the nodes External IP listed for connectivity.

### Step 4: Deploying our Demo Application

Our application is a simple single pod application running in Flask. We can deploy it using the below command.

```bash
kubectl apply -f https://gist.githubusercontent.com/codyde/a6f67354431de6c3f9a0a0dd1cceada5/raw/36cdf40ec72ce86048a52b08cb0adc6fe9e12dc8/kubecon-base.yaml
```
This YAML deploys the pod and creates a service, however this service is not exposed. We will use Contour to create an HTTPProxy route inbound to the service

### Step 5: Creating Our First HTTPProxy Route

In this step we will create our HTTPProxy object to route to our services. In the following series of commands we will export our external hostname to an environment variable, copy down a HTTPProxy configuration YAML, update it with our External FQDN, and then apply the configuration.

```bash
export externalfqdn=$(curl -s http://169.254.169.254/latest/meta-data/public-hostname)
wget https://gist.githubusercontent.com/codyde/b8efa88f24c167403cf9a09e84126462/raw/741a756f89eae0d29f04302450d7ffbc9fba505c/contour-route.yaml
sed -i 's/REPLACEME/'"$externalfqdn"'/g' contour-route.yaml
kubectl apply -f contour-route.yaml
```

If we inspect the `contour-route.yaml` file, we see the following details...

```yaml
apiVersion: projectcontour.io/v1
kind: HTTPProxy
metadata:
  name: kubecon
  namespace: default
spec:
  virtualhost:
    fqdn: your.instance.fqdn
  routes:
    - conditions:
      - prefix: /
      services:
        - name: kubecon-minilab-svc
          port: 80
```

This creates an HTTPProxy object that creates a route to the kubecon-minilab-svc on port 80. Running he below command allows us to observe any other HTTPProxy objects created in our environment.

```bash
kubectl get httpproxy
```

In Strigo, we have created a browser interface that is pointed to your cloud instance. If you refresh this page, you should see our demo application resolve. This application has several external links on the top (for things like Kubernetes.Academy, etc...), but also has a second set of links below it to other parts of our application. These links are currently not functional and clicking them will cause the application to fail to load. There are 2 reasons for this... 

* The core application is responding on the default route ("/"), and the application itself doesn't have anything listening on the other paths listed
* No other routes are defined to send traffic to other services which may be able to handle the other paths in question (i.e. "/kubecon" or "/marketing")

Since our application is broken up into multiple services, let's add additional pods, services, and Contour HTTPProxy routes to support the connectivity

### Step 6: Adding Additional HTTPProxy Routes

```bash

```

### Step 7: Replacing Our Service (Blue/Green Deployment)

