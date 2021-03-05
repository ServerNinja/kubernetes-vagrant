# kubernetes-vagrant

Multi-Node Kubernetes Cluster running in Vagrant

[![asciicast](https://asciinema.org/a/UhPUqwsaCxrbXnZk19UxAAC3Z.svg)](https://asciinema.org/a/UhPUqwsaCxrbXnZk19UxAAC3Z)

## Pre-requisites:
* Operating System: Mac OSX (although not tested, this may also work on Linux)
* [Virtualbox](https://www.virtualbox.org/) >= 6
* [Vagrant](https://www.vagrantup.com/) >= 2.2.8
* Python 3
* [Ansible](https://docs.ansible.com/) >= 2.9
  * Installation:
  ``` pip install -r requirements.txt ```
* [Kubectl](https://kubernetes.io/docs/tasks/tools/) command (latest version)
* [helm](https://helm.sh/) v3

# IP Addressing / Configuration:
IMPORTANT: This work exposes your vagrant nodes to the bridged network of your computer. This means that they will run on the same IP subnet / range as your computer.

Edit the settings.yml file and configure the node settings for each hostname at the top to use IP addresses that are friendly with your local network.

NOTE: Currently this project only supports a single master at this time

Examples
```
node_info:
  MasterNode1:
      ip: 192.168.1.60
      #mac: 08002799F0D4 # <=== Optional Mac Address
      role: master       # <=== Must be either "master" or "worker"
      memory: 4096       # <=== Default to 1024 if not defined
      cpus: 1            # <=== Default to 1 if not defined
  WorkerNode1:
      ip: 192.168.1.61
      role: worker
      memory: 4096
  WorkerNode2:
      ip: 192.168.1.62
      role: worker
      memory: 4096
```

In the *"Network Settings"* section, edit the *"net_bride order"* just below this so that vagrant knows which adapters to bridge networking to. Use the following section in the official Vagrant documentation to understand how this works: https://www.vagrantup.com/docs/networking/public_network#default-network-interface

Example:
```
net_bridge_order:
  - "en0: Wi-Fi (Wireless)"
```

# Building Cluster
Run the following command to create the cluster:
```
vagrant up
```
NOTE: First time run will take a while to download the vagrant box file. Once its downloaded and cached, the cluster will be up in roughly 5 minutes.

Run the following command to destroy the cluster:
```
vagrant destroy -f
```

## Running kubectl commands

```
export KUBECONFIG=$(PWD)/provisioning/admin.conf
```

```
$ kubectl version --short | grep Server
Server Version: v1.19.0

$ kubectl cluster-info
Kubernetes master is running at https://192.168.1.60:6443
KubeDNS is running at https://192.168.1.60:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

# [Optional] Post-install stuff you can do to kick off your cluster's usability

**NOTE:** The helm values.yaml files and k8s manifests in this repo are tailored to install these components for this project and are not recommended for production configurations.

**Set up persistent volumes**:
```
# Create the "local-storage" storageClass
kubectl apply -f k8s/pv/storageClass.yml

# Creates 15 local storage PVs
kubectl apply -f k8s/pv/pv.yml
```

**[Prometheus / Grafana](https://prometheus.io/):**

```
kubectl create namespace monitoring

helm repo add kube-prometheus-stack https://prometheus-community.github.io/helm-charts

helm install kube-prometheus-stack kube-prometheus-stack/kube-prometheus-stack --namespace monitoring --values ./helm/prometheus/values.yaml
```

* **Accessing Grafana**:

In order to get to grafana, you'll need to port-forward the grafana service to your computer and browse to it locally (http://localhost:8080)
```
# Port forward Grafana Console - http://localhost:8080/
kubectl -n monitoring port-forward service/kube-prometheus-stack-grafana 8080:80
```

* **Accessing Prometheus Web Console**:

In order to get to prometheus, you'll need to port-forward the prometheus service to your computer and browse to it locally (http://localhost:9090)
```
# Port forward Prometheus Console - http://localhost:9090/
kubectl -n monitoring port-forward service/kube-prometheus-stack-prometheus 9090:9090
```

* **Accessing Alertmanager Web Console**:

In order to get to alertmanager, you'll need to port-forward the alertmanager service to your computer and browse to it locally (http://localhost:9093)
```
# Port forward Alertmanager Console - http://localhost:9093/
kubectl -n monitoring port-forward service/kube-prometheus-stack-alertmanager 9093:9093
```

**Grafana [Loki / Promtail](https://grafana.com/oss/loki/)**:
```
kubectl create namespace loki

helm repo add grafana https://grafana.github.io/helm-charts

helm install loki grafana/loki --namespace loki --values ./helm/loki/values.yaml

helm install promtail grafana/promtail --namespace loki --values ./helm/promtail/values.yaml
```


**Installing [kubed](https://appscode.com/products/kubed/)**

```
helm repo add appscode https://charts.appscode.com/stable/

helm install kubed appscode/kubed --namespace kube-system --values ./helm/kubed/values.yaml
```

**Installing [cert-manager](https://cert-manager.io/docs/) (lets-encrypt)**

[Cert-Manager Installation Instructions](https://cert-manager.io/docs/installation/kubernetes)


# License
Copyright 2021 Jennifer Reed

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this work except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.