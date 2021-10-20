# kubernetes-vagrant

Multi-Node Kubernetes Cluster running in Vagrant

## Pre-requisites:
* Operating System: Mac OSX or Linux
* Supported Hypervisors
  * [Virtualbox](https://www.virtualbox.org/) >= 6
  * [libvirt](https://libvirt.org/) on linux (KVM)
* [Vagrant](https://www.vagrantup.com/) >= 2.2.8
* Python 3
* [Ansible](https://docs.ansible.com/) >= 2.10.5
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

### Adjusting optional helm charts

Out of the box, kubernetes-vagrant will install some helm charts. You may flag these to install with `true` / `false` in the settings.yaml under the `helm` section:

```
######## Optional Helm Charts to be installed by ansible #######
helm:
  kubed: false
  prometheus: false
  loki: false
  promtail: false
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

Re-running ansible provisioner at any time:
```
vagrant provision
```

## Running kubectl commands

```
export KUBECONFIG=$(pwd)/provisioning/admin.conf
```

```
$ kubectl version --short | grep Server
Server Version: v1.19.0

$ kubectl cluster-info
Kubernetes master is running at https://192.168.1.60:6443
KubeDNS is running at https://192.168.1.60:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

# Accessing management features

* **Accessing Grafana**:

In order to get to grafana, you'll need to port-forward the grafana service to your computer and browse to it locally (http://localhost:8080)
```
# Port forward Grafana Console - http://localhost:8080/
kubectl -n monitoring port-forward service/kube-prometheus-stack-grafana 8080:80
```

NOTE: Viewing logs with loki in grafana:

> To see the logs, click Explore on the sidebar, select the Loki datasource in the top-left dropdown, and then choose a log stream using the Log labels button.

Ref to Loki Docs: [Click Here](https://grafana.com/docs/loki/latest/getting-started/grafana/#loki-in-grafana)

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

# Future Improvements
* Support for CentOS as alternative to Ubuntu
* Extend ansible code so it can deploy to RPis with Ubuntu
* Option for dedicated etcd node(s)
* Option for multi-master K8s nodes + LB (thinking Haproxy or Nginx)

# Version Management via Github Action (Tagging)

Module version is managed by the following github action: https://github.com/marketplace/actions/github-tag-bump


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
