# kubernetes-vagrant

## Requirements (versions):
- Virtualbox >= 6
- Vagrant >= 2.2.8
- Python 3
- Kubectl command (latest version)

# IP Addressing / Configuration:
IMPORTANT: This work exposes your vagrant nodes to the bridged network of your computer. This means that they will run on the same IP subnet / range as your computer.

Edit the Vagrantfile and configure the node info hash at the top to use IP addresses that are friendly with your local network. At this time the "mac" address fields are required but will change in the future.

Examples
```
dictNodeInfo = {
  'node0' => {
      'ip'       => "192.168.1.60",
      'mac'      => "08002799F0D4",
      'hostname' => "k8s-master-1",
      'role'     => "master"
  },
  'node1' => {
      'ip'       => "192.168.1.61",
      'mac'      => "080027F69436",
      'hostname' => "k8s-worker-1",
      'role'     => "worker"
  },
  'node2' => {
      'ip'       => "192.168.1.62",
      'mac'      => "080027D8A361",
      'hostname' => "k8s-worker-2",
      'role'     => "worker"
  },
}
```

Edit the "bride order" just below this so that vagrant knows which adapters to bridge networking to. Use the following section in the official Vagrant documentation to understand how this works: https://www.vagrantup.com/docs/networking/public_network#default-network-interface

Example:
```
bridgeOrder = [
  "en0: Wi-Fi (Wireless)",
]
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

## Some post-install stuff you can do to kick off your cluster's usability

Installing cert-manager (lets-encrypt)

NOTE: The following yml files located in the `k8s` directory have been pulled from other sources and are not covered under the license of this work. They are just there for examples of "kick starter" components that can be added to the cluster once its up and running.

```
# Cert-manager
kubectl apply -f k8s/cert-manager/cert-manager.yml

# You'll have to create this file - see https://docs.cert-manager.io/en/release-0.11/reference/clusterissuers.html for details
kubectl apply -f k8s/cert-manager/cert-manager-clusterissuer.yml
```

Installing kubed
```
# Kubed
kubectl apply -f k8s/kubed/kubed-v0.12.0.yaml
```

Setting up helm
```
# Setup RBAC
kubectl apply -f k8s/helm/rbac.yml

# Install helm
helm init

# Fix issue with tiller using default service account instead of tiller
kubectl --namespace kube-system patch deploy tiller-deploy -p "{\"spec\":{\"template\":{\"spec\":{\"serviceAccount\":\"tiller\"}}}}"

# Wait for tiller to come up
while [[ $(kubectl get pods -l name=tiller  -n kube-system -o 'jsonpath={..status.conditions[?(@.type=="Ready")].status}') != "True" ]]; do echo "waiting for pod" && sleep 1; done
```

# License
Copyright 2020 Jennifer Reed

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this work except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
