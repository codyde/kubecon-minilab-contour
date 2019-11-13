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
kubectl apply -f https://gist.githubusercontent.com/codyde/a6f67354431de6c3f9a0a0dd1cceada5/raw/7d752bbcc0f085026010314d821b764608c06d4c/kubecon-base.yaml
```

This YAML deploys all the necessary pods and services for this mini-lab. Our application consists of 4 pods, and 4 services. These services are not exposed externally. We will use Contour to create an HTTPProxy inbound to the various services. This enables us to leverage a single externally exposed endpoint (Contour) to control access to our application!

### Step 5: Creating Our First HTTPProxy

In this step we will create our HTTPProxy object to route to our services. In the following series of commands we will export our external hostname to an environment variable, copy down a HTTPProxy configuration YAML, update it with our External FQDN, and then apply the configuration.

```bash
export externalfqdn=$(curl -s http://169.254.169.254/latest/meta-data/public-hostname)
wget https://gist.githubusercontent.com/codyde/b8efa88f24c167403cf9a09e84126462/raw/260ccbe57f36ff9a9dc79c6c3195b85cd59b36d3/contour-route.yaml
sed -i 's/REPLACEME/'"$externalfqdn"'/g' contour-route.yaml
kubectl apply -f contour-route.yaml
```

If we inspect the `contour-route.yaml` file, we see the following details...

```yaml
apiVersion: projectcontour.io/v1
kind: HTTPProxy
metadata:
  name: basic
spec:
  virtualhost:
    fqdn: REPLACEME
  routes:
  - services:
    - name: kubecon-minilab-app-svc
      port: 80
    conditions:
      - prefix: /
```

This creates an HTTPProxy object that creates a route to the main kubecon-minilab-app-svc on port 80. Running the below command allows us to observe any other HTTPProxy objects created in our environment.

```bash
kubectl get httpproxy
```

In Strigo, we have created a browser interface that is pointed to your cloud instance. If you refresh this page, you should see our demo application resolve. This application has several external links on the top (for things like Kubernetes.Academy, etc...), but also has a second set of links on the bottom to other parts of our application. These cards are currently not functional and clicking them will cause the application to fail to load. There are 2 reasons for this...

* The core application is responding on the default route ("/"), and the application itself doesn't have anything listening on the other paths listed
* No other routes are defined to send traffic to other services which may be able to handle the other paths in question (i.e. "/kubecon" or "/kubecon")

Since our application is broken up into multiple services, let use Contour's HTTPProxy CRD to route to the services we created in the beginning.

### Step 6: Exposing Additional Routes with HTTPProxy

Execute the following commands to pull down, update, and apply an updated Contour HTTPProxy configuration

```bash
export externalfqdn=$(curl -s http://169.254.169.254/latest/meta-data/public-hostname)
wget https://gist.githubusercontent.com/codyde/b6c7e4e0a5e96d7daebfd0c0725fc3a0/raw/9d71462dd3d9f9866278b53a41a1e61d4133d853/contour-route-2.yaml
sed -i 's/REPLACEME/'"$externalfqdn"'/g' contour-route-2.yaml
kubectl apply -f contour-route-2.yaml
```

This deploys the following configuration

```yaml
apiVersion: projectcontour.io/v1
kind: HTTPProxy
metadata:
  name: basic
spec:
  virtualhost:
    fqdn: REPLACEME
  routes:
  - services:
    - name: kubecon-minilab-app-svc
      port: 80
    conditions:
      - prefix: /
  - services:
    - name: kubecon-minilab-ll-svc
      port: 80
    conditions:
      - prefix: /minilabs
```

If we refresh our application and select the mini-lab card, it should resolve successfully!

#### Includes and Delegation

We can also leverage `include` blocks in our HTTPProxy configuration to bring in additional configurations. Let's use includes to bring in the configurations for our other 2 cards, KubeCon and Build Run Manage.

```bash
export externalfqdn=$(curl -s http://169.254.169.254/latest/meta-data/public-hostname)
wget https://gist.githubusercontent.com/codyde/b6c7e4e0a5e96d7daebfd0c0725fc3a0/raw/9d71462dd3d9f9866278b53a41a1e61d4133d853/contour-route-2.yaml
sed -i 's/REPLACEME/'"$externalfqdn"'/g' contour-route-2.yaml
kubectl apply -f contour-route-2.yaml
```

This applies the following configuration, consisting of a main HTTPProxy configuration, with 2 included configurations.

```yaml
apiVersion: projectcontour.io/v1
kind: HTTPProxy
metadata:
  name: basic
spec:
  virtualhost:
    fqdn: REPLACEME
  includes:
  - name: kubecon
    namespace: default
    conditions:
    - prefix: /kubecon
  - name: build-run-manage
    namespace: default
    conditions:
    - prefix: /brm
  routes:
  - services:
    - name: kubecon-minilab-app-svc
      port: 80
    conditions:
      - prefix: /
  - services:
    - name: kubecon-minilab-ll-svc
      port: 80
    conditions:
      - prefix: /minilabs
---
apiVersion: projectcontour.io/v1
kind: HTTPProxy
metadata:
  name: kubecon
  namespace: default
spec:
  routes:
    - services:
        - name: kubecon-minilab-kc-svc
          port: 80
---
apiVersion: projectcontour.io/v1
kind: HTTPProxy
metadata:
  name: build-run-manage
  namespace: default
spec:
  routes:
    - services:
        - name: kubecon-minilab-brm-svc
          port: 80
```

With this configuration applied, all of our card should be active and able to be clicked through!

### Step 7: Replacing Our Service (Blue/Green Deployment)

